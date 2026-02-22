# OpenClaw Cost Optimization Analysis

**Date:** 2026-02-16
**Agents:** Flux (aransas) & Metamorph (brazos)
**Context:** Video "I Cut My OpenClaw Costs by 97%" - https://youtu.be/RX-fQTW2To8

---

## Executive Summary

Metamorph and Flux analyzed OpenClaw token optimization strategies. Key findings:
- Context files already optimized (22.5KB total)
- Local LLM for heartbeats = biggest savings opportunity
- Context caching saves ~80% on repeated context uploads
- GLM-4.7-FlashX is 8.5x cheaper than GLM-4.7 ($0.07 vs $0.60/M)
- GLM-5 available for complex tasks with 200K context

---

## Current Status

### Flux (aransas)
- Context files total: 13.7KB (SOUL, USER, AGENTS, IDENTITY, TOOLS, HEARTBEAT)
- MEMORY.md: 8.8KB
- Total context: 22.5KB (already optimized - video mentioned 50-100KB bloat as problematic)
- Model: GLM-4.7 (default)
- Heartbeats: Every 30 min
- Local LLM: None installed
- Gateway memory: 2.7Gi/28Gi stable

### Metamorph (brazos)
- Gateway memory: 580MB stable
- Heartbeats: Every 30 min
- Agent Queue Processor: Every 15 min (isolated session)
- Local LLM: koboldcpp running with Gemma 3-1B (not currently used for heartbeats)
- MEMORY.md truncated at 106 chars in injected context

---

## Key Optimization Strategies

### 1. Local LLM for Heartbeats (HIGH PRIORITY)
**Video insight:** $90/month → $0/month idle costs by routing heartbeats locally

**Implementation:**
- brazos: Route to existing koboldcpp (Gemma 3-1B)
- aransas: Install Ollama with gemma:2b
- Mechanism: Local LLM runs quick checks (disk, memory, gateway, queues)
- Escalation: Only hit API if actual work needed

**Expected savings:** 80-100% of idle heartbeat costs

---

### 2. Context Caching (CRITICAL)
**Video insight:** Saved 80% by using cache APIs to avoid re-uploading context files

**GLM pricing with caching:**
| Model | Input | Cached Input | Savings |
|-------|-------|--------------|---------|
| GLM-4.7 | $0.60 | $0.11 | ~82% |
| GLM-5 | $1.00 | $0.20 | ~80% |
| GLM-4.7-FlashX | $0.07 | $0.01 | ~86% |

**Action:** Investigate whether OpenClaw enables context caching automatically or requires configuration

---

### 3. Model Routing Policy

**Model Comparison (per 1M tokens):**
| Model | Input | Output | Best For |
|-------|-------|--------|----------|
| **Local LLM** | FREE | FREE | Heartbeats, quick system checks |
| **GLM-4.7-FlashX** | $0.07 | $0.40 | Simple parsing, basic tasks |
| **GLM-4.7** | $0.60 | $2.20 | Standard daily operations |
| **GLM-5** | $1.00 | $3.20 | Complex research, long-horizon tasks |

**GLM-5 Advantages:**
- 200K context (vs 128K on GLM-4.7)
- 128K max output
- Coding performance on par with Claude Opus 4.5
- SOTA in open-weight models for agentic capabilities
- Built-in context caching

**Proposed Escalation Hierarchy:**
1. **Heartbeat checks** → Local LLM
2. **Simple tasks** (status checks, basic parsing) → GLM-4.7-FlashX
3. **Standard operations** (daily tasks, coordination) → **GLM-4.7 (95% of work)**
4. **Emergency escalation** → GLM-5 (edge cases only)

### When to Use GLM-5 (Emergency Escalation Only)

**GLM-5 is 67% more expensive** than GLM-4.7 ($1.00 vs $0.60/M input). Given our current workflow, we will rarely use it.

**Use GLM-5 only for:**
- **Context overflow** - When we exceed 128K context (our current total is 22.5KB, so this is rare)
- **Long-horizon agent tasks** - Complex multi-step planning that needs to maintain coherence over many operations
- **Deep coding/refactoring** - Autonomous debugging, backend refactoring with minimal human intervention
- **Exceptional tasks** - When GLM-4.7 repeatedly fails at a specific task

**Do NOT use GLM-5 for:**
- Document collaboration (GLM-4.7 handles this)
- Inter-agent coordination (GLM-4.7 handles this)
- Research synthesis (GLM-4.7 handles this)
- Standard daily operations

**Rationale:**
- GLM-5 is overkill for our current workflow
- GLM-4.7 fits 95% of our needs at 40% lower cost
- GLM-5 is an emergency tool, not a daily workhorse

---

### 4. Agent Queue Processor Model Selection

**OPEN QUESTION:** Should this use cheaper model or capable model?

**Arguments for cheap model (GLM-4.7-FlashX):**
- Simple message parsing doesn't need full capabilities
- Most inter-agent messages are basic coordination
- 8.5x cost savings

**Arguments for capable model (GLM-4.7):**
- Handles "research" messages - could be actual research tasks
- Handles "collaborate" messages - joint work on shared projects
- Consistency with current setup
- Message types: research, status, collaborate, info

**Recommendation:** Use GLM-4.7 for Agent Queue Processor (not cheapest) due to potential research workload

---

### 5. Session History Management

**Video insight:** Session history was being uploaded every time (111KB), causing waste

**Investigation needed:**
- Does OpenClaw have session-history-clear mechanism?
- Are we uploading full history on every request?
- Strategy for clearing before long-running tasks

---

## Priority Order (Agreed by Flux & Metamorph)

1. **IMMEDIATE:** Install Ollama on aransas (gemma:2b)
2. **Route heartbeats to local LLM** on both machines (test)
3. **Define Agent Queue Processor model** (GLM-4.7 recommended due to research workload)
4. **Route Agent Queue Processor to appropriate model**
5. **Define escalation rules document** (shared in /shared/ folder)
6. **Investigate session history clearing** (lower priority)
7. **Test overnight** (measure actual vs. predicted token usage)

---

## Questions for Wilywit

1. **Install Ollama on aransas NOW?** (Priority 1)
2. **Monthly cost target?** (Video: $90 → $6, we're likely lower already)
3. **Local model preference?** (gemma:2b suggested, other options?)
4. **Agent Queue Processor model?** (GLM-4.7 recommended due to research workload)
5. **Context caching:** Auto-enabled or manual configuration needed?

---

## Expected Outcomes

**Before optimization (estimated):**
- Heartbeats: ~30 min intervals, hitting paid API
- Agent Queue Processor: GLM-4.7 on all messages
- Context uploaded on every message (no caching)
- Session history uploaded repeatedly

**After optimization (projected):**
- Heartbeats: Local LLM (FREE), only escalate if work needed
- Agent Queue Processor: GLM-4.7 for research, cheaper model for simple tasks
- Context: Cached uploads (~80% savings)
- Session history: Managed strategically

**Cost reduction target:** 70-90% (video achieved 97%, starting from higher baseline)

---

## Next Steps

1. Wilywit approval on priorities and Ollama installation
2. Install Ollama on aransas with gemma:2b
3. Test heartbeat routing to local LLM
4. Create escalation rules document in /shared/
5. Define Agent Queue Processor model strategy (research vs. simple parsing)
6. Overnight test to measure actual token savings

---

*Analysis by Flux (aransas) & Metamorph (brazos)*
*Date: 2026-02-16*
*Status: Awaiting Wilywit approval to proceed*
