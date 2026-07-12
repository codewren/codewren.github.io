# Home Lab VPN with WireGuard

**Goal: reach your corp Ubuntu server (behind symmetric NAT) from home.** All open source, no cloud services, no third-party identity provider — identity is managed locally with WireGuard key pairs.

## Architecture

```
 Home network (10.0.0.0/24)            Corp network (192.168.0.0/16, symmetric NAT)
┌─────────────────────────────┐   ┌───────────────────────────┐
│  Router (public IP)         │   │  Corp Ubuntu server       │
│  UDP 51820 → forwarded to   │◀──│  wg0: 172.16.99.2         │
│  Home Ubuntu server (hub)   │   │  (dials OUT, keepalive)   │
│  wg0: 172.16.99.1           │   └───────────────────────────┘
│  LAN IP: 10.0.0.50          │
└─────────────────────────────┘
```

Two directions to keep separate:

- **Tunnel establishment** must go corp → home, because symmetric NAT blocks inbound connections to the corp server, while your home router has a public IP and can port-forward. The corp server makes an outbound UDP connection and a `PersistentKeepalive` holds the NAT mapping open.
- **Actual usage** is home → corp: once the tunnel is up it's fully bidirectional, so from home you SSH/connect to the corp server at `172.16.99.2` anytime.

**VPN subnet choice:** your home LAN uses `10.0.*` and corp uses `192.168.*`, so the VPN must use neither. We use `172.16.99.0/24` from the third private range (172.16.0.0/12) — no overlap, no routing ambiguity.

**Identity model:** each machine generates its own Curve25519 key pair locally. The public key *is* the identity. You authorize a device by adding its public key to the hub config; you revoke it by removing the key. No accounts, no certificates, no external CA or IdP.

Adjust these values to your environment:

| Placeholder | Meaning | Value used below |
|---|---|---|
| `home.example.com` | Your home public IP or DDNS name | `203.0.113.10` |
| VPN subnet | Must not overlap home or corp LANs | `172.16.99.0/24` |
| Home LAN | Your home network | `10.0.0.0/24` |
| Home server LAN IP | Hub's address on home LAN | `10.0.0.50` |
| Corp LAN | Corp network | `192.168.0.0/16` |
| WireGuard port | UDP | `51820` |

---

## Step 1 — Install WireGuard on both servers

On **both** the home and corp Ubuntu servers:

```bash
sudo apt update
sudo apt install -y wireguard
```

That's it — WireGuard is in the mainline kernel; the package just adds the `wg` and `wg-quick` tools.

## Step 2 — Generate keys (local identity)

Run on **each** server separately, so private keys never leave the machine:

```bash
umask 077
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```

Note down each machine's **public** key. Private keys stay put.

Optionally add a pre-shared key per peer for extra (post-quantum-resistant) symmetric protection — generate it once and put the same value on both sides:

```bash
wg genpsk   # run once, copy the output to both configs
```

## Step 3 — Configure the home server (hub)

Create `/etc/wireguard/wg0.conf` on the **home** server:

```ini
[Interface]
Address    = 172.16.99.1/24
ListenPort = 51820
PrivateKey = <HOME_PRIVATE_KEY>

# --- Peer registry: one block per authorized device ---

[Peer]
# corp-server
PublicKey    = <CORP_PUBLIC_KEY>
PresharedKey = <PSK>            # optional, delete line if unused
AllowedIPs   = 172.16.99.2/32
```

`AllowedIPs = 172.16.99.2/32` means: only accept packets claiming to be 172.16.99.2 from this key, and route 172.16.99.2 to this peer. This is your access control — one line per identity.

**Optional — let other home devices (not just the hub) reach the corp server.** Enable forwarding on the hub by adding to `[Interface]`:

```ini
PostUp   = sysctl -w net.ipv4.ip_forward=1; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -A FORWARD -i wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT; iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
PostDown = iptables -D FORWARD -o wg0 -j ACCEPT; iptables -D FORWARD -i wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT; iptables -t nat -D POSTROUTING -o wg0 -j MASQUERADE
```

Then either add a static route on your home router (`172.16.99.0/24 via 10.0.0.50`) so every home device can use it, or add the route only on the machines that need it:

```bash
sudo ip route add 172.16.99.0/24 via 10.0.0.50   # on a home client
```

Skip this whole block if you'll always connect from the hub itself.

Lock down permissions and start:

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
sudo systemctl enable --now wg-quick@wg0
```

## Step 4 — Port-forward on the home router

In your router admin UI, forward:

- **UDP 51820** → home server's LAN IP `10.0.0.50`, port 51820

Give the home server a static LAN IP (DHCP reservation) so the forward doesn't break.

If your public IP is dynamic, run a self-hosted DDNS updater (e.g. `ddclient`, open source) or a small cron script against your own domain's DNS API — or just use the raw IP and update the corp config when it changes.

## Step 5 — Configure the corp server (client)

Create `/etc/wireguard/wg0.conf` on the **corp** server:

```ini
[Interface]
Address    = 172.16.99.2/24
PrivateKey = <CORP_PRIVATE_KEY>

