# Windows Terminal

### [split pane](https://www.howtogeek.com/673729/heres-why-the-new-windows-10-terminal-is-amazing/#:~:text=Split%20Panes%20for%20Multiple%20Shells%20at%20Once&text=To%20create%20a%20new%20pane,D%20to%20keep%20splitting%20it.)

To create a new pane, press Alt+Shift+D. The Terminal will split the current pane into two and give you a second one. Click a pane to select it. You can click a pane and press Alt+Shift+D to keep splitting it.

These panes are linked to tabs, so you can easily have several multi-pane environments in the same Windows Terminal window and switch between them from the tab bar.

Here are some other keyboard shortcuts for working with panes:

- **Create a new pane, splitting horizontally**: Alt+Shift+- (Alt, Shift, and a minus sign)
- **Create a new pane, splitting vertically**: Alt+Shift++ (Alt, Shift, and a plus sign)
- **Move pane focus**: Alt+Left, Alt+Right, Alt+Down, Alt+Up
- **Resize the focused pane**: Alt+Shift+Left, Alt+Shift+Right, Alt+Shift+Down, Alt+Shift+Up
- **Close a pane**: Ctrl+Shift+W

# WLS 2.0

This notes is based on the content of [David Bombal WSL 2 Getting Started tutorial](https://www.youtube.com/watch?v=_fntjriRe48)

we are going to use virtual machine there rather than to use translation layer between windows and linux system.

> Note: if you use WSL, you could have some problem to VMware

## Install Ubuntu Sub-Linux System

### Install by Windows GUI

1. support in win10 version 2004

   win start, `winver`  to see the current version of windows

2. **turn windows features on or off**

   uncheck **Windows Hypervisor Platform**

   check **Windows Subsystem for Linux**

   check **Virtual Machine Platform**

3. Restart you computer

4. Once the computer is restarted, go to the windows store, search for Ubuntu and download the latest version

5. Launch Ubunbu and may prompt a message that you need to download the latest WSL2 Linux kernel

6. After you down the kernel, install the kernel, and launch Ubuntu again

7. After you install ubuntu, create a default account

### Install by powershell

refer the [windows document](https://docs.microsoft.com/en-us/windows/wsl/install-win10)

open powershell as adiministrator

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
wsl --set-default-version 2
```

## Run Ubuntu Sub-Linux System

* win start, search for Ubuntu, you could have the ubuntu shell

* After Ubuntu is running, we can use powershell to manage all the wsl version. 

  open powershell. you could configure its appearance by edit the property

  ```powershell
  wsl -l -v # list all the windows sub linux system
  wsl --set-default-version 2 # set the default version 2
  wsl --set-version Ubuntu-20.04 1 # set back the version to 1
  wsl # run default wsl in powershell console
  exit # exit the wsl
  wsl --help
  wsl --shutdown  # shutdown virtual machine
  ```

# Ubuntu 

### Install python

[refer](https://phoenixnap.com/kb/how-to-install-python-3-ubuntu)

```shell

lsb_release -a # view the release version
sudo apt update 
# install a package management
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.8
# install pip
sudo apt install python3-pip
sudo pip3 install -U netmiko

```

### Port listening

```shell
nc -l 8081 # listen to 8081 port
nc 127.0.0.1 8081 # speak to 8081 port
```

### Search for DNS

```shell
dig google.com # search for dns 
# response
; <<>> DiG 9.16.1-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40125
;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             0       IN      A       216.58.211.238 # This is google server

;; Query time: 10 msec
;; SERVER: 172.22.192.1#53(172.22.192.1)  # This is dns server
;; WHEN: Sun Oct 18 22:12:43 CEST 2020
;; MSG SIZE  rcvd: 54
```

### Enable SSH in WSL

when you enable SSH, you could use SFTP service

```shell
# remove the older version if necessary
sudo apt remove openssh-server
sudo apt install openssh-server

# Edit the sshd_config
sudo vi /etc/ssh/sshd_config

# changes
PasswordAuthentication yes
# Add your login user to the bottom of file 
AllowUsers yourusername
# save & quit

# restart ssh service
sudo service ssh --full-restart
# check the status of ssh
service ssh status
# start ssh service
sudo service ssh start
```

### Allow SSH Service to start without password

```shell
sudo visudo
# add the following line after %sudo ALL=(ALL:ALL) ALL
%sudo ALL=NOPASSWD: /usr/sbin/sshd
```



# Features

### code

input `code .` will open a visual code editor but save in wsl, don't forget to install remote-wsl extension

# express

express library is a JavaScript server

node.js is JavaScript runtime

# Latency & Throughput

**Latency** is basically low long it takes for data to traverse a system 

**Throughput** is how much work a machine could perform in a given period of time. For instance the throughput of a server can be measured in requests per second (RPS or QPS)

### Scenarios

Design is based on the kind for system. For instance, game system not accept high latency, but for bank system, or payment system, accuracy and uptime have higher priority. Based on different demand, architect will think of the priority and tradeoff

### The orders of magnitude

> 1 second (s) = 1000 microsecond (ms) = 10^6 millisecond (&mu;s)

* Reading 1 MB from RAM: 250 &mu;s (0.25 ms), read variable in code
* Reading 1 MB from SSD: 1,000 &mu;s (1 ms), IO frequently accessed and updated data
* Reading 1 MB from over Network: 10,000 &mu;s (10 ms), API call
* Reading 1 MB from HDD: 20,000 &mu;s (20 ms), IO long time, rarely accessed
* Inter-Continental Round Trip - One package (1500 byte): 150,000 &mu;s (150 ms)

**LAG** = High Latency

Notes:

* latency and Throughput are very important measure of a system performance. 

* Latency are Throughput are not necessarily correlated. 

# Cache

### LRU vs LFU

Let's consider a constant stream of cache requests with a cache capacity of 3, see below:

```
A, B, C, A, A, A, A, A, A, A, A, A, A, A, B, C, D
```

If we just consider a **Least Recently Used (LRU)** cache with a HashMap + doubly linked list implementation with O(1) eviction time and O(1) load time, we would have the following elements cached while processing the caching requests as mentioned above.

```
[A]
[A, B]
[A, B, C]
[B, C, A] <- a stream of As keeps A at the head of the list.
[C, A, B]
[A, B, C]
[B, C, D] <- here, we evict A, we can do better! 
```

When you look at this example, you can easily see that we can do better - given the higher expected chance of requesting an A in the future, we should not evict it even if it was least recently used.

```
A - 12
B - 2
C - 2
D - 1
```

**Least Frequently Used (LFU)** cache takes advantage of this information by keeping track of how many times the cache request has been used in its eviction algorithm.

[ref from stackoverflow](https://stackoverflow.com/questions/17759560/what-is-the-difference-between-lru-and-lfu/29225598#:~:text=LRU%20is%20a%20cache%20eviction%20algorithm%20called%20least%20recently%20used%20cache.&text=LFU%20is%20a%20cache%20eviction%20algorithm%20called%20least%20frequently%20used%20cache.&text=the%20main%20difference%20is%20that,based%20on%20recent%20used%20pages.)

# Proxy

# git

```shell
# download repository from URL
git clone url
pip3 freeze > requirements.txt
pip3 install -r requirements.txt
git add . # add all changes to the stage
git commit -m"init" # commit it before push
git push origin master # push to master branch

# fetch meta data from remote
git fetch origin
# pull changes from remote
git pull origin


```

