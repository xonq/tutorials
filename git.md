# git wisdom

NOTE - git commands need to be run in the git repository folder

<br /><br />

### initializing your repository
#### cloning (downloading) an existing repository

to clone
```
git clone <URL.git>
```

to update
```
git pull
```

to setup for pushing changes (see [tracking](https://gitlab.com/xonq/tutorials/-/blob/git.md#tracking-your-repository) to push changes)
```
git remote set-url --add --push origin <URL>
``` 

<br />

#### creating a new repository

initialize git in a folder
```
git init
```

link online repository, add files, and push
```
git remote add origin <URL>
git add .
git commit -m 'init'
git push -u origin master
```

<br /><br />

### tracking your repository

add and push a file:
```
git add <FILE>
git commit -m '<MESSAGE>'
git push
```
alternatively, you can use `git add .` to add all files and commit them together

<br /><br />

### hide from git
edit/create `.gitignore` in the repository root folder and edit with folders to hide. Make sure the path(s) are *relative* to the repository root directory.
```
<HIDE1>
<HIDE2>
```

add `.gitignore`, but DON'T PUSH UNTIL YOUR HIDDEN FOLDER IS CACHED (below)
```
git add .gitignore
git commit -m 'init'
```

remove hidden path from tracking, but keep a copy and push changes
```
git rm -r --cached <YOUR/HIDDEN/PATH>
git commit -m '<YOURCOMMIT>'
git push
```
