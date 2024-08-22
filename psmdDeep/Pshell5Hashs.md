# Reading and writing hashtables to file
## Saving to CSV
```
    $person | ForEach-Object{ [pscustomobject]$_ } | Export-CSV -Path $path
```
## Saving a nested hashtable to file
```
    $people | ConvertTo-JSON | Set-Content -Path $path
    $people = Get-Content -Path $path -Raw | ConvertFrom-JSON
```
>   There are two important points about this method.
    First is that the JSON is written out multiline
    so I need to use the `-Raw` option to read it back into a single string.
    The Second is that the imported object is no longer a `[hashtable]`.
    It's now a `[pscustomobject]` and that can cause issues if you don't expect it.

+ Watch for deeply-nested hashtables. When you convert it to JSON you might not get the results you expect.
```
    @{ a = @{ b = @{ c = @{ d = "e" }}}} | ConvertTo-Json

    {
      "a": {
        "b": {
          "c": "System.Collections.Hashtable"
        }
      }
    }
```
+ Use **Depth** parameter to ensure that you have expanded all the nested hashtables.
```
    @{ a = @{ b = @{ c = @{ d = "e" }}}} | ConvertTo-Json -Depth 3

    {
      "a": {
        "b": {
          "c": {
            "d": "e"
          }
        }
      }
    }
```
+ _If you need it to be a `[hashtable]` on import, then you need to use the `Export-CliXml` and `Import-CliXml` commands._

## Converting JSON to Hashtable
+ If you need to convert JSON to a `[hashtable]`,
  there's one way that I know of to do it with the **JavaScriptSerializer** in .NET.
```
    [Reflection.Assembly]::LoadWithPartialName("System.Web.Script.Serialization")
    $JSSerializer = [System.Web.Script.Serialization.JavaScriptSerializer]::new()
    $JSSerializer.Deserialize($json,'Hashtable')
```
+ Beginning in PowerShell v6, JSON support uses the NewtonSoft JSON.NET and adds hashtable support.
```
    '{ "a": "b" }' | ConvertFrom-Json -AsHashtable

    Name      Value
    ----      -----
    a         b
```
- *PowerShell 6.2 added the Depth parameter to ConvertFrom-Json. The default Depth is 1024.

## Reading directly from a file
- _If you have a file that contains a hashtable using PowerShell syntax, there's a way to import it directly._
```
    $content = Get-Content -Path $Path -Raw -ErrorAction Stop
    $scriptBlock = [scriptblock]::Create( $content )
    $scriptBlock.CheckRestrictedLanguage( $allowedCommands, $allowedVariables, $true )
    $hashtable = ( & $scriptBlock )
```
- It imports the contents of the file into a scriptblock, then checks to make sure
  it doesn't have any other PowerShell commands in it before it executes it.

+ **On that note, did you know that a module manifest (the psd1 file) is just a hashtable?**

## Keys can be any object
- Most of the time, the keys are just strings. So we can put quotes around anything and make it a key.
```
    $person = @{
    'full name' = 'Kevin Marquette'
    '#' = 3978
  }
    $person['full name']
```
  _You can do some odd things that you may not have realized you could do._
```
    $person.'full name'

    $key = 'full name'
    $person.$key
```
>   Just because you can do something, it doesn't mean that you should.
    That last one just looks like a bug waiting to happen
    and would be easily misunderstood by anyone reading your code.

+ Technically your key doesn't have to be a string but they're easier to think about if you only use strings.
  However, indexing doesn't work well with the complex keys.
```
    $ht = @{ @(1,2,3) = "a" }
    $ht

    Name                           Value
    ----                           -----
    {1, 2, 3}                      a
```
- Accessing a value in the hashtable by its key doesn't always work. For example:
```
    $key = $ht.keys[0]
    $ht.$($key)
    a
    $ht[$key]
    a
```
>   When the key is an array, you must wrap the `$key` variable in a subexpression
    so that it can be used with member access `(.)` notation.
    Or, you can use array index `([])` notation.
====================================================================================================================
# Use in automatic variables
## $PSBoundParameters
>   **$PSBoundParameters** is an automatic variable that only exists inside the context of a function.
    It contains all the parameters that the function was called with. This isn't exactly a hashtable
    but close enough that you can treat it like one.

## PSBoundParameters gotcha
>   One important thing to remember is that this only includes the values that are passed in as parameters.
    If you also have parameters with default values but aren't passed in by the caller,
    `$PSBoundParameters` doesn't contain those values. This is commonly overlooked.

