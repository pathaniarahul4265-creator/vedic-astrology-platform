# Vedic Astrology Platform - Complete Setup Guide

## 🚀 Quick Start

### 1. Prerequisites
- Node.js 18+ (LTS recommended)
- npm or yarn
- Supabase account (free tier works)
- Anthropic API key (for Claude AI integration)

### 2. Project Structure
```
vedic-astrology-platform/
├── app/
│   ├── layout.tsx
│   ├── page.tsx (main platform)
│   ├── api/
│   │   ├── birth-chart/route.ts
│   │   ├── interpretation/route.ts
│   │   ├── kundli-match/route.ts
│   │   ├── pdf-report/route.ts
│   │   └── user/route.ts
│   └── dashboard/
│       ├── page.tsx
│       ├── daily-insights/page.tsx
│       ├── kundli-matching/page.tsx
│       └── ai-guide/page.tsx
├── lib/
│   ├── vedic-calculations.ts
│   ├── ai-interpreter.ts
│   ├── database.ts
│   ├── pdf-generator.ts
│   └── utils.ts
├── components/
│   ├── cosmic-background.tsx
│   ├── zodiac-wheel.tsx
│   ├── glass-card.tsx
│   ├── birth-chart-display.tsx
│   └── ui/
├── public/
├── .env.local (create this)
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

### 3. Installation

```bash
# Create Next.js project
npx create-next-app@latest vedic-astrology-platform --typescript

cd vedic-astrology-platform

# Install dependencies
npm install

# Install required packages
npm install framer-motion \
  @anthropic-ai/sdk \
  @supabase/supabase-js \
  jspdf \
  html2canvas \
  recharts \
  swisseph \
  lucide-react
```

### 4. Environment Variables (.env.local)

Create `.env.local` in project root:

```env
# Anthropic API for AI Interpretations
NEXT_PUBLIC_ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxx
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxx

# Supabase Database
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxxxxxxxxxxxx
SUPABASE_SERVICE_ROLE_KEY=xxxxxxxxxxxxx

# App Settings
NEXT_PUBLIC_APP_NAME=Vedic Astrology Platform
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### 5. Next.js Configuration (next.config.js)

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  images: {
    unoptimized: true,
  },
  headers: async () => {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=3600, stale-while-revalidate=86400'
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

### 6. Tailwind Configuration (tailwind.config.ts)

```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        'deep-space': '#0A0E27',
        'sacred-gold': '#D4AF37',
        'cosmic-purple': '#6B5B95',
        'light-text': '#E8E6E1',
        'dark-text': '#1A1A2E',
      },
      fontFamily: {
        display: ['Playfair Display', 'serif'],
        body: ['Inter', 'sans-serif'],
        accent: ['Satisfy', 'cursive'],
      },
      backdropBlur: {
        'xl': '20px',
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
  ],
}
export default config
```

### 7. TypeScript Configuration (tsconfig.json)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "jsxImportSource": "react",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["app", "components", "lib"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## 📦 Package.json Template

```json
{
  "name": "vedic-astrology-platform",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev -p 3000",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "type-check": "tsc --noEmit",
    "format": "prettier --write ."
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "next": "^14.0.0",
    "typescript": "^5.3.0",
    "framer-motion": "^10.16.0",
    "@anthropic-ai/sdk": "^0.11.0",
    "@supabase/supabase-js": "^2.38.0",
    "jspdf": "^2.5.1",
    "html2canvas": "^1.4.1",
    "recharts": "^2.10.0",
    "swisseph": "^2.2.0",
    "lucide-react": "^0.294.0",
    "tailwindcss": "^3.3.0",
    "@tailwindcss/forms": "^0.5.7",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "@types/node": "^20.10.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@types/jspdf": "^2.5.0",
    "prettier": "^3.1.0",
    "eslint": "^8.55.0",
    "eslint-config-next": "^14.0.0"
  }
}
```

## 🗄️ Supabase Database Schema

### Run in Supabase SQL Editor:

```sql
-- Users Table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR UNIQUE NOT NULL,
  full_name VARCHAR NOT NULL,
  gender VARCHAR(20),
  date_of_birth DATE,
  time_of_birth TIME,
  birth_place VARCHAR,
  current_city VARCHAR,
  current_country VARCHAR,
  occupation VARCHAR,
  relationship_status VARCHAR(20),
  preferred_language VARCHAR(10),
  latitude DECIMAL(10, 8),
  longitude DECIMAL(11, 8),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Birth Charts Table
CREATE TABLE birth_charts (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  sun_sign VARCHAR(20),
  sun_degree DECIMAL(5, 2),
  moon_sign VARCHAR(20),
  moon_degree DECIMAL(5, 2),
  lagna_sign VARCHAR(20),
  lagna_degree DECIMAL(5, 2),
  mahadasha VARCHAR(20),
  antardasha VARCHAR(20),
  yogas TEXT[],
  doshas TEXT[],
  nakshatras JSONB,
  house_placements JSONB,
  planet_strengths JSONB,
  chart_data JSONB,
  generated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Interpretations Table
CREATE TABLE interpretations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  birth_chart_id UUID REFERENCES birth_charts(id) ON DELETE CASCADE,
  report_type VARCHAR(50), -- personality, career, relationships, etc
  content TEXT,
  generated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Kundli Matches Table
CREATE TABLE kundli_matches (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  partner_id UUID REFERENCES users(id) ON DELETE CASCADE,
  compatibility_score DECIMAL(3, 1),
  gun_milan_score INT,
  manglik_status VARCHAR(50),
  nadi_compatibility VARCHAR(50),
  match_analysis JSONB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Daily Insights Table
CREATE TABLE daily_insights (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  date DATE,
  daily_focus TEXT,
  energy_level INT,
  relationships_insight TEXT,
  career_insight TEXT,
  meditation_mantra VARCHAR(255),
  reflection_prompt TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_birth_charts_user_id ON birth_charts(user_id);
CREATE INDEX idx_interpretations_user_id ON interpretations(user_id);
CREATE INDEX idx_daily_insights_user_id ON daily_insights(user_id);
CREATE INDEX idx_daily_insights_date ON daily_insights(date);

-- Enable RLS (Row Level Security)
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE birth_charts ENABLE ROW LEVEL SECURITY;
ALTER TABLE interpretations ENABLE ROW LEVEL SECURITY;
ALTER TABLE kundli_matches ENABLE ROW LEVEL SECURITY;
ALTER TABLE daily_insights ENABLE ROW LEVEL SECURITY;

-- Create RLS policies
CREATE POLICY "Users can view their own data" ON users
  FOR SELECT USING (auth.uid()::text = id::text);

CREATE POLICY "Users can view their own birth charts" ON birth_charts
  FOR SELECT USING (auth.uid()::text = user_id::text);
```

## 🔌 API Route Examples

### app/api/birth-chart/route.ts

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { VedicAstrologyEngine } from '@/lib/vedic-calculations';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function POST(request: NextRequest) {
  try {
    const birthData = await request.json();
    const engine = new VedicAstrologyEngine();
    const astroData = engine.generate(birthData);

    // Save to database
    const { data, error } = await supabase
      .from('birth_charts')
      .insert([{
        user_id: birthData.userId,
        sun_sign: astroData.sunSign,
        sun_degree: astroData.sunDegree,
        moon_sign: astroData.moonSign,
        moon_degree: astroData.moonDegree,
        lagna_sign: astroData.lagnaSign,
        lagna_degree: astroData.lagnaDegree,
        mahadasha: astroData.mahadasha,
        antardasha: astroData.antardasha,
        yogas: astroData.yogas,
        doshas: astroData.doshas,
        chart_data: astroData,
      }]);

    if (error) throw error;

    return NextResponse.json({ success: true, data: astroData });
  } catch (error) {
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Unknown error' },
      { status: 500 }
    );
  }
}
```

### app/api/interpretation/route.ts

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { generateAIInterpretation } from '@/lib/ai-interpreter';

export async function POST(request: NextRequest) {
  try {
    const { birthData, astroData, reportType } = await request.json();

    const interpretation = await generateAIInterpretation({
      birthData,
      astroData,
      reportType,
    });

    return NextResponse.json({ interpretation });
  } catch (error) {
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Unknown error' },
      { status: 500 }
    );
  }
}
```

