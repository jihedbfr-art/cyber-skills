# 12 — Active Directory & Windows Security

In most enterprises, owning Active Directory means owning the company. Attackers know it, and the paths from a phished laptop to Domain Admin are well-worn. This domain maps those paths and closes them.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [ad-enumeration-bloodhound](01-ad-enumeration-bloodhound/SKILL.md) | Map attack paths across the domain | ✅ |
| 02 | [kerberoasting](02-kerberoasting/SKILL.md) | Crack service account tickets, and rotate them away | ✅ |
| 03 | [as-rep-roasting](03-as-rep-roasting/SKILL.md) | Abuse accounts without pre-auth | ✅ |
| 04 | [ntlm-relay](04-ntlm-relay/SKILL.md) | Relay authentication, and enforce signing | ✅ |
| 05 | [dcsync-and-credential-dumping](05-dcsync-and-credential-dumping/SKILL.md) | Pull hashes, and detect it | ✅ |
| 06 | [delegation-abuse](06-delegation-abuse/SKILL.md) | Unconstrained and constrained delegation attacks | ✅ |
| 07 | [gpo-security](07-gpo-security/SKILL.md) | Group Policy as an attack and defence surface | ✅ |
| 08 | [tiered-admin-model](08-tiered-admin-model/SKILL.md) | Break the credential-theft chain by design | ✅ |
| 09 | [lsass-protection](09-lsass-protection/SKILL.md) | Credential Guard, LSA protection, hardening | ✅ |
| 10 | [ad-hardening-baseline](10-ad-hardening-baseline/SKILL.md) | The settings that shut most paths | ✅ |

This domain is complete (10/10). `ad-enumeration-bloodhound` first — you attack and defend the graph you can see; `ad-hardening-baseline` is the index that ties the defensive side together.
