# Arrays of Objects
```powershell
    $data = @(
    [pscustomobject]@{FirstName='Kevin';LastName='Marquette'}
    [pscustomobject]@{FirstName='John'; LastName='Doe'}
    )
```
## Accessing properties
```
    PS> $data[0]

        FirstName LastName
        -----     ----
        Kevin     Marquette
```
+ We can access and update properties directly.
```
    PS> $data[0].FirstName

        Kevin

        PS> $data[0].FirstName = 'Jay'
        PS> $data[0]

        FirstName LastName
        -----     ----
        Jay       Marquette
```
## Array properties
```
    PS> $data | ForEach-Object {$_.LastName}

        Marquette
        Doe
```
- _Or by using the Select-Object -ExpandProperty cmdlet._
```
    PS> $data | Select-Object -ExpandProperty LastName

        Marquette
        Doe
-------------------------------------------------------------
    PS> $data.LastName

    Marquette
    Doe
```
# Where-Object filtering
```
    PS> $data | Where-Object {$_.FirstName -eq 'Kevin'}

    FirstName LastName
    -----     ----
    Kevin     Marquette
```
```
    We can write that same query to get the FirstName we are looking for.

    $data | Where FirstName -eq Kevin
```
## Where()
```
    $data.Where({$_.FirstName -eq 'Kevin'})
```
## Updating objects in loops
```
    foreach($person in $data)
    {
    $person.FirstName = 'Kevin'
    }
```
- _You still can't replace the whole object this way._
```
    foreach($person in $data)
  {
    $person = [pscustomobject]@{
        FirstName='Kevin'
        LastName='Marquette'
    }
  }
```
# Operators
## -join
```
    PS> $data = @(1,2,3,4)
    PS> $data -join '-'
    1-2-3-4
    PS> $data -join ','
    1,2,3,4
```
+ One of the features that I like about the -join operator is that it handles single items.
```
      PS> 1 -join '-'
        1
```
_*I use this inside logging and verbose messages.*_
```
    PS> $data = @(1,2,3,4)
    PS> "Data is $($data -join ',')."
    Data is 1,2,3,4.
```
## -join $array
_If you ever want to join everything without a delimiter, instead of doing this:_
```
    PS> $data = @(1,2,3,4)
    PS> $data -join $null
    1234
```
+ You can use -join with the array as the parameter with no prefix.
```
    PS> $data = @(1,2,3,4)
    PS> -join $data
    1234
```
## -replace and -split
>   The other operators like -replace and -split execute on each item in the array.
    I can't say that I have ever used them this way but here is an example.
```
    PS> $data = @('ATX-SQL-01','ATX-SQL-02','ATX-SQL-03')
    PS> $data -replace 'ATX','LAX'
    LAX-SQL-01
    LAX-SQL-02
    LAX-SQL-03
```
## -contains
+ The -contains operator allows you to check an array of values to see if it contains a specified value.
```
    PS> $data = @('red','green','blue')
    PS> $data -contains 'green'
    True
```
## -in
```
    PS> $data = @('red','green','blue')
    PS> 'green' -in $data
    True
```
+ _This can get expensive if the list is large. I often use a regex pattern if I'm checking more than a few values._
```
    PS> $data = @('red','green','blue')
    PS> $pattern = "^({0})$" -f ($data -join '|')
    PS> $pattern
    ^(red|green|blue)$

    PS> 'green' -match $pattern
    True
```
## -eq and -ne
- Equality and arrays can get complicated. When the array is on the left side, every item gets compared.
  Instead of returning True, it returns the object that matches.
```
    PS> $data = @('red','green','blue')
    PS> $data -eq 'green'
    green
```
+ When you use the -ne operator, we get all the values that are not equal to our value.
```
    PS> $data = @('red','green','blue')
    PS> $data -ne 'green'
    red
    blue
```
- **When you use this in an if() statement**
```
    $data = @('red','green','blue')
    if ( $data -eq 'green' )
    {
        'Green was found'
    }
    if ( $data -ne 'green' )
    {
        'And green was not found'
    }
```
## -match
```
    PS> $servers = @(
    'LAX-SQL-01'
    'LAX-API-01'
    'ATX-SQL-01'
    'ATX-API-01'
  )
    PS> $servers -match 'SQL'
    LAX-SQL-01
    ATX-SQL-01
```
_We can take the same approach with Select-String._
```
    $servers | Select-String SQL
```
## $null or empty
- *At a glance, this statement looks like it should work.
```
    if ( $array -eq $null)
  {
    'Array is $null'
  }
```
> But I just went over how -eq checks each item in the array.
  So we can have an array of several items with a single `$null` value and
  it would evaluate to `$true`.
```
    $array = @('one',$null,'three')
  if ( $array -eq $null)
  {
    'I think Array is $null, but I would be wrong'
  }
```
> This is why it's a best practice to place the $null on the left side of the operator.
  This makes this scenario a non-issue.
```
    if ( $null -eq $array )
  {
    'Array actually is $null'
  }
```
> A $null array isn't the same thing as an empty array.
  If you know you have an array, check the count of objects in it.
  If the array is $null, the count is 0.
```
    if ( $array.count -gt 0 )
  {
    "Array isn't empty"
  }
```
> There is one more trap to watch out for here. You can use the count even if you have a single object,
  unless that object is a PSCustomObject.
```
    PS> $object = [PSCustomObject]@{Name='TestObject'}
    PS> $object.count
    $null
```
- _If you're still on PowerShell 5.1, you can wrap the object in an array before checking the count to get an accurate count._
```
    if ( @($array).count -gt 0 )
  {
    "Array isn't empty"
  }
```
- __To fully play it safe, check for `$null`, then check the count.__
```
    if ( $null -ne $array -and @($array).count -gt 0 )
  {
    "Array isn't empty"
  }
```

