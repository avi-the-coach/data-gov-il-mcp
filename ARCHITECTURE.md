# Data.gov.il MCP Server - Technical Architecture & Implementation Plan

## Project Overview

This MCP (Model Context Protocol) server provides seamless access to Israeli Government Open Data through the data.gov.il CKAN API, enabling AI assistants like Claude to discover, search, and analyze government datasets in real-time.

### Goals
- **Accessibility**: Make Israeli government data easily discoverable through AI
- **Intelligence**: Enable AI-powered data analysis and citizen insights  
- **Reliability**: Production-ready with robust error handling and caching
- **Extensibility**: Modular architecture supporting future enhancements

### Key Value Proposition
Transform how citizens and organizations interact with Israeli government data by making it accessible through natural language queries to AI assistants.

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

## Technology Stack

### Core Framework
- **Runtime**: Node.js 20 LTS with TypeScript 5.0+
- **MCP SDK**: @modelcontextprotocol/sdk v1.17.4+
- **HTTP Client**: Axios v1.6+ for CKAN API integration
- **Validation**: Joi v17+ for input parameter validation
- **Logging**: Winston v3+ structured logging

### Development Tools
- **Testing**: Jest with ts-jest, Supertest, Nock for mocking
- **Build**: TypeScript compiler with strict mode
- **Linting**: ESLint with TypeScript support
- **Development**: Nodemon for auto-reload

### Infrastructure
- **Caching**: Node-cache (dev) / Redis (production)
- **Environment**: Dotenv for configuration management
- **Deployment**: Docker containers with health checks

## MCP Tools Specification

The server implements 20+ MCP tools across four categories, enabling comprehensive Israeli government data access.

### Phase 1: Essential Discovery Tools (Priority 1)

#### search_datasets
**Purpose**: Intelligent search across 1,179+ Israeli government datasets
```typescript
interface SearchDatasetsParams {
  query?: string;           // Search term (Hebrew/English)
  organization?: string;    // Filter by ministry/agency
  tags?: string[];         // Topic filtering
  format?: string[];       // Data format (CSV, JSON, XML)
  limit?: number;          // Results per page (max 100)
  offset?: number;         // Pagination offset
  sort?: 'relevance' | 'name' | 'modified' | 'popularity';
}
```

#### query_datastore
**Purpose**: Search within actual dataset content (vehicle records, etc.)
```typescript
interface QueryDatastoreParams {
  resource_id: string;      // Dataset resource ID
  query?: string;          // Content search query
  filters?: Record<string, any>; // Field-based filtering
  limit?: number;          // Results limit
  offset?: number;         // Pagination
  fields?: string[];       // Specific fields to return
}
```

#### similarity_search  
**Purpose**: Find datasets by topic similarity ("cars", "banks", "insurance")
```typescript
interface SimilaritySearchParams {
  query: string;           // Topic or concept
  threshold?: number;      // Similarity score threshold
  limit?: number;          // Max results
  categories?: string[];   // Restrict to categories
}
```

#### get_dataset_schema
**Purpose**: Understand available fields for targeted searches
```typescript
interface DatasetSchemaParams {
  dataset_id: string;      // Dataset identifier
  resource_id?: string;    // Specific resource
  include_sample?: boolean; // Include sample data
}
```

#### list_organizations
**Purpose**: Browse government ministries and agencies
```typescript
interface ListOrganizationsParams {
  include_dataset_count?: boolean;
  sort?: 'name' | 'dataset_count';
  category?: string;       // Ministry type filtering
}
```

### Phase 2: Enhanced Data Access Tools

#### get_dataset_details
**Purpose**: Complete metadata and resource information
```typescript
interface DatasetDetailsParams {
  id: string;              // Dataset ID
  include_resources?: boolean;
  include_stats?: boolean;
}
```

#### browse_by_category
**Purpose**: Explore datasets by topic or theme
```typescript  
interface BrowseCategoryParams {
  category: string;        // Government category
  subcategory?: string;    // Refined filtering
  limit?: number;
  sort?: string;
}
```

#### get_resource_data
**Purpose**: Access actual data files and content
```typescript
interface ResourceDataParams {
  resource_id: string;     // Resource identifier
  format?: 'json' | 'csv' | 'xml';
  limit?: number;          // Data rows limit
  preview?: boolean;       // Sample data only
}
```

### Phase 3: Advanced Analytics Tools

