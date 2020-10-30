# git wisdom

### installation
UNDER CONSTRUCTION

initialize git in a folder, set o
```
git init
```

<br />

### general use

add and push a file:
```
git add <FILE>
git commit -m '<MESSAGE>'
git push
```
alternatively, you can use `git add .` to add all files and commit them together

<br />

### hide from git
edit `.gitignore`:
```
<YOUR/HIDDEN/PATH>
```

remove from tracking, but keep a copy
```
git rm -r --cached <YOUR/HIDDEN/PATH>
git commit -m '<YOURCOMMIT>'
git push
```


