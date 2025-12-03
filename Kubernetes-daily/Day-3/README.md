# Day-3: Gateway API Lab – From Total Chaos to Final Victory

**Date**: 02 December 2025  
**Duration**: ~8 hours of war  
**Result**: 100 % working Gateway API lab with external access  

## Goal
Create a clean, reliable Kubernetes Gateway API lab on an AWS t3.xlarge so I can:
- Master `Gateway` + `HTTPRoute` (the modern replacement for Ingress)
- Crush Day-53 networking labs for CKA 2025
- Have a stable playground for NetworkPolicy, TLS, multi-node CNI, etc.

## The Painful Journey (What Actually Happened)

| Phase | What I Tried                                 | Outcome                                 |
|------|-----------------------------------------------|-----------------------------------------|
| 1    | K3s + ingress-nginx bare-metal                | Worked 5 mins → died on reboot          |
| 2    | Every hostPort/NodePort trick                 | Never stable                            |
| 3    | KIND inside EC2                               | Good direction                          |
| 4    | NGINX Gateway Fabric (Helm + manifests)       | 404s, CRD >262144 errors, cert-generator crashes, Helm schema disasters → endless 502s |
| 5    | Nuclear wipe → fresh Docker + KIND            | Clean slate (necessary!)                |
| 6    | Contour + Envoy (official quickstart)         | Everything inside cluster works perfectly |
| 7    | Port mapping still broken                     | extraPortMappings ignored               |
| 8    | Manual port forward (`socat`)                 | **VICTORY – external access works!**    |

## Why I Went Nuclear
Old setup was pure Frankenstein:
- Mixed K3s + leftover KIND containers
- Wrong kubeconfigs (sudo vs non-sudo)
- Broken Docker daemon after purge
Nothing was salvageable. Clean slate was the only way.

## Final Working Stack (100 % Reliable)
- Ubuntu 22.04 on t3.xlarge
- Docker → KIND cluster (`kind-lab`)
- Gateway API CRDs v1.2.0
- **Contour + Envoy** – the exact stack used by official Kubernetes conformance tests
- `hello-app` (nginxdemos/hello) → `hello-svc` → `HTTPRoute` → `Gateway` → Envoy → external world

## Current URLs (Working Right Now)
```bash
# Inside the EC2 (after socat forward)
http://127.0.0.1:30080

# From anywhere on the internet
http://<YOUR_EC2_PUBLIC_IP>:30080
```
## Why Contour + Envoy Won

| Requirement                               | NGINX Gateway Fabric | Contour + Envoy |
|-------------------------------------------|------------------------|------------------|
| Full Gateway API v1 conformance           | Partial                | Yes              |
| Works reliably in KIND                    | No (cert issues)       | Yes              |
| Tiny footprint                            | No                     | Yes              |
| No Helm drama                             | No                     | Yes              |
| Used by every CKA course                  | No                     | Yes              |

> **Contour is the boring, battle-tested choice that just works.**

---

## Key Lessons Learned

- NGINX Gateway Fabric → production beast, **lab nightmare**  
- Contour + Envoy → boring = **perfect for certification**  
- `extraPortMappings` in KIND is flaky — **always verify NodePort**  
- **Never run** `sudo kubectl` after setting up KIND  
- When everything is cursed → **nuke & rebuild is faster**

---

## Next Steps (Whenever I Want)

- TLS → cert-manager + Let’s Encrypt in **3 minutes**
- NetworkPolicy → only Gateway can reach the app
- Multi-node KIND cluster for **real CNI labs**
- Swap to NGINX Gateway Fabric later for **advanced HTTP features**
