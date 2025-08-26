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

**📋 MUST READ**: Key project documents:
- **`ARCHITECTURE.md`**: Complete technical specification and system design
- **`IMPLEMENTATION.md`**: Detailed Phase 1 execution plan with task breakdown
- **System Architecture**: 4-layer design (Tools → Service → Data → API)
- **20+ MCP Tools**: Across discovery, access, analytics, and utilities
- **3-Phase Implementation**: Essential → Enhanced → Advanced features
- **Production Features**: Caching, rate limiting, error handling, monitoring

### Technology Stack
- **Runtime**: Node.js 20 LTS with TypeScript (⚠️ AVOID v23 - has npm timeout issues)
- **MCP Framework**: @modelcontextprotocol/sdk v1.17.4+
- **API Integration**: CKAN v3 REST API (data.gov.il)
- **Testing**: Jest with ts-jest (requires specific ES module configuration)
- **Deployment**: Docker containers with monitoring

## Current Project State

### ✅ Completed & Working
- [x] **Research Phase**: CKAN API analysis and Israeli data portal investigation
- [x] **Architecture Design**: Comprehensive technical specification  
- [x] **Repository Setup**: GitHub repo connected with Git workflow
- [x] **Node.js Project Setup**: package.json, TypeScript config, MCP SDK ✅ WORKING
- [x] **TDD Testing Setup**: Jest + TypeScript testing framework ✅ WORKING
- [x] **CKAN Client**: API integration with error handling ✅ TESTED & WORKING
- [x] **MCP Server Core**: Tool registration and request handling ✅ WORKING
- [x] **Claude Desktop Integration**: Configuration and testing ✅ WORKING
- [x] **search_datasets Tool**: Searches 1,179+ datasets across 49 orgs ✅ WORKING

### 🔥 CRITICAL Missing Tools (High Priority)
- [ ] **query_datastore Tool**: Search within actual data content (vehicle records, etc.)
- [ ] **similarity_search Tool**: Find datasets by similarity ("cars", "banks", "insurance")
- [ ] **get_dataset_schema Tool**: Understand available data fields for targeted search
- [ ] **list_organizations Tool**: Get org list with dataset counts (partial working)

## 🔧 MANUAL SETUP REQUIREMENTS

**CRITICAL**: The following steps must be performed manually BEFORE starting automated implementation:

### Prerequisites (One-Time Global Setup)
```bash
# Install Node.js v18+ (LTS recommended, avoid v23 for stability)
# Download from: https://nodejs.org/en/download/
# Verify installation:
node --version  # Should be v18.x.x or v20.x.x
npm --version   # Should be 9.x.x or 10.x.x

# Optional: Install global development tools for convenience
npm install -g typescript      # TypeScript compiler
npm install -g nodemon         # Development auto-reload
npm install -g yarn           # Alternative package manager (if preferred)
```

### Project-Specific Setup (Required for Each New Project)
```bash
# 1. Navigate to project directory
cd "C:\Users\aviba\Documents\Automation\data-gov-il-mcp"

# 2. Install project dependencies
# OPTION A: Standard npm (may timeout on some systems)
npm install

# OPTION B: If npm times out, try these alternatives:
npm install --no-audit --prefer-offline
# OR
npm config set fetch-retry-maxtimeout 600000
npm config set fetch-retry-mintimeout 120000
npm install
# OR
yarn install  # If yarn is installed

# 3. Verify installation success
npm test              # Should run Jest test suite
npm run build         # Should compile TypeScript
npm run dev           # Should start development server
```

### Windows-Specific Troubleshooting
```bash
# If npm install keeps timing out:
# 1. Run Command Prompt as Administrator
# 2. Temporarily disable Windows Defender real-time protection
# 3. Try: npm cache clean --force
# 4. Delete node_modules and package-lock.json, then retry
# 5. Consider using Node.js v20 LTS instead of v23
```

### Expected Dependencies (Auto-installed by npm/yarn)

**Production Dependencies:**
- `@modelcontextprotocol/sdk: ^1.17.4` - MCP protocol implementation
- `axios: ^1.6.0` - HTTP client for CKAN API
- `winston: ^3.17.0` - Logging framework
- `node-cache: ^5.1.0` - In-memory caching
- `joi: ^17.11.0` - Schema validation
- `dotenv: ^16.3.0` - Environment variable management

