
# Module 4: Dependency Injection for Tools

## Task 1: Isolating Agents from Accidental Complexity

When building AI agents, accidental complexity often creeps in when tools and environment details are tightly coupled with the agent's core reasoning. To isolate this complexity, we use **dependency injection**—a design pattern where the environment or external capabilities are passed into the agent, rather than being hardcoded.

### Why Isolation Matters

* **Modularity**: Keeps agents focused on reasoning, not tool orchestration.
* **Reusability**: Agents can be reused across different environments.
* **Testability**: Easier to swap in mocks for testing.

### Simple Example: Injecting the Tool Registry

```python
class PythonEnvironment(Environment):
    def __init__(self, tools):
        self.tools = tools

# Define an agent that doesn’t know how the tools are implemented
invoice_agent = Agent(
    goals=[...],
    environment=PythonEnvironment(tools={
        'extract_invoice': extract_invoice_data,
        'validate_format': validate_invoice_fields
    })
)
```

### Without Injection (Bad Example)

```python
# Agent depends directly on global function
result = extract_invoice_data(text)
```

This makes the agent fragile and tightly bound to implementation specifics.

### With Injection (Good Example)

```python
# Injected at runtime
result = agent.environment.tools['extract_invoice'](text)
```

Dependency injection isolates reasoning logic from operational details, resulting in agents that are easier to debug, maintain, and scale.

---

## Task 2: Decoupling Tools from Agent and Other Dependencies

The `action_context` pattern enables tools to operate independently from agent internals by injecting all necessary runtime dependencies dynamically.

### Why ActionContext Matters

* Avoids tight coupling between tools and agents
* Supports memory access, authentication, and request-specific properties
* Makes tools reusable and testable across environments

### The Problem

```python
@register_tool(description="Analyze code quality")
def analyze_code_quality(code: str):
    # Needs memory context – but can’t directly access agent
    return prompt_expert(prompt=f"Review this code:
{code}")
```

### The Solution: ActionContext

```python
class ActionContext:
    def __init__(self, properties: Dict=None):
        self.context_id = str(uuid.uuid4())
        self.properties = properties or {}

    def get(self, key: str, default=None):
        return self.properties.get(key, default)

    def get_memory(self):
        return self.properties.get("memory")
```

### Refactored Tool with Dependency Injection

```python
@register_tool(description="Analyze code quality", tags=["code_quality"])
def analyze_code_quality(action_context: ActionContext, code: str):
    memory = action_context.get_memory()
    context_lines = [
        f"User: {m['content']}" if m["type"] == "user" else f"Decision: {m['content']}"
        for m in memory.get_memories() if "implementation" in m["content"]
    ]
    review_prompt = f"""
Review this code in the context of its development history:

{chr(10).join(context_lines)}

Code:
{code}
"""
    generate_response = action_context.get("llm")
    return generate_response(review_prompt)
```

### Context-Specific Authentication Example

```python
@register_tool(description="Update code review status", tags=["project_management"])
def update_review_status(action_context: ActionContext, review_id: str, status: str):
    auth_token = action_context.get("auth_token")
    headers = {"Authorization": f"Bearer {auth_token}"}
    response = requests.post(
        f"https://.../reviews/{review_id}/status",
        headers=headers,
        json={"status": status}
    )
    return {"status": "updated", "review_id": review_id} if response.ok else response.text
```

### Agent Execution with Custom Context

```python
def run(self, user_input, memory=None, action_context_props=None):
    memory = memory or Memory()
    action_context = ActionContext({
        'memory': memory,
        'llm': self.generate_response,
        **(action_context_props or {})
    })
    ...

some_agent.run("Update project status...", memory=..., action_context_props={"auth_token": "my_auth_token"})
```

With ActionContext, agents and tools stay modular, reusable, and safe across varying environments.

---

## Task 3: Dependency Injection, the Environment, and the Decorator

Now that we have `ActionContext` to pass shared resources, we need a way to provide dependencies only to the tools that require them. Some tools are simple and need only basic arguments, while others need memory, authentication tokens, or configuration.

### Why Selective Injection Matters

* Avoids exposing sensitive context to tools that don’t need it
* Keeps tool signatures minimal and secure
* Maintains clarity and modularity in agent logic

### Clean Agent Execution

```python
def handle_agent_response(self, action_context: ActionContext, response: str):
    action_def, action = self.get_action(response)
    result = self.environment.execute_action(self, action_context, action_def, action["args"])
    return result
```

### Updated Environment System with Injection Logic

```python
class PythonEnvironment(Environment):
    def execute_action(self, agent, action_context: ActionContext, action: Action, args: dict) -> dict:
        try:
            args_copy = args.copy()

            if has_named_parameter(action.function, "action_context"):
                args_copy["action_context"] = action_context

            for key, value in action_context.properties.items():
                param_name = f"_{key}"
                if has_named_parameter(action.function, param_name):
                    args_copy[param_name] = value

            result = action.execute(**args_copy)
            return self.format_result(result)
        except Exception as e:
            return {"tool_executed": False, "error": str(e)}
```

### Example Tool with Hidden Dependencies

```python
@register_tool(description="Update user settings in the system")
def update_settings(action_context: ActionContext,
                   setting_name: str,
                   new_value: str,
                   _auth_token: str,
                   _user_config: dict) -> dict:
    headers = {"Authorization": f"Bearer {_auth_token}"}

    if setting_name not in _user_config["allowed_settings"]:
        raise ValueError(f"Setting {setting_name} not allowed")

    response = requests.post(
        "https://api.example.com/settings",
        headers=headers,
        json={"setting": setting_name, "value": new_value}
    )
    return {"updated": True, "setting": setting_name}
```

### Updated Tool Metadata for Schema Generation

```python
def get_tool_metadata(func, ...):
    signature = inspect.signature(func)
    type_hints = get_type_hints(func)

    args_schema = {"type": "object", "properties": {}, "required": []}

    for param_name, param in signature.parameters.items():
        if param_name in ["action_context", "action_agent"] or param_name.startswith("_"):
            continue

        param_type = type_hints.get(param_name, str)
        args_schema["properties"][param_name] = {"type": "string"}

        if param.default == param.empty:
            args_schema["required"].append(param_name)

    return {
        "name": func.__name__,
        "description": func.__doc__,
        "parameters": args_schema,
        "tags": [],
        "terminal": False,
        "function": func
    }
```

### Agent Perspective Stays Simple

```python
action = {
    "tool": "update_settings",
    "args": {
        "setting_name": "theme",
        "new_value": "dark"
    }
}
```

The agent only handles surface-level arguments while the environment injects context-specific dependencies behind the scenes.

---
