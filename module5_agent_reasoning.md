# Module 5: Improving AI Agent Reasoning

## Task 1: Improving AI Agent Reasoning with In-Context Learning

In-context learning (ICL) is a powerful technique where large language models (LLMs) improve their performance by being provided with examples or demonstrations directly in the prompt. This allows agents to reason better and generate more accurate responses without any parameter tuning.

### Why Use In-Context Learning?

* **Improves reasoning quality** without needing to fine-tune the model
* **Quickly adapts behavior** to task-specific patterns
* **Keeps agents stateless** by using dynamic prompt engineering

### Basic In-Context Learning Structure

```python
prompt = f"""
You are an assistant that extracts city names from input text.

Examples:
Input: I visited Paris and London last summer.
Output: ["Paris", "London"]

Input: The trip from Berlin to Rome was amazing.
Output: ["Berlin", "Rome"]

Now process this:
Input: {user_input}
Output:
"""
```

This format allows the LLM to generalize from a few examples without being retrained.

### Integrating ICL into an Agent

Agents can be structured to generate in-context prompts dynamically based on goals, memory, or retrieved demonstrations:

```python
def generate_prompt_with_examples(input_text: str, examples: List[Tuple[str, str]]) -> str:
    example_section = "\n".join([f"Input: {ex[0]}\nOutput: {ex[1]}" for ex in examples])
    return f"""
Use the following examples to guide your output:
{example_section}

Now process this:
Input: {input_text}
Output:
"""
```

### Benefits of Using ICL in Agents

* **Adaptable**: Easily add or change examples without touching agent code.
* **Explainable**: Clear reasoning path for generated responses.
* **Domain-Aware**: Tailors agent behavior to the task context using real examples.

ICL provides a low-effort, high-reward upgrade for improving reasoning and accuracy in prompt-based agent architectures.

---

## Task 2: Improving AI Agent Reasoning with Up-front Planning & Chain of Thought

In more complex scenarios, agents benefit from planning their steps before acting. Chain of Thought (CoT) prompting encourages LLMs to break down problems into intermediate reasoning steps.

### What Is Up-front Planning?

Before taking any actions, the agent outlines its approach—what needs to be done and in what order. This improves reasoning and reduces errors.

### Example: Step-by-Step Planning

```python
prompt = f"""
You are an assistant solving this problem:
{user_input}

First, outline your plan step-by-step.
Then, execute each step in order.
"""
```

This pattern provides the LLM a structure to think through the problem before responding.

### Chain of Thought Prompting

Chain of Thought is a technique where the model is encouraged to explain its reasoning out loud before giving a final answer.

```python
prompt = f"""
Question: If there are 3 red balls and 2 blue balls in a bag, and I take one out without looking, what is the probability it’s red?

Let’s think step-by-step.

Answer:
- Total balls = 3 red + 2 blue = 5
- Probability of red = 3 out of 5 = 3/5
- Final answer: 60%
"""
```

### Integrating into Agents

CoT prompting can be used in:

* **Planning agents** that reason through steps before acting
* **Validation agents** that explain answers to confirm correctness
* **Tutoring agents** that guide users through a problem

### Benefits

* **Improved accuracy** on reasoning-heavy tasks
* **Transparency** for easier debugging and review
* **Better generalization** from structured prompting

Combining up-front planning and CoT produces more deliberate, verifiable agent behavior—critical for high-stakes use cases like finance, law, and healthcare.

---

## Task 3: Extending the Agent Loop with Capabilities

While tools provide specific functions, sometimes we need to modify the agent’s core behavior more fundamentally. The **Capability pattern** encapsulates agent loop customizations into reusable classes that plug into the lifecycle without modifying core logic.

### Why Use Capabilities?

* **Modularity**: Cleanly isolate cross-cutting behaviors (e.g., logging, time-awareness)
* **Composability**: Add and remove features without touching the agent loop
* **Reusability**: Build once, apply to multiple agents

### Lifecycle Hooks in a Capability

```python
class Capability:
    def init(self, agent, action_context): pass
    def start_agent_loop(self, agent, action_context): return True
    def process_prompt(self, agent, action_context, prompt): return prompt
    def process_response(self, agent, action_context, response): return response
    def process_action(self, agent, action_context, action): return action
    def process_result(self, agent, action_context, response, action_def, action, result): return result
    def process_new_memories(self, agent, action_context, memory, response, result, memories): return memories
    def end_agent_loop(self, agent, action_context): pass
    def should_terminate(self, agent, action_context, response): return False
    def terminate(self, agent, action_context): pass
```

Each method is called at a specific phase in the agent's execution. This design allows for layered control similar to middleware.

### Example: TimeAwareCapability

Adds real-time awareness to the agent.

```python
from datetime import datetime
from zoneinfo import ZoneInfo

class TimeAwareCapability(Capability):
    def init(self, agent, action_context):
        timezone = ZoneInfo(action_context.get("time_zone", "America/Chicago"))
        current_time = datetime.now(timezone)
        memory = action_context.get_memory()
        memory.add_memory({
            "type": "system",
            "content": f"Current time is {current_time.strftime('%H:%M %A, %B %d, %Y')}"
        })

    def process_prompt(self, agent, action_context, prompt):
        timezone = ZoneInfo(action_context.get("time_zone", "America/Chicago"))
        current_time = datetime.now(timezone)
        messages = prompt.messages
        system_msg = f"Current time: {current_time.strftime('%H:%M %A, %B %d, %Y')} ({timezone.key})

"

        if messages and messages[0]["role"] == "system":
            messages[0]["content"] = system_msg + messages[0]["content"]
        else:
            messages.insert(0, {"role": "system", "content": system_msg})
        return Prompt(messages=messages)
```

### Enhanced Example: Tracking Action Duration

