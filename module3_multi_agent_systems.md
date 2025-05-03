# Module 3: Multi-Agent Systems

## Task 1: Building Multi-Agent Systems: Agent-to-Agent Communication

### A `call_agent` Tool

In a multi-agent setup, it's important to let specialized agents delegate tasks to one another. The `call_agent` tool enables this by allowing one agent to invoke another's logic cleanly.

```python
@register_tool()
def call_agent(action_context: ActionContext, agent_name: str, task: str) -> dict:
    agent_registry = action_context.get_agent_registry()
    if not agent_registry:
        raise ValueError("No agent registry found in context")

    agent_run = agent_registry.get_agent(agent_name)
    if not agent_run:
        raise ValueError(f"Agent '{agent_name}' not found in registry")

    invoked_memory = Memory()
    try:
        result_memory = agent_run(
            user_input=task,
            memory=invoked_memory,
            action_context_props={
                'auth_token': action_context.get('auth_token'),
                'user_config': action_context.get('user_config'),
            }
        )
        if result_memory.items:
            last_memory = result_memory.items[-1]
            return {
                "success": True,
                "agent": agent_name,
                "result": last_memory.get("content", "No result content")
            }
        else:
            return {"success": False, "error": "Agent failed to run."}
    except Exception as e:
        return {"success": False, "error": str(e)}
```

### Example: Project Manager + Scheduler Agent Collaboration

**Scheduler Agent Goals:**

```python
scheduler_agent = Agent(
    goals=[
        Goal(
            name="schedule_meetings",
            description="""Schedule meetings efficiently by:
            1. Finding times that work for all attendees
            2. Creating and sending calendar invites
            3. Handling any scheduling conflicts"""
        )
    ],
    ...
)
```

**Tools for Scheduling Agent:**

```python
@register_tool()
def check_availability(...):
    return calendar_service.find_available_slots(...)

@register_tool()
def create_calendar_invite(...):
    return calendar_service.create_event(...)
```

**Project Management Agent:**

```python
project_manager = Agent(
    goals=[
        Goal(
            name="project_oversight",
            description="""Manage project progress by:
            1. Getting the current project status
            2. Identifying when meetings are needed
            3. Delegating meeting scheduling to 'scheduler_agent'
            4. Logging project decisions"""
        )
    ],
    ...
)
```

**Shared Tools:**

```python
@register_tool()
def get_project_status(...):
    return project_service.get_status(...)

@register_tool()
def update_project_log(...):
    return project_service.log_update(...)
```

### Agent Registry Setup

```python
class AgentRegistry:
    def __init__(self):
        self.agents = {}

    def register_agent(self, name: str, run_function: callable):
        self.agents[name] = run_function

    def get_agent(self, name: str) -> callable:
        return self.agents.get(name)

registry = AgentRegistry()
registry.register_agent("scheduler_agent", scheduler_agent.run)

action_context = ActionContext({
    'agent_registry': registry,
})
```

### Benefits of `call_agent`

* **Memory Isolation**: Each call has clean memory context.
* **Context Management**: Limited, safe context sharing.
* **Structured Result Handling**: Final memory result is returned cleanly.

This architectural pattern allows agents to focus on their core expertise while collaborating smoothly with others.

---

## Task 2: Agent Interaction & Memory

Effective multi-agent systems rely on structured communication and memory sharing strategies. Memory plays a crucial role in preserving context across interactions and enabling intelligent behavior over time.

### Types of Memory in Agents:

* **Short-Term Memory**: Stores temporary results or context within a session.
* **Long-Term Memory**: Persists over time, allowing agents to recall previous decisions, preferences, or interactions.
* **Shared Memory**: A common knowledge base accessible to multiple agents.

### Memory-Driven Collaboration

Agents often interact by writing to and reading from shared memory or passing memory snapshots. This promotes:

* Coordinated execution
* Result validation and feedback loops
* Reduced redundancy in agent behavior

### Example: Collaborative Memory Sharing

```python
# Research agent writes summary to memory
research_result = research_agent.run(user_input="Summarize this article")
shared_memory.store("summary", research_result.last_item())

# Reviewer agent reads from memory
review_input = shared_memory.get("summary")
review_result = reviewer_agent.run(user_input=f"Please review: {review_input}")
```

### Benefits

* Promotes reusable reasoning chains
* Ensures consistent context transfer
* Enables debugging and explainability

