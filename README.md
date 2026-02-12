# Juxtaposition

Juxtaposition is the Pretendo made Miiverse replacement and successor.

## Philosophy

This project is not meant to be just a simple clone of Miiverse. We aim to evolve it and bring the platform into the modern era.

This means we both want to bring all features originally found in Miiverse into Juxtaposition, but go even further with even more features.

## Limitations

- The web platforms on the 3DS and Wii U are old, thus we need to use old web methodologies like AJAX. 
- The XML API of the Miiverse platform cannot be modified or extended, it needs to stay exactly as the consoles expect it.

# Development Setup

📚 **For detailed development environment setup instructions, see [DEVELOPMENT.md](./DEVELOPMENT.md)**

## Quick Start

Prerequisites:
- Node.js 20 or higher
- Docker (highly recommended)

```bash
# 1. Start Docker services
cd .docker && docker compose up -d && cd ..

# 2. Run miiverse-api
cd apps/miiverse-api && npm install
PN_MIIVERSE_API_USE_PRESETS=docker npm run dev
```

In another terminal:
```bash
# 3. Run juxtaposition-ui
cd apps/juxtaposition-ui && npm install
PN_JUXTAPOSITION_UI_USE_PRESETS=docker npm run dev
```

The [DEVELOPMENT.md](./DEVELOPMENT.md) guide includes:
- Complete prerequisites and setup steps
- Configuration options and environment variables
- Database initialization
- Testing with consoles/emulators
- Troubleshooting common issues

# Translation

If you'd like to help localize Pretendo Network, you can contribute to the translations on our project on [Weblate](https://hosted.weblate.org/engage/pretendonetwork/).
