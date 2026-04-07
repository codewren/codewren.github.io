# Podman Ubuntu 24.04 SSH Setup Guide 4. Specific Dev Project

## Daft.ai Source Code Insight

Bash  create_space.sh
```bash
#!/bin/bash

# --- 1. Configuration ---
# Project-specific (Will be created)
export CNTNER_HOME="${CNTNER_HOME:-$HOME/dev_spaces/project_alpha}"
# Shared caches (Must already exist)
export CNTNER_COMMON="${CNTNER_COMMON:-$HOME/dev_container_common}"

echo "Checking Dev Environment..."

# --- 2. CNTNER_HOME Validation (Aggressive Creation) ---
# If the root doesn't exist, create it.
if [ ! -d "$CNTNER_HOME" ]; then
    echo "[INFO] Creating project home: $CNTNER_HOME"
    mkdir -p "$CNTNER_HOME"
fi

# Define the required subdirectories for project-specific data
PROJECT_SUBS=("devspace" "vscode_server")

for sub in "${PROJECT_SUBS[@]}"; do
    SUB_PATH="$CNTNER_HOME/$sub"
    if [ ! -d "$SUB_PATH" ]; then
        echo "[INFO] Creating missing project subdirectory: $sub"
        mkdir -p "$SUB_PATH"
    else
        echo "[OK] Project subdirectory exists: $sub"
    fi
done

# --- 3. Handle Common Shared Dirs (Warning only) ---
# We check the root of the common directory
if [ ! -d "$CNTNER_COMMON" ]; then
    echo "############################################################"
    echo "WARNING: CNTNER_COMMON directory NOT FOUND!"
    echo "Expected at: $CNTNER_COMMON"
    echo "If this is an external drive, please mount it."
    echo "The container will likely fail or be empty of caches."
    echo "############################################################"
    echo "ERROR: Shared cache missing. Aborting."
    exit 1
else
    echo "[OK] Common cache found: $CNTNER_COMMON"
    
    # Verify sub-directories exist inside common to avoid mount-point masking
    for dir in cargo_home rustup_home maven_cache sbt_cache coursier_cache uv_cache; do
        if [ ! -d "$CNTNER_COMMON/$dir" ]; then
             echo "[WARN] Sub-cache missing: $CNTNER_COMMON/$dir (should be create by init container)"
        fi
    done
     echo "[OK] Common cache validation completed"
fi

echo "Setup check complete."
```


```bash

PODMAN_OPTS=( 
   -d -it 
  --name daftai  --hostname daftai --add-host daftai:127.0.0.1  --network host 
  --user devuser 

  # Identity mappings (devuser=1001)
  --uidmap 1001:0:1 
  --uidmap 0:1:1001 
  --uidmap 1002:1002:64534 
  --gidmap 1001:0:1 
  --gidmap 0:1:1001 
  --gidmap 1002:1002:64534 

  # Security overrides
  --cap-add=SYS_PTRACE 
  --security-opt seccomp=unconfined 
  --security-opt label=disable 

  # --- Common Cache Mounts (Shared) --- 
  -v "$CNTNER_COMMON/cargo_home:/home/devuser/.cargo:U" 
  -v "$CNTNER_COMMON/rustup_home:/home/devuser/.rustup:U" 
  -v "$CNTNER_COMMON/maven_cache:/home/devuser/.m2:U" 
  -v "$CNTNER_COMMON/sbt_cache:/home/devuser/.sbt:U" 
  -v "$CNTNER_COMMON/coursier_cache:/home/devuser/.cache/coursier:U" 
  -v "$CNTNER_COMMON/uv_cache:/home/devuser/.cache/uv:U" 

  # --- Project Specific Mounts ---  \
  -v "$CNTNER_HOME/devspace:/home/devuser/devspace:U" 
  -v "$CNTNER_HOME/vscode_server:/home/devuser/.vscode-server:U" 
)
podman run "${PODMAN_OPTS[@]}"  labdev:v3_devtool_ext

```

###  Behind Company's Proxy

```bash

PODMAN_OPTS=( 
   -d -it 
  --name daftai  --hostname daftai --add-host daftai:127.0.0.1  
  -p 10122:22
  --network slirp4netns:allow_host_loopback=true 
  --user devuser 

  # Identity mappings (devuser=1001)
  --uidmap 1001:0:1 
  --uidmap 0:1:1001 
  --uidmap 1002:1002:64534 
  --gidmap 1001:0:1 
  --gidmap 0:1:1001 
  --gidmap 1002:1002:64534 

  # Security overrides
  --cap-add=SYS_PTRACE 
  --security-opt seccomp=unconfined 
  --security-opt label=disable 

  # --- Common Cache Mounts (Shared) --- 
  -v "$CNTNER_COMMON/cargo_home:/home/devuser/.cargo:U" 
  -v "$CNTNER_COMMON/rustup_home:/home/devuser/.rustup:U" 
  -v "$CNTNER_COMMON/maven_cache:/home/devuser/.m2:U" 
  -v "$CNTNER_COMMON/sbt_cache:/home/devuser/.sbt:U" 
  -v "$CNTNER_COMMON/coursier_cache:/home/devuser/.cache/coursier:U" 
  -v "$CNTNER_COMMON/uv_cache:/home/devuser/.cache/uv:U" 

  # --- Project Specific Mounts ---  \
  -v "$CNTNER_HOME/devspace:/home/devuser/devspace:U" 
  -v "$CNTNER_HOME/vscode_server:/home/devuser/.vscode-server:U" 
)
podman run "${PODMAN_OPTS[@]}"  labdev:v3_devtool_ext

```
----
