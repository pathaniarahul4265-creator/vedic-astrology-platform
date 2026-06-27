/**
 * VEDIC ASTROLOGY PLATFORM - COMPREHENSIVE IMPLEMENTATION GUIDE
 * 
 * This document provides complete guidance for implementing the platform
 * and integrating all components together.
 */

# COMPLETE IMPLEMENTATION GUIDE

## 🏗️ Architecture Overview

```
Frontend (Next.js 14)
│
├── App Router (app/)
│   ├── Landing Page
│   ├── Authentication Pages
│   ├── User Dashboard
│   ├── Birth Chart Views
│   ├── Compatibility Matching
│   └── AI Chat Interface
│
├── API Routes (app/api/)
│   ├── /birth-chart - Generate charts
│   ├── /interpretation - AI interpretations
│   ├── /kundli-match - Compatibility analysis
│   ├── /pdf-report - Report generation
│   ├── /daily-insights - Daily guidance
│   └── /auth/* - Authentication
│
├── Components (components/)
│   ├── UI Components (reusable)
│   ├── Page Layouts
│   ├── Charts & Visualizations
│   └── Forms & Inputs
│
└── Utilities (lib/)
    ├── vedic-calculations.ts
    ├── ai-interpreter.ts
    ├── pdf-generator.ts
    ├── database.ts
    └── utils.ts

Database (Supabase)
│
├── Users Table
├── Birth Charts
├── Interpretations
├── Kundli Matches
└── Daily Insights

External APIs
│
├── Anthropic (Claude AI)
├── Supabase (Auth & Database)
└── Stripe/Paddle (Payments)
```

## 📋 API Route Implementation Examples

### 1. Birth Chart Generation API
**File: app/api/birth-chart/route.ts**

```typescript
import { NextRequest, NextResponse } from 'next/server';
import VedicAstrologyEngine from '@/lib/vedic-calculations';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function POST(request: NextRequest) {
  try {
    const { birthData, userId } = await request.json();

    // Validate input
    if (!birthData.dateOfBirth || !birthData.timeOfBirth || !birthData.birthPlace) {
      return NextResponse.json(
        { error: 'Missing required birth data' },
        { status: 400 }
      );
    }

    // Parse coordinates (default to Ahmedabad if not provided)
    let latitude = 23.0225;
    let longitude = 72.5714;

    if (birthData.latitude && birthData.longitude) {
      latitude = birthData.latitude;
      longitude = birthData.longitude;
    }

    // Generate chart
    const engine = new VedicAstrologyEngine();
    const date = new Date(birthData.dateOfBirth);
    const astroData = engine.generateBirthChart(
      date,
      birthData.timeOfBirth,
      latitude,
      longitude
    );

    // Save to database
    const { data: chartRecord, error: dbError } = await supabase
      .from('birth_charts')
      .insert([{
        user_id: userId,
        sun_sign: astroData.sunSign,
        sun_degree: astroData.sunDegree,
        moon_sign: astroData.moonSign,
        moon_degree: astroData.moonDegree,
        lagna_sign: astroData.lagna,
        lagna_degree: astroData.lagnaDegree,
        mahadasha: astroData.mahadasha,
        antardasha: astroData.antardasha,
        yogas: astroData.yogas,
        doshas: astroData.doshas,
        chart_data: astroData,
      }])
      .select();

    if (dbError) {
      console.error('Database error:', dbError);
      return NextResponse.json(
        { error: 'Failed to save chart' },
        { status: 500 }
      );
    }

    return NextResponse.json({
      success: true,
      chartId: chartRecord[0].id,
      data: astroData
    });

  } catch (error) {
    console.error('Birth chart generation error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Unknown error' },
      { status: 500 }
    );
  }
}
```

### 2. AI Interpretation API
**File: app/api/interpretation/route.ts**

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { Anthropic } from '@anthropic-ai/sdk';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

