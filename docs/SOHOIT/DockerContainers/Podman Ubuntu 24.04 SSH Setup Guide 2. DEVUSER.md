# Podman Ubuntu 24.04 SSH Setup  DEVUSER


## Optional in WSL 

### 1. Create containers
```
podman run -d -it --name labWSL  --hostname labWSL --add-host labWSL:127.0.0.1  --network host   labdev:v1_sshd
```
__Note__ sshd port is 10122

### 2. Access and Environment Setup

Enter the container's interactive terminal to begin the configuration.

Bash

```
podman exec -it labWSL /bin/bash
```


###  3. Allow root login via SSH (Disabled by default) and set the sshd port
```
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/#Port 22/Port 10122/' /etc/ssh/sshd_config
```

###  restart sshd service

Bash

```
service ssh restart
```


### 4. Connecting from your Host
Now that the service is running inside the container and listening on port 22, you can connect using the host port (2222) you mapped earlier:

Bash
```
ssh root@localhost -p 10122
```


##  Create a dev user for daily activitiy (Avoid using root)


Setting up a normal user is a much safer practice than using root for everything. On Ubuntu, this involves creating the user, assigning a shell (like Bash), and adding them to the sudo group.

Since you've already unminimized the container, the sudo package should be available. If not, make sure to install it first.

### 1. Create the User
Run these commands inside your container (via podman exec):

Bash
```

# Install sudo if it's missing
apt update && apt install sudo -y

# Create a new user (let's call him 'devuser')
adduser devuser

```


### 2. Add to Sudoers
In Ubuntu, any user in the sudo group automatically gets administrator privileges.

Bash

```
# Add the user to the sudo group
usermod -aG sudo devuser
```

### 3. Verify the Setup
You can test if the user can actually use sudo without leaving your current root session:

Bash

```
su - devuser -c "sudo -S whoami"
```

If it returns root, your user successfully leveled up!

-----
### 4. Logging in via SSH
Now, instead of using root, you can SSH into your container using the normal user on the port you mapped (2222):

Bash
```
ssh devuser@localhost -p 10022
```

### 5. Export the Image

Once the setup is verified, exit the container and commit your changes to a new image. This preserves the SSH configuration for future use.

Bash

```
podman commit labWSL labdev:v2_devuser
```


## Dev Engineer Toolchain  setup

### 1. Create Container (on WSL/Linux Server)

- On WSL

```
podman run -d -it --name labWSL  --hostname labWSL --add-host labWSL:127.0.0.1  --network host   labdev:v2_devuser
```

- On Linux Server

```
podman run -d -it --name labLinux  --hostname labLinux --add-host labWSL:127.0.0.1  -p 10122:22    labdev:v2_devuser
```


### 2. Start open sshd server
```
podman exec -it labWSL /bin/bash
```

```
service ssh restart
```


@end