**Development Dependencies:**
- `typescript: ^5.0.0` - TypeScript compiler
- `@types/node: ^20.19.11` - Node.js type definitions
- `jest: ^29.0.0` - Testing framework
- `ts-jest: ^29.0.0` - TypeScript Jest preprocessor
- `@types/jest: ^29.0.0` - Jest type definitions
- `supertest: ^6.3.0` - HTTP testing library
- `@types/supertest: ^2.0.0` - Supertest type definitions
- `nock: ^13.3.0` - HTTP mocking library
- `eslint: ^8.0.0` - Code linting
- `@typescript-eslint/parser: ^6.0.0` - TypeScript ESLint parser
- `@typescript-eslint/eslint-plugin: ^6.0.0` - TypeScript ESLint rules
- `nodemon: ^3.0.0` - Development auto-reload
- `ts-node: ^10.9.0` - TypeScript execution engine

### Verification Commands
```bash
# Verify all dependencies installed correctly:
ls node_modules                    # Should show all packages
npm list --depth=0                # Show installed package versions

# Verify TypeScript compilation:
npx tsc --version                  # Should show TypeScript version
npx tsc --noEmit                   # Check for TypeScript errors

# Verify test framework:
npx jest --version                 # Should show Jest version

# Verify MCP SDK:
node -e "console.log(require('@modelcontextprotocol/sdk/package.json').version)"
```

## Implementation Priorities

### Phase 1: Essential Tools (Priority 1)
Focus on these **CRITICAL MCP tools**:
1. `search_datasets` - Smart search with filters ✅ **WORKING** (1,179+ datasets)
2. `query_datastore` - **MISSING & CRITICAL**: Search within actual data content 🔥
3. `similarity_search` - **MISSING & REQUIRED**: Find datasets by similarity 🔥
4. `get_dataset_schema` - **MISSING & REQUIRED**: Understand data fields 🔥
5. `list_organizations` - Government ministries/agencies (partial working)

### Critical Implementation Notes (BATTLE-TESTED)
- **API Base URL**: `https://data.gov.il/api/3/action/` ✅ WORKING & TESTED
- **Authentication**: API key optional for public data ✅ CONFIRMED
- **Error Handling**: Circuit breaker pattern implemented ✅ WORKING
- **Performance**: < 2 seconds response time ✅ VERIFIED
- **Hebrew Support**: Full Hebrew text search working ✅ TESTED
- **Real Data Volume**: 1,179+ datasets, 49 organizations ✅ CONFIRMED

## Development Workflow

### Git Strategy
- **Main Branch**: `main` (production-ready)
- **Feature Branches**: `feature/tool-name` or `feature/phase-1-setup`
- **Commits**: Use conventional commits (feat:, fix:, docs:, etc.)

### Before You Start Coding
1. **Read ARCHITECTURE.md** - Full technical specification and system design
2. **Read IMPLEMENTATION.md** - Phase 1 detailed execution plan and task breakdown
3. **Check Current Branch** - Create feature branch if needed
4. **Review Current Epic/Task** - Follow implementation plan priorities
5. **Test API Access** - Verify data.gov.il connectivity

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

## Testing Strategy (TDD Methodology)

**🔴 RED → 🟢 GREEN → 🔵 REFACTOR**: Write failing tests first, make them pass, then refactor.

### Testing Framework Stack
- **Jest**: Test runner and assertions (`npm i -D jest @types/jest ts-jest`)
- **TypeScript**: Full TS support with ts-jest preset
- **Supertest**: HTTP endpoint testing (`npm i -D supertest @types/supertest`)
- **Nock**: Mock CKAN API responses (`npm i -D nock`)
- **@testcontainers/node**: Optional Redis cache testing

### TDD Test Structure
```typescript
// Example TDD cycle for search_datasets tool:
describe('search_datasets', () => {
  it('should return datasets matching query', async () => {
    // 🔴 RED: Write test first (will fail)
    const result = await searchDatasets({ query: 'health' })
    expect(result.datasets).toHaveLength(5)
  })
})
```

