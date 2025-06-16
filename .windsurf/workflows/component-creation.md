---
description: Création composants optimisés
---

# Component Creation Workflow

## Trigger Commands
- `@workflow:component` - Créer composant standard
- `@workflow:conversion-component` - Composant optimisé conversion
- `@workflow:revenue-component` - Composant générateur revenus

## Workflow Steps

### 1. Requirements Analysis
- [ ] **Business purpose**: Quel objectif business ?
- [ ] **User interaction**: Comment l'utilisateur l'utilise ?
- [ ] **Conversion goal**: Impact sur revenus ?
- [ ] **Performance requirements**: Contraintes performance ?

### 2. Component Architecture
```typescript
// Standard structure template
interface ComponentProps {
  // Required props
  id?: string;
  className?: string;
  children?: React.ReactNode;
  
  // Business props
  trackingId?: string;
  conversionGoal?: string;
  testVariant?: 'A' | 'B';
  
  // Callback props
  onInteraction?: (event: InteractionEvent) => void;
  onConversion?: (data: ConversionData) => void;
}

const Component = memo<ComponentProps>(({
  id,
  className = '',
  children,
  trackingId,
  conversionGoal,
  testVariant = 'A',
  onInteraction,
  onConversion,
  ...props
}) => {
  // Performance optimizations
  const memoizedValue = useMemo(() => expensiveCalculation(), [deps]);
  const stableCallback = useCallback((data) => {
    onInteraction?.(data);
    trackEvent('component_interaction', { trackingId, conversionGoal });
  }, [trackingId, conversionGoal, onInteraction]);

  return (
    <div 
      id={id}
      className={cn('component-base', className)}
      data-testid={`component-${trackingId}`}
      data-test-variant={testVariant}
      {...props}
    >
      {children}
    </div>
  );
});

Component.displayName = 'Component';
export default Component;
export type { ComponentProps };
```

### 3. Analytics Integration
```typescript
// Auto-include analytics hooks
const useComponentAnalytics = (componentName: string, trackingId?: string) => {
  const trackInteraction = useCallback((action: string, data?: object) => {
    analytics.track('component_interaction', {
      component: componentName,
      action,
      trackingId,
      timestamp: Date.now(),
      ...data
    });
  }, [componentName, trackingId]);

  const trackConversion = useCallback((conversionType: string, value?: number) => {
    analytics.track('conversion', {
      component: componentName,
      type: conversionType,
      value,
      trackingId,
      timestamp: Date.now()
    });
  }, [componentName, trackingId]);

  return { trackInteraction, trackConversion };
};
```

### 4. Testing Setup
```typescript
// Auto-generate tests
describe('Component', () => {
  const defaultProps = {
    trackingId: 'test-component',
    onInteraction: jest.fn(),
    onConversion: jest.fn()
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders with required props', () => {
    render(<Component {...defaultProps} />);
    expect(screen.getByTestId('component-test-component')).toBeInTheDocument();
  });

  it('tracks interactions correctly', () => {
    const { trackInteraction } = useComponentAnalytics('Component', 'test');
    // Test interaction tracking
  });

  it('handles conversion events', () => {
    const { trackConversion } = useComponentAnalytics('Component', 'test');
    // Test conversion tracking
  });

  it('supports A/B testing variants', () => {
    render(<Component {...defaultProps} testVariant="B" />);
    expect(screen.getByTestId('component-test-component')).toHaveAttribute('data-test-variant', 'B');
  });
});
```

### 5. Documentation Generation
```markdown
# ComponentName

## Purpose
[Auto-generate from requirements analysis]

## Usage
```typescript
import Component from '@/components/Component';

<Component
  trackingId="unique-identifier"
  conversionGoal="email_signup"
  onConversion={(data) => console.log('Converted!', data)}
>
  Content here
</Component>
```

## Props
[Auto-generate props table]

## Analytics Events
- `component_interaction`: Fired on user interactions
- `conversion`: Fired when conversion goal reached

## A/B Testing
Component supports variant testing via `testVariant` prop.
```

### 6. Storybook Integration
```typescript
// Auto-generate stories
export default {
  title: 'Components/ComponentName',
  component: Component,
  parameters: {
    docs: {
      description: {
        component: 'Auto-generated component description'
      }
    }
  }
};

export const Default = {
  args: {
    trackingId: 'storybook-component',
    children: 'Default content'
  }
};

export const ConversionOptimized = {
  args: {
    trackingId: 'conversion-component',
    conversionGoal: 'email_signup',
    testVariant: 'A',
    children: 'Conversion-focused content'
  }
};
```

## Output Files Generated
- `components/ComponentName/index.tsx` - Main component
- `components/ComponentName/ComponentName.test.tsx` - Tests
- `components/ComponentName/ComponentName.stories.tsx` - Storybook
- `components/ComponentName/types.ts` - TypeScript definitions
- `components/ComponentName/README.md` - Documentation

## Activation Commands
- **"@workflow:component Button"** → Crée bouton optimisé conversion
- **"@workflow:conversion-component EmailCapture"** → Composant capture email
- **"@workflow:revenue-component PricingCard"** → Carte de prix avec analytics