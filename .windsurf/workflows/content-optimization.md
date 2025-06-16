---
description: Optimisation contenu SEO
---

# Content Optimization Workflow

## Trigger Commands
- `@workflow:seo` - Optimisation SEO complète
- `@workflow:content` - Création contenu optimisé
- `@workflow:keywords` - Recherche et optimisation mots-clés

## Content Strategy

### 1. SEO-Optimized Content Creation
```typescript
// SEO content optimizer
interface SEOContentData {
  title: string;
  content: string;
  targetKeywords: string[];
  category: string;
  language: string;
  competitorUrls?: string[];
}

class SEOContentOptimizer {
  async optimizeContent(data: SEOContentData): Promise<OptimizedContent> {
    // 1. Keyword analysis
    const keywordData = await this.analyzeKeywords(data.targetKeywords);
    
    // 2. Content structure optimization
    const structuredContent = await this.optimizeStructure(data.content, keywordData);
    
    // 3. Meta data generation
    const metaData = await this.generateMetaData(data.title, structuredContent, keywordData);
    
    // 4. Internal linking suggestions
    const internalLinks = await this.suggestInternalLinks(structuredContent);
    
    // 5. Readability analysis
    const readabilityScore = await this.analyzeReadability(structuredContent);
    
    return {
      optimizedContent: structuredContent,
      metaData,
      internalLinks,
      readabilityScore,
      seoScore: this.calculateSEOScore(keywordData, metaData, readabilityScore),
      suggestions: this.generateSuggestions(keywordData, readabilityScore)
    };
  }

  private async analyzeKeywords(keywords: string[]): Promise<KeywordAnalysis> {
    const analysis = [];
    
    for (const keyword of keywords) {
      // Simulate keyword research (integrate with real tools like SEMrush, Ahrefs)
      const data = await this.getKeywordData(keyword);
      analysis.push({
        keyword,
        searchVolume: data.volume,
        difficulty: data.difficulty,
        intent: this.classifyIntent(keyword),
        relatedKeywords: data.related
      });
    }
    
    return {
      primary: analysis[0],
      secondary: analysis.slice(1),
      longTail: this.generateLongTailKeywords(keywords)
    };
  }

  private async optimizeStructure(content: string, keywordData: KeywordAnalysis): Promise<string> {
    // Parse content structure
    const sections = this.parseContentSections(content);
    
    // Optimize headings
    const optimizedSections = sections.map(section => ({
      ...section,
      heading: this.optimizeHeading(section.heading, keywordData),
      content: this.optimizeParagraph(section.content, keywordData)
    }));
    
    // Add table of contents
    const toc = this.generateTableOfContents(optimizedSections);
    
    // Ensure proper keyword density (1-2%)
    const keywordDensity = this.calculateKeywordDensity(content, keywordData.primary.keyword);
    if (keywordDensity < 0.01) {
      // Suggest keyword additions
    }
    
    return this.reconstructContent(optimizedSections, toc);
  }

  private async generateMetaData(title: string, content: string, keywordData: KeywordAnalysis): Promise<MetaData> {
    return {
      title: this.optimizeTitle(title, keywordData.primary.keyword),
      description: this.generateMetaDescription(content, keywordData),
      keywords: [keywordData.primary.keyword, ...keywordData.secondary.map(k => k.keyword)],
      ogTitle: this.generateOGTitle(title, keywordData.primary.keyword),
      ogDescription: this.generateOGDescription(content),
      twitterCard: 'summary_large_image',
      canonicalUrl: this.generateCanonicalUrl(title),
      structuredData: this.generateStructuredData(title, content, keywordData)
    };
  }
}
```

