## When Power Platform Is a Trap

### Build in Power Apps when:
- The app is forms-over-data with standard CRUD operations
- Users need a quick internal tool (not customer-facing)
- The data model fits Dataverse naturally
- The team includes citizen developers with business knowledge
- Time-to-value matters more than pixel-perfect UI

### Build a "real app" when:
- Complex custom UI requirements (animations, complex layouts, real-time collaboration)
- Extreme performance requirements (sub-100ms response times, millions of records in view)
- Complex offline requirements beyond Power Apps offline capabilities
- Need full source control, unit testing, CI/CD that Power Platform can't match
- Customer-facing application where brand experience is critical
- Integration complexity exceeds what connectors/custom connectors can handle cleanly
- The app will have >50,000 active users (licensing cost becomes prohibitive vs custom app)
- The team is entirely pro-dev with no citizen developer involvement
