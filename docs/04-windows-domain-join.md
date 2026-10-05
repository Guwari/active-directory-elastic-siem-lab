# 04 — Windows domain join

Use Windows 11 Pro, Enterprise or Education. Home edition cannot perform this conventional AD domain join.

## Prepare and join

1. Confirm the workstation is `WIN11-LAB02` at `192.168.64.10/24` with gateway `192.168.64.1` and DNS `192.168.64.12`.
2. Verify DC DNS/SRV resolution using the [network checks](02-network-architecture.md).
3. Open advanced system settings → Computer Name → Change, set the hostname if needed, and select Domain: `LAB.LOCAL`.
4. Enter a lab account authorized to join the domain at the Windows prompt. Do not capture that prompt for publication.
5. Restart and sign in using `LAB\\testuser` or `testuser@lab.local`.

Equivalent domain join from an elevated PowerShell prompt (credentials are prompted):

```powershell
Add-Computer -DomainName "LAB.LOCAL" -Credential (Get-Credential "LAB\Administrator") -Restart
```

## Verify

```powershell
whoami
$env:USERDOMAIN
Get-CimInstance Win32_ComputerSystem | Select-Object Name, Domain, PartOfDomain
nltest /dsgetdc:lab.local
klist
```

Expect `LAB\\testuser`, domain membership and DC01 discovery. A domain sign-in alone can be cached; validate current DC contact and inspect a newly generated DC log event. `klist` helps verify the Kerberos ticket context.

## Failure generation

Use a disposable account and first inspect lockout settings on the DC. A locked workstation can use cached credentials in some circumstances, so confirm each test actually produces a DC audit event. Keep DC connectivity during testing and stop after the planned attempts; count actual log events in Elastic.

Capture only sanitized system/domain settings and a domain-user identity check. Exclude passwords, personal account information and unrelated windows.

Next: [05 — Elastic Security deployment](05-elastic-security-deployment.md).
