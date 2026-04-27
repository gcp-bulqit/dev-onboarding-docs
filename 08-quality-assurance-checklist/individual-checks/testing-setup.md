# Testing Setup Documentation

This document provides a comprehensive overview of the testing infrastructure for the Bulqit website project.

## Overview

The project implements a multi-layered testing strategy with:
- **Unit Tests**: Fast, isolated tests using Vitest
- **Integration tests**: Slower with real database connections, good for testing Next.js Server Actions
- **End-to-End Tests**: Browser-based tests using Playwright
- **CI/CD Integration**: Automated testing workflows in GitHub Actions
- **Database Testing**: Integration tests with Neon database

## Test Types

### 1. Unit Tests

Unit tests are located in the `apps/web` and `apps/temporal-workers` directories and use **Vitest** as the testing framework.

#### Configuration

**Web App (`apps/web/vitest.config.mjs`)**:
```javascript
import { defineConfig } from 'vitest/config'
import path from 'path'

export default defineConfig({
  test: {
    environment: 'node',
    globals: true,
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/**',
        'dist/**',
        '**/*.config.*',
        '**/*.test.*',
        '**/*.spec.*',
      ],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve('./src'),
    },
  },
})
```

**Temporal Workers (`apps/temporal-workers/vitest.config.mjs`)**:
```javascript
import { defineConfig } from 'vitest/config'

export default defineConfig({
    test: {
        environment: 'node',
        globals: true,
        passWithNoTests: true, // Don't fail when no test files are found
    },
})
```

#### Test Files

Unit tests are co-located with the code they test and follow the naming convention:
- `*.test.ts` - Test files (excluding: `*.integration.test.ts`)
- `*.spec.ts` - Test specification files

**Example test files**:
- `apps/web/src/utils/phoneInput.util.test.ts`
- `apps/web/src/utils/eligibility.util.test.ts`
- `apps/web/src/utils/emailStatus.util.test.ts`
- `apps/web/src/actions/validateVendorEmailAction.test.ts`

#### Running Unit Tests

```bash
# Run all unit tests
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run tests with coverage
pnpm test:coverage

# Run tests for specific app
pnpm --filter ./apps/web test
```

### 2. Integration Tests

#### Test Files
`**/*.integration.test.*`

#### Running tests
The integration tests are configured separately and require the `INTEGRATION_DB_URL` environment variable. which is configured automatically in `integration-tests.yaml` after db runs in docker container

Locally one can run those tests if `docker` and `act` CLI are installed.
- navigate to .github/workflows
- run: `act -j integration-tests`

### 3. End-to-End Tests

E2E tests are located in the `e2e/` directory and use **Playwright** for browser automation.

#### Configuration

**Playwright Config (`e2e/playwright.config.ts`)**:
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
    testDir: './tests',
    timeout: 30000,
    retries: 1,
    workers: 1,
    reporter: [['list'], ['html', { open: 'never' }]],
    use: {
        baseURL: process.env.BASE_URL || 'http://localhost:3000',
        trace: 'on-first-retry',
        screenshot: 'only-on-failure',
        video: 'retain-on-failure',
        headless: true,
    },
    projects: [
        {
            name: 'chromium',
            use: { ...devices['Desktop Chrome'] },
        },
    ],
});
```

#### Test Structure

**Test Files**:
- `e2e/tests/homeowners.spec.ts` - Homeowner registration flow tests
- `e2e/tests/vercel-deployment.spec.ts` - Vercel deployment verification tests

**Utilities**:
- `e2e/utils/database.ts` - Database connection and operations
- `e2e/utils/databaseVerification.ts` - Database verification helpers

#### Test Scenarios

**Homeowner Registration Flow**:
- Page loading and navigation
- Address search and selection
- Form filling and submission
- Service selection
- Database verification
- Data cleanup

**Vercel Deployment Verification**:
- Page accessibility
- Status code verification
- Content validation
- Error page detection

#### Running E2E Tests

```bash
# Run all e2e tests
cd e2e && pnpm test:e2e

# Run tests in headed mode (with browser UI)
cd e2e && pnpm test:e2e:headed

