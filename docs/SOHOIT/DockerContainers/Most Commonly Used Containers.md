# Most Commonly Used Containers

## Ubuntu 24 in WSL,  Podman created

### 1. Create containers
```
podman run -d -it --name cntlcenter  --hostname cntlcenter --add-host cntlcenter:127.0.0.1  --network host   docker.io/library/ubuntu:24.04
```

### 2. Post Set up 
#### 2.1  Unminimize Ubuntu

Ubuntu 24.04 container is "minimized" is that official Ubuntu cloud and docker images are stripped of man pages, documentation, and various interactive tools to keep the image size small.

To "unminimize" it, Ubuntu provides a built-in script precisely for this purpose.

Step-by-Step Execution
If your container is currently running in the background (using the -it or sleep infinity trick we discussed), follow these steps:

- Enter the container:

Bash

```
$ podman exec -it cntlcenter  /bin/bash
```

Update the package list:

Bash

```
apt update
```
Run the hydration script:

Bash
```
unminimize
```

Note: This will ask for confirmation (y/n). It will then download and reinstall the standard "human-friendly" packages, which takes a few minutes.

### 2.2 Enable SSHD server

To enable the SSH service in an Ubuntu container, you need to perform three main steps: install the server, configure it to allow logins, and start the daemon.

Since containers don't use systemd by default, you can't always rely on systemctl. Instead, we use the direct service command.

#### 1. Install and Configure SSH
Run these commands inside your container (podman exec -it <name> bash):

Bash
```
# Update and install the server
apt update && apt install openssh-server -y

# Create the run directory (required for the daemon to start)
mkdir /var/run/sshd

# Set a root password (so you can actually log in)
echo 'root:changeit' | chpasswd

# Optional: Allow root login via SSH (Disabled by default)
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/#Port 22/Port 10022/' /etc/ssh/sshd_config
```
#### 2. Start the Service
In a standard Linux OS, you'd use systemctl enable, but in a container, you simply start the binary or use the service helper:

Bash

```
/usr/sbin/sshd 
```



#### 3. Connecting from your Host
Now that the service is running inside the container and listening on port 22, you can connect using the host port (2222) you mapped earlier:

Bash
```
ssh root@localhost -p 10022
```

### 2.3. Create a dev user for daily activitiy (Avoid using root)

Setting up a normal user is a much safer practice than using root for everything. On Ubuntu, this involves creating the user, assigning a shell (like Bash), and adding them to the sudo group.

Since you've already unminimized the container, the sudo package should be available. If not, make sure to install it first.

#### 1. Create the User
Run these commands inside your container (via podman exec):

Bash
```

# Install sudo if it's missing
apt update && apt install sudo -y

# Create a new user (let's call him 'devuser')

adduser devuser

```


#### 2. Add to Sudoers
In Ubuntu, any user in the sudo group automatically gets administrator privileges.

Bash

```
# Add the user to the sudo group
usermod -aG sudo devuser
```

#### 3. Verify the Setup
You can test if the user can actually use sudo without leaving your current root session:

Bash

```
su - devuser -c "sudo -S whoami"
```

If it returns root, your user successfully leveled up!

#### 4. Logging in via SSH
Now, instead of using root, you can SSH into your container using the normal user on the port you mapped (2222):

Bash
```
ssh devuser@localhost -p 10022
```

---@end
