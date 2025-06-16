---
description: Développement multilingue
---

# Multi-Language Development Workflow

## Trigger Commands
- `@workflow:i18n` - Setup internationalization complète
- `@workflow:translate` - Traduction automatique optimisée
- `@workflow:localize` - Localisation marché spécifique

## Internationalization Strategy

### 1. next-intl Configuration
```typescript
// next.config.js
const withNextIntl = require('next-intl/plugin')('./i18n.ts');

module.exports = withNextIntl({
  experimental: {
    appDir: true
  },
  i18n: {
    locales: ['fr', 'en', 'es', 'de', 'it'],
    defaultLocale: 'fr',
    localeDetection: true,
    domains: [
      {
        domain: 'ai-tools.fr',
        defaultLocale: 'fr',
      },
      {
        domain: 'ai-tools.com',
        defaultLocale: 'en',
      },
      {
        domain: 'ai-tools.es',
        defaultLocale: 'es',
      }
    ]
  }
});

// i18n.ts
import { getRequestConfig } from 'next-intl/server';
import { getUserLocale, getTimeZone, getCurrency } from './lib/locale-utils';

export default getRequestConfig(async ({ locale }) => {
  const messages = (await import(`./messages/${locale}.json`)).default;
  const marketConfig = getMarketConfig(locale);

  return {
    messages,
    timeZone: marketConfig.timeZone,
    currency: marketConfig.currency,
    formats: {
      dateTime: {
        short: {
          day: 'numeric',
          month: 'short',
          year: 'numeric'
        }
      },
      number: {
        currency: {
          style: 'currency',
          currency: marketConfig.currency
        }
      }
    }
  };
});
```

### 2. Translation Automation System
```typescript
// Translation service with quality control
class TranslationService {
  private translationProviders = {
    deepl: new DeepLTranslator(),
    openai: new OpenAITranslator(),
    google: new GoogleTranslator()
  };

  async translateContent(
    content: string, 
    sourceLocale: string, 
    targetLocales: string[],
    options: TranslationOptions = {}
  ): Promise<TranslationResult> {
    const results: Record<string, TranslatedContent> = {};

    for (const targetLocale of targetLocales) {
      try {
        // Use appropriate translator based on language pair and content type
        const translator = this.selectBestTranslator(sourceLocale, targetLocale, options.contentType);
        
        // Translate content
        const translation = await translator.translate(content, sourceLocale, targetLocale, {
          preserveFormatting: true,
          glossary: await this.getGlossary(sourceLocale, targetLocale),
          context: options.context
        });

        // Quality assessment
        const qualityScore = await this.assessTranslationQuality(
          content, 
          translation, 
          sourceLocale, 
          targetLocale
        );

        // Human review if quality is low
        if (qualityScore < 0.8) {
          translation.needsReview = true;
          await this.queueForHumanReview(translation, qualityScore);
        }

        // Market-specific adaptations
        const localizedTranslation = await this.localizeForMarket(translation, targetLocale);

        results[targetLocale] = {
          content: localizedTranslation,
          qualityScore,
          translator: translator.name,
          needsReview: qualityScore < 0.8,
          adaptations: await this.getMarketAdaptations(targetLocale)
        };

      } catch (error) {
        console.error(`Translation failed for ${targetLocale}:`, error);
        results[targetLocale] = {
          error: error.message,
          fallback: content // Use original content as fallback
        };
      }
    }

    return {
      sourceContent: content,
      sourceLocale,
      translations: results,
      metadata: {
        timestamp: new Date(),
        contentType: options.contentType,
        totalWords: this.countWords(content)
      }
    };
  }

  private selectBestTranslator(
    sourceLocale: string, 
    targetLocale: string, 
    contentType?: string
  ): Translator {
    // DeepL for European languages
    if (['fr', 'de', 'es', 'it'].includes(targetLocale)) {
      return this.translationProviders.deepl;
    }

    // OpenAI for technical/marketing content
    if (contentType === 'marketing' || contentType === 'technical') {
      return this.translationProviders.openai;
    }

    // Google as fallback
    return this.translationProviders.google;
  }

  private async assessTranslationQuality(
    original: string,
    translation: string,
    sourceLocale: string,
    targetLocale: string
  ): Promise<number> {
    // Multiple quality checks
    const checks = await Promise.all([
      this.checkGrammar(translation, targetLocale),
      this.checkTerminologyConsistency(original, translation, sourceLocale, targetLocale),
      this.checkCulturalAppropriateness(translation, targetLocale),
      this.checkTechnicalAccuracy(original, translation)
    ]);

    // Weighted average (grammar: 30%, terminology: 30%, cultural: 20%, technical: 20%)
    return (checks[0] * 0.3 + checks[1] * 0.3 + checks[2] * 0.2 + checks[3] * 0.2);
  }
}
```

