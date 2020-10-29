# git wisdom

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


