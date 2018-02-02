title: My presentation
output: presentation.html
theme: sudodoki/reveal-cleaver-theme

---

# Managing Maglev with Helm

---

## Why?

---

### Maglev in multiple form factors
- Cloud, Dev, Appliance etc.,
    - Has resulted in multiple manifests for core addons
- Heavily templated. No single place of to see values.
- The manifests that go as part of the release are not stored anywhere. No easy way to recreate them (will need to re-run ansible provisioning)
- The orchestration logic to bring up maglev is tightly coupled with logic to create infrastructure, bring up k8s cluster etc.,
- Makes it harder to port to other form factors (managed k8s cloud, tectonic etc.,)

---

### Developer productivity
- When they need a new maglev version, they usually create a new cluster
- They will need to keep multiple clusters to keep older changes
- No mechanism to upgrade/downgrade in-place (system update works only for ISO)
- Changes to a core manifest is complex, requires a lot of time to test
- Need to validate on all form factors

---

### Complex CI/CD
- No single place to download core maglev manifests
- Update logic is currently complex (env variables are not preserved etc.,)
- No easy way to slice/dice/compose a maglev release
    - for ex. upgrades to stateful sets will be rare, and requires complex orchestration logic
    - But web/compute type services can/need to be upgraded more often

---

## What is helm?

Official package manager for k8s apps

---

- apt/yum/homebrew for k8s
- 2 parts. `helm` (client), `tiller` (server)
- k8s apps are organized as charts
    - charts can contain any valid k8s object (deployment, svc, configmap, sts etc.,)
- Useful constructs
    - Can express dependencies and ordering
    - Provides lifecycle hooks
        - pre-install, post-install, pre-upgrade, post-upgrade, pre-delete, post-delete

---

- Provides a notion of a `release`
    - can be composed of multiple charts
    - `releases` are stored as custom objects in k8s cluster (via tiller) 
    - can be upgraded/rollback
    
- Provides a `helm repository`
    - Charts can be distributed, versioned, fetched
    - similar to dockerhub but for k8s apps
    - several standard charts are already available (`helm install nginx`)
- Official k8s project
    - Actively maintained. Decent docs and support
    - Will support upstream k8s enhancements

---

## Approach

Maglev should come up on ANY k8s cluster
(irrespective of how they are created)

---

## Logical View

```
        +---------------------------------------------+
        |            Maglev Distribution (helm)       |
        |  +---------------+   +-----------------+    |
        |  |   managed     |   |     core addons |    | 
        |  |   services    |   |                 |    |
        |  |               |   |                 |    |
        |  +---------------+   +-----------------+    |
        +---------------------------------------------+
        +---------------------------------------------+
        |                    K8S Cluster              |
        |            kubeadm, tectonic, kops, managed |
        +---------------------------------------------+
        +---------------------------------------------+
        |               Infrastructure                | 
        |   (IaaS, VSphere, BareMetal + any)          |
        +-----------------------------+---------------+
```

---

# DEMO

---

## Opening new doors

---

### Composability

- Slice/Dice/Compose a maglev-distro.
- Ability to treat STS, stateless addons separately
- Moving forward:
    - Every core service can be its own chart/release
    - Teams that develop are responsible for those charts
    - Maglev release just has to express dependencies on them

---

### Portability

- Services can be made self sufficient
    - For ex. creation of pki certs for kong/cred/encryption mgr is a `pre-install` hook
    - Hooks can encapsulate complex orchestration logic
- Maglev will run on any k8s cluster
    - No ansible, `helm install`
    - Can easily swap our k8s providers
---

### Upgrades

- SystemUpdater just downloads a maglev-distro tarball
- `maglev-distro` can be another `kind` supported by catalogserver
- Can upgrade/rollback
- Changes to OS can be done by including a privileged pod/daemonset in the distro
- Pre-requesites required by newer version of services will be done by hooks
    - No ancillary image, loose ansible scripts

---

### Developer productivity

- Can selectively upgrade/downgrade
    - Dont wait for new ISO to install
    - Dont have to recreate dev clusters
- One place to change addons

---

## Next

- AWS deployments with helm
- ISO installer has to be ported
- Remove complex orchestration logic from ansible as much as possible
    - Kong API seeding (modelled as CRDs with custom controller)
    - Docker registry seeding (modelled as a configmap with a controller)
    - IDM resource creation (post-install hook)
    - Managed Services user creation (post-install hook)
    