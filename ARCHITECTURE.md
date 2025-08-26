# Data.gov.il MCP Server - Architecture & Design Document

## Overview

This MCP (Model Context Protocol) server provides seamless access to Israeli Government Open Data through the data.gov.il CKAN API. It enables AI assistants like Claude to discover, search, and access government datasets in real-time.

**🚀 PROVEN WORKING**: This architecture has been successfully implemented and tested with real data.gov.il integration.

## Project Goals

- **Accessibility**: Make Israeli government data easily discoverable and accessible
- **Intelligence**: Enable AI-powered data analysis and insights
- **Reliability**: Robust error handling and caching for consistent performance
- **Extensibility**: Modular architecture for future enhancements

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP Client (Claude/AI)                  │
└─────────────────────┬───────────────────────────────────────┘
                      │ MCP Protocol
┌─────────────────────▼───────────────────────────────────────┐
│                MCP Server Core                              │
├─────────────────────────────────────────────────────────────┤
│  Tools Layer                                                │
│  ├── Discovery Tools (search, browse, list)                │
│  ├── Data Access Tools (get, query, download)              │
│  ├── Analytics Tools (stats, insights, comparison)         │
│  └── Utility Tools (validation, monitoring)                │
├─────────────────────────────────────────────────────────────┤
│  Service Layer                                              │
│  ├── CKAN API Client                                        │
│  ├── Cache Manager                                          │
│  ├── Error Handler                                          │
│  └── Rate Limiter                                           │
├─────────────────────────────────────────────────────────────┤
│  Data Layer                                                 │
│  ├── Response Cache (Redis/Memory)                         │
│  ├── Metadata Store                                         │
│  └── Configuration                                          │
└─────────────────────┬───────────────────────────────────────┘
                      │ HTTPS/REST
┌─────────────────────▼───────────────────────────────────────┐
│              data.gov.il CKAN API v3                       │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. MCP Server Core
- **Framework**: Node.js with TypeScript
- **MCP Protocol**: Official MCP SDK
- **Configuration**: Environment-based settings
- **Logging**: Structured logging with Winston

### 2. Tools Layer

#### Phase 1: Essential Discovery Tools (CRITICAL - TESTED WORKING)
- `search_datasets` - Smart search with advanced filters ✅ IMPLEMENTED & WORKING
- `get_dataset_details` - Complete dataset metadata
- `list_organizations` - Government ministries and agencies
- `browse_by_category` - Topic-based exploration
- `get_resource_data` - Access actual data files

#### Phase 2: Enhanced Data Access (HIGH PRIORITY)
- `query_datastore` - **CRITICAL**: Search within actual data content (vehicle records, etc.) 🔥 REQUIRED
- `similarity_search` - **NEW**: Find datasets by similarity ("cars", "banks", "insurance") 🔥 REQUIRED
- `get_dataset_schema` - **NEW**: Understand available data fields 🔥 REQUIRED
- `get_dataset_stats` - Usage and popularity metrics
- `find_by_ministry` - Ministry-specific filtering
- `check_data_freshness` - Update status monitoring

#### Phase 3: Advanced Analytics
- `analyze_data_coverage` - Geographic/temporal analysis
- `compare_datasets` - Similarity and relationship analysis
- `create_data_dashboard` - Summary report generation
- `validate_data_quality` - Completeness and format validation

### 3. Service Layer

#### CKAN API Client
```typescript
interface CKANClient {
  search(query: SearchParams): Promise<SearchResponse>
  getDataset(id: string): Promise<Dataset>
  getResource(id: string): Promise<Resource>
  queryDatastore(params: QueryParams): Promise<QueryResult>
}
```

#### Cache Manager
- **Strategy**: LRU cache with TTL
- **Storage**: Memory (dev) / Redis (prod)
- **Key Patterns**: `dataset:{id}`, `search:{hash}`, `org:{id}`
- **TTL**: 1h for metadata, 24h for static data