#### get_dataset_stats
**Purpose**: Usage metrics and popularity data
#### analyze_data_coverage
**Purpose**: Geographic and temporal analysis
#### compare_datasets  
**Purpose**: Similarity and relationship analysis
#### validate_data_quality
**Purpose**: Completeness and format validation

## Data Models

### Core Dataset Structure
```typescript
interface Dataset {
  id: string;
  title: string;
  description: string;
  organization: {
    id: string;
    name: string;
    title: string;
  };
  tags: Array<{
    name: string;
    display_name: string;
  }>;
  resources: Resource[];
  metadata: {
    created: Date;
    modified: Date;
    license: string;
    language: 'he' | 'en' | 'ar';
    coverage: {
      temporal?: string;
      geographic?: string;
    };
  };
  statistics: {
    views: number;
    downloads: number;
    popularity_score: number;
  };
}
```

### Resource Definition
```typescript
interface Resource {
  id: string;
  name: string;
  description?: string;
  format: string;          // CSV, JSON, XML, XLS, etc.
  size: number;           // File size in bytes
  url: string;            // Download URL
  datastore_active: boolean;
  created: Date;
  modified: Date;
  schema?: {
    fields: Array<{
      name: string;
      type: string;
      description?: string;
    }>;
  };
}
```

## CKAN API Integration

### Base Configuration
- **Primary URL**: `https://data.gov.il/api/3/action/`
- **Fallback URL**: `https://info.data.gov.il/api/3/action/`
- **Authentication**: Optional API key for enhanced rate limits
- **Default Headers**: `Accept: application/json`, `User-Agent: data-gov-il-mcp`

### Critical Endpoints
```typescript
interface CKANEndpoints {
  // Metadata search (searches titles, descriptions, tags)
  '/package_search': SearchDatasetsParams;
  
  // Content search (searches within actual data)
  '/datastore_search': QueryDatastoreParams;
  
  // Dataset details
  '/package_show': { id: string };
  
  // Resource information  
  '/resource_show': { id: string };
  
  // Organization list (49 ministries/agencies available)
  '/organization_list': { all_fields: boolean };
  
  // Available tags and categories
  '/tag_list': Record<string, never>;
}
```

### API Behavior Insights
- **Hebrew Language**: Fully supported for queries and responses
- **Performance**: Most requests complete under 2 seconds
- **Rate Limiting**: No strict limits for public data access
- **Error Responses**: Standard HTTP codes with detailed JSON messages
- **Search Scoping**: `package_search` for metadata, `datastore_search` for content

## Service Layer Architecture

### CKAN API Client
```typescript
class CKANClient {
  constructor(baseUrl: string, apiKey?: string);
  
  // Core search methods
  async searchDatasets(params: SearchDatasetsParams): Promise<SearchResponse>;
  async queryDatastore(params: QueryDatastoreParams): Promise<QueryResponse>;
  async getDataset(id: string): Promise<Dataset>;
  async getResource(id: string): Promise<Resource>;
  async listOrganizations(): Promise<Organization[]>;
  
  // Internal utilities
  private async makeRequest<T>(endpoint: string, params: any): Promise<T>;
  private handleApiError(error: any): never;
  private validateParams(params: any, schema: Joi.Schema): any;
}
```

### Cache Manager
```typescript
interface CacheConfig {
  ttl: {
    metadata: number;      // 1 hour for dataset metadata
    search: number;        // 30 minutes for search results  
    static: number;        // 24 hours for organizations/tags
  };
  maxSize: number;         // Maximum cache entries
  strategy: 'lru';         // Least Recently Used eviction
}
```

### Error Handler
```typescript
interface ErrorHandlingStrategy {
  retryAttempts: 3;
  backoffMultiplier: 2;
  circuitBreakerThreshold: 5;
  fallbackToCache: boolean;
  userFriendlyMessages: boolean;
}
```

### Rate Limiter
```typescript
interface RateLimitConfig {
  search: { requests: 100, window: 60000 };     // 100/minute
  data: { requests: 50, window: 60000 };        // 50/minute  
  bulk: { requests: 10, window: 60000 };        // 10/minute
}
```

## Real-World Data Landscape

### Government Organizations (49 Available)
- **Ministry of Transportation**: 605+ datasets
  - Vehicle licensing (private & commercial from 1996+)
  - Traffic data, road infrastructure
  - License plate lookup capability