```python
class EnhancedTimeAwareCapability(TimeAwareCapability):
    def process_action(self, agent, action_context, action):
        action["execution_time"] = datetime.now(ZoneInfo(action_context.get("time_zone", "America/Chicago"))).isoformat()
        return action

    def process_result(self, agent, action_context, response, action_def, action, result):
        if isinstance(result, dict):
            result["action_duration"] = (
                datetime.now(ZoneInfo(action_context.get("time_zone"))) -
                datetime.fromisoformat(action["execution_time"])
            ).total_seconds()
        return result
```

### Agent Configuration Example

```python
agent = Agent(
    goals=[Goal(name="task", description="Complete task considering current time")],
    agent_language=JSONAgentLanguage(),
    action_registry=registry,
    generate_response=llm.generate,
    environment=PythonEnvironment(),
    capabilities=[TimeAwareCapability()]
)
```

With capabilities, the agent becomes flexible and extensible. Each enhancement—time-awareness, logging, monitoring—can be encapsulated cleanly and reused across contexts.

---

## Task 4: Ahead of Time Planning for Improving Agent Reasoning

One key to making agents more effective is getting them to think strategically before taking action. Instead of jumping straight into tool execution, agents should first develop a comprehensive plan.

### The Plan First Pattern

We use a capability to enforce this behavior:

* At agent startup, generate a step-by-step plan
* Store the plan in memory
* Refer back to the plan throughout execution

### PlanFirstCapability Example

```python
class PlanFirstCapability(Capability):
    def __init__(self, plan_memory_type="system", track_progress=False):
        super().__init__(
            name="Plan First Capability",
            description="The Agent will always create a plan and add it to memory"
        )
        self.plan_memory_type = plan_memory_type
        self.first_call = True
        self.track_progress = track_progress

    def init(self, agent, action_context):
        if self.first_call:
            self.first_call = False
            plan = create_plan(
                action_context=action_context,
                memory=action_context.get_memory(),
                action_registry=action_context.get_action_registry()
            )
            action_context.get_memory().add_memory({
                "type": self.plan_memory_type,
                "content": "You must follow these instructions carefully to complete the task:
" + plan
            })
```

### Plan Creation Tool

```python
@register_tool(tags=["planning"])
def create_plan(action_context: ActionContext, memory: Memory, action_registry: ActionRegistry) -> str:
    tool_descriptions = "
".join(f"- {a.name}: {a.description}" for a in action_registry.get_actions())
    memory_content = "
".join(f"{m['type']}: {m['content']}" for m in memory.items if m['type'] in ['user', 'system'])

    prompt = f"""Given the task in memory and the available tools, create a detailed plan.

1. Identify key components of the task
2. Match steps to available tools
3. For each step, specify:
   - What to do
   - What tools to use
   - Required input
   - Expected result

Available tools:
{tool_descriptions}

Task context from memory:
{memory_content}

Create a plan that accomplishes this task effectively."""

    return prompt_llm(action_context=action_context, prompt=prompt)
```

### Example Plan Output

```text
Plan for Sales Data Analysis:

1. Data Validation - Tool: validate_data()
2. Initial Analysis - Tool: analyze_data()
3. Trend Identification - Tool: find_patterns()
4. Visualization - Tool: create_visualization()
5. Report Generation - Tool: generate_report()
```

### Agent Configuration

```python
agent = Agent(
    goals=[Goal(name="analysis", description="Analyze sales data and create a report")],
    capabilities=[PlanFirstCapability(track_progress=True)],
    # other config...
)

result = agent.run("Analyze our Q4 sales data and create a report")
```

### Benefits

* **Upfront clarity** on process
* **Reduces mistakes** by following a planned path
* **Encourages structured thinking** and tool use

This strategy improves agent discipline and output consistency for tasks requiring multiple steps.

---

## Task 5: Improving AI Agent Reasoning with In-loop Planning

In-loop planning complements up-front planning by allowing agents to adjust their strategy dynamically as they gather new information or face unexpected challenges. This helps maintain flexibility and resilience within long-running tasks.

### What Is In-loop Planning?

Instead of planning everything at the start, the agent revisits and revises its plan at each step based on current memory and results. This mimics how humans adjust their approach as they learn more about a problem.

### Benefits

* **Improves adaptability** in dynamic environments
* **Supports error recovery** by adjusting to failed steps
* **Enables reactive problem-solving** with memory-based feedback loops

### Implementation via Capability

You can implement this pattern using a capability that triggers re-planning logic mid-execution:

```python
class InLoopPlanningCapability(Capability):
    def __init__(self, planning_interval=1):
        super().__init__(
            name="In-loop Planner",
            description="Periodically re-evaluates and updates the execution plan."
        )
        self.counter = 0
        self.planning_interval = planning_interval

    def end_agent_loop(self, agent, action_context):
        self.counter += 1
        if self.counter % self.planning_interval == 0:
            plan = create_plan(
                action_context=action_context,
                memory=action_context.get_memory(),
                action_registry=action_context.get_action_registry()
            )
            action_context.get_memory().add_memory({
                "type": "system",
                "content": f"Updated plan based on progress:
{plan}"
            })
```

### When to Use

* **Long-running processes** that evolve over time
* **Multi-agent collaborations** where new context may arise
* **Monitoring or correction loops** with dynamic feedback

### Combining with PlanFirstCapability

```python
agent = Agent(
    goals=[Goal(name="qa_review", description="Review customer QA logs and propose fixes")],
    capabilities=[
        PlanFirstCapability(),
        InLoopPlanningCapability(planning_interval=2)
    ],
    # other configuration...
)
```

### Summary

In-loop planning augments agent robustness by making them plan and revise continuously as conditions change. When paired with up-front planning, this creates agents that are both strategic and reactive.

---