# Show test report
cd e2e && pnpm test:e2e:report
```

## CI/CD Workflows

### 1. Unit Tests Workflow

**File**: `.github/workflows/unit-tests.yml`

**Triggers**:
- Pull requests to `main`, `staging`, `development` branches
- Manual workflow dispatch

**Features**:
- Concurrent job cancellation
- Turbo cache optimization
- Artifact upload for failed tests
- pnpm workspace support

### 2. Integration Tests

**File**: `.github/workflows/integration-tests.yml`

**Triggers**:
- Pull requests to `main`, `staging` branches
- Manual workflow dispatch

**Features**:
- Automatically runs PostgreSQL service docker containers
- No need for external database credentials
- No need to manage test database instances
- Self-contained CI environment

### 3. E2E Tests Workflow

**File**: `.github/workflows/e2e-tests.yml`

**Triggers**:
- Vercel deployment success events
- Repository dispatch events

**Features**:
- Runs against Vercel preview URLs
- Vercel automation bypass support
- Playwright browser installation
- Test result artifact upload

### 4. Automated Build and Testing

**File**: `.github/workflows/automated-build-and-testing.yml`

**Triggers**:
- Manual workflow dispatch only

**Features**:
- Parallel job execution
- Neon database branch creation
- Playwright browser caching
- Comprehensive build pipeline
- Database cleanup

## Database Testing

### Neon Database Integration

The testing setup includes integration with Neon database for realistic testing scenarios:

**Database Utilities (`e2e/utils/database.ts`)**:
- Connection management
- Test data creation and cleanup
- Homeowner data verification
- Form input validation

**Key Functions**:
- `getHomeownerByEmail()` - Retrieve test data
- `cleanupTestData()` - Clean up after tests
- `verifyHomeownerInDatabase()` - Verify data persistence
- `isDatabaseAvailable()` - Check connection status

### Database Verification (`e2e/utils/databaseVerification.ts`)

Provides helper functions for verifying database operations:
- `verifyHomeownerInDatabase()` - Comprehensive data verification
- `cleanupHomeownerTestData()` - Test data cleanup
- `verifyDatabaseConnection()` - Connection validation

## Environment Variables

### Required for E2E Tests

```bash
# Base URL for testing
BASE_URL=http://localhost:3000

# Vercel automation bypass (for deployment testing)
VERCEL_AUTOMATION_BYPASS_SECRET=your_secret

# Database connection
DB_URL=your_neon_database_url
```

### Required for Unit Tests

```bash
# Test environment
NODE_ENV=test
CI=true

# Turbo cache (optional)
TURBO_TOKEN=your_turbo_token
TURBO_TEAM=your_turbo_team
```

## Test Data Management

### Test Data Creation

E2E tests create realistic test data including:
- Homeowner information (name, email, address)
- Service selections
- Address coordinates
- Registration status

### Test Data Cleanup

All test data is automatically cleaned up after tests complete:
- Database records deletion
- Email-based cleanup (using unique test emails)
- Address and service data removal

## Best Practices

### Writing Unit Tests

1. **Co-location**: Place test files next to the code they test
2. **Naming**: Use descriptive test names that explain the scenario
3. **Isolation**: Each test should be independent and not rely on other tests
4. **Coverage**: Aim for high test coverage, especially for critical business logic

### Writing E2E Tests

1. **Page Objects**: Use page object pattern for maintainable tests
2. **Data Cleanup**: Always clean up test data after tests
3. **Retries**: Use retries for flaky operations (network, database)
4. **Screenshots**: Capture screenshots on failures for debugging

### CI/CD Best Practices

1. **Parallel Execution**: Run independent tests in parallel
2. **Caching**: Cache dependencies and build artifacts
3. **Artifacts**: Upload test results and reports for debugging
4. **Database Isolation**: Use separate database branches for each test run

## Troubleshooting

### Common Issues

1. **Playwright Browser Installation**:
   ```bash
   cd e2e && npx playwright install --with-deps
   ```

2. **Database Connection Issues**:
   - Verify `DB_URL` environment variable
   - Check Neon database status
   - Ensure proper network access

3. **Vercel Deployment Testing**:
   - Verify `VERCEL_AUTOMATION_BYPASS_SECRET` is set
   - Check deployment URL accessibility
   - Ensure bypass headers are properly configured

### Debugging

1. **Local E2E Testing**:
   ```bash
   # Start the application
   pnpm dev
   
   # Run tests in headed mode
   cd e2e && pnpm test:e2e:headed
   ```

2. **Test Reports**:
   ```bash
   # View HTML report
   cd e2e && pnpm test:e2e:report
   ```

3. **Database Verification**:
   ```bash
   # Check database connection
   node -e "console.log(require('./e2e/utils/database').isDatabaseAvailable())"
   ```

## Performance Considerations

### Test Execution Time

- **Unit Tests**: ~30 seconds
- **E2E Tests**: ~2-3 minutes
- **Full CI Pipeline**: ~8-10 minutes

### Optimization Strategies

1. **Turbo Cache**: Leverage Turborepo for build and test caching
2. **Parallel Jobs**: Run independent tests simultaneously
3. **Browser Caching**: Cache Playwright browsers between runs
4. **Database Branching**: Use Neon branches for isolated testing

## Future Improvements

1. **Visual Regression Testing**: Add visual comparison tests
2. **Performance Testing**: Include Lighthouse CI for performance metrics
3. **Accessibility Testing**: Add automated accessibility checks
4. **API Testing**: Expand API endpoint testing coverage
5. **Mobile Testing**: Add mobile device testing scenarios 