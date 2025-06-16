---
description: Features génératrices de revenus
---

# Revenue Feature Development Workflow

## Trigger Commands
- `@workflow:subscription` - Système abonnements
- `@workflow:affiliate` - Tracking affiliés
- `@workflow:analytics` - Analytics revenus

## Revenue-Focused Development

### 1. Subscription Feature Template
```typescript
// Subscription management component
interface SubscriptionManagerProps {
  currentPlan: 'free' | 'pro' | 'premium';
  onUpgrade: (plan: string) => void;
  onDowngrade: () => void;
  showTrialOffer?: boolean;
}

const SubscriptionManager = ({ 
  currentPlan, 
  onUpgrade, 
  onDowngrade,
  showTrialOffer = true 
}: SubscriptionManagerProps) => {
  const { trackConversion } = useAnalytics();

  const handleUpgrade = async (plan: string) => {
    trackConversion('subscription_upgrade_attempt', { from: currentPlan, to: plan });
    
    try {
      await onUpgrade(plan);
      trackConversion('subscription_upgrade_success', { from: currentPlan, to: plan });
    } catch (error) {
      trackConversion('subscription_upgrade_failed', { 
        from: currentPlan, 
        to: plan, 
        error: error.message 
      });
    }
  };

  return (
    <div className="subscription-manager">
      {currentPlan === 'free' && showTrialOffer && (
        <TrialOfferBanner onAccept={() => handleUpgrade('pro')} />
      )}
      
      <PricingCards 
        currentPlan={currentPlan}
        onSelectPlan={handleUpgrade}
        showAnnualDiscount={true}
        highlightMostPopular={true}
      />
      
      <PaymentMethods />
      <SecurityBadges />
      <MoneyBackGuarantee />
    </div>
  );
};
```

### 2. Affiliate Link Optimization
```typescript
// Smart affiliate link component
interface AffiliateTrackingProps {
  tool: string;
  commission: number;
  originalUrl: string;
  placement: 'header' | 'content' | 'footer' | 'sidebar';
  userId?: string;
}

const SmartAffiliateLink = ({ 
  tool, 
  commission, 
  originalUrl, 
  placement, 
  userId 
}: AffiliateTrackingProps) => {
  const [isClicked, setIsClicked] = useState(false);
  const { trackAffiliateClick } = useAffiliateTracking();

  const handleClick = async (e: React.MouseEvent) => {
    e.preventDefault();
    setIsClicked(true);

    // Track click with comprehensive data
    await trackAffiliateClick({
      tool,
      commission,
      placement,
      userId,
      timestamp: Date.now(),
      referrer: document.referrer,
      userAgent: navigator.userAgent
    });

    // Delayed redirect for tracking
    setTimeout(() => {
      window.open(originalUrl, '_blank', 'noopener,noreferrer');
    }, 100);
  };

  return (
    <a
      href={originalUrl}
      onClick={handleClick}
      className={cn(
        'affiliate-link',
        `placement-${placement}`,
        isClicked && 'clicked'
      )}
      data-tool={tool}
      data-commission={commission}
      rel="nofollow sponsored"
    >
      Try {tool} Free
      {commission > 100 && <span className="high-commission">💰</span>}
    </a>
  );
};
```

### 3. Revenue Analytics Dashboard
```typescript
// Revenue metrics component
const RevenueDashboard = () => {
  const { data: revenueData, isLoading } = useRevenueMetrics();
  const { data: topPerformers } = useTopPerformingContent();

  if (isLoading) return <DashboardSkeleton />;

  return (
    <div className="revenue-dashboard">
      <RevenueOverview 
        mrr={revenueData?.mrr}
        growth={revenueData?.growth}
        churn={revenueData?.churn}
      />
      
      <RevenueBreakdown
        subscription={revenueData?.subscription}
        affiliate={revenueData?.affiliate}
        sponsored={revenueData?.sponsored}
      />
      
      <TopPerformingContent 
        posts={topPerformers?.posts}
        affiliateLinks={topPerformers?.affiliateLinks}
      />
      
      <ConversionFunnels />
      <RevenueForecasting />
    </div>
  );
};

// Revenue tracking hooks
const useRevenueMetrics = () => {
  return useQuery({
    queryKey: ['revenue-metrics'],
    queryFn: async () => {
      const response = await fetch('/api/analytics/revenue');
      return response.json();
    },
    refetchInterval: 5 * 60 * 1000 // 5 minutes
  });
};
```

