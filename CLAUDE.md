# Data.gov.il MCP Server - Claude Code Development Instructions

## Project Overview

This is an **MCP (Model Context Protocol) server** that provides AI assistants like Claude with seamless access to **Israeli Government Open Data** through the data.gov.il CKAN API.

**Goal**: Build a production-ready MCP server enabling AI assistants to search and access Israeli government datasets in real-time.

**Related Document**: `ARCHITECTURE.md` contains complete technical specification and implementation plan.

## Development Methodology

### Test-Driven Development (TDD) - MANDATORY

**🔴 RED → 🟢 GREEN → 🔵 REFACTOR**: All code must follow strict TDD methodology.

#### TDD Workflow
1. **🔴 Write Test**: Create failing test for new feature
2. **🟢 Minimal Code**: Write just enough code to pass
3. **🔵 Refactor**: Clean up code while tests stay green
4. **🔁 Repeat**: For each MCP tool and component

#### Testing Framework Stack
- **Jest**: Test runner and assertions (`npm i -D jest @types/jest ts-jest`)
- **TypeScript**: Full TS support with ts-jest preset
- **Supertest**: HTTP endpoint testing (`npm i -D supertest @types/supertest`)
- **Nock**: Mock CKAN API responses (`npm i -D nock`)

#### Test Categories
- **Unit Tests**: MCP tool functions, CKAN client methods, cache operations
- **Integration Tests**: End-to-end MCP protocol, real API calls, error scenarios
- **Mock Strategy**: Nock interceptors for CKAN API, edge cases, Hebrew text handling

### Sub-Agent Strategic Deployment

**🤖 Use sub-agents for complex, multi-step tasks requiring specialized focus.**

#### ✅ High-Value Sub-Agent Tasks
- **Complex Research**: Deep CKAN API behavior analysis, edge cases investigation
- **Multi-Component Integration**: Cache + Rate Limiter + Error Handler + MCP Protocol
- **Comprehensive Testing**: End-to-end test suites with mocks and fixtures
- **Performance Engineering**: Load testing, optimization, memory profiling
- **Quality Validation**: Full user journey testing, compatibility verification

#### ❌ Direct Implementation Tasks
- **Simple Unit Tests**: Individual function tests, basic validation
- **Single Component Code**: Basic API client methods, TypeScript interfaces
- **Straightforward Features**: Simple MCP tool implementations following patterns

## Technology Stack & Setup Requirements

### Runtime Requirements
- **Node.js**: v20 LTS ONLY (⚠️ CRITICAL: AVOID v23 - causes npm timeout issues)
- **TypeScript**: v5.0+ with strict mode configuration
- **MCP Framework**: @modelcontextprotocol/sdk v1.17.4+

### Critical Setup Instructions

#### npm Configuration (REQUIRED)
```bash
# MUST run before any npm install to prevent timeouts:
npm config set fetch-retry-maxtimeout 600000
npm config set fetch-retry-mintimeout 120000
npm config set fetch-timeout 600000
```

#### Project Dependencies (Auto-installed)
```json
{
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.17.4",
    "axios": "^1.6.0",
    "winston": "^3.17.0", 
    "node-cache": "^5.1.0",
    "joi": "^17.11.0",
    "dotenv": "^16.3.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^20.19.11",
    "jest": "^29.0.0",
    "ts-jest": "^29.0.0",
    "@types/jest": "^29.0.0",
    "supertest": "^6.3.0",
    "nock": "^13.3.0",
    "eslint": "^8.0.0"
  }
}
```

### Windows-Specific Troubleshooting
```bash
# If npm install fails:
# 1. Run Command Prompt as Administrator
# 2. Temporarily disable Windows Defender real-time protection
# 3. npm cache clean --force
# 4. Delete node_modules and package-lock.json, retry
# 5. Consider: npm install --no-audit --prefer-offline
```

## Critical Implementation Lessons

### TypeScript Configuration Gotchas
- `exactOptionalPropertyTypes: true` requires explicit undefined handling
- **Solution patterns**:
  - `this.property = undefined as undefined`
  - `apiKey?: string | undefined` in interfaces

### CKAN API Integration Insights
- **Base URL**: `https://data.gov.il/api/3/action/` (tested and working)
- **Hebrew Text**: Fully supported in search and results
- **Performance**: < 2 seconds for most searches
- **Rate Limits**: No authentication required for public data
- **Critical Behavior**: `package_search` searches metadata only, `datastore_search` searches content