### Test Categories (All TDD)

#### Unit Tests (Red → Green → Refactor)
- **MCP Tool Functions**: Each tool's core logic
- **CKAN Client Methods**: API request/response handling
- **Cache Operations**: Set, get, invalidate operations
- **Error Handlers**: Network failures, API errors
- **Validators**: Input parameter validation

#### Integration Tests (End-to-End TDD)
- **CKAN API Connectivity**: Real API calls (with rate limits)
- **MCP Protocol**: Full request/response cycle
- **Cache Integration**: Redis/Memory cache behavior
- **Error Scenarios**: Timeout, 5xx errors, malformed responses

#### Test Data Strategy
- **Mock Data**: Nock interceptors for CKAN API responses
- **Real Data**: Limited integration tests with actual data.gov.il
- **Edge Cases**: Empty results, Hebrew text, large datasets
- **Error Conditions**: 404s, timeouts, invalid JSON

### TDD Development Workflow
1. **🔴 Write Test**: Create failing test for new feature
2. **🟢 Minimal Code**: Write just enough code to pass
3. **🔵 Refactor**: Clean up code while tests stay green
4. **🔁 Repeat**: For each MCP tool and component

## Sub-Agent Utilization Strategy

**🤖 Strategic Use**: Deploy sub-agents for complex, multi-step tasks that benefit from specialized focus.

### When to Use Sub-Agents

#### ✅ High-Value Sub-Agent Tasks
- **Complex Research**: Deep CKAN API behavior analysis, edge cases, Hebrew text handling
- **Multi-Component Integration**: Cache + Rate Limiter + Error Handler + MCP Protocol
- **Comprehensive Testing**: End-to-end test suites with mocks, fixtures, and integration scenarios
- **Performance Engineering**: Load testing, optimization, memory profiling across all tools
- **Quality Validation**: Full user journey testing, compatibility verification

#### ❌ Direct Implementation Tasks
- **Simple Unit Tests**: Individual function tests, basic validation
- **Single Component Code**: Basic API client methods, TypeScript interfaces
- **Straightforward Features**: Simple MCP tool implementations following existing patterns

### Sub-Agent Deployment by Phase

#### Phase 1: Foundation
- **Sub-Agent**: "Setup comprehensive TDD framework with Jest, TypeScript, mocks, and integration testing"
- **Sub-Agent**: "Implement robust CKAN API client with full error handling, retry logic, and Hebrew text support"
- **Direct**: Individual MCP tool implementations (search_datasets, get_dataset_details, etc.)

#### Phase 2: Enhancement  
- **Sub-Agent**: "Design and implement multi-level caching architecture with Redis, invalidation, and performance monitoring"
- **Sub-Agent**: "Build comprehensive performance testing suite with load testing and optimization recommendations"
- **Direct**: Tool enhancements, additional features

#### Phase 3: Advanced Features
- **Sub-Agent**: "Implement advanced analytics tools with complex data processing and visualization"
- **Sub-Agent**: "Build production monitoring and observability with metrics, logging, and alerting"

### Sub-Agent Task Characteristics

#### Use Sub-Agents When Task Involves:
- **3+ interconnected components** working together
- **Research requiring multiple rounds** of investigation and analysis  
- **Complex configuration** with multiple dependencies and edge cases
- **Cross-cutting concerns** affecting multiple parts of the system
- **Performance optimization** requiring systematic analysis and testing

#### Handle Directly When Task Is:
- **Single-purpose functions** with clear inputs/outputs
- **Following established patterns** already defined in codebase
- **Simple CRUD operations** or basic data transformations
- **Incremental additions** to existing components

### Collaboration Protocol

#### Sub-Agent Handoff Process:
1. **Define Clear Scope**: Specific deliverables and success criteria
2. **Provide Context**: Relevant architecture docs, existing code, constraints
3. **Set Expectations**: Timeline, quality standards, integration requirements
4. **Review Results**: Validate deliverables meet project standards
5. **Integration**: Merge sub-agent work into main codebase following TDD principles

