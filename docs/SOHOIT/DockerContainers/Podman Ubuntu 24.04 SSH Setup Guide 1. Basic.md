# Podman Ubuntu 24.04 SSH Setup Guide



This guide outlines the process for setting up a persistent Ubuntu 24.04 environment with SSH access using Podman, concluding with an image commit for future use.
## 1. Deploy the Container

Start the Ubuntu 24.04 container. This command maps host port **10022** to container port **22** and relaxes security constraints to allow the SSH daemon to run.

Bash

```
podman run -d -it --name labdev --hostname labdev -p 10022:22 -network slirp4netns:allow_host_loopback=true --security-opt seccomp=unconfined public.ecr.aws/ubuntu/ubuntu:24.04
```

## 2. Access and Environment Setup

Enter the container's interactive terminal to begin the configuration.

Bash

```
podman exec -it labdev /bin/bash
```

Update the package repositories and install the OpenSSH server along with essential networking and editing utilities.


Bash

```
apt update
```

Bash

```
apt install -y openssh-server curl iproute2 vim coreutils sudo
```


### 2.1 Cntlm proxy

If the host is using cntlm proxy to access the internet, one may need to set proxy inside the container 

```
unset proxy
expoert https_proxy=http://10.0.2.2:3128
expoert http_proxy=http://10.0.2.2:3128
```

** Where does 10.0.2.2 come from? see more detail in "Rootless Podman Gateway IP Access".



### 2.2 Ubuntu Apt Proxy Configuration 

1.  **Open the terminal** and create the file:
    ```bash
    sudo nano /etc/apt/apt.conf.d/99proxy
    ```

    or 

    ```bash
    sudo nano /etc/apt/apt.conf
    ```

2.  **Add the following lines**, replacing the placeholders with your actual proxy details:
    ```text
    Acquire::http::Proxy "http://proxy-server-address:port/";
    Acquire::https::Proxy "http://proxy-server-address:port/";
    ```
    * *Note: Even for HTTPS traffic, many proxies use `http://` as the protocol for the proxy address itself.*


## 3. Configure the SSH Service

Create the required directory for the SSH daemon to handle process IDs.

Bash

```
mkdir /var/run/sshd
```

Enable root login by modifying the SSH configuration file.

Bash

```
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
```

Set the root password to ensure you can authenticate during login.

Bash

```
echo 'root:your_secure_password' | chpasswd
```

## 4. Initialize and Manage the Service

Launch the SSH daemon manually to start listening for connections.

Bash

```
/usr/sbin/sshd
```

If you need to restart the service using the standard utility:

Bash

```
service ssh restart
```

If the service utility is unavailable, locate the process ID (PID) to restart manually:

Bash

```
ps aux | grep sshd
```

Bash

```
kill <PID_NUMBER>
```

Bash

```
/usr/sbin/sshd
```

## 5. Export the Image

Once the setup is verified, exit the container and commit your changes to a new image. This preserves the SSH configuration for future use.

Bash

```
podman commit labdev labdev:v1_sshd
```

---
@end