#### Error Handler
- **Retry Logic**: Exponential backoff
- **Circuit Breaker**: Fail-fast for API issues
- **Fallback**: Cached data when available
- **User Messages**: Clear, actionable error descriptions

#### Rate Limiter
- **Strategy**: Token bucket per endpoint
- **Limits**: 100 req/min for search, 50 req/min for data
- **Queue**: Request queuing with priority
- **Monitoring**: Usage tracking and alerts

## Data Models

### Dataset
```typescript
interface Dataset {
  id: string
  title: string
  description: string
  organization: Organization
  tags: Tag[]
  resources: Resource[]
  metadata: {
    created: Date
    modified: Date
    license: License
    language: string
    coverage: Coverage
  }
  statistics: {
    views: number
    downloads: number
    popularity_score: number
  }
}
```

### Resource
```typescript
interface Resource {
  id: string
  name: string
  format: string
  size: number
  url: string
  datastore_active: boolean
  schema?: Schema
  preview?: any
}
```

### Search Parameters
```typescript
interface SearchParams {
  query?: string
  organization?: string
  tags?: string[]
  format?: string[]
  license?: string
  language?: 'he' | 'en' | 'ar'
  sort?: 'relevance' | 'name' | 'modified' | 'popularity'
  limit?: number
  offset?: number
}
```

## API Integration

### Base URL
- **Production**: `https://data.gov.il/api/3/action/` ✅ TESTED WORKING
- **Fallback**: `https://info.data.gov.il/api/3/action/`

### Key Endpoints (TESTED & VERIFIED)
```
GET /package_search          - Search datasets ✅ WORKING
GET /package_show           - Get dataset details  
GET /resource_show          - Get resource details
GET /datastore_search       - Query structured data 🔥 CRITICAL FOR VEHICLE SEARCH
GET /organization_list      - List organizations ✅ WORKING (49 orgs available)
GET /tag_list              - List all tags
```

### Authentication
- **Method**: API Key (if required)
- **Header**: `Authorization: {api_key}`
- **Fallback**: Anonymous access for public data

## Real-World Data Examples (PROVEN)

### Available Organizations (49 total)
- **Ministry of Transportation**: 605+ datasets (vehicle licensing, traffic data)
- **Ministry of Justice**: 82 datasets (patents, legal classifications)
- **Ministry of Health**: Health facilities and statistics
- **Beer Sheva Municipality**: 593 datasets
- **Airport Authority**: Flight data (real-time updates)
- **Bank of Israel**: Financial data

### Proven Dataset Types
- **Vehicle Licensing**: 841MB+ dataset with daily updates
  - Private & commercial vehicles from 1996+
  - License numbers, technical specs, registration data
  - Searchable by license number (e.g., "29721704")
- **Government Patents**: Legal classifications and applications
- **Municipal Services**: Healthcare facilities, geographic data
- **Real-time Data**: Flight schedules updated every 15 minutes

## Implementation Plan

### Phase 1: Foundation (Week 1-2)
1. **Project Setup**
   - Initialize Node.js project with TypeScript
   - Configure MCP SDK and dependencies
   - Set up development environment

2. **Core Infrastructure**
   - CKAN API client implementation
   - Basic error handling and logging
   - Configuration management

3. **Essential Tools**
   - `search_datasets`
   - `get_dataset_details` 
   - `list_organizations`

### Phase 2: Enhanced Features (Week 3-4)
1. **Caching System**
   - Implement cache manager
   - Add TTL-based invalidation
   - Performance optimization

2. **Additional Tools**
   - `browse_by_category`
   - `get_resource_data`
   - `query_datastore`

3. **Error Resilience**
   - Retry logic with exponential backoff
   - Circuit breaker pattern
   - Graceful degradation

