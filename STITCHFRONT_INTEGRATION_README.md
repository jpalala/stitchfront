# README.md

# 🛡️ StitchFront: Tech Design Guide

Welcome to the StitchFront integration. 

This guide helps you secure your middleware so that your **AI-generated UI assets** (CSS, JS, or JSON) are only served to authenticated Shopify stores.

## Table of Contents

1.  [Overview](#overview)
2.  [Environment Setup](#environment-setup)
3.  [The Verification Utility](#the-verification-utility)
4.  [Express Middleware Integration](#express-middleware-integration)
5.  [Connecting to Shopify CLI](#connecting-to-shopify-cli)

-----

## Overview

To ensure your server is never used to serve unauthorized assets, we use a **SHA-256 HMAC** verification. This allows your backend to verify that a request actually came from Shopify using your "Client Secret" as the shared key.

This works regardless of your UI choices—whether you are using **FRUI (React)**, **Tailwind CSS**, or your own custom CSS framework.

-----

##  Environment Setup

You must store your Shopify API Secret in your production environment (e.g., Vercel, Railway, or Heroku). **Never hardcode this value.**

1.  Go to your **Shopify Partner Dashboard**.
2.  Select your App -\> **API Keys**.
3.  Copy the **API Secret Key**.
4.  In your Vercel/Backend Dashboard, add a new Environment Variable:
      - **Key:** `SHOPIFY_API_SECRET`
      - **Value:** `your_copied_secret_here`

-----

## The Verification Utility

Create a file named `lib/shopifyVerify.ts` (or `.js`) in your backend source folder. This logic performs a "Constant-time comparison" to prevent timing attacks.

```typescript
import crypto from 'crypto';

/**
 * Validates Shopify HMAC to ensure "Zero Trust" 
 * between the Storefront and your Theme Server.
 */
export function verifyShopifyRequest(queryString: any, apiSecret: string): boolean {
  const { hmac, ...params } = queryString;
  
  if (!hmac) return false;

  // 1. Sort the query parameters alphabetically to match Shopify's signature
  const message = Object.keys(params)
    .sort()
    .map(key => `${key}=${params[key]}`)
    .join('&');

  // 2. Generate the SHA-256 HMAC using your Secret
  const generatedHmac = crypto
    .createHmac('sha256', apiSecret)
    .update(message)
    .digest('hex');

  // 3. Constant-time comparison to prevent timing attacks
  return crypto.timingSafeEqual(
    Buffer.from(generatedHmac),
    Buffer.from(hmac)
  );
}
```

-----

## Express Middleware Integration

Now, apply this to your Express backend. This middleware acts as a gatekeeper. If a hacker tries to access your **FRUI** components or **Tailwind** configs directly without a valid Shopify signature, they will receive a `403 Forbidden`.

```nodejs
import { verifyShopifyRequest } from './lib/shopifyVerify';

const app = express();

// Middleware to protect your StitchFront assets
const zeroTrustAuth = (req, res, next) => {
  const isVerified = verifyShopifyRequest(
    req.query, 
    process.env.SHOPIFY_API_SECRET
  );

  if (!isVerified) {
    return res.status(403).json({ error: 'Unauthorized: HMAC verification failed.' });
  }

  next();
};

// Example: Serving an AI-generated CSS Theme
app.get('/api/v1/theme.css', zeroTrustAuth, (req, res) => {
  // 1. Determine which mood board to use based on the query param
  // This allows the Shopify storefront to request ?mood=artsy or ?mood=fashion
  const selectedMood = req.query.mood || 'tech';
  const theme = MOOD_BOARDS[selectedMood] || MOOD_BOARDS.tech;

  // 2. Construct the CSS Variable Injector
  // These variables act as the "Source of Truth" for FRUI components or Tailwind 'var()' configs
  const cssContent = `
    :root {
      /* Base Design Tokens */
      --brand-primary: ${theme.primary};
      --brand-bg: ${theme.bg};
      --font-main: ${theme.font};
      --radius-ui: ${theme.radius};
      
      /* Component Level Mapping (FRUI / Your Framework) */
      --frui-button-bg: var(--brand-primary);
      --frui-card-radius: var(--radius-ui);
      --frui-input-border: var(--brand-primary);
    }

    body {
      background-color: var(--brand-bg);
      font-family: var(--font-main);
      color: ${selectedMood === 'tech' ? '#FFFFFF' : '#333333'};
      margin: 0;
      padding: 0;
      transition: background-color 0.3s ease;
    }
    
    /* Global Utility for AI-generated components */
    .stitch-container {
      max-width: 1200px;
      margin: 0 auto;
      padding: 2rem;
    }
  `.trim();

  // 3. Set proper headers and send
  // We use text/css so the browser interprets it as a stylesheet immediately
  res.setHeader('Content-Type', 'text/css');
  res.setHeader('Cache-Control', 'public, max-age=3600'); // Cache for 1 hour to improve performance
  
  res.status(200).send(cssContent);
});
```

-----

## 4\. Connecting to Shopify CLI

After you have run `shopify app init` and created your project (React Router, Remix, or Hydrogen), connect it to your zero-trust endpoint:

1.  In your Shopify App settings, set up an **App Proxy**.
      - **Subpath prefix:** `apps`
      - **Subpath:** `stitch-theme`
      - **Proxy URL:** `https://your-vercel-backend.com/api/v1`
2.  In your Shopify frontend code, request your styles via the proxy:
    ```html
    <link rel="stylesheet" href="/apps/stitch-theme/theme.css">
    ```

-----

### ✅ Success Checklist

  - [ ] `SHOPIFY_API_SECRET` is set in your Backend Environment.
  - [ ] `verifyShopifyRequest` is protecting your asset routes.
  - [ ] Your frontend framework (FRUI, Tailwind, etc.) is consuming the variables from the verified endpoint.
  - [ ] You have tested that the endpoint returns `403` when called directly without Shopify parameters.
