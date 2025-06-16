---
description: Tests automatisés
---

# Testing Automation Workflow

## Trigger Commands
- `@workflow:test` - Setup tests automatisés
- `@workflow:e2e` - Tests end-to-end complets
- `@workflow:performance` - Tests de performance

## Testing Strategy

### 1. Unit Testing Template
```typescript
// Auto-generated unit tests
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { jest } from '@jest/globals';
import Component from './Component';

describe('Component', () => {
  const defaultProps = {
    id: 'test-component',
    trackingId: 'test-tracking',
    onInteraction: jest.fn(),
    onConversion: jest.fn()
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    cleanup();
  });

  describe('Rendering', () => {
    it('renders with required props', () => {
      render(<Component {...defaultProps} />);
      expect(screen.getByTestId('test-component')).toBeInTheDocument();
    });

    it('applies custom className', () => {
      render(<Component {...defaultProps} className="custom-class" />);
      expect(screen.getByTestId('test-component')).toHaveClass('custom-class');
    });

    it('renders children correctly', () => {
      render(
        <Component {...defaultProps}>
          <span>Test content</span>
        </Component>
      );
      expect(screen.getByText('Test content')).toBeInTheDocument();
    });
  });

  describe('Interactions', () => {
    it('handles click interactions', async () => {
      render(<Component {...defaultProps} />);
      
      const component = screen.getByTestId('test-component');
      fireEvent.click(component);

      await waitFor(() => {
        expect(defaultProps.onInteraction).toHaveBeenCalledWith(
          expect.objectContaining({
            type: 'click',
            trackingId: 'test-tracking'
          })
        );
      });
    });

    it('tracks conversions correctly', async () => {
      render(<Component {...defaultProps} conversionGoal="email_signup" />);
      
      // Simulate conversion action
      const submitButton = screen.getByRole('button');
      fireEvent.click(submitButton);

      await waitFor(() => {
        expect(defaultProps.onConversion).toHaveBeenCalledWith(
          expect.objectContaining({
            goal: 'email_signup',
            timestamp: expect.any(Number)
          })
        );
      });
    });
  });

  describe('A/B Testing', () => {
    it('supports variant A', () => {
      render(<Component {...defaultProps} testVariant="A" />);
      expect(screen.getByTestId('test-component')).toHaveAttribute('data-test-variant', 'A');
    });

    it('supports variant B', () => {
      render(<Component {...defaultProps} testVariant="B" />);
      expect(screen.getByTestId('test-component')).toHaveAttribute('data-test-variant', 'B');
    });
  });

  describe('Error Handling', () => {
    it('handles missing props gracefully', () => {
      const consoleSpy = jest.spyOn(console, 'error').mockImplementation();
      
      render(<Component />);
      
      expect(consoleSpy).not.toHaveBeenCalled();
      consoleSpy.mockRestore();
    });
  });
});
```

### 2. Integration Testing
```typescript
// API integration tests
import request from 'supertest';
import app from '../app';
import { setupTestDB, clearTestDB } from '../utils/testDb';

describe('API Integration Tests', () => {
  beforeAll(async () => {
    await setupTestDB();
  });

  afterAll(async () => {
    await clearTestDB();
  });

  beforeEach(async () => {
    await clearTestData();
    await seedTestData();
  });

  describe('POST /api/posts', () => {
    const validPostData = {
      title: 'Test AI Tool Review',
      content: 'Comprehensive review content with minimum 100 characters for validation.',
      category: 'ai-tools',
      tags: ['chatgpt', 'ai', 'productivity'],
      affiliateLinks: [{
        text: 'Try ChatGPT Plus',
        url: 'https://openai.com/chatgpt',
        commission: 25
      }]
    };

    it('creates post with valid data', async () => {
      const response = await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send(validPostData)
        .expect(201);

      expect(response.body).toMatchObject({
        success: true,
        data: {
          title: validPostData.title,
          category: validPostData.category,
          affiliateLinks: expect.arrayContaining([
            expect.objectContaining({
              text: 'Try ChatGPT Plus',
              commission: 25
            })
          ])
        }
      });
    });

    it('validates required fields', async () => {
      const invalidData = { ...validPostData, title: '' };
      
      const response = await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send(invalidData)
        .expect(400);

      expect(response.body).toMatchObject({
        success: false,
        error: 'Validation failed'
      });
    });

    it('tracks analytics events', async () => {
      const analyticsSpy = jest.spyOn(analytics, 'track');
      
      await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send(validPostData);

      expect(analyticsSpy).toHaveBeenCalledWith('post_created', {
        category: 'ai-tools',
        affiliateLinksCount: 1,
        isPremium: false
      });
    });
  });
});
```

