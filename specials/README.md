# Special versions

Note the needed chars `<?=>` are only relevant when you are not in php context and need to declare it.
If you are already in php you can ignore them and use teh shells without it but to be fair I included them in the
payload length and at the needed chars.

## Length Ranking

| Rank | Version                                               | Method                     | Length | Needed Chars             |
|------|-------------------------------------------------------|----------------------------|--------|--------------------------|
| 1    | without dollar signs, brackets                        | SYSTEM + CHR payload craft | 419    | <?=(9*.+1^2-87)56034;    |
| 2    | without dollar signs, brackets, semicolon             | SYSTEM + CHR payload craft | 420    | <?=(9*.+1^2-87)56034>    |
| 3    | without dollar signs (`join(array_map(chr,[1,2,3]))`) | SYSTEM + CHR payload craft | 452    | <?=(9*.+1^2-87)56034[],; |

With the version `without dollar signs` you can add text to the args of system while the other will dramatically
increase in size.