#### Quality Standards for Sub-Agent Work:
- **TDD Compliance**: All code must have tests written first
- **Documentation**: Clear comments and documentation for complex logic
- **TypeScript**: Full type safety with proper interfaces
- **Error Handling**: Comprehensive error scenarios covered
- **Performance**: Meets defined performance benchmarks

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

## 🚨 Critical Lessons Learned for Seamless Implementation

### CKAN API Critical Understanding (BATTLE-TESTED)
- **Metadata vs Data Content**: `package_search` ONLY searches dataset metadata (titles, descriptions)
- **Data Content Search**: Requires `datastore_search` with specific resource_id to search actual data
- **Vehicle Data Example**: To find license "29721704", must use `datastore_search` on vehicle dataset resource
- **Resource IDs**: Each dataset has multiple resources, each with unique ID for datastore queries
- **Real Working Example**: Ministry of Transport has 841MB vehicle dataset with daily updates

### MCP Tools Implementation Gap (CRITICAL)
- **CKAN Client**: ✅ Fully implemented in `src/services/ckan-client.ts` including `queryDatastore()`
- **MCP Tool Wrapper**: ❌ Missing MCP tool to expose `queryDatastore()` functionality
- **Impact**: Cannot search actual vehicle records, only dataset descriptions
- **Solution**: Add `query_datastore` MCP tool to wrap existing CKAN client method

### Proven Real-World Data (TESTED)
```typescript
// WORKING: Search dataset metadata
search_datasets({ organization: "ministry_of_transport" })
// Returns: 82 datasets including vehicle licensing

// MISSING: Search actual vehicle data  
query_datastore({ 
  resource_id: "053cea08-09bc-40ec-8f7a-156f0677aff3",
  filters: { "license_number": "29721704" }
})
// Would return: Actual vehicle record with specs
```

### Node.js Version Issues (DEPLOYMENT BLOCKER)
- **❌ Node.js v23**: Causes consistent npm install timeouts on Windows (tested and confirmed)
- **✅ Node.js v20 LTS**: Stable, tested, works reliably with all dependencies
- **Required Action**: Always verify and enforce v20 LTS in setup documentation

### npm Installation Reliability Issues
```bash
# REQUIRED configuration before any npm install (prevents timeouts):
npm config set fetch-retry-maxtimeout 600000
npm config set fetch-retry-mintimeout 120000
npm config set fetch-timeout 600000

# Often needed: Manual installation of missing transitive dependencies
npm install picocolors @types/node
```

### TypeScript Strict Mode Configuration Gotchas
- `exactOptionalPropertyTypes: true` requires explicit undefined handling
- **Solution patterns**:
  - `this.property = undefined as undefined`
  - `apiKey?: string | undefined` in interfaces

### Real-World Success Criteria (Battle Tested)
**✅ PROVEN WORKING (Current State):**
1. `npm run build` - Compiles TypeScript without errors ✅ WORKING
2. `node dist/index.js` - Starts MCP server successfully ✅ WORKING
3. Server logs show "MCP Server initialized" and CKAN connection ✅ WORKING
4. MCP search_datasets tool functional with real data ✅ WORKING
5. Claude Desktop integration working ✅ TESTED
6. Hebrew text search fully functional ✅ TESTED
7. Real-time data.gov.il connectivity ✅ VERIFIED

**✅ ALSO WORKING:**
- npm test - Jest test framework ✅ WORKING
- npm run dev - Development server ✅ WORKING
- TypeScript strict mode compilation ✅ WORKING
- Error handling and retry logic ✅ TESTED

### Zero-Interruption Implementation Checklist
- [x] Node.js v20 LTS installed and verified (`node --version`) ✅ WORKING
- [x] npm timeout configuration applied globally ✅ CONFIGURED
- [x] All critical dependencies verified (`ls node_modules/@modelcontextprotocol`) ✅ INSTALLED
- [x] TypeScript compilation successful (`npm run build`) ✅ WORKING
- [x] MCP server startup verified (`node dist/index.js`) ✅ WORKING
- [x] Claude Desktop connection tested ✅ WORKING
- [x] Real data.gov.il queries working ✅ TESTED

