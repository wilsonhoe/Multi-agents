# Multi-Agent Workflow System Documentation

## Overview

The Multi-Agent System is a distributed AI architecture where specialized agents (Ubuntu, Lisa, Nyx, Kael) collaborate to execute complex tasks across multiple platforms (Discord, Telegram, CLI). The system uses session handoff, persistent memory, and bridge communication to enable seamless agent collaboration.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         MULTI-AGENT SYSTEM                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐          │
│  │  Ubuntu  │◄──►│   Lisa   │◄──►│   Nyx    │◄──►│   Kael   │          │
│  │ (Leader) │    │ (Discord)│    │ (Research)│   │ (Code)   │          │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘    └────┬─────┘          │
│       │               │               │               │               │
│       └───────────────┴───────────────┴───────────────┘               │
│                         │                                               │
│                    ┌────┴────┐                                          │
│                    │ Handoff │                                          │
│                    │   DB    │                                          │
│                    └────┬────┘                                          │
│                         │                                               │
│       ┌─────────────────┼─────────────────┐                          │
│       │                 │                 │                          │
│  ┌────┴────┐      ┌────┴────┐      ┌────┴────┐                         │
│  │ Discord │      │Telegram │      │ GitHub  │                         │
│  │  Bot    │      │  Bot    │      │  Repos  │                         │
│  └─────────┘      └─────────┘      └─────────┘                         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Agent Roles & Responsibilities

### Ubuntu (Coordinator)
- **Primary Role**: System orchestrator and primary user interface
- **Platforms**: CLI, Telegram (@wilsonhoe_claude_bot)
- **Responsibilities**:
  - Task delegation to specialized agents
  - Session state management
  - Cross-agent communication coordination
  - Tool execution and environment control
  - Final output delivery to user

