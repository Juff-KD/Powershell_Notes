# Everything you wanted to know about $null

## What is NULL?
>   You can think of NULL as an unknown or empty value.
    A variable is NULL until you assign a value or an object to it.
    This can be important because there are some commands that require a value
    and generate errors if the value is NULL.

## PowerShell $null
>   `$null` is an automatic variable in PowerShell used to represent `NULL`.
    You can assign it to variables, use it in comparisons and use it as a place holder for NULL in a collection.
    _*PowerShell treats `$null` as an object with a value of `NULL`.
    This is different than what you may expect if you come from another language._

## Examples of $null
- Anytime you try to use a variable that you have not initialized, the value is `$null`.
- This is one of the most common ways that `$null` values sneak into your code.
```ps1
    PS> $null -eq $undefinedVariable
    True
```
_*If you happen to mistype a variable name then PowerShell sees it as a different variable and the value is `$null`._

- *The other way you find `$null` values is when they come from other commands that don't give you any results.
```ps1
    PS> function Get-Nothing {}
    PS> $value = Get-Nothing
    PS> $null -eq $value
    True
```

## Impact of $null

**-In strings**
_If you use $null in a string, then it's a blank value (or empty string)._
```ps1
    PS> $value = $null
    PS> Write-Output "The value is $value"
    The value is
```
```ps1
    PS> $value = $null
    PS> Write-Output "The value is [$value]"
    The value is []
```
>   This is one of the reasons that I like to place brackets around variables
    when using them in log messages.
    This makes empty strings and $null values easy to spot.


**-In numeric equation**
>   When a `$null` value is used in a numeric equation then your results are invalid
    if they don't give an error.
    Sometimes the `$null` evaluates to `0` and other times it makes the whole result `$null`.
```ps1
PS> $null * 5
PS> $null -eq ( $null * 5 )
True

PS> 5 * $null
0
PS> $null -eq ( 5 * $null )
False
```

**-In place of a collection**
>   A collection allows you use an index to access values.
    If you try to index into a collection that is actually `null`,
    you get this error: `Cannot index into a null array`.
**:Example**
```ps1
    PS> $value = $null
    PS> $value[10]
    Cannot index into a null array.
    At line:1 char:1
    + $value[10]
    + ~~~~~~~~~~
        + CategoryInfo          : InvalidOperation: (:) [], RuntimeException
        + FullyQualifiedErrorId : NullArray
```
_If you have a collection but try to access an element that is not in the collection,
you get a `$null` result._
```ps1
    $array = @( 'one','two','three' )
    $null -eq $array[100]
    True
```

**-In place of an object**
>   If you try to access a property or sub property of an object that doesn't have the specified property,
    you get a $null value like you would for an undefined variable.
    It doesn't matter if the variable is `$null` or an actual object in this case.
```ps1
    PS> $null -eq $undefined.some.fake.property
    True

    PS> $date = Get-Date
    PS> $null -eq $date.some.fake.property
    True
```

**-Method on a null-valued expression**
>    Calling a method on a `$null` object throws a RuntimeException.
```ps1
    PS> $value = $null
    PS> $value.toString()
    You cannot call a method on a null-valued expression.
    At line:1 char:1
    + $value.tostring()
    + ~~~~~~~~~~~~~~~~~
        + CategoryInfo          : InvalidOperation: (:) [], RuntimeException
        + FullyQualifiedErrorId : InvokeMethodOnNull
```
>   Whenever I see the phrase `You cannot call a method on a null-valued expression`
    then the first thing I look for are places where I am calling a method on a variable
    without first checking it for $null.

# Checking for $null
>    **You may have noticed that I always place the `$null` on the left
     when checking for `$null` in my examples. This is intentional and accepte
     as a PowerShell best practice. There are some scenarios
     where placing it on the right doesn't give you the expected result.**
_**:Example**_
```ps1
    if ( $value -eq $null )
    {
        'The array is $null'
    }
    if ( $value -ne $null )
    {
        'The array is not $null'
    }
```
- If I do not define `$value`, the first one evaluates to `$true` and
  our message is The array is $null. The trap here is that
  it's possible to create a `$value` that allows both of them to be `$false`
```ps1
    $value = @( $null )
```
>    In this case, the `$value` is an array that contains a `$null`.
     The `-eq` checks every value in the array and returns the `$null` that is matched.
     This evaluates to `$false`. The `-ne` returns everything that doesn't match `$null` 
     and in this case there are no results (This also evaluates to `$false`).
     Neither one is `$true` even though it looks like one of them should be.

- Not only can we create a value that makes both of them evaluate to `$false`,
  it's possible to create a value where they both evaluate to `$true`.
  Mathias Jessen (@IISResetMe) has a good post that dives into that scenario.