### 3. Market-Specific Configuration
```typescript
// Market configuration for different locales
const marketConfigs = {
  'fr': {
    currency: 'EUR',
    symbol: '€',
    dateFormat: 'DD/MM/YYYY',
    timeZone: 'Europe/Paris',
    pricing: {
      pro: 14,
      premium: 29
    },
    paymentMethods: ['card', 'sepa', 'paypal'],
    legalRequirements: ['gdpr', 'french_commerce'],
    culturalAdaptations: {
      formality: 'formal', // Use "vous" instead of "tu"
      businessHours: '9h-17h',
      holidays: ['bastille_day', 'christmas', 'easter'],
      colorPreferences: ['blue', 'white', 'red'],
      contentStyle: 'detailed_explanations'
    },
    localCompetitors: ['tool-fr.com', 'outil-ia.fr'],
    keywordModifiers: ['gratuit', 'français', 'outil', 'logiciel']
  },

  'en': {
    currency: 'USD',
    symbol: '$',
    dateFormat: 'MM/DD/YYYY',
    timeZone: 'America/New_York',
    pricing: {
      pro: 19,
      premium: 39
    },
    paymentMethods: ['card', 'paypal', 'apple_pay'],
    legalRequirements: ['ccpa', 'coppa'],
    culturalAdaptations: {
      formality: 'casual',
      businessHours: '9AM-5PM',
      holidays: ['thanksgiving', 'july_4th', 'christmas'],
      colorPreferences: ['blue', 'green'],
      contentStyle: 'concise_benefits'
    },
    localCompetitors: ['tool.com', 'aitool.co'],
    keywordModifiers: ['free', 'best', 'tool', 'software', 'app']
  },

  'es': {
    currency: 'EUR',
    symbol: '€',
    dateFormat: 'DD/MM/YYYY',
    timeZone: 'Europe/Madrid',
    pricing: {
      pro: 12,
      premium: 25
    },
    paymentMethods: ['card', 'sepa', 'bizum'],
    legalRequirements: ['gdpr', 'spanish_commerce'],
    culturalAdaptations: {
      formality: 'mixed',
      businessHours: '9h-17h',
      holidays: ['dia_hispanidad', 'navidad', 'semana_santa'],
      colorPreferences: ['red', 'yellow', 'orange'],
      contentStyle: 'warm_personal'
    },
    localCompetitors: ['herramienta-ia.es', 'tools.es'],
    keywordModifiers: ['gratis', 'mejor', 'herramienta', 'aplicación']
  },

  'de': {
    currency: 'EUR',
    symbol: '€',
    dateFormat: 'DD.MM.YYYY',
    timeZone: 'Europe/Berlin',
    pricing: {
      pro: 17,
      premium: 34
    },
    paymentMethods: ['card', 'sepa', 'sofort', 'giropay'],
    legalRequirements: ['gdpr', 'german_privacy', 'data_protection'],
    culturalAdaptations: {
      formality: 'very_formal',
      businessHours: '9:00-17:00',
      holidays: ['oktoberfest', 'weihnachten', 'ostern'],
      colorPreferences: ['black', 'red', 'gold'],
      contentStyle: 'technical_precision'
    },
    localCompetitors: ['ki-tools.de', 'werkzeug.de'],
    keywordModifiers: ['kostenlos', 'deutsch', 'werkzeug', 'software']
  }
};

const useMarketConfig = (locale: string) => {
  return marketConfigs[locale] || marketConfigs['en'];
};
```

