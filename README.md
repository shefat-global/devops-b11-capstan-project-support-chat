# Capstan Project - Support Chat DevOps Implementation

## 1. Introduction

In this project, I took the provided Support Chat application and built a practical DevOps workflow around it. My main goal was to move the application from a simple source-code project to a more structured setup with Git branching, CI validation, Docker images, Kubernetes manifests, GitHub Actions, Docker Hub, and Argo CD.

I worked with separate `dev`, `stage`, and `prod` environments and used GitHub as the main source of truth for the application and deployment configuration. I also verified the production deployment directly from Kubernetes after the final image update.

**Repository:** `shefat-global/devops-b11-capstan-project-support-chat`

---

## 2. Project Objectives

The main objectives I followed were:

- Set up a clean Git workflow using `main`, `dev`, `stage`, and `prod` branches.
- Keep development work isolated through feature/chore branches and pull requests.
- Add continuous integration checks for the backend and frontend.
- Containerize both the backend and frontend applications.
- Build and publish versioned Docker images to Docker Hub.
- Create Kubernetes manifests for the application environments.
- Use Argo CD for GitOps-based Kubernetes synchronization.
- Configure different GitHub environments for `dev`, `stage`, and `prod`.
- Verify that the production cluster is actually running the intended Docker image versions.
- Keep the repository clean after the implementation was completed.

---

## 3. Application Overview

The project is a Support Chat application with a frontend and backend. The frontend provides the chat interface, while the backend provides the server-side application logic. I kept the original application structure and added the DevOps components around it instead of changing the core application unnecessarily.

The main repository structure is:

```text
.
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── k8s/
│   ├── dev/
│   ├── stage/
│   └── prod/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── docker-compose.yml
└── README.md
```

### Application test

I also tested the application locally and confirmed that the Support Chat interface could be opened successfully.

<p align="center"><img src="screenshots/Screenshot_74.png" alt="Screenshot 74" width="820"></p>
*Figure 1. Local Support Chat application running successfully.*

---

## 4. Git Branching Strategy

I used four long-lived branches:

| Branch | Purpose |
|---|---|
| `main` | Main integration branch |
| `dev` | Development environment |
| `stage` | Staging environment |
| `prod` | Production environment |

For individual changes, I created temporary `chore/*` branches and merged them through pull requests. After the changes were merged and verified, I removed the temporary branches. At the end of the project, only the four main working branches remained on the local and remote repositories.

<p align="center"><img src="screenshots/Screenshot_191.png" alt="Screenshot 191" width="820"></p>
*Figure 2. GitHub environments showing `dev`, `stage`, and `prod` configuration.*

<p align="center"><img src="screenshots/Screenshot_210.png" alt="Screenshot 210" width="820"></p>
*Figure 3. Pull request used to update the production image references.*

---

## 5. GitHub Environments

I configured separate GitHub environments for `dev`, `stage`, and `prod`. This allows deployment jobs to refer to an environment explicitly instead of treating all deployments as the same process. The production environment was also used for the production deployment workflow and approval flow.

The environment configuration was checked from GitHub Settings, and the final repository showed the three required deployment environments.

<p align="center"><img src="screenshots/Screenshot_191.png" alt="Screenshot 191" width="820"></p>
*Figure 4. Final GitHub environment configuration.*

---

## 6. Continuous Integration with GitHub Actions

I created the main workflow at `.github/workflows/deploy.yml`. The CI job checks both parts of the application before deployment-related jobs continue.

The backend validation includes:

- Installing dependencies with `npm ci`.
- Running the backend type check.
- Building the backend.

The frontend validation includes:

- Installing frontend dependencies.
- Running the frontend lint check.
- Building the frontend.

The workflow also contains environment verification and deployment-related jobs.

A successful workflow run was verified from GitHub Actions.

<p align="center"><img src="screenshots/Screenshot_207.png" alt="Screenshot 207" width="820"></p>
*Figure 5. Successful GitHub Actions run showing CI validation and Docker image build/push.*

---

## 7. Dockerization

I containerized the backend and frontend separately. This gives the two application components independent images and makes them easier to build, publish, and deploy.

### 7.1 Backend Docker image

The backend image is built from `backend/Dockerfile`. The workflow uses the backend directory as the Docker build context and publishes the image as:

```text
shiftrobin/support-chat-backend:<tag>
```

### 7.2 Frontend Docker image

The frontend image is built from `frontend/Dockerfile`. The frontend uses a build stage with Node.js and then serves the generated application through Nginx.

The published image is:

```text
shiftrobin/support-chat-frontend:<tag>
```

Both Dockerfiles were verified on the development and production branches.

<p align="center"><img src="screenshots/Screenshot_202.png" alt="Screenshot 202" width="820"></p>
*Figure 6. Backend Docker Hub repository and versioned image tags.*