# PSScriptAnalyzer and VSCode
- The `PSScriptAnalyzer` module has a rule that checks for this issue called `PSPossibleIncorrectComparisonWithNull`.
```ps1
    PS> Invoke-ScriptAnalyzer ./myscript.ps1

    RuleName                              Message
    --------                              -------
    PSPossibleIncorrectComparisonWithNull $null should be on the left side of equality comparisons.
```

## Simple if check
- A common way that people check for a `non-$null` value is to use a simple `if()` statement without the comparison.
```ps1
    if ( $value )
    {
        Do-Something
    }
```
```##
    If the value is $null, this evaluates to $false. This is easy to read, but be careful that it's looking for exactly
    what you're expecting it to look for. I read that line of code as:

>    If $value has a value.

But that's not the whole story. That line is actually saying:

>    If $value is not $null or 0 or $false or an empty string or an empty array.
```

- Here is a more complete sample of that statement.
```ps1
    if ( $null -ne $value -and
            $value -ne 0 -and
            $value -ne '' -and
            ($value -isnot [array] -or $value.Length -ne 0) -and
            $value -ne $false )
    {
        Do-Something
    }
```
>:: It's perfectly OK to use a basic if check as long as you remember those other values count as `$false`
    and not just that a variable has a value.

- I ran into this issue when refactoring some code a few days ago. It had a basic property check like this.
```ps1
    if ( $object.property )
    {
        $object.property = $value
    }
```
- I added a proper $null check and everything worked.
```ps1
    if ( $null -ne $object.property )
    {
        $object.property = $value
    }
```
_It's little bugs like these that are hard to spot and make me aggressively check values for `$null`._

## $null.Count
- If you try to access a property on a `$null` value, that the property is also `$null`.
  The count property is the exception to this rule.
```ps1
    PS> $value = $null
    PS> $value.count
    0
```
_When you have a `$null` value, then the count is 0. This special property is added by PowerShell._

## [PSCustomObject] Count
- Almost all objects in PowerShell have that count property. One important exception is the `[PSCustomObject]`
  in Windows PowerShell 5.1 (This is fixed in PowerShell 6.0). It doesn't have a count property
  so you get a `$null` value if you try to use it. I call this out here so that you don't try to use `.Count` instead of a `$null` check.

- Running this example on Windows PowerShell 5.1 and PowerShell 6.0 gives you different results.
```ps1
    $value = [PSCustomObject]@{Name='MyObject'}
    if ( $value.count -eq 1 )
    {
        "We have a value"
    }
```

## Enumerable null
>    There is one special type of `$null` that acts differently than the others.
     I am going to call it the enumerable null but it's really a `System.Management.Automation.Internal.AutomationNull`.
     This enumerable null is the one you get as the result of a function or script block that returns nothing (a void result).
```ps1
    PS> function Get-Nothing {}
    PS> $nothing = Get-Nothing
    PS> $null -eq $nothing
    True
```
- If you compare it with $null, you get a `$null` value. When used in an evaluation where a value is required,
  the value is always `$null`. But if you place it inside an array, it's treated the same as an empty array.
```ps1
    PS> $containempty = @( @() )
    PS> $containnothing = @($nothing)
    PS> $containnull = @($null)

    PS> $containempty.count
    0
    PS> $containnothing.count
    0
    PS> $containnull.count
    1
```
~~~##
You can have an array that contains one `$null` value and its count is 1.
But if you place an empty array inside an array then it's not counted as an item. The count is 0.

If you treat the enumerable null like a collection, then it's empty.

If you pass in an enumerable null to a function parameter that isn't strongly typed,
PowerShell coerces the enumerable null into a `$null` value by default.
This means inside the function, the value is treated as $null instead of the `System.Management.Automation.Internal.AutomationNull` type.
~~~

## Pipeline
- The primary place you see the difference is when using the pipeline. You can pipe a `$null` value but not an enumerable null value.
```ps1
    PS> $null | ForEach-Object{ Write-Output 'NULL Value' }
    'NULL Value'
    PS> $nothing | ForEach-Object{ Write-Output 'No Value' }
```
- Depending on your code, you should account for the $null in your logic.

- Either check for `$null` first
  - Filter out null on the pipeline (... | Where {$null -ne $_} | ...)
  - Handle it in the pipeline function

**-foreach**
- One of my favorite features of foreach is that it doesn't enumerate over a `$null` collection.
```ps1
    foreach ( $node in $null )
    {
        #skipped
    }
```
_This saves me from having to $null check the collection before I enumerate it.
If you have a collection of `$null` values, the `$node` can still be `$null`._

## Value types
>    Technically, only reference types can be `$null`. But PowerShell is very generous
     and allows for variables to be any type. If you decide to strongly type a value type, it cannot be `$null`.
     PowerShell converts `$null` to a default value for many types.
