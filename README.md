# Example Domain's Local Development Environment

A full environment, but `local: true` flags are passed to applications that you want to run locally - in this example, all of the microservices, and the ui, are configured to run in "local" mode. In this mode the chart will only create dependencies needed for running the service - like secrets, brokers, triggers, and databases - and leave you to run the services themselves locally. It is preconfigured to use `host.docker.internal` when running in local mode to access your local network.

From your local network you may want to access the Kubernetes network, for that use `localizer`.

```
sudo localizer --namespace example-local-env
```

This will create tunnels, and update your `/etc/hosts` for accessing those tunnels via the same URL that would be used internally – `http://<service-name>.<namespace>.svc.cluster.local`, so `http://example-hasura.example-local-env.svc.cluster.local` for example.

Create the following resource in your local gitops `/projects` directory

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: example-local-env
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "2"
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  description: Example Domain Production Environment
  sourceRepos:
  - '*'
  destinations:
  - namespace: '*'
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'
  namespaceResourceWhitelist:
  - group: '*'
    kind: '*'

---

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-local-env
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "3"
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: example-local-env
  source:
    repoURL: https://github.com/cloudnativeentrepreneur/example-local-env.git
    targetRevision: HEAD
    path: helm
  destination:
    server: https://kubernetes.default.svc
    namespace: example-local-env
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
    syncOptions:
    - CreateNamespace=true
```
