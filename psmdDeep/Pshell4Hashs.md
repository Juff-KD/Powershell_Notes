# What is a hashtable?
>   A hashtable is a data structure, much like an array,
    except you store each value (object) using a key. 
    It's a basic key/value store. First, we create an empty hashtable.
```
    $ageList = @{}
```
- _Notice that braces, instead of parentheses, 
  are used to define a hashtable. Then we add an item using a key like this:_
~~~powershell
    $key = 'Kevin'
    $value = 36
    $ageList.add( $key, $value )

    $ageList.add( 'Alex', 9 )
~~~
## Using the brackets for access
```
    $ageList['Kevin']
    $ageList['Alex']
```
- _We can use this approach to add or update values into the hashtable too. 
  This is just like using the add() function above._
```
    $ageList = @{}

    $key = 'Kevin'
    $value = 36
    $ageList[$key] = $value

    $ageList['Alex'] = 9
```
## Creating hashtables with values
- You can pre-populate the keys and values when you create them.
```
    $ageList = @{
    Kevin = 36
    Alex  = 9
  }
```
## As a lookup table
- The real value of this type of a hashtable is that 
  you can use them as a lookup table.
```
    $environments = @{
    Prod = 'SrvProd05'
    QA   = 'SrvQA02'
    Dev  = 'SrvDev12'
  }

    $server = $environments[$env]
```
## Multiselection
- PowerShell allows you to provide an array of keys to get multiple values.
```
    $environments[@('QA','DEV')]
    $environments[('QA','DEV')]
    $environments['QA','DEV']
```
## Iterating hashtables
- The first thing to notice is that if you pipe your hashtable, 
  the pipe treats it like one object.
```powershell
    PS> $ageList | Measure-Object
    count : 1
```
_Even though the `.count` property tells you how many values it contains._
```
    PS> $ageList.count
    2
```
_You get around this issue by using the `.values` property if all you need is just the values._
```
    PS> $ageList.values | Measure-Object -Average
    Count   : 2
    Average : 22.5
```
+ It's often more useful to enumerate the keys and use them to access the values.
```
    PS> $ageList.keys | ForEach-Object{
        $message = '{0} is {1} years old!' -f $_, $ageList[$_]
        Write-Output $message
    }
    Kevin is 36 years old
    Alex is 9 years old
```
+ Here is the same example with a foreach(){...} loop.
```
    foreach($key in $ageList.keys)
  {
    $message = '{0} is {1} years old' -f $key, $ageList[$key]
    Write-Output $message
  }
```
## GetEnumerator()
+ That brings us to GetEnumerator() for iterating over our hashtable.
  The enumerator gives you each key/value pair one after another.
```
    $ageList.GetEnumerator() | ForEach-Object{
    $message = '{0} is {1} years old!' -f $_.key, $_.value
    Write-Output $message
  }
```
## BadEnumeration
+ One important detail is that you can't modify a hashtable while 
  it's being enumerated. If we start with our basic `$environments` example:
```
    $environments = @{
    Prod = 'SrvProd05'
    QA   = 'SrvQA02'
    Dev  = 'SrvDev12'
  }
```
_And trying to set every key to the same server value fails._
```
    $environments.Keys | ForEach-Object {
    $environments[$_] = 'SrvDev03'
  }

  An error occurred while enumerating through a collection: Collection was modified;
  enumeration operation may not execute.
  + CategoryInfo          : InvalidOperation: tableEnumerator:HashtableEnumerator) [],
  RuntimeException
  + FullyQualifiedErrorId : BadEnumeration
```
_This will also fail even though it looks like it should also be fine:_
```
    foreach($key in $environments.keys) {
    $environments[$key] = 'SrvDev03'
  }

  Collection was modified; enumeration operation may not execute.
    + CategoryInfo          : OperationStopped: (:) [], InvalidOperationException
    + FullyQualifiedErrorId : System.InvalidOperationException
```
+ ***The trick to this situation is to clone the keys before doing the enumeration.**
```
    $environments.Keys.Clone() | ForEach-Object {
    $environments[$_] = 'SrvDev03'
  }
```

