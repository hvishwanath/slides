title: maglev-x
output: index.html
theme: sjaakvandenberg/cleaver-light

---

# maglev-x
## x = .*
## Rethinking dev/ops workflow

---

# Why

---

Developing and testing maglev is hard

Releasing maglev is getting **harder**

More developers != more productivity

Simple things are NOT simple

---
**Microservices... but...**

- *We are micro services, but dont have any build/deployment advantages that come with it*
- *We release as a monolith*
- *Simple, localized fixes to be delivered requires a new build*

**Getting a maglev build out is a huge task**
- *Significant overhead with branches, release management*
- *Babysitting builds sap out energy*

---

**CI/CD tooling is not streamlined**
- *There is no specification of how this must happen*
- *Any activity (brancing, registry cleanup etc.,) in CI area will cause ripples almost in every area*

**Butterfly effect and blast radius**
- *Issue in one library/component will hold up the entire build*
- *No way to revert to previous known stable version of that component*

---

**Code layout is not uniform**
- *Source is somewhere, build is somewhere else, deployment is somewhere else*
- *Increased cognitive overload for new comers*
- *Requires significant context for anyone to be productive and deliver with confidence*

**Developer workflow**
- *Not uniform, Not straightforward*
- *Significant setup overhead to test features/fixes*

---

**Developer Agility**
- *Simple fixes take a LOT of time*
- *No clear interfaces/contracts between services*
- *Adding more engineers to the team can actually complicate this problem*

**Return of the monolith**
- *Current code organization doesn't facilitate writing tests for managed services*
- *Developers spend significant time testing manually, but all is lost and cannot be repeated*
- *Not conducive to focus on a single service/unit*
    - *For ex. a fix to redis config requires one to write sample package/appstack/managed_service_bundle*
    - *Cannot treat redis is a single unit and test its ha/failover/clustering aspects*
---

# Returning to first principles

---

**As a Developer, I - **

Want to develop, fix, iterate quickly

Want to test my fixes on the unit it is affecting without having to wait for hours

Want a very high confidence rate when I merge my PR 

Want to accumulate all my manual testing effort into code

Dont want to boil the ocean everytime I touch some code
 
Dont want to chase builds