## 🎨 Premium Features Checklist

- [ ] Animated cosmic background with nebula
- [ ] Rotating zodiac wheel (signature element)
- [ ] Glassmorphism cards with premium styling
- [ ] Birth chart generation (Lagna, Moon, Sun)
- [ ] Nakshatras and planetary positions
- [ ] AI-powered personality interpretation
- [ ] Career compatibility analysis
- [ ] Relationship guidance
- [ ] Kundli matching module
- [ ] Daily insights generation
- [ ] AI astro guide (chat interface)
- [ ] PDF report generation (100+ pages)
- [ ] User authentication (Supabase Auth)
- [ ] Dark/Light mode toggle
- [ ] Responsive design (mobile → desktop)
- [ ] PWA support (offline capability)
- [ ] SEO optimization
- [ ] Analytics integration
- [ ] Email notifications
- [ ] Subscription management (Paddle/Stripe)

## 🚀 Deployment

### Vercel (Recommended)
```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Set environment variables in Vercel dashboard
```

### Docker Deployment
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

## 📊 Performance Optimization

- Code splitting with Next.js dynamic imports
- Image optimization with next/image
- Lazy loading for components
- Caching strategy (ISR/SSR)
- Minification and compression
- Font optimization (system fonts primary)
- CSS-in-JS optimization
- Database query optimization
- API response caching

## 🔐 Security Best Practices

- Environment variables for sensitive data
- Supabase RLS (Row Level Security)
- CORS configuration
- Input validation and sanitization
- Rate limiting on API routes
- HTTPS enforcement
- Content Security Policy headers
- XSS protection
- CSRF tokens for forms

## 📱 Responsive Breakpoints

- Mobile: 320px - 640px
- Tablet: 640px - 1024px
- Desktop: 1024px+

## 🎯 Next Steps for Production

1. Set up Supabase project
2. Configure authentication
3. Add payment processing (Paddle/Stripe)
4. Implement email notifications
5. Set up error tracking (Sentry)
6. Configure analytics (Posthog/Mixpanel)
7. Add customer support (Intercom)
8. Set up CI/CD pipeline
9. Configure backup strategy
10. Launch and monitor

---

**Built with ❤️ for spiritual seekers and cosmic explorers**
