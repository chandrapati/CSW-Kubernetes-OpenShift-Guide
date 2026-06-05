# How to install

Deploy the Secure Workload agent (DaemonSet) and the metadata connector to a
Kubernetes / OpenShift cluster, using Cisco's documented Agent Script Installer.

| # | Page | Purpose |
|---|---|---|
| 01 | [Prerequisites](./01-prerequisites.md) | Access, connectivity, platform/CNI support, PSA/SCC, namespace |
| 02 | [Connector / orchestrator](./02-connector-orchestrator.md) | Cluster metadata (labels/annotations) for label-based policy |
| 03 | [Agent Script Installer](./03-agent-script-installer.md) | **The supported DaemonSet install** |
| 04 | [OpenShift specifics](./04-openshift.md) | SCC, OVN/IPVS caveats, `oc` commands |
| 05 | [Verification](./05-verification.md) | Confirm pods, registration, flows, inventory |
| 06 | [Uninstall](./06-uninstall.md) | Clean removal (incl. Windows DaemonSet caveat) |

> **Supported vs community.** The Agent Script Installer is Cisco's supported
> path. Helm/GitOps/raw-manifest patterns are community approaches — see the
> [CSW-Agent-Installation-Guide `kubernetes/` folder](https://github.com/chandrapati/CSW-Agent-Installation-Guide/tree/main/kubernetes).

→ To understand what you're deploying, read [`../docs/`](../docs/) first.
