# Architecture Decisions Record

## Technology Stack
- **Frontend**: Next.js 14 (App Router) + TypeScript + Tailwind CSS
- **Backend**: Express.js + MongoDB (existing foundation)
- **Authentication**: JWT with refresh tokens
- **Payments**: Stripe integration
- **Email**: ConvertKit for marketing automation
- **Analytics**: Custom dashboard + Google Analytics 4
- **Hosting**: Vercel (frontend) + Railway/DigitalOcean (backend)

## Key Architectural Decisions

### ADR-001: Next.js App Router over Pages Router
**Decision**: Use Next.js 14 App Router for new frontend
**Rationale**: 
- Better performance with Server Components
- Improved SEO capabilities
- Simplified data fetching patterns
- Future-proof architecture

### ADR-002: Monolithic Backend vs Microservices
**Decision**: Keep Express.js monolithic backend
**Rationale**:
- Existing codebase is well-structured
- Easier to maintain for small team
- Faster development initially
- Can extract services later if needed

### ADR-003: MongoDB Document Design
**Decision**: Embedded documents for blog posts with references for users
```typescript
// BlogPost Schema
{
  _id: ObjectId,
  title: String,
  content: String,
  author: ObjectId, // Reference to User
  translations: {
    de: { title: String, content: String, slug: String },
    es: { title: String, content: String, slug: String },
    en: { title: String, content: String, slug: String }
  },
  affiliateLinks: [{
    text: String,
    url: String,
    commission: Number,
    clicks: Number,
    conversions: Number
  }],
  seo: {
    metaTitle: String,
    metaDescription: String,
    keywords: [String]
  },
  analytics: {
    views: Number,
    readTime: Number,
    bounceRate: Number
  }
}