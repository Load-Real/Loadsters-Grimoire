---
status: stub
tags:
  - cloud
  - containers
  - methodology
  - process
---
> *Cloud & Containers Methodology:* The approach and tooling map for GCP, Entra ID, and Kubernetes/IAM.

> [!TODO] Needs Work
> This document is currently the first and only note in an otherwise-empty section, so it is the process skeleton and the pointers, the per platform technique notes (named in the stubs below) get build as I do the work.

> [!WARNING] Naming Notice!
> Azure AD was renamed **Entra ID**.

# The Two Planes

Cloud work splits into two questions, and good coverage answers both:
- **Config Posture:** Is the environment misconfigured? Run a scanner for the baseline.
- **Attack Paths:** Given an identity, where can it reach? Map the graph.

`ScoutSuite` and `Prowler` cover posture across providers. The attack-path tooling is provider-specific below.

# GCP

- **Posture:** `ScouteSuite`, `Prowler`
- **Storage:** `GCPBucketBrute` for public read/write listing
- **Metadata:** Server-side fetch (SSRF from [[02AA - Methodology]]) → `169.254.169.254` for instance creds
- **Reference + Practice:** HackTricks Cloud (GCP section), `GCPGoat` labs, Rhino Security Lab's GCP research for the deep end
- *Future note:* `04B - GCP`

# Azure / Entra ID

- **Directory enum:** `ROADtools`/`roadrecon`
- **Attack paths:** `AzureHound` maps into BloodHound, so this ties straight back to [[03AA - Methodology]]
- **Tradecraft:** `MicroBurst` (PowerShell), `PowerZure`
- **Deep Entra:** `AADInternals`
- **User enum / spray:** `o365spray`, `AADInternals`
- **Posture:** `ScoutSuite`, `Prowler`
- _Future note:_ `04C - Entra ID`

# Kubernetes & IAM

- Metadata endpoints and mounted cloud creds, SSRF to the control plane, IAM privilege-escalation paths
- _Future note:_ `04D - Kubernetes & Containers`

> [!NOTE] Tooling Currency
> `AzureHound` output lands in the same BloodHound CE you use for on-prem AD, so cloud and on-prem attack paths share one graph. `ScoutSuite`/`Prowler` are the cross-provider posture default.

# See Also

- [[00G - Methodology & Mindset]] for the operating model
- [[03AA - Methodology]] because AzureHound feeds the same BloodHound graph
- [[02AA - Methodology]] for the SSRF-to-metadata pivot that starts most cloud footholds
- [[00B - References]] for HackTricks Cloud and the cloud toolkit