---
**Versioning**
- *[SemVer2](https://semver.org/): Version numbers should follow conventions and indicate meaning*
- *Every independently deployable entity MUST carry its own version*
- *Can revert/checkout to a specific fix of a specific service*

**Micro Services**
- *Can be developed, tested, built, deployed and delivered independently*
- *Container Boundaries*
- *A service team / service owner is responsible for everything about that service*
    - *Stability, Performance, Quality, Security, CSDL, Documentation etc.,*
    - *BYOO (Bring your own observability)*
    - *Tooling can be developed to test these aspects and integrated during builds*

---

**Explicit is better than implicit**
- *Current CI is a bit of magic. Requires lot of context, easy to make mistakes, fairly easy to fix them, but very hard to integrate into a build*
- *Maglev release is essentially a specification of all participating services*
- *Helps with unified tooling, vs reinventing for every maglev form factor*

**Separate concerns, unify solutions**
- *Any service in maglev must have its code, tests, container builds, deployment artifacts, functional test etc., together, in one place*
- *Facilitates ability to develop, build, deploy, test and iterate as a unit*
- *Provides a framework for developers to automate and re-use the manual corner cases they test currently*

**Code repositories**
- *Recommend that anything that can have its own deployment cadence, should probably be in its own repo*
- *Makes it easy to version, revert, checkout. Contains the change domain*

---

**CI Stuff**

- *Artifact for each service a helm chart (implies the corresponding container). No more tarballs.*
- *Do not rebuild containers/artifacts between stages*
- *Run tests in a clustered environment*

**Leverage community tools**

- *Fair bit of tooling/documentation available*
- *More advancements by the day*
- *Align and refactor in such a way that we can take advantage of them*

---

# How ?

---

## Users and Roles

- Service Developers : Team responsible for a service. End To End. 
- Service Owner: Admin for the service. Responsible for stability, performance, `release` of the service
- CI team
- QA team
- Maglev Admin : Team responsible for `maglev-release`
- Platform Teams: Teams responsible for getting a `maglev-release` into a platform (ex. DNAC)

These are just logical roles/definitions. In our setup, there can be a few people that play (are already playing) these roles.
---

## Service Development (Service Team)

### Process
- Anything requiring a separate deployment cadence is in its own repo
- Has a `VERSION` file
- Developers commit, raise PR
- PR jobs builds temporary containers/helm chart. Runs tests in a cluster
- Developers can do this in their local environments pointing to their dev clusters
- PR Gate runs styling, formatting, UT, coverage, static analysis etc.,
- Tests ensure all aspects of the service are tested
    - For ex. tests clustering/failover for managed services

### Artifacts
- Temporary artifacts. Pushed into a PR infrastructure. Has agressive (daily) GC policies
  
---

## Service Release (Service Owner)

### Process
- Release owner determines when to make a release (after all required bugfixes/features have landed)
- Every release is a GitRelease. Automated documentation, release notes
- Release Gate
    - Runs all tests along with smoke/sanity/TestOnDemand with a previously known stable build of maglev
- Jenkins job to make a release. Updates version
- Releases Container and Chart

### Artifacts
- Helm Chart. Container.
- Present in service-release repository
- GC policy can be once in 2 weeks

---
## Release Candidate / Daily Integration (CI Team)
### Process
- Maglev release is a specification of participating components
- Release specification is maintained in a git repo
- Pulls components as per the specification. Creates cluster
- RC Gate
    - Runs Smoke/Sanity/Automated Regression/Test On Demand* on RC cluster
- If successful, creates a release candidate

### Artifacts
- Creates a `rc.yaml`, which contains specific versions of the components that qualified RC gate
- Commits `rc.yaml` back into the repo
- Copies helm/docker artifacts to rc repo
- GC policy can be once in a month

---

## Maglev Release (QA / CI Team)
### Process

- QA Team will pick up a release candidate
- Full blown tests
    - Regressions
    - System update (from a prev qualified release or combination)
    - HA, Backup/Restore, Service Dependency, Multi Master, FailOvers etc., 
- Promotes a qualifying RC to Release

### Artifacts
- Creates a `release.yaml`, which is essentially a snapshot of qualifying `rc.yaml`
- Commits `release.yaml` back into the repo
- Copies helm/docker artifacts to rc repo
- GC policy can be once in 6 months 
---

## In a nutshell

---

![workflow](https://github3.cisco.com/raw/havishwa/slides/master/maglev-x/workflow.png?token=AAAOrfn4F-MbpBopr8pZuK91A_Sc7T0sks5blbEzwA%3D%3D)

---

# Demo

---

## Use Cases

### Stateless Service
- `maglev-catalog`
- Repo has src, tests, deployment code in a single place
- All depedencies are locked down
- Has language specific styling, coverage, static analysis
- PR job build, deploy to cluster and run tests
- https://github3.cisco.com/maglev-x/maglev-catalog


---

### Stateful (ManagedService)
- `rabbitmq`
- Repo has src, tests, deployment code in a single place
- Documentation about how to use it (first step towards service catalog)
- Can be developed, built, tested, qualified separately (for ex. performance/scale numbers, supported ha topology etc.,)
- PR job builds, deploys to cluster, runs HA/Clustering tests

### Common
- BYOCI: Jenkinsfile in the repo dictates how CI must happen*
- Leverage open source tooling 
    - Harbor
    - Jenkins with k8s plugin
    - Github Branch plugin (which autocreates jenkins jobs)
- BYOO: Bring your own observability
    - `maglev-catalog` brings its own prometheus scrape targets
    - Grafana charts to visualize key metrics

---

# Action!

---

# Before / After

---

**Branching**

- Each maglev release train is a branch in `maglev-release-manifest` repo
- Service level branching is upto the service teams - more ofen than not, branching at service level may not even be required
- Forces us to think about backward compatibility

**Incremental builds**
- If a component has not changed, its version in the release doesn't change. So, comes for free.

**System Updates**
- System update will basically be a controller that ensures a particular release spec is realized on a cluster
- Unified tooling on top of release spec. Can be re-used during installation, upgrades etc.,

**Somebody did a library change**
- No problem. Service *MUST* explicitly pull it. No more magic variables.

---

**A bug in one of the released components**
- Localized fixes.
- If not revert that particular service.
- It is just a single line change in `tip.yaml`

**Capability Version**
- `release spec` carries the capability

**Common libraries**
- In a single repo. Separately versioned. Communicate to consumers regarding new release/features (auto generated)

**Maglev Catalog manifests**
- Convert everything to addons

---

# Cons
---

**Mono vs Multi repo**

- If something can have a different deployment cadence, it is better to be in a different repo
- Increases complexity in terms of dealing/navigating code
    - git submodule, git subtree can help
- Can leverage tooling (automated release notes, versioning, )

**Common changes can take a long time to get into services**
- Explicit is better than implicit
- Better to pay the price during development, than during release

---

# Migration Path

---

There is no difference in how `maglev` is seen / consumed by external teams

There is no difference in how `maglev` is qualified

- Code reorg
- System updates
- CI tooling
- Generate builds and start running existing test pipelines to get a stable build out

---

# Next

---

**Log Sanitization**

- *Send reports of logging rates across categories per service*

**Delivery of ManagedServices**
- *Must be charts. Existing template mechanism has limitations*
- *Catalog can carry these charts*
- *Externalize config. Add Tests. Document.*
- *Configuration is a deployment time concern*

**TestOnDemand**
- *maglev-ci categorized test suites*
- *CI robots to tag PRs and run appropriate test suites*