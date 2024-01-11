# Special versions

Note the needed chars `<?=>` are only relevant when you are not in php context and need to declare it.
If you are already in php you can ignore them and use the shells without it but I included them anyway in the
payload `Length` and at the `Needed Chars`.

## Length Ranking

| Rank | Version                                               | Method                     | Length | Needed Chars             |
|------|-------------------------------------------------------|----------------------------|--------|--------------------------|
| 1    | without dollar signs, brackets                        | SYSTEM + CHR payload craft | 419    | <?=(9*.+1^2-87)56034;    |
| 2    | without dollar signs, brackets, semicolon             | SYSTEM + CHR payload craft | 420    | <?=(9*.+1^2-87)56034>    |
| 3    | without dollar signs (`join(array_map(chr,[1,2,3]))`) | SYSTEM + CHR payload craft | 452    | <?=(9*.+1^2-87)56034[],; |

With the version `without dollar signs` you can add more text to the args of system since its more compact but requires
initially more length. I recommend using this when your arg is longer than `10` chars.

## Extend

I can add different bypasses if requested, not everything is possible to bypass but i will try my best.

Just create a issue with the following format (Example):

```
Title: Extend specials with xxxxx
=============
Blacklisted chars: ...
OR
Whitelisted chars: ...
=============

Other context:
====================
This needs to work as a standalone php file.

PHP version: 7.4

Existing Bypass?
...
```

And i will try to bypass it and add it to the versions.

Pleas do not send me active ctf challenges, i will not add them directly anyway.
