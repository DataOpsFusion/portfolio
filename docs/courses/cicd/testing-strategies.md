# Testing Strategies

Learn to build comprehensive testing strategies that catch bugs early, provide confidence in deployments, and don't slow down development.

## The Testing Pyramid

```
       🔺 E2E Tests
      🔸🔸 Integration Tests  
   🔹🔹🔹🔹 Unit Tests
```

### Unit Tests (Base of Pyramid)
**What**: Test individual functions/components in isolation
**Why**: Fast feedback, easy to debug, high coverage
**When**: Every function, every edge case

```javascript
// Example: Testing a utility function
describe('calculateTax', () => {
  test('calculates tax for standard rate', () => {
    expect(calculateTax(100, 0.08)).toBe(8);
  });
  
  test('handles zero amount', () => {
    expect(calculateTax(0, 0.08)).toBe(0);
  });
  
  test('throws error for negative amount', () => {
    expect(() => calculateTax(-10, 0.08)).toThrow();
  });
});
```

### Integration Tests (Middle)
**What**: Test how components work together
**Why**: Catch interface issues, validate workflows
**When**: Critical user journeys, API endpoints

```python
# Example: Testing API endpoint
def test_create_user_endpoint():
    response = client.post('/users', json={
        'name': 'John Doe',
        'email': 'john@example.com'
    })
    
    assert response.status_code == 201
    assert response.json()['name'] == 'John Doe'
    
    # Verify user was created in database
    user = db.query(User).filter(User.email == 'john@example.com').first()
    assert user is not None
```

### End-to-End Tests (Top)
**What**: Test complete user workflows through the UI
**Why**: Validate real user scenarios
**When**: Critical business flows, major features

```javascript
// Example: Cypress E2E test
describe('User Registration Flow', () => {
  it('allows new user to register and login', () => {
    cy.visit('/register');
    cy.get('[data-testid=name-input]').type('John Doe');
    cy.get('[data-testid=email-input]').type('john@example.com');
    cy.get('[data-testid=password-input]').type('password123');
    cy.get('[data-testid=submit-button]').click();
    
    cy.url().should('include', '/dashboard');
    cy.contains('Welcome, John Doe');
  });
});
```

## Testing Strategy by Application Type

### Web Applications
```yaml
Testing Stack:
  Unit: Jest, Vitest, pytest
  Integration: Supertest, TestContainers
  E2E: Cypress, Playwright
  Visual: Percy, Chromatic
  Performance: Lighthouse CI
```

### APIs/Microservices
```yaml
Testing Stack:
  Unit: Jest, pytest, JUnit
  Integration: Postman, REST Assured
  Contract: Pact, Spring Cloud Contract
  Load: k6, JMeter
  Security: OWASP ZAP
```

### Mobile Applications
```yaml
Testing Stack:
  Unit: XCTest, Espresso
  Integration: Detox, Appium
  Device: Firebase Test Lab, BrowserStack
  Performance: Instruments, Android Profiler
```

## Test Data Management

### Test Data Strategies

#### 1. Static Test Data
Predefined datasets for consistent testing:
```json
// fixtures/users.json
{
  "validUser": {
    "name": "John Doe",
    "email": "john@example.com",
    "age": 30
  },
  "invalidUser": {
    "name": "",
    "email": "invalid-email",
    "age": -5
  }
}
```

#### 2. Generated Test Data
Dynamic data generation for broader coverage:
```python
# Using faker library
from faker import Faker
fake = Faker()

def generate_user():
    return {
        'name': fake.name(),
        'email': fake.email(),
        'address': fake.address()
    }
```

#### 3. Database Seeding
Consistent database state for tests:
```sql
-- seeds/test_data.sql
INSERT INTO users (id, name, email, created_at) VALUES
(1, 'Test User 1', 'test1@example.com', '2023-01-01'),
(2, 'Test User 2', 'test2@example.com', '2023-01-02');

INSERT INTO orders (id, user_id, total, status) VALUES
(1, 1, 99.99, 'completed'),
(2, 1, 149.99, 'pending');
```

