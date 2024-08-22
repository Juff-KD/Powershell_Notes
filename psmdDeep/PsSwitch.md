# Everything you ever wanted to know about the switch statement

# The `if` statement
```
    if ( Test-Path $Path )
    {
        Remove-Item $Path
    }
```
```
    $day = 3

    if ( $day -eq 0 ) { $result = 'Sunday'        }
    elseif ( $day -eq 1 ) { $result = 'Monday'    }
    elseif ( $day -eq 2 ) { $result = 'Tuesday'   }
    elseif ( $day -eq 3 ) { $result = 'Wednesday' }
    elseif ( $day -eq 4 ) { $result = 'Thursday'  }
    elseif ( $day -eq 5 ) { $result = 'Friday'    }
    elseif ( $day -eq 6 ) { $result = 'Saturday'  }

    $result
```

# Switch statement
_*The switch statement allows you to provide a variable and a list of possible values. If the value matches the variable, then its scriptblock is executed._
```
    $day = 3

    switch ( $day )
    {
        0 { $result = 'Sunday'    }
        1 { $result = 'Monday'    }
        2 { $result = 'Tuesday'   }
        3 { $result = 'Wednesday' }
        4 { $result = 'Thursday'  }
        5 { $result = 'Friday'    }
        6 { $result = 'Saturday'  }
    }

    $result
```

## Assign to a variable
- We can write that last example in another way.
```
    $result = switch ( $day )
    {
        0 { 'Sunday'    }
        1 { 'Monday'    }
        2 { 'Tuesday'   }
        3 { 'Wednesday' }
        4 { 'Thursday'  }
        5 { 'Friday'    }
        6 { 'Saturday'  }
    }
```
## Default
_We can use the default keyword to identify the what should happen if there is no match._
```
    $result = switch ( $day )
    {
        0 { 'Sunday' }
        # ...
        6 { 'Saturday' }
        default { 'Unknown' }
    }
```
## Strings
_I was matching numbers in those last examples, but you can also match strings._
```
    $item = 'Role'

    switch ( $item )
    {
        Component
        {
            'is a component'
        }
        Role
        {
            'is a role'
        }
        Location
        {
            'is a location'
        }
    }
```
**Output:**
```
    is a role
```
_I decided not to wrap the Component,Role and Location matches in quotes here to highlight that they're optional.
The switch treats those as a string in most cases._

## Arrays
- One of the cool features of the PowerShell switch is the way it handles arrays.
  If you give a switch an array, it processes each element in that collection.
```
    $roles = @('WEB','Database')

    switch ( $roles ) {
        'Database'   { 'Configure SQL' }
        'WEB'        { 'Configure IIS' }
        'FileServer' { 'Configure Share' }
    }
```
**Output**
```
    Configure IIS
    Configure SQL
```
_If you have repeated items in your array,
then they're matched multiple times by the appropriate section._

## PSItem
>   You can use the `$PSItem` or `$_` to reference the current item that was processed.
    When we do a simple match, $PSItem is the value that we're matching.
    I'll be performing some advanced matches in the next section where this variable is used.

## Parameters
### -CaseSensitive
_The matches aren't case-sensitive by default. If you need to be case-sensitive, you can use -CaseSensitive.
This can be used in combination with the other switch parameters._

### -Wildcard
_We can enable wildcard support with the `-wildcard` switch.
This uses the same wildcard logic as the `-like` operator to do each match._
```
    $Message = 'Warning, out of disk space'

    switch -Wildcard ( $message )
    {
        'Error*'
        {
            Write-Error -Message $Message
        }
        'Warning*'
        {
            Write-Warning -Message $Message
        }
        default
        {
            Write-Information $message
        }
    }
```
**Output**
```
    Warning, out of disk space
```
_Here we're processing a message and then outputting it on different streams based on the contents._

### -Regex
- The switch statement supports regex matches just like it does wildcards.
```
    switch -Regex ( $message )
    {
        '^Error'
        {
            Write-Error -Message $Message
        }
        '^Warning'
        {
            Write-Warning -Message $Message
        }
        default
        {
            Write-Information $message
        }
    }
```
### -File
>   _A little known feature of the switch statement is that it can process a file with the `-File` parameter.
    You use `-file` with a path to a file instead of giving it a variable expression._
```
    switch -Wildcard -File $path
    {
        'Error*'
        {
            Write-Error -Message $PSItem
        }
        'Warning*'
        {
            Write-Warning -Message $PSItem
        }
        default
        {
            Write-Output $PSItem
        }
    }
```
_It works just like processing an array. In this example, I combine it with wildcard matching and make use of the `$PSItem`.
This would process a log file and convert it to warning and error messages depending on the regex matches._

# Advanced details
## Expressions
- The switch can be on an expression instead of a variable.
```
    switch ( ( Get-Service | Where status -eq 'running' ).name ) {...}
```
## Multiple matches
>   ou may have already picked up on this, but a switch can match to multiple conditions.
    This is especially true when using `-wildcard` or `-regex` matches.
    You can add the same condition multiple times and all are triggered.
