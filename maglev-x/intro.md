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
We are micro services, but dont have any build/deployment advantages that come with it
- *We release as a monolith*
- *Simple fixes to be delivered requires a new build*

Getting a maglev build out is a huge task
- *Significant overhead with branches, release management*
- *Babysitting builds sap out energy*

CI/CD tooling is not streamlined
- *There is no specification of how this must happen*
- *Any activity (brancing, registry cleanup etc.,) here will cause ripples almost in every area*

Butterfly effect and blast radius
- *Issue in one library/component will hold up the entire build*
- *No way to revert to previous known stable version of that component*

---

Code layout is not uniform
- *Source is somewhere, build is somewhere else, deployment is somewhere else*
- *Increased cognitive overload for new comers*
- *Requires significant context for anyone to be productive and deliver with confidence*

Developer workflow
- *Not uniform*
- *Significant setup overhead to test features/fixes*

Return of the monolith
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
Versioning
- *[SemVer2](https://semver.org/): Version numbers should follow conventions and indicate meaning*
- *Every independently deployable entity MUST carry its own version*
- *Can revert/checkout to a specific fix of a specific service*

MicroServices
- *Can be developed, tested, built, deployed and delivered independently*
- *Container Boundaries*
- *A service team / service owner is responsible for everything about that service*
    - *Stability, Performance, Quality, Security, CSDL, Documentation etc.,*
    - *Tooling can be developed to test these aspects and integrated during builds*

---

Explicit is better than implicit
- *Current CI is a bit of magic. Requires lot of context, easy to make mistakes, fairly easy to fix them, but very hard to integrate into a build*
- *Maglev release is essentially a specification of all participating services*
- *Helps with unified tooling, vs reinventing for every maglev form factor*

Separate concerns, but unify solutions
- **

---

### Dev workflow

![workflow](https://github3.cisco.com/raw/havishwa/slides/master/maglev-x/workflow.png?token=AAAOrfn4F-MbpBopr8pZuK91A_Sc7T0sks5blbEzwA%3D%3D)

---
## Next

- AWS deployments with helm
- ISO installer has to be ported
- Remove complex orchestration logic from ansible as much as possible
    - Kong API seeding (modelled as CRDs with custom controller)
    - Docker registry seeding (modelled as a configmap with a controller)
    - IDM resource creation (post-install hook)
    - Managed Services user creation (post-install hook)