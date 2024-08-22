# Everything you wanted to know about PSCustomObject
- The idea behind using a PSCustomObject is to have a simple way **to create structured data**.

## Creating a PSCustomObject
```
    $myObject = [PSCustomObject]@{
    Name     = 'Kevin'
    Language = 'PowerShell'
    State    = 'Texas'
    }
```
- _You can then access and use the values like you would a normal object._
```
    $myObject.Name
```
## Converting a hashtable
```
    $myHashtable = @{
    Name     = 'Kevin'
    Language = 'PowerShell'
    State    = 'Texas'
    }
    $myObject = [pscustomobject]$myHashtable
```
## Legacy approach
- You may have seen people use New-Object to create custom objects.
```
    $myHashtable = @{
    Name     = 'Kevin'
    Language = 'PowerShell'
    State    = 'Texas'
    }

    $myObject = New-Object -TypeName PSObject -Property $myHashtable
```
_This way is quite a bit slower but it may be your best option on early versions of PowerShell._

## Saving to a file
- I find the best way to save a hashtable to a file is to save it as JSON.
  You can import it back into a `[PSCustomObject]`
```
    $myObject | ConvertTo-Json -depth 1 | Set-Content -Path $Path
    $myObject = Get-Content -Path $Path | ConvertFrom-Json
```
# Working with properties
## Adding properties
- You can still add new properties to your `PSCustomObject` with `Add-Member`.
```
    $myObject | Add-Member -MemberType NoteProperty -Name 'ID' -Value 'KevinMarquette'

    $myObject.ID
```

## Remove properties
```
    $myObject.psobject.properties.remove('ID')
```
>   The `.psobject` is an intrinsic member that gives you access to base object metadata.
    For more information about intrinsic members, see **about_Intrinsic_Members**.

## Enumerating property names
- Sometimes you need a list of all the property names on an object.
```
    $myObject | Get-Member -MemberType NoteProperty | Select -ExpandProperty Name
```
_We can get this same list off of the psobject property too._
```
    $myobject.psobject.properties.name
```
>   Note
    `Get-Member` returns the properties in alphabetical order.
    Using the member-access operator to enumerate the property names returns the properties in the order they were defined on the object.

## Dynamically accessing properties
- I already mentioned that you can access property values directly.
```
    $myObject.Name
```

- You can use a string for the property name and it will still work.
```
    $myObject.'Name'
```
- We can take this one more step and use a variable for the property name.
```
    $property = 'Name'
    $myObject.$property
```
_I know that looks strange, but it works._

## Convert PSCustomObject into a hashtable
- To continue on from the last section, you can dynamically walk the properties
  and create a hashtable from them.
```
    $hashtable = @{}
    foreach( $property in $myobject.psobject.properties.name )
    {
        $hashtable[$property] = $myObject.$property
    }
```
## Testing for properties
_If you need to know if a property exists,
you could just check for that property to have a value._
```
    if( $null -ne $myObject.ID )
```
- But if the value could be `$null` you can check to see if it exists by checking the `psobject.properties` for it.
```
    if( $myobject.psobject.properties.match('ID').Count )
```
## Adding object methods
>   If you need to add a script method to an object, you can do it with `Add-Member` and a `ScriptBlock`.
    You have to use the `this` automatic variable reference the current object.
    Here is a `scriptblock` to turn an object into a hashtable.
```
    $ScriptBlock = {
    $hashtable = @{}
    foreach( $property in $this.psobject.properties.name )
    {
        $hashtable[$property] = $this.$property
    }
    return $hashtable
  }
```
_Then we add it to our object as a script property._
```
    $memberParam = @{
    MemberType = "ScriptMethod"
    InputObject = $myobject
    Name = "ToHashtable"
    Value = $scriptBlock
  }
    Add-Member @memberParam
```
- Then we can call our function like this:
```
    $myObject.ToHashtable()
```

## Objects vs Value types
- Objects and value types don't handle variable assignments the same way.
  If you assign value types to each other, only the value get copied to the new variable.