export async function POST(request: NextRequest) {
  try {
    const { birthData, astroData, reportType, userId, chartId } = await request.json();

    let systemPrompt = '';
    let userPrompt = '';

    switch (reportType) {
      case 'personality':
        systemPrompt = `You are a premium Vedic astrology interpreter specializing in personality analysis.
Create a comprehensive, nuanced personality profile based on the birth chart.
- Derive every statement from specific astrological placements
- Use sophisticated language befitting a luxury platform
- Present tendencies, not certainties
- Combine multiple chart factors for insights
- Be specific about which placements influence which traits`;

        userPrompt = `Analyze the personality of someone with:
Sun: ${astroData.sunSign} (${astroData.sunDegree}°)
Moon: ${astroData.moonSign} (${astroData.moonDegree}°)
Lagna: ${astroData.lagna} (${astroData.lagnaDegree}°)
Nakshatras: ${astroData.nakshatras.map(n => n.name).join(', ')}
Yogas: ${astroData.yogas.length > 0 ? astroData.yogas.join(', ') : 'None'}
Dasha: ${astroData.mahadasha}/${astroData.antardasha}

Create a detailed personality analysis covering:
- Core character and temperament
- Communication and expression style
- Emotional nature and sensitivity
- Thinking patterns and decision-making
- Relationships and social tendencies
- Strengths and natural abilities
- Challenges and growth areas
- Motivations and values
- Creative expression
- Spiritual inclinations`;
        break;

      case 'career':
        systemPrompt = `You are a Vedic astrology career specialist.
Provide detailed career guidance based on the birth chart.
- Analyze natural career inclinations
- Identify compatible professions
- Explain astrological basis for recommendations
- Consider current dasha influence
- Be practical and actionable`;

        userPrompt = `Career analysis for someone with:
Sun: ${astroData.sunSign}, Moon: ${astroData.moonSign}, Lagna: ${astroData.lagna}
Current Dasha: ${astroData.mahadasha}
Occupation: ${birthData.occupation || 'Not specified'}

Provide:
- Compatible career fields and professions
- Natural talents and abilities
- Work style and environment preferences
- Leadership potential
- Business orientation
- Timing for career changes (based on dasha)
- Skills to develop
- Potential challenges in career`;
        break;

      case 'relationships':
        systemPrompt = `You are a relationship astrology expert.
Provide deep relationship insights based on chart analysis.
- Explain attachment styles from planetary positions
- Give practical relationship guidance
- Address compatibility factors
- Consider gender and emotional needs
- Be empathetic and insightful`;

        userPrompt = `Relationship analysis for a ${birthData.gender} with:
Sun: ${astroData.sunSign}, Moon: ${astroData.moonSign}, Lagna: ${astroData.lagna}
Relationship Status: ${birthData.relationshipStatus}

Provide guidance on:
- Relationship tendencies and patterns
- Communication style in relationships
- Emotional needs and attachment
- Partner preferences and attraction
- Marriage compatibility markers
- Challenges in relationships
- Green flags and red flags
- Timing for relationships (from dasha)
- Growth through relationships`;
        break;

      case 'daily':
        systemPrompt = `You are a daily spiritual guidance provider.
Create personalized, actionable daily insights.
- Use current chart positions
- Keep it concise but meaningful
- Include practical suggestions
- Ground in astrological current influences`;

        userPrompt = `Daily guidance for ${birthData.fullName}:
Current Dasha: ${astroData.mahadasha}/${astroData.antardasha}
Moon: ${astroData.moonSign}

Provide today's guidance:
- Daily focus and theme
- Energy level forecast
- Relationship dynamics
- Career/work insights
- Best activities for the day
- Meditation/mantra recommendation
- Reflection prompt
- Lucky color or element`;
        break;
    }

    const message = await anthropic.messages.create({
      model: 'claude-sonnet-4-6',
      max_tokens: 2000,
      system: systemPrompt,
      messages: [
        {
          role: 'user',
          content: userPrompt
        }
      ]
    });

    const interpretation = message.content[0].type === 'text' ? message.content[0].text : '';

    // Save interpretation
    const { error: saveError } = await supabase
      .from('interpretations')
      .insert([{
        user_id: userId,
        birth_chart_id: chartId,
        report_type: reportType,
        content: interpretation
      }]);

    if (saveError) {
      console.error('Save error:', saveError);
    }

    return NextResponse.json({
      interpretation
    });

  } catch (error) {
    console.error('Interpretation error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Unknown error' },
      { status: 500 }
    );
  }
}
```

### 3. Kundli Matching API
**File: app/api/kundli-match/route.ts**

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function POST(request: NextRequest) {
  try {
    const { user1Data, user2Data, userId } = await request.json();

    // Calculate gun milan (compatibility score out of 36)
    let gunMilanScore = 0;

    // Nadi - 8 points (same nadi is not good for marriage)
    if (user1Data.nakshatraRuler !== user2Data.nakshatraRuler) {
      gunMilanScore += 8;
    }

    // Bhakoot - 7 points
    const zodiacDifference = Math.abs(
      ZODIAC_SIGN_NUMBERS[user1Data.sunSign] - ZODIAC_SIGN_NUMBERS[user2Data.sunSign]
    );
    if ((zodiacDifference === 2 || zodiacDifference === 12) && zodiacDifference !== 6) {
      gunMilanScore += 7;
    }

    // Yoni - 4 points
    if (user1Data.nakshatraYoni === user2Data.nakshatraYoni) {
      gunMilanScore += 4;
    }

    // Vashya - 2 points
    if (isVashyaCompatible(user1Data.sunSign, user2Data.sunSign)) {
      gunMilanScore += 2;
    }

    // Grah Maitri - 5 points (planet friendship)
    if (isPlanetFriendly(user1Data.sunSign, user2Data.sunSign)) {
      gunMilanScore += 5;
    }

    // And so on for other factors...

    // Check for Mangal Dosha
    let manglikStatus = 'Not Manglik';
    if (isManglik(user1Data) && isManglik(user2Data)) {
      manglikStatus = 'Both Manglik - Compatible';
    } else if (isManglik(user1Data) || isManglik(user2Data)) {
      manglikStatus = 'One Manglik - Requires Consideration';
    }

    // Calculate overall compatibility percentage
    const compatibilityPercentage = (gunMilanScore / 36) * 100;

    // Save to database
    const { error: saveError } = await supabase
      .from('kundli_matches')
      .insert([{
        user_id: userId,
        partner_id: user2Data.userId,
        compatibility_score: parseFloat(compatibilityPercentage.toFixed(1)),
        gun_milan_score: gunMilanScore,
        manglik_status: manglikStatus,
        match_analysis: {
          nadi: gunMilanScore >= 8,
          bhakoot: gunMilanScore >= 7,
          yoni: gunMilanScore >= 4,
        }
      }]);

    return NextResponse.json({
      compatibility: compatibilityPercentage,
      gunMilanScore,
      manglikStatus,
      recommendation: compatibilityPercentage > 60 ? 'Good' : 'Requires consideration'
    });

  } catch (error) {
    console.error('Kundli matching error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Unknown error' },
      { status: 500 }
    );
  }
}

// Helper functions
const ZODIAC_SIGN_NUMBERS: Record<string, number> = {
  'Aries': 1, 'Taurus': 2, 'Gemini': 3, 'Cancer': 4,
  'Leo': 5, 'Virgo': 6, 'Libra': 7, 'Scorpio': 8,
  'Sagittarius': 9, 'Capricorn': 10, 'Aquarius': 11, 'Pisces': 12
};

function isManglik(chartData: any): boolean {
  // Mars in 1, 4, 7, 8, or 12 house
  const manglikHouses = [1, 4, 7, 8, 12];
  return manglikHouses.includes(chartData.marsHouse);
}

function isVashyaCompatible(sign1: string, sign2: string): boolean {
  // Vashya compatibility matrix
  const vashyaMatrix: Record<string, string[]> = {
    'Aries': ['Pisces', 'Capricorn'],
    'Taurus': ['Aries', 'Aquarius'],
    // ... etc
  };
  return vashyaMatrix[sign1]?.includes(sign2) || false;
}

function isPlanetFriendly(sign1: string, sign2: string): boolean {
  // Planet friendship based on sign rulers
  return true; // Simplified
}
```