# Hashtable as a collection of properties
## Property-based access
```
    $ageList = @{}
    $ageList.Kevin = 35
    $ageList.Alex = 9
```
```
    $person = @{
    name = 'Kevin'
    age  = 36
  }
```
- And we can add and access attributes on the `$person` like this.
```
    $person.city = 'Austin'
    $person.state = 'TX'
```
## Checking for keys and values
_In most cases, you can just test for the value with something like this:_
```
    if( $person.age ){...}
```
>   It's simple but has been the source of many bugs for me because 
    I was overlooking one important detail in my logic.
    I started to use it to test if a key was present.
    When the value was $false or zero, that statement would return $false unexpectedly.
```
    if( $person.age -ne $null ){...}
```
>   _This works around that issue for zero values but not $null vs non-existent keys.
    Most of the time you don't need to make that distinction but there are functions for when you do._
```
    if( $person.ContainsKey('age') ){...}
```
>   *We also have a ContainsValue() for the situation where you need to test for a value
    without knowing the key or iterating the whole collection.

# Removing and clearing keys
+ You can remove keys with the .Remove() function.
```
    $person.remove('age')
```
+ Assigning them a `$null` value just leaves you with a key that has a `$null` value.

- *A common way to clear a hashtable is to just initialize it to an empty hashtable.
```
    $person = @{}
```
- While that does work, try to use the clear() function instead.
```
    $person.clear()
```
# All the fun stuff
## Ordered hashtables
- By default, hashtables aren't ordered (or sorted).
  Thankfully, there's a way to do that with the ordered keyword.
```
    $person = [ordered]@{
    name = 'Kevin'
    age  = 36
  }
```
## Inline hashtables
- you can separate the key/value pairs with a semicolon.
```
    $person = @{ name = 'kevin'; age = 36; }
```
## Custom expressions in common pipeline commands
>   There are a few cmdlets that support the use of hashtables to create custom
    or calculated properties. You commonly see this with Select-Object and Format-Table.
    The hashtables have a special syntax that looks like this when fully expanded.
```
    $property = @{
    name = 'totalSpaceGB'
    expression = { ($_.used + $_.free) / 1GB }
  }
```
>   The name is what the cmdlet would label that column.
    The expression is a script block that is executed where `$_` is the value of the object on the pipe.
    Here is that script in action:
```
    $drives = Get-PSDrive | Where Used
    $drives | Select-Object -Property name, $property

    Name     totalSpaceGB
    ----     ------------
    C    238.472652435303
```
- I placed that in a variable but it could easily be defined inline
  and you can shorten `name` to `n` and `expression` to `e` while you're at it.
```
    $drives | Select-Object -property name, @{n='totalSpaceGB';e={($_.used + $_.free) / 1GB}}
```
## Custom sort expression
- You can either add the data to the object before you sort it or create a custom expression for `Sort-Object`.
```
    Get-ADUser | Sort-Object -Property @{ e={ Get-TotalSales $_.Name } }
```
## Sort a list of Hashtables
>   If you have a list of hashtables that you want to sort,
    you'll find that the Sort-Object doesn't treat your keys as properties.
    We can get a round that by using a custom sort expression.
```
    $data = @(
    @{name='a'}
    @{name='c'}
    @{name='e'}
    @{name='f'}
    @{name='d'}
    @{name='b'}
  )

    $data | Sort-Object -Property @{e={$_.name}}
```
## Splatting hashtables at cmdlets
- This is one of my favorite things about hashtables that many people don't discover early on.
>   *The idea is that instead of providing all the properties to a cmdlet on one line,
    you can instead pack them into a hashtable first.
    Then you can give the hashtable to the function in a special way.
    Here is an example of creating a DHCP scope the normal way. 
```
    Add-DhcpServerV4Scope -Name 'TestNetwork' -StartRange '10.0.0.2' -EndRange '10.0.0.254' -SubnetMask '255.255.255.0' -Description 'Network for testlab A' -LeaseDuration (New-TimeSpan -Days 8) -Type "Both"
```
>   Without using splatting, all those things need to be defined on a single line.
    It either scrolls off the screen or will wrap where ever it feels like. Now compare that to a command that uses splatting.
