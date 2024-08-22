# Everything you wanted to know about variable substitution in strings
## Concatenation
```
    $name = 'Kevin Marquette'
    $message = 'Hello, ' + $name
```
_Concatenation works out OK when there are only a few values to add. But this can get complicated quickly._
```
    $first = 'Kevin'
    $last = 'Marquette'
```
```
    $message = 'Hello, ' + $first + ' ' + $last + '.'
```
_This simple example is already getting harder to read._

## Variable substitution
- PowerShell has another option that is easier. You can specify your variables directly in the strings.
```
    $message = "Hello, $first $last."
```
>   The type of quotes you use around the string makes a difference.
    A double quoted string allows the substitution but a single quoted string doesn't.
    There are times you want one or the other so you have an option.

## Command substitution
- Things get a little tricky when you start trying to get the values of properties into a string.
```
    $directory = Get-Item 'c:\windows'
    $message = "Time: $directory.CreationTime"
```
- PowerShell allows you to do command execution inside the string with a special syntax.
```
    $message = "Time: $($directory.CreationTime)"
```

## Command execution
>   You can run commands inside a string. Even though I have this option,
    I don't like it. It gets cluttered quickly and hard to debug.
    I either run the command and save to a variable or use a format string.
```
    $message = "Date: $(Get-Date)"
```

## Format string
>   .NET has a way to format strings that I find fairly easy to work with.
    First let me show you the static method for it before I show you the PowerShell shortcut to do the same thing.
```
    # .NET string format string
    [string]::Format('Hello, {0} {1}.',$first,$last)

    # PowerShell format string
    'Hello, {0} {1}.' -f $first, $last
```
_What is happening here is that the string is parsed for the tokens {0} and {1}, 
then it uses that number to pick from the values provided. If you want to repeat one value some place in the string,
you can reuse that values number._
- _The more complicated the string gets, the more value you get out of this approach._

## Format values as arrays
- If your format line gets too long, you can place your values into an array first.
```
    $values = @(
    "Kevin"
    "Marquette"
  )
    'Hello, {0} {1}.' -f $values
```
_This is not splatting because I'm passing the whole array in, but the idea is similar._

## Advanced formatting
>   I intentionally called these out as coming from .NET because there are lots of formatting options already well documented on it.
    There are built-in ways to format various data types.
```
    "{0:yyyyMMdd}" -f (Get-Date)
    "Population {0:N0}" -f  8175133
```
```
    20211110
    Population 8,175,133
```

## Joining strings
- Sometimes you actually do want to concatenate a list of values together. 
  There's a `-join` operator that can do that for you.
```
    $servers = @(
    'server1'
    'server2'
    'server3'
  )

    $servers  -join ','
```
_If you want to `-join` some strings without a separator, you need to specify an empty string `''`. 
But if that is all you need, there's a faster option._
```
    [string]::Concat('server1','server2','server3')
    [string]::Concat($servers)
```
- _It's also worth pointing out that you can also `-split` strings too._

## Join-Path
- This is often overlooked but a great cmdlet for building a file path.
```
    $folder = 'Temp'
    Join-Path -Path 'c:\windows' -ChildPath $folder
```
>   The great thing about this is it works out the backslashes correctly when it puts the values together. 
    This is especially important if you are taking values from users or config files.
    _This also goes well with `Split-Path` and `Test-Path`. I also cover these in my post about reading and saving to files._

## Strings are arrays
- Remember that a string is just an array of characters. When you add multiple strings together, a new array is created each time.
```
    $message = "Numbers: "
    foreach($number in 1..10000)
    {
        $message += " $number"
    }
```

## StringBuilder
- StringBuilder is also very popular for building large strings from lots of smaller strings.
```
    $stringBuilder = New-Object -TypeName "System.Text.StringBuilder"

    [void]$stringBuilder.Append("Numbers: ")
    foreach($number in 1..10000)
    {
        [void]$stringBuilder.Append(" $number")
    }
    $message = $stringBuilder.ToString()
```
_Again, this is something that I'm reaching out to .NET for. I don't use it often anymore 
but it's good to know it's there._

## Delineation with braces
_This is used for suffix concatenation within the string. 
Sometimes your variable doesn't have a clean word boundary._
```
    $test = "Bet"
    $tester = "Better"
    Write-Host "$test $tester ${test}ter"
```
_Here is an alternate to this approach:_
```
    Write-Host "$test $tester $($test)ter"
    Write-Host "{0} {1} {0}ter" -f $test, $tester
```

## Find and replace tokens
```
    $letter = Get-Content -Path TemplateLetter.txt -RAW
    $letter = $letter -replace '#FULL_NAME#', 'Kevin Marquette'
```
>   You may have lots of tokens to replace. 
    The trick is to use a very distinct token that is easy to find and replace.
    I tend to use a special character at both ends to help distinguish it.

## Replace multiple tokens
```
    $tokenList = @{
    Full_Name = 'Kevin Marquette'
    Location = 'Orange County'
    State = 'CA'
    }

    $letter = Get-Content -Path TemplateLetter.txt -RAW
    foreach( $token in $tokenList.GetEnumerator() )
    {
        $pattern = '#{0}#' -f $token.key
        $letter = $letter -replace $pattern, $token.Value
    }
```
_Those tokens could be loaded from JSON or CSV if needed._


## ExecutionContext ExpandString
- There's a clever way to define a substitution string with single quotes and expand the variables later.
  Look at this example:
```
    $message = 'Hello, $Name!'
    $name = 'Kevin Marquette'
    $string = $ExecutionContext.InvokeCommand.ExpandString($message)
```
- If we expand on that just a little bit, we can perform this substitution over and over with different values.
```
    $message = 'Hello, $Name!'
    $nameList = 'Mark Kraus','Kevin Marquette','Lee Dailey'
    foreach($name in $nameList){
        $ExecutionContext.InvokeCommand.ExpandString($message)
    }
```
- _To keep going on this idea; you could be importing a large email template from a text file to do this.
I have to thank Mark Kraus for this suggestion._



