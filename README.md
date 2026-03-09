# ecommerce-order-service
REST API for order management

## Git Workflow

### Branch Types
5 branches:
1. The `main` branch holds production ready code only. 

2. The `develop` branch is the integration branch where all completed features come together before a release. 

3. Feature branches `feature/*` are branches for individual features or bug fixes, always created from `develop` and merged back into `develop` when done. 

4. Release branches `release/*` are created from `develop` when we're ready to prepare a release candidate. Get tested and then merged into both `main` and `develop`. 

5. Hotfix branches `hotfix/*` are for emergency production fixes only. It is directly off `main`, get fixed fast, and merge back into both `main` and `develop` so the fix doesn't get lost.

### When to Use Each Branch

Starting a new feature means branching off `develop` with a name like `feature/feature-name`. When the work is done, open a PR back into `develop`. When enough features are ready for a release, create a `release/v1.x` branch from `develop`, do final testing there, then merge it into `main` and back into `develop`. If something breaks in production and needs an immediate fix, cut a `hotfix/fix-something` branch straight from `main`, fix it, and merge it into both `main` and `develop`.

### PR Process

No direct pushes to `main` or `develop` — everything goes through a Pull Request. This keeps the main branches stable and ensures every change is reviewed.

---

## Design Decisions

**Why Git Flow?**
I went with Git Flow over simpler models like GitHub Flow because this project has multiple environments (dev, staging, prod) that map naturally to Git Flow's branch structure. The `develop` branch feeds the dev environment, `release/*` branches feed staging, and `main` is production. It keeps things clean and predictable.

**Why separate repos per service?**
Each microservice is independently deployable — the frontend shouldn't need to rebuild just because the order service changed. Separate repos enforce that boundary and make CI/CD pipelines simpler since each repo has its own Jenkinsfile and triggers.

**Why branch protection rules?**
Even working solo, branch protection forces me to go through PRs which creates a clear commit history and makes it obvious when and why changes were made. It also mirrors real-world team practices where no one pushes directly to main.