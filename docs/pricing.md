# OpenAI Image Proxy - Pricing Model

## Overview

Our service uses a simple credit-based pricing system. Credits are deducted based on actual token usage from OpenAI's gpt-image-1 model, ensuring fair and transparent pricing.

## How Pricing Works

### Token-Based Calculation

We calculate costs based on OpenAI's token pricing:
- Text input tokens: $5.00 per 1M tokens
- Image input tokens: $10.00 per 1M tokens  
- Image output tokens: $40.00 per 1M tokens

Your credit cost = (Actual OpenAI cost × 3.0) rounded up to the nearest credit

The 3x multiplier covers:
- Infrastructure costs (Cloudflare Workers, R2 storage, D1 database)
- Service maintenance and development
- Sustainable profit margin

### Typical Credit Costs

| Quality | Size | Typical Credits | Use Case |
|---------|------|-----------------|----------|
| Low | 1024×1024 | 3-5 | Quick drafts, thumbnails |
| Medium | 1024×1024 | 12-15 | Social media, web content |
| High | 1024×1024 | 50-55 | Print quality, detailed art |
| High | 1536×1024 | 75-80 | Wide format, banners |

**Image Editing**: Add 2-5 credits for input image processing

*Note: Actual costs vary slightly based on prompt complexity and length*

## Credit Packs

| Pack | Credits | Bonus | Value |
|------|---------|-------|-------|
| **Starter** | 150 | - | ~30 low or 11 medium images |
| **Basic** | 500 | 25 (5%) | ~38 medium images |
| **Popular** ⭐ | 1,250 | 114 (10%) | ~96 medium images |
| **Business** | 2,500 | 326 (15%) | ~192 medium images |
| **Enterprise** | 5,000 | 833 (20%) | ~384 medium images |


## Cost Comparison

### vs Direct OpenAI API
- **OpenAI**: Requires API key setup, technical knowledge, pay-as-you-go billing
- **Our Service**: Simple credit system, no setup required, includes storage & CDN

### vs Subscription Services
- **ChatGPT Plus**: Monthly subscription with usage limits
- **Midjourney**: Monthly subscription required
- **Our Service**: No subscription, pay only for what you use

### Example Monthly Usage

| Usage Level | Images/Month | Credits Needed |
|-------------|--------------|----------------|
| Casual | 50 medium | ~750 |
| Regular | 200 medium | ~3,000 |
| Power User | 500 medium | ~7,500 |
| Business | 1000+ medium | ~15,000+ |

## Features Included

Every credit purchase includes:
- ✅ Instant image generation via API
- ✅ 7-day cloud storage for all images
- ✅ CDN delivery for fast access
- ✅ Gallery and history tracking
- ✅ RESTful API compatible with OpenAI
- ✅ No monthly fees or subscriptions
- ✅ Usage analytics and reporting

## API Endpoints

### Check Credit Balance
```
GET /v1/credits/balance
Authorization: Bearer <your-api-key>
```

### Estimate Image Cost
```
POST /v1/credits/estimate
{
  "prompt": "Your prompt here",
  "quality": "medium",
  "size": "1024x1024"
}
```

### View Pricing
```
GET /v1/credits/pricing
```

### Purchase Credits
```
POST /v1/credits/purchase
{
  "pack_id": "popular",
  "payment_provider": "stripe",  // or "nowpayments" for crypto
  "payment_id": "",
  "payment_currency": "btc"  // only for crypto payments
}
```

## Billing Details

### Payment Methods
- **Credit/Debit Cards** via Stripe
  - All major cards accepted (Visa, Mastercard, American Express, Discover)
  - Available for all credit packs
  - Instant credit delivery upon successful payment
  - Secure checkout with 3D Secure authentication
- **Cryptocurrency** (BTC, ETH, DOGE, LTC) via NOWPayments
  - **Note**: Due to minimum transaction requirements, crypto payments are only available for Basic pack and above
  - QR code provided for easy mobile wallet scanning
  - Automatic confirmation upon blockchain verification
- No subscription required
- Secure payment processing

### Credit Expiration
- Credits never expire
- No monthly minimums
- Use at your own pace

### Refund Policy
- Unused credits refundable within 14 days
- No refunds for used credits
- Contact support for issues

## Business Model Transparency

We believe in transparent pricing. Here's how our 3x multiplier breaks down:

| Component | Percentage |
|-----------|------------|
| OpenAI API costs | 33.3% |
| Infrastructure & Storage | 5% |
| Payment processing (Stripe) | 3% |
| Payment processing (NOWPayments)* | 0.5-1% |
| Platform fees (App stores) | 15% |
| Development & Maintenance | 15% |
| Profit margin | 27.2-27.7% |

*NOWPayments cryptocurrency fee breakdown:
- Service fee: 0.5% (0.4-0.5% for high volume)
- Exchange fee: 0.5% (if currency conversion needed)
- Network fees: Variable by blockchain
  - **BTC, DOGE, LTC**: Low network fees
  - **ETH**: High gas fees, may make smaller transactions uneconomical

This ensures sustainable service while keeping prices fair for users.

## Questions?

- **Email**: support@your-domain.com
- **API Status**: https://status.your-domain.com
- **Documentation**: https://docs.your-domain.com

---

*Last updated: January 2025*