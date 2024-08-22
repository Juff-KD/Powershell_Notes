# Everything you wanted to know about the if statement

## The if statement
```
    $condition = $true
    if ( $condition )
    {
        Write-Output "The condition was true"
    }
```

## Comparison operators
  ### -eq for equality
```
    $value = Get-MysteryValue
    if ( 5 -eq $value )
    {
        # do something
    }
```
  - This operator (and others) has a few variations.
    -eq case-insensitive equality
    -ieq case-insensitive equality
    -ceq case-sensitive equality

  ### -ne not equal
    - Many operators have a related operator that is checking for the opposite result.
     `-ne` verifies that the values don't equal each other.
```
      if ( 5 -ne $value )
      {
          # do something
      }
```
  - Variations:
    - -ne case-insensitive not equal
    - -ine case-insensitive not equal
    - -cne case-sensitive not equal
  _These are inverse variations of -eq. I'll group these types together when I list variations for other operators._

### -gt -ge -lt -le for greater than or less than
```
    if ( $value -gt 5 )
    {
        # do something
    }
```
  - Variations:

    - -gt greater than
    - -igt greater than, case-insensitive
    - -cgt greater than, case-sensitive
    - -ge greater than or equal
    - -ige greater than or equal, case-insensitive
    - -cge greater than or equal, case-sensitive
    - -lt less than
    - -ilt less than, case-insensitive
    - -clt less than, case-sensitive
    - -le less than or equal
    - -ile less than or equal, case-insensitive
    - -cle less than or equal, case-sensitive
- _I don't know why you would use case-sensitive and insensitive options for these operators._

### -like wildcard matches
  - PowerShell has its own wildcard-based pattern matching syntax and you can use it with the -like operator.
    These wildcard patterns are fairly basic
      - ? matches any single character
      - `*` matches any number of characters
```
    $value = 'S-ATX-SQL01'
    if ( $value -like 'S-*-SQL??')
    {
        # do something
    }
```
```
    $value = 'S-ATX-SQL02'
    if ( $value -like '*SQL*')
    {
        # do something
    }
```
  - Variations:
    - -like case-insensitive wildcard
    - -ilike case-insensitive wildcard
    - -clike case-sensitive wildcard
    - -notlike case-insensitive wildcard not matched
    - -inotlike case-insensitive wildcard not matched
    - -cnotlike case-sensitive wildcard not matched

### -match regular expression
- The -match operator allows you to check a string for a regular-expression-based match.
  Use this when the wildcard patterns aren't flexible enough for you.
```
    $value = 'S-ATX-SQL01'
    if ( $value -match 'S-\w\w\w-SQL\d\d')
    {
        # do something
    }
```
_A regex pattern matches anywhere in the string by default.
So you can specify a substring that you want matched like this:_
```
    $value = 'S-ATX-SQL01'
    if ( $value -match 'SQL')
    {
        # do something
    }
```
- Variations:
  - -match case-insensitive regex
  - -imatch case-insensitive regex
  - -cmatch case-sensitive regex
  - -notmatch case-insensitive regex not matched
  - -inotmatch case-insensitive regex not matched
  - -cnotmatch case-sensitive regex not matched

### -is of type
- You can check a value's type with the `-is` operator.
```
    if ( $value -is [string] )
    {
        # do something
    }
```
>   You may use this if you're working with classes or accepting various objects over the pipeline.
    You could have either a service or a service name as your input.
    Then check to see if you have a service and fetch the service if you only have the name.
```
    if ( $Service -isnot [System.ServiceProcess.ServiceController] )
    {
        $Service = Get-Service -Name $Service
    }
```
- Variations:
  - -is of type
  - -isnot not of type

### Collection operators
```
    PS> 1,2,3,4 -eq 3
    3
```
_This still works correctly in an if statement.
So a value is returned by your operator, then the whole statement is `$true`._
```
    $array = 1..6
    if ( $array -gt 3 )
    {
        # do something
    }
```
>   When using the `-ne` operator this way, it's easy to mistakenly look at the logic backwards.
    Using `-ne` with a collection returns $true if any item in the collection doesn't match your value.
```
    PS> 1,2,3 -ne 4
    1
    2
    3
```
- *_This may look like a clever trick, but we have operators `-contains` and `-in` that handle this more efficiently.
  And `-notcontains` does what you expect._