- **Ministry of Justice**: 82 datasets  
  - Patent registrations, legal classifications
  - Court data and legal proceedings
- **Ministry of Health**: Healthcare facilities and statistics
- **Bank of Israel**: Financial and economic data
- **Airport Authority**: Real-time flight information
- **Beer Sheva Municipality**: 593 municipal datasets

### Proven Dataset Examples
- **Vehicle Registration**: 841MB+ dataset with daily updates
  - Searchable by license number (e.g., "29721704")
  - Technical specifications, registration history
  - Private and commercial vehicle data from 1996+
- **Government Patents**: Legal classifications and applications
- **Healthcare Facilities**: Location data and services offered
- **Flight Data**: Updated every 15 minutes with arrivals/departures
- **Municipal Services**: Geographic data, public services

## Implementation Plan

### Phase 1: Foundation & Core Tools (Weeks 1-2)

#### Epic 1: Project Infrastructure
**Tasks:**
1. **Node.js Project Setup**
   - Initialize package.json with dependencies
   - Configure TypeScript with strict mode
   - Set up Jest testing framework with ts-jest
   - Create project structure (src/, tests/, dist/)

2. **MCP Server Core**
   - Implement MCP SDK integration
   - Set up tool registration system
   - Create request/response handling
   - Add structured logging with Winston

3. **CKAN API Client**
   - Build HTTP client with Axios
   - Implement error handling with retry logic
   - Add input validation with Joi schemas
   - Create response caching layer

#### Epic 2: Essential MCP Tools
**Priority Order:**
1. **search_datasets** - Core discovery functionality
2. **query_datastore** - Content search capability  
3. **similarity_search** - Intelligent topic discovery
4. **get_dataset_schema** - Field exploration
5. **list_organizations** - Ministry browsing

**Acceptance Criteria per Tool:**
- TDD implementation with 90%+ test coverage
- Input validation and error handling
- Response caching with appropriate TTL
- Hebrew text support verified
- Performance under 2 seconds

#### Epic 3: Integration & Testing
**Tasks:**
1. **Claude Desktop Integration**
   - Configure MCP server connection
   - Test tool discovery and execution
   - Verify request/response formatting

2. **End-to-End Testing**
   - Real CKAN API connectivity tests
   - Hebrew language query testing
   - Error scenario validation
   - Performance benchmarking

### Phase 2: Enhanced Features (Weeks 3-4)

#### Advanced Caching System
- Implement Redis for production caching
- Add cache invalidation strategies
- Multi-level cache hierarchy
- Cache hit rate monitoring

#### Additional Discovery Tools
- get_dataset_details with full metadata
- browse_by_category with topic filtering  
- get_resource_data with format conversion
- Enhanced search with faceted filtering

#### Performance Optimization
- Connection pooling for HTTP requests  
- Response compression and batching
- Memory usage optimization
- Load testing and benchmarking

### Phase 3: Production Features (Weeks 5-6)

#### Analytics & Monitoring
- Usage statistics and popular queries
- Performance metrics and alerting
- Data freshness monitoring
- User behavior analysis

#### Advanced Analytics Tools
- Cross-dataset comparison and correlation
- Geographic data visualization support
- Temporal trend analysis
- Data quality scoring

## Configuration Management

### Environment Variables
```bash
# API Configuration
CKAN_BASE_URL=https://data.gov.il/api/3/action
CKAN_API_KEY=optional_api_key_for_enhanced_limits
CKAN_FALLBACK_URL=https://info.data.gov.il/api/3/action

# Cache Settings
CACHE_PROVIDER=memory          # memory|redis
CACHE_TTL_METADATA=3600        # 1 hour
CACHE_TTL_SEARCH=1800          # 30 minutes  
CACHE_TTL_STATIC=86400         # 24 hours
CACHE_MAX_SIZE=1000            # Max entries

# Rate Limiting
RATE_LIMIT_SEARCH_RPM=100      # Requests per minute
RATE_LIMIT_DATA_RPM=50
RATE_LIMIT_BULK_RPM=10

# Server Configuration
PORT=3000
LOG_LEVEL=info
LOG_FORMAT=json
NODE_ENV=development

# Redis (Production)
REDIS_URL=redis://localhost:6379
REDIS_DB=0
```