```
    $DHCPScope = @{
    Name          = 'TestNetwork'
    StartRange    = '10.0.0.2'
    EndRange      = '10.0.0.254'
    SubnetMask    = '255.255.255.0'
    Description   = 'Network for testlab A'
    LeaseDuration = (New-TimeSpan -Days 8)
    Type          = "Both"
  }
    Add-DhcpServerV4Scope @DHCPScope
```
_*The use of the `@` sign instead of the `$` is what invokes the splat operation._

## Splatting for optional parameters
-  Let's say I have a function that wraps a `Get-CIMInstance` call that has an optional $Credential argument.
```
    $CIMParams = @{
    ClassName = 'Win32_Bios'
    ComputerName = $ComputerName
  }

  if($Credential)
  {
      $CIMParams.Credential = $Credential
  }

  Get-CIMInstance @CIMParams
```
- I start by creating my hashtable with common parameters. Then I add the $Credential if it exists.
  Because I'm using splatting here, I only need to have the call to Get-CIMInstance in my code once.
## Multiple splats
```
    $Common = @{
    SubnetMask  = '255.255.255.0'
    LeaseDuration = (New-TimeSpan -Days 8)
    Type = "Both"
  }

  $DHCPScope = @{
      Name        = 'TestNetwork'
      StartRange  = '10.0.0.2'
      EndRange    = '10.0.0.254'
      Description = 'Network for testlab A'
  }

  Add-DhcpServerv4Scope @DHCPScope @Common
```
## Splatting for clean code
```
    $log = @{Path = '.\logfile.log'}
    Add-Content "logging this command" @log
```
## Splatting executables
- _Splatting also works on some executables that use a `/param:value` syntax.
  `Robocopy.exe`, for example, has some parameters like this._
```
    $robo = @{R=1;W=1;MT=8}
    robocopy source destination @robo
```
## Adding hashtables
```
    $person += @{Zip = '78701'}
```
- ***This only works if the two hashtables don't share a key.**

## Nested hashtables
```
    $person = @{
    name = 'Kevin'
    age  = 36
  }
    $person.location = @{}
    $person.location.city = 'Austin'
    $person.location.state = 'TX'
```
_We can do this all inline too._
```
    $person = @{
    name = 'Kevin'
    age  = 36
    location = @{
        city  = 'Austin'
        state = 'TX'
      }
    }
```
```
    $person.location.city
    Austin
```
```
    $people = @{
    Kevin = @{
        age  = 36
        city = 'Austin'
    }
    Alex = @{
        age  = 9
        city = 'Austin'
    }
  }
```
- The values are still easy to access even when they're nested using whatever approach you prefer.
```
    PS> $people.kevin.age
    36
    PS> $people.kevin['city']
    Austin
    PS> $people['Alex'].age
    9
    PS> $people['Alex']['City']
    Austin
```
- If I need to walk the list or programmatically access the keys,
  I use the brackets to provide the key name.
```
    foreach($name in $people.keys)
  {
    $person = $people[$name]
    '{0}, age {1}, is in {2}' -f $name, $person.age, $person.city
  }
```
## Looking at nested hashtables
```
    PS> $people
    Name                           Value
    ----                           -----
    Kevin                          {age, city}
    Alex                           {age, city}
```
- My go to command for looking at these things is `ConvertTo-JSON` 
  because it's very clean and I frequently use JSON on other things.
```
    PS> $people | ConvertTo-Json
    {
    "Kevin":  {
                "age":  36,
                "city":  "Austin"
            },
    "Alex":  {
                "age":  9,
                "city":  "Austin"
            }
    }
```
## Creating objects
>   Sometimes you just need to have an object and using a hashtable to hold properties just isn't getting the job done.
    Most commonly you want to see the keys as column names.
    *A pscustomobject makes that easy.
```
    $person = [pscustomobject]@{
    name = 'Kevin'
    age  = 36
    }

    $person

    name  age
    ----  ---
    Kevin  36
```
- ***Even if you don't create it as a pscustomobject initially, you can always cast it later when needed.**
```
    $person = @{
    name = 'Kevin'
    age  = 36
  }

    [pscustomobject]$person

    name  age
    ----  ---
    Kevin  36
```

















