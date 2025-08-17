# Java → AWS EKS via Harness CI/CD (Public GitHub Repo)

This repo contains a minimal Spring Boot app, Dockerfile, Kubernetes manifests, and a Harness pipeline to:

1. Build a JAR with Maven
2. Build & push a Docker image to Amazon ECR
3. Deploy the image to an AWS EKS cluster (K8s rolling deploy)

## Repo Layout
```text
.
├── src/main/java/com/example/demo/DemoApplication.java
├── pom.xml
├── Dockerfile
├── k8s/deployment.yaml
└── .harness/pipeline.yaml (Ashley: removed. Created pipeline myself in harness)
```

## Prereqs
- **AWS ECR** repo created (e.g., `java-eks-demo`)
- **EKS cluster** reachable by Harness
- **Harness connectors**
  - `github_public_repo` → points to this GitHub repo (type: Git)
  - `aws_ecr_connector` → Docker Registry connector to your ECR registry (Ashley: I used dockerhub registry)
  - `aws_eks_connector` → Kubernetes connector to the EKS cluster (service account or IAM auth)

## Quick Start (Harness)
1. Push this repo to **GitHub (public)**.
2. In Harness, create the three connectors with the names above.
3. Create a **Pipeline** using `.harness/pipeline.yaml` (Git Experience).
4. Run the pipeline with:
   - **Codebase**: Branch = `main`
5. Watch the CI stage build & push `:<sequenceId>` to ECR, then the CD stage deploy to EKS.

## Common Gotchas
- Update `repoName` under both `properties.ci.codebase` and the K8s manifest `store` to your real GitHub owner and repo.
- Update `repo` under `Build & Push to ECR` to your real ECR repo URI.
- Ensure your `aws_eks_connector` has cluster access and the `default` namespace exists, or change it in `pipeline.yaml`.

## Local Build (optional)
```bash
mvn -DskipTests clean package
docker build -t java-eks-demo:dev .
```

## Test after deploy
Get the ELB hostname from the `Service`:
```bash
kubectl get svc demo-service -o wide
curl http://<external-ip-or-hostname>/
```
