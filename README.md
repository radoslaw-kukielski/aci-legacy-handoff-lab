# ACI Legacy Handoff Lab

Build a Cisco ACI fabric step by step as the target of a migration from a legacy EVPN-VXLAN fabric (for example Juniper), and understand why every object exists.

> **Not affiliated with or endorsed by Cisco.** All names, VLANs and IP addresses in this repository are invented lab data. This is a learning lab, not a validated production design.

**Read the guide:** [docs/index.html](docs/index.html) (or the GitHub Pages link of this repository, once Pages is enabled)

## Status

| Part | State |
|---|---|
| Overview, A. Access policies, B. Tenant, C. Final checks | Re-run and checked with screenshots on APIC 6.1(4h), DevNet simulator, November 2026 |
| D. Faults, E. Scale, F. Lab tests T1-T9, G, H | Written from an earlier run on the same version. Re-check in progress: GUI labels may differ |
| `lab-data/` restore files | Exported after A to C. Restore to be re-tested in the next reservation |

## What you build

Phase 1 of a migration: ACI works as an L2-only fabric behind one vPC handoff toward the legacy fabric. The gateways stay on the legacy router.

- **Access policies (A1-A9):** once per fabric the vPC pair and MCP global (A1, A2); then per handoff the VLAN pool, physical domain, AAEP, a vPC policy group with its LACP and MCP interface policies, and the interface and switch profiles.
- **Tenant (B1-B14):** tenant LAB, VRF-PROD, three L2 bridge domains and three EPGs (VLAN 310, 320, 330) on the same vPC, filters and contracts for a three-tier app.
- **Checks and faults (C, D):** how to prove each link, and what F0467 means when one is missing.
- **Lab tests (T1-T9):** API Inspector, permit-all options (preferred group, vzAny), an L3Out to a firewall, the first gateway move, save and restore.

The guide follows one idea through every step:

```
Infra side (each object points to the one before it, built from the pool upward)
  switch profile -> interface profile -> vPC policy group -> AAEP -> physical domain -> VLAN pool

Tenant side (each object contains the next, built from the tenant downward)
  tenant -> VRF -> bridge domain -> application profile -> EPG -> domain + static port
```

Both sides meet at the physical domain. Every step explains what the object is for, which field values matter and why, what it points to, its DN, and how to check it. A "What is what" section maps every ACI object to classic switch terms (VLAN, SVI, ACL, port-channel, vPC domain).

## What this lab is not

The DevNet ACI simulator has no data plane. Ports stay down and no traffic is forwarded. The tests prove objects, relations, faults and API behaviour, not pings.

## Prerequisites

- A free Cisco DevNet account and a reservation of the ACI simulator sandbox.
- The VPN client and credentials from the reservation e-mail.
- A web browser, and an SSH client for the save and restore step.
- Basic knowledge of VLANs, LACP, vPC and IP routing.

## How to use the guide

1. Open the guide and read **The chain** and **What is what** first.
2. Work through A, B and C in order. Each step ends with a **Check**: pass it before you move on.
3. Use D when something goes red, E for the scale question, and the T tests when the base build is done.
4. Before the reservation ends, save the build (T9).

## Lab data at a glance

| Object | Name / value |
|---|---|
| VLAN pool | VP-LEGACY-HANDOFF, static, 310-340 |
| Physical domain | PD-LEGACY |
| AAEP | AAEP-LEGACY |
| LACP / MCP policies | LACP-ACTIVE, MCP-ON |
| vPC policy group | IPG-VPC-LEGACY, port 1/45 on leaves 101 and 102 |
| Interface / switch profile | IFP-LEAF101-102 (selector LEGACY-HANDOFF), SWP-LEAF101-102 |
| Tenant / VRF / AP | LAB / VRF-PROD / AP-NETWORK |
| BDs / EPGs | BD-WEB, BD-APP, BD-DB / EPG-WEB 310, EPG-APP 320, EPG-DB 330 |
| Contracts | CON-WEB-TO-APP (TCP 8443), CON-APP-TO-DB (TCP 1433) |
| Later tests | EPG-BACKUP 340, CON-PERMIT-ANY, L3OUT-FW on 1/46 (VLAN 350, 172.16.1.0/29) |

## Restore files

`lab-data/` holds the build after sections A to C, exported with config only and stored as plain objects ready for a REST POST:

| File | Contains |
|---|---|
| `lab-infra.json` | The lab's access policies only (pool, AAEP, LACP, MCP, vPC policy group, profiles) |
| `lab-domains.json` | PD-LEGACY and its link to the pool |
| `lab-tenant-LAB.json` | Tenant LAB with VRF, BDs, EPGs, static ports, filters and contracts |

Restore in this order: infra, domains, tenant. The MCP key is never exported, so set MCP global by hand afterwards (step A2). Details and commands are in step T9.

## Tested with

APIC 6.1(4h) on the Cisco DevNet ACI simulator (one APIC, spine 201, leaves 101 and 102), November 2026.

## Repository layout

```
README.md          this file
LICENSE            CC BY 4.0 for the guide text, diagrams and README
docs/index.html    the guide (works with GitHub Pages)
docs/img/          screenshots from the DevNet sandbox
lab-data/          restore files, with their own MIT LICENSE
```

## Issues and corrections

Found a wrong value, a changed GUI label or a missing check? Open an issue and include your APIC version.

## License and trademarks

- Guide text, diagrams and README: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You may share and adapt them with credit.
- `lab-data/` (and `scripts/`, when added): MIT, see `lab-data/LICENSE`.
- Screenshots show Cisco software and are included for illustration. They are not covered by CC BY 4.0.
- Cisco, ACI and APIC are trademarks of Cisco Systems, Inc. They are used here only to name the products the lab is about.