[Peer]
# home-hub
PublicKey           = <HOME_PUBLIC_KEY>
PresharedKey        = <PSK>              # optional, must match hub
Endpoint            = home.example.com:51820
AllowedIPs          = 172.16.99.0/24
PersistentKeepalive = 25
```

- `Endpoint` — your home public IP/DDNS name. The corp server always initiates, so no inbound corp firewall rule is needed.
- `PersistentKeepalive = 25` — sends a packet every 25 s so the symmetric NAT keeps the mapping alive; without it the tunnel goes silent after idle timeout and you can't reach the corp server from home.
- `AllowedIPs = 172.16.99.0/24` — corp accepts/answers traffic from the whole VPN subnet (needed if other home devices connect through the hub per Step 3). Do **not** use `0.0.0.0/0`.

Start it:

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
sudo systemctl enable --now wg-quick@wg0
```

## Step 6 — Verify

On either side:

```bash
sudo wg show
```

Look for `latest handshake:` under the peer — a recent timestamp means the tunnel is up. Then, the main event — from **home**:

```bash
ping 172.16.99.2          # corp server over VPN
ssh <user>@172.16.99.2    # your actual use case
```

And sanity-check the reverse: from corp, `ping 172.16.99.1`.

Tip: add `172.16.99.2 corp` to `/etc/hosts` on your home machines so you can just `ssh corp`.

## Step 7 — Managing identities (add / revoke devices)

**Add a device** (laptop, phone, another server):

1. On the new device: generate a key pair (Step 2).
2. On the hub, append a `[Peer]` block with the new public key and the next free IP (`172.16.99.3/32`, `172.16.99.4/32`, …).
3. Apply without dropping existing sessions:

   ```bash
   sudo wg syncconf wg0 <(wg-quick strip wg0)
   ```

4. Configure the device like Step 5 (phones: use the official open-source WireGuard app; generate a QR code with `qrencode -t ansiutf8 < client.conf`). A roaming laptop with this config can reach the corp server from anywhere, via your home hub.

**Revoke a device:** delete its `[Peer]` block from the hub and run the same `syncconf` command. The key is instantly useless.

**Rotate keys:** generate a new pair on the device, update its `PrivateKey` and the hub's `PublicKey` for that peer, `syncconf` both sides.

Keep a comment line (`# hostname — owner — date added`) above each `[Peer]` block; the config file is your identity registry.

## Optional — reach the corp LAN beyond the server

If you also need other corp machines (e.g. `192.168.1.20`) through the corp server:

1. On the corp server, enable forwarding and NAT toward its LAN in `[Interface]`:

   ```ini
   PostUp   = sysctl -w net.ipv4.ip_forward=1; iptables -t nat -A POSTROUTING -s 172.16.99.0/24 -o <CORP_LAN_IFACE> -j MASQUERADE
   PostDown = iptables -t nat -D POSTROUTING -s 172.16.99.0/24 -o <CORP_LAN_IFACE> -j MASQUERADE
   ```

2. On the hub's corp `[Peer]` block, widen: `AllowedIPs = 172.16.99.2/32, 192.168.0.0/16` (or a narrower corp subnet).

Since home is `10.0.*` and corp is `192.168.*`, there's no subnet collision. But this exposes more of the corp network — see the caution below.

## Hardening checklist

- `chmod 600` on all `wg0.conf` files (contain private keys).
- Use a `PresharedKey` per peer.
- On the home server: `sudo ufw allow 51820/udp` and nothing else from the internet.
- Narrow `AllowedIPs` per peer to exactly what each identity should reach — it's enforced cryptographically per key.
- On the corp server, consider limiting inbound VPN traffic to what you need, e.g. `sudo ufw allow in on wg0 to any port 22 proto tcp`.
- WireGuard is silent to scanners: unauthenticated packets get no reply, so the open port is not discoverable.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| No handshake | Wrong port forward, wrong `Endpoint`, or key mismatch (check `sudo wg show` and `journalctl -u wg-quick@wg0`) |
| Handshake but no ping | `AllowedIPs` missing the destination, or `ip_forward` not enabled where forwarding is needed |
| Can't reach corp from home after idle | Missing `PersistentKeepalive` on the corp peer |
| Breaks occasionally | Home public IP changed — set up DDNS (Step 4) |
| Home clients (non-hub) can't reach corp | Missing route to `172.16.99.0/24` via `10.0.0.50`, or missing forward/MASQUERADE rules on hub (Step 3) |

@end