### -contains
- The `-contains` operator checks the collection for your value. As soon as it finds a match, it returns `$true`.
```
    $array = 1..6
    if ( $array -contains 3 )
    {
        # do something
    }
```
_This is the preferred way to see if a collection contains your value.
Using `Where-Object (or -eq)` walks the entire list every time and is significantly slower._
- Variations:
  - -contains case-insensitive match
  - -icontains case-insensitive match
  - -ccontains case-sensitive match
  - -notcontains case-insensitive not matched
  - -inotcontains case-insensitive not matched
  - -cnotcontains case-sensitive not matched

### -in
- The `-in` operator is just like the `-contains` operator except the collection is on the right-hand side.
```
    $array = 1..6
    if ( 3 -in $array )
    {
        # do something
    }
```
- Variations:
  - -in case-insensitive match
  - -iin case-insensitive match
  - -cin case-sensitive match
  - -notin case-insensitive not matched
  - -inotin case-insensitive not matched
  - -cnotin case-sensitive not matched

## Logical operators

### -not
```
    if ( -not ( Test-Path -Path $path ) )
```

### ! operator
- You can use `!` as an alias for `-not`.
```
    if ( -not $value ){}
    if ( !$value ){}
```
### -and
- You can combine expressions with the `-and` operator.
  When you do that, both sides need to be `$true` for the whole expression to be `$true`.
```
    if ( ($age -gt 13) -and ($age -lt 55) )
```
```
    if ( $age -gt 13 -and $age -lt 55 )
```
_Evaluation happens from left to right.
If the first item evaluates to `$false`, it exits early and doesn't perform the right comparison._
_For example, `Test-Path` throws an error if you give it a $null path._
```
    if ( $null -ne $path -and (Test-Path -Path $path) )
```
### -or
- The `-or` allows for you to specify two expressions and returns `$true` if either one of them is `$true`.
```
     if ( $age -le 13 -or $age -ge 55 )
```
### -xor exclusive or
>   This one is a little unusual.
    `-xor` allows only one expression to evaluate to `$true`.
    So if both items are `$false` or both items are `$true`, then the whole expression is `$false`.
    Another way to look at this is the expression is only `$true` when the results of the expression are different.
_*It's rare that anyone would ever use this logical operator and I can't think up a good example as to why I would ever use it._

## Bitwise operators
>   Bitwise operators perform calculations on the bits within the values and produce a new value as the result.
    Teaching bitwise operators is beyond the scope of this article, but here is the list of them.
  - -band binary AND
  - -bor binary OR
  - -bxor binary exclusive OR
  - -bnot binary NOT
  - -shl shift left
  - -shr shift right

## PowerShell expressions
_We can use normal PowerShell inside the condition statement._
```
    if ( Test-Path -Path $Path )
```
- `Test-Path` returns `$true` or `$false` when it executes. This also applies to commands that return other values.
```
    if ( Get-Process Notepad* )
```
_It evaluates to `$true` if there's a returned process and `$false` if there isn't.
It's perfectly valid to use pipeline expressions or other PowerShell statements like this:_
```
    if ( Get-Process | Where Name -eq Notepad )
```
_These expressions can be combined with each other with the `-and` and `-or` operators,
but you may have to use parenthesis to break them into subexpressions._
```
    if ( (Get-Process) -and (Get-Service) )
```
## Checking for $null
>   Having a no result or a `$null` value evaluates to `$false` in the if statement.
    When checking specifically for `$null`, it's a best practice to place the `$null` on the left-hand side.
```
    if ( $null -eq $value )
```
## Variable assignment within the condition
```
    if ($process=Get-Process notepad -ErrorAction ignore) {$process} else {$false}
```
_Normally when you assign a value to a variable, the value isn't passed onto the pipeline or console.
When you do a variable assignment in a sub expression, it does get passed on to the pipeline._
```
    PS> $first = 1
    PS> ($second = 2)
    2
```
- When an assignment is done in an if statement, it executes just like the $second assignment above.
  Here is a clean example on how you could use it:
```
    if ( $process = Get-Process Notepad* )
    {
        $process | Stop-Process
    }
```
_*Make sure you don't confuse this with `-eq` because this isn't an equality check.
This is a more obscure feature that most people don't realize works this way._

## Variable assignment from the scriptblock
- You can also use the if statement scriptblock to assign a value to a variable.
```
    $discount = if ( $age -ge 55 )
    {
        Get-SeniorDiscount
    }
    elseif ( $age -le 13 )
    {
        Get-ChildDiscount
    }
    else
    {
        0.00
    }
```
_Each script block is writing the results of the commands, or the value, as output._
_That example could have just as easily assigned those values to the `$discount` variable directly in each scriptblock._