```
    $first = 1
    $second = $first
    $second = 2
```
- Object variables hold a reference to the actual object.
  When you assign one object to a new variable, they still reference the same object.
```
    $third = [PSCustomObject]@{Key=3}
    $fourth = $third
    $fourth.Key = 4
```
_Because `$third` and `$fourth` reference the same instance of an object, both `$third.key` and `$fourth.Key` are 4._

## psobject.copy()
- If you need a true copy of an object, you can clone it.
```
    $third = [PSCustomObject]@{Key=3}
    $fourth = $third.psobject.copy()
    $fourth.Key = 4
```
- _I call this a shallow copy because if you have nested objects (objects with properties contain other objects),
  only the top-level values are copied.
  The child objects will reference each other._

## PSTypeName for custom object types
>   Now that we have an object, there are a few more things we can do with it that may not be nearly as obvious.
    First thing we need to do is give it a `PSTypeName`. This is the most common way I see people do it:
```
    $myObject.PSObject.TypeNames.Insert(0,"My.Object")
```
- I recently discovered another way to do this from Redditor `u/markekraus`.
  He talks about this approach that allows you to define it inline.
```
    $myObject = [PSCustomObject]@{
    PSTypeName = 'My.Object'
    Name       = 'Kevin'
    Language   = 'PowerShell'
    State      = 'Texas'
  }
```

## Using DefaultPropertySet (the long way)
>   PowerShell decides for us what properties to display by default.
    A lot of the native commands have a `.ps1xml` formatting file that does all the heavy lifting.
    From this post by Boe Prox, there's another way for us to do this on our custom object using just PowerShell.
    We can give it a `MemberSet` for it to use.
```
    $defaultDisplaySet = 'Name','Language'
    $defaultDisplayPropertySet = New-Object System.Management.Automation.PSPropertySet('DefaultDisplayPropertySet',[string[]]$defaultDisplaySet)
    $PSStandardMembers = [System.Management.Automation.PSMemberInfo[]]@($defaultDisplayPropertySet)
    $MyObject | Add-Member MemberSet PSStandardMembers $PSStandardMembers
```
_Now when my object just falls to the shell, it will only show those properties by default._

## Update-TypeData with DefaultPropertySet
- This is nice but I recently saw a better way using `Update-TypeData` to specify the default properties.
```
    $TypeData = @{
    TypeName = 'My.Object'
    DefaultDisplayPropertySet = 'Name','Language'
  }
    Update-TypeData @TypeData
```
- Now I can easily create objects with lots of properties and still give it a nice clean view
  when looking at it from the shell. If I need to access or see those other properties, they're still there.
```
    $myObject | Format-List *
```
## Update-TypeData with ScriptProperty
- This would be a good time to point out that this works for existing objects too.
```
    $TypeData = @{
    TypeName = 'My.Object'
    MemberType = 'ScriptProperty'
    MemberName = 'UpperCaseName'
    Value = {$this.Name.toUpper()}
  }
    Update-TypeData @TypeData
```
>   _You can do this before your object is created or after and it will still work.
    This is what makes this different than using `Add-Member` with a script property.
    When you use `Add-Member` the way I referenced earlier, it only exists on that specific instance of the object.
    This one applies to all objects with this `TypeName`._

# Function parameters
- You can now use these custom types for parameters in your functions and scripts.
  You can have one function create these custom objects and then pass them into other functions.
```
    param( [PSTypeName('My.Object')]$Data )
```
>   _PowerShell requires that the object is the type you specified.
    It throws a validation error if the type doesn't match automatically to save you the step of testing for it in your code.
    A great example of letting PowerShell do what it does best._

## Function OutputType
- You can also define an `OutputType` for your advanced functions.
```
    function Get-MyObject
  {
    [OutputType('My.Object')]
    [CmdletBinding()]
        param
        (
            ...
```
_The OutputType attribute value is only a documentation note.
It isn't derived from the function code or compared to the actual function output._

- With that said, if you're using Pester to unit test your functions then
  it would be a good idea to validate the output objects match your `OutputType`.
  This could catch variables that just fall to the pipe when they shouldn't.

