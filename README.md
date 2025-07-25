# OpenAI Image Proxy

A high-performance Rust-based Cloudflare Worker that proxies OpenAI's gpt-image-1 model with enhanced features like automatic image storage, usage tracking, and public galleries.

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [CLI Tool](#cli-tool)
- [API Documentation](#api-documentation)
- [Development](#development)
- [Credits & Billing](#credits--billing)

## Features

- **OpenAI API Compatibility**: Full compatibility with OpenAI's gpt-image-1 image generation and editing APIs
- **Automatic Image Storage**: All generated images stored in Cloudflare R2 with public URLs
- **Usage Tracking**: Comprehensive usage analytics with D1 database
- **Public Gallery**: Browse all generated images with their prompts
- **High Performance**: Built with Rust for optimal WASM performance
- **Image Serving**: Direct image access through worker URLs
- **Credits System**: Token-based pricing with transparent credit packs
- **Device Authentication**: OAuth device flow for CLI and mobile apps
- **Multiple Deployment Modes**: Official hosted service or self-hosted options

## Architecture

- **Runtime**: Cloudflare Workers (Rust/WASM)
- **Database**: Cloudflare D1 (SQLite)
- **Storage**: Cloudflare R2 (S3-compatible)
- **Language**: Rust with worker-rs

<details>
<summary><b>Current Status</b></summary>

**Implemented:**
- gpt-image-1 image generation endpoint
- Image editing endpoint
- R2 storage with public URL access
- D1 database integration
- Public gallery endpoints
- Usage tracking and analytics
- OAuth authentication (Apple*, GitHub, Google)
  - *Apple Sign In not supported on Windows servers
- Admin dashboard in CLI
- Rate limiting (per-user concurrency control)
- Structured logging with JSON output

</details>

<details>
<summary><b>Deployment Modes</b></summary>

The service supports two deployment modes:

### Official Mode (Hosted Service)
- Managed credit system with payment processing
- No OpenAI API key required from users
- Automatic usage tracking and billing
- Suitable for SaaS deployment

### Self-Hosted Mode
- Users provide their own OpenAI API keys
- No credit system or payment processing
- Direct pass-through to OpenAI API
- Suitable for personal or enterprise deployment

Set the deployment mode in your environment:
```bash
npx wrangler secret put DEPLOYMENT_MODE # "official" or "self-hosted"
npx wrangler secret put REQUIRE_OWN_OPENAI_KEY # "true" for self-hosted
```

</details>

## Platform Notes

**Sign in with Apple**: Not supported on Windows servers due to cryptographic library dependencies. Windows users should use GitHub or Google authentication instead. This limitation applies to both the server deployment and CLI tool.

## Quick Start

<details>
<summary><b>Installation & Setup</b></summary>

1. Clone the repository:
```bash
git clone https://github.com/guitaripod/openai-image-proxy.git
cd openai-image-proxy
```

2. Install dependencies:
```bash
npm install
cargo install worker-build
```

3. Configure your `wrangler.toml` with your Cloudflare account details (see [docs/wrangler.toml.example](docs/wrangler.toml.example))

4. Create the D1 database:
```bash
npx wrangler d1 create openai-image-proxy
# Update the database_id in wrangler.toml
```

5. Run migrations:
```bash
npx wrangler d1 migrations apply DB --local
```

6. Create R2 bucket:
```bash
npx wrangler r2 bucket create openai-image-proxy-images
```

7. Configure environment variables and secrets:
   
   See [docs/ENVIRONMENT_VARIABLES.md](docs/ENVIRONMENT_VARIABLES.md) for the complete list of required environment variables and secrets.

   Quick setup for secrets:
   ```bash
   npx wrangler secret put OPENAI_API_KEY
   # OAuth secrets
   npx wrangler secret put GITHUB_CLIENT_SECRET
   npx wrangler secret put GOOGLE_CLIENT_SECRET
   npx wrangler secret put APPLE_PRIVATE_KEY
   # Stripe secrets (for official mode)
   npx wrangler secret put STRIPE_SECRET_KEY
   npx wrangler secret put STRIPE_WEBHOOK_SECRET
   ```

8. Deploy:
```bash
npx wrangler deploy
```

For detailed setup instructions, see [docs/SETUP.md](docs/SETUP.md).

</details>

## CLI Tool

A comprehensive command-line interface is available for managing the OpenAI Image Proxy service.

<details>
<summary><b>CLI Installation & Usage</b></summary>

### Installation

```bash
cd cli
cargo install --path .
# The CLI will be installed as 'pixie'
```

### Features

- Generate images from the command line with advanced options
- Edit existing images with high fidelity support
- Manage credits and view balance
- Admin commands for system management
- Device authentication support
- View transaction history
- Comprehensive help documentation
- Size aliases for easy use (square, landscape, portrait)
- Background control (transparent, white, black)
- Output format selection (PNG, JPEG, WebP)
- Compression settings for JPEG/WebP
- Moderation level control

### Basic Usage

```bash
# Authenticate with the service
pixie auth github
# or
pixie auth google

# Generate an image with size alias
pixie generate "A beautiful sunset" -s landscape -q medium -o ./images

# Generate with transparent background
pixie generate "logo design" -s square -b transparent -f png

# Generate with compression
pixie generate "product photo" -b white -f jpeg -c 85

# Edit an image with high fidelity
pixie edit photo.png "add company logo" --fidelity high -o ./edited

# Edit with size alias
pixie edit banner.jpg "enhance colors" -s landscape -q high

# Check credit balance
pixie credits

# Check API health
pixie health

# Browse gallery
pixie gallery list

# View help
pixie --help
```

### CLI Parameters

#### Size Options
- **Aliases**: `square`, `landscape`, `portrait`, `auto`
- **Explicit**: `1024x1024`, `1536x1024`, `1024x1536`
- Example: `pixie generate "icon" -s square`

#### Quality Levels
- `low` - Fast generation, ~4-6 credits
- `medium` - Balanced quality, ~16-24 credits  
- `high` - Maximum detail, ~62-94 credits
- `auto` - AI selects based on prompt

#### Background Options (Generation only)
- `auto` - AI decides (default)
- `transparent` - Transparent background (PNG/WebP only)
- `white` - White background
- `black` - Black background

#### Output Formats
- `png` - Default, supports transparency
- `jpeg` - Faster, smaller files
- `webp` - Modern format, supports transparency

#### Advanced Options
- `--compress` - JPEG/WebP compression (0-100)
- `--moderation` - Content filter level (`auto`, `low`)
- `--fidelity` - Edit mode only (`low`, `high`)

### New Commands

#### Health Check
```bash
# Check if the API is online and responding
pixie health
pixie health --api-url https://custom-api.com
```

#### Device Authentication Status
```bash
# Check the status of a device authentication flow
pixie auth device-status DEVICE-CODE-HERE

# This is primarily for debugging/support purposes
# Use cases:
# - Check if a user completed device authentication
# - Debug stuck authentication flows
# - Verify device codes are working correctly
# 
# Expected statuses: pending, completed, expired, or invalid
```

For detailed CLI documentation, run `pixie help` after installation.

</details>

## API Documentation

<details>
<summary><b>Image Generation & Editing</b></summary>

### Generate Image
```bash
curl -X POST https://your-worker.workers.dev/v1/images/generations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "model": "gpt-image-1",
    "prompt": "A serene mountain landscape",
    "size": "1024x1024",
    "quality": "high"
  }'
```

### Edit Image
```bash
curl -X POST https://your-worker.workers.dev/v1/images/edits \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -F image="@original.png" \
  -F mask="@mask.png" \
  -F prompt="Add a sunset to the background"
```

</details>

<details>
<summary><b>Gallery & Images</b></summary>

### Browse Public Gallery
```bash
curl https://your-worker.workers.dev/v1/images
```

### Get Specific Image
```bash
curl https://your-worker.workers.dev/v1/images/{image_id}
```

</details>

<details>
<summary><b>Credits Management</b></summary>

### Check Balance
```bash
curl https://your-worker.workers.dev/v1/credits/balance \
  -H "Authorization: Bearer your-api-key"
```

### Estimate Cost
```bash
curl -X POST https://your-worker.workers.dev/v1/credits/estimate \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "prompt": "Your prompt here",
    "quality": "medium",
    "size": "1024x1024"
  }'
```

### View Credit Packs
```bash
curl https://your-worker.workers.dev/v1/credits/packs
```

### Purchase Credits
```bash
# Crypto payment (currently supported)
curl -X POST https://your-worker.workers.dev/v1/credits/purchase \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "pack_id": "starter",
    "payment_provider": "nowpayments",
    "payment_id": "",
    "payment_currency": "btc"  // btc, eth, doge, or ltc
  }'

# Card payment (via Stripe)
curl -X POST https://your-worker.workers.dev/v1/credits/purchase \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "pack_id": "starter",
    "payment_provider": "stripe",
    "payment_id": ""
  }'
```

### Transaction History
```bash
curl https://your-worker.workers.dev/v1/credits/transactions \
  -H "Authorization: Bearer your-api-key"
```

</details>

<details>
<summary><b>Authentication & System</b></summary>

### Device Authentication

#### Initialize Device Flow
```bash
curl -X POST https://your-worker.workers.dev/v1/auth/device/code \
  -H "Content-Type: application/json" \
  -d '{
    "client_id": "your-client-id"
  }'
```

#### Poll for Token
```bash
curl -X POST https://your-worker.workers.dev/v1/auth/device/token \
  -H "Content-Type: application/json" \
  -d '{
    "device_code": "XXXX-XXXX",
    "client_id": "your-client-id"
  }'
```

#### Check Device Auth Status
```bash
# Check if a device authentication has been completed
# Returns: {status: "pending|completed|expired", message: "..."}
curl https://your-worker.workers.dev/v1/auth/device/{device_code}/status

# Note: This endpoint is primarily for debugging and support purposes.
# The device auth flow normally handles polling automatically.
```

### System Status

#### Health Check
```bash
curl https://your-worker.workers.dev/
```

</details>

<details>
<summary><b>Admin Endpoints</b></summary>

### Adjust User Credits (Admin Only)
```bash
curl -X POST https://your-worker.workers.dev/v1/admin/credits/adjust \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer admin-api-key" \
  -d '{
    "user_id": "user123",
    "amount": 100,
    "reason": "Manual adjustment"
  }'
```

### Credit Statistics (Admin Only)
```bash
curl https://your-worker.workers.dev/v1/admin/credits/stats \
  -H "Authorization: Bearer admin-api-key"
```

</details>

## Development

<details>
<summary><b>Development Setup</b></summary>

Run locally:
```bash
npx wrangler dev
```

Run with live reload:
```bash
cargo watch -s "npx wrangler dev"
```

View logs:
```bash
npx wrangler tail
```

</details>

<details>
<summary><b>Project Structure</b></summary>

```
├── src/
│   ├── lib.rs              # Main worker entry point
│   ├── handlers/           # Request handlers
│   │   ├── images.rs       # Image generation
│   │   ├── gallery.rs      # Public gallery
│   │   ├── usage.rs        # Usage tracking
│   │   ├── r2.rs           # Image serving
│   │   ├── credits.rs      # Credits management
│   │   ├── device_auth.rs  # Device authentication flow
│   │   └── oauth.rs        # OAuth handlers
│   ├── models.rs           # Data models
│   ├── auth.rs             # Authentication
│   ├── credits.rs          # Credits system logic
│   ├── deployment.rs       # Deployment mode configuration
│   ├── storage.rs          # R2 storage
│   └── error.rs            # Error handling
├── cli/                    # CLI application
│   ├── src/
│   │   ├── cli/            # Command definitions
│   │   ├── commands/       # Command handlers
│   │   └── main.rs         # CLI entry point
│   └── Cargo.toml          # CLI dependencies
├── migrations/             # D1 database migrations
├── docs/                   # Documentation
│   ├── pricing.md          # Detailed pricing information
│   ├── SETUP.md            # Setup instructions
│   └── wrangler.toml.example # Example configuration
├── examples/               # Example scripts
└── wrangler.toml          # Worker configuration
```

</details>

## Credits & Billing

The service uses a credit-based pricing system where **1 credit = $0.01 USD**. Credits are deducted based on actual token usage.

<details>
<summary><b>Pricing Details</b></summary>

### Typical Credit Costs

| Quality | Size | Credits | USD Cost |
|---------|------|---------|----------|
| Low | 1024×1024 | 3-5 | $0.03-0.05 |
| Medium | 1024×1024 | 12-15 | $0.12-0.15 |
| High | 1024×1024 | 50-55 | $0.50-0.55 |

### Credit Packs

| Pack | Credits | Price | Bonus |
|------|---------|-------|-------|
| Starter | 150 | $2.99 | - |
| Basic | 500 | $9.99 | 25 (5%) |
| Popular | 1,250 | $24.99 | 114 (10%) |
| Business | 2,500 | $49.99 | 326 (15%) |
| Enterprise | 5,000 | $99.99 | 833 (20%) |

### Payment Methods

- **Credit/Debit Cards**: All major cards via Stripe
  - Available for all credit packs
  - Secure checkout with instant credit delivery
- **Cryptocurrency**: Bitcoin (BTC), Ethereum (ETH), Dogecoin (DOGE), Litecoin (LTC) via NOWPayments
  - *Note: Crypto payments only available for Basic pack ($9.99) and above due to minimum transaction requirements*

For detailed pricing information, see [docs/pricing.md](docs/pricing.md).

</details>

## Documentation

Additional documentation is available in the [docs/](docs/) directory:

- [Pricing Details](docs/pricing.md) - Comprehensive pricing model and credit system
- [Setup Guide](docs/SETUP.md) - Detailed setup and configuration instructions
- [Example Configuration](docs/wrangler.toml.example) - Sample wrangler.toml configuration

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

GPL-3.0