This task demonstrates how thoughtful memory design enhances the effectiveness and reliability of multi-agent workflows.

---

## Task 3: Memory Interaction Patterns in Multi-Agent Systems

How agents share and manage memory directly impacts their ability to collaborate effectively. Here are three core patterns:

### 1. Message Passing: The Basic Pattern

A simple request-response flow. One agent invokes another and receives only the final result.

```python
@register_tool()
def call_agent(action_context: ActionContext, agent_name: str, task: str) -> dict:
    agent_registry = action_context.get_agent_registry()
    agent_run = agent_registry.get_agent(agent_name)
    invoked_memory = Memory()
    result_memory = agent_run(user_input=task, memory=invoked_memory)
    return {
        "result": result_memory.items[-1].get("content", "No result")
    }
```

Useful when the first agent doesn’t need to understand *how* the result was generated—just the outcome.

---

### 2. Memory Reflection: Learning from the Process

Here, the calling agent retrieves all memories from the invoked agent to understand its reasoning.

```python
@register_tool()
def call_agent_with_reflection(action_context: ActionContext, agent_name: str, task: str) -> dict:
    agent_registry = action_context.get_agent_registry()
    agent_run = agent_registry.get_agent(agent_name)
    invoked_memory = Memory()
    result_memory = agent_run(user_input=task, memory=invoked_memory)
    caller_memory = action_context.get_memory()
    for memory_item in result_memory.items:
        caller_memory.add_memory({
            "type": f"{agent_name}_thought",
            "content": memory_item["content"]
        })
    return {
        "result": result_memory.items[-1].get("content", "No result"),
        "memories_added": len(result_memory.items)
    }
```

Ideal for tasks where transparency or traceability is important, such as auditing or knowledge transfer.

---

### 3. Memory Handoff: Continuing the Conversation

Passes the entire memory context to the next agent so they can continue the task seamlessly.

```python
@register_tool()
def hand_off_to_agent(action_context: ActionContext, agent_name: str, task: str) -> dict:
    agent_registry = action_context.get_agent_registry()
    agent_run = agent_registry.get_agent(agent_name)
    current_memory = action_context.get_memory()
    result_memory = agent_run(user_input=task, memory=current_memory)
    return {
        "result": result_memory.items[-1].get("content", "No result"),
        "memory_id": id(result_memory)
    }
```

Best suited for task delegation where context continuity is critical, like escalations in support systems.

These patterns provide a toolkit for designing reliable, flexible multi-agent workflows where memory handling is intentional and strategic.

---

## Task 4: Removing Noise: Focusing Agent Attention

In multi-agent systems, reducing unnecessary distractions—"noise"—is crucial for optimal decision-making. Agents must focus on the most relevant signals from memory, tools, or context to improve clarity and accuracy.

### Types of Noise

* **Overloaded Contexts**: Too much history or irrelevant details can clutter memory.
* **Unfiltered Tool Output**: Agents may receive verbose or off-topic results.
* **Ambiguous Prompts**: Poorly defined goals introduce uncertainty in reasoning.

### Strategy 1: Prompt Narrowing

Define specific goals and scopes within prompts to reduce ambiguity.

```python
prompt = """
You are a legal assistant. Extract only termination clauses from the contract below.
"""
```

### Strategy 2: Memory Filtering

Apply constraints to only surface relevant memory entries.

```python
relevant_notes = memory.filter(lambda item: "budget" in item["content"])
```

### Strategy 3: Focused Tool Invocation

Use tool wrappers to limit output or control verbosity.

```python
@register_tool()
def extract_key_points(action_context, text):
    """Return only the top 3 key points from the given text."""
    return summarize(text, max_points=3)
```

### Strategy 4: Role-Specific Perspectives

Assign agent personas narrowly to ensure task focus.

```python
agent = Agent(
    goals=[Goal(name="financial_auditor", description="Audit only expenses above $10,000")],
    ...
)
```

### Benefits of Focusing Attention

* Reduced hallucinations and irrelevant outputs
* Faster and more efficient agent reasoning
* Better alignment with task objectives

These strategies help agents act decisively and effectively by filtering the signal from the noise.

---

## Task 5: Selective Memory Sharing: Using LLM Understanding for Context Selection

Sometimes an agent needs to share only the most relevant pieces of memory with another agent. Instead of rule-based filtering, we can use LLM understanding to select context dynamically.

### Intelligent Memory Filtering