### 2. Content Template System
```typescript
// Content templates for different types
const contentTemplates = {
  aiToolReview: {
    structure: [
      'introduction',
      'whatIsTheTool',
      'keyFeatures',
      'pricingAndPlans',
      'prosAndCons',
      'userExperience',
      'alternativesComparison',
      'conclusion',
      'faq'
    ],
    seoRequirements: {
      minWords: 2500,
      keywordDensity: { min: 0.01, max: 0.02 },
      headingStructure: 'h1 -> h2 -> h3',
      internalLinks: { min: 5, max: 10 },
      externalLinks: { min: 3, max: 7 }
    },
    monetization: {
      affiliateLinks: { optimal: 3, placement: ['introduction', 'pricingAndPlans', 'conclusion'] },
      ctaButtons: { count: 2, placement: ['keyFeatures', 'conclusion'] },
      emailCapture: { placement: 'conclusion' }
    }
  },

  tutorial: {
    structure: [
      'introduction',
      'prerequisites',
      'stepByStep',
      'troubleshooting',
      'conclusion',
      'nextSteps'
    ],
    seoRequirements: {
      minWords: 1500,
      keywordDensity: { min: 0.015, max: 0.025 },
      codeExamples: true,
      screenshots: { min: 3, max: 8 }
    }
  },

  newsArticle: {
    structure: [
      'headline',
      'summary',
      'details',
      'implications',
      'expertOpinions',
      'conclusion'
    ],
    seoRequirements: {
      minWords: 800,
      freshness: 'critical',
      sources: { min: 3 }
    }
  }
};

// Auto-generate content from template
class ContentGenerator {
  async generateFromTemplate(
    templateType: keyof typeof contentTemplates,
    data: ContentData
  ): Promise<GeneratedContent> {
    const template = contentTemplates[templateType];
    
    const sections = await Promise.all(
      template.structure.map(async (sectionType) => {
        return await this.generateSection(sectionType, data, template);
      })
    );

    const content = this.combineContentSections(sections);
    const optimizedContent = await new SEOContentOptimizer().optimizeContent({
      title: data.title,
      content,
      targetKeywords: data.keywords,
      category: data.category,
      language: data.language || 'fr'
    });

    return {
      ...optimizedContent,
      template: templateType,
      wordCount: this.countWords(content),
      estimatedReadingTime: this.calculateReadingTime(content)
    };
  }

  private async generateSection(
    sectionType: string,
    data: ContentData,
    template: any
  ): Promise<ContentSection> {
    // Use AI to generate section content based on type and data
    switch (sectionType) {
      case 'introduction':
        return this.generateIntroduction(data);
      case 'keyFeatures':
        return this.generateKeyFeatures(data);
      case 'pricingAndPlans':
        return this.generatePricingSection(data);
      // ... other section types
      default:
        return this.generateGenericSection(sectionType, data);
    }
  }
}
```

### 3. Competitor Analysis
```typescript
// Competitor content analysis
class CompetitorAnalyzer {
  async analyzeCompetitors(keyword: string, limit: number = 10): Promise<CompetitorAnalysis> {
    // Get top-ranking pages for keyword
    const topPages = await this.getTopRankingPages(keyword, limit);
    
    const analysis = await Promise.all(
      topPages.map(async (page) => {
        const content = await this.scrapePageContent(page.url);
        return {
          url: page.url,
          title: content.title,
          wordCount: this.countWords(content.text),
          headingStructure: this.analyzeHeadings(content.headings),
          keywordUsage: this.analyzeKeywordUsage(content.text, keyword),
          backlinks: page.backlinks,
          domainAuthority: page.domainAuthority,
          contentGaps: this.identifyContentGaps(content.text, keyword)
        };
      })
    );

    return {
      averageWordCount: this.calculateAverage(analysis.map(a => a.wordCount)),
      commonHeadings: this.findCommonHeadings(analysis),
      contentGaps: this.aggregateContentGaps(analysis),
      opportunityKeywords: this.findOpportunityKeywords(analysis),
      recommendations: this.generateRecommendations(analysis)
    };
  }

  private generateRecommendations(analysis: any[]): string[] {
    const recommendations = [];
    
    const avgWordCount = this.calculateAverage(analysis.map(a => a.wordCount));
    recommendations.push(`Viser ${Math.ceil(avgWordCount * 1.2)} mots pour surpasser la concurrence`);
    
    const commonTopics = this.findCommonTopics(analysis);
    recommendations.push(`Inclure les sujets: ${commonTopics.join(', ')}`);
    
    return recommendations;
  }
}
```

