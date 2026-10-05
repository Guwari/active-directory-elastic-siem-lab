# 03 — Samba Active Directory

This sequence is for a **fresh Debian guest**. Do not reprovision an existing DC or overwrite its working configuration.

## Install and provision

After setting DC01's address, hostname and working DNS:

```bash
sudo apt update
sudo apt install samba krb5-user winbind smbclient dnsutils
sudo systemctl stop smbd nmbd winbind
sudo systemctl disable smbd nmbd winbind
```

If package prompts request a Kerberos realm, use `LAB.LOCAL`. Review the Debian/Samba documentation for the chosen release; separate packages or service defaults can vary.

On a fresh guest only, preserve the package's default `/etc/samba/smb.conf` as a local backup before provisioning. Do not publish backups.

```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.pre-ad
sudo samba-tool domain provision \
  --realm=LAB.LOCAL \
  --domain=LAB \
  --server-role=dc \
  --dns-backend=SAMBA_INTERNAL \
  --use-rfc2307
```

Enter a unique administrator password at the prompt; never use `--adminpass` with a real password in a saved command. Set a reachable DNS forwarder when prompted. Copy the generated Kerberos config after backing up the local existing file:

```bash
sudo cp /etc/krb5.conf /etc/krb5.conf.pre-ad
sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
sudo systemctl unmask samba-ad-dc
sudo systemctl enable --now samba-ad-dc
sudo testparm -s
```

Point the DC resolver to `192.168.64.12` and verify outbound resolution through Samba's configured forwarder. Avoid an AD DNS port conflict with another local DNS service.

## Verify AD and Kerberos

```bash
host -t SRV _ldap._tcp.dc._msdcs.lab.local 192.168.64.12
host -t SRV _kerberos._tcp.lab.local 192.168.64.12
kinit Administrator@LAB.LOCAL
klist
sudo samba-tool domain level show
sudo samba-tool user create testuser
sudo samba-tool domain passwordsettings show
```

Passwords are entered interactively. Confirm a Kerberos ticket and create the disposable `LAB\\testuser` account. Record lockout policy before failure tests.

## Enable authentication auditing

Merge [the audit fragment](../configs/samba-audit.example.conf) into the existing `[global]` section. Preserve generated domain settings. Validate and restart:

```bash
sudo testparm -s
sudo systemctl restart samba-ad-dc
sudo tail -n 50 /var/log/samba/log.samba
```

The lab setting is `log level = 1 auth_audit:3`. Generate a known authentication request and verify the outcome reaches the expected file. A failed password should produce `NT_STATUS_WRONG_PASSWORD`; inspect the event to establish its actual authentication mechanism. Do not replace this with SMB file-operation `full_audit`.

Restrict log permissions and configure rotation. Verify Elastic Agent can still read the current file after rotation; never make logs world-readable.

Reference: [Samba smb.conf audit classes](https://www.samba.org/samba/docs/current/man-html/smb.conf.5).

Next: [04 — Windows domain join](04-windows-domain-join.md).
