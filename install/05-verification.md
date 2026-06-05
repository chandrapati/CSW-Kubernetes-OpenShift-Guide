# Verify the installation

> **Cisco source.** [Service Management for Kubernetes Agent Installations (4.0)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/deploy-software-agents.html).

The installer prints verification steps on completion; here's the practical
checklist. Use `kubectl` (or `oc` on OpenShift).

---

## 1. DaemonSet and pods are up

```bash
# One agent pod per schedulable node, all Running
kubectl get pod -n tetration -o wide
oc get pod -n tetration -o wide        # OpenShift

# DaemonSet desired == ready
kubectl get ds -n tetration
```

Expect **DESIRED = CURRENT = READY = number of (covered) nodes**. A node short?
Check tolerations (control-plane nodes) and PSA/SCC.

---

## 2. Pods are healthy (not crash-looping)

```bash
kubectl get pod -n tetration -o wide
kubectl describe pod <agent-pod> -n tetration | sed -n '/Events/,$p'
kubectl logs <agent-pod> -n tetration --tail=50
```

| Symptom | Likely cause | Fix |
|---|---|---|
| `Pending` | PSA/SCC blocking privileged pod, or taint without toleration | allow privileged in `tetration` ns / bind SCC / add toleration |
| `ImagePullBackOff` | node can't reach `CFG-SERVER:443`, or runtime proxy missing | fix node→CSW connectivity / runtime proxy / mirror image |
| `CrashLoopBackOff` | missing host mounts or `hostNetwork` | confirm DaemonSet spec from the script (don't hand-edit) |

---

## 3. Agents registered in the CSW UI

- **Manage → Agents** (or the workload/inventory view): the node agents should
  appear as **registered**, with recent **check-in** times and **Agent Health**.
- The agent **type/OS/version** and last check-in are shown per agent.

---

## 4. Flows are arriving (and label-enriched)

- Open the **flow/visibility** view and filter to the cluster scope.
- You should see pod-to-pod and pod-to-external conversations, **enriched with
  pod/namespace/label metadata** (that enrichment confirms the **connector** is
  also working — see [`02-connector-orchestrator.md`](./02-connector-orchestrator.md)).
- If agents register but **zero flows** appear: verify `hostNetwork: true` and
  the host-path mounts — without them the agent only sees its own pod netns.

---

## 5. Inventory is syncing (connector)

- Pod/service **profiles** show labels/annotations.
- Add a label to a test pod and confirm it appears in CSW in near real-time.

---

## Service-management quick reference

| Task | Command |
|---|---|
| List agent pods | `kubectl get pod -n tetration` / `oc get pod -n tetration` |
| Restart agent on a node | delete the pod; the DaemonSet recreates it automatically |
| Pod logs | `kubectl logs <agent-pod> -n tetration` |
| Rollout status | `kubectl rollout status ds/<daemonset> -n tetration` |

---

## See also

- [`operations/02-troubleshooting.md`](../operations/02-troubleshooting.md) — deeper diagnosis
- [`docs/02-flow-visibility.md`](../docs/02-flow-visibility.md) — what flows to expect
- [`install/06-uninstall.md`](./06-uninstall.md) — removal