<p align="center"><img src="screenshots/Screenshot_203.png" alt="Screenshot 203" width="820"></p>
*Figure 7. Frontend Docker Hub repository and versioned image tags.*

---

## 8. Docker Image Versioning

I used commit-based image tags so that the deployed image can be traced back to a Git commit. The workflow generates tags in the following format:

```text
dev-<short-commit>
prod-<short-commit>
```

For example, the production deployment used:

```text
prod-fe78d34
```

This is better than using only `latest` because I can identify exactly which version is running in Kubernetes.

<p align="center"><img src="screenshots/Screenshot_203.png" alt="Screenshot 203" width="820"></p>
*Figure 8. Docker Hub showing versioned image tags.*

---

## 9. Docker Compose

I also kept a project-level `docker-compose.yml` for local container-based development. The Compose setup contains separate frontend and backend services and maps their application ports for local use.

The Compose file is kept at the project root because it is a local development and testing configuration rather than an environment-specific Kubernetes manifest.

I verified that the repository contains `docker-compose.yml` on the main branch.

<p align="center"><img src="screenshots/Screenshot_128.png" alt="Screenshot 128" width="820"></p>
*Figure 9. Local Docker Desktop/container view used during container testing.*

---

## 10. Kubernetes Deployment

I created Kubernetes manifests for the application components. Each environment has separate backend and frontend Deployment and Service manifests.

The environment-specific structure is:

```text
k8s/
├── dev/
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   └── frontend-service.yaml
├── stage/
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   └── frontend-service.yaml
└── prod/
    ├── backend-deployment.yaml
    ├── backend-service.yaml
    ├── frontend-deployment.yaml
    └── frontend-service.yaml
```

The services use Kubernetes `ClusterIP` networking. In the final production verification, the backend service exposed port `5000` and the frontend service exposed port `80`. No Ingress resource was configured in the final setup.

<p align="center"><img src="screenshots/Screenshot_213.png" alt="Screenshot 213" width="820"></p>
*Figure 10. Final repository audit showing the Kubernetes manifests for dev, stage, and prod and the Dockerfile checks.*

---

## 11. Argo CD GitOps Deployment

I used Argo CD to manage Kubernetes application synchronization from the Git repository. I created separate Argo CD applications for:

- `support-chat-dev`
- `support-chat-stage`
- `support-chat-prod`

The applications point to the environment-specific Kubernetes paths. Argo CD provides the GitOps layer so that Kubernetes state can be reconciled from the repository configuration.

<p align="center"><img src="screenshots/Screenshot_177.png" alt="Screenshot 177" width="820"></p>
*Figure 11. Argo CD showing the three Support Chat applications in Healthy and Synced state.*

For production, I also verified the application directly from the Kubernetes CLI. The final production state was:

```text
REVISION   SYNCED   HEALTHY
prod       Yes      Yes
```

---

## 12. Production Deployment Verification

The most important part of my final verification was checking the actual Kubernetes Pods rather than only checking GitHub Actions.

The production Deployment was updated to use:

```text
shiftrobin/support-chat-backend:prod-fe78d34
shiftrobin/support-chat-frontend:prod-fe78d34
```

After refreshing and synchronizing the Argo CD application, Kubernetes created new Pods. The final verification showed both Pods in `Running` state with the new image tag.

This confirmed the complete flow from Git to the container registry, GitOps controller, and Kubernetes workload.

<p align="center"><img src="screenshots/Screenshot_213.png" alt="Screenshot 213" width="820"></p>
*Figure 12. Final repository and deployment verification evidence.*

The final Kubernetes state was confirmed with commands equivalent to:

```bash
kubectl get application support-chat-prod -n argocd
kubectl get pods -n support-chat-prod
kubectl get deployment -n support-chat-prod
```

The final result was:

```text
support-chat-prod   prod   Synced   Healthy

support-chat-backend-...    shiftrobin/support-chat-backend:prod-fe78d34    Running
support-chat-frontend-...   shiftrobin/support-chat-frontend:prod-fe78d34   Running
```

---

## 13. CI/CD and GitOps Flow

My final deployment flow can be summarized as follows:

```mermaid
flowchart LR
    A[GitHub Repository] --> B[Pull Request / Push]
    B --> C[GitHub Actions CI]
    C --> D[Docker Build]
    D --> E[Docker Hub]
    E --> F[Versioned Images]
    A --> G[Argo CD]
    G --> H[Kubernetes]
    F --> H
    H --> I[Running Support Chat Pods]
```

The important point is that the Docker image version and Kubernetes manifest are both traceable to the Git workflow.

<p align="center"><img src="screenshots/Screenshot_207.png" alt="Screenshot 207" width="820"></p>
*Figure 13. Successful CI/CD workflow after the production image build and push implementation.*

---

## 14. Security and Secrets

I did not put Docker Hub credentials directly inside the workflow file. The workflow reads the credentials from GitHub Actions secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

The workflow therefore logs in to Docker Hub without exposing the actual credential values in the repository.