### NEXT IMPLEMENTATION PRIORITIES (Ready to Code)
1. **Add query_datastore MCP Tool**: Wrapper for existing CKAN client method
2. **Add similarity_search MCP Tool**: Smart dataset discovery
3. **Add get_dataset_schema MCP Tool**: Field exploration
4. **Test with real vehicle data**: Verify license number search
5. **Test similarity search**: "cars", "banks", "insurance" queries

## Context for Future Sessions

### When You Return to This Project
1. **Read This File First** - Get project context quickly
2. **Check LESSONS-LEARNED.md** - Review all implementation gotchas and solutions
3. **Check ARCHITECTURE.md** - Review technical specifications
4. **Check IMPLEMENTATION.md** - Review detailed Phase 1 execution plan
5. **Review Git Status** - See current progress and branch
6. **Continue Implementation** - Follow implementation plan task priorities

### Common Commands
```bash
# TDD Development workflow
git status                    # Check current progress
npm test                      # Run all tests
npm run test:watch            # Watch mode for TDD
npm run test:coverage         # Code coverage report
npm run dev                   # Start development server

# TDD specific commands
npm test -- --watch           # Jest watch mode
npm test search_datasets      # Run specific test file
npm test -- --verbose         # Detailed test output

# Quick status check
ls -la                        # Project files
git branch -v                 # Current branch info
```

## Project Success Criteria

### Phase 1 Success Metrics (TDD)
- [x] **API Integration**: Successfully connects to data.gov.il CKAN API ✅ WORKING
- [x] **Error Handling**: Graceful failure handling (network, API errors) ✅ IMPLEMENTED
- [x] **Performance**: Responds to searches within 2 seconds ✅ VERIFIED
- [x] **MCP Compatibility**: Works with Claude Desktop MCP integration ✅ TESTED
- [x] **Hebrew Support**: Full Hebrew text search and display ✅ WORKING
- [x] **Real Data**: Searches 1,179+ datasets across 49 organizations ✅ VERIFIED
- [ ] **Data Content Search**: Can find specific vehicle records 🔥 MISSING - CRITICAL
- [ ] **Similarity Search**: Can find datasets by topic similarity 🔥 MISSING - REQUIRED
- [ ] **Schema Discovery**: Can explore available data fields 🔥 MISSING - REQUIRED

### Long-term Vision
- **Government Integration**: Adopted by Israeli government agencies
- **AI Accessibility**: Standard tool for AI-powered civic analysis
- **Open Source Impact**: Template for other government MCP servers
- **Data Democracy**: Citizens access government data through AI assistants

---

## Quick Start Reminder

When continuing work:
1. **Review current todo list** (if using TodoWrite tool)
2. **Read relevant sections** of ARCHITECTURE.md and IMPLEMENTATION.md
3. **Check git status** and current branch
4. **Identify current Epic/Task** from IMPLEMENTATION.md plan
5. **🔴 TDD FIRST**: Always write failing tests before implementation
6. **Follow implementation plan** priorities and sub-agent deployment strategy

The goal is to make Israeli government data easily accessible through AI assistants - keep this mission in mind with every implementation decision.

## IMMEDIATE ACTION PLAN (Ready to Start)

When starting fresh, implement in this order:

### 1. Verify Current Working State (Should Work Immediately)
```bash
# These should work out of the box
npm run build
node dist/index.js
# Test with Claude Desktop - search_datasets should work
```

### 2. Add Missing Critical Tools (High Priority)
1. **query_datastore Tool**: 
   - CKAN client method exists: `src/services/ckan-client.ts:257`
   - Need: MCP tool wrapper in `src/tools/query-datastore.ts`
   - Register in: `src/server/mcp-server.ts`

2. **similarity_search Tool**:
   - New functionality for "cars", "banks", "insurance" queries
   - Use fuzzy matching on dataset titles/descriptions

3. **get_dataset_schema Tool**:
   - Explore dataset fields for targeted search
   - Use CKAN resource_show + datastore_search preview

### 3. Test Real-World Use Cases
- Find vehicle license "29721704" in Ministry of Transport data
- Search for "insurance" datasets and get field schema  
- List all organizations with dataset counts

**The foundation is SOLID and WORKING. Focus on the missing tools!** 🚀