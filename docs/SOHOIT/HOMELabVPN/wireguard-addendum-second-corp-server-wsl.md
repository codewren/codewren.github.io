# Addendum: Second Corp Server on WSL Ubuntu

Delta guide on top of **Home Lab VPN with WireGuard**. Only changes and additions are listed — everything not mentioned stays as in the base guide.

## Updated architecture

The second corp machine is Ubuntu running inside **WSL2** on a Windows host. It joins as another spoke: it also dials out to the home hub, so the extra NAT layer (WSL2's own NAT behind Windows, behind the corp symmetric NAT) changes nothing — outbound UDP + keepalive still works.

```
 Corp network (192.168.0.0/16)              Home
┌──────────────────────────────┐
│ Corp Ubuntu (native)         │──┐
│   wg0: 172.16.99.2           │  │      ┌────────────────────┐
│                              │  ├─────▶│ Home hub           │
│ Windows host                 │  │      │ wg0: 172.16.99.1   │
│ └─ WSL2 Ubuntu               │──┘      └────────────────────┘
│      wg0: 172.16.99.3        │
└──────────────────────────────┘
```

New address assignment: **corp-wsl = `172.16.99.3`**.

Note: the two corp servers can also reach *each other* — but only via the hub (spoke ↔ hub ↔ spoke), since each peer's config only knows the hub.

## Change to Step 3 — hub config

Append a second `[Peer]` block on the home server's `/etc/wireguard/wg0.conf`:

```ini
[Peer]
# corp-wsl — Windows host WSL2 Ubuntu
PublicKey    = <CORP_WSL_PUBLIC_KEY>
PresharedKey = <PSK2>            # generate a NEW psk, don't reuse corp-server's
AllowedIPs   = 172.16.99.3/32
```

Apply without dropping the existing tunnel:

```bash
sudo wg syncconf wg0 <(wg-quick strip wg0)
```

## New Step 5b — configure the WSL Ubuntu server

### Prerequisites (differences from a native server)

1. **Must be WSL2**, not WSL1 (WSL1 has no Linux kernel, no WireGuard). Check in PowerShell:

   ```powershell
   wsl -l -v        # VERSION column must be 2
   ```

   The stock WSL2 kernel (5.10+) already includes the WireGuard module.

2. **Enable systemd** inside WSL so `wg-quick@wg0` can run as a service. In WSL, create/edit `/etc/wsl.conf`:

   ```ini
   [boot]
   systemd=true
   ```

   Then from PowerShell: `wsl --shutdown`, and reopen WSL.

### Install, keys, config

Steps 1–2 of the base guide are identical (`apt install wireguard`, generate keys inside WSL). The config is the same as base Step 5 with only the address changed:

```ini
[Interface]
Address    = 172.16.99.3/24
PrivateKey = <CORP_WSL_PRIVATE_KEY>

[Peer]
# home-hub
PublicKey           = <HOME_PUBLIC_KEY>
PresharedKey        = <PSK2>
Endpoint            = home.example.com:51820
AllowedIPs          = 172.16.99.0/24
PersistentKeepalive = 25
```

Start it as usual:

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
sudo systemctl enable --now wg-quick@wg0
```

### Keep the tunnel alive (WSL-specific, important)

Unlike a native server, **WSL2 shuts down when its last process exits, and the tunnel dies with it.** To make the VPN survive without an open terminal:

1. **Auto-start WSL at Windows logon.** Task Scheduler → Create Task:
   - Trigger: *At log on*; check *Run whether user is logged on or not* if desired.
   - Action: `wsl.exe -d Ubuntu -- sleep infinity` (keeps the WSL VM running; systemd then keeps wg0 up).
   - Uncheck *Stop the task if it runs longer than…* under Settings.

2. **Stop Windows from sleeping** (sleep suspends WSL and the tunnel): Settings → System → Power → *Sleep: Never* when plugged in.

3. Optional, Windows 11: in `%UserProfile%\.wslconfig` disable the idle shutdown timer:

   ```ini
   [wsl2]
   vmIdleTimeout=-1
   ```

## Change to Step 6 — verify

From home, additionally:

```bash
ping 172.16.99.3           # WSL server
ssh <user>@172.16.99.3
```

For SSH into WSL, make sure `openssh-server` is installed and running *inside WSL* (`sudo apt install -y openssh-server && sudo systemctl enable --now ssh`). You land in the WSL Ubuntu, not Windows.

Optional `/etc/hosts` addition on home machines: `172.16.99.3 corp-wsl`.

## Optional — reach the Windows host itself

Traffic to `172.16.99.3` terminates inside WSL. To reach a service on the Windows host (e.g. RDP), forward a port from WSL to the host. Inside WSL:

```bash
# Windows host as seen from WSL = default gateway
HOST_IP=$(ip route show default | awk '{print $3}')
sudo apt install -y socat
socat TCP-LISTEN:13389,fork TCP:$HOST_IP:3389 &   # RDP example
```

Then from home: connect to `172.16.99.3:13389`. Also allow the port in Windows Defender Firewall for the WSL (vEthernet) interface.

## Additional troubleshooting (WSL-specific)

| Symptom | Likely cause |
|---|---|
| Tunnel dies when terminal closes | WSL VM shut down — set up the keep-alive scheduled task (Step 5b) |
| Tunnel dead after laptop wake | Windows sleep suspended WSL; `sudo systemctl restart wg-quick@wg0` in WSL, and disable sleep |
| `systemctl` errors ("not booted with systemd") | `/etc/wsl.conf` missing `systemd=true`, or no `wsl --shutdown` after editing |
| Handshake fails only from WSL | Corp firewall may treat WSL's NAT'd traffic differently; also check Windows Defender Firewall outbound rules |
| Ping works, SSH/transfers stall | MTU too high through double NAT — add `MTU = 1360` to the WSL `[Interface]` section |
| Clock-related handshake rejection after resume | WSL clock drift — `sudo hwclock -s` or restart WireGuard |

Everything else — identity management, revocation (`syncconf` after deleting the peer block), hardening — works exactly as in the base guide; the WSL server is just one more public key in the hub's registry.
