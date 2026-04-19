## DevOps Test Principles

### Shift Left
Move quality upstream by performing testing tasks earlier in the pipeline. Through test and process improvements, shifting left reduces both the time for tests to run and the impact of failures later in the cycle. Most testing should complete before a change merges to the main branch.

### Test Taxonomy (L0-L4)
| Level | Type | Dependencies | Speed | When to Run |
|---|---|---|---|---|
| **L0** | Unit tests (in-memory) | None — code under test only | <60ms per test | Every build, every PR |
| **L1** | Unit tests (light deps) | Assembly + mocks | <400ms per test | Every build, every PR |
| **L2** | Functional tests | Assembly + real deps (SQL, filesystem) | Seconds | PR validation, CI |
| **L3** | Functional tests | Service deployment + stubs | Minutes | Post-deploy, staging |
| **L4** | Integration tests | Full production deployment | Minutes-hours | Production validation |

### Core Principles
1. **Write tests at the lowest possible level** — unit tests over functional tests when they produce the same signal
2. **Aim for test reliability** — unreliable tests are organizationally expensive, they erode trust
3. **Write functional tests that run anywhere** — use only public APIs, no internal implementation details
4. **Design for testability** — testability is a first-class design concern, not an afterthought
5. **Treat test code as product code** — same quality bar, same code review rigor
6. **Use shared test infrastructure** — testing is a shared service for the entire team
7. **Code owners are responsible for testing** — don't rely on others to test your component