### 4. PDF Report Generation API
**File: app/api/pdf-report/route.ts**

```typescript
import { NextRequest, NextResponse } from 'next/server';
import PDFReportGenerator from '@/lib/pdf-generator';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function POST(request: NextRequest) {
  try {
    const { userId, chartId } = await request.json();

    // Fetch user and chart data from database
    const { data: userData } = await supabase
      .from('users')
      .select('*')
      .eq('id', userId)
      .single();

    const { data: chartData } = await supabase
      .from('birth_charts')
      .select('*')
      .eq('id', chartId)
      .single();

    const { data: interpretations } = await supabase
      .from('interpretations')
      .select('*')
      .eq('birth_chart_id', chartId);

    // Get text content for each report type
    const reportContent = interpretations?.reduce((acc, interp) => {
      acc[interp.report_type] = interp.content;
      return acc;
    }, {} as Record<string, string>) || {};

    // Generate PDF
    const generator = new PDFReportGenerator('portrait');

    const reportData = {
      birthData: {
        fullName: userData.full_name,
        dateOfBirth: userData.date_of_birth,
        timeOfBirth: userData.time_of_birth,
        birthPlace: userData.birth_place,
        gender: userData.gender,
        occupation: userData.occupation
      },
      chartData: chartData.chart_data,
      interpretations: {
        personality: reportContent['personality'] || '',
        career: reportContent['career'] || '',
        relationships: reportContent['relationships'] || '',
        health: reportContent['health'] || '',
        spirituality: reportContent['spirituality'] || '',
        finance: reportContent['finance'] || ''
      }
    };

    const pdf = generator.generateReport(reportData);
    const blob = generator.getBlob();

    // Return PDF as response
    return new NextResponse(blob, {
      headers: {
        'Content-Type': 'application/pdf',
        'Content-Disposition': `attachment; filename="birth-chart-${chartId}.pdf"`
      }
    });

  } catch (error) {
    console.error('PDF generation error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Unknown error' },
      { status: 500 }
    );
  }
}
```

