# Troubleshooting (Kubernetes / OpenShift)

> Pairs with [`install/05-verification.md`](../install/05-verification.md). Use
> `kubectl` or `oc`.

---

## Install / pod problems

| Symptom | Likely cause | Fix |
|---|---|---|
| Agent pod `Pending` | PSA `restricted` blocks privileged pod, or SCC (OpenShift), or node taint | Allow privileged in `tetration` ns; bind `privileged` SCC; add `--toleration` |
| `ImagePullBackOff` | Node can't reach `CFG-SERVER:443`; runtime proxy not set; air-gapped | Open node→CSW path; set **container runtime** proxy (not the installer proxy); or mirror image to internal registry |
| `CrashLoopBackOff` | Missing host mounts / `hostNetwork` (hand-edited spec) | Re-run the installer; don't hand-edit the DaemonSet |
| Fewer agent pods than nodes | Control-plane taints; unschedulable nodes | Add toleration; check `kubectl describe node` |

> **Installer proxy ≠ image-pull proxy.** The proxy on the installer page only
> controls **agent → CSW** traffic. **Node image pulls use the container
> runtime's own proxy.** This is the #1 air-gapped gotcha.

---

## Registered but no flows

| Symptom | Cause | Fix |
|---|---|---|
| Agents registered, **zero flows** | Missing `hostNetwork: true` / host-path mounts → agent only sees its own netns | Confirm DaemonSet spec from the script; redeploy |
| Flows present but **no labels** | Connector not configured / not syncing | Set up the [connector](../install/02-connector-orchestrator.md); check its health & RBAC |
| "Extra" node→node UDP/TCP flows | Overlay CNI encapsulation (VXLAN/Geneve/IPIP) | Expected — see [`docs/02-flow-visibility.md`](../docs/02-flow-visibility.md); don't policy the tunnel |

---

## Enforcement problems

| Symptom | Cause | Fix |
|---|---|---|
| Flow blocked though "policy looks right" | **Split-scope**: NodePort/health-check needs allow on **both** parent (node) and child (app) scope | Add the allow rule on both scopes — see [`docs/04-scope-design.md`](../docs/04-scope-design.md) |
| Rules intermittently stop working | CNI (e.g. Felix) rewriting chains | Enable **Preserve Rules**; set Calico `IptablesRefreshInterval: 0` — see [`operations/01-cni-coexistence.md`](./01-cni-coexistence.md) |
| Enforcement not available on OpenShift | OVN networking or IPVS kube-proxy | Not supported — use SDN; confirm with `oc get network.operator cluster -o yaml` |
| ClusterIP policy not matching backend | Forgot the auto dst=podIP behavior | The engine auto-generates the post-DNAT pod-IP INGRESS rule — verify the provider pod rule exists ([`docs/03-enforcement-translation.md`](../docs/03-enforcement-translation.md)) |

---

## Diagnostic commands

```bash
# Pod / DaemonSet state
kubectl get pod -n tetration -o wide
kubectl describe pod <agent-pod> -n tetration | sed -n '/Events/,$p'
kubectl logs <agent-pod> -n tetration --tail=100

# Node connectivity to the CSW cluster (run from a node/debug pod)
curl -vk https://<CFG-SERVER-IP>:443    # expect TLS handshake, not timeout

# CNI / networking
kubectl get felixconfiguration default -o yaml      # Calico
oc get network.operator cluster -o yaml             # OpenShift SDN/OVN

# Inspect node iptables for CSW vs CNI chains (host shell)
iptables-save | grep -iE 'ta_|cali|cilium|KUBE-'
```

---

## When to call TAC

- Calico version/Felix config differs from the supported 4.0 matrix.
- OpenShift on OVN where you need enforcement.
- Enforcement rules conflict with CNI despite *Preserve Rules*.

Collect agent logs first (CSW UI: **Initiate Log Collection** for the root scope,
or pod logs above).

---

## See also

- [`install/05-verification.md`](../install/05-verification.md)
- [`operations/01-cni-coexistence.md`](./01-cni-coexistence.md)
- [`operations/03-faq.md`](./03-faq.md)