### Feature Flags
```typescript
interface FeatureFlags {
  enableDatastoreQueries: boolean;    // query_datastore tool
  enableSimilaritySearch: boolean;    // AI-powered discovery
  enableAdvancedAnalytics: boolean;   // Stats and insights
  enableCaching: boolean;             // Response caching
  enableRateLimiting: boolean;        // Request throttling
  enableMetrics: boolean;             // Performance monitoring
}
```

## Error Handling & Resilience

### Error Categories & Responses
```typescript
enum ErrorType {
  NETWORK_ERROR = 'network_error',        // Connection issues
  API_ERROR = 'api_error',               // CKAN API failures  
  VALIDATION_ERROR = 'validation_error', // Invalid parameters
  RATE_LIMIT_ERROR = 'rate_limit_error', // Too many requests
  CACHE_ERROR = 'cache_error',           // Cache operations
  INTERNAL_ERROR = 'internal_error'      // Server issues
}

interface ErrorResponse {
  error: {
    type: ErrorType;
    message: string;
    details?: any;
    retryAfter?: number;
    suggestions?: string[];
  };
}
```

### Resilience Strategies
- **Circuit Breaker**: Fail-fast after consecutive failures
- **Retry Logic**: Exponential backoff for transient errors
- **Timeout Handling**: Request timeouts with graceful degradation
- **Fallback Data**: Return cached results when API unavailable
- **Health Checks**: Monitor CKAN API availability

## Security Considerations

### Data Protection
- **Public Data Focus**: Only access open government datasets
- **No PII Storage**: Cache metadata only, not personal information
- **Input Sanitization**: Validate all user inputs before API calls
- **API Key Security**: Environment-based configuration only

### Access Control
- **Rate Limiting**: Prevent abuse and resource exhaustion
- **Request Logging**: Audit trail for usage monitoring  
- **Error Sanitization**: No sensitive information in error messages
- **HTTPS Only**: Secure communication with CKAN API

## Performance Targets

### Response Times
- **Cached Responses**: < 100ms
- **Metadata Queries**: < 500ms  
- **Data Content Queries**: < 2 seconds
- **Bulk Operations**: < 5 seconds

### Throughput
- **Concurrent Requests**: 50+ simultaneous
- **Daily Request Capacity**: 100,000+ requests
- **Cache Hit Rate**: > 80% for repeated queries
- **Memory Usage**: < 512MB baseline

### Reliability
- **Uptime Target**: 99.9% availability
- **Error Rate**: < 1% of requests
- **Recovery Time**: < 30 seconds after failures
- **Data Freshness**: Updates within 1 hour of source changes

## Deployment Architecture

### Development Environment
```bash
# Quick start
npm install
npm run dev         # Start with hot reload
npm test           # Run test suite
npm run build      # Compile TypeScript
```

### Production Deployment
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY dist ./dist
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

### Monitoring & Observability
- **Health Endpoint**: `/health` - API availability
- **Metrics Endpoint**: `/metrics` - Prometheus format
- **Ready Endpoint**: `/ready` - Dependency health
- **Logging**: Structured JSON logs with correlation IDs

## Getting Started

### Prerequisites
1. **Node.js v20 LTS** installed and verified
2. **Git** for version control
3. **Code Editor** with TypeScript support
4. **Terminal** access for npm commands

### Initial Setup
```bash
# Clone repository
git clone https://github.com/avi-the-coach/data-gov-il-mcp.git
cd data-gov-il-mcp

# Install dependencies
npm install

# Run tests
npm test

# Start development
npm run dev
```

### First Implementation Steps
1. **Read CLAUDE.md** - Development methodology and best practices
2. **Set up TDD environment** - Jest, TypeScript, test structure
3. **Implement search_datasets** - Start with core discovery tool
4. **Test CKAN API connectivity** - Verify data.gov.il access
5. **Add query_datastore** - Enable content search capability
6. **Integrate with Claude Desktop** - Test MCP protocol

### Success Metrics
- [ ] All 5 Phase 1 MCP tools implemented and tested
- [ ] Search across 1,179+ datasets functional  
- [ ] Vehicle data searchable by license number
- [ ] Hebrew language support verified
- [ ] Claude Desktop integration working
- [ ] 90%+ test coverage achieved
- [ ] Performance targets met

---

This architecture provides a solid foundation for building a production-ready MCP server that democratizes access to Israeli government data through AI assistants. The phased implementation approach ensures rapid value delivery while building toward a comprehensive, scalable solution.