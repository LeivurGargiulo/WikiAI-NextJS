# WikiAI - AI-Powered Personal Wiki

A modern, feature-rich personal wiki platform powered by AI. Create, edit, and manage articles with AI assistance for text editing and image generation. Perfect for organizing knowledge, creating documentation, or building your own personal encyclopedia.

## Features

### Core Features
- **Article Management**: Create, edit, and organize articles with full Markdown support
- **AI-Powered Text Editing**: Enhance your content with AI assistance
  - Rewrite text with custom instructions
  - Summarize long articles
  - Expand content with more details
  - Translate between languages
- **AI Image Generation**: Generate images for articles using ComfyUI or OpenRouter
- **Advanced Search**: Multiple search modes for finding content
  - **Full-Text Search**: PostgreSQL full-text search with ranking
  - **Vector Search**: Semantic similarity search using embeddings
  - **Hybrid Search**: Combines full-text and vector search for best results
- **Featured & Trending Articles**: Highlight important content
- **Categories & Statistics**: Organize articles with custom categories and infobox statistics
- **Progressive Web App (PWA)**: Install as a native app with offline support
  - Service worker for offline functionality
  - Install prompt for easy app installation
  - Offline indicator
  - Caching strategies for optimal performance

### Customization
- **Multiple AI Providers**: Support for Ollama (local), OpenAI, Claude, and OpenRouter
- **Customizable Prompts**: Configure AI prompt templates for different operations
- **Color Scheme Customization**: Personalize the appearance with custom color schemes
- **Wiki Branding**: Customize wiki name, description, and topic
- **Wiki Context**: Provide background information to improve AI-generated content
- **Multi-language Support**: English and Spanish (with extensible i18n system)
- **Custom Categories**: Define your own article categories
- **Custom Statistics Fields**: Create custom infobox fields for articles
- **Date Format Customization**: Configure date display format (MM/DD/YYYY, DD/MM/YYYY, YYYY-MM-DD, etc.)
- **Reference Year**: Set a custom reference year for age calculations in statistics

### Data Management
- **Export/Import**: Backup and restore your entire wiki
- **Static Website Export**: Export your wiki as a static website for hosting anywhere
- **Image Management**: Upload and manage images for articles

## Prerequisites

