# Powersehll101\ Chapter 9 - Functions
## Naming
>   When naming your functions in PowerShell, use a Pascal case name with an approved verb and a singular noun.
    I also recommend prefixing the noun. For example: `<ApprovedVerb>-<Prefix><SingularNoun>`.

- _In PowerShell, there's a specific list of approved verbs that can be obtained by running `Get-Verb`._
```ps1
    Get-Verb | Sort-Object -Property Verb
```
_It's important to choose an approved verb in PowerShell when functions are added to a module.
The module generates a warning message at load time if you choose an unapproved verb._
_Unapproved verbs also limit the discoverability of your functions._

## A simple function
```ps1
    function Get-Version {
        $PSVersionTable.PSVersion
    }
```
- Once loaded into memory, you can see functions on the Function PSDrive.
```ps1
    Get-ChildItem -Path Function:\Get-*Version
```
_If you want to remove these functions from your current session,
you'll have to remove them from the Function PSDrive or close and reopen PowerShell._
```ps1
    Get-ChildItem -Path Function:\Get-*Version | Remove-Item
```
_Verify that the functions were indeed removed._
```ps1
    Get-ChildItem -Path Function:\Get-*Version
```

- If the functions were loaded as part of a module, the module can be unloaded to remove them.
```ps1
    Remove-Module -Name <ModuleName>
```
_*The `Remove-Module` cmdlet removes modules from memory in your current PowerShell session,
it doesn't remove them from your system or from disk._

## Parameters
```ps1
    function Test-MrParameter {

    param (
            $ComputerName
        )

    Write-Output $ComputerName

    }
```

- I'll create a function to query all of the commands on a system and return the number of them that have specific parameter names.
```ps1
    function Get-MrParameterCount {
        param (
            [string[]]$ParameterName
        )

    foreach ($Parameter in $ParameterName) {
            $Results = Get-Command -ParameterName $Parameter -ErrorAction SilentlyContinue

    [pscustomobject]@{
                ParameterName = $Parameter
                NumberOfCmdlets = $Results.Count
            }
        }
    }
```
```ps1
    Get-MrParameterCount -ParameterName ComputerName, Computer, ServerName, Host, Machine
```
**Output**
```
    ParameterName NumberOfCmdlets
    ------------- ---------------
    ComputerName               39
    Computer                    0
    ServerName                  0
    Host                        0
    Machine                     0
```
_I also recommend using the same case for your parameter names as the default cmdlets.
Use `ComputerName`, not `computername`._

>   The param statement allows you to define one or more parameters.
    The parameter definitions are separated by a `comma (,)`. For more information, see about_Functions_Advanced_Parameters.

# Advanced Functions
>    Advanced functions have a number of common parameters that are added to the function automatically.
    These common parameters include parameters such as Verbose and Debug.

- There are a couple of different ways to see the common parameters. One is by viewing the syntax using `Get-Command`.
```ps1
    Get-Command -Name Test-MrParameter -Syntax
```
**Output:**
```ps1
    Test-MrParameter [[-ComputerName] <Object>]
```

- *Another is to drill down into the parameters with `Get-Command`.
```ps1
    (Get-Command -Name Test-MrParameter).Parameters.Keys
```
**Output:**
```ps1
    ComputerName
```

- *Add `CmdletBinding` to turn the function into an advanced function.
```ps1
    function Test-MrCmdletBinding {

    [CmdletBinding()] #<<-- This turns a regular function into an advanced function
        param (
            $ComputerName
        )

    Write-Output $ComputerName

    }
```
- *Adding `CmdletBinding` adds the common parameters automatically.
  `CmdletBinding` requires a `param` block, but the `param` block can be empty.
```ps1
    Get-Command -Name Test-MrCmdletBinding -Syntax
```
**Output:**
```ps1
    Test-MrCmdletBinding [[-ComputerName] <Object>] [<CommonParameters>]
```

- *Drilling down into the parameters with `Get-Command` shows the actual parameter names including the common ones.
```ps1
    (Get-Command -Name Test-MrCmdletBinding).Parameters.Keys
```
_then you can see **Outputs**._

## SupportsShouldProcess
- `SupportsShouldProcess` adds **WhatIf** and **Confirm** parameters. These are only needed for commands that make changes.
```ps1
    function Test-MrSupportsShouldProcess {

    [CmdletBinding(SupportsShouldProcess)]
        param (
            $ComputerName
        )

    Write-Output $ComputerName

    }
```
_*Notice that there are now **WhatIf** and **Confirm** parameters._
```ps1
    Get-Command -Name Test-MrSupportsShouldProcess -Syntax
```
**Output:**
```ps1
    Test-MrSupportsShouldProcess [[-ComputerName] <Object>] [-WhatIf] [-Confirm] [<CommonParameters>]
```
- Once again, you can also use `Get-Command` to return a list of the actual parameter names including the common ones along with `WhatIf` and `Confirm`.
```ps1
    (Get-Command -Name Test-MrSupportsShouldProcess).Parameters.Keys
```