### Test Environment Management
```yaml
# docker-compose.test.yml
version: '3.8'
services:
  app:
    build: .
    environment:
      NODE_ENV: test
      DATABASE_URL: postgresql://test:test@db:5432/test_db
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: test_db
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
    tmpfs:
      - /var/lib/postgresql/data  # In-memory for speed
```

## Testing in CI/CD Pipelines

### Parallel Test Execution
```yaml
# GitHub Actions parallel testing
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        test-group: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          npm test -- --shard=${{ matrix.test-group }}/4
```

### Test Result Reporting
```yaml
# Test reporting with artifacts
- name: Run tests
  run: npm test -- --coverage --reporter=junit
  
- name: Publish test results
  uses: dorny/test-reporter@v1
  if: always()
  with:
    name: 'Test Results'
    path: 'test-results.xml'
    reporter: 'jest-junit'
    
- name: Upload coverage
  uses: codecov/codecov-action@v3
  with:
    files: ./coverage/lcov.info
```

### Flaky Test Management
```yaml
# Retry flaky tests
- name: Run tests with retry
  uses: nick-invision/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: npm test
```

## Advanced Testing Patterns

### Test Doubles

#### Mocks
Replace dependencies with controlled implementations:
```javascript
// Mock external API
jest.mock('axios');
const mockedAxios = axios as jest.Mocked<typeof axios>;

test('fetches user data', async () => {
  mockedAxios.get.mockResolvedValue({
    data: { id: 1, name: 'John' }
  });
  
  const user = await fetchUser(1);
  expect(user.name).toBe('John');
});
```

#### Stubs
Provide fixed responses:
```python
# Stub payment service
class PaymentServiceStub:
    def charge(self, amount):
        if amount > 1000:
            return {'status': 'failed', 'error': 'Amount too high'}
        return {'status': 'success', 'transaction_id': '12345'}
```

#### Fakes
Working implementations with shortcuts:
```javascript
// In-memory database fake
class FakeUserRepository {
  constructor() {
    this.users = new Map();
  }
  
  async save(user) {
    this.users.set(user.id, user);
    return user;
  }
  
  async findById(id) {
    return this.users.get(id);
  }
}
```

### Contract Testing
Ensure service compatibility:
```javascript
// Provider contract test
const { Verifier } = require('@pact-foundation/pact');

describe('User Service Provider', () => {
  it('validates the expectations of UserConsumer', () => {
    return new Verifier({
      provider: 'UserService',
      providerBaseUrl: 'http://localhost:3000',
      pactUrls: ['./pacts/userconsumer-userservice.json']
    }).verifyProvider();
  });
});
```

### Mutation Testing
Test the quality of your tests:
```bash
# Install and run mutation testing
npm install --save-dev stryker-cli
npx stryker run
```

## Performance Testing

### Load Testing with k6
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp up
    { duration: '5m', target: 100 },  // Stay at 100 users
    { duration: '2m', target: 200 },  // Ramp up
    { duration: '5m', target: 200 },  // Stay at 200 users
    { duration: '2m', target: 0 },    // Ramp down
  ],
};

export default function () {
  let response = http.get('https://api.example.com/users');
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  sleep(1);
}
```

### Database Performance Testing
```sql
-- Test query performance
EXPLAIN ANALYZE 
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2023-01-01'
GROUP BY u.id, u.name
ORDER BY order_count DESC
LIMIT 100;
```

## Test Metrics and Quality Gates

### Coverage Metrics
```yaml
# Coverage thresholds
coverage:
  threshold:
    global:
      branches: 80
      functions: 80
      lines: 80
      statements: 80
```

### Quality Gates
```yaml
# SonarQube quality gate
sonar.qualitygate.wait=true
sonar.coverage.exclusions=**/*test*/**,**/migrations/**
sonar.test.inclusions=**/*test*/**
```

### Test Reporting Dashboard
```yaml
# Allure test reporting
- name: Generate Allure Report
  uses: simple-elf/allure-report-action@master
  if: always()
  with:
    allure_results: allure-results
    allure_history: allure-history
```

---

*Good testing strategy is about finding the right balance: enough tests to catch bugs and give confidence, but not so many that they slow down development.*