```
    switch ( 'Word' )
    {
        'word' { 'lower case word match' }
        'Word' { 'mixed case word match' }
        'WORD' { 'upper case word match' }
    }
```
**Output**
```
    lower case word match
    mixed case word match
    upper case word match
```
_All three of these statements are fired.
This shows that every condition is checked (in order).
This holds true for processing arrays where each item checks each condition._

### Continue
>   Normally, this is where I would introduce the `break` statement, but it's better that we learn how to use `continue` first.
    Just like with a `foreach` loop, `continue` continues onto the next item in the collection or exits the `switch` if there are no more items.
    We can rewrite that last example with continue statements so that only one statement executes.
```
    switch ( 'Word' )
    {
        'word'
        {
            'lower case word match'
            continue
        }
        'Word'
        {
            'mixed case word match'
            continue
        }
        'WORD'
        {
            'upper case word match'
            continue
        }
    }
```
**Output**
```
    mixed case word match
```
>   Instead of matching all three items, the first one is matched and the switch continues to the next value.
    Because there are no values left to process, the switch exits. This next example is showing how a wildcard could match multiple items.
```
    switch -Wildcard -File $path
    {
        '*Error*'
        {
            Write-Error -Message $PSItem
            continue
        }
        '*Warning*'
        {
            Write-Warning -Message $PSItem
            continue
        }
        default
        {
            Write-Output $PSItem
        }
    }
```
_*Because a line in the input file could contain both the word Error and Warning,
we only want the first one to execute and then continue processing the file._

### Break
>   A `break` statement exits the switch. This is the same behavior that `continue` presents for single values.
    The difference is shown when processing an array.
    `break` stops all processing in the switch and `continue` moves onto the next item.
```
    $Messages = @(
    'Downloading update'
    'Ran into errors downloading file'
    'Error: out of disk space'
    'Sending email'
    '...'
    )

    switch -Wildcard ($Messages)
    {
        'Error*'
        {
            Write-Error -Message $PSItem
            break
        }
        '*Error*'
        {
            Write-Warning -Message $PSItem
            continue
        }
        '*Warning*'
        {
            Write-Warning -Message $PSItem
            continue
        }
        default
        {
            Write-Output $PSItem
        }
    }
```
**Output**
```
  Downloading update
  WARNING: Ran into errors downloading file
  write-error -message $PSItem : Error: out of disk space
  + CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
  + FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException
```
>   In this case, if we hit any lines that start with `Error` then we get an error and the switch stops.
    This is what that `break` statement is doing for us.
    If we find `Error` inside the string and not just at the beginning, we write it as a warning.
    We do the same thing for `Warning`. It's possible that a line could have both the word `Error` and `Warning`, but we only need one to process.
    This is what the `continue` statement is doing for us.

### Break labels
- The switch statement supports break/continue labels just like foreach.
```
    :filelist foreach($path in $logs)
    {
        :logFile switch -Wildcard -File $path
        {
            'Error*'
            {
                Write-Error -Message $PSItem
                break filelist
            }
            'Warning*'
            {
                Write-Error -Message $PSItem
                break logFile
            }
            default
            {
                Write-Output $PSItem
            }
        }
    }
```
>   I personally don't like the use of break labels but I wanted to point them out
    because they're confusing if you've never seen them before.
    When you have multiple `switch` or `foreach` statements that are nested,
    you may want to break out of more than the inner most item.
    You can place a label on a `switch` that can be the target of your `break`.

### Enum
- PowerShell 5.0 gave us enums and we can use them in a switch.
```
    enum Context {
    Component
    Role
    Location
    }

    $item = [Context]::Role

    switch ( $item )
    {
        Component
        {
            'is a component'
        }
        Role
        {
            'is a role'
        }
        Location
        {
            'is a location'
        }
    }
```
**Output**
```
    is a role
```
- *If you want to keep everything as strongly typed enums, then you can place them in parentheses.
```
    switch ($item )
    {
        ([Context]::Component)
        {
            'is a component'
        }
        ([Context]::Role)
        {
            'is a role'
        }
        ([Context]::Location)
        {
            'is a location'
        }
    }
```
_*The parentheses are needed here so that the switch doesn't treat the value `[Context]::Location` as a literal string._

### ScriptBlock
- We can use a scriptblock to perform the evaluation for a match if needed.
```
    $age = 37

    switch ( $age )
    {
        {$PSItem -le 18}
        {
            'child'
        }
        {$PSItem -gt 18}
        {
            'adult'
        }
    }
```
**Output**
```
    adult
```
- One thing that I think helps with legibility is to place the scriptblock in parentheses.
```
    switch ( $age )
    {
        ({$PSItem -le 18})
        {
            'child'
        }
        ({$PSItem -gt 18})
        {
            'adult'
        }
    }
```
_It still executes the same way and gives a better visual break when quickly looking at it._