## Parameter Validation
- Always type the variables that are being used for your parameters (specify a datatype).
```ps1
    function Test-MrParameterValidation {

    [CmdletBinding()]
        param (
            [string]$ComputerName
        )

    Write-Output $ComputerName

    }
```
>   I've specified `String` as the datatype for the `ComputerName` parameter.
    This causes it to allow only a `single computer name` to be specified.
    If more than one computer name is specified via a `comma-separated` list, an error is generated.
```ps1
    Test-MrParameterValidation -ComputerName Server01, Server02
```
_And error **outputs**._

>   The problem with the current definition is that it's valid to omit the value of the `ComputerName` parameter,
    but a value is required for the function to complete successfully. This is where the `Mandatory` parameter attribute comes in handy.
```ps1
    function Test-MrParameterValidation {

    [CmdletBinding()]
        param (
            [Parameter(Mandatory)]
            [string]$ComputerName
        )

    Write-Output $ComputerName

    }
```
- **My note:**
  - The above example is what about "saying the `function` that parameter is required by adding `[Parameter (Mandatory)]` in `param` block of function."

>   `[Parameter(Mandatory=$true)]` could be specified instead to make the function compatible with PowerShell version 2.0 and higher.
    Now that the `ComputerName` is required, if one isn't specified, the function will prompt for one.
```ps1
    Test-MrParameterValidation
```
**Output:**
```ps1
    cmdlet Test-MrParameterValidation at command pipeline position 1
    Supply values for the following parameters:
    ComputerName:
```

>   *If you want to allow for more than one value for the `ComputerName` parameter,
    use the `String` datatype but add `open and closed` square brackets to the datatype to allow for an `array of strings`.
```ps1
    function Test-MrParameterValidation {

    [CmdletBinding()]
        param (
            [Parameter(Mandatory)]
            [string[]]$ComputerName
        )

    Write-Output $ComputerName

    }
```
>   Maybe you want to specify a default value for the `ComputerName` parameter if one isn't specified.
    The problem is that default values can't be used with `mandatory` parameters.
    Instead, you'll need to use the `ValidateNotNullOrEmpty` parameter validation attribute with a `default value`.
```ps1
    function Test-MrParameterValidation {

    [CmdletBinding()]
        param (
            [ValidateNotNullOrEmpty()]
            [string[]]$ComputerName = $env:COMPUTERNAME
        )

    Write-Output $ComputerName

    }
```
- *Even when setting a default value, try not to use static values.
  In the previous example, `$env:COMPUTERNAME` is used as the default value,
  which is automatically translated into the local computer name if a value is not provided.

## Verbose Output
- The function shown in the following example has an inline comment in the `foreach` loop.
```ps1
    function Test-MrVerboseOutput {

    [CmdletBinding()]
        param (
            [ValidateNotNullOrEmpty()]
            [string[]]$ComputerName = $env:COMPUTERNAME
        )

    foreach ($Computer in $ComputerName) {
            #Attempting to perform some action on $Computer <<-- Don't use
            #inline comments like this, use write verbose instead.
            Write-Output $Computer
        }

    }
```
- **A better option is to use `Write-Verbose` instead of `inline comments`.**
```ps1
    function Test-MrVerboseOutput {

    [CmdletBinding()]
        param (
            [ValidateNotNullOrEmpty()]
            [string[]]$ComputerName = $env:COMPUTERNAME
        )

    foreach ($Computer in $ComputerName) {
            Write-Verbose -Message "Attempting to perform some action on $Computer"
            Write-Output $Computer
        }

    }
```
_When the function is called without the **Verbose** parameter,
the verbose output won't be displayed._
```ps1
    Test-MrVerboseOutput -ComputerName Server01, Server02
```
_When it's called with the Verbose parameter, the verbose output will be displayed._
```ps1
    Test-MrVerboseOutput -ComputerName Server01, Server02 -Verbose
```

## Pipeline Input
- Commands can accept pipeline input by value (by type) or by property name.
  You can write your functions just like the native commands so that they accept either one or both of these types of input.
- To accept pipeline input by value, specified the `ValueFromPipeline` parameter attribute for that particular parameter.
  Keep in mind that you can only accept pipeline input by value from one of each datatype.

