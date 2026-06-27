/**
 * VEDIC ASTROLOGY PLATFORM - DEPLOYMENT & CUSTOMIZATION GUIDE
 * Tailored for production deployment and custom enhancements
 */

# DEPLOYMENT & CUSTOMIZATION GUIDE

## 🚀 Quick Deployment to Vercel

### 1. Prepare Your Repository

```bash
# Initialize git repo if needed
git init
git add .
git commit -m "Initial Vedic Astrology Platform commit"

# Create GitHub repository and push
git remote add origin https://github.com/yourusername/vedic-astrology-platform.git
git branch -M main
git push -u origin main
```

### 2. Deploy to Vercel

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Or connect GitHub repo directly at vercel.com
# Vercel will auto-deploy on every push to main
```

### 3. Environment Variables in Vercel

In your Vercel dashboard:
```
NEXT_PUBLIC_ANTHROPIC_API_KEY=sk-ant-xxxxx
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxxxx
SUPABASE_SERVICE_ROLE_KEY=xxxxx
NEXT_PUBLIC_APP_URL=https://yourdomain.com
```

### 4. Custom Domain

- In Vercel dashboard → Settings → Domains
- Add your custom domain (e.g., astrology.yourdomain.com)
- Update DNS records as shown in Vercel

## 🛠️ Docker Deployment

### Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

# Build the application
RUN npm run build

# Production stage
FROM node:18-alpine

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --only=production

COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public

EXPOSE 3000

ENV NODE_ENV=production

CMD ["npm", "start"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_ANTHROPIC_API_KEY: ${NEXT_PUBLIC_ANTHROPIC_API_KEY}
      NEXT_PUBLIC_SUPABASE_URL: ${NEXT_PUBLIC_SUPABASE_URL}
      NEXT_PUBLIC_SUPABASE_ANON_KEY: ${NEXT_PUBLIC_SUPABASE_ANON_KEY}
      SUPABASE_SERVICE_ROLE_KEY: ${SUPABASE_SERVICE_ROLE_KEY}
    depends_on:
      - db

  db:
    image: supabase/postgres:latest
    environment:
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Deploy with Docker

```bash
# Build
docker build -t vedic-astrology-platform .

# Run
docker run -p 3000:3000 \
  -e NEXT_PUBLIC_ANTHROPIC_API_KEY=your_key \
  -e NEXT_PUBLIC_SUPABASE_URL=your_url \
  vedic-astrology-platform
```

## ☁️ AWS Deployment (ECS + RDS)

### 1. Push to ECR

```bash
# Create ECR repository
aws ecr create-repository --repository-name vedic-astrology

# Get login token
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin xxx.dkr.ecr.us-east-1.amazonaws.com

# Tag and push
docker tag vedic-astrology-platform:latest xxx.dkr.ecr.us-east-1.amazonaws.com/vedic-astrology:latest
docker push xxx.dkr.ecr.us-east-1.amazonaws.com/vedic-astrology:latest
```

### 2. Create ECS Task Definition

```json
{
  "family": "vedic-astrology",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "vedic-astrology-app",
      "image": "xxx.dkr.ecr.us-east-1.amazonaws.com/vedic-astrology:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "hostPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "NEXT_PUBLIC_ANTHROPIC_API_KEY",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:astrology-api-key"
        }
      ]
    }
  ]
}
```

### 3. Create Fargate Service

```bash
aws ecs create-service \
  --cluster vedic-astrology-cluster \
  --service-name vedic-astrology-service \
  --task-definition vedic-astrology:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxxx],securityGroups=[sg-xxxxx],assignPublicIp=ENABLED}" \
  --load-balancers "targetGroupArn=arn:aws:elasticloadbalancing:...,containerName=vedic-astrology-app,containerPort=3000"
```

## 🔄 CI/CD Pipeline with GitHub Actions

### .github/workflows/deploy.yml

```yaml
name: Deploy to Vercel

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x]

    steps:
      - uses: actions/checkout@v3
      
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run type checking
        run: npm run type-check

      - name: Build
        run: npm run build

      - name: Run tests (if available)
        run: npm test --if-present

      - name: Deploy to Vercel
        uses: vercel/action@master
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
```

## 🎨 Customization Guide

### 1. Change Color Scheme

**File: lib/design-tokens.ts**

```typescript
const DESIGN_TOKENS = {
  colors: {
    deepSpace: '#0A0E27',           // Change this
    sacredGold: '#D4AF37',          // Change this
    cosmicPurple: '#6B5B95',        // Change this
    lightText: '#E8E6E1',           // Change this
    // ...
  },
};