I also kept `.env.example` files in the application directories so that environment variable structure can be documented without committing the real local `.env` values as part of the deployment configuration.

---

## 15. Pull Requests and Change Management

I used pull requests for the major changes instead of directly editing the long-lived environment branches. Examples included adding the frontend Dockerfile, adding Docker build and push automation, adding production Kubernetes manifests, and updating production image tags.

This gave me a clear history of how the project evolved. After successful merges, I removed the temporary feature/chore branches.

<p align="center"><img src="screenshots/Screenshot_210.png" alt="Screenshot 210" width="820"></p>
*Figure 14. Production image update pull request with successful checks and no merge conflicts.*

---

## 16. Troubleshooting and Problems I Faced

During the implementation I faced a few practical problems and solved them step by step.

### Frontend Dockerfile was initially missing

The backend Dockerfile was available, but the frontend Dockerfile was not present on the required branches. I created the frontend Dockerfile and merged it through pull requests. I then verified that both `backend/Dockerfile` and `frontend/Dockerfile` existed on the development and production branches.

### Docker workflow needed correction

While editing the GitHub Actions workflow, I accidentally produced an incomplete/duplicated Docker section. I checked the file with `tail`, `git diff`, and `git diff --check`, corrected the workflow, and verified that the final Docker job contained one backend build step and one frontend build step.

### Production was still using the old image

After the new production Docker images were pushed, the Kubernetes manifests still referenced the previous tag. I updated the production manifests from `prod-7d684b8` to `prod-fe78d34`, merged the change into `prod`, refreshed Argo CD, and manually synchronized the production application.

The final Pods confirmed that the new image was actually running.

### Branch cleanup

After the required pull requests were merged, I deleted the temporary `chore/*` branches locally and remotely and ran `git fetch --prune`. The repository was left with only `main`, `dev`, `stage`, and `prod`.

---

## 17. Final Repository State

At the end of the implementation, the repository was clean and synchronized with the remote `prod` branch.

```text
main
dev
stage
prod
```

The final production GitOps state was also verified as:

```text
Application: support-chat-prod
Revision:    prod
Sync:        Synced
Health:      Healthy
Auto-sync:   Enabled
```

And the production Pods were running the same versioned image tag:

```text
backend  -> prod-fe78d34
frontend -> prod-fe78d34
```

---

## 18. What I Learned

Through this project I learned how the different DevOps components connect together rather than treating them as separate tools. I practiced Git branching and pull requests, CI validation with GitHub Actions, Docker image creation and versioning, Docker Hub publishing, Kubernetes deployment, and Argo CD GitOps synchronization.

The most useful part for me was the final production verification. It showed me that a successful GitHub Actions run alone does not prove that the new application version is running in Kubernetes. I therefore checked Argo CD, Deployments, Pods, and image tags directly.

---

## 19. Conclusion

I completed the Support Chat DevOps implementation with a Git-based development workflow, automated CI validation, Docker image build and push, Kubernetes manifests, and Argo CD-based deployment management.

The final production environment was successfully synchronized and healthy, and both backend and frontend Pods were running the intended `prod-fe78d34` Docker images. I also cleaned the temporary branches and verified that the repository was in a clean final state.

This project gave me practical experience in building a complete DevOps delivery path from source code to a running Kubernetes workload.

---

# 20. Screenshot Appendix

The following section contains the remaining screenshots captured during the implementation. The selected evidence screenshots are already placed in the relevant sections above. I kept the rest here as supporting evidence and project history.

<details>
<summary><strong>Show all remaining screenshots</strong></summary>

