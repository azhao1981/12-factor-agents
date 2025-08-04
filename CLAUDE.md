# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the "12-Factor Agents" project - a comprehensive guide and educational resource for building reliable LLM-powered applications following 12 key principles. The project includes documentation, hands-on workshops, and code examples demonstrating how to build production-ready AI agents without relying on heavy frameworks.

## Architecture

### Core Structure
- **Content** (`content/`): Individual markdown files for each of the 12 factors, plus supplementary materials
- **Workshops** (`workshops/`): Hands-on coding workshops demonstrating the 12 factors in practice
  - `2025-05/`: Full workshop with progressive sections (00-12) covering basic to advanced agent concepts
  - `2025-05-17/`: Similar workshop structure with some variations
  - `2025-07-16/`: Python-focused workshop with enhanced features
- **Packages** (`packages/`): Supporting tools and utilities for the project
- **Hack** (`hack/`): Development utilities and contributor tools

### Workshop Architecture
Each workshop follows a progressive learning approach:
- **Section 00**: Basic hello world setup
- **Section 01-02**: CLI and agent fundamentals
- **Section 03-04**: Tool loops and BAML testing
- **Section 05-07**: Human interaction, prompts, and context management
- **Section 08-09**: API endpoints and state management
- **Section 10-12**: Human approval workflows and webhooks

### Technology Stack
- **Primary**: TypeScript with Node.js runtime
- **BAML**: BoundaryML for schema-aligned LLM interactions and tool definitions
- **Express**: Web server framework for API endpoints
- **HumanLayer**: Human-in-the-loop approval workflows
- **tsx**: TypeScript execution environment
- **Testing**: Jest for unit tests, BAML for LLM behavior testing

## Common Commands

### Project Setup
```bash
# Install dependencies (works with npm, bun, or yarn)
make setup
# or manually:
npm install || bun install || yarn install
```

### Development Workshops
```bash
# Navigate to any workshop section, e.g.:
cd workshops/2025-05/final
npm run dev          # Run with tsx
npm run build        # Compile TypeScript

cd workshops/2025-05/sections/03-tool-loop
npm run dev          # Run specific workshop section
```

### Testing
```bash
# Run BAML tests (in workshop directories)
baml test

# Run Jest tests (in packages/walkthroughgen)
npm test
npm run test:watch
```

### Linting
```bash
# ESLint (available in workshop directories)
npm run lint         # if configured in package.json
npx eslint .        # manual linting
```

## Key Patterns

### Agent Loop Pattern
All workshops demonstrate the core agent loop pattern found in `workshops/2025-05/final/src/agent.ts:84`:
1. Serialize thread context for LLM
2. Determine next step using BAML schema
3. Execute tool calls (calculator, human interaction, etc.)
4. Append results to thread
5. Repeat until completion or human input required

### Event-Driven Architecture
Agents use an event-driven approach with:
- `Thread` class managing conversation state
- `Event` interface for structured interactions
- XML-style serialization for LLM context
- Human approval workflows for sensitive operations

### BAML Schema Patterns
Tool definitions follow consistent patterns:
- `intent` field for tool identification
- Structured input/output validation
- Built-in testing with `@@assert` statements
- Multi-client support (OpenAI GPT-4o primary)

## Development Guidelines

### Working with Workshops
- Each section builds upon previous concepts
- Start with section 00 for basic setup
- Progress through sections sequentially
- Refer to corresponding factor documentation in `content/`

### Code Organization
- Keep agents small and focused (Factor 10)
- Unify execution and business state (Factor 5)
- Use structured outputs for tools (Factor 4)
- Implement human approval workflows (Factor 7)

### Testing Approach
- BAML tests for LLM behavior validation
- Jest tests for deterministic code
- Workshop sections include progressive test coverage
- Focus on agent behavior rather than implementation details

## Important Files

### Configuration
- `Makefile`: Basic setup/teardown commands
- `README.md`: Comprehensive project overview and factor explanations
- Individual workshop `package.json`: Scripts and dependencies

### Core Workshop Examples
- `workshops/2025-05/final/src/agent.ts`: Complete agent implementation
- `workshops/2025-05/final/baml_src/agent.baml`: BAML schema and prompts
- `workshops/2025-05/sections/*/README.md`: Section-specific guidance

### Documentation
- `content/factor-*.md`: Detailed explanations of each factor
- `workshops/*/walkthrough.md`: Step-by-step workshop guides
- `img/`: Diagrams and visual aids for concepts