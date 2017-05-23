# Cheatsheet
My personal cheatsheet

## GIT

### Change master branch

```
git checkout before-branch
git merge --strategy=ours --no-commit master
git commit -m "[COMMIT MESSAGE]"
git checkout master
git merge before-branch
```

### Add all files to git despite .gitignore

```
git add [FOLDERNAME]/* -A -f
```


## SSH

### ssh-copy-id on Windows

Run:

```
cat ~/.ssh/id_rsa.pub | ssh [USERNAME]@[IP/HOST] "cat >> ~/.ssh/authorized_keys
```

If `~/.ssh` or `authorized_keys` doesn't exist, you have to create the `.ssh` directory and the `authorized_keys` file the first time.

 1. Create the `.ssh` directory & set the right permissions (if needed):

        mkdir ~/.ssh
        chmod 700 ~/.ssh
 3. Create the `authorized_keys` file:

        touch ~/.ssh/authorized_keys
 4. Set the right permissions:

        chmod 600 ~/.ssh/authorized_keys
        
-- Credit: [Louis Matthijssen](https://askubuntu.com/a/466558)
