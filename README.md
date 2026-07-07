# gitops-platform

This repo is what ArgoCD watches. Nothing in the EKS cluster changes except
through a commit here (after the one-time bootstrap below).

## Structure

```
bootstrap/
  root-app.yaml        <- applied ONCE, manually. Everything after this is automatic.
apps/
  platform/             <- platform components (namespaces, later: cert-manager,
                            external-secrets, nginx-ingress, kyverno policies)
  workloads/             <- your actual applications (added in a later step)
clusters/
  namespaces.yaml       <- dev/staging/prod namespace definitions
```

## How the app-of-apps pattern works here

1. You apply `bootstrap/root-app.yaml` manually, once.
2. It tells ArgoCD: "watch the `apps/` folder in this repo, recursively."
3. ArgoCD finds `apps/platform/namespaces-app.yaml` inside it - which is
   itself an Application - and creates that too.
4. That second Application points at the `clusters/` folder, where it finds
   `namespaces.yaml` and creates the dev/staging/prod namespaces.
5. From here on: adding a new platform component (Step 12+) means adding one
   new Application YAML file under `apps/platform/`, committing, and pushing.
   No kubectl, ever, for anything this pattern covers.

## Push this to GitHub

```bash
cd gitops-platform
git init
git add .
git commit -m "Initial GitOps repo structure"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/gitops-platform.git
git push -u origin main
```

## Then apply the root Application (the one manual step, ever)

```bash
kubectl apply -f bootstrap/root-app.yaml
```

## Verify

In the ArgoCD UI, you should see:
- `root-app` - Synced, Healthy
- `namespaces` - appears automatically, Synced, Healthy

```bash
kubectl get namespaces
```
Should show `dev`, `staging`, `prod` now existing - created by ArgoCD, not
by you running `kubectl create namespace`.
