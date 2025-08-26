# Data.gov.il MCP Server - Claude Context

## Project Overview

This is an **MCP (Model Context Protocol) server** that provides AI assistants like Claude with seamless access to **Israeli Government Open Data** through the data.gov.il CKAN API.

### What We're Building
- **Government Data Bridge**: Connect AI to Israeli public datasets in real-time
- **Intelligent Discovery**: Enable smart search and exploration of government data
- **Production-Ready Tool**: Robust, scalable MCP server with comprehensive features

### Key Value Proposition
Make Israeli government data easily accessible to AI assistants for analysis, insights, and citizen services.

## Architecture Foundation

**📋 MUST READ**: `ARCHITECTURE.md` contains the complete technical specification:
- **System Architecture**: 4-layer design (Tools → Service → Data → API)
- **20+ MCP Tools**: Across discovery, access, analytics, and utilities
- **3-Phase Implementation**: Essential → Enhanced → Advanced features
- **Production Features**: Caching, rate limiting, error handling, monitoring

### Technology Stack
- **Runtime**: Node.js 18+ with TypeScript
- **MCP Framework**: @modelcontextprotocol/sdk
- **API Integration**: CKAN v3 REST API (data.gov.il)
- **Deployment**: Docker containers with monitoring

## Current Project State

### ✅ Completed
- [x] **Research Phase**: CKAN API analysis and Israeli data portal investigation
- [x] **Architecture Design**: Comprehensive technical specification
- [x] **Repository Setup**: GitHub repo connected with Git workflow
- [x] **Documentation**: Complete ARCHITECTURE.md with 3-phase implementation plan

### 🚧 Next Steps (Implementation Ready)
- [ ] **Node.js Project Setup**: package.json, TypeScript config, MCP SDK
- [ ] **Phase 1 Tools**: Core discovery tools (search, details, organizations)
- [ ] **CKAN Client**: API integration with error handling
- [ ] **Testing Framework**: Unit and integration tests

## Implementation Priorities

### Phase 1: Essential Tools (Priority 1)
Focus on these **5 core MCP tools** first:
1. `search_datasets` - Smart search with filters
2. `get_dataset_details` - Complete dataset metadata
3. `list_organizations` - Government ministries/agencies
4. `browse_by_category` - Topic-based exploration
5. `get_resource_data` - Access actual data files

### Critical Implementation Notes
- **API Base URL**: `https://data.gov.il/api/3/action/`
- **Authentication**: API key optional for public data
- **Error Handling**: Implement circuit breaker pattern from day 1
- **Caching Strategy**: 1h TTL for metadata, 24h for static data
- **Rate Limiting**: 100 req/min search, 50 req/min data access

## Development Workflow

### Git Strategy
- **Main Branch**: `main` (production-ready)
- **Feature Branches**: `feature/tool-name` or `feature/phase-1-setup`
- **Commits**: Use conventional commits (feat:, fix:, docs:, etc.)

### Before You Start Coding
1. **Read ARCHITECTURE.md** - Full technical specification
2. **Check Current Branch** - Create feature branch if needed
3. **Review Phase Plan** - Focus on Phase 1 essential tools first
4. **Test API Access** - Verify data.gov.il connectivity

## Key Architectural Decisions

### MCP Tools Design Pattern
```typescript
// Each tool follows this structure:
interface MCPTool {
  name: string
  description: string
  inputSchema: JSONSchema
  handler: (params: any) => Promise<ToolResult>
}
```

### CKAN API Integration
- **Search Pattern**: `/package_search` with advanced filtering
- **Detail Pattern**: `/package_show` for complete metadata
- **Data Access**: `/datastore_search` for structured queries
- **Error Strategy**: Retry with exponential backoff

### Caching Architecture
- **Memory Cache**: Development (node-cache)
- **Redis Cache**: Production (optional)
- **Cache Keys**: `dataset:{id}`, `search:{hash}`, `org:{id}`
- **Invalidation**: TTL-based with manual refresh capability

## Testing Strategy

### Must-Have Tests
- **Unit Tests**: Each MCP tool function
- **Integration Tests**: CKAN API connectivity
- **Error Handling**: Network failures, API errors
- **Cache Tests**: Hit/miss rates, invalidation

### Test Data Sources
- Use real data.gov.il datasets for integration tests
- Mock CKAN responses for unit tests
- Test Hebrew/English language support

## Deployment Considerations

### Environment Variables
```bash
CKAN_BASE_URL=https://data.gov.il/api/3/action
CACHE_TTL_SECONDS=3600
RATE_LIMIT_REQUESTS=100
LOG_LEVEL=info
```

### Performance Targets
- **Response Time**: < 500ms for cached data, < 2s for API calls
- **Throughput**: 50+ concurrent requests
- **Availability**: 99.9% uptime target

## Context for Future Sessions

### When You Return to This Project
1. **Read This File First** - Get project context quickly
2. **Check ARCHITECTURE.md** - Review technical specifications
3. **Review Git Status** - See current progress and branch
4. **Continue Implementation** - Follow 3-phase plan

### Common Commands
```bash
# Development workflow
git status                    # Check current progress
npm run dev                   # Start development server
npm test                      # Run test suite
git log --oneline -10         # Recent commits

# Quick status check
ls -la                        # Project files
git branch -v                 # Current branch info
```

## Project Success Criteria

### Phase 1 Success Metrics
- [ ] 5 essential MCP tools implemented and tested
- [ ] Successfully connects to data.gov.il CKAN API
- [ ] Handles basic errors gracefully (network, API failures)
- [ ] Responds to searches within 2 seconds
- [ ] Compatible with Claude Desktop MCP integration

### Long-term Vision
- **Government Integration**: Adopted by Israeli government agencies
- **AI Accessibility**: Standard tool for AI-powered civic analysis
- **Open Source Impact**: Template for other government MCP servers
- **Data Democracy**: Citizens access government data through AI assistants

---

## Quick Start Reminder

When continuing work:
1. **Review current todo list** (if using TodoWrite tool)
2. **Read relevant sections** of ARCHITECTURE.md
3. **Check git status** and current branch
4. **Focus on Phase 1 implementation** unless specifically working on other phases

The goal is to make Israeli government data easily accessible through AI assistants - keep this mission in mind with every implementation decision.