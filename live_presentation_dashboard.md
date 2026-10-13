# 🚀 GitOps Cloud Automation — Live Presentation Dashboard

All components are **100% active, healthy, and accessible**.

---

## 🌐 Live URLs & Access Credentials

| Service | Location | Credentials / Notes |
| :--- | :--- | :--- |
| **Public Web App (Podinfo)** | [https://secondary-called-reflect-distance.trycloudflare.com](https://secondary-called-reflect-distance.trycloudflare.com)<br>*(or local: [http://localhost:9898](http://localhost:9898))* | Accessible from any browser or phone |
| **ArgoCD Dashboard** | [https://localhost:8080](https://localhost:8080) | Username: `admin`<br>Password: `ZkjaGmfMrPXLqUsV` |
| **GitHub Source of Truth** | [https://github.com/YatharthJangid/gitops-cloud-automation](https://github.com/YatharthJangid/gitops-cloud-automation) | Public repo on `main` branch |

> [!NOTE]
> When accessing **ArgoCD at https://localhost:8080**, your browser will show a self-signed certificate warning (`Your connection is not private`). Click **Advanced** -> **Proceed to localhost (unsafe)**.

---

## 🖥️ Screen Layout for Presentation (Step 5)

Arrange three side-by-side browser windows:

```
┌────────────────────────┬────────────────────────┬────────────────────────┐
│      GitHub Repo       │    ArgoCD Dashboard    │    Live Web App UI     │
│  (Declarative Source)  │  (Reconciliation Loop) │  (Public Cloud View)   │
│                        │                        │                        │
│ github.com/.../        │ https://localhost:8080 │ https://secondary-ca...│
│ deployment.yaml        │ card: podinfo          │ Blue theme v1.0        │
└────────────────────────┴────────────────────────┴────────────────────────┘
```

---

## 🎭 Live Demonstration Flow & Scripts

### Act 1: The GitOps Architecture Pitch (1 Minute)
* **What you show:** Focus on center window (ArgoCD UI). Point out the green status badges:
  - `Status: Synced`
  - `Health: Healthy`
  - Pods: 2 active replicas.
* **What you say:**
  > *"Traditional CI/CD relies on push-based pipelines where external runners store cluster credentials to push changes. In our Cloud Automation project, we implemented GitOps using a pull-based model. Git is our single source of truth. ArgoCD runs directly inside the cluster, continuously comparing the live Kubernetes cluster state against our declarative Git repository."*

---

### Act 2: Proving Self-Healing & Eliminating Configuration Drift (1.5 Minutes)
* **Action:** Open terminal and run either of these drift simulation commands:
  ```bash
  # Delete a pod manually
  kubectl delete pod -l app=podinfo --now
  ```
  *or*
  ```bash
  # Force replicas to 0 manually
  kubectl scale deployment podinfo --replicas=0
  ```
* **What happens live:**
  - ArgoCD immediately flags the drift / OutOfSync.
  - Within 3 to 5 seconds, ArgoCD's self-healing engine detects the drift from Git and spins the pods right back up to 2.
* **What you say:**
  > *"If an engineer makes unauthorized manual changes via kubectl or a pod crashes, the cluster detects the configuration drift. Because Git dictates two replicas, ArgoCD immediately self-heals and overrides the manual change. Git remains the sole authority."*

---

### Act 3: Automated Zero-Downtime Rolling Update (2 Minutes)
* **Action:** In GitHub, click edit (pencil icon) on `deployment.yaml`:
  1. Change `PODINFO_UI_COLOR` from `"#245bf5"` (Blue) to `"#16a34a"` (Green).
  2. Change `PODINFO_UI_MESSAGE` to `"Production v2.0 - Deployed Live via GitOps!"`.
  3. Change `replicas: 2` to `replicas: 4`.
  4. Click **Commit changes**.
* **What happens live:**
  - In ArgoCD, watch the rolling update trigger (or click **SYNC** if you don't want to wait for polling).
  - ArgoCD will spin up 4 green pods while smoothly terminating old ones.
  - Refresh the right window (Public URL): UI turns green with 4 distinct hostnames and the v2.0 message!
* **What you say:**
  > *"With a single Git commit, we triggered an automated rolling update. ArgoCD synchronized the state, scaled our deployment from 2 to 4 pods, and updated the visual environment with zero downtime."*

---

## 💡 Evaluation Q&A Cheat Sheet

| Question | Strong Answer |
| :--- | :--- |
| **Why is pull-based GitOps safer than push-based CI/CD?** | Push-based CI systems (like Jenkins or GitHub Actions runners) require administrative cluster credentials (`kubeconfig`) stored externally. If the CI runner is compromised, the entire cluster is exposed. GitOps uses an internal pull agent (ArgoCD), so cluster access credentials never leave the cluster perimeter. |
| **How does rollback work in this system?** | Since the entire deployment history is version-controlled, rolling back is just running `git revert <commit-hash>`. Pushing that commit causes ArgoCD to immediately roll back the cluster. Alternatively, you can click "Rollback" directly in the ArgoCD UI to revert to a previous revision. |
| **What happens if Git goes down?** | The cluster continues running uninterrupted in its last known desired state. Only new deployments or sync operations are paused until Git connectivity resumes. |