```python
@register_tool(description="Delegate a task to another agent with selected context")
def call_agent_with_selected_context(action_context: ActionContext,
                                   agent_name: str,
                                   task: str) -> dict:
    agent_registry = action_context.get_agent_registry()
    agent_run = agent_registry.get_agent(agent_name)

    current_memory = action_context.get_memory()
    memory_with_ids = [
        {**item, "memory_id": f"mem_{idx}"}
        for idx, item in enumerate(current_memory.items)
    ]

    selection_schema = {
        "type": "object",
        "properties": {
            "selected_memories": {
                "type": "array",
                "items": {"type": "string"}
            },
            "reasoning": {"type": "string"}
        },
        "required": ["selected_memories", "reasoning"]
    }

    memory_text = "
".join([
        f"Memory {m['memory_id']}: {m['content']}" for m in memory_with_ids
    ])

    selection_prompt = f"""Review these memories and select the ones relevant for this task:

Task: {task}

Available Memories:
{memory_text}

Select memories that provide important context.
"""

    selection = prompt_llm_for_json(
        action_context=action_context,
        schema=selection_schema,
        prompt=selection_prompt
    )

    filtered_memory = Memory()
    selected_ids = set(selection["selected_memories"])
    for item in memory_with_ids:
        if item["memory_id"] in selected_ids:
            item_copy = item.copy()
            del item_copy["memory_id"]
            filtered_memory.add_memory(item_copy)

    result_memory = agent_run(user_input=task, memory=filtered_memory)

    current_memory.add_memory({
        "type": "system",
        "content": f"Memory selection reasoning: {selection['reasoning']}"
    })
    for memory_item in result_memory.items:
        current_memory.add_memory(memory_item)

    return {
        "result": result_memory.items[-1].get("content", "No result"),
        "shared_memories": len(filtered_memory.items),
        "selection_reasoning": selection["reasoning"]
    }
```

### Key Features:

* Each memory is tagged with an ID.
* The LLM selects relevant memories using structured reasoning.
* The rationale is preserved for traceability.

### Example LLM Output:

```json
{
  "selected_memories": ["mem_1", "mem_3", "mem_5"],
  "reasoning": "Selected memories contain cost details and the request for reduction."
}
```

This approach ensures the second agent receives only relevant context, minimizing distraction and maximizing utility.

### Recap of Memory Sharing Patterns:

* **Message Passing**: Simple, focused responses
* **Memory Reflection**: Transparency into reasoning
* **Memory Handoff**: Seamless task continuation
* **Selective Sharing**: Context filtering with LLM intelligence

Choose the pattern that best fits your agent collaboration needs.

---

## Task 6: Providing Agentic AI Information About the World

For agents to reason effectively, they must be informed about the state of the world. This can include dynamic data, domain knowledge, or contextual information relevant to the task.

### Why It's Critical

* **Contextual Accuracy**: Prevents hallucinations by grounding agent logic.
* **Real-Time Decisions**: Allows agents to adapt to the latest updates.
* **Policy Enforcement**: Agents can check rules, constraints, or norms dynamically.

### Strategies to Inform Agents

#### 1. Environment Injection

Pass world-state variables or references into the agent's execution context.

```python
action_context = ActionContext({
    'current_user': 'noor.cs2@yahoo.com',
    'timezone': 'Asia/Karachi',
    'project_status': get_latest_project_status()
})
```

#### 2. Tool-Based Queries

Expose structured APIs or database lookups as tools.

```python
@register_tool()
def lookup_policy(action_context, policy_name: str):
    return company_policy_db.get(policy_name)
```

#### 3. Shared Knowledge Modules

Create shared knowledge graphs, ontologies, or JSON files accessible by all agents.

```python
agent_knowledge = load_json("./schemas/budget_rules.json")
```

#### 4. Self-Prompted Retrieval

Use prompt-based summarization or embedding search to extract facts.

```python
prompt = f"""
Retrieve facts related to user behavior anomalies from the past 30 days.
"""
data = agent.invoke(prompt)
```

### Use Case: Budgeting Agent

A budgeting agent might:

* Pull exchange rates via an API
* Check past spend from a finance database
* Use rules from a JSON-based cost policy
* Validate each action against real-world constraints before committing

### Best Practices

* Minimize hardcoding: prefer externalized facts
* Log all information sources used for transparency
* Keep context size efficient: use summarization when needed

Informing agents with grounded, relevant world information boosts their reliability and utility significantly.

---