// Then update Tailwind config
// tailwind.config.ts:
theme: {
  extend: {
    colors: {
      'deep-space': '#0A0E27',
      'sacred-gold': '#D4AF37',
      // ...
    },
  },
}
```

### 2. Add Your Own Branding

**Update these files:**

```typescript
// app/layout.tsx
<title>Your Brand - Vedic Astrology</title>
<meta name="description" content="Your description" />

// public/manifest.json
{
  "name": "Your Brand Name",
  "short_name": "Brand",
  "description": "Your description",
  "theme_color": "#D4AF37"
}

// components/branding.tsx
export function Logo() {
  return <img src="/your-logo.svg" alt="Your Brand" />
}
```

### 3. Integrate Payment Processing

**Add Paddle (easier than Stripe for this use case):**

```typescript
// lib/paddle-client.ts
import { initializePaddle } from "@paddle/paddle-js";

export const paddle = await initializePaddle({
  token: process.env.NEXT_PUBLIC_PADDLE_CLIENT_TOKEN!,
  environment: "production",
});

export async function openCheckout(priceId: string) {
  paddle.Checkout.open({
    items: [{ priceId, quantity: 1 }],
    customer: {
      email: userEmail,
    },
  });
}
```

### 4. Add Email Notifications

**Using Supabase Functions + SendGrid:**

```typescript
// supabase/functions/send-email/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts"
import { Resend } from "https://esm.sh/resend@1.0.0"

const resend = new Resend(Deno.env.get("RESEND_API_KEY"))

serve(async (req) => {
  const { email, subject, html } = await req.json()

  const data = await resend.emails.send({
    from: "Vedic Astrology <noreply@astrology.yourdomain.com>",
    to: email,
    subject,
    html,
  })

  return new Response(JSON.stringify(data), {
    headers: { "Content-Type": "application/json" },
  })
})
```

### 5. Add Push Notifications

**Using Firebase Cloud Messaging:**

```typescript
// lib/fcm.ts
import { initializeApp } from 'firebase/app';
import { getMessaging, getToken, onMessage } from 'firebase/messaging';

const firebaseApp = initializeApp({
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
});

const messaging = getMessaging(firebaseApp);

export async function requestNotificationPermission() {
  try {
    const permission = await Notification.requestPermission();
    if (permission === 'granted') {
      const token = await getToken(messaging, {
        vapidKey: process.env.NEXT_PUBLIC_FIREBASE_VAPID_KEY,
      });
      return token;
    }
  } catch (error) {
    console.error('Failed to get notification permission:', error);
  }
}

export function listenForMessages() {
  onMessage(messaging, (payload) => {
    console.log('Message received:', payload);
    // Handle foreground message
  });
}
```

## 📊 Analytics Integration

### Google Analytics

```typescript
// app/layout.tsx
import { GoogleAnalytics } from '@next/third-parties/google'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>{children}</body>
      <GoogleAnalytics gaId="G-XXXXXXXXXX" />
    </html>
  )
}
```

### Posthog (Better for SaaS)

```typescript
// lib/posthog.ts
import posthog from 'posthog-js'

export function initPosthog() {
  if (typeof window !== 'undefined') {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
    })
  }
}

// Track events
posthog.capture('chart_generated', {
  sunSign: 'Aries',
  userType: 'new',
})
```

## 🔍 SEO Optimization

### Add Open Graph Meta Tags

```typescript
// app/layout.tsx
export const metadata: Metadata = {
  title: 'Vedic Astrology Platform',
  description: 'Discover yourself through ancient wisdom and AI',
  openGraph: {
    title: 'Vedic Astrology Platform',
    description: 'AI-powered Vedic astrology readings',
    type: 'website',
    locale: 'en_US',
    url: 'https://yourdomain.com',
    images: [
      {
        url: 'https://yourdomain.com/og-image.png',
        width: 1200,
        height: 630,
      },
    ],
  },
}
```

### Add JSON-LD Schema

```typescript
// app/layout.tsx
<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: JSON.stringify({
      '@context': 'https://schema.org',
      '@type': 'Service',
      name: 'Vedic Astrology Platform',
      description: 'AI-powered Vedic astrology readings',
      url: 'https://yourdomain.com',
      image: 'https://yourdomain.com/og-image.png',
    }),
  }}