## Regex $matches
_We need to revisit regex to touch on something that isn't immediately obvious.
The use of regex populates the `$matches` variable._
```
    $message = 'my ssn is 123-23-3456 and credit card: 1234-5678-1234-5678'

    switch -regex ($message)
    {
        '(?<SSN>\d\d\d-\d\d-\d\d\d\d)'
        {
            Write-Warning "message contains a SSN: $($matches.SSN)"
        }
        '(?<CC>\d\d\d\d-\d\d\d\d-\d\d\d\d-\d\d\d\d)'
        {
            Write-Warning "message contains a credit card number: $($matches.CC)"
        }
        '(?<Phone>\d\d\d-\d\d\d-\d\d\d\d)'
        {
            Write-Warning "message contains a phone number: $($matches.Phone)"
        }
    }
```
**Output**
```
    WARNING: message may contain a SSN: 123-23-3456
    WARNING: message may contain a credit card number: 1234-5678-1234-5678
```
### $null
- You can match a $null value that doesn't have to be the default.
```
    $values = '', 5, $null
    switch ( $values )
    {
        $null          { "Value '$_' is `$null" }
        { '' -eq $_ }  { "Value '$_' is an empty string" }
        default        { "Value [$_] isn't an empty string or `$null" }
    }
```
**Output**
```
    Value '' is an empty string
    Value [5] isn't an empty string or $null
    Value '' is $null
```
_When testing for an empty string in a switch statement,
it's important to use the comparison statement as shown in this example instead of the raw value ''.
In a switch statement, the raw value '' also matches $null. For example:_
```
    $values = '', 5, $null
    switch ( $values )
    {
        $null          { "Value '$_' is `$null" }
        ''             { "Value '$_' is an empty string" }
        default        { "Value [$_] isn't an empty string or `$null" }
    }
```
**Output**
```
    Value '' is an empty string
    Value [5] isn't an empty string or $null
    Value '' is $null
    Value '' is an empty string
```
_Also, be careful with empty returns from cmdlets.
Cmdlets or pipelines that have no output are treated as an empty array
that doesn't match anything, including the `default` case._
```
    $file = Get-ChildItem NonExistantFile*
    switch ( $file )
    {
        $null   { '$file is $null' }
        default { "`$file is type $($file.GetType().Name)" }
    }
    # No matches
```
## Constant expression
- Lee Dailey pointed out that we can use a constant `$true` expression to evaluate `[bool]` items.
  Imagine if we have several boolean checks that need to happen.
```
    $isVisible = $false
    $isEnabled = $true
    $isSecure = $true

    switch ( $true )
    {
        $isEnabled
        {
            'Do-Action'
        }
        $isVisible
        {
            'Show-Animation'
        }
        $isSecure
        {
            'Enable-AdminMenu'
        }
    }
```
**Output**
```
    Do-Action
    Enabled-AdminMenu
```
_This is a clean way to evaluate and take action on the status of several boolean fields.
The cool thing about this is that you can have one match flip the status of a value that hasn't been evaluated yet._
```
    $isVisible = $false
    $isEnabled = $true
    $isAdmin = $false

    switch ( $true )
    {
        $isEnabled
        {
            'Do-Action'
            $isVisible = $true
        }
        $isVisible
        {
            'Show-Animation'
        }
        $isAdmin
        {
            'Enable-AdminMenu'
        }
    }
```
**Output**
```
    Do-Action
    Show-Animation
```
_Setting $isEnabled to `$true` in this example makes sure that `$isVisible` is also set to `$true`.
Then when `$isVisible` gets evaluated, its scriptblock is invoked.
This is a bit counter-intuitive but is a clever use of the mechanics._

## $switch automatic variable
>   When the `switch` is processing its values, it creates an enumerator and calls it `$switch`.
    This is an automatic variable created by PowerShell and you can manipulate it directly.
```
    $a = 1, 2, 3, 4

    switch($a) {
        1 { [void]$switch.MoveNext(); $switch.Current }
        3 { [void]$switch.MoveNext(); $switch.Current }
    }
```
**Output**
```
    2
    4
```
## Other patterns
### Hashtables
>    One of the use cases for a hashtable is to be a lookup table.
    That's an alternate approach to a common pattern that a switch statement is often addressing.
```
    $day = 3

    $lookup = @{
        0 = 'Sunday'
        1 = 'Monday'
        2 = 'Tuesday'
        3 = 'Wednesday'
        4 = 'Thursday'
        5 = 'Friday'
        6 = 'Saturday'
    }

    $lookup[$day]
```
**Output**
```
    Wednesday
```
### Enum
```
    $day = 3

    enum DayOfTheWeek {
        Sunday
        Monday
        Tuesday
        Wednesday
        Thursday
        Friday
        Saturday
    }

    [DayOfTheWeek]$day
```
**Output**
```
    Wednesday
```


