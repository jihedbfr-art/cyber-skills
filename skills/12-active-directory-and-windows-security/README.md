# 12 — Active Directory & Windows Security

In most enterprises, owning Active Directory means owning the company. Attackers know it, and the paths from a phished laptop to Domain Admin are well-worn. This domain maps those paths and closes them.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | ad-enumeration-bloodhound | Map attack paths across the domain | TODO |
| 02 | kerberoasting | Crack service account tickets, and rotate them away | TODO |
| 03 | as-rep-roasting | Abuse accounts without pre-auth | TODO |
| 04 | ntlm-relay | Relay authentication, and enforce signing | TODO |
| 05 | dcsync-and-credential-dumping | Pull hashes, and detect it | TODO |
| 06 | delegation-abuse | Unconstrained and constrained delegation attacks | TODO |
| 07 | gpo-security | Group Policy as an attack and defence surface | TODO |
| 08 | tiered-admin-model | Break the credential-theft chain by design | TODO |
| 09 | lsass-protection | Credential Guard, LSA protection, hardening | TODO |
| 10 | ad-hardening-baseline | The settings that shut most paths | TODO |

TODO: domain scaffolded. `ad-enumeration-bloodhound` first — you attack and defend the graph you can see.
