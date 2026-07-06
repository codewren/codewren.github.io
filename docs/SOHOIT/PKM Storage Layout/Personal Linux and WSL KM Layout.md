
# 1. Common Layout

##  1.1 Top Location for Linux

- arena
- zpaces

## 1.2. $HOME Base Location in Linux

### 1.2.1 Only For Linux Desktop of  $HOME-based  Application
- Chrome
- Obsidian
-  rclone
-  electerm
-  jetbrains
#### Location name
 -  $HOME/zone/apphub
 -  $HOME/zone/appimages
 -  $HOME/zone/bin

### 1.2.2 Popular Dev Tool Chain Location
- nvm and nodejs
- uv and python
- rust (rustc and cargo)
 - claude code cli
 - gemini cli
 - opencode cli

#### Location Name
 -  $HOME/.local/__product__


# 2 Layout

```
<TopLocation>/
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
├── databox                ##  the app data
    └── obsidian           ## obsidian vault spaces

```



# 3. Layout of WSL

	```
	/mnt/c/zpaces/appdata/obsidian/codewren/codewren.github.io
	```