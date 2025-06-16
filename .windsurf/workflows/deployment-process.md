---
description: Déploiement optimisé
---

# Deployment Process Workflow

## Trigger Commands
- `@workflow:deploy` - Déploiement production optimisé
- `@workflow:preview` - Déploiement preview/staging
- `@workflow:rollback` - Rollback sécurisé

## Deployment Strategy

### 1. Pre-Deployment Checks
```yaml
# .github/workflows/pre-deploy.yml
name: Pre-Deployment Checks

on:
  pull_request:
    branches: [main]

jobs:
  quality_gates:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Type checking
        run: npm run type-check
      
      - name: Linting
        run: npm run lint
      
      - name: Unit tests
        run: npm run test -- --coverage
      
      - name: Build check
        run: npm run build
      
      - name: Bundle analysis
        run: npm run analyze
        
      - name: Security scan
        run: npm audit --audit-level high
      
      - name: Performance budget check
        run: npm run lighthouse:ci

  e2e_tests:
    runs-on: ubuntu-latest
    needs: quality_gates
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install
      
      - name: Start application
        run: npm run start:test &
        
      - name: Wait for application
        run: npx wait-on http://localhost:3000
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

### 2. Staging Deployment
```yaml
# .github/workflows/staging.yml
name: Deploy to Staging

on:
  push:
    branches: [develop]

jobs:
  deploy_staging:
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          NEXT_PUBLIC_ENV: staging
          NEXT_PUBLIC_API_URL: ${{ secrets.STAGING_API_URL }}
          NEXT_PUBLIC_ANALYTICS_ID: ${{ secrets.STAGING_ANALYTICS_ID }}
      
      - name: Deploy to Vercel Staging
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          working-directory: ./
          alias-domains: staging.ai-tools.example.com
      
      - name: Run smoke tests on staging
        run: npm run test:smoke -- --base-url=https://staging.ai-tools.example.com
      
      - name: Notify team
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Staging deployment completed: https://staging.ai-tools.example.com'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### 3. Production Deployment
```yaml
# .github/workflows/production.yml
name: Deploy to Production

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      deployment_type:
        description: 'Type of deployment'
        required: true
        default: 'standard'
        type: choice
        options:
        - standard
        - hotfix
        - rollback

jobs:
  deploy_production:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate deployment window
        run: |
          HOUR=$(date +%H)
          if [ $HOUR -lt 9 ] || [ $HOUR -gt 17 ]; then
            echo "❌ Deployment outside business hours (9-17 UTC)"
            exit 1
          fi
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run security audit
        run: npm audit --audit-level critical
      
      - name: Build application
        run: npm run build
        env:
          NEXT_PUBLIC_ENV: production
          NEXT_PUBLIC_API_URL: ${{ secrets.PROD_API_URL }}
          NEXT_PUBLIC_ANALYTICS_ID: ${{ secrets.PROD_ANALYTICS_ID }}
          NEXT_PUBLIC_STRIPE_KEY: ${{ secrets.STRIPE_PUBLISHABLE_KEY }}
      
      - name: Pre-deployment backup
        run: |
          curl -X POST "${{ secrets.BACKUP_WEBHOOK }}" \
            -H "Authorization: Bearer ${{ secrets.BACKUP_TOKEN }}" \
            -d '{"action": "backup", "environment": "production"}'
      
      - name: Deploy to Vercel Production
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
          working-directory: ./
      
      - name: Health check
        run: |
          for i in {1..10}; do
            if curl -f https://ai-tools.example.com/api/health; then
              echo "✅ Health check passed"
              break
            else
              echo "⏳ Waiting for deployment... ($i/10)"
              sleep 30
            fi
          done
      
      - name: Run critical path tests
        run: npm run test:critical -- --base-url=https://ai-tools.example.com
      
      - name: Update monitoring
        run: |
          curl -X POST "https://api.datadoghq.com/api/v1/events" \
            -H "Content-Type: application/json" \
            -H "DD-API-KEY: ${{ secrets.DATADOG_API_KEY }}" \
            -d '{
              "title": "Production Deployment",
              "text": "New version deployed to production",
              "tags": ["environment:production", "service:ai-tools"]
            }'
      
      - name: Notify success
        uses: 8398a7/action-slack@v3
        with:
          status: success
          text: '🚀 Production deployment successful!'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
      
      - name: Notify failure
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          text: '🔥 Production deployment failed! @channel'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### 4. Database Migration Strategy
```typescript
// scripts/migrate.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

