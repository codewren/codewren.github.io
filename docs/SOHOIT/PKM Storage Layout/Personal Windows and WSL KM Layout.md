# 1. Windows KM  Layout

##  1.1 Top Location 

- zone    for app
- zpaces  for appdata and other data

## 1.2. _\$Windows_\$HOME Base Location 

### 1.2.1  $HOME-based  Desktop Application 

-  Obsidian
-  rclone
-  electerm
-  jetbrains
#### Location name
 -  _[C]_/zone/_appname_
 -  _[C]_/zone/bin

#### Data Location
-  _[C]_/zpaces/appdata/_appname_    (WSL /mnt/c/zpaces/appdata/_appname_ )
### 1.2.2 Popular Dev Tool Chain Location
- Optional - Most dev tool are in WSL
#### Location Name
 -  $HOME/.local/__product__

# 2. WSL KM Layout

## 2.1  $HOME-based  WSL Application
- Optional  Most app should be in Windows

## 2.2. Popular Dev Tool Chain Location
- nvm and nodejs
- uv and python
- rust (rustc and cargo)
 - claude code cli
 - gemini cli
 - opencode cli

#### Location Name
 -  $HOME/.local/__product__

##  2.3 Dev Spaces Layout

```
<TopLocation /arena/>/
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
├── databox (optional)        ##  almost none, only for WSL app databox. Most Windows app data are in ${Windows}/${zpaces}/appdata
    └── tbd              ## 

```