### 4. A/B Testing Infrastructure
```typescript
// A/B testing for revenue optimization
const useABTest = (testName: string, variants: string[]) => {
  const [variant, setVariant] = useState<string>('');
  const { userId } = useAuth();

  useEffect(() => {
    const assignedVariant = getABTestVariant(testName, userId, variants);
    setVariant(assignedVariant);
    
    // Track assignment
    analytics.track('ab_test_assigned', {
      testName,
      variant: assignedVariant,
      userId
    });
  }, [testName, userId]);

  const trackConversion = (conversionType: string, value?: number) => {
    analytics.track('ab_test_conversion', {
      testName,
      variant,
      conversionType,
      value,
      userId
    });
  };

  return { variant, trackConversion };
};

// Usage in pricing component
const PricingPage = () => {
  const { variant, trackConversion } = useABTest('pricing_layout', ['original', 'emphasize_annual']);

  const handleSubscribe = (plan: string) => {
    trackConversion('subscription', plan === 'pro' ? 14 : 0);
    // Handle subscription logic
  };

  return (
    <div className={`pricing-layout ${variant}`}>
      {variant === 'emphasize_annual' ? (
        <AnnualFirstPricing onSubscribe={handleSubscribe} />
      ) : (
        <StandardPricing onSubscribe={handleSubscribe} />
      )}
    </div>
  );
};
```

### 5. Conversion Optimization
```typescript
// Conversion-focused components
const ConversionOptimizedButton = ({ 
  children, 
  variant = 'primary',
  conversionGoal,
  ...props 
}) => {
  const { trackConversion } = useAnalytics();
  
  const handleClick = (e) => {
    trackConversion(conversionGoal, { variant, timestamp: Date.now() });
    props.onClick?.(e);
  };

  return (
    <button
      {...props}
      onClick={handleClick}
      className={cn(
        'conversion-button',
        `variant-${variant}`,
        'hover:scale-105 transition-transform'
      )}
    >
      {children}
    </button>
  );
};

// Lead capture form
const LeadCaptureForm = ({ source, incentive }: { source: string, incentive?: string }) => {
  const [email, setEmail] = useState('');
  const { trackConversion } = useAnalytics();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    try {
      await fetch('/api/leads', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, source, incentive })
      });

      trackConversion('email_capture', { source, incentive });
      setEmail('');
      // Show success message
    } catch (error) {
      // Handle error
    }
  };

  return (
    <form onSubmit={handleSubmit} className="lead-capture-form">
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Votre email pour recevoir les outils AI gratuits"
        required
      />
      <ConversionOptimizedButton 
        type="submit"
        conversionGoal="email_capture"
      >
        Recevoir les outils 🚀
      </ConversionOptimizedButton>
    </form>
  );
};
```

### 6. Revenue Tracking Service
```typescript
// Comprehensive revenue tracking
class RevenueTrackingService {
  static async trackSubscription(data: {
    userId: string;
    plan: string;
    amount: number;
    currency: string;
    period: 'monthly' | 'annual';
  }) {
    await Promise.all([
      // Internal analytics
      analytics.track('subscription_created', data),
      
      // External services
      this.sendToStripe(data),
      this.updateUserSegment(data.userId, data.plan),
      this.triggerWelcomeSequence(data.userId, data.plan)
    ]);
  }

  static async trackAffiliateConversion(data: {
    clickId: string;
    tool: string;
    commission: number;
    userId?: string;
  }) {
    await analytics.track('affiliate_conversion', {
      ...data,
      timestamp: Date.now(),
      revenue: data.commission
    });
  }

  static async calculateMRR(): Promise<number> {
    // Calculate Monthly Recurring Revenue
    const subscriptions = await this.getActiveSubscriptions();
    return subscriptions.reduce((total, sub) => {
      const monthlyAmount = sub.period === 'annual' ? sub.amount / 12 : sub.amount;
      return total + monthlyAmount;
    }, 0);
  }
}
```

## Business Metrics Tracking
- **Monthly Recurring Revenue (MRR)**
- **Customer Acquisition Cost (CAC)**
- **Lifetime Value (LTV)**
- **Churn rate by plan**
- **Conversion rates by funnel stage**
- **Revenue per visitor**
- **Affiliate click-to-conversion rates**

## Output Files Generated
- `components/revenue/SubscriptionManager.tsx`
- `components/revenue/AffiliateTracker.tsx`
- `hooks/useRevenueTracking.ts`
- `services/RevenueService.ts`
- `tests/revenue/revenue.test.ts`

## Activation Commands
- **"@workflow:subscription"** → Setup système d'abonnements complet
- **"@workflow:affiliate ChatGPT"** → Tracking liens affiliés pour outil
- **"@workflow:analytics revenue"** → Dashboard analytics revenus