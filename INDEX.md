# Quick Index — by topic and by question

> See [`README.md`](./README.md) for the overview, architecture diagram, and
> lifecycle. This index jumps straight to the page that answers a question.

---

## By topic

| Topic | Page |
|---|---|
| Architecture — the four components | [`docs/01-architecture.md`](./docs/01-architecture.md) |
| Flow visibility — which flows get logged | [`docs/02-flow-visibility.md`](./docs/02-flow-visibility.md) |
| Enforcement — node vs pod rule translation | [`docs/03-enforcement-translation.md`](./docs/03-enforcement-translation.md) |
| Scope design — single vs split | [`docs/04-scope-design.md`](./docs/04-scope-design.md) |
| Vulnerability scanning (CVEs) | [`docs/05-vulnerability-scanning.md`](./docs/05-vulnerability-scanning.md) |
| Prerequisites | [`install/01-prerequisites.md`](./install/01-prerequisites.md) |
| Connector / orchestrator (metadata) | [`install/02-connector-orchestrator.md`](./install/02-connector-orchestrator.md) |
| Install the DaemonSet (Agent Script Installer) | [`install/03-agent-script-installer.md`](./install/03-agent-script-installer.md) |
| OpenShift specifics (SCC, OVN/IPVS) | [`install/04-openshift.md`](./install/04-openshift.md) |
| Verify the install | [`install/05-verification.md`](./install/05-verification.md) |
| Uninstall | [`install/06-uninstall.md`](./install/06-uninstall.md) |
| CNI coexistence (Preserve Rules) | [`operations/01-cni-coexistence.md`](./operations/01-cni-coexistence.md) |
| Troubleshooting | [`operations/02-troubleshooting.md`](./operations/02-troubleshooting.md) |
| FAQ | [`operations/03-faq.md`](./operations/03-faq.md) |
| Official Cisco references | [`docs/00-official-references.md`](./docs/00-official-references.md) |

---

## By question

| If you're asking… | Start here |
|---|---|
| *"How does CSW see traffic in a cluster without a sidecar?"* | [`docs/01-architecture.md`](./docs/01-architecture.md) |
| *"Why do I see two/three flows for one connection?"* | [`docs/02-flow-visibility.md`](./docs/02-flow-visibility.md) |
| *"What's a UDP node→node flow next to my pod→pod flow?"* | [`docs/02-flow-visibility.md`](./docs/02-flow-visibility.md) |
| *"How does my policy intent become actual iptables rules?"* | [`docs/03-enforcement-translation.md`](./docs/03-enforcement-translation.md) |
| *"My NodePort policy looks right but traffic is blocked."* | [`docs/04-scope-design.md`](./docs/04-scope-design.md) (split-scope) + [`docs/03`](./docs/03-enforcement-translation.md) |
| *"Single scope or split scope for my cluster?"* | [`docs/04-scope-design.md`](./docs/04-scope-design.md) |
| *"How do I install the agent on my cluster?"* | [`install/03-agent-script-installer.md`](./install/03-agent-script-installer.md) |
| *"Pods are Pending / ImagePullBackOff."* | [`operations/02-troubleshooting.md`](./operations/02-troubleshooting.md) |
| *"Agents registered but no flows."* | [`operations/02-troubleshooting.md`](./operations/02-troubleshooting.md) |
| *"OpenShift — what's different?"* | [`install/04-openshift.md`](./install/04-openshift.md) |
| *"Does it work with my CNI / can I keep CNI policies?"* | [`operations/01-cni-coexistence.md`](./operations/01-cni-coexistence.md) · [`operations/03-faq.md`](./operations/03-faq.md) |
| *"How do I scan container images for CVEs?"* | [`docs/05-vulnerability-scanning.md`](./docs/05-vulnerability-scanning.md) |
| *"How do I remove the agent cleanly?"* | [`install/06-uninstall.md`](./install/06-uninstall.md) |