/>
```

## 🧪 Testing Setup

### Unit Tests with Vitest

```typescript
// lib/__tests__/vedic-calculations.test.ts
import { describe, it, expect } from 'vitest'
import VedicAstrologyEngine from '@/lib/vedic-calculations'

describe('VedicAstrologyEngine', () => {
  it('should calculate sun sign correctly', () => {
    const engine = new VedicAstrologyEngine()
    const date = new Date('1990-06-15')
    const result = engine.calculateSunSign(date)
    expect(result.sign).toBe('Gemini')
  })
})
```

### E2E Tests with Playwright

```typescript
// e2e/birth-chart.spec.ts
import { test, expect } from '@playwright/test'

test('should generate birth chart', async ({ page }) => {
  await page.goto('/')
  await page.click('text=Generate My Birth Chart')
  
  // Fill form
  await page.fill('input[name="fullName"]', 'John Doe')
  await page.fill('input[name="dateOfBirth"]', '1990-06-15')
  
  // Submit
  await page.click('text=Generate')
  
  // Verify chart appears
  await expect(page.locator('text=Gemini')).toBeVisible()
})
```

## 📈 Performance Monitoring

### Setup Sentry for Error Tracking

```bash
npm install @sentry/nextjs
```

```typescript
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  integrations: [
    new Sentry.Replay({
      maskAllText: true,
      blockAllMedia: true,
    }),
  ],
  tracesSampleRate: 1,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
});
```

## 🔐 Security Hardening

### Add Security Headers

```typescript
// next.config.js
async headers() {
  return [
    {
      source: '/:path*',
      headers: [
        {
          key: 'X-Content-Type-Options',
          value: 'nosniff'
        },
        {
          key: 'X-Frame-Options',
          value: 'DENY'
        },
        {
          key: 'X-XSS-Protection',
          value: '1; mode=block'
        },
        {
          key: 'Referrer-Policy',
          value: 'strict-origin-when-cross-origin'
        },
        {
          key: 'Permissions-Policy',
          value: 'camera=(), microphone=(), geolocation=(self)'
        },
      ],
    },
  ]
}
```

### Rate Limiting

```typescript
// lib/rate-limit.ts
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(100, "1 h"),
});

export async function checkRateLimit(identifier: string) {
  const { success } = await ratelimit.limit(identifier);
  return success;
}
```

## 📱 PWA Configuration

### Create manifest.json

```json
{
  "name": "Vedic Astrology Platform",
  "short_name": "Vedic Astrology",
  "description": "AI-powered Vedic astrology readings",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "background_color": "#0A0E27",
  "theme_color": "#D4AF37",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ]
}
```

## 🚨 Troubleshooting Common Issues

### 1. Anthropic API Rate Limiting
```typescript
// Add exponential backoff
async function callAnthropicWithRetry(params, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await anthropic.messages.create(params);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
      } else {
        throw error;
      }
    }
  }
}
```

### 2. Supabase Connection Issues
```typescript
// Add connection pool and retry logic
const supabase = createClient(url, key, {
  auth: {
    persistSession: true,
    autoRefreshToken: true,
  },
  realtime: {
    enabled: true,
  },
  db: {
    schema: 'public',
  },
});
```

### 3. PDF Generation Timeout
```typescript
// Use streaming for large PDFs
export async function generateLargePDF(data) {
  const generator = new PDFReportGenerator();
  // Generate in chunks
  for (const section of sections) {
    generator.addSection(section);
    // Yield control to prevent blocking
    await new Promise(resolve => setImmediate(resolve));
  }
  return generator.getBlob();
}
```

---

**You're now ready to deploy your premium Vedic Astrology platform to production!**
