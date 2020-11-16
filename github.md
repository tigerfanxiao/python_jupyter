# Basic Practice

```shell
# config git
git config --global user.name "Xiao fan"
git config --global user.email tigerfanxiao@gmail.com 
# show config 
git cofig --list

# copy remote repo
git clone url
# commit
git add .
git commit -m 'first init'

# push to remote repo
git push origin master
# pull from remote repo
git pull origin master
```

### git pull origin  VS git pull origin/master

* `git pull origin master` will pull changes from the **origin remote**, master branch and merge them to the local checked-out branch. 
* `git pull origin/master` will pull changes from the **locally stored** branch origin/master and merge that to the local checked-out branch

### remote

remote is remote repository

```shell
 # view the remote
 git remote -v
 # set remote 
 git remote add origin https://github.com/user/repo.git
```

### Branch

```shell
git checkout master
```