### Lisa (Communication Specialist)
- **Primary Role**: Discord community management and communication
- **Platforms**: Discord (Lisa#1234)
- **Responsibilities**:
  - Discord server management
  - Multi-channel message coordination
  - Bridge message processing (LISA_TO_CLAUDE.md)
  - Community engagement and responses
  - Standup coordination and reporting

### Nyx (Research Specialist)
- **Primary Role**: Information gathering and analysis
- **Platforms**: Web, APIs, external data sources
- **Responsibilities**:
  - Bounty platform monitoring
  - Security research and analysis
  - Data collection and synthesis
  - Trend analysis and reporting

### Kael (Development Specialist)
- **Primary Role**: Code generation and technical implementation
- **Platforms**: Development environments, GitHub
- **Responsibilities**:
  - Feature implementation
  - Code review and quality assurance
  - Technical documentation
  - Build and deployment automation

---

## Communication Flow

### 1. Session Handoff Protocol

```
┌─────────┐     ┌──────────────┐     ┌─────────┐
│ Agent A │────►│ Handoff DB   │────►│ Agent B │
└─────────┘     └──────────────┘     └─────────┘
                     │
              ┌──────┴──────┐
              │  Sessions   │
              │   Memory    │
              │   State     │
              └─────────────┘
```

**Mechanism**:
- SQLite database at `~/.claude/session-handoff/handoff.db`
- Stores session context, memory pointers, and task state
- Enables asynchronous agent handoffs
- Preserves context across agent switches

### 2. Bridge Communication (Discord Integration)

**File-Based Bridge**:
```
/home/wls/bridge/
├── LISA_TO_CLAUDE.md    # Lisa messages to Ubuntu
├── CLAUDE_TO_LISA.md    # Ubuntu responses to Lisa
└── bridge.lock          # Concurrency control
```

**Protocol**:
1. Lisa writes to `LISA_TO_CLAUDE.md`
2. Ubuntu monitors and processes
3. Ubuntu responds via `CLAUDE_TO_LISA.md`
4. Lisa reads and deletes processed messages

### 3. Telegram Bot Architecture

**TypeScript Bot (Grammy Framework)**:
```
User Message → Grammy Bot → Claude CLI (child_process)
                                │
                                ▼
                         Stream JSON Output
                                │
                                ▼
                    Edit/Send Telegram Message
```

**Key Features**:
- Per-chat session persistence (SQLite)
- Streaming response updates
- File attachment support
- `--no-chrome` flag prevents browser exhaustion

---

## Data Flow Diagram

```
┌──────────┐    ┌──────────────┐    ┌──────────┐
│   User   │───►│    Ubuntu    │───►│  Task    │
└──────────┘    └──────────────┘    └────┬─────┘
                                         │
                    ┌────────────────────┼────────────────────┐
                    │                    │                    │
                    ▼                    ▼                    ▼
              ┌──────────┐       ┌──────────┐       ┌──────────┐
              │   Lisa   │       │   Nyx    │       │   Kael   │
              │(Discord) │       │(Research)│       │  (Code)  │
              └────┬─────┘       └────┬─────┘       └────┬─────┘
                   │                   │                   │
                   ▼                   ▼                   ▼
              ┌──────────┐       ┌──────────┐       ┌──────────┐
              │ Discord  │       │  Web/API │       │  GitHub  │
              │  Server  │       │  Sources │       │   Repos  │
              └──────────┘       └──────────┘       └──────────┘
```

---

## SWOT Analysis

### Strengths

| Strength | Description |
|----------|-------------|
| **Specialization** | Each agent has a dedicated role (coordination, communication, research, development) |
| **Persistence** | SQLite session handoff enables stateful cross-agent collaboration |
| **Platform Coverage** | Discord, Telegram, CLI, and GitHub integration |
| **Scalability** | Easy to add new agents with specific skill sets |
| **Redundancy** | Multiple agents can handle failover scenarios |
| **Modularity** | Independent components reduce single points of failure |

### Weaknesses

| Weakness | Description | Mitigation |
|----------|-------------|------------|
| **Token Conflicts** | Multiple processes using same bot token cause 409 errors | Use distinct tokens per service |
| **Context Overflow** | Long sessions can hit token limits (60K threshold) | Auto-compaction and session rotation |
| **Duplicate Processes** | Systemd Restart=always can spawn zombie processes | Use Restart=on-failure with PID checks |
| **Bridge Latency** | File-based bridge has polling delays | Consider message queue (Redis/RabbitMQ) |
| **Chrome Exhaustion** | Each session spawns browser processes | Use `--no-chrome` flag |
| **Manual Handoff** | Agent switching requires explicit coordination | Implement automated load balancing |

### Opportunities

| Opportunity | Description |
|-------------|-------------|
| **Auto-scaling** | Dynamic agent spawning based on workload |
| **Memory Integration** | MemPalace knowledge graph for persistent learning |
| **Workflow Templates** | Predefined multi-agent workflows for common tasks |
| **Ecosystem Growth** | Open-source agent marketplace |
| **Cross-platform** | WhatsApp, Slack, Matrix integration |
| **Bounty Automation** | Self-improving bounty hunting pipelines |

### Threats

| Threat | Description | Mitigation |
|--------|-------------|------------|
| **Rate Limiting** | Discord/Telegram API limits | Implement exponential backoff |
| **Cost Escalation** | Multiple Claude sessions = higher API costs | Budget tracking per agent |
| **Security Leaks** | Agent session logs may contain secrets | Sanitize logs, rotate secrets |
| **Hallucination** | Agents may report false completion | Grounding framework with evidence |
| **Dependency Failures** | MCP server or bridge file issues | Health checks and auto-restart |
| **Token Exhaustion** | Context window overflow | Aggressive compaction at 50% |

---

## Best Practices

### 1. Session Management

```bash
# Check active sessions
ps aux | grep "claude -p" | grep -v grep

# Kill stuck sessions
pkill -f "claude -p"

# Monitor token usage
tail -f ~/.claude/projects/*/sessions.log
```

### 2. Bot Deployment

```ini
# systemd service - AVOID Restart=always
[Service]
Type=simple
ExecStart=/usr/bin/npm start
Restart=on-failure
RestartSec=10
```

### 3. Bridge Monitoring

```bash
# Watch bridge files
watch -n 2 "ls -la /home/wls/bridge/"

# Monitor Discord bot logs
journalctl -u discord-bot -f
```

### 4. Context Management

```typescript
// Trigger compaction at 50% threshold
if (contextUsage > 0.5) {
  await compactSession();
}
```

---

## Configuration Reference

### Environment Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `TELEGRAM_BOT_TOKEN` | Telegram bot authentication | `123456:ABC-DEF...` |
| `ALLOWED_TELEGRAM_USERS` | User whitelist | `12345678,87654321` |
| `DEFAULT_CWD` | Default working directory | `/home/wls/projects` |
| `DEFAULT_MODEL` | Claude model selection | `claude-sonnet-4-6` |
| `THREADS_DB_PATH` | Session persistence | `./data/sessions.db` |
| `MAX_STREAM_MS` | Timeout for long operations | `5400000` (90 min) |

### File Locations

| Component | Path |
|-----------|------|
| Session Handoff DB | `~/.claude/session-handoff/handoff.db` |
| Bridge Directory | `/home/wls/bridge/` |
| Agent Definitions | `~/.claude/agents/` |
| Telegram Bot | `/home/wls/telegram-claude-bot/` |
| Discord Bot | `/home/wls/discord-claude-code-bot/` |

---

## Troubleshooting Guide

### Problem: Bot not responding

```bash
# 1. Check if process is running
ps aux | grep telegram-claude-bot

# 2. Check for 409 conflicts (duplicate bots)
grep "409" /home/wls/telegram-claude-bot/bot.log

# 3. Verify token is not shared
# Check MCP plugin is disabled if using TypeScript bot

# 4. Kill and restart
pkill -f telegram-claude-bot
npm start
```

### Problem: Duplicate responses

```bash
# Check for multiple processes
ps aux | grep "discord.*bot" | wc -l

# If > 1, kill all and restart
pkill -f discord-claude-code-bot
systemctl restart discord-bot
```

### Problem: Session context overflow

```bash
# Clear session files
rm -rf ~/.claude/projects/*/.session*

# Restart agent
# Or use /compact command
```

---

## Metrics & Monitoring

### Key Performance Indicators

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Response Time | < 5s | > 30s |
| Session Uptime | > 99% | < 95% |
| Token Usage | < 70% | > 85% |
| Error Rate | < 1% | > 5% |
| Cost per Task | Track | +50% baseline |

### Cost Tracking

```
[telegram] session abc123 cost: $0.4324 exit: 0
[discord] session def456 cost: $1.2345 exit: 0
```

---

## Future Roadmap

### Phase 1: Stability (Current)
- ✅ Session handoff working
- ✅ Bridge communication established
- ✅ Multi-platform deployment

### Phase 2: Intelligence
- 🔄 Auto-agent selection based on task type
- 🔄 Self-healing (auto-restart failed agents)
- 🔄 Predictive context compaction

### Phase 3: Scale
- ⏳ Kubernetes deployment
- ⏳ Horizontal agent scaling
- ⏳ Load balancing across agent pools

### Phase 4: Ecosystem
- ⏳ Agent marketplace
- ⏳ Community-contributed skills
- ⏳ Inter-agent learning

---

## Conclusion

The Multi-Agent System provides a robust foundation for distributed AI workflows. By understanding the architecture, following best practices, and monitoring key metrics, the system can scale to handle complex multi-step tasks while maintaining reliability and cost efficiency.

**Last Updated**: 2026-04-25  
**Version**: 1.0  
**Maintainers**: Ubuntu (Lead), Lisa (Discord), Nyx (Research), Kael (Code)
