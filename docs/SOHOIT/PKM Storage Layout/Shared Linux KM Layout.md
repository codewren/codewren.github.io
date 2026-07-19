# 1. ALL  $HOME Base Linux KM Layout
 
## 1.1   Working Space Layout
    - top location 
    
```
$HOME/zpaces/
├── cntnrspc              ## Container Spaces
│   ├── <container name>
│   └── <container name>
├── packages              ##  instalaler package
│   ├── <jdk.tar.gz>
├── spcmecury             ## Space <Name: mecury>
│    ├── codehive          ### coding project
│    │   └── armv9qemu     ###  <prj name>
│    ├── devopt            ###  <tool chain of this space, clang, rust
│    ├── env
│    │   └── setenv.sh     ### All environment variable setting for this spaces   
│    └── tmp
├── envutil
│   ├── shell script    ## help to detect and fix the tools env setting
├── databox
│   ├── _appname_    ## app's data folder
```
  

# 2.  Linux Desktop Application -  $HOME-based 
## 2.1  Location name
 -  $HOME/zone/apphub
 -  $HOME/zone/appimages
 -  $HOME/zone/bin
## 2.2 Freq-Used App
- Chrome
- Obsidian
-  rclone
-  electerm
-  jetbrains

# 3. Popular Dev Tool Chain Location

## 3.1  Location Name
 -  $HOME/.local/__{product}__
## 3.2  Freq-Used Tool
### 3.2.1 $HOME-Based Dev Tool
- nvm and nodejs
- uv and python
- rust (rustc and cargo)
### 3.2.2   Coding Agent
 - claude code cli
 - gemini cli
 - opencode cli

#  4.  Common System Dev Tool
- git
- gcc
- llvm?

# 5 Default Env Setup and $PATH 

$HOME/.bashrc

```
export PATH=$HOME/.local/bin:$PATH
```


