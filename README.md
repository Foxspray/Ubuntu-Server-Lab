# Ubuntu Server Home Lab

A practical home lab built to develop Linux administration, networking and troubleshooting skills using **Ubuntu Server**, a **Windows 10 client** and **VirtualBox**.

The lab covers remote administration with SSH, Linux users and permissions, Apache2, DHCP, DNS with BIND9, and basic troubleshooting.

---

## Lab overview

| Component | Configuration |
|---|---|
| Server | Ubuntu Server 26.04 LTS |
| Client | Windows 10 |
| Hypervisor | VirtualBox |
| Internal network | `10.0.0.0/24` |
| Ubuntu Server | `10.0.0.1` |
| DHCP pool | `10.0.0.101 - 10.0.0.200` |
| DNS zone | `ubuntu.lab` |
| Test DNS record | `server.ubuntu.lab -> 10.0.0.1` |

### Network flow

```text
Windows 10 Client
      |
      |  DHCP / DNS
      v
Ubuntu Server - 10.0.0.1
      |
      +-- SSH
      +-- Apache2
      +-- ISC DHCP Server
      +-- BIND9 DNS
```

---

## SSH administration

SSH was used to remotely administer the Ubuntu Server.

During testing, the Windows SSH client reported:

```text
REMOTE HOST IDENTIFICATION HAS CHANGED!
```

The VM had been recreated, so the saved host key in `known_hosts` no longer matched the server.

After verifying that the change was expected, the old key was removed:

```bash
ssh-keygen -R 127.0.0.1
```

The connection was then established again and the new fingerprint was accepted.

![SSH host key troubleshooting](ssh-host-key-troubleshooting.png)

---

## Users, groups and permissions

Multiple users and a Linux group were created to practice access control.

The group `newgroup1234` contained authorized users such as `adam` and `testuser`.

A shared directory was configured with group ownership and the **setgid** bit:

```bash
sudo mkdir -p /srv/newgroupfolder
sudo chown :newgroup1234 /srv/newgroupfolder
sudo chmod 2770 /srv/newgroupfolder
```

This allows members of the group to work in the directory while users outside the group are denied access. New files inherit the directory's group.

![Group permissions access test](group-permissions-access-test.png)

### Troubleshooting group membership

After adding `adam` to the group, the already-open SSH session still used the previous supplementary group list.

Reconnecting refreshed the user's group membership and the new permissions became effective.

![Group membership session troubleshooting](group-membership-session-troubleshooting.png)

---

## Apache2 web server

Apache2 was installed and managed as a systemd service.

Example service check:

```bash
sudo systemctl status apache2
```

The Ubuntu VM was behind VirtualBox NAT, so HTTP traffic was forwarded from the Windows host:

```text
127.0.0.1:8080  ->  Ubuntu port 80
```

The default Apache page was successfully accessed from Windows.

![Apache2 default page](apache2-default-page.png)

The default website was later replaced with a custom test page stored in `/var/www/html/`.

![Custom Apache2 page](apache2-custom-page.png)

---

## DHCP server

ISC DHCP Server was configured on the internal VirtualBox network.

The server distributes addresses from:

```text
10.0.0.101 - 10.0.0.200
```

for the `10.0.0.0/24` network, with `10.0.0.1` used as the gateway.

![DHCP server configuration](dhcp-server-configuration.png)

The server logs showed the standard DHCP **DORA** process:

```text
DHCPDISCOVER
DHCPOFFER
DHCPREQUEST
DHCPACK
```

![DHCP DORA process](dhcp-dora-process.png)

The Windows client successfully obtained its network configuration from the Ubuntu DHCP server.

![Windows DHCP lease](dhcp-client-lease-windows.png)

---

## DNS server with BIND9

BIND9 was configured as the DNS server for the internal network.

A forward lookup zone was created:

```text
ubuntu.lab
```

with an A record:

```text
server.ubuntu.lab -> 10.0.0.1
```

The zone was validated using:

```bash
sudo named-checkzone ubuntu.lab /etc/bind/db.ubuntu.lab
```

Local resolution was tested directly against the DNS service:

```bash
dig @127.0.0.1 server.ubuntu.lab
```

![Local DNS resolution](dns-local-resolution-test.png)

The Windows client was then tested against the Ubuntu DNS server directly:

```cmd
nslookup server.ubuntu.lab 10.0.0.1
```

![Explicit DNS server test](dns-explicit-server-test.png)

Finally, DHCP was configured to provide `10.0.0.1` as the DNS server automatically. After renewing the lease, the client could resolve the hostname without manually specifying a DNS server:

```cmd
nslookup server.ubuntu.lab
```

![Windows DNS resolution](dns-client-resolution-windows.png)

---

## Troubleshooting performed

During the lab I encountered and resolved several practical issues:

- stale SSH host key after recreating the virtual machine
- supplementary Linux group membership not updating in an existing SSH session
- DHCP communication between Ubuntu and a Windows client on an internal VirtualBox network
- DNS resolution from both the server and client side
- differences between older Ubuntu tutorials and the current BIND9 package layout
- VirtualBox NAT port forwarding for SSH and HTTP access

---

## What I practiced

- Ubuntu Server administration
- SSH remote access
- Linux users, groups and file permissions
- `systemd` and `systemctl`
- Apache2 basics
- DHCP configuration and DORA
- BIND9 DNS zones and A records
- DHCP and DNS integration
- VirtualBox NAT and internal networking
- port forwarding
- Linux/Windows troubleshooting

---

## Planned improvement

A reverse DNS zone and PTR record can be added later so that `10.0.0.1` resolves back to `server.ubuntu.lab`.
