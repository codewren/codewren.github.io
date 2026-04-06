# Rootless Podman Gateway IP Access

 One may have no `ip` and no `python` with a bare-bones image (like a minimal Alpine or a slimmed-down Debian/RedHat) but you almost certainly have `cat` and `grep`.

Here is the most direct way to get that IP using only built-in kernel interfaces.

### 1. The "Zero-Dependency" Gateway Check
Run this **inside the container**. It reads the hex-encoded default gateway directly from the kernel routing table and converts it to decimal using a shell-only loop.

**Run this one-liner inside the container:**
```bash
read _ dest gw _ < <(grep "00000000" /proc/net/route); printf "%d.%d.%d.%d\n" $((0x${gw:6:2})) $((0x${gw:4:2})) $((0x${gw:2:2})) $((0x${gw:0:2}))
```

* **How it works:** It finds the `00000000` (default route) line in `/proc/net/route`. The 3rd column is the Gateway in Little-Endian Hex. The `printf` part swaps the byte order and converts it to a human-readable IP.
* **Result:** This will output `10.0.2.2` in a standard Podman rootless setup.

---

### 2. Verify why 10.0.2.2 is the Gateway
In rootless Podman, **slirp4netns** simulates an entire TCP/IP stack in user space. It mimics a traditional router setup:
* **10.0.2.1:** The Virtual Router/DHCP server.
* **10.0.2.2:** The **Virtual Gateway** (this is actually the slirp4netns process on your host).
* **10.0.2.3:** Virtual DNS server.
* **10.0.2.15 (to 100):** Your container's IP.



---

### 3. Why `allow_host_loopback=true` is necessary
Normally, for security, slirp4netns blocks the container from reaching `127.0.0.1` on the host to prevent a container from accidentally attacking services on the host that aren't meant to be public.

When you use `--network slirp4netns:allow_host_loopback=true`:
1.  The container sends a packet to `10.0.2.2`.
2.  The `slirp4netns` process sees the packet.
3.  Because the flag is enabled, it permits the translation of that packet to the **host's loopback interface**.
4.  Your **Cntlm** (listening on `127.0.0.1` on the host) sees the connection as coming from the host itself.

---

### 4. Direct Action (Host One-Liner)
If you want to verify your container can talk to your Cntlm proxy (on port 3128) right now from the **host command line**:

```bash
podman run --rm --network slirp4netns:allow_host_loopback=true alpine sh -c 'printf "" > /dev/tcp/10.0.2.2/3128 && echo "SUCCESS: Host proxy is reachable"'
```
