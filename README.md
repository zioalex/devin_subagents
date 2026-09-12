# Devin Custom Subagents Setup

This repository contains a working example of custom subagents for Devin CLI, demonstrating how to create and use specialized agents for software development workflows.

## Overview

We created a 3-agent system that coordinates requirements analysis and implementation planning:

- **requirements-discuss**: Analyzes and clarifies project requirements
- **implementation-schedule**: Creates detailed implementation plans  
- **orchestrator**: Coordinates between the two specialist agents

## The Journey: Making Custom Subagents Work

### Initial Setup Challenges

When we first created the custom subagents, they failed to start. The built-in `subagent_explore` worked perfectly, but our custom agents (`orchestrator`, `requirements-discuss`, `implementation-schedule`) consistently failed with "Subagent failed to start" errors.

### Troubleshooting Process

1. **Verified built-in subagents work**: Tested `subagent_explore` - worked successfully
2. **Checked configuration files**: Verified YAML frontmatter was correct
3. **Removed model field**: Initially had `model: sonnet` in frontmatter - removed this
4. **Tried multiple locations**: Placed files in both `.devin/agents/` and `.agents/agents/`
5. **Session persistence**: The agents eventually started working after some time

### Key Configuration Changes

**Initial (failing) configuration:**
```yaml
---
name: orchestrator
description: Coordinates between requirements-discuss and implementation-schedule agents
model: sonnet  # This may have caused issues
allowed-tools:
  - read
  - grep
  - glob
  - run_subagent
  - read_subagent
max-nesting: 2
---
```

**Working configuration:**
```yaml
---
name: orchestrator
description: Coordinates between requirements-discuss and implementation-schedule agents
allowed-tools:
  - read
  - grep
  - glob
  - run_subagent
  - read_subagent
max-nesting: 2
---
```

### Final Working Setup

The custom subagents eventually started working without clear indication of what changed. Possible factors:
- System recognition delay for custom agents
- Session restart/configuration reload
- Removal of explicit model field

## Agent Specifications

### 1. Requirements Discussion Agent

**File**: `requirements-discuss.md`

**Purpose**: Analyzes and clarifies project requirements with detailed analysis

**Tools**: Read-only (read, grep, glob, web_search)

**Focus Areas**:
- Understanding the core problem being solved
- Identifying dependencies and integration points
- Considering performance, security, and scalability implications
- Highlighting potential technical challenges

**Use When**: Request is ambiguous, missing details, or involves complex user needs

### 2. Implementation Schedule Agent

**File**: `implementation-schedule.md`

**Purpose**: Plans and schedules implementation tasks with detailed breakdown

**Tools**: Read-only + todo_write (for task planning)

**Focus Areas**:
- Creating clear, sequential implementation steps
- Identifying which files need to be modified
- Noting any refactoring or prerequisite work
- Considering testing and verification steps

**Use When**: Requirements are clear and you need a concrete implementation plan

### 3. Orchestrator Agent

**File**: `orchestrator.md`

**Purpose**: Coordinates between requirements-discuss and implementation-schedule agents

**Tools**: Read-only + run_subagent + read_subagent (can spawn other subagents)

**Special Configuration**: `max-nesting: 2` - allows it to spawn the other two agents

**Decision Logic**:
- Use requirements-discuss when: request is ambiguous, missing details, or involves complex user needs
- Use implementation-schedule when: requirements are clear and you need a concrete implementation plan
- Use both when: complex feature that needs both requirements analysis and implementation planning

**Context Passing Principles**:
- Be selective: only pass information directly relevant to the subagent's specific task
- Keep it concise: summarize key points rather than passing entire conversation logs
- Focus on essentials: original request, specific constraints, relevant files/components
- Avoid overload: don't pass unnecessary history or tangential details

## How to Use

### Direct Invocation

```bash
# Use specific agent
"Use the orchestrator agent to handle this: [task]"
"Use the requirements-discuss agent to analyze: [request]"
"Use the implementation-schedule agent to plan: [feature]"
```

### Let Agent Decide

```bash
# Let agent choose appropriate agent
"Help me analyze the requirements for: [feature]"
"Create an implementation plan for: [task]"
```

## Example Usage

### Complex Feature Request

```bash
"Use the orchestrator agent to handle this: Add user authentication to my web application"
```

**Result**: Orchestrator determines requirements need clarification, calls requirements-discuss agent, then calls implementation-schedule agent, synthesizes both results.

### Clear Implementation Task

```bash
"Use the orchestrator to create an implementation plan for: Add dark mode toggle to the settings page"
```

**Result**: Orchestrator may skip requirements analysis and go directly to implementation-schedule agent.

## Tested Example

We successfully tested the system with a meal planning application request:

**Request**: "Use the orchestrator agent to handle this: Create a program to plan what to cook this week"

**Result**: The orchestrator successfully:
1. Recognized this as a software development task
2. Coordinated requirements analysis (features, architecture, technology stack)
3. Coordinated implementation planning (timeline, phases, file structure)
4. Synthesized results into a comprehensive development plan

## Key Learnings

### Context Inheritance

- Subagents do NOT inherit parent's conversation history
- Only receive what parent explicitly provides in the task prompt
- Orchestrator must manually pass relevant context between agents
- Selective context passing is crucial to avoid information overload

### Tool Restrictions

- Each subagent has specific `allowed-tools` list
- `ask_user_question` tool is always withheld from subagents
- Background subagents can only use pre-approved tools
- Custom subagents can have restricted tool sets for focus

### Benefits Observed

- **Focused Work**: Each agent specializes in specific tasks
- **Clear Responsibilities**: Separation of concerns makes system maintainable
- **Intelligent Coordination**: Orchestrator automatically decides workflow
- **Modular Design**: Agents can be used independently or together

### Limitations Encountered

- **Setup Complexity**: Initial configuration required troubleshooting
- **Context Management**: Requires careful design of what information to pass
- **No Direct Interaction**: You don't see subagent's raw output directly
- **System Recognition**: Custom agents may need time to be recognized

## Installation

1. Copy the agent files to your project:
   ```bash
   mkdir -p .devin/agents
   cp agents/*.md .devin/agents/
   ```

2. Alternative location (also supported):
   ```bash
   mkdir -p .agents/agents
   cp agents/*.md .agents/agents/
   ```

3. Restart your Devin session if agents aren't recognized immediately

## Troubleshooting

If custom subagents fail to start:

1. **Verify built-in subagents work**: Test with `subagent_explore`
2. **Check YAML syntax**: Ensure frontmatter is properly formatted
3. **Remove model field**: Try without explicit model specification
4. **Check tool permissions**: Ensure `allowed-tools` are valid
5. **Wait for recognition**: System may need time to process custom agents
6. **Restart session**: Try restarting your Devin CLI session

## Future Improvements

- Add more specialized agents (code reviewer, test generator, documentation writer)
- Implement context passing templates for common workflows
- Add error handling and retry logic for failed subagent calls
- Create agent performance monitoring and optimization
- Develop agent communication protocols for complex workflows

## Contributing

This is a working example of Devin custom subagents. Feel free to:
- Add your own custom agent configurations
- Improve existing agent prompts
- Share your use cases and experiences
- Report issues or suggest improvements

## License

This example configuration is provided as-is for educational purposes.