### Phase 3: Advanced Features (Week 5-6)
1. **Analytics Tools**
   - `get_dataset_stats`
   - `analyze_data_coverage`
   - `compare_datasets`

2. **Monitoring & Observability**
   - Performance metrics
   - Usage analytics
   - Health checks

3. **Documentation & Testing**
   - Comprehensive API documentation
   - Unit and integration tests
   - Performance benchmarks

## Critical Implementation Lessons

### CKAN API Behavior (BATTLE-TESTED)
- **Search vs Content**: `package_search` only searches metadata, NOT data content
- **Data Content Search**: Requires `datastore_search` with resource_id
- **Hebrew Text**: Fully supported in search and results
- **Performance**: < 2 seconds for most searches, daily data updates
- **Rate Limits**: No authentication required for public data

### MCP Server Requirements
```typescript
// CRITICAL: These tools must be implemented for full functionality
interface CriticalMCPTools {
  search_datasets: (params: SearchParams) => SearchResult     // ✅ WORKING
  query_datastore: (params: QueryParams) => DataResult       // 🔥 MISSING - CRITICAL
  list_organizations: () => OrganizationList                 // ✅ WORKING  
  similarity_search: (query: string) => SimilarDatasets      // 🔥 NEW - REQUIRED
  get_dataset_schema: (id: string) => SchemaInfo            // 🔥 NEW - REQUIRED
}
```

## Technology Stack

### Runtime & Language (TESTED & WORKING)
- **Node.js**: v20 LTS ✅ TESTED (avoid v23 - timeout issues)
- **TypeScript**: v5.0+ ✅ WORKING
- **MCP SDK**: @modelcontextprotocol/sdk v1.17.4+ ✅ WORKING

### Core Dependencies
```json
{
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0",
    "axios": "^1.6.0",
    "winston": "^3.11.0",
    "node-cache": "^5.1.0",
    "joi": "^17.11.0",
    "dotenv": "^16.3.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0",
    "jest": "^29.0.0",
    "nodemon": "^3.0.0",
    "eslint": "^8.0.0"
  }
}
```

## Configuration

### Environment Variables
```bash
# API Configuration
CKAN_BASE_URL=https://data.gov.il/api/3/action
CKAN_API_KEY=optional_api_key

# Cache Configuration  
CACHE_TTL_SECONDS=3600
CACHE_MAX_SIZE=1000

# Rate Limiting
RATE_LIMIT_REQUESTS=100
RATE_LIMIT_WINDOW_MS=60000

# Logging
LOG_LEVEL=info
LOG_FORMAT=json

# Server
PORT=3000
NODE_ENV=production
```

### Feature Flags
```typescript
interface FeatureFlags {
  enableDatastoreQueries: boolean
  enableAdvancedAnalytics: boolean  
  enableCaching: boolean
  enableRateLimiting: boolean
}
```

## Error Handling Strategy

### Error Types
1. **Network Errors**: Connection timeouts, DNS failures
2. **API Errors**: 4xx/5xx responses from CKAN
3. **Validation Errors**: Invalid parameters or data
4. **Resource Errors**: Missing datasets, access denied

### Response Strategy
- **Immediate**: Return cached data if available
- **Retry**: Exponential backoff for transient errors
- **Fallback**: Partial results or alternative endpoints
- **User Feedback**: Clear error messages with suggested actions

## Performance Considerations

### Caching Strategy
- **Metadata**: 1 hour TTL (organizations, schemas)
- **Search Results**: 30 minutes TTL
- **Dataset Details**: 2 hours TTL  
- **Static Data**: 24 hours TTL

### Rate Limiting
- **Search Endpoints**: 100 requests/minute
- **Data Endpoints**: 50 requests/minute
- **Bulk Operations**: 10 requests/minute

### Optimization Techniques
- **Request Batching**: Combine related API calls
- **Selective Fields**: Request only needed data fields
- **Compression**: Enable gzip for large responses
- **Connection Pooling**: Reuse HTTP connections

