# PowerShell101 \ Chapter 6 - Flow control

## ForEach-Object
- `ForEach-Object` is a cmdlet for iterating through items in a pipeline such as with PowerShell one-liners.
  `ForEach-Object` streams the objects through the pipeline.
```ps1
    'ActiveDirectory', 'SQLServer' |
      ForEach-Object {Get-Command -Module $_} |
        Group-Object -Property ModuleName -NoElement |
            Sort-Object -Property Count -Descending
```
**Output**
```ps1
    Count Name
    ----- ----
      147 ActiveDirectory
      82 SqlServer
```
>   When using the foreach keyword, you must store all of the items in memory before iterating through them,
    which could be difficult if you don't know how many items you're working with.
```ps1
    $ComputerName = 'DC01', 'WEB01'
    foreach ($Computer in $ComputerName) {
      Get-ADComputer -Identity $Computer
    }
```
>   Other times, you can get the same results while eliminating the loop altogether.
    Consult the cmdlet help to understand your options.
```ps1
    'DC01', 'WEB01' | Get-ADComputer
```

## For
- A for loop iterates while a specified condition is true.
  The for loop is not something that I use often, but it does have its uses.
```ps1
    for ($i = 1; $i -lt 5; $i++) {
      Write-Output "Sleeping for $i seconds"
      Start-Sleep -Seconds $i
    }
```

## Do
- There are two different `do` loops in PowerShell. `Do Until` runs while the specified condition is false.
```ps1
    $number = Get-Random -Minimum 1 -Maximum 10
    do {
      $guess = Read-Host -Prompt "What's your guess?"
      if ($guess -lt $number) {
        Write-Output 'Too low!'
      }
      elseif ($guess -gt $number) {
        Write-Output 'Too high!'
      }
    }
    until ($guess -eq $number)
```

- `Do While` is just the opposite. It runs as long as the specified condition evaluates to true.
```ps1
    $number = Get-Random -Minimum 1 -Maximum 10
    do {
      $guess = Read-Host -Prompt "What's your guess?"
      if ($guess -lt $number) {
        Write-Output 'Too low!'
      } elseif ($guess -gt $number) {
        Write-Output 'Too high!'
      }
    }
    while ($guess -ne $number)
```

## While
- Similar to the `Do While` loop, a While loop runs as long as the specified condition is true.
```ps1
    $date = Get-Date -Date 'November 22'
    while ($date.DayOfWeek -ne 'Thursday') {
      $date = $date.AddDays(1)
    }
    Write-Output $date
```

## Break, Continue, and Return
- `Break` is designed to break out of a loop. It's also commonly used with the `switch` statement.
```ps1
    for ($i = 1; $i -lt 5; $i++) {
      Write-Output "Sleeping for $i seconds"
      Start-Sleep -Seconds $i
      break
    }
```
- Continue is designed to skip to the next iteration of a loop.
```ps1
    while ($i -lt 5) {
      $i += 1
      if ($i -eq 3) {
        continue
      }
      Write-Output $i
    }
```
- Return is designed to exit out of the existing scope.
```ps1
    $number = 1..10
    foreach ($n in $number) {
      if ($n -ge 4) {
        Return $n
      }
    }
```

