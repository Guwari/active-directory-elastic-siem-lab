# 02 — Network architecture

Use the same UTM shared virtual network for both guests. The recorded Mac interface is `bridge100`, gateway `192.168.64.1`, subnet `192.168.64.0/24`.

| Host | IPv4 / prefix | Gateway | DNS |
|---|---|---|---|
| DC01 | 192.168.64.12/24 | 192.168.64.1 | 192.168.64.12 after AD provisioning |
| WIN11-LAB02 | 192.168.64.10/24 | 192.168.64.1 | 192.168.64.12 |

Reserve these addresses or ensure UTM DHCP will not assign them to another guest. Shared networking is not a guarantee of isolation. Restrict lab service access with the host firewall.

## Debian

Find the interface with `ip -br address`, then configure the static address using the guest's existing NetworkManager, systemd-networkd or interfaces configuration. Do not run multiple network managers on one interface.

Set hostname `dc01` and map `192.168.64.12 dc01.lab.local dc01` in `/etc/hosts`. Do not map the AD FQDN only to 127.0.0.1. Before provisioning, use a working resolver for package installation; afterward point the DC resolver to its own AD DNS and configure Samba's DNS forwarder to a reachable resolver.

```bash
hostname -f
ip -br address
ip route
timedatectl status
```

Expected hostname: `dc01.lab.local`. Address and default route must match the table.

## Windows

Set the static values through Settings → Network & internet → adapter IPv4 properties. Keep DC01 as the sole domain DNS server; adding a public resolver as an alternate can cause intermittent domain discovery failure.

After Samba provisioning, run:

```powershell
ipconfig /all
Resolve-DnsName dc01.lab.local -Server 192.168.64.12
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.lab.local -Server 192.168.64.12
Resolve-DnsName -Type SRV _kerberos._tcp.lab.local -Server 192.168.64.12
Test-NetConnection 192.168.64.12 -Port 445
w32tm /query /status
```

Expect the DC address and SRV records pointing to DC01. Synchronize time before Kerberos testing.

## Host and ports

On the Mac, confirm bridge100 has `192.168.64.1` (`ifconfig bridge100`). Docker publishes Elasticsearch :9200 and Fleet :8220 to that interface; Kibana :5601 is loopback only.

Domain services require DNS TCP/UDP 53, Kerberos TCP/UDP 88, LDAP TCP/UDP 389, SMB TCP 445 and other AD services including RPC and time synchronization. Use Samba's full AD firewall requirements rather than opening only those four ports. Limit access to the lab subnet; do not disable the firewall as a general fix.

Next: [03 — Samba Active Directory](03-samba-active-directory.md).
