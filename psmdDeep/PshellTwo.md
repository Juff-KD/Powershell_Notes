# All -eq
- this clever solution that checks for any incorrect values and then flips the result.
```
    $results = Test-Something
    if ( -not ( $results -ne 'Passed') )
  {
    'All results a Passed'
  }
```
# Adding to arrays
## Array addition
- We can use the addition operator with arrays to create a new array.
```
    $first = @(
    'Zero'
    'One'
  )
    $second = @(
    'Two'
    'Three'
  )
```
- **We can add them together to get a new array.**
```
    PS> $first + $second

    Zero
    One
    Two
    Three
```
## Plus equals +=
- **We can create a new array in place and add an item to it like this:**
```
    $data = @(
    'Zero'
    'One'
    'Two'
    'Three'
  )
    $data += 'four'
```
>   Just remember that every time you use `+=` that you're duplicating and creating a new array. 
    This is a not an issue for small datasets but it scales extremely poorly.
------------------------------------------------------------------------------------------------
## Pipeline assignment
```
    $array = 1..5 | ForEach-Object {
    "ATX-SQL-$PSItem"
    }
```
- We can leverage the pipeline with foreach() statements and other loops.
```
    $array = foreach ( $node in (1..5))
  {
    "ATX-SQL-$node"
  }
```

# Array Types
>   By default, an array in PowerShell is created as a [PSObject[]] type. 
    This allows it to contain any type of object or value. 
    This works because everything is inherited from the PSObject type.

## Strongly typed arrays
- _When you create a strongly typed array, it can only contain values or objects the specified type._
```
    PS> [int[]] $numbers = 1,2,3
    PS> [int[]] $numbers2 = 'one','two','three'
    ERROR: Cannot convert value "one" to type "System.Int32". Input string was not in a correct format."

    PS> [string[]] $strings = 'one','two','three'
```
## ArrayList
```
$myarray = [System.Collections.ArrayList]::new()
[void]$myArray.Add('Value')
```
## Generic List
> _A generic type is a special type in C# that defines a generalized class and
  the user specifies the data types it uses when created. 
  So if you want a list of numbers or strings, you would define that you want list of int or string types._

+ **Here is how you create a List for strings.**
```
    $mylist = [System.Collections.Generic.List[string]]::new()
```
+ **Or a list for numbers.**
```
    $mylist = [System.Collections.Generic.List[int]]::new()
```
+ **We can cast an existing array to a list like this without creating the object first:**
```
    $mylist = [System.Collections.Generic.List[int]]@(1,2,3)
```
+ **We can shorten the syntax with the using namespace statement in PowerShell 5 and newer.
  The using statement needs to be the first line of your script. By declaring a namespace,
  PowerShell lets you leave it off of the data types when you reference them.**
```
    using namespace System.Collections.Generic
    $myList = [List[int]]@(1,2,3)
```
+ **You have a similar Add method available to you. Unlike the ArrayList, 
  there is no return value on the Add method so we don't have to void it.**
```
    $myList.Add(10)
```
+ _And we can still access the elements like other arrays._
```
    PS> $myList[-1]
    10
```
-----------------------------------------------------------------------------------------------------
## `List[PSObject]`
- You can have a list of any type, but 
  when you don't know the type of objects, you can use `[List[PSObject]]` to contain them.
```
    $list = [List[PSObject]]::new()
```
## Remove()
+ The ArrayList and the generic List[] both support removing items from the collection.
```
    using namespace System.Collections.Generic
    $myList = [List[string]]@('Zero','One','Two','Three')
    [void]$myList.Remove("Two")
    Zero
    One
    Three
```
>   *When working with value types, it removes the first one from the list.
    You can call it over and over again to keep removing that value.
    If you have reference types, you have to provide the object that you want removed.
```
    [list[System.Management.Automation.PSDriveInfo]]$drives = Get-PSDrive
    $drives.remove($drives[2])
```
```
    $delete = $drives[2]
    $drives.remove($delete)
```
- The remove method returns true if it was able to find and remove the item from the collection.
======================================================================================================
# Other nuances
## Pre-sized arrays
>   I mentioned that you can't change the size of an array once it's created.
    We can create an array of a pre-determined size by calling it with the new($size) constructor.
```
    $data = [Object[]]::new(4)
    $data.count
    4
```
## Multiplying arrays
```
    PS> $data = @('red','green','blue')
    PS> $data * 3
    red
    green
    blue
    red
    green
    blue
    red
    green
    blue
```
## Initialize with 0
- A common scenario is that you want to create an array with all zeros.
  If you're only going to have integers, a strongly typed array of integers defaults to all zeros.
```
    PS> [int[]]::new(4)
    0
    0
    0
    0
```
- _We can use the multiplying trick to do this too._
```
    PS> $data = @(0) * 4
    PS> $data
    0
    0
    0
    0
```
- **The nice thing about the multiplying trick is that you can use any value.
  So if you would rather have 255 as your default value, this would be a good way to do it.**
```
    PS> $data = @(255) * 4
    PS> $data
    255
    255
    255
    255
```
## Nested arrays
- :Here are two ways we can create a two-dimensional array.
```
    $data = @(@(1,2,3),@(4,5,6),@(7,8,9))

    $data2 = @(
        @(1,2,3),
        @(4,5,6),
        @(7,8,9)
    )
```
_The comma is very important in those examples._
- The way we use the index notation changes slightly now that we've a nested array.
  Using the $data above, this is how we would access the value 3.
```
    PS> $outside = 0
    PS> $inside = 2
    PS> $data[$outside][$inside]
    3
```
## Write-Output -NoEnumerate
- Normal & Unwrapped...
```
    PS> $data = @('red','green','blue')
    PS> $data | Get-Member
    TypeName: System.String
    ...
```
- ***To prevent that unwrap of the array, you can use Write-Output -NoEnumerate.**
```
    PS> Write-Output -NoEnumerate $data | Get-Member
    TypeName: System.Object[]
    ...
```
>   You can place a comma in front of the array before you pipe it.
    This wraps $data into another array where it is the only element, 
    so after the unwrapping the outer array we get back $data unwrapped.
```
    PS> ,$data | Get-Member
    TypeName: System.Object[]
    ...
```
## Return an array
- The catch is that you have a new array. If that is ever a problem,
  you can use `Write-Output -NoEnumerate $array` or `return ,$array` to work around it.