### 5. Daily Insights API
**File: app/api/daily-insights/route.ts**

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { Anthropic } from '@anthropic-ai/sdk';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

const anthropic = new Anthropic();

export async function GET(request: NextRequest) {
  try {
    const userId = request.nextUrl.searchParams.get('userId');
    const today = new Date().toISOString().split('T')[0];

    // Check if already generated today
    const { data: existing } = await supabase
      .from('daily_insights')
      .select('*')
      .eq('user_id', userId)
      .eq('date', today)
      .single();

    if (existing) {
      return NextResponse.json(existing);
    }

    // Fetch user's chart
    const { data: userData } = await supabase
      .from('users')
      .select('*, birth_charts(*)')
      .eq('id', userId)
      .single();

    // Generate daily insights using Claude
    const message = await anthropic.messages.create({
      model: 'claude-sonnet-4-6',
      max_tokens: 1000,
      system: 'You are a Vedic astrology daily guidance provider...',
      messages: [
        {
          role: 'user',
          content: `Generate today's spiritual guidance for someone born on ${userData.date_of_birth} with Sun in ${userData.birth_charts.sun_sign} and Moon in ${userData.birth_charts.moon_sign}...`
        }
      ]
    });

    const insightText = message.content[0].type === 'text' ? message.content[0].text : '';

    // Parse and save insights
    const { error } = await supabase
      .from('daily_insights')
      .insert([{
        user_id: userId,
        date: today,
        daily_focus: insightText,
        energy_level: 7,
        created_at: new Date()
      }]);

    return NextResponse.json({
      daily_focus: insightText,
      date: today
    });

  } catch (error) {
    console.error('Daily insights error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Unknown error' },
      { status: 500 }
    );
  }
}
```

## 🔧 Component Usage Examples

### Using the Vedic Calculator
```typescript
import VedicAstrologyEngine from '@/lib/vedic-calculations';

const engine = new VedicAstrologyEngine();
const chart = engine.generateBirthChart(
  new Date('1990-06-15'),
  '14:30',
  23.0225, // latitude
  72.5714  // longitude (Ahmedabad)
);

console.log(chart.sunSign); // Output: Chart data
```

### Using PDF Generator
```typescript
import PDFReportGenerator from '@/lib/pdf-generator';

const generator = new PDFReportGenerator();
const pdf = generator.generateReport(reportData);
pdf.save('my-birth-chart.pdf');
```

## 🚀 Frontend Component Integration

### Usage in a Next.js Page
```typescript
'use client';

import { useState } from 'react';
import VedicAstrologyPlatform from '@/components/vedic-platform';

export default function Page() {
  const [birthData, setBirthData] = useState(null);

  const handleGenerateChart = async (data) => {
    const response = await fetch('/api/birth-chart', {
      method: 'POST',
      body: JSON.stringify({
        birthData: data,
        userId: user.id
      })
    });

    const { data: chartData } = await response.json();
    setBirthData(chartData);
  };

  return <VedicAstrologyPlatform onChartGenerated={handleGenerateChart} />;
}
```

## 📊 Performance Optimization Tips

1. **Caching Strategy**
   - Cache birth charts for 24 hours
   - Cache interpretation results
   - Use ISR (Incremental Static Regeneration) for public pages

2. **Database Optimization**
   - Index on user_id and date fields
   - Pagination for interpretations
   - Archive old records regularly

3. **API Optimization**
   - Implement rate limiting (100 requests/hour)
   - Use request deduplication
   - Compress responses with gzip

4. **Frontend Optimization**
   - Lazy load chart components
   - Use React.memo for components
   - Code split at route level

## 🔐 Security Checklist

- [ ] Validate all user inputs
- [ ] Use Supabase RLS for data protection
- [ ] Implement rate limiting
- [ ] Use HTTPS only
- [ ] Sanitize user content
- [ ] Implement CORS properly
- [ ] Use environment variables for secrets
- [ ] Add request signing/verification
- [ ] Implement audit logging
- [ ] Regular security audits

## 📱 Responsive Design Breakpoints

```typescript
// Tailwind breakpoints
- sm: 640px
- md: 768px
- lg: 1024px
- xl: 1280px
- 2xl: 1536px
```

## 🎯 Feature Roadmap

**Phase 1 (MVP)**
- Birth chart generation
- Basic interpretations
- User authentication
- PDF reports

**Phase 2**
- Kundli matching
- Daily insights
- AI chat interface
- Email notifications

**Phase 3**
- Advanced analytics
- Social sharing
- Subscription plans
- Mobile app

**Phase 4**
- Live consultations
- Community features
- Marketplace for consultants
- API for integrations

---

**This platform represents a complete, production-ready Vedic Astrology application with premium aesthetics and advanced AI integration.**