- **Docker Engine** 20.10+ and **Docker Compose** 2.0+
- **AI Provider** (choose one or more):
  - **Ollama** (included in Docker, recommended for local use)
  - **OpenAI API Key**: Get from [OpenAI Platform](https://platform.openai.com)
  - **Claude API Key**: Get from [Anthropic Console](https://console.anthropic.com)
  - **OpenRouter API Key**: Get from [OpenRouter](https://openrouter.ai)
- **Image Generation** (optional):
  - **ComfyUI** (can be enabled in docker-compose, requires GPU)
  - **OpenRouter** (cloud): Requires OpenRouter API key

## Installation

### Quick Start with Docker (Recommended)

WikiAI runs entirely in Docker, including the database, application, and optional AI services.

1. **Clone the repository**:
```bash
git clone https://github.com/LeivurGargiulo/WikiAI-NextJS.git
cd WikiAI-NextJS
```

2. **Create data directories** (for persistence):
```bash
mkdir -p public/data/images
```

3. **Configure environment variables** (optional):
Create a `.env` file in the root directory for API keys:
```env
# Optional: Set API keys for cloud AI providers
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
OPENROUTER_API_KEY=sk-or-...

# Optional: Supabase API keys (if using full Supabase stack)
SUPABASE_URL=http://localhost:54321
SUPABASE_ANON_KEY=your-anon-key-here
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here
```

4. **Start all services**:
```bash
docker-compose up -d
```

This will start:
- **WikiAI application** on port 3000
- **PostgreSQL database** with pgvector on port 54330
- **Ollama** (local AI) on port 11435

5. **Pull Ollama models** (if using Ollama):
```bash
# Pull text generation models (choose one or more)
docker exec -it wikiai-ollama ollama pull llama3.2
# or
docker exec -it wikiai-ollama ollama pull mistral
# or
docker exec -it wikiai-ollama ollama pull codellama

# Pull image generation model (optional)
docker exec -it wikiai-ollama ollama pull flux
```

6. **Access the application**:
- **WikiAI**: [http://localhost:3000](http://localhost:3000)
- **Ollama API**: http://localhost:11435

The database will be automatically initialized with migrations on first start.

### Manual Installation (Advanced)

If you prefer to run without Docker, see the [Manual Installation Guide](./docs/manual-installation.md) for detailed instructions.

## Production Deployment

> **⚠️ Important**: Before updating WikiAI, it is strongly advised to export your wiki data using the export feature in Settings → Data Management. This ensures you have a backup of all your articles, images, and settings in case anything goes wrong during the update process.

### Docker Production Deployment

WikiAI is designed to run in Docker for production. The Docker setup includes all necessary services.

1. **Configure environment variables**:
Create a `.env` file with your production settings:
```env
# Database (automatically configured in Docker)
DATABASE_URL=postgresql://postgres:postgres@supabase-db:5432/postgres

# Application URL
NEXT_PUBLIC_APP_URL=https://your-domain.com

# AI Provider API Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
OPENROUTER_API_KEY=sk-or-...
```

2. **Start services**:
```bash
docker-compose up -d
```

3. **Verify services are running**:
```bash
docker-compose ps
docker-compose logs -f wikiai
```

### Production Considerations

- **Data Persistence**: Ensure `public/data/images/` directory is backed up regularly
- **Database Backups**: The PostgreSQL data is stored in a Docker volume (`supabase-db-data`). Back it up regularly:
  ```bash
  docker exec wikiai-supabase-db pg_dump -U postgres postgres > backup.sql
  ```
- **Database Migrations**: Migrations run automatically on container start. For manual migration:
  ```bash
  docker exec -it wikiai npx prisma migrate deploy
  ```
- **AI Services**: Configure API keys in `.env` or docker-compose.yml
- **Image Storage**: Images are persisted in `./public/data/images/` - ensure this directory is backed up
- **Static Export**: For static hosting, use the built-in static export feature in Settings → Data Management
- **Resource Limits**: Adjust CPU/memory limits in docker-compose.yml for production workloads
- **Security**: Use secrets management for API keys in production (Docker secrets, environment variables, etc.)

### Deployment Platforms

WikiAI can be deployed to various platforms:

- **Docker Hosting**: Any platform supporting Docker Compose (VPS, cloud providers, etc.)
- **Docker Swarm/Kubernetes**: For orchestrated deployments
- **Static Hosting**: Export as static website and host on GitHub Pages, Netlify, etc.

## Docker Services

The `docker-compose.yml` includes all necessary services:

- **wikiai**: Main application container (Next.js)
- **supabase-db**: PostgreSQL database with pgvector extension
- **ollama**: Local AI model server (optional, for local AI)
- **comfyui**: Image generation service (commented out by default, requires GPU)

### Docker Commands

**Start all services**:
```bash
docker-compose up -d
```

**Stop services**:
```bash
docker-compose down
```

**View logs**:
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f wikiai
```

**Rebuild after changes**:
```bash
docker-compose up -d --build
```

**Access container shell**:
```bash
docker exec -it wikiai sh
```

**Restart a specific service**:
```bash
docker-compose restart wikiai
```

### Volume Mounts

Data is persisted in the following locations:

- `./public/data/images/` - Uploaded images (mounted from host)
- `supabase-db-data` (Docker volume) - PostgreSQL database data
- `ollama-data` (Docker volume) - Ollama models

### Environment Variables

Configure environment variables in `docker-compose.yml` or via `.env` file:

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection (internal) | `postgresql://postgres:postgres@supabase-db:5432/postgres` |
| `NEXT_PUBLIC_APP_URL` | Application URL | `http://localhost:3000` |
| `OLLAMA_BASE_URL` | Ollama service URL (internal) | `http://ollama:11434` |
| `OPENAI_API_KEY` | OpenAI API key | - |
| `ANTHROPIC_API_KEY` | Claude API key | - |
| `OPENROUTER_API_KEY` | OpenRouter API key | - |
| `SUPABASE_URL` | Supabase API URL | `http://localhost:54321` |
| `SUPABASE_ANON_KEY` | Supabase anonymous key | - |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key | - |

### Docker Compose Override

For local development customization, copy `docker-compose.override.yml.example` to `docker-compose.override.yml` and modify as needed. This file is automatically loaded by docker-compose and allows you to override settings without modifying the main docker-compose.yml file.

## Docker Image Deployment

WikiAI Docker images are automatically built and published to GitHub Container Registry (ghcr.io) when you create a new release tag.

### Automated Image Builds

When you push a tag to the repository (e.g., `v1.0.0`, `v1.2.3`), GitHub Actions automatically:
- Builds the Docker image using the Dockerfile
- Pushes it to `ghcr.io/leivurgargiulo/wikiai`
- Tags the image with the version tag and semantic version tags (e.g., `v1.0.0`, `1.0.0`, `1.0`, `1`, `latest`)

### Pulling and Running the Published Image

1. **Authenticate with GitHub Container Registry** (if the repository is private):
   ```bash
   echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
   ```
   Or use a Personal Access Token with `read:packages` permission.

2. **Pull the image**:
   ```bash
   docker pull ghcr.io/leivurgargiulo/wikiai:latest
   ```
   Or pull a specific version:
   ```bash
   docker pull ghcr.io/leivurgargiulo/wikiai:v1.0.0
   ```

3. **Run the container**:
   ```bash
   docker run -d \
     --name wikiai \
     -p 3000:3000 \
     -v $(pwd)/data/prisma:/app/prisma \
     -v $(pwd)/public/data/images:/app/public/images \
     -e DATABASE_URL="file:./prisma/dev.db" \
     -e NEXT_PUBLIC_APP_URL="http://localhost:3000" \
     ghcr.io/leivurgargiulo/wikiai:latest
   ```

### Available Image Tags

- `latest` - Most recent release
- `v1.0.0` - Specific version tag
- `1.0.0` - Semantic version (without 'v' prefix)
- `1.0` - Major.minor version
- `1` - Major version only

### View Published Images

You can view all published images on the [GitHub Container Registry](https://github.com/LeivurGargiulo/WikiAI-NextJS/pkgs/container/wikiai).

### Using in Docker Compose

You can also use the published image in your `docker-compose.yml`:

```yaml
services:
  wikiai:
    image: ghcr.io/leivurgargiulo/wikiai:latest
    ports:
      - "3000:3000"
    volumes:
      - ./data/prisma:/app/prisma
      - ./public/data/images:/app/public/images
    environment:
      - DATABASE_URL=file:./prisma/dev.db
      - NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Configuration

### Environment Variables

When running in Docker, environment variables can be set in:
1. `.env` file (recommended for secrets)
2. `docker-compose.yml` (for service configuration)
3. `docker-compose.override.yml` (for local overrides)

| Variable | Description | Required | Default (Docker) |
|----------|-------------|----------|------------------|
| `DATABASE_URL` | PostgreSQL connection string | Yes | `postgresql://postgres:postgres@supabase-db:5432/postgres` |
| `SUPABASE_URL` | Supabase API URL (if using full stack) | No | `http://localhost:54321` |
| `SUPABASE_ANON_KEY` | Supabase anonymous key | No | - |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key | No | - |
| `OLLAMA_BASE_URL` | Ollama API base URL (internal) | No* | `http://ollama:11434` |
| `OPENAI_API_KEY` | OpenAI API key | No* | - |
| `ANTHROPIC_API_KEY` | Claude API key | No* | - |
| `OPENROUTER_API_KEY` | OpenRouter API key | No* | - |
| `NEXT_PUBLIC_APP_URL` | Application base URL | No | `http://localhost:3000` |

*Required only if using the respective AI provider. You can also configure API keys in the Settings page.

**Note**: When accessing services from outside Docker (e.g., from your host machine), use:
- Database: `postgresql://postgres:postgres@localhost:54330/postgres` (port 54330)
- Ollama: `http://localhost:11435` (port 11435)

### Settings Configuration

Most settings can be configured through the web interface at `/settings`:

- **AI Provider**: Choose between Ollama, OpenAI, Claude, or OpenRouter
- **Text Model**: Select the model for text generation
- **Image Provider**: Choose between ComfyUI (local) or OpenRouter (cloud)
- **Image Model**: Select the model for image generation
- **Language**: Choose interface language (English/Spanish)
- **Prompts**: Customize AI prompt templates
- **Color Scheme**: Customize the appearance
- **Wiki Customization**: Set wiki name, description, topic, and context
- **Date Format**: Configure date display format (MM/DD/YYYY, DD/MM/YYYY, YYYY-MM-DD, etc.)
- **Reference Year**: Set a custom reference year for age calculations

### Search API

WikiAI provides three search endpoints for different use cases:

#### Full-Text Search
```bash
GET /api/search/fulltext?query=search+term&limit=10
```
- Uses PostgreSQL full-text search with ranking
- Fast keyword-based search
- Returns results ranked by relevance

#### Vector Search
```bash
GET /api/search/vector?query=search+term&limit=10&threshold=0.7
```
- Semantic similarity search using embeddings
- Finds articles with similar meaning, not just keywords
- Requires embeddings to be generated (automatic for new articles)

#### Hybrid Search
```bash
GET /api/search/hybrid?query=search+term&limit=10&vectorWeight=0.6&fulltextWeight=0.4
```
- Combines full-text and vector search
- Best of both worlds: keyword matching + semantic understanding
- Configurable weights for each search type

## Usage

For detailed usage instructions, see [USAGE.md](./USAGE.md).

### Quick Start

1. **Create an article**: Click "New Article" in the navigation, enter a title and content
2. **Edit with AI**: Open any article, select text, and use AI editing features
3. **Generate images**: Use the image generation feature in the article editor
4. **Customize**: Visit Settings to configure AI providers, appearance, and more

## Technology Stack

- **Framework**: Next.js 15 (App Router)
- **UI Library**: React 19
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Database**: PostgreSQL with Supabase (local) and Prisma ORM
- **Vector Search**: pgvector extension for semantic similarity search with HNSW indexing
- **Full-Text Search**: PostgreSQL built-in full-text search with ts_rank ranking
- **Embeddings**: Automatic embedding generation for semantic search
- **State Management**: React Query (TanStack Query)
- **AI Integration**: 
  - Ollama (local AI)
  - OpenAI API
  - Anthropic Claude API
  - OpenRouter API
- **Image Generation**: ComfyUI, OpenRouter
- **Markdown**: react-markdown with remark-gfm
- **PWA**: Service Worker, Web App Manifest, Offline support
- **Testing**: Playwright for E2E, component, and API testing

## Project Structure

```
WikiAI-NextJS/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── ai/           # AI-related endpoints
│   │   ├── articles/     # Article CRUD operations
│   │   ├── export/       # Export functionality
│   │   ├── import/       # Import functionality
│   │   ├── search/       # Search endpoints (fulltext, vector, hybrid)
│   │   └── wiki-settings/ # Wiki settings API
│   ├── create/           # Create article page
│   ├── settings/         # Settings page
│   ├── wiki/             # Article pages
│   │   └── [slug]/      # Dynamic article routes
│   ├── layout.tsx        # Root layout
│   ├── page.tsx          # Home page
│   └── globals.css       # Global styles
├── components/            # React components
│   ├── article/         # Article-related components
│   ├── home/            # Home page components
│   ├── layout/          # Layout components
│   ├── pwa/             # PWA components (install prompt, offline indicator)
│   └── ui/              # UI components
├── hooks/                # Custom React hooks
├── lib/                  # Utilities and libraries
│   ├── providers/       # AI provider implementations
│   ├── pwa/             # PWA service worker code
│   ├── ai-provider.ts   # AI provider interface
│   ├── embeddings.ts    # Embedding generation utilities
│   ├── date-utils.ts    # Date formatting utilities
│   ├── prisma.ts        # Prisma client
│   ├── settings.ts      # Settings management
│   └── ...
├── locales/              # Internationalization files
│   ├── en.json          # English translations
│   └── es.json          # Spanish translations
├── prisma/               # Database schema and migrations
│   ├── schema.prisma    # Prisma schema
│   └── migrations/      # Database migrations
├── supabase/            # Supabase configuration
│   └── migrations/      # Supabase SQL migrations (pgvector, RLS)
├── scripts/             # Utility scripts
│   ├── migrate-sqlite-to-postgres.ts  # Data migration script
│   └── generate-embeddings-for-existing.ts  # Embedding generation script
├── tests/               # Test suite
│   ├── e2e/            # End-to-end tests
│   ├── component/      # Component tests
│   ├── api/            # API endpoint tests
│   └── helpers/        # Test utilities and helpers
├── docs/                # Documentation
│   ├── migration-guide.md  # SQLite to PostgreSQL migration guide
│   └── novel-ai-integration.md  # Integration guide for Novel AI project
├── public/               # Static assets
│   └── images/          # Uploaded images
└── types/                # TypeScript type definitions
```

## Development

### Running in Development Mode

For development, you can either:

1. **Use Docker with volume mounts** (recommended):
   - Copy `docker-compose.override.yml.example` to `docker-compose.override.yml`
   - Uncomment the volume mounts for live code reloading
   - Run `docker-compose up -d`

2. **Run locally** (requires Node.js installed):
   ```bash
   # Install dependencies
   npm install
   
   # Start database
   docker-compose up -d supabase-db
   
   # Set environment variables
   export DATABASE_URL="postgresql://postgres:postgres@localhost:54330/postgres"
   
   # Run development server
   npm run dev
   ```

### Available Scripts (Local Development)

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint
- `npm run db:push` - Push Prisma schema to database
- `npm run db:studio` - Open Prisma Studio (database GUI)
- `npm run db:generate` - Generate Prisma client
- `npm run test` - Run all tests
- `npm run test:e2e` - Run end-to-end tests
- `npm run test:component` - Run component tests
- `npm run test:api` - Run API tests
- `npm run test:ui` - Run tests with Playwright UI

### Database Management (Docker)

```bash
# Generate Prisma client after schema changes
docker exec -it wikiai npx prisma generate

# Push schema changes to database (development only)
docker exec -it wikiai npx prisma db push

# Open Prisma Studio to view/edit data
docker exec -it wikiai npx prisma studio
# Then access at http://localhost:5555 (if port is exposed)

# Or run Prisma Studio locally (requires DATABASE_URL pointing to localhost:54330)
npx prisma studio
```

### Database Migrations

When you update the Prisma schema, you need to migrate your database to apply the changes. The migration process differs between development and production environments.

#### Development Migrations

For development, you can use `prisma db push` for quick iterations, or use migrations for better tracking:

**Using Migrations (Recommended)**:
```bash
# Create a new migration after schema changes
npx prisma migrate dev --name describe_your_changes

# This will:
# - Create a migration file in prisma/migrations/
# - Apply the migration to your development database
# - Regenerate the Prisma client
```

**Using db push (Quick development)**:
```bash
# For rapid development iterations (development only)
npm run db:push
```

#### Production Migrations (Docker)

**⚠️ IMPORTANT: Always backup your production database before migrating!**

> **Note**: Before updating WikiAI, export your wiki using the export feature in Settings → Data Management. This creates a complete backup of all your articles, images, and settings that can be easily restored if needed.

1. **Backup your production database**:
   ```bash
   # Backup PostgreSQL database
   docker exec wikiai-supabase-db pg_dump -U postgres postgres > backup_$(date +%Y%m%d_%H%M%S).sql
   
   # Or backup the Docker volume
   docker run --rm -v wikiai_supabase-db-data:/data -v $(pwd):/backup alpine tar czf /backup/db_backup_$(date +%Y%m%d_%H%M%S).tar.gz /data
   ```

2. **Test the migration locally with production data**:
   ```bash
   # Restore production backup to local database
   docker exec -i wikiai-supabase-db psql -U postgres postgres < backup.sql
   
   # Test the migration
   docker exec -it wikiai npx prisma migrate deploy
   ```

3. **Deploy migration to production**:
   ```bash
   # Run migration inside container
   docker exec -it wikiai npx prisma migrate deploy
   
   # Or restart the container (migrations run automatically on start)
   docker-compose restart wikiai
   ```

4. **Verify migration**:
   ```bash
   # Check migration status
   docker exec -it wikiai npx prisma migrate status
   ```

#### Migration Best Practices

- **Always backup before migrating**: Create a backup of your production database before applying any migrations
- **Test migrations first**: Test migrations on a copy of production data in a development environment
- **Use migrations in production**: Use `prisma migrate deploy` in production, never use `db push --accept-data-loss`
- **Review migration SQL**: Check the generated SQL in `prisma/migrations/` before applying to production
- **Version control migrations**: Commit migration files to git so they can be applied consistently across environments

#### Initializing Migrations (First Time Setup)

If you're switching from `db push` to migrations:

```bash
# Create initial migration from current schema
npx prisma migrate dev --name init

# This creates the migrations folder and initial migration
```

#### Troubleshooting

**Migration conflicts**:
- If migrations are out of sync, you may need to reset (development only):
  ```bash
  npx prisma migrate reset  # ⚠️ This deletes all data - development only!
  ```

**Manual migration**:
- For complex changes, you can create a custom migration:
  ```bash
  npx prisma migrate dev --create-only --name custom_migration
  # Edit the generated SQL file in prisma/migrations/
  npx prisma migrate dev
  ```

**Check migration status**:
```bash
# See which migrations have been applied
npx prisma migrate status
```

## Testing

WikiAI includes a comprehensive test suite using Playwright:

### Running Tests

```bash
# Run all tests
npm run test

# Run specific test suites
npm run test:e2e          # End-to-end tests
npm run test:component    # Component tests
npm run test:api          # API endpoint tests

# Run tests with UI
npm run test:ui
```

### Test Structure

- **E2E Tests** (`tests/e2e/`): Full user workflows (create article, edit, search, etc.)
- **Component Tests** (`tests/component/`): Individual React component testing
- **API Tests** (`tests/api/`): API endpoint testing
- **Test Helpers** (`tests/helpers/`): Shared utilities and test data

### Test Coverage

The test suite covers:
- Article creation and editing
- AI features (text editing, image generation)
- Search functionality (full-text, vector, hybrid)
- Settings management
- Export/import functionality
- Wiki customization

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Make your changes and ensure tests pass (`npm run test`)
4. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
5. Push to the branch (`git push origin feature/AmazingFeature`)
6. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.

## Acknowledgments

- Built with [Next.js](https://nextjs.org/)
- AI capabilities powered by [Ollama](https://ollama.ai), [OpenAI](https://openai.com), [Anthropic](https://anthropic.com), and [OpenRouter](https://openrouter.ai)
- Image generation via [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- Vector search powered by [pgvector](https://github.com/pgvector/pgvector)
- Testing with [Playwright](https://playwright.dev)

## Support

For issues, questions, or contributions, please open an issue on [GitHub](https://github.com/LeivurGargiulo/WikiAI-NextJS/issues).