## Alternate execution path
>   The if statement allows you to specify an action for not only when the statement is `$true`,
    but also for when it's `$false`. This is where the else statement comes into play.

## else
```
    if ( Test-Path -Path $Path -PathType Leaf )
    {
        Move-Item -Path $Path -Destination $archivePath
    }
    else
    {
        Write-Warning "$path doesn't exist or isn't a file."
    }
```
_In this example, we check the $path to make sure it's a file.
If we find the file, we move it. If not, we write a warning. This type of branching logic is very common._

## Nested if
```
    if ( Test-Path -Path $Path -PathType Leaf )
    {
        Move-Item -Path $Path -Destination $archivePath
    }
    else
    {
        if ( Test-Path -Path $Path )
        {
            Write-Warning "A file was required but a directory was found instead."
        }
        else
        {
            Write-Warning "$path could not be found."
        }
    }
```
## elseif
```
    if ( Test-Path -Path $Path -PathType Leaf )
    {
        Move-Item -Path $Path -Destination $archivePath
    }
    elseif ( Test-Path -Path $Path )
    {
        Write-Warning "A file was required but a directory was found instead."
    }
    else
    {
        Write-Warning "$path could not be found."
    }
```
## switch
```
    $itemType = 'Role'
    switch ( $itemType )
    {
        'Component'
        {
            'is a component'
        }
        'Role'
        {
            'is a role'
        }
        'Location'
        {
            'is a location'
        }
    }
```
_There three possible values that can match the `$itemType`.
In this case, it matches with Role._

## Array inline
>   I have a function called `Invoke-SnowSql` that launches an executable with several command-line arguments.
    Here is a clip from that function where I build the array of arguments.
```
    $snowSqlParam = @(
    '--accountname', $Endpoint
    '--username', $Credential.UserName
    '--option', 'exit_on_error=true'
    '--option', 'output_format=csv'
    '--option', 'friendly=false'
    '--option', 'timing=false'
    if ($Debug)
    {
        '--option', 'log_level=DEBUG'
    }
    if ($Path)
    {
        '--filename', $Path
    }
    else
    {
        '--query', $singleLineQuery
    }
  )
```
## Simplify complex operations
_It's inevitable that you run into a situation that has way too many comparisons to check 
and your `If` statement scrolls way off the right side of the screen._
```
    $user = Get-ADUser -Identity $UserName
    if ( $null -ne $user -and $user.Department -eq 'Finance' -and $user.Title -match 'Senior' -and $user.HomeDrive -notlike '\\server\*' )
    {
        # Do Something
    }
```

## Line continuation
>   There some operators in PowerShell that let you wrap you command to the next line.
    The logical operators `-and` and `-or` are good operators to use if you want to break your expression into multiple lines.
```
    if ($null -ne $user -and
    $user.Department -eq 'Finance' -and
    $user.Title -match 'Senior' -and
    $user.HomeDrive -notlike '\\server\*'
    )
    {
        # Do Something
    }
```
## Pre-calculating results
- We can take that statement out of the if statement and only check the result.
```
    $needsSecureHomeDrive = $null -ne $user -and
    $user.Department -eq 'Finance' -and
    $user.Title -match 'Senior' -and
    $user.HomeDrive -notlike '\\server\*'

    if ( $needsSecureHomeDrive )
    {
        # Do Something
    }
```
_This is also and example of self-documenting code that saves unnecessary comments._

## Multiple if statements
_*In this case, we use a flag or a tracking variable to combine the results._
```
    $skipUser = $false

    if( $null -eq $user )
    {
        $skipUser = $true
    }

    if( $user.Department -ne 'Finance' )
    {
        Write-Verbose "isn't in Finance department"
        $skipUser = $true
    }

    if( $user.Title -match 'Senior' )
    {
        Write-Verbose "Doesn't have Senior title"
        $skipUser = $true
    }

    if( $user.HomeDrive -like '\\server\*' )
    {
        Write-Verbose "Home drive already configured"
        $skipUser = $true
    }

    if ( -not $skipUser )
    {
        # do something
    }
```
## Using functions
```
    if ( Test-SecureDriveConfiguration -ADUser $user )
    {
        # do something
    }
```
## Error handling
_One important use of the if statement is to check for error conditions before you run into errors.
A good example is to check if a folder already exists before you try to create it._
```
    if ( -not (Test-Path -Path $folder) )
    {
        New-Item -Type Directory -Path $folder
    }
```