### 3. End-to-End Testing (Playwright)
```typescript
// E2E tests with Playwright
import { test, expect } from '@playwright/test';

test.describe('Revenue Flow E2E', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('complete subscription flow', async ({ page }) => {
    // 1. Navigate to pricing page
    await page.click('[data-testid="pricing-link"]');
    await expect(page).toHaveURL('/pricing');

    // 2. Select Pro plan
    await page.click('[data-testid="select-pro-plan"]');

    // 3. Track analytics event
    await page.waitForFunction(() => {
      return window.analytics && window.analytics.track;
    });

    // 4. Fill payment form
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="card-number"]', '4242424242424242');
    await page.fill('[data-testid="card-expiry"]', '12/25');
    await page.fill('[data-testid="card-cvc"]', '123');

    // 5. Submit payment
    await page.click('[data-testid="submit-payment"]');

    // 6. Verify success
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible();
    await expect(page).toHaveURL('/dashboard');
  });

  test('affiliate link tracking', async ({ page }) => {
    // 1. Visit blog post with affiliate links
    await page.goto('/blog/best-ai-tools-2024');

    // 2. Click affiliate link
    const affiliateLink = page.locator('[data-tool="chatgpt"]');
    await affiliateLink.click();

    // 3. Verify tracking
    await page.waitForFunction(() => {
      return window.analytics.track.calls.some(call => 
        call[0] === 'affiliate_click' && 
        call[1].tool === 'chatgpt'
      );
    });

    // 4. Verify redirect (in new tab)
    const [newPage] = await Promise.all([
      page.waitForEvent('popup'),
      affiliateLink.click()
    ]);
    
    await expect(newPage).toHaveURL(/openai\.com/);
  });

  test('A/B test variant assignment', async ({ page, context }) => {
    // Set user ID for consistent variant assignment
    await context.addCookies([{
      name: 'user_id',
      value: 'test-user-123',
      domain: 'localhost',
      path: '/'
    }]);

    await page.goto('/pricing');

    // Check variant assignment
    const variant = await page.getAttribute('[data-testid="pricing-container"]', 'data-test-variant');
    expect(['A', 'B']).toContain(variant);

    // Verify variant-specific elements
    if (variant === 'B') {
      await expect(page.locator('[data-testid="annual-highlight"]')).toBeVisible();
    }
  });
});
```

### 4. Performance Testing
```typescript
// Performance tests with Lighthouse CI
const lighthouse = require('lighthouse');
const chromeLauncher = require('chrome-launcher');

describe('Performance Tests', () => {
  let chrome;
  let port;

  beforeAll(async () => {
    chrome = await chromeLauncher.launch({ chromeFlags: ['--headless'] });
    port = chrome.port;
  });

  afterAll(async () => {
    await chrome.kill();
  });

  test('homepage performance meets standards', async () => {
    const runnerResult = await lighthouse('http://localhost:3000', {
      port,
      onlyCategories: ['performance'],
    });

    const performanceScore = runnerResult.lhr.categories.performance.score * 100;
    
    expect(performanceScore).toBeGreaterThanOrEqual(90); // 90+ performance score
  });

  test('pricing page loads quickly', async () => {
    const runnerResult = await lighthouse('http://localhost:3000/pricing', {
      port,
      onlyCategories: ['performance'],
    });

    const metrics = runnerResult.lhr.audits;
    const firstContentfulPaint = metrics['first-contentful-paint'].numericValue;
    const largestContentfulPaint = metrics['largest-contentful-paint'].numericValue;

    expect(firstContentfulPaint).toBeLessThan(1500); // < 1.5s FCP
    expect(largestContentfulPaint).toBeLessThan(2500); // < 2.5s LCP
  });
});
```

### 5. Visual Regression Testing
```typescript
// Visual regression with Chromatic/Percy
import { test } from '@playwright/test';
import { percySnapshot } from '@percy/playwright';

test.describe('Visual Regression Tests', () => {
  test('pricing page visual consistency', async ({ page }) => {
    await page.goto('/pricing');
    
    // Wait for page to fully load
    await page.waitForLoadState('networkidle');
    
    // Take screenshot for visual comparison
    await percySnapshot(page, 'Pricing Page - Desktop');
  });

  test('mobile responsive design', async ({ page }) => {
    // Set mobile viewport
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto('/pricing');
    
    await page.waitForLoadState('networkidle');
    await percySnapshot(page, 'Pricing Page - Mobile');
  });

  test('component states', async ({ page }) => {
    await page.goto('/storybook');
    
    // Test different component states
    const states = ['default', 'loading', 'error', 'success'];
    
    for (const state of states) {
      await page.click(`[data-testid="state-${state}"]`);
      await percySnapshot(page, `Component - ${state} state`);
    }
  });
});
```

### 6. Test Configuration
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.stories.tsx',
    '!src/**/*.d.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{ts,tsx}',
  ],
};

// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
});
```

## Test Coverage Requirements
- **Unit Tests**: 90%+ code coverage
- **Integration Tests**: All API endpoints
- **E2E Tests**: Critical user journeys
- **Performance**: 90+ Lighthouse score
