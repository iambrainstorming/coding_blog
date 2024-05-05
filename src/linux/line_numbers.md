# Find line numbers of files in a folder

<https://stackoverflow.com/questions/4822471/count-number-of-lines-in-a-git-repository>

```sh
find . -type f -name '*.*' -exec wc -l {} + 
```

## Get line numbers of rust files

```sh
find . -type f -name '*.rs' -exec wc -l {} + 
```
