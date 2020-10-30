# git wisdom

NOTE - git commands need to be run in the git repository folder

<br />

### cloning a repository

clone
```
git clone <URL.git>
```

update
```
git pull
```

<br />

### initializing your repository
UNDER CONSTRUCTION

initialize git in a folder, set o
```
git init
```

<br />

### tracking your repository

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