### Real-World Success Criteria
**✅ Deployment readiness checklist:**
1. `npm run build` - Compiles TypeScript without errors
2. `node dist/index.js` - Starts MCP server successfully
3. Server logs show "MCP Server initialized"
4. 403 CKAN error is expected (not a blocker)

### Zero-Interruption Implementation
- [ ] Node.js v20 LTS verified (`node --version`)
- [ ] npm timeout configuration applied
- [ ] TypeScript compilation successful
- [ ] MCP server startup verified

## Development Workflow

### Git Strategy
- **Main Branch**: `main` (production-ready)
- **Feature Branches**: `feature/tool-name` or `feature/phase-1-setup`
- **Commits**: Conventional commits (feat:, fix:, docs:, test:, refactor:)

### Commit Convention
```bash
feat: add query_datastore MCP tool
fix: resolve cache invalidation issue
docs: update API documentation
test: add integration tests for CKAN client
refactor: restructure error handling
```

### TDD Development Commands
```bash
npm test                      # Run all tests
npm run test:watch            # Jest watch mode for TDD
npm test -- --coverage       # Code coverage report
npm test search_datasets      # Run specific test file
npm run build                 # TypeScript compilation
npm run dev                   # Development server
```

## Performance & Quality Standards

### Performance Targets
- **Response Time**: < 500ms cached, < 2s API calls
- **Throughput**: 50+ concurrent requests
- **Test Coverage**: 90%+ for all MCP tools
- **Memory Usage**: Efficient cache management

### Error Handling Strategy
- **Retry Logic**: Exponential backoff for transient errors
- **Circuit Breaker**: Fail-fast for API issues
- **Fallback**: Cached data when available
- **User Messages**: Clear, actionable error descriptions

### Caching Architecture
- **Strategy**: LRU cache with TTL
- **Storage**: Memory (dev) / Redis (prod)
- **TTL**: 1h metadata, 24h static data
- **Key Patterns**: `dataset:{id}`, `search:{hash}`, `org:{id}`

### Rate Limiting
- **Search Endpoints**: 100 requests/minute
- **Data Endpoints**: 50 requests/minute
- **Implementation**: Token bucket per endpoint

## Security & Best Practices

### Code Standards
- **No Comments**: DO NOT ADD comments unless explicitly requested
- **TypeScript Strict**: Full type safety with proper interfaces
- **Input Validation**: Sanitize all user inputs with Joi
- **Error Boundaries**: Comprehensive error handling
- **No Secrets**: Never log or expose API keys

### Data Protection
- **Public Data Only**: Focus on open government data
- **No PII Storage**: Cache only public metadata
- **Audit Logging**: Track usage patterns and errors

## Session Management

### When Starting New Development Session
1. **Read ARCHITECTURE.md** - Complete technical specification
2. **Check Git Status** - Current progress and branch
3. **Review TodoWrite** - Session-level task tracking
4. **Follow TDD First** - Always write tests before implementation
5. **Use Sub-Agents** - For complex multi-component tasks

### Common Quick Commands
```bash
git status                    # Check current progress
npm test                      # Verify all tests pass
ls -la                        # Project files overview
git branch -v                 # Current branch info
```

### Context Handoff Between Sessions
- All architectural decisions and implementation details in `ARCHITECTURE.md`
- Current session progress tracked with TodoWrite tool
- Git commits provide detailed change history
- This file (CLAUDE.md) provides development methodology only

## Quality Assurance

### Definition of Done
- [ ] TDD implemented (tests written first)
- [ ] All tests passing (unit + integration)
- [ ] TypeScript compilation successful
- [ ] Code coverage > 90%
- [ ] Error handling implemented
- [ ] Performance targets met
- [ ] No security vulnerabilities
- [ ] Documentation updated

### Integration Testing
- **MCP Protocol**: Full request/response cycles
- **CKAN API**: Real API connectivity tests (rate limited)
- **Cache Integration**: Redis/Memory cache behavior
- **Hebrew Text**: Unicode handling verification

---

## Development Philosophy

**Focus**: Implement the minimum viable MCP tools that enable AI assistants to effectively search and access Israeli government data.

**Quality**: Every line of code is test-driven, well-architected, and production-ready.

**Efficiency**: Use sub-agents strategically for complex tasks, handle simple implementations directly.

**User Value**: Enable Claude and other AI assistants to provide intelligent insights from Israeli government data.

The goal is to create a robust, scalable MCP server that democratizes access to Israeli government data through AI assistants.