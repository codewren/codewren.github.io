

## WSL 2 Setup for SSH Remote Access
Tristan Wu Dec 19, 2023
ref : https://medium.com/@wuzhenquan/windows-and-wsl-2-setup-for-ssh-remote-access-013955b2f421

### Run book

#### 1. Detect if systemctl works
```
~$ systemctl --help
systemctl [OPTIONS...] COMMAND ...

Query or send control commands to the system manager.

Unit Commands:
  list-units [PATTERN...]             List units currently in memory
  list-sockets [PATTERN...]           List socket units currently in memory,
                                      ordered by address
  list-timers [PATTERN...]            List timer units currently in memory,
                                      ordered by next elapse
  is-active PATTERN...                Check whether units are active
  is-failed PATTERN...                Check whether units are failed
  status [PATTERN...|PID...]          Show runtime status of one or more units
  show [PATTERN...|JOB...]            Show properties of one or more
                                      units/jobs or the manager
```

#### 2. Reinstalling SSH

- Uninstall: sudo apt remove openssh-server
- Reinstall: sudo apt install openssh-server

#### 3.Configuring SSH
```
Open SSH config file: sudo vi /etc/ssh/sshd_config
Change #Port 22 to Port 22 to enable the default port.
Change #PasswordAuthentication yes to PasswordAuthentication yes to enable password authentication.
```


#### 4. Start/Stop/Validation

```
~$ sudo systemctl restart  sshd
~$ sudo systemctl stop  sshd
~$ sudo systemctl start  sshd
```
alternative way 

```
sudo service ssh restart
```

List config
```
~$ systemctl list-units ssh*
  UNIT        LOAD   ACTIVE SUB     DESCRIPTION
  ssh.service loaded active running OpenBSD Secure Shell server

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
1 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```