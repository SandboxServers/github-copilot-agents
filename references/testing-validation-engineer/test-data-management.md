## Test Data Management

### Test Data Strategies

Effective testing requires realistic, controlled, and isolated data. Poor test data management is one of the most common causes of flaky tests and unreliable results.

### Synthetic Data Generation

- Generate test data programmatically — do not copy production databases
- Use libraries like Bogus (.NET), Faker (Python/JS), or AutoFixture for realistic data
- Match production data distribution — if 80% of users are in one region, reflect that
- Generate edge cases explicitly — empty strings, max-length values, Unicode, special characters
- Version control your data generation scripts alongside test code

### Database Seeding

- Seed databases with a known dataset before test suites run
- Use migration scripts to set up schema and reference data consistently
- Separate seed data from test-specific data — seed data is shared, test data is per-test
- Automate seeding as part of test environment provisioning
- Keep seed datasets small — large datasets slow down test setup and obscure failures

### Test Isolation

- **Each test gets a clean state** — never rely on another test's output
- Use transactions that roll back after each test for database-backed tests
- Create and destroy test-specific resources (queues, containers, topics) per test
- Parallel test execution requires strict isolation — no shared mutable state
- Use unique identifiers (GUIDs, timestamps) to avoid collisions in shared environments

### Data Masking for Production-Like Data

- When production-like data is required, mask all PII before use in test environments
- Replace names, emails, addresses, SSNs with realistic but fake equivalents
- Preserve data relationships and referential integrity during masking
- Automate masking as a pipeline step — never manually copy production data
- Comply with data residency and privacy regulations (GDPR, HIPAA) when handling any production-derived data

### Avoiding Shared Mutable State

- Shared mutable state between tests is the number one cause of flaky tests
- Each test should set up its own preconditions and clean up after itself
- Avoid static variables, singletons, or global configuration that persists between tests
- If tests must share expensive resources (database server), isolate at the schema or database level
- Run tests in random order to detect hidden dependencies

### Fixtures and Factories Pattern

- **Fixtures** — predefined, reusable test data configurations for common scenarios
- **Factories** — functions that generate test data with sensible defaults and overrides
- Prefer factories over fixtures for flexibility — override only what the test cares about
- Keep factory definitions close to the tests that use them
- Compose complex test scenarios from simple factory building blocks

```csharp
// Factory pattern example
var user = UserFactory.Create(role: "admin", email: "test@example.com");
var order = OrderFactory.Create(userId: user.Id, status: "pending");
```

```python
# Factory pattern example
user = UserFactory.create(role="admin", email="test@example.com")
order = OrderFactory.create(user_id=user.id, status="pending")
```
