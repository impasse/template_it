# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a SvelteKit project called "template-it" - a Liquid template renderer tool built with Svelte 5 and Vite. The application allows users to create, test, and debug Liquid templates by entering templates and JSON data, then viewing the rendered output in real-time.

## Architecture

### Technology Stack
- **Frontend**: Svelte 5 with SvelteKit framework
- **Build Tool**: Vite
- **Styling**: TailwindCSS v4
- **Template Engine**: Liquid.js (for Liquid template parsing and rendering)
- **Type Checking**: TypeScript with svelte-check
- **Deployment**: Cloudflare adapter for deployment to Cloudflare Pages

### Key Components
- **Main Page** (`src/routes/+page.svelte`): The core liquid template renderer interface with three main sections:
  - Template input textarea
  - JSON data input and rendered output side-by-side
  - History sidebar for saving/loading template configurations
- **Layout** (`src/routes/+layout.svelte`): Root layout component with favicon
- **Styling**: Uses TailwindCSS classes throughout

### State Management
- Uses Svelte 5's `$state` for reactive state management
- Local storage integration for template history persistence
- Real-time template rendering with Liquid.js

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Deploy to Cloudflare
npm run deploy

# Type checking
npm run check

# Type checking in watch mode
npm run check:watch
```

## Configuration

### Build Configuration
- Uses Cloudflare adapter (`@sveltejs/adapter-cloudflare`) for deployment
- Vite preprocessor for Svelte components
- TailwindCSS integration via `@tailwindcss/vite` plugin
- DevTools JSON plugin for development debugging

### TypeScript Configuration
- Extends SvelteKit's TypeScript configuration
- Strict mode enabled
- Module resolution set to bundler

## Key Features

### Template History
- Saves up to 10 template configurations locally
- Tracks template content, JSON data, and timestamps
- Provides navigation through history with dirty state detection
- Individual item deletion and bulk clear functionality

### Real-time Rendering
- Uses Liquid.js to parse and render templates
- Async rendering with proper loading/error states
- JSON input validation with automatic parsing

### Responsive Design
- Mobile-friendly layout with responsive grid system
- Three-column layout on large screens, two-column on medium
- Properly sized text areas and scrollable content areas

## File Structure

```
src/
├── app.css                 # TailwindCSS styles
├── app.d.ts                # TypeScript app declarations
├── lib/
│   └── index.ts           # Library utilities (currently empty)
└── routes/
    ├── +layout.svelte      # Root layout component
    └── +page.svelte        # Main template renderer interface
```

## Dependencies

### Core Dependencies
- `svelte`: Svelte framework
- `@sveltejs/kit`: SvelteKit framework
- `vite`: Build tool and development server
- `liquidjs`: Liquid template engine

### Development Dependencies
- `@sveltejs/adapter-cloudflare`: Cloudflare adapter
- `@tailwindcss/vite`: TailwindCSS Vite plugin
- `svelte-check`: TypeScript type checking
- `typescript`: TypeScript compiler

## Important Notes

- The project uses Svelte 5's new `$state` syntax for reactive state
- Template history is stored in localStorage with a maximum of 10 items
- Real-time template rendering uses Liquid.js's `parseAndRender` method
- The application includes proper error handling for invalid JSON and template syntax
- Cloudflare deployment is configured via the `deploy` script