interface MigrationStep {
  name: string;
  description: string;
  up: () => Promise<void>;
  down: () => Promise<void>;
  validateBefore?: () => Promise<boolean>;
  validateAfter?: () => Promise<boolean>;
}

const migrations: MigrationStep[] = [
  {
    name: '001_add_revenue_tracking',
    description: 'Add revenue tracking fields to posts table',
    validateBefore: async () => {
      // Check if migration is needed
      const tableInfo = await prisma.$queryRaw`
        SELECT column_name FROM information_schema.columns 
        WHERE table_name = 'posts' AND column_name = 'affiliate_revenue'
      `;
      return Array.isArray(tableInfo) && tableInfo.length === 0;
    },
    up: async () => {
      await prisma.$executeRaw`
        ALTER TABLE posts 
        ADD COLUMN affiliate_revenue DECIMAL(10,2) DEFAULT 0,
        ADD COLUMN conversion_rate DECIMAL(5,4) DEFAULT 0
      `;
    },
    down: async () => {
      await prisma.$executeRaw`
        ALTER TABLE posts 
        DROP COLUMN affiliate_revenue,
        DROP COLUMN conversion_rate
      `;
    },
    validateAfter: async () => {
      const result = await prisma.$queryRaw`
        SELECT column_name FROM information_schema.columns 
        WHERE table_name = 'posts' AND column_name IN ('affiliate_revenue', 'conversion_rate')
      `;
      return Array.isArray(result) && result.length === 2;
    }
  }
];

async function runMigrations() {
  console.log('🔄 Starting database migrations...');
  
  for (const migration of migrations) {
    console.log(`\n📋 Running migration: ${migration.name}`);
    console.log(`   Description: ${migration.description}`);
    
    try {
      // Validate pre-conditions
      if (migration.validateBefore) {
        const shouldRun = await migration.validateBefore();
        if (!shouldRun) {
          console.log('   ⏭️  Migration already applied, skipping');
          continue;
        }
      }
      
      // Run migration
      await migration.up();
      
      // Validate post-conditions
      if (migration.validateAfter) {
        const success = await migration.validateAfter();
        if (!success) {
          throw new Error('Migration validation failed');
        }
      }
      
      console.log('   ✅ Migration completed successfully');
      
    } catch (error) {
      console.error(`   ❌ Migration failed: ${error.message}`);
      
      // Attempt rollback
      try {
        console.log('   🔄 Attempting rollback...');
        await migration.down();
        console.log('   ✅ Rollback successful');
      } catch (rollbackError) {
        console.error(`   💥 Rollback failed: ${rollbackError.message}`);
      }
      
      throw error;
    }
  }
  
  console.log('\n🎉 All migrations completed successfully!');
}

if (require.main === module) {
  runMigrations()
    .catch(console.error)
    .finally(() => prisma.$disconnect());
}
```

### 5. Rollback Strategy
```yaml
# .github/workflows/rollback.yml
name: Emergency Rollback

on:
  workflow_dispatch:
    inputs:
      target_deployment:
        description: 'Target deployment to rollback to'
        required: true
        type: string
      reason:
        description: 'Reason for rollback'
        required: true
        type: string

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Validate rollback request
        run: |
          echo "🔄 Initiating rollback to: ${{ github.event.inputs.target_deployment }}"
          echo "📝 Reason: ${{ github.event.inputs.reason }}"
      
      - name: Notify team of rollback start
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              text: "🚨 EMERGENCY ROLLBACK INITIATED",
              attachments: [{
                color: "warning",
                fields: [{
                  title: "Target Deployment",
                  value: "${{ github.event.inputs.target_deployment }}",
                  short: true
                }, {
                  title: "Reason",
                  value: "${{ github.event.inputs.reason }}",
                  short: true
                }]
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
      
      - name: Rollback Vercel deployment
        run: |
          curl -X POST "https://api.vercel.com/v6/deployments" \
            -H "Authorization: Bearer ${{ secrets.VERCEL_TOKEN }}" \
            -d '{
              "name": "ai-tools",
              "deploymentId": "${{ github.event.inputs.target_deployment }}",
              "target": "production"
            }'
      
      - name: Rollback database if needed
        run: |
          # Run database rollback script
          curl -X POST "${{ secrets.DB_ROLLBACK_WEBHOOK }}" \
            -H "Authorization: Bearer ${{ secrets.DB_ROLLBACK_TOKEN }}" \
            -d '{
              "target": "${{ github.event.inputs.target_deployment }}",
              "reason": "${{ github.event.inputs.reason }}"
            }'
      