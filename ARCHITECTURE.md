# OpenClaw Hub Architecture

## System Overview
```
┌─────────────────────────────────────────────────────┐
│                    OpenClaw                         │
│                  (MCP Client)                       │
└────────────────────┬────────────────────────────────┘
                     │ MCP Protocol
                     │ (stdio/SSE)
┌────────────────────▼────────────────────────────────┐
│              OpenClaw Hub MCP Server                │
│                                                     │
│  ┌─────────────────────────────────────────────┐    │
│  │         Core Layer                          │    │
│  │  - Credential Manager                       │    │
│  │  - Request Router                           │    │
│  │  - Audit Logger                             │    │
│  │  - Config Manager                           │    │
│  └─────────────────────────────────────────────┘    │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │  Media   │  │   Git    │  │   Data   │           │
│  │  Domain  │  │  Domain  │  │  Domain  │           │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘           │
└───────┼─────────────┼─────────────┼─────────────────┘
        │             │             │
        ▼             ▼             ▼
   ┌────────┐    ┌────────┐    ┌─────────┐
   │ Sora   │    │ GitHub │    │ Google  │
   │ Kie.ai │    │ GitLab │    │ Drive   │
   │ Luma   │    └────────┘    └─────────┘
   └────────┘
```

## Core Components

### Credential Manager
**Purpose:** Centralized, secure API key management
**Location:** `src/core/auth.py`
**Responsibilities:**
- Load credentials from environment
- Provide authenticated clients to domains
- Audit all credential access
- Support credential rotation

### Domain Architecture
Each domain follows consistent pattern:
```
domains/{domain_name}/
├── __init__.py
├── providers/          # External service integrations
│   ├── provider_a.py
│   └── provider_b.py
├── orchestrator.py     # Smart routing logic
└── tools.py            # MCP tool definitions
```

## Design Decisions

### Why Python over TypeScript?
- OpenClaw users more likely familiar with Python
- Better AI/ML library ecosystem (for routing logic)
- Anthropic's MCP Python SDK is mature
- Easier credential/environment management

### Why Single Server vs Multiple?
- Reduces OpenClaw connection overhead
- Enables cross-domain orchestration
- Shared credential management
- Simpler mental model for users

### Why YAML Config?
- Human-readable
- Easy to document
- Standard for infrastructure tools
- Can validate with schemas

## Security Architecture

### Credential Isolation
- API keys NEVER in git
- Environment variables only
- `.env.example` for documentation
- Audit log of all credential access

### Network Security
- Works in double-NAT environments
- No external requirements beyond API calls
- Configurable timeouts/retries
- Rate limiting per provider

## Data Flow Example
```
User: "Generate a video about AI existentialism"
  ↓
OpenClaw calls: hub.media.generate_video(prompt="...", priority="auto")
  ↓
OpenClaw Hub receives request
  ↓
Routing Logic:
  - Analyzes prompt complexity
  - Checks budget constraints
  - Reviews provider availability
  - Decides: Kie.ai (cost-effective, sufficient quality)
  ↓
Media Domain:
  - Retrieves Kie.ai credentials from CredentialManager
  - Calls Kie.ai API
  - Handles errors/retries
  ↓
Audit Logger:
  - Records: timestamp, provider, cost, prompt (hashed)
  ↓
Returns result to OpenClaw with metadata:
  {
    "video_url": "...",
    "provider_used": "kie",
    "cost": 0.24,
    "reasoning": "Chosen for cost optimization"
  }
```

## Extension Points

### Adding a New Provider
1. Create `domains/media/providers/new_provider.py`
2. Implement `BaseProvider` interface
3. Add to orchestrator routing logic
4. Update configuration schema
5. Add tests

### Adding a New Domain
1. Create `domains/new_domain/` directory
2. Implement domain-specific tools
3. Register in main server
4. Document capabilities
5. Add examples

## Performance Considerations

### Token Efficiency
- Capability discovery reduces OpenClaw memory needs
- Self-documenting tools minimize prompt engineering
- Efficient error messages

### Cost Optimization
- Smart routing reduces unnecessary premium API calls
- Budget controls prevent overruns
- Analytics track spending patterns

## Future Enhancements
- Database persistence for analytics
- Web UI for monitoring
- Advanced ML routing models
- Plugin system for community providers
