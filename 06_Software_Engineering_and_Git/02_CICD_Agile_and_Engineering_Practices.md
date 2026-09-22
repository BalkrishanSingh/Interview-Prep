# Module 06: Software Engineering — CI/CD, Deployment & Agile Practices

---

## 1. CI/CD Pipeline Architecture

Modern software engineering automates the pathway from source code commit to production delivery via **Continuous Integration (CI)** and **Continuous Deployment (CD)** pipelines.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        AUTOMATED CI/CD PIPELINE                        │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
    ┌───────────────────────────────┴───────────────────────────────┐
    ▼ (CONTINUOUS INTEGRATION)                                      ▼ (CONTINUOUS DEPLOYMENT)
┌────────────┐     ┌────────────┐     ┌────────────┐          ┌────────────┐     ┌────────────┐
│ Code Push  │ ──► │  Lint &    │ ──► │ Unit Tests │ ──Build─►│ Staging &  │ ──► │ Production │
│  (Git)     │     │ Static Ana │     │ & Coverage │  Artifact│ Smoke Test│     │ Deployment │
└────────────┘     └────────────┘     └────────────┘          └────────────┘     └────────────┘
```

### 1.1 Continuous Delivery vs. Continuous Deployment
- **Continuous Delivery**: Every successful build artifact is technically ready for immediate production deployment; however, the final trigger to deploy to production requires an explicit manual business approval button.
- **Continuous Deployment**: Every commit passing the automated test suite is deployed directly to live production environments with zero human intervention.

---

## 2. Production Deployment Strategies

```
BLUE-GREEN DEPLOYMENT:
[ Router / Load Balancer ] ──► [ Green Environment (Active v1.0) ]
                               [ Blue Environment  (Idle v2.0 Staged) ]
(Instant cutover by repointing the router; instant rollback if errors spike)

CANARY DEPLOYMENT:
[ Router ] ──┬── 95% Traffic ──► [ Version 1.0 (Current Stable) ]
             └─── 5% Traffic ──► [ Version 2.0 (Canary Instance) ]
(Exposes new release to small percentage of real users while monitoring error rates)
```

### Detailed Strategy Trade-off Matrix
| Strategy | Downtime | Resource Cost | Rollback Speed | Best Used When |
| :--- | :--- | :--- | :--- | :--- |
| **Recreate** | High (Downtime during switch) | Low (Reuses same servers) | Slow | Non-critical internal dev/test environments. |
| **Rolling Update** | **Zero** | Low (Replaces instances incrementally) | Medium (Requires rolling backward) | Microservices clusters (Kubernetes default). |
| **Blue-Green** | **Zero** | **High** (Requires 2x infrastructure capacity) | **Instant** (Flip router back to green) | High-stakes mission-critical services. |
| **Canary** | **Zero** | Medium | Fast (Shift traffic back to 100% stable) | Validating performance and error rates under real user load. |

---

## 3. The Testing Pyramid & TDD

```
               ▲
              / \
             /   \      E2E / UI Tests (Slowest, Expensive, Fewest)
            /─────\
           /       \    Integration Tests (API, DB, External services)
          /─────────\
         /           \  Unit Tests (Fastest, In-Memory, Isolated, Majority)
        /─────────────\
```

### Test-Driven Development (TDD) Cycle:
1. **Red**: Write a small, failing automated unit test that specifies a required feature before writing any production code.
2. **Green**: Write the minimum amount of production code necessary to pass the test.
3. **Refactor**: Clean up the implementation, remove duplication, and optimize design while ensuring all tests stay green.

---

## 4. Agile Scrum Framework & Ceremonies

Scrum is an iterative framework for delivering complex software in fixed-length iterations called **Sprints** (typically 2 weeks).

### 4.1 The Three Core Scrum Roles
1. **Product Owner (PO)**: Owns the Product Backlog, defines user stories, and prioritizes features based on customer business value.
2. **Scrum Master**: Facilitator who removes impediments, shields the development team from external disruptions, and ensures Scrum practices are followed.
3. **Development Team**: Cross-functional group of engineers responsible for designing, building, and testing the sprint increment.

### 4.2 The Four Scrum Ceremonies
| Ceremony | Cadence | Purpose |
| :--- | :--- | :--- |
| **Sprint Planning** | Beginning of Sprint | PO presents top backlog items; team commits to the Sprint Backlog for the iteration. |
| **Daily Standup** | Daily (15 minutes) | Synchronize progress by answering: 1) What did I complete yesterday? 2) What will I do today? 3) Any blockers? |
| **Sprint Review** | End of Sprint | Live demonstration of the completed, potentially shippable increment to stakeholders. |
| **Sprint Retrospective** | End of Sprint | Internal team reflection: What went well? What went wrong? What concrete changes will we adopt next sprint? |
