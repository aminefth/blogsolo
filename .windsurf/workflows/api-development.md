---
description: Développement API endpoints
---


# API Development Workflow

## Trigger Commands
- `@workflow:api` - Créer endpoint API standard
- `@workflow:revenue-api` - API générateur revenus
- `@workflow:analytics-api` - API tracking analytics

## Workflow Steps

### 1. Endpoint Design
```typescript
// Endpoint specification template
interface APIEndpoint {
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  path: string;
  description: string;
  authentication: boolean;
  rateLimit: number; // requests per minute
  businessImpact: 'revenue' | 'analytics' | 'core' | 'utility';
}

// Auto-generate OpenAPI specification
const endpointSpec = {
  '/api/posts': {
    get: {
      summary: 'Get blog posts',
      parameters: [
        { name: 'page', in: 'query', schema: { type: 'integer' } },
        { name: 'limit', in: 'query', schema: { type: 'integer' } },
        { name: 'category', in: 'query', schema: { type: 'string' } }
      ],
      responses: {
        200: { description: 'Success', content: { 'application/json': { schema: PostsResponse } } },
        400: { description: 'Bad Request' },
        500: { description: 'Internal Server Error' }
      }
    }
  }
};
```

### 2. Input Validation
```typescript
// Auto-generate Zod schemas
const CreatePostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(100),
  category: z.enum(['ai-tools', 'tutorials', 'news']),
  tags: z.array(z.string()).max(10),
  isPremium: z.boolean().default(false),
  affiliateLinks: z.array(z.object({
    text: z.string(),
    url: z.string().url(),
    commission: z.number().positive()
  })).optional()
});

// Validation middleware
const validateRequest = (schema: z.ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      const validated = schema.parse(req.body);
      req.body = validated;
      next();
    } catch (error) {
      res.status(400).json({
        success: false,
        error: 'Validation failed',
        details: error.errors
      });
    }
  };
};
```

### 3. Business Logic Implementation
```typescript
// Service layer with business rules
class PostService {
  async createPost(data: CreatePostData, authorId: string): Promise<Post> {
    // Business rules validation
    if (data.isPremium && !this.canCreatePremiumContent(authorId)) {
      throw new Error('Insufficient permissions for premium content');
    }

    // Revenue tracking
    if (data.affiliateLinks?.length > 0) {
      await this.trackAffiliateLinksCreation(data.affiliateLinks, authorId);
    }

    // Content optimization
    const optimizedContent = await this.optimizeForSEO(data.content);
    
    // Create post
    const post = await PostModel.create({
      ...data,
      content: optimizedContent,
      author: authorId,
      createdAt: new Date()
    });

    // Analytics tracking
    await this.trackEvent('post_created', {
      postId: post._id,
      category: data.category,
      isPremium: data.isPremium,
      affiliateLinksCount: data.affiliateLinks?.length || 0
    });

    return post;
  }

  private async optimizeForSEO(content: string): Promise<string> {
    // Auto-optimize content for SEO
    // Add internal links, optimize headings, etc.
    return content;
  }

  private async trackAffiliateLinksCreation(links: AffiliateLink[], authorId: string) {
    // Track potential revenue from affiliate links
    const totalCommission = links.reduce((sum, link) => sum + link.commission, 0);
    await AnalyticsService.track('affiliate_links_added', {
      authorId,
      count: links.length,
      totalCommission
    });
  }
}
```

### 4. Error Handling
```typescript
// Comprehensive error handling
const errorHandler = (error: Error, req: Request, res: Response, next: NextFunction) => {
  console.error('API Error:', error);

  // Business-specific error handling
  if (error instanceof ValidationError) {
    return res.status(400).json({
      success: false,
      error: 'Validation failed',
      code: 'VALIDATION_ERROR',
      details: error.details
    });
  }

  if (error instanceof AuthenticationError) {
    return res.status(401).json({
      success: false,
      error: 'Authentication required',
      code: 'AUTH_REQUIRED'
    });
  }

  if (error instanceof BusinessRuleError) {
    return res.status(422).json({
      success: false,
      error: error.message,
      code: 'BUSINESS_RULE_VIOLATION'
    });
  }

  // Generic error
  res.status(500).json({
    success: false,
    error: 'Internal server error',
    code: 'INTERNAL_ERROR',
    requestId: req.id
  });
};
```

### 5. Testing Suite
```typescript
// Auto-generate API tests
describe('POST /api/posts', () => {
  beforeEach(async () => {
    await clearDatabase();
    await seedTestData();
  });

  it('creates post successfully with valid data', async () => {
    const postData = {
      title: 'Test AI Tool Review',
      content: 'Comprehensive review content...',
      category: 'ai-tools',
      affiliateLinks: [{
        text: 'Try Tool',
        url: 'https://example.com',
        commission: 50
      }]
    };

    const response = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${userToken}`)
      .send(postData)
      .expect(201);

    expect(response.body.success).toBe(true);
    expect(response.body.data.title).toBe(postData.title);
    expect(response.body.data.affiliateLinks).toHaveLength(1);
  });

  it('tracks analytics for post creation', async () => {
    const analyticsTrackSpy = jest.spyOn(AnalyticsService, 'track');
    
    await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${userToken}`)
      .send(validPostData)
      .expect(201);

    expect(analyticsTrackSpy).toHaveBeenCalledWith('post_created', expect.objectContaining({
      category: 'ai-tools',
      affiliateLinksCount: 1
    }));
  });

  it('enforces business rules for premium content', async () => {
    const premiumPostData = { ...validPostData, isPremium: true };

    const response = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${basicUserToken}`)
      .send(premiumPostData)
      .expect(422);

    expect(response.body.code).toBe('BUSINESS_RULE_VIOLATION');
  });
});
```

### 6. Documentation Auto-Generation
```typescript
// Swagger/OpenAPI documentation
const generateAPIDoc = (endpoint: APIEndpoint) => ({
  [`${endpoint.path}`]: {
    [endpoint.method.toLowerCase()]: {
      summary: endpoint.description,
      tags: [endpoint.businessImpact],
      security: endpoint.authentication ? [{ bearerAuth: [] }] : [],
      'x-rate-limit': endpoint.rateLimit,
      responses: {
        200: { description: 'Success' },
        400: { description: 'Bad Request' },
        401: { description: 'Unauthorized' },
        500: { description: 'Internal Server Error' }
      }
    }
  }
});
```

## Output Files Generated
- `routes/api/endpoint.ts` - Route handler
- `services/EndpointService.ts` - Business logic
- `schemas/endpoint.schemas.ts` - Validation schemas
- `tests/api/endpoint.test.ts` - Comprehensive tests
- `docs/api/endpoint.md` - API documentation

## Activation Commands
- **"@workflow:api posts"** → Crée endpoint CRUD posts
- **"@workflow:revenue-api subscriptions"** → API gestion abonnements
- **"@workflow:analytics-api events"** → API tracking événements