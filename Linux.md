# Linux

## grep 

Mix multiple time-ordered logs in a single ordered logstream and find stuff inside.

```Shell
grep -h 'xxxx' *.log | sort
```

Filter well-formed CSV measures:

```Shell
egrep '^".{24}","[a-zA-Z]+",[0-9\.]+$' measure*
```