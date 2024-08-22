# Powershell Arrays

## Create an Array
```powershell 
    $data = @()
    $data.count
```

## We can create an array and seed it with values just by placing them in the @() parentheses.
```
    PS> $data = @('Zero','One','Two','Three')
    PS> $data.count
    4
```

## We can declare an array on multiple lines. The comma is optional in this case and generally left out.
```
    $data = @(
    'Zero'
    'One'
    'Two'
    'Three'
)
```

# Other syntax
>    $data = 'Zero','One','Two','Three'

## Write-Output to create arrays
>    $data = Write-Output Zero One Two Three

-------------------------------------------------------------------

# Accessing items
## Offset
```
    PS> $data = 'Zero','One','Two','Three'
    PS> $data[0]
        Zero
```

## Special index tricks
```
    PS> $data[0,2,3]
    Zero
    Two
    Three

    PS> $data[3,0,3]
    Three
    Zero
    Three

    PS> $data[1..3]
    One
    Two
    Three

    PS> $data[3..1]
    Three
    Two
    One

    PS> $data[-1]
    Three

    PS> $a = 1,2,3,4,5,6,7,8
    PS> $a[2..-1]
    3
    2
    1
    8

    PS> $a[2,1,0,-1]
    3
    2
    1
    8
```

# Off-by-one errors

>    $data[ $data.count ]

```
    PS> $data[ $data.count - 1 ]
    Three
```

- This is where you can use the -1 index to get the last element.
```
    PS> $data[ -1 ]
    Three
```

+ Lee Dailey also pointed out to me that we can use $data.GetUpperBound(0) to get the max index number.
```
    PS> $data.GetUpperBound(0)
    3
    PS> $data[ $data.GetUpperBound(0) ]
    Three
```

# Updating items

- We can use the same index to update existing items in the array. This gives us direct access to update individual items.
```
    $data[2] = 'dos'
    $data[3] = 'tres'
```

- If we try to update an item that is past the last element, then we get an Index was outside the bounds of the array. error.
```
    PS> $data[4] = 'four'
    Index was outside the bounds of the array.
    At line:1 char:1
    + $data[4] = 'four'
    + ~~~~~~~~~~~~~
    + CategoryInfo          : OperationStopped: (:) [], IndexOutOfRangeException
    + FullyQualifiedErrorId : System.IndexOutOfRangeException
```

# Iteration

## Pipeline (|)
```
    PS> $data = 'Zero','One','Two','Three'
    PS> $data | ForEach-Object {"Item: [$PSItem]"}
    Item: [Zero]
    Item: [One]
    Item: [Two]
    Item: [Three]
```
- If you have not seen $PSItem before, just know that it's the same thing as $_. You can use either one because they both represent the current object in the pipeline.

## ForEach loop
- The ForEach loop works well with collections. Using the syntax: `foreach ( <variable> in <collection> )`
```
    foreach ( $node in $data )
    {
      "Item: [$node]"
    }
```

## ForEach method
- I tend to forget about this one but it works well for simple operations. PowerShell allows you to call .ForEach() on a collection.
```
    PS> $data.foreach({"Item [$PSItem]"})
    Item [Zero]
    Item [One]
    Item [Two]
    Item [Three]
```
  - The .foreach() takes a parameter that is a script block. You can drop the parentheses and just provide the script block.
  >    `$data.foreach{"Item [$PSItem]"}`

## For loop
```
    for ( $index = 0; $index -lt $data.count; $index++)
    {
    "Item: [{0}]" -f $data[$index]
    }
```
+ Here: The format operator (-f) is used to insert the value of `$data[$index]` in the output string.

## Switch loop
```
    $data = 'Zero','One','Two','Three'
switch( $data )
{
    'One'
    {
        'Tock'
    }
    'Three'
    {
        'Tock'
    }
    Default
    {
        'Tick'
    }
}
```
>    output:
```
    Tick
    Tock
    Tick
    Tock
```

## Updating values
- The exception to that statement is the for loop. If you want to walk an array and update values inside it,
  then the for loop is what you're looking for.
```
    for ( $index = 0; $index -lt $data.count; $index++ )
  {
    $data[$index] = "Item: [{0}]" -f $data[$index]
  }
```


























