# argo2yaml

Render out all of the manifests defined under an argocd App of App pattern.

## Reason
As more and more applications depend on shared manifests, it becomes increasingly difficult to be certian that changes wont affect other applications negatively. 
This script aims to give visibility into what changes will take place when override files or charts are updated using ArgoCD's built in helm templating configuration.

## Use:
In the root of your project directory. Point argo2yaml at one of your top level CRD's that begins your AOA tree.

### ex 1) Rendering out manifests from a single CRD definition
```YAML
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample1
  namespace: argocd
spec:
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  project: default
  source:
    helm:
      valueFiles:
        - ../../overrides/sample.yaml
    path: charts/test-aoa
    repoURL: https://github.com/sample-repo/argocd-configs
    targetRevision: HEAD
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
```
```SHELL
$ argo2yaml -crd test_crds/crd-single.yaml
INFO[0000] helm template . --name-template sample1 --namespace argocd --values /Users/USER/Documents/repos/personal/argo2yaml/overrides/sample.yaml --include-crds  dir=charts/test-aoa execID=d7b89
INFO[0000] Trace                                         args="[helm template . --name-template sample1 --namespace argocd --values /Users/USER/Documents/repos/personal/argo2yaml/overrides/sample.yaml --include-crds]" dir=charts/test-aoa operation_name="exec helm" time_ms=749.043
INFO[0000] helm template . --name-template testing-App --namespace argocd --values /Users/USER/Documents/repos/personal/argo2yaml/overrides/sample-aoa.yaml --include-crds  dir=charts/test-chart execID=fcdfa
INFO[0001] Trace                                         args="[helm template . --name-template testing-App --namespace argocd --values /Users/USER/Documents/repos/personal/argo2yaml/overrides/sample-aoa.yaml --include-crds]" dir=charts/test-chart operation_name="exec helm" time_ms=719.490167
```

### ex 2) Rendering manifest from files with multiple CRDs
```YAML
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample1
  namespace: argocd
spec:
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  project: default
  source:
    helm:
      valueFiles:
        - ../../overrides/sample.yaml
    path: charts/test-aoa
    repoURL: https://github.com/test-repo/argocd-configs
    targetRevision: HEAD
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample2
  namespace: argocd
spec:
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  project: default
  source:
    helm:
      valueFiles:
        - ../../overrides/sample.yaml
    path: charts/test-aoa
    repoURL: https://github.com/test-repo/argocd-configs
    targetRevision: HEAD
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
```
```SHELL
$ argo2yaml -crd test_crds/crd-multiple.yaml
INFO[0000] helm template . --name-template sample1 --namespace argocd --values /Users/marmanse1219/Documents/repos/personal/argo2yaml/overrides/sample.yaml --include-crds  dir=charts/test-aoa execID=e58df
INFO[0000] Trace                                         args="[helm template . --name-template sample1 --namespace argocd --values /Users/marmanse1219/Documents/repos/personal/argo2yaml/overrides/sample.yaml --include-crds]" dir=charts/test-aoa operation_name="exec helm" time_ms=762.33075
INFO[0000] helm template . --name-template testing-App --namespace argocd --values /Users/marmanse1219/Documents/repos/personal/argo2yaml/overrides/sample-aoa.yaml --include-crds  dir=charts/test-chart execID=2bc03
INFO[0001] Trace                                         args="[helm template . --name-template testing-App --namespace argocd --values /Users/marmanse1219/Documents/repos/personal/argo2yaml/overrides/sample-aoa.yaml --include-crds]" dir=charts/test-chart operation_name="exec helm" time_ms=733.955958
INFO[0001] helm template . --name-template sample2 --namespace argocd --values /Users/marmanse1219/Documents/repos/personal/argo2yaml/overrides/sample.yaml --include-crds  dir=charts/test-aoa execID=9e695
INFO[0002] Trace                                         args="[helm template . --name-template sample2 --namespace argocd --values /Users/marmanse1219/Documents/repos/personal/argo2yaml/overrides/sample.yaml --include-crds]" dir=charts/test-aoa operation_name="exec helm" time_ms=722.203667
INFO[0002] helm template . --name-template testing-App --namespace argocd --values /Users/marmanse1219/Documents/repos/personal/argo2yaml/overrides/sample-aoa.yaml --include-crds  dir=charts/test-chart execID=ee9a9
INFO[0003] Trace                                         args="[helm template . --name-template testing-App --namespace argocd --values /Users/marmanse1219/Documents/repos/personal/argo2yaml/overrides/sample-aoa.yaml --include-crds]" dir=charts/test-chart operation_name="exec helm" time_ms=722.61325
```

This will render all the manifest to the projects ~/zz.autogenerated directory. These files can be included in git history to see the extent of your changes. They may also be used in conjunction with ArgoCD render paths, so you can controll application syncs better.

```SHELL
$ tree zz.autogenerated
zz.autogenerated
├── sample1
│   └── manifest.yaml
├── sample2
│   └── manifest.yaml
└── testing-App
    └── manifest.yaml

4 directories, 3 files
```
