# cc-kustomize
Deploying AquaSec Cyber Center using Kustomize

```bash
  kustomize build --enable-helm ./dev/aqua00/aquasec-core/
```
argocd-image-updater.argoproj.io/image-list: some/image[:<version_constraint>]
# Specifying update-strategy is optional, because semver is the default
argocd-image-updater.argoproj.io/<image>.update-strategy: semver


argocd-image-updater.argoproj.io/cc-premium





argocd-image-updater.argoproj.io/cc.force-update: true
argocd-image-updater.argoproj.io/cc.pull-secret: pullsecret:argocd/aqua-registry
argocd-image-updater.argoproj.io/image-list: cc=registry.aquasec.com/cc-premium
argocd-image-updater.argoproj.io/write-back-method: argocd
