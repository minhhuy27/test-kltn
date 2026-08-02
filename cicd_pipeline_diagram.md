# Sơ đồ Pipeline CI/CD hoàn chỉnh — Spring Petclinic Microservices

## Tổng quan kiến trúc

Dự án sử dụng **3 pipeline** phối hợp với nhau:

| Pipeline | File | Mục đích |
|---|---|---|
| **CI Pipeline** (Jenkinsfile) | [Jenkinsfile](file:///d:/University/Year%204/Semester%201/Advanced%20Devops/Lab/lab02/Lab02-DevOps/Jenkinsfile) | Build, scan bảo mật, đóng Docker image, push trigger CD |
| **CD Manual Pipeline** (Jenkinsfile-CD) | [Jenkinsfile-CD](file:///d:/University/Year%204/Semester%201/Advanced%20Devops/Lab/lab02/Lab02-DevOps/Jenkinsfile-CD) | Deploy thủ công qua Helm cho môi trường dev-review |
| **GitOps CD** (ArgoCD) | [argocd-apps/](file:///d:/University/Year%204/Semester%201/Advanced%20Devops/Lab/lab02/Lab02-DevOps-CD/argocd-apps) | Tự động sync từ Git repo CD → Kubernetes |

---

## 1. Sơ đồ tổng thể CI/CD

```mermaid
flowchart TB
    subgraph DEV["👨‍💻 Developer"]
        A["Git Push / Tag"]
    end

    subgraph CI["🔧 Jenkins CI Pipeline (Jenkinsfile)"]
        direction TB
        B["1. Checkout SCM"]
        C{"2. Detect Release\n(Git Tag?)"}
        D["3. Detect Changes\n(git diff)"]
        E["4. Docker Login"]
        F["5. Maven Build\n+ Docker Build & Push"]
        G["6. Docker Cleanup"]
        H["7. Push Commit\nto Helm CD Repo"]
        I["8. SAST\nSonarQube Analysis"]
        J["9. SCA\nSnyk Dependency Scan"]
        K["10. DAST\nOWASP ZAP Scan"]
    end

    subgraph CD_GITOPS["🔄 GitOps CD (ArgoCD)"]
        direction TB
        L["ArgoCD detects\nGit changes"]
        M["Helm template\nrender"]
        N{"Môi trường?"}
        O["Deploy → dev namespace"]
        P["Deploy → staging namespace"]
    end

    subgraph CD_MANUAL["🛠️ Jenkins CD Pipeline (Jenkinsfile-CD)"]
        direction TB
        Q["Clone Helm Chart\n& Source Code"]
        R["Generate values.yaml\n(per branch)"]
        S["Helm upgrade\n--install"]
        T["Deploy → dev-review\nnamespace"]
    end

    subgraph K8S["☸️ Kubernetes Cluster"]
        direction TB
        U["Istio Sidecar\nInjection"]
        V["mTLS + AuthZ\n+ Retry Policy"]
        W["Petclinic\nMicroservices"]
    end

    subgraph OBS["📊 Observability"]
        X["Prometheus\n(Alerting)"]
        Y["Zipkin\n(Tracing)"]
        Z["Kiali\n(Service Mesh UI)"]
    end

    A --> B
    B --> C
    C -->|"Tag detected"| E
    C -->|"No tag"| D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J
    J --> K

    H -->|"Update dev/values.yaml\n(commit SHA)"| L
    H -->|"Update staging/values.yaml\n(release tag)"| L
    L --> M
    M --> N
    N -->|"dev"| O
    N -->|"staging"| P
    O --> U
    P --> U

    A -->|"Manual trigger"| Q
    Q --> R
    R --> S
    S --> T
    T --> U

    U --> V
    V --> W
    W --> X
    W --> Y
    W --> Z
```

---

## 2. Chi tiết CI Pipeline (Jenkinsfile)

```mermaid
flowchart LR
    subgraph TRIGGER["Trigger"]
        T1["Push to main"]
        T2["Git Tag\n(Release)"]
        T3["Push to\nfeature branch"]
    end

    subgraph BUILD["Build Phase"]
        B1["Checkout SCM\n+ fetch tags"]
        B2{"Release Tag\ndetected?"}
        B3["Detect Changed\nServices\n(git diff)"]
        B4["Build ALL\nservices"]
        B5["Build CHANGED\nservices only"]
    end

    subgraph DOCKER["Docker Phase"]
        D1["Docker Login\n(credentials)"]
        D2["Maven clean install\n-DskipTests"]
        D3["Docker Build\n(multi-stage)"]
        D4["Docker Push\n(tag: commit SHA)"]
        D5["Docker Push\n(tag: latest)"]
        D6["Docker Push\n(tag: release)"]
        D7["Docker Cleanup\n+ Logout"]
    end

    subgraph SECURITY["DevSecOps Phase"]
        S1["SAST: SonarQube\n(sonar-maven-plugin)"]
        S2["SCA: Snyk\n(Docker container)"]
        S3["DAST: OWASP ZAP\n(baseline scan)"]
    end

    subgraph CD_TRIGGER["CD Trigger"]
        C1["Clone Helm CD Repo"]
        C2{"Tag or Main?"}
        C3["Update dev/values.yaml\n(per-service commit SHA)"]
        C4["Update staging/values.yaml\n(release tag cho ALL)"]
        C5["Bump Chart.yaml\nversion"]
        C6["Git push\nto CD repo"]
    end

    T1 --> B1
    T2 --> B1
    T3 --> B1
    B1 --> B2
    B2 -->|"Yes"| B4
    B2 -->|"No"| B3
    B3 --> B5

    B4 --> D1
    B5 --> D1
    D1 --> D2 --> D3 --> D4
    D4 -->|"main branch"| D5
    D4 -->|"release tag"| D6
    D4 --> D7

    D7 --> C1
    C1 --> C5
    C5 --> C2
    C2 -->|"main"| C3
    C2 -->|"tag"| C4
    C3 --> C6
    C4 --> C6

    D7 --> S1 --> S2 --> S3
```

### Chiến lược Image Tagging

| Trigger | Image Tag | Môi trường target | Services được build |
|---|---|---|---|
| Push to `main` | `<commit-SHA>` + `latest` | **dev** | Chỉ service có thay đổi |
| Git Tag (vd: `V5.7`) | `<tag-name>` | **staging** | Tất cả 6 services |
| Feature branch | `<commit-SHA>` | **dev-review** (manual) | Chỉ service có thay đổi |

---

## 3. Chi tiết CD — GitOps Flow (ArgoCD)

```mermaid
flowchart LR
    subgraph JENKINS["Jenkins CI"]
        J1["git push to\nLab02-DevOps-CD repo"]
    end

    subgraph ARGOCD["ArgoCD Controller"]
        A1["Poll/Webhook\ndetect changes"]
        A2["Render Helm\nChart + values"]
        A3["Compare desired\nvs live state"]
        A4{"Drift\ndetected?"}
        A5["Auto Sync\n(selfHeal + prune)"]
    end

    subgraph HELM["Helm Values Merge"]
        H1["values.yaml\n(base defaults)"]
        H2["environments/dev/values.yaml\n(commit SHA tags)"]
        H3["environments/staging/values.yaml\n(release tag V5.7)"]
    end

    subgraph K8S["Kubernetes"]
        K1["Namespace: dev"]
        K2["Namespace: staging"]
        K3["CreateNamespace=true"]
    end

    J1 --> A1
    A1 --> A2
    H1 --> A2
    H2 --> A2
    H3 --> A2
    A2 --> A3
    A3 --> A4
    A4 -->|"Yes"| A5
    A4 -->|"No"| A1
    A5 --> K3
    K3 --> K1
    K3 --> K2
```

> [!IMPORTANT]
> ArgoCD được cấu hình **automated sync** với `selfHeal: true` và `prune: true`. Điều này nghĩa là:
> - Mọi thay đổi trên Git sẽ **tự động deploy** mà không cần phê duyệt
> - Nếu ai đó sửa trực tiếp trên cluster (kubectl edit), ArgoCD sẽ **tự revert** về trạng thái Git
> - Resources thừa (không còn trong Git) sẽ bị **tự động xóa**

---

## 4. Chi tiết CD — Manual Pipeline (Jenkinsfile-CD)

```mermaid
flowchart LR
    subgraph PARAMS["Input Parameters"]
        P1["namespace\n(default: dev)"]
        P2["Branch per service\n(default: main)"]
    end

    subgraph STEPS["Pipeline Steps"]
        S1["Clone Helm CD Repo"]
        S2["Clone Source Code"]
        S3["Generate values.yaml\n(resolve branch → commit SHA)"]
        S4["helm upgrade --install\n--namespace \u003cns\u003e"]
    end

    subgraph OUTPUT["Result"]
        O1["Deploy to\ncustom namespace"]
        O2["Review Environment\nready"]
    end

    P1 --> S1
    P2 --> S3
    S1 --> S2 --> S3 --> S4 --> O1 --> O2
```

> [!NOTE]
> Pipeline này cho phép deploy **bất kỳ branch nào** của từng service riêng biệt vào một namespace tùy chọn. Hữu ích cho **review environment** khi cần test feature branch trước khi merge.

---

## 5. Bảo mật trong Pipeline (DevSecOps)

```mermaid
flowchart LR
    subgraph SAST["🔍 SAST - Static Analysis"]
        SA1["SonarQube Server\n(34.123.198.80:9000)"]
        SA2["sonar-maven-plugin\n3.9.1.2184"]
        SA3["Quét: bugs, code smells\nsecurity hotspots"]
    end

    subgraph SCA["📦 SCA - Software Composition"]
        SC1["Snyk CLI\n(Docker container)"]
        SC2["Quét: CVEs trong\nMaven dependencies"]
        SC3["snyk test\n--all-projects"]
    end

    subgraph DAST["🌐 DAST - Dynamic Analysis"]
        DA1["OWASP ZAP\n(zaproxy container)"]
        DA2["zap-baseline.py\ntarget: 34.67.163.58:8080"]
        DA3["Output:\nzap_report.html"]
    end

    SA2 --> SA1 --> SA3
    SC1 --> SC2 --> SC3
    DA1 --> DA2 --> DA3
```

---

## 6. Runtime Security & Observability (Post-Deploy)

```mermaid
flowchart TB
    subgraph ISTIO["🛡️ Istio Service Mesh"]
        I1["mTLS STRICT\n(PeerAuthentication)"]
        I2["Zero Trust AuthZ\n(AuthorizationPolicy)"]
        I3["Retry Policy\n(VirtualService: 3 retries on 5xx)"]
    end

    subgraph MONITOR["📊 Monitoring Stack"]
        M1["Prometheus\n(scrape /actuator/prometheus)"]
        M2["Alert: HighErrorRate\n(>10 5xx in 30s)"]
        M3["Zipkin Tracing\n(distributed traces)"]
        M4["Kiali Dashboard\n(service mesh topology)"]
    end

    subgraph CHAOS["🐒 Chaos Engineering"]
        CH1["Chaos Monkey\n(visits-service)"]
        CH2["Inject failures\nto test resilience"]
    end

    I1 --> I2 --> I3
    I3 --> M1
    M1 --> M2
    I3 --> M3
    I3 --> M4
    I3 --> CH1 --> CH2
```

---

## 7. Sơ đồ End-to-End hoàn chỉnh

```mermaid
flowchart TB
    DEV["👨‍💻 Developer\nGit Push / Tag"] --> JENKINS

    subgraph JENKINS["Jenkins CI"]
        direction LR
        JB["Build & Test"] --> JD["Docker\nBuild & Push"] --> JS["Security\nSAST/SCA/DAST"] --> JH["Update\nHelm CD Repo"]
    end

    JENKINS -->|"git push to CD repo"| ARGOCD

    subgraph ARGOCD["ArgoCD (GitOps)"]
        direction LR
        AG1["Detect Git\nChanges"] --> AG2["Helm Render\n+ Auto Sync"]
    end

    JENKINS -->|"Manual trigger\n(Jenkinsfile-CD)"| HELM_MANUAL
    subgraph HELM_MANUAL["Jenkins CD (Manual)"]
        HM1["helm upgrade\n--install"]
    end

    ARGOCD --> K8S
    HELM_MANUAL --> K8S

    subgraph K8S["Kubernetes Cluster"]
        direction TB
        subgraph NS["Namespaces"]
            N1["dev\n(commit SHA tags)"]
            N2["staging\n(release tags)"]
            N3["dev-review\n(branch tags)"]
        end
        subgraph MESH["Istio Service Mesh"]
            IS1["mTLS"]
            IS2["AuthZ Policy"]
            IS3["Retry"]
        end
        subgraph APPS["Petclinic Services"]
            APP1["api-gateway"]
            APP2["config-server"]
            APP3["discovery-server"]
            APP4["customers-service"]
            APP5["visits-service"]
            APP6["vets-service"]
        end
        NS --> MESH --> APPS
    end

    K8S --> OBS
    subgraph OBS["Observability"]
        direction LR
        O1["Prometheus\n+ Alerts"]
        O2["Zipkin\nTracing"]
        O3["Kiali\nDashboard"]
    end

    style DEV fill:#4CAF50,color:#fff
    style JENKINS fill:#D24939,color:#fff
    style ARGOCD fill:#EF7B4D,color:#fff
    style K8S fill:#326CE5,color:#fff
    style OBS fill:#9C27B0,color:#fff
    style HELM_MANUAL fill:#FF9800,color:#fff
```

---

## Tóm tắt luồng hoạt động

### Flow 1: Development → Dev (Tự động)
```
Developer push code → Jenkins CI detect changed services → Maven build → Docker build & push (tag: commit SHA)
→ Clone CD repo → Update environments/dev/values.yaml (per-service tag) → Bump Chart version → Git push
→ ArgoCD auto-sync → Deploy to dev namespace
```

### Flow 2: Release → Staging (Tự động)
```
Developer tạo Git tag (vd: V5.7) → Jenkins CI build ALL services → Docker build & push (tag: V5.7)
→ Clone CD repo → Update environments/staging/values.yaml (all services = V5.7) → Git push
→ ArgoCD auto-sync → Deploy to staging namespace
```

### Flow 3: Feature Review → Dev-review (Thủ công)
```
Developer trigger Jenkinsfile-CD → Chọn branch per service + namespace
→ Clone source code → Resolve branch → commit SHA → Generate values.yaml
→ Helm upgrade --install → Deploy to custom namespace
```

### Security Pipeline (Chạy song song/sau build)
```
SonarQube (SAST) → Snyk (SCA) → OWASP ZAP (DAST) → Reports archived in Jenkins
```
