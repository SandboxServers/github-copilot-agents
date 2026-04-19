## Root Cause Analysis Methods

### Five Whys
Ask "why?" iteratively until you reach the systemic cause:
1. Why did production fail? → The deployment included a breaking change
2. Why wasn't the breaking change caught? → The integration tests didn't cover that path
3. Why didn't integration tests cover it? → The test suite hasn't been updated since the API changed
4. Why hasn't it been updated? → No one owns test maintenance for shared APIs
5. Why is there no ownership? → The team structure doesn't assign cross-service test ownership

**Root cause**: Missing ownership model for cross-service test maintenance
**Action item**: Assign test ownership per service boundary with quarterly review

### Fishbone (Ishikawa) Diagram
Categorize potential causes across dimensions:
| Category | Example Causes |
|---|---|
| **Process** | Missing review step, unclear handoff, no approval gate |
| **People** | Knowledge gap, single point of expertise, onboarding gap |
| **Technology** | Tool limitation, stale dependency, missing monitoring |
| **Environment** | Config drift, network change, permission change |
| **Communication** | Missing stakeholder, unclear requirement, lost context |
| **Measurement** | Wrong metric, missing alert, threshold too high |

### Timeline Analysis
Reconstruct the sequence of events:
1. What happened and when (fact-based, not memory-based)
2. What decisions were made and with what information
3. Where were the delays between detection and action
4. What information would have changed decisions
5. Where did the actual sequence diverge from the expected process
