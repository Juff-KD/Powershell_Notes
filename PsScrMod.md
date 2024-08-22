# Powershell101 \ Chapter 10 - Script modules
- After scripting `.ps1` files and to run that file 
  The function needs to be loaded into the Global scope.
  That can be accomplished by dot-sourcing the script that contains the function.
  The relative path can be used.
```ps1
    . .\Get-MrPSVersion.ps1
```
_The fully qualified path can also be used._
```ps1
    . C:\Demo\Get-MrPSVersion.ps1
```
- If a portion of the path is stored in a variable, it can be combined with the remainder of the path.
```ps1
    $Path = 'C:\'
    . $Path\Get-MrPSVersion.ps1
```
- Now when I check the Function PSDrive, the `Get-MrPSVersion` function exists.
```ps1
    Get-ChildItem -Path Function:\Get-MrPSVersion
```

## Script Modules
- A script module in PowerShell is simply a file containing one or more functions
  that's saved as a `.PSM1` file instead of a `.PS1` file.

>   While there is a command in PowerShell named `New-Module`, that command creates a dynamic module, not a script module.

>   I mentioned that functions should use approved verbs otherwise they'll generate a warning message when the module is imported.
    `New-Module` demonstrates the unapproved verb warning.

- Save the following two functions in a file named MyScriptModule.psm1.
```ps1
    function Get-MrPSVersion {
        $PSVersionTable
    }

    function Get-MrComputerName {
        $env:COMPUTERNAME
    }
```
- And you try to call one of the functions.
>   An error message is generated saying the function can't be found.
    You could also check the Function PSDrive just like before
    and you'll find that it doesn't exist there either.

- *You could manually import the file with the `Import-Module` cmdlet.
```ps1
    Import-Module C:\MyScriptModule.psm1
```
> *The module autoloading feature was introduced in PowerShell version 3.
  To take advantage of module autoloading, 
  a script module needs to be saved in a folder with the same base name as the `.PSM1` file
  and in a location specified in `$env:PSModulePath`.

**To Check PSModules**
```ps1
    $env:PSModulePath
```
_ you can split the results to return each path on a separate line._
```ps1
    $env:PSModulePath -split ';'
```
>   The first three paths in the list are the default.
    When SQL Server Management Studio was installed, it dded the last path.
    For module autoloading to work, the `MyScriptModule.psm1` file needs to be located in a folder named `MyScriptModule` directly inside one of those paths.

- Once the `.PSM1` file is located in the correct path, the module will load automatically when one of its commands is called.

## Module Manifests
>   A module manifest contains metadata about your module.
    The file extension for a module manifest file is .PSD1.
    Not all files with a .PSD1 extension are module manifests.
    They can also be used for things such as storing the environmental portion of a DSC configuration.
    New-ModuleManifest is used to create a module manifest. Path is the only value that's required.
    Howeer, the module won't work if RootModule isn't specified.
    It's a good idea to specify Author and Description in case you decide to upload your module to a NuGet repository with PowerShellGet
    since those values are required in that scenario.

- The version of a module without a manifest is 0.0. This is a dead giveaway that the module doesn't have a manifest.
```ps1
    Get-Module -Name MyScriptModule
```

- The module manifest can be created with all of the recommended information.
```ps1
    New-ModuleManifest -Path $env:ProgramFiles\WindowsPowerShell\Modules\MyScriptModule\MyScriptModule.psd1 -RootModule MyScriptModule -Author 'Mike F Robbins' -Description 'MyScriptModule' -CompanyName 'mikefrobbins.com'
```
>*  If any of this information is missed during the initial creation of the module manifest,
    it can be added or updated later using Update-ModuleManifest.
    Don't recreate the manifest using New-ModuleManifest once it's already created because the GUID will change.

## Defining Public and Private Functions
- You may have helper functions that you may want to be private and only accessible by other functions within the module.
  They are not intended to be accessible to users of your module.

- If you're not following the best practices and only have a `.PSM1` file,
  then your only option is to use the `Export-ModuleMember` cmdlet.
```ps1
    function Get-MrPSVersion {
        $PSVersionTable
    }

    function Get-MrComputerName {
        $env:COMPUTERNAME
    }

    Export-ModuleMember -Function Get-MrPSVersion
```
>   In the previous example, only the Get-MrPSVersion function is available to the users of your module,
    but the Get-MrComputerName function is available to other functions within the module itself.
```ps1
    Get-Command -Module MyScriptModule
```
_If you've added a module manifest to your module (and you should),
then I recommend specifying the individual functions
you want to export in the FunctionsToExport section of the module manifest._
```ps1
    FunctionsToExport = 'Get-MrPSVersion'
```
- *It's not necessary to use both Export-ModuleMember in the .PSM1 file
  and the FunctionsToExport section of the module manifest. One or the other is sufficient.