```ps1
    PS> [int]$number = $null
    PS> $number
    0

    PS> [bool]$boolean = $null
    PS> $boolean
    False

    PS> [string]$string = $null
    PS> $string -eq ''
    True
```
_There are some types that do not have a valid conversion from `$null`.
These types generate a Cannot convert null to type error._
**:Example**
```ps1
    PS> [datetime]$date = $null
    Cannot convert null to type "System.DateTime".
    At line:1 char:1
    + [datetime]$date = $null
    + ~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : MetadataError: (:) [], ArgumentTransformationMetadataException
        + FullyQualifiedErrorId : RuntimeException
```

## Function parameters
>    Using a strongly typed values in function parameters is very common.
     We generally learn to define the types of our parameters
     even if we tend not to define the types of other variables in our scripts.
     You may already have some strongly typed variables in your functions and not even realize it.
```ps1
    function Do-Something
    {
        param(
            [String] $Value
        )
    }
```
- As soon as you set the type of the parameter as a string, the value can never be `$null`.
  It's common to check if a value is $null to see if the user provided a value or not.
```ps1
    if ( $null -ne $Value ){...}
```
- `$Value` is an empty string `''` when no value is provided.
  Use the automatic variable `$PSBoundParameters.Value` instead.
```ps1
    if ( $null -ne $PSBoundParameters.Value ){...}
```
>   `$PSBoundParameters` only contains the parameters that were specified when the function was called.
    You can also use the ContainsKey method to check for the property.
```ps1
    if ( $PSBoundParameters.ContainsKey('Value') ){...}
```

## IsNotNullOrEmpty
- If the value is a string, you can use a static string function to check
  if the value is `$null` or an empty string at the same time.
```ps1
    if ( -not [string]::IsNullOrEmpty( $value ) ){...}
```

## When I $null check
- I am a defensive scripter. Anytime I call a function and assign it to a variable, I check it for `$null`.
```ps1
    $userList = Get-ADUser kevmar
    if ($null -ne $userList){...}
```
~~~##
I much prefer using `if` or `foreach` over using `try/catch`.
Don't get me wrong, I still use `try/catch` a lot.
But if I can test for an error condition or an empty set of results,
I can allow my exception handling be for true exceptions.
~~~

## No results scenario
**-Initializing to $null**
_One habit that I have picked up is initializing all my variables before I use them._
```ps1
    function Do-Something
    {
        foreach ( $node in 1..6 )
        {
            try
            {
                $result = Get-Something -ID $node
            }
            catch
            {
                Write-Verbose "[$result] not valid"
            }

    if ( $null -ne $result )
            {
                Update-Something $result
            }
        }
    }
```
~~~**
The expectation here is that `Get-Something` returns either a result or an enumerable null. If there's an error, we log it.
Then we check to make sure we got a valid result before processing it.

The bug hiding in this code is when Get-Something throws an exception and doesn't assign a value to `$result`.
It fails before the assignment so we don't even assign $null to the $result variable.
`$result` still contains the previous valid `$result` from other iterations.
`Update-Something` to execute multiple times on the same object in this example.

I set `$result` to `$null` right inside the foreach loop before I use it to mitigate this issue.
~~~
```ps1
    foreach ( $node in 1..6 )
    {
        $result = $null
        try
        {
            ...
```

## Scope issues
- An uninitialized variable in your function is not `$null` if it's set to a value in a parent scope.
  The parent scope could be another function that calls your function and uses the same variable names.
- If I take that same `Do-something` example and remove the loop, I would end up with something that looks like this example:
```ps1
    function Invoke-Something
    {
        $result = 'ParentScope'
        Do-Something
    }

    function Do-Something
    {
        try
        {
            $result = Get-Something -ID $node
        }
        catch
        {
            Write-Verbose "[$result] not valid"
        }

    if ( $null -ne $result )
        {
            Update-Something $result
        }
    }
```
_If the call to `Get-Something` throws an exception,
then my `$null` check finds the `$result` from `Invoke-Something`.
Initializing the value inside your function mitigates this issue._

## Redirect output to $null
- here are times when you have commands that output information or objects
  that you want to suppress. Redirecting output to `$null` does that.

**:Out-Null**
- The `Out-Null` command is the built-in way to redirect pipeline data to `$null`.
```ps1
    New-Item -Type Directory -Path $path | Out-Null
```

**:Assign to $null**
- You can assign the results of a command to `$null` for the same effect as using `Out-Null`.
```ps1
    $null = New-Item -Type Directory -Path $path
```
_Because `$null` is a constant value, you can never overwrite it.
I don't like the way it looks in my code but it often performs faster than `Out-Null`._

**:Redirect to $null**
- You can also use the redirection operator to send output to `$null`.
```ps1
    New-Item -Type Directory -Path $path > $null
```
_If you're dealing with command-line executables that output on the different streams.
You can redirect all output streams to `$null` like this:_
```ps1
    git status *> $null
```

