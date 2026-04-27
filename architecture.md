# GitOps Pipeline — Architecture Diagram

## Detailed view (with namespaces and image flow)

```mermaid
flowchart LR
  classDef user    fill:#ECEFF1,stroke:#546E7A,color:#000,stroke-width:2px
  classDef ext     fill:#FFFFFF,stroke:#24292E,color:#000,stroke-width:2px
  classDef docker  fill:#E3F2FD,stroke:#2496ED,color:#000,stroke-width:2px
  classDef argo    fill:#EF7B4D,stroke:#B85525,color:#FFFFFF,stroke-width:2px
  classDef pod     fill:#326CE5,stroke:#1A4FA8,color:#FFFFFF,stroke-width:2px
  classDef svc     fill:#5A8DEE,stroke:#1A4FA8,color:#FFFFFF,stroke-width:2px

  Dev["👨‍💻 Developer<br/>(Laptop)"]:::user
  Browser["🌐 Browser"]:::user
  GH[("🐙 GitHub Repo<br/>k8s/deployment.yaml<br/>k8s/service.yaml")]:::ext
  DH[("🐳 Docker Hub<br/>gitops-demo:v1")]:::docker

  subgraph EC2["☁️ AWS EC2 — Ubuntu 22.04 · SG: 22 / 30080 / 30090"]
    direction TB
    subgraph Dock["Docker Engine"]
      direction TB
      subgraph Kind["kind cluster (control-plane)"]
        direction LR
        subgraph NSArgo["namespace: argocd"]
          ArgoSrv["argocd-server<br/>Service: NodePort 30080"]:::argo
        end
        subgraph NSDef["namespace: default"]
          NgDeploy["nginx Deployment<br/>(2 pods)<br/><br/>Phase 1: nginx:1.27<br/>Phase 2: gitops-demo:v1"]:::pod
          NgSvc["nginx Service<br/>NodePort 30090<br/>selector: app=nginx"]:::svc
        end
      end
    end
  end

  Dev -->|"git push (manifests)"| GH
  Dev -->|"docker build & push"| DH
  ArgoSrv -.->|"poll & pull manifests"| GH
  ArgoSrv -->|"kubectl apply (sync)"| NgDeploy
  NgDeploy -.->|"pull image (rollout)"| DH
  NgSvc -->|"selects pods"| NgDeploy
  Browser -->|"HTTP :30080 (ArgoCD UI)"| ArgoSrv
  Browser -->|"HTTP :30090 (Demo page)"| NgSvc
```

---

## Legend

| Style | Meaning |
|---|---|
| Solid arrow `-->` | Active push / apply (someone is doing something now) |
| Dashed arrow `-.->` | Poll / pull (happens on a schedule or on-demand) |
| Orange box | ArgoCD / AWS |
| Blue box | Kubernetes / Docker |
| Light gray | User-facing actor (laptop / browser) |

---

## Simpler one-liner version

For slides or a 30-second explanation:

```mermaid
flowchart LR
  Dev[Developer] -->|git push| GH[(GitHub)]
  GH -.->|pulled by| Argo[ArgoCD on EC2/kind]
  Argo -->|deploys| App[App pod<br/>NodePort 30090]
  Browser[Browser] -->|HTTP| App
```

---

## How to view / export

- **VS Code:** install the *Markdown Preview Mermaid Support* extension, then open this file and `Ctrl+Shift+V` to preview
- **GitHub:** the diagram renders automatically in any `.md` file pushed to a public repo
- **Live editor:** paste the code block into https://mermaid.live to tweak and export PNG / SVG
- **Inside slides:** export as PNG from mermaid.live, or use the *Mermaid* plugin for Notion / Obsidian / Confluence
