# How it works

The CSW-on-Kubernetes architecture and data path — read these in order to
understand *what the platform is doing* before you install or enforce.

| # | Page | What you'll learn |
|---|---|---|
| 00 | [Official references](./00-official-references.md) | The canonical Cisco sources |
| 01 | [Architecture](./01-architecture.md) | The four components; node-level (not sidecar) capture |
| 02 | [Flow visibility](./02-flow-visibility.md) | Which flows get logged for every traffic pattern (and why there can be 2–3) |
| 03 | [Enforcement translation](./03-enforcement-translation.md) | How a policy intent compiles into node + pod iptables rules |
| 04 | [Scope design](./04-scope-design.md) | Single vs split scope, and the dual-scope allow-rule gotcha |
| 05 | [Vulnerability scanning](./05-vulnerability-scanning.md) | Container image CVE scanning and CVE-risk segmentation |

→ When you're ready to deploy, go to [`../install/`](../install/).