<table><tr><td align="center"><img src="screenshots/Screenshot_1.png" alt="Screenshot 1" width="300"><br><sub>Screenshot 1</sub></td><td align="center"><img src="screenshots/Screenshot_2.png" alt="Screenshot 2" width="300"><br><sub>Screenshot 2</sub></td><td align="center"><img src="screenshots/Screenshot_3.png" alt="Screenshot 3" width="300"><br><sub>Screenshot 3</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_4.png" alt="Screenshot 4" width="300"><br><sub>Screenshot 4</sub></td><td align="center"><img src="screenshots/Screenshot_5.png" alt="Screenshot 5" width="300"><br><sub>Screenshot 5</sub></td><td align="center"><img src="screenshots/Screenshot_6.png" alt="Screenshot 6" width="300"><br><sub>Screenshot 6</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_7.png" alt="Screenshot 7" width="300"><br><sub>Screenshot 7</sub></td><td align="center"><img src="screenshots/Screenshot_8.png" alt="Screenshot 8" width="300"><br><sub>Screenshot 8</sub></td><td align="center"><img src="screenshots/Screenshot_9.png" alt="Screenshot 9" width="300"><br><sub>Screenshot 9</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_10.png" alt="Screenshot 10" width="300"><br><sub>Screenshot 10</sub></td><td align="center"><img src="screenshots/Screenshot_11.png" alt="Screenshot 11" width="300"><br><sub>Screenshot 11</sub></td><td align="center"><img src="screenshots/Screenshot_12.png" alt="Screenshot 12" width="300"><br><sub>Screenshot 12</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_13.png" alt="Screenshot 13" width="300"><br><sub>Screenshot 13</sub></td><td align="center"><img src="screenshots/Screenshot_14.png" alt="Screenshot 14" width="300"><br><sub>Screenshot 14</sub></td><td align="center"><img src="screenshots/Screenshot_15.png" alt="Screenshot 15" width="300"><br><sub>Screenshot 15</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_16.png" alt="Screenshot 16" width="300"><br><sub>Screenshot 16</sub></td><td align="center"><img src="screenshots/Screenshot_17.png" alt="Screenshot 17" width="300"><br><sub>Screenshot 17</sub></td><td align="center"><img src="screenshots/Screenshot_18.png" alt="Screenshot 18" width="300"><br><sub>Screenshot 18</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_19.png" alt="Screenshot 19" width="300"><br><sub>Screenshot 19</sub></td><td align="center"><img src="screenshots/Screenshot_20.png" alt="Screenshot 20" width="300"><br><sub>Screenshot 20</sub></td><td align="center"><img src="screenshots/Screenshot_21.png" alt="Screenshot 21" width="300"><br><sub>Screenshot 21</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_22.png" alt="Screenshot 22" width="300"><br><sub>Screenshot 22</sub></td><td align="center"><img src="screenshots/Screenshot_23.png" alt="Screenshot 23" width="300"><br><sub>Screenshot 23</sub></td><td align="center"><img src="screenshots/Screenshot_24.png" alt="Screenshot 24" width="300"><br><sub>Screenshot 24</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_25.png" alt="Screenshot 25" width="300"><br><sub>Screenshot 25</sub></td><td align="center"><img src="screenshots/Screenshot_26.png" alt="Screenshot 26" width="300"><br><sub>Screenshot 26</sub></td><td align="center"><img src="screenshots/Screenshot_27.png" alt="Screenshot 27" width="300"><br><sub>Screenshot 27</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_28.png" alt="Screenshot 28" width="300"><br><sub>Screenshot 28</sub></td><td align="center"><img src="screenshots/Screenshot_29.png" alt="Screenshot 29" width="300"><br><sub>Screenshot 29</sub></td><td align="center"><img src="screenshots/Screenshot_30.png" alt="Screenshot 30" width="300"><br><sub>Screenshot 30</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_31.png" alt="Screenshot 31" width="300"><br><sub>Screenshot 31</sub></td><td align="center"><img src="screenshots/Screenshot_32.png" alt="Screenshot 32" width="300"><br><sub>Screenshot 32</sub></td><td align="center"><img src="screenshots/Screenshot_33.png" alt="Screenshot 33" width="300"><br><sub>Screenshot 33</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_34.png" alt="Screenshot 34" width="300"><br><sub>Screenshot 34</sub></td><td align="center"><img src="screenshots/Screenshot_35.png" alt="Screenshot 35" width="300"><br><sub>Screenshot 35</sub></td><td align="center"><img src="screenshots/Screenshot_36.png" alt="Screenshot 36" width="300"><br><sub>Screenshot 36</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_37.png" alt="Screenshot 37" width="300"><br><sub>Screenshot 37</sub></td><td align="center"><img src="screenshots/Screenshot_38.png" alt="Screenshot 38" width="300"><br><sub>Screenshot 38</sub></td><td align="center"><img src="screenshots/Screenshot_39.png" alt="Screenshot 39" width="300"><br><sub>Screenshot 39</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_40.png" alt="Screenshot 40" width="300"><br><sub>Screenshot 40</sub></td><td align="center"><img src="screenshots/Screenshot_41.png" alt="Screenshot 41" width="300"><br><sub>Screenshot 41</sub></td><td align="center"><img src="screenshots/Screenshot_42.png" alt="Screenshot 42" width="300"><br><sub>Screenshot 42</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_43.png" alt="Screenshot 43" width="300"><br><sub>Screenshot 43</sub></td><td align="center"><img src="screenshots/Screenshot_44.png" alt="Screenshot 44" width="300"><br><sub>Screenshot 44</sub></td><td align="center"><img src="screenshots/Screenshot_45.png" alt="Screenshot 45" width="300"><br><sub>Screenshot 45</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_46.png" alt="Screenshot 46" width="300"><br><sub>Screenshot 46</sub></td><td align="center"><img src="screenshots/Screenshot_47.png" alt="Screenshot 47" width="300"><br><sub>Screenshot 47</sub></td><td align="center"><img src="screenshots/Screenshot_48.png" alt="Screenshot 48" width="300"><br><sub>Screenshot 48</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_49.png" alt="Screenshot 49" width="300"><br><sub>Screenshot 49</sub></td><td align="center"><img src="screenshots/Screenshot_50.png" alt="Screenshot 50" width="300"><br><sub>Screenshot 50</sub></td><td align="center"><img src="screenshots/Screenshot_51.png" alt="Screenshot 51" width="300"><br><sub>Screenshot 51</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_52.png" alt="Screenshot 52" width="300"><br><sub>Screenshot 52</sub></td><td align="center"><img src="screenshots/Screenshot_53.png" alt="Screenshot 53" width="300"><br><sub>Screenshot 53</sub></td><td align="center"><img src="screenshots/Screenshot_54.png" alt="Screenshot 54" width="300"><br><sub>Screenshot 54</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_55.png" alt="Screenshot 55" width="300"><br><sub>Screenshot 55</sub></td><td align="center"><img src="screenshots/Screenshot_56.png" alt="Screenshot 56" width="300"><br><sub>Screenshot 56</sub></td><td align="center"><img src="screenshots/Screenshot_57.png" alt="Screenshot 57" width="300"><br><sub>Screenshot 57</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_58.png" alt="Screenshot 58" width="300"><br><sub>Screenshot 58</sub></td><td align="center"><img src="screenshots/Screenshot_59.png" alt="Screenshot 59" width="300"><br><sub>Screenshot 59</sub></td><td align="center"><img src="screenshots/Screenshot_60.png" alt="Screenshot 60" width="300"><br><sub>Screenshot 60</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_61.png" alt="Screenshot 61" width="300"><br><sub>Screenshot 61</sub></td><td align="center"><img src="screenshots/Screenshot_62.png" alt="Screenshot 62" width="300"><br><sub>Screenshot 62</sub></td><td align="center"><img src="screenshots/Screenshot_63.png" alt="Screenshot 63" width="300"><br><sub>Screenshot 63</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_64.png" alt="Screenshot 64" width="300"><br><sub>Screenshot 64</sub></td><td align="center"><img src="screenshots/Screenshot_65.png" alt="Screenshot 65" width="300"><br><sub>Screenshot 65</sub></td><td align="center"><img src="screenshots/Screenshot_66.png" alt="Screenshot 66" width="300"><br><sub>Screenshot 66</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_67.png" alt="Screenshot 67" width="300"><br><sub>Screenshot 67</sub></td><td align="center"><img src="screenshots/Screenshot_68.png" alt="Screenshot 68" width="300"><br><sub>Screenshot 68</sub></td><td align="center"><img src="screenshots/Screenshot_69.png" alt="Screenshot 69" width="300"><br><sub>Screenshot 69</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_70.png" alt="Screenshot 70" width="300"><br><sub>Screenshot 70</sub></td><td align="center"><img src="screenshots/Screenshot_71.png" alt="Screenshot 71" width="300"><br><sub>Screenshot 71</sub></td><td align="center"><img src="screenshots/Screenshot_72.png" alt="Screenshot 72" width="300"><br><sub>Screenshot 72</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_73.png" alt="Screenshot 73" width="300"><br><sub>Screenshot 73</sub></td><td align="center"><img src="screenshots/Screenshot_75.png" alt="Screenshot 75" width="300"><br><sub>Screenshot 75</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_76.png" alt="Screenshot 76" width="300"><br><sub>Screenshot 76</sub></td><td align="center"><img src="screenshots/Screenshot_77.png" alt="Screenshot 77" width="300"><br><sub>Screenshot 77</sub></td><td align="center"><img src="screenshots/Screenshot_78.png" alt="Screenshot 78" width="300"><br><sub>Screenshot 78</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_79.png" alt="Screenshot 79" width="300"><br><sub>Screenshot 79</sub></td><td align="center"><img src="screenshots/Screenshot_80.png" alt="Screenshot 80" width="300"><br><sub>Screenshot 80</sub></td><td align="center"><img src="screenshots/Screenshot_81.png" alt="Screenshot 81" width="300"><br><sub>Screenshot 81</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_82.png" alt="Screenshot 82" width="300"><br><sub>Screenshot 82</sub></td><td align="center"><img src="screenshots/Screenshot_83.png" alt="Screenshot 83" width="300"><br><sub>Screenshot 83</sub></td><td align="center"><img src="screenshots/Screenshot_84.png" alt="Screenshot 84" width="300"><br><sub>Screenshot 84</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_85.png" alt="Screenshot 85" width="300"><br><sub>Screenshot 85</sub></td><td align="center"><img src="screenshots/Screenshot_86.png" alt="Screenshot 86" width="300"><br><sub>Screenshot 86</sub></td><td align="center"><img src="screenshots/Screenshot_87.png" alt="Screenshot 87" width="300"><br><sub>Screenshot 87</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_88.png" alt="Screenshot 88" width="300"><br><sub>Screenshot 88</sub></td><td align="center"><img src="screenshots/Screenshot_89.png" alt="Screenshot 89" width="300"><br><sub>Screenshot 89</sub></td><td align="center"><img src="screenshots/Screenshot_90.png" alt="Screenshot 90" width="300"><br><sub>Screenshot 90</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_91.png" alt="Screenshot 91" width="300"><br><sub>Screenshot 91</sub></td><td align="center"><img src="screenshots/Screenshot_92.png" alt="Screenshot 92" width="300"><br><sub>Screenshot 92</sub></td><td align="center"><img src="screenshots/Screenshot_93.png" alt="Screenshot 93" width="300"><br><sub>Screenshot 93</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_94.png" alt="Screenshot 94" width="300"><br><sub>Screenshot 94</sub></td><td align="center"><img src="screenshots/Screenshot_95.png" alt="Screenshot 95" width="300"><br><sub>Screenshot 95</sub></td><td align="center"><img src="screenshots/Screenshot_96.png" alt="Screenshot 96" width="300"><br><sub>Screenshot 96</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_97.png" alt="Screenshot 97" width="300"><br><sub>Screenshot 97</sub></td><td align="center"><img src="screenshots/Screenshot_98.png" alt="Screenshot 98" width="300"><br><sub>Screenshot 98</sub></td><td align="center"><img src="screenshots/Screenshot_99.png" alt="Screenshot 99" width="300"><br><sub>Screenshot 99</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_100.png" alt="Screenshot 100" width="300"><br><sub>Screenshot 100</sub></td><td align="center"><img src="screenshots/Screenshot_101.png" alt="Screenshot 101" width="300"><br><sub>Screenshot 101</sub></td><td align="center"><img src="screenshots/Screenshot_102.png" alt="Screenshot 102" width="300"><br><sub>Screenshot 102</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_103.png" alt="Screenshot 103" width="300"><br><sub>Screenshot 103</sub></td><td align="center"><img src="screenshots/Screenshot_104.png" alt="Screenshot 104" width="300"><br><sub>Screenshot 104</sub></td><td align="center"><img src="screenshots/Screenshot_105.png" alt="Screenshot 105" width="300"><br><sub>Screenshot 105</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_106.png" alt="Screenshot 106" width="300"><br><sub>Screenshot 106</sub></td><td align="center"><img src="screenshots/Screenshot_107.png" alt="Screenshot 107" width="300"><br><sub>Screenshot 107</sub></td><td align="center"><img src="screenshots/Screenshot_108.png" alt="Screenshot 108" width="300"><br><sub>Screenshot 108</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_109.png" alt="Screenshot 109" width="300"><br><sub>Screenshot 109</sub></td><td align="center"><img src="screenshots/Screenshot_110.png" alt="Screenshot 110" width="300"><br><sub>Screenshot 110</sub></td><td align="center"><img src="screenshots/Screenshot_111.png" alt="Screenshot 111" width="300"><br><sub>Screenshot 111</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_112.png" alt="Screenshot 112" width="300"><br><sub>Screenshot 112</sub></td><td align="center"><img src="screenshots/Screenshot_113.png" alt="Screenshot 113" width="300"><br><sub>Screenshot 113</sub></td><td align="center"><img src="screenshots/Screenshot_114.png" alt="Screenshot 114" width="300"><br><sub>Screenshot 114</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_115.png" alt="Screenshot 115" width="300"><br><sub>Screenshot 115</sub></td><td align="center"><img src="screenshots/Screenshot_116.png" alt="Screenshot 116" width="300"><br><sub>Screenshot 116</sub></td><td align="center"><img src="screenshots/Screenshot_117.png" alt="Screenshot 117" width="300"><br><sub>Screenshot 117</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_118.png" alt="Screenshot 118" width="300"><br><sub>Screenshot 118</sub></td><td align="center"><img src="screenshots/Screenshot_119.png" alt="Screenshot 119" width="300"><br><sub>Screenshot 119</sub></td><td align="center"><img src="screenshots/Screenshot_120.png" alt="Screenshot 120" width="300"><br><sub>Screenshot 120</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_121.png" alt="Screenshot 121" width="300"><br><sub>Screenshot 121</sub></td><td align="center"><img src="screenshots/Screenshot_122.png" alt="Screenshot 122" width="300"><br><sub>Screenshot 122</sub></td><td align="center"><img src="screenshots/Screenshot_123.png" alt="Screenshot 123" width="300"><br><sub>Screenshot 123</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_124.png" alt="Screenshot 124" width="300"><br><sub>Screenshot 124</sub></td><td align="center"><img src="screenshots/Screenshot_125.png" alt="Screenshot 125" width="300"><br><sub>Screenshot 125</sub></td><td align="center"><img src="screenshots/Screenshot_126.png" alt="Screenshot 126" width="300"><br><sub>Screenshot 126</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_127.png" alt="Screenshot 127" width="300"><br><sub>Screenshot 127</sub></td><td align="center"><img src="screenshots/Screenshot_129.png" alt="Screenshot 129" width="300"><br><sub>Screenshot 129</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_130.png" alt="Screenshot 130" width="300"><br><sub>Screenshot 130</sub></td><td align="center"><img src="screenshots/Screenshot_131.png" alt="Screenshot 131" width="300"><br><sub>Screenshot 131</sub></td><td align="center"><img src="screenshots/Screenshot_132.png" alt="Screenshot 132" width="300"><br><sub>Screenshot 132</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_133.png" alt="Screenshot 133" width="300"><br><sub>Screenshot 133</sub></td><td align="center"><img src="screenshots/Screenshot_134.png" alt="Screenshot 134" width="300"><br><sub>Screenshot 134</sub></td><td align="center"><img src="screenshots/Screenshot_135.png" alt="Screenshot 135" width="300"><br><sub>Screenshot 135</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_136.png" alt="Screenshot 136" width="300"><br><sub>Screenshot 136</sub></td><td align="center"><img src="screenshots/Screenshot_137.png" alt="Screenshot 137" width="300"><br><sub>Screenshot 137</sub></td><td align="center"><img src="screenshots/Screenshot_138.png" alt="Screenshot 138" width="300"><br><sub>Screenshot 138</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_139.png" alt="Screenshot 139" width="300"><br><sub>Screenshot 139</sub></td><td align="center"><img src="screenshots/Screenshot_140.png" alt="Screenshot 140" width="300"><br><sub>Screenshot 140</sub></td><td align="center"><img src="screenshots/Screenshot_141.png" alt="Screenshot 141" width="300"><br><sub>Screenshot 141</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_142.png" alt="Screenshot 142" width="300"><br><sub>Screenshot 142</sub></td><td align="center"><img src="screenshots/Screenshot_143.png" alt="Screenshot 143" width="300"><br><sub>Screenshot 143</sub></td><td align="center"><img src="screenshots/Screenshot_144.png" alt="Screenshot 144" width="300"><br><sub>Screenshot 144</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_145.png" alt="Screenshot 145" width="300"><br><sub>Screenshot 145</sub></td><td align="center"><img src="screenshots/Screenshot_146.png" alt="Screenshot 146" width="300"><br><sub>Screenshot 146</sub></td><td align="center"><img src="screenshots/Screenshot_147.png" alt="Screenshot 147" width="300"><br><sub>Screenshot 147</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_148.png" alt="Screenshot 148" width="300"><br><sub>Screenshot 148</sub></td><td align="center"><img src="screenshots/Screenshot_149.png" alt="Screenshot 149" width="300"><br><sub>Screenshot 149</sub></td><td align="center"><img src="screenshots/Screenshot_150.png" alt="Screenshot 150" width="300"><br><sub>Screenshot 150</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_151.png" alt="Screenshot 151" width="300"><br><sub>Screenshot 151</sub></td><td align="center"><img src="screenshots/Screenshot_152.png" alt="Screenshot 152" width="300"><br><sub>Screenshot 152</sub></td><td align="center"><img src="screenshots/Screenshot_153.png" alt="Screenshot 153" width="300"><br><sub>Screenshot 153</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_154.png" alt="Screenshot 154" width="300"><br><sub>Screenshot 154</sub></td><td align="center"><img src="screenshots/Screenshot_155.png" alt="Screenshot 155" width="300"><br><sub>Screenshot 155</sub></td><td align="center"><img src="screenshots/Screenshot_156.png" alt="Screenshot 156" width="300"><br><sub>Screenshot 156</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_157.png" alt="Screenshot 157" width="300"><br><sub>Screenshot 157</sub></td><td align="center"><img src="screenshots/Screenshot_158.png" alt="Screenshot 158" width="300"><br><sub>Screenshot 158</sub></td><td align="center"><img src="screenshots/Screenshot_159.png" alt="Screenshot 159" width="300"><br><sub>Screenshot 159</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_160.png" alt="Screenshot 160" width="300"><br><sub>Screenshot 160</sub></td><td align="center"><img src="screenshots/Screenshot_161.png" alt="Screenshot 161" width="300"><br><sub>Screenshot 161</sub></td><td align="center"><img src="screenshots/Screenshot_162.png" alt="Screenshot 162" width="300"><br><sub>Screenshot 162</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_163.png" alt="Screenshot 163" width="300"><br><sub>Screenshot 163</sub></td><td align="center"><img src="screenshots/Screenshot_164.png" alt="Screenshot 164" width="300"><br><sub>Screenshot 164</sub></td><td align="center"><img src="screenshots/Screenshot_165.png" alt="Screenshot 165" width="300"><br><sub>Screenshot 165</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_166.png" alt="Screenshot 166" width="300"><br><sub>Screenshot 166</sub></td><td align="center"><img src="screenshots/Screenshot_167.png" alt="Screenshot 167" width="300"><br><sub>Screenshot 167</sub></td><td align="center"><img src="screenshots/Screenshot_168.png" alt="Screenshot 168" width="300"><br><sub>Screenshot 168</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_169.png" alt="Screenshot 169" width="300"><br><sub>Screenshot 169</sub></td><td align="center"><img src="screenshots/Screenshot_170.png" alt="Screenshot 170" width="300"><br><sub>Screenshot 170</sub></td><td align="center"><img src="screenshots/Screenshot_171.png" alt="Screenshot 171" width="300"><br><sub>Screenshot 171</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_172.png" alt="Screenshot 172" width="300"><br><sub>Screenshot 172</sub></td><td align="center"><img src="screenshots/Screenshot_173.png" alt="Screenshot 173" width="300"><br><sub>Screenshot 173</sub></td><td align="center"><img src="screenshots/Screenshot_174.png" alt="Screenshot 174" width="300"><br><sub>Screenshot 174</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_175.png" alt="Screenshot 175" width="300"><br><sub>Screenshot 175</sub></td><td align="center"><img src="screenshots/Screenshot_176.png" alt="Screenshot 176" width="300"><br><sub>Screenshot 176</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_178.png" alt="Screenshot 178" width="300"><br><sub>Screenshot 178</sub></td><td align="center"><img src="screenshots/Screenshot_179.png" alt="Screenshot 179" width="300"><br><sub>Screenshot 179</sub></td><td align="center"><img src="screenshots/Screenshot_180.png" alt="Screenshot 180" width="300"><br><sub>Screenshot 180</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_181.png" alt="Screenshot 181" width="300"><br><sub>Screenshot 181</sub></td><td align="center"><img src="screenshots/Screenshot_182.png" alt="Screenshot 182" width="300"><br><sub>Screenshot 182</sub></td><td align="center"><img src="screenshots/Screenshot_183.png" alt="Screenshot 183" width="300"><br><sub>Screenshot 183</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_184.png" alt="Screenshot 184" width="300"><br><sub>Screenshot 184</sub></td><td align="center"><img src="screenshots/Screenshot_185.png" alt="Screenshot 185" width="300"><br><sub>Screenshot 185</sub></td><td align="center"><img src="screenshots/Screenshot_186.png" alt="Screenshot 186" width="300"><br><sub>Screenshot 186</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_187.png" alt="Screenshot 187" width="300"><br><sub>Screenshot 187</sub></td><td align="center"><img src="screenshots/Screenshot_188.png" alt="Screenshot 188" width="300"><br><sub>Screenshot 188</sub></td><td align="center"><img src="screenshots/Screenshot_189.png" alt="Screenshot 189" width="300"><br><sub>Screenshot 189</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_190.png" alt="Screenshot 190" width="300"><br><sub>Screenshot 190</sub></td><td align="center"><img src="screenshots/Screenshot_192.png" alt="Screenshot 192" width="300"><br><sub>Screenshot 192</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_193.png" alt="Screenshot 193" width="300"><br><sub>Screenshot 193</sub></td><td align="center"><img src="screenshots/Screenshot_194.png" alt="Screenshot 194" width="300"><br><sub>Screenshot 194</sub></td><td align="center"><img src="screenshots/Screenshot_195.png" alt="Screenshot 195" width="300"><br><sub>Screenshot 195</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_196.png" alt="Screenshot 196" width="300"><br><sub>Screenshot 196</sub></td><td align="center"><img src="screenshots/Screenshot_197.png" alt="Screenshot 197" width="300"><br><sub>Screenshot 197</sub></td><td align="center"><img src="screenshots/Screenshot_198.png" alt="Screenshot 198" width="300"><br><sub>Screenshot 198</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_199.png" alt="Screenshot 199" width="300"><br><sub>Screenshot 199</sub></td><td align="center"><img src="screenshots/Screenshot_200.png" alt="Screenshot 200" width="300"><br><sub>Screenshot 200</sub></td><td align="center"><img src="screenshots/Screenshot_201.png" alt="Screenshot 201" width="300"><br><sub>Screenshot 201</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_204.png" alt="Screenshot 204" width="300"><br><sub>Screenshot 204</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_205.png" alt="Screenshot 205" width="300"><br><sub>Screenshot 205</sub></td><td align="center"><img src="screenshots/Screenshot_206.png" alt="Screenshot 206" width="300"><br><sub>Screenshot 206</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_208.png" alt="Screenshot 208" width="300"><br><sub>Screenshot 208</sub></td><td align="center"><img src="screenshots/Screenshot_209.png" alt="Screenshot 209" width="300"><br><sub>Screenshot 209</sub></td></tr></table>
<table><tr><td align="center"><img src="screenshots/Screenshot_211.png" alt="Screenshot 211" width="300"><br><sub>Screenshot 211</sub></td><td align="center"><img src="screenshots/Screenshot_212.png" alt="Screenshot 212" width="300"><br><sub>Screenshot 212</sub></td></tr></table>
</details>
