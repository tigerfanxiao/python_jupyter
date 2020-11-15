[ref](https://stackoverflow.com/questions/4037939/powershell-says-execution-of-scripts-is-disabled-on-this-system)

If you're using [Windows Server 2008](https://en.wikipedia.org/wiki/Windows_Server_2008) R2 then there is an *x64* and *x86* version of PowerShell both of which have to have their execution policies set. Did you set the execution policy on both hosts?

As an *Administrator*, you can set the execution policy by typing this into your PowerShell window:

```bsh
Set-ExecutionPolicy RemoteSigned
```

For more information, see *[Using the Set-ExecutionPolicy Cmdlet](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/set-executionpolicy)*.

When you are done, you can set the policy back to its default value with:

```bsh
Set-ExecutionPolicy Restricted
```



配置powershell

不显示完整当前目录

https://medium.com/@stojanpeshov/how-to-configure-the-windows-powershell-to-display-only-the-current-folder-name-97a056f60c91