>   Pipeline input comes in one item at a time similar to the way items are handled in a `foreach` loop.
    At a minimum, a process block is required to process each of these items if you're accepting an array as input.
    If you're only accepting a single value as input, a process block isn't necessary,
    but I still recommend specifying it for consistency.
```ps1
    function Test-MrPipelineInput {

    [CmdletBinding()]
        param (
            [Parameter(Mandatory,
                      ValueFromPipeline)]
            [string[]]$ComputerName
        )

    PROCESS {
            Write-Output $ComputerName
        }

    }
```
>   Accepting pipeline input by property name is similar except it's specified with the `ValueFromPipelineByPropertyName` parameter attribute
    and it can be specified for any number of parameters regardless of datatype. The key is that the output of the command
    that's being piped in has to have a property name that matches the name of the parameter or a parameter alias of your function.
```ps1
    function Test-MrPipelineInput {

    [CmdletBinding()]
        param (
            [Parameter(Mandatory,
                      ValueFromPipelineByPropertyName)]
            [string[]]$ComputerName
        )

    PROCESS {
                Write-Output $ComputerName
        }

    }
```
_***BEGIN and END blocks are optional**._

## Error Handling
- The function shown in the following example generates an unhandled exception when a computer can't be contacted.
```ps1
    function Test-MrErrorHandling {

    [CmdletBinding()]
        param (
            [Parameter(Mandatory,
                      ValueFromPipeline,
                      ValueFromPipelineByPropertyName)]
            [string[]]$ComputerName
        )

    PROCESS {
            foreach ($Computer in $ComputerName) {
                Test-WSMan -ComputerName $Computer
            }
        }

    }
```
- There are a couple of different ways to handle errors in PowerShell. `Try/Catch` is the more modern way to handle errors.
```ps1
    function Test-MrErrorHandling {

    [CmdletBinding()]
        param (
            [Parameter(Mandatory,
                      ValueFromPipeline,
                      ValueFromPipelineByPropertyName)]
            [string[]]$ComputerName
        )

    PROCESS {
            foreach ($Computer in $ComputerName) {
                try {
                    Test-WSMan -ComputerName $Computer
                }
                catch {
                    Write-Warning -Message "Unable to connect to Computer: $Computer"
                }
            }
        }

    }
```
- The command doesn't generate a terminating error.
  This is also important to understand. Only terminating errors are caught.
  Specify the ErrorAction parameter with Stop as the value to turn a non-terminating error into a terminating one.
```ps1
    function Test-MrErrorHandling {

    [CmdletBinding()]
        param (
            [Parameter(Mandatory,
                      ValueFromPipeline,
                      ValueFromPipelineByPropertyName)]
            [string[]]$ComputerName
        )

    PROCESS {
            foreach ($Computer in $ComputerName) {
                try {
                    Test-WSMan -ComputerName $Computer -ErrorAction Stop
                }
                catch {
                    Write-Warning -Message "Unable to connect to Computer: $Computer"
                }
            }
        }

    }
```
>   *Don't modify the global `$ErrorActionPreference` variable unless absolutely necessary. If you're using something like .NET directly from within your PowerShell function,
    you can't specify the ErrorAction on the command itself. In that scenario, you might need to change the global `$ErrorActionPreference` variable,
    but if you do change it, change it back immediately after trying the command.

## Comment-Based Help
- It's considered to be a best practice to add comment based help to your functions so the people you're sharing them with will know how to use them.
```ps1
    function Get-MrAutoStoppedService {

  <#
    .SYNOPSIS
        Returns a list of services that are set to start automatically, are not
        currently running, excluding the services that are set to delayed start.

    .DESCRIPTION
        Get-MrAutoStoppedService is a function that returns a list of services from
        the specified remote computer(s) that are set to start automatically, are not
        currently running, and it excludes the services that are set to start automatically
        with a delayed startup.

    .PARAMETER ComputerName
        The remote computer(s) to check the status of the services on.

    .PARAMETER Credential
        Specifies a user account that has permission to perform this action. The default
        is the current user.

    .EXAMPLE
        Get-MrAutoStoppedService -ComputerName 'Server1', 'Server2'

    .EXAMPLE
        'Server1', 'Server2' | Get-MrAutoStoppedService

    .EXAMPLE
        Get-MrAutoStoppedService -ComputerName 'Server1' -Credential (Get-Credential)

    .INPUTS
        String

    .OUTPUTS
        PSCustomObject

    .NOTES
        Author:  Mike F Robbins
        Website: http://mikefrobbins.com
        Twitter: @mikefrobbins
  #>

    [CmdletBinding()]
        param (

    )

    #Function Body

    }
```
_When you add comment based help to your functions,
help can be retrieved for them just like the default built-in commands._

