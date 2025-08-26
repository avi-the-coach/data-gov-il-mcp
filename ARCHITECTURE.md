# Data.gov.il MCP Server - Architecture & Design Document

## Overview

This MCP (Model Context Protocol) server provides seamless access to Israeli Government Open Data through the data.gov.il CKAN API. It enables AI assistants like Claude to discover, search, and access government datasets in real-time.

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

#### Phase 1: Essential Discovery Tools
- `search_datasets` - Smart search with advanced filters
- `get_dataset_details` - Complete dataset metadata
- `list_organizations` - Government ministries and agencies
- `browse_by_category` - Topic-based exploration
- `get_resource_data` - Access actual data files

#### Phase 2: Enhanced Data Access
- `query_datastore` - SQL-like queries on structured data
- `get_dataset_stats` - Usage and popularity metrics
- `find_by_ministry` - Ministry-specific filtering
- `check_data_freshness` - Update status monitoring
- `list_recent_datasets` - Recently published/updated data

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
- Production: `https://data.gov.il/api/3/action/`
- Fallback: `https://info.data.gov.il/api/3/action/`

### Key Endpoints
```
GET /package_search          - Search datasets
GET /package_show           - Get dataset details  
GET /resource_show          - Get resource details
GET /datastore_search       - Query structured data
GET /organization_list      - List organizations
GET /tag_list              - List all tags
```

### Authentication
- **Method**: API Key (if required)
- **Header**: `Authorization: {api_key}`
- **Fallback**: Anonymous access for public data

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

## Technology Stack

### Runtime & Language
- **Node.js**: v18+ (LTS)
- **TypeScript**: v5.0+
- **MCP SDK**: @modelcontextprotocol/sdk

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

## Getting Started

Ready to implement? Start with:
1. Review this architecture document
2. Set up the development environment
3. Begin Phase 1 implementation
4. Test with basic search functionality
5. Iterate based on user feedback

For questions or clarifications, refer to the implementation guide in `IMPLEMENTATION.md` (to be created).