### 4. Localized SEO Optimization
```typescript
// SEO optimization per market
class LocalizedSEO {
  async optimizeForMarket(
    content: Content,
    targetLocale: string
  ): Promise<LocalizedSEOContent> {
    const marketConfig = useMarketConfig(targetLocale);
    
    // Localized keyword research
    const localKeywords = await this.researchLocalKeywords(
      content.keywords,
      targetLocale,
      marketConfig.keywordModifiers
    );

    // Competitor analysis for local market
    const localCompetitors = await this.analyzeLocalCompetitors(
      marketConfig.localCompetitors,
      localKeywords.primary
    );

    // Cultural content adaptation
    const culturallyAdaptedContent = await this.adaptContentCulturally(
      content.body,
      marketConfig.culturalAdaptations
    );

    // Local search optimization
    const localSearchOptimizations = await this.optimizeForLocalSearch(
      content,
      targetLocale,
      marketConfig
    );

    return {
      title: this.localizeTitle(content.title, localKeywords.primary, targetLocale),
      description: this.localizeDescription(content.description, localKeywords, targetLocale),
      content: culturallyAdaptedContent,
      keywords: localKeywords,
      hreflang: this.generateHreflang(content.slug, targetLocale),
      structuredData: this.generateLocalStructuredData(content, marketConfig),
      localOptimizations: localSearchOptimizations
    };
  }

  private async researchLocalKeywords(
    baseKeywords: string[],
    locale: string,
    modifiers: string[]
  ): Promise<LocalKeywordData> {
    const localKeywords = [];

    for (const keyword of baseKeywords) {
      // Translate base keyword
      const translatedKeyword = await this.translateKeyword(keyword, locale);
      
      // Add local modifiers
      const localVariations = modifiers.map(modifier => 
        `${translatedKeyword} ${modifier}`
      );

      // Research local search volume
      const searchData = await this.getLocalSearchVolume(
        [translatedKeyword, ...localVariations],
        locale
      );

      localKeywords.push(...searchData);
    }

    return {
      primary: localKeywords[0],
      secondary: localKeywords.slice(1, 5),
      longTail: localKeywords.slice(5)
    };
  }

  private generateLocalStructuredData(content: Content, marketConfig: MarketConfig) {
    return {
      "@context": "https://schema.org",
      "@type": "Article",
      "headline": content.title,
      "author": {
        "@type": "Organization",
        "name": "AI Tools Hub"
      },
      "publisher": {
        "@type": "Organization",
        "name": "AI Tools Hub",
        "logo": {
          "@type": "ImageObject",
          "url": `${process.env.SITE_URL}/logo-${marketConfig.locale}.png`
        }
      },
      "datePublished": content.publishedAt,
      "dateModified": content.updatedAt,
      "mainEntityOfPage": content.url,
      "offers": content.pricing ? {
        "@type": "Offer",
        "price": marketConfig.pricing.pro,
        "priceCurrency": marketConfig.currency,
        "availability": "https://schema.org/InStock"
      } : undefined
    };
  }
}
```

### 5. Content Management for Multiple Languages
```typescript
// Multi-language content editor
const MultiLanguageContentEditor = ({ postId }: { postId: string }) => {
  const [activeLocale, setActiveLocale] = useState('fr');
  const { data: post } = usePost(postId);
  const { mutate: updateTranslation } = useUpdateTranslation();
  const marketConfig = useMarketConfig(activeLocale);

  const handleContentChange = (locale: string, field: string, value: string) => {
    updateTranslation({
      postId,
      locale,
      field,
      value,
      marketOptimized: true
    });
  };

  const autoTranslateAll = async () => {
    const sourceLocale = 'fr';
    const targetLocales = ['en', 'es', 'de'];
    
    for (const targetLocale of targetLocales) {
      const translationResult = await TranslationService.translateContent(
        post.content,
        sourceLocale,
        [targetLocale],
        {
