# PIP

```shell

# show pip version
pip --version
# list all the package you installed 
pip list
# install package
pip install {package_name} == {version}
# uninstall package
pip uninstall {package_name} == {version}
```

## windows

you could create virtual env by using the command as follows

```shell

# create a virtual env with folder name venv
py -3 -m venv venv
# activate the virtual env
venv\Scripts\Activate.ps1


```

if you cannot put these command in the PowerShell, you should configure your Powershell as follows.

Open power shell as administrator and change configure in power shell

```powershell
get-executionpolicy
set-executionpolicy remotesigned
```

## Env for Python2

```shell
virtualenv venv
```

## Env in PyCharm

pycharm creates a folder named env by default for every new created project. The virtual env engine used in pycharm is `Virtualenv`

