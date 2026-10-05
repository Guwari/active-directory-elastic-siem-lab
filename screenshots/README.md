# Screenshot evidence

This directory intentionally starts with guidance only. Add authentic screenshots from the lab after redaction; do not manufacture events or substitute documentation diagrams for observed results.

| Suggested file | Show | Keep visible |
|---|---|---|
| 01-domain-membership.png | Windows system/domain settings | WIN11-LAB02 and LAB.LOCAL |
| 02-domain-user.png | Domain identity / live DC check | Lab username and domain |
| 03-fleet-dc01-healthy.png | Fleet Agents | DC01, Healthy, policy |
| 04-samba-discover.png | Discover dataset results | samba.auth, timestamps, source |
| 05-wrong-password-events.png | Filtered authentication failures | KQL and NT_STATUS_WRONG_PASSWORD |
| 06-threshold-rule.png | Rule definition | Name, threshold 3, no grouping |
| 07-rule-schedule.png | Rule schedule/execution | 1-minute interval and look-back |
| 08-security-alert.png | Actual generated alert after validation | Rule name, timestamp and count |

The original conversation does not confirm a fired alert. Add the last image only after completing [validation](../docs/09-testing-and-validation.md).

## Redaction checklist

- Rotate exposed setup credentials before capturing fresh evidence.
- Exclude enrollment commands, Fleet service/enrollment tokens, API keys, passwords, encryption keys, cookies and private keys.
- Crop personal account menus, unrelated tabs, email addresses, desktop notifications and other projects.
- Use solid opaque redaction applied permanently to the exported raster image; blur and reversible annotations are insufficient.
- Inspect the final exported file at full size and check metadata before committing.
- Keep originals outside the repository. Raw screenshot folders are ignored, but .gitignore cannot detect secrets embedded in pixels.
- Retain the documented lab IP addresses and names when safe; they explain the scenario.

After review, add relative image links to the root README with descriptive alt text and captions stating exactly what each screenshot demonstrates. Do not claim the screenshot proves an attack, a distinct number of password guesses or a protocol unless its evidence establishes that claim.