## $PSDefaultParameterValues
- This automatic variable lets you assign default values to any cmdlet without changing the cmdlet.
  Take a look at this example.
```
    $PSDefaultParameterValues["Out-File:Encoding"] = "UTF8"
```
>   This adds an entry to the `$PSDefaultParameterValues` hashtable that
    sets `UTF8` as the default value for the Out-File -Encoding parameter.
    This is session-specific so you should place it in your $profile.

- I use this often to pre-assign values that I type quite often.
```
    $PSDefaultParameterValues[ "Connect-VIServer:Server" ] = 'VCENTER01.contoso.local'
```
- This also accepts wildcards so you can set values in bulk. Here are some ways you can use that:
```
    $PSDefaultParameterValues[ "Get-*:Verbose" ] = $true
    $PSDefaultParameterValues[ "*:Credential" ] = Get-Credential
```
## Regex $Matches
- When you use the `-match` operator, an automatic variable called `$matches` is created with the results of the match.
  If you have any sub expressions in your regex, those sub matches are also listed.
```
    $message = 'My SSN is 123-45-6789.'

    $message -match 'My SSN is (.+)\.'
    $Matches[0]
    $Matches[1]
```
## Named matches
  _If you use a named regex match, then you can access that match by name on the matches._
```
    $message = 'My Name is Kevin and my SSN is 123-45-6789.'

    if($message -match 'My Name is (?<Name>.+) and my SSN is (?<SSN>.+)\.')
    {
        $Matches.Name
        $Matches.SSN
    }
```
  _In the example above, the `(?<Name>.*)` is a named sub expression.
  This value is then placed in the $Matches.Name property._

## Group-Object -AsHashtable
- One little known feature of `Group-Object` is that it can turn some datasets into a hashtable for you.
```
    Import-CSV $Path | Group-Object -AsHashtable -Property email
```

# Copying Hashtables
## Assigning reference types
- When you have one hashtable and assign it to a second variable, both variables point to the same hashtable.
```
    PS> $orig = @{name='orig'}
    PS> $copy = $orig
    PS> $copy.name = 'copy'
    PS> 'Copy: [{0}]' -f $copy.name
    PS> 'Orig: [{0}]' -f $orig.name

    Copy: [copy]
    Orig: [copy]
```
## Shallow copies, single level
- If we have a simple hashtable like our example above,
  we can use `.Clone()` to make a shallow copy.
```
    PS> $orig = @{name='orig'}
    PS> $copy = $orig.Clone()
    PS> $copy.name = 'copy'
    PS> 'Copy: [{0}]' -f $copy.name
    PS> 'Orig: [{0}]' -f $orig.name

    Copy: [copy]
    Orig: [orig]
```
## Shallow copies, nested
```
    PS> $orig = @{
        person=@{
            name='orig'
        }
    }
    PS> $copy = $orig.Clone()
    PS> $copy.person.name = 'copy'
    PS> 'Copy: [{0}]' -f $copy.person.name
    PS> 'Orig: [{0}]' -f $orig.person.name

    Copy: [copy]
    Orig: [copy]
```
- _So you can see that even though I cloned the hashtable, the reference to person wasn't cloned.
  We need to make a deep copy to truly have a second hashtable that isn't linked to the first._

## Deep copies
-  Here's a function using PowerShell to recursively create a deep copy:
```
    function Get-DeepClone
    {
        [CmdletBinding()]
        param(
            $InputObject
        )
        process
        {
            if($InputObject -is [hashtable]) {
                $clone = @{}
                foreach($key in $InputObject.keys)
                {
                    $clone[$key] = Get-DeepClone $InputObject[$key]
                }
                return $clone
            } else {
                return $InputObject
            }
        }
    }
```
_It doesn't handle any other reference types or arrays, but it's a good starting point._

- Another way is to use .Net to deserialize it using **CliXml** like in this function:
```
    function Get-DeepClone
    {
        param(
            $InputObject
        )
        $TempCliXmlString = [System.Management.Automation.PSSerializer]::Serialize($obj, [int32]::MaxValue)
        return [System.Management.Automation.PSSerializer]::Deserialize($TempCliXmlString)
    }
```
>   For extremely large hashtables, the deserializing function is faster as it scales out.
    However, there are some things to consider when using this method.
    Since it uses CliXml, it's memory intensive and if you are cloning huge hashtables, that might be a problem.
    Another limitation of the CliXml is there is a depth limitation of 48.
      - Meaning, if you have a hashtable with 48 layers of nested hashtables, the cloning will fail and no hashtable will be output at all.