### 4. Content Performance Tracking
```typescript
// Content analytics and performance tracking
class ContentAnalytics {
  async trackContentPerformance(postId: string): Promise<ContentMetrics> {
    const metrics = await Promise.all([
      this.getTrafficMetrics(postId),
      this.getEngagementMetrics(postId),
      this.getConversionMetrics(postId),
      this.getSEOMetrics(postId)
    ]);

    return {
      traffic: metrics[0],
      engagement: metrics[1],
      conversions: metrics[2],
      seo: metrics[3],
      overallScore: this.calculateOverallScore(metrics),
      recommendations: this.generatePerformanceRecommendations(metrics)
    };
  }

  private async getTrafficMetrics(postId: string): Promise<TrafficMetrics> {
    return {
      pageViews: await this.getPageViews(postId),
      uniqueVisitors: await this.getUniqueVisitors(postId),
      avgSessionDuration: await this.getAvgSessionDuration(postId),
      bounceRate: await this.getBounceRate(postId),
      trafficSources: await this.getTrafficSources(postId)
    };
  }

  private async getConversionMetrics(postId: string): Promise<ConversionMetrics> {
    return {
      emailSignups: await this.getEmailSignups(postId),
      affiliateClicks: await this.getAffiliateClicks(postId),
      subscriptionConversions: await this.getSubscriptionConversions(postId),
      revenueGenerated: await this.getRevenueGenerated(postId),
      conversionRate: await this.getOverallConversionRate(postId)
    };
  }
}
```

### 5. Content Automation Workflows
```typescript
// Automated content workflows
class ContentWorkflow {
  async executeContentPipeline(topic: string, type: 'review' | 'tutorial' | 'news'): Promise<void> {
    // 1. Research phase
    const research = await this.conductResearch(topic, type);
    
    // 2. Content planning
    const contentPlan = await this.createContentPlan(research, type);
    
    // 3. Content generation
    const draftContent = await this.generateContent(contentPlan);
    
    // 4. SEO optimization
    const optimizedContent = await this.optimizeForSEO(draftContent);
    
    // 5. Quality review
    const qualityScore = await this.assessQuality(optimizedContent);
    
    // 6. Publish or queue for review
    if (qualityScore >= 8.0) {
      await this.schedulePublication(optimizedContent);
    } else {
      await this.queueForManualReview(optimizedContent, qualityScore);
    }
    
    // 7. Track performance
    setTimeout(() => {
      this.trackPerformance(optimizedContent.id);
    }, 24 * 60 * 60 * 1000); // Track after 24 hours
  }

  private async conductResearch(topic: string, type: string): Promise<ResearchData> {
    return {
      keywordAnalysis: await new KeywordResearcher().research(topic),
      competitorAnalysis: await new CompetitorAnalyzer().analyzeCompetitors(topic),
      trendAnalysis: await this.analyzeTrends(topic),
      audienceInsights: await this.getAudienceInsights(topic)
    };
  }
}
```

### 6. Content Calendar Integration
```typescript
// Content calendar with SEO planning
interface ContentCalendarEntry {
  id: string;
  title: string;
  type: 'review' | 'tutorial' | 'news' | 'comparison';
  targetKeywords: string[];
  scheduledDate: Date;
  author: string;
  status: 'planned' | 'in-progress' | 'review' | 'published';
  seoTarget: {
    difficulty: number;
    searchVolume: number;
    expectedTraffic: number;
  };
  monetization: {
    affiliateTools: string[];
    expectedRevenue: number;
  };
}

class ContentCalendar {
  async planMonthlyContent(): Promise<ContentCalendarEntry[]> {
    // Analyze trending keywords
    const trendingKeywords = await this.getTrendingKeywords();
    
    // Identify content gaps
    const contentGaps = 