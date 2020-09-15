## Find

https://www.everythingcli.org/find-exec-vs-find-xargs/

```
find . [args] -exec [cmd] {} \;
```

{} Is a placeholder for the result found by find
\; Says that for each found result, the command (in this case ‘grep’) is executed once with the found result

So in the above example ‘grep’ is executed as many times as a file with the specified pattern has been found:

grep [...] file1
grep [...] file2
...

```
find . [args] -exec [cmd] {} \+
```
{} Is a placeholder for the result found by find
\+ All result lines are concatenated and the command (in this case ‘grep’) is executed only a single time with all found results as a parameter

So in the above example ‘grep’ is executed only once and its parameter are all files found with the specified pattern.

grep [...] file1 file2 ...