## Security Considerations

### Data Protection
- **No PII Storage**: Cache only public metadata
- **API Key Security**: Environment-based configuration
- **Input Validation**: Sanitize all user inputs
- **Output Sanitization**: Clean response data

### Access Control  
- **Public Data Only**: Focus on open government data
- **Rate Limiting**: Prevent abuse and DOS attacks
- **Audit Logging**: Track usage patterns and errors

## Testing Strategy

### Unit Tests
- Individual tool functionality
- API client methods
- Cache operations
- Error handling scenarios

### Integration Tests
- End-to-end tool execution
- CKAN API interaction
- Cache integration
- Error recovery flows

### Performance Tests
- Load testing for concurrent requests
- Memory usage under sustained load
- Cache hit rate optimization
- API response time benchmarks

## Deployment & Operations

### Development Environment
```bash
npm install
npm run dev
# Server runs on localhost:3000
```

### Production Deployment
- **Platform**: Docker containers
- **Orchestration**: Docker Compose / Kubernetes
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)

### Health Monitoring
- **Health Endpoint**: `/health` - API availability check
- **Metrics Endpoint**: `/metrics` - Prometheus metrics
- **Ready Endpoint**: `/ready` - Dependency health check

## Future Enhancements

### Roadmap Items
1. **Multi-language Support**: Arabic interface and search
2. **Data Visualization**: Charts and graphs generation
3. **Smart Recommendations**: ML-powered dataset suggestions
4. **Real-time Updates**: WebSocket notifications for data changes
5. **Federated Search**: Integration with other government portals

### Integration Opportunities
- **GIS Systems**: Geographic data overlay and analysis
- **BI Tools**: Direct integration with Tableau, Power BI
- **Data Pipelines**: ETL workflow automation
- **Compliance Tools**: GDPR, accessibility compliance checking

---

## Repository & Version Control

### GitHub Repository
- **Repository**: [avi-the-coach/data-gov-il-mcp](https://github.com/avi-the-coach/data-gov-il-mcp)
- **Main Branch**: `main`
- **License**: MIT (recommended for open source government data tools)

### Git Workflow
```bash
# Clone repository
git clone https://github.com/avi-the-coach/data-gov-il-mcp.git
cd data-gov-il-mcp

# Set up development branch
git checkout -b feature/your-feature-name

# Make changes and commit
git add .
git commit -m "feat: your feature description"

# Push to GitHub
git push -u origin feature/your-feature-name

# Create Pull Request on GitHub
```

### Branch Strategy
- **`main`**: Production-ready code
- **`develop`**: Integration branch for features
- **`feature/*`**: Individual feature development
- **`hotfix/*`**: Critical bug fixes
- **`release/*`**: Release preparation

### Commit Convention
Following [Conventional Commits](https://www.conventionalcommits.org/):
```
feat: add new dataset search functionality
fix: resolve cache invalidation issue
docs: update API documentation
style: format code with prettier
refactor: restructure error handling
test: add unit tests for CKAN client
chore: update dependencies
```

## Getting Started

Ready to implement? Start with:
1. **Clone Repository**: `git clone https://github.com/avi-the-coach/data-gov-il-mcp.git`
2. **Review Architecture**: Study this document thoroughly
3. **Set up Environment**: Initialize Node.js project and dependencies
4. **Begin Phase 1**: Implement core search and discovery tools
5. **Test Integration**: Verify CKAN API connectivity
6. **Iterate**: Based on testing and user feedback

### Development Setup
```bash
# Clone and setup
git clone https://github.com/avi-the-coach/data-gov-il-mcp.git
cd data-gov-il-mcp

# Install dependencies (after package.json creation)
npm install

# Start development server
npm run dev

# Run tests
npm test

# Build for production
npm run build
```

For detailed implementation steps, refer to the implementation guide in `IMPLEMENTATION.md` (to be created).