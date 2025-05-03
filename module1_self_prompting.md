# Module 1: Extending AI Agents with Self-Prompting

## Task 1: Prompts as Computation

Prompts act as executable code-like constructs that drive agent logic. In this model, prompts become the control structures, enabling:

* Flexible execution paths
* Reusable logic components
* Externalized behaviors

This approach enhances adaptability, allowing developers to revise agent behavior without retraining models or modifying core logic. Prompts become the key interface through which reasoning and planning are orchestrated.

### Key Points:

* Prompts are not just instructions—they define the behavior of the agent.
* They enable dynamic execution, task delegation, and modular thinking.
* Useful for extending LLMs with reusable and human-readable logic blocks.

### Example Prompt-as-Code Perspective

```python
# Instead of hardcoding behavior
if user_intent == "search":
    agent.execute(search_query)

# Use prompt-as-logic
prompt = "Search for relevant articles based on user question: {{question}}"
agent.invoke(prompt)
```

This marks the transition of traditional control logic to prompt-driven agent behavior—a cornerstone of the agentic paradigm.

---

## Task 2: Bridging Computer Tools & Unstructured Data with Prompting – the AI Shim

The AI Shim acts as an interface layer between agents, unstructured data, and traditional tools. By using prompts to unify these disparate sources, agents can dynamically:

* Convert unstructured input (e.g., text, web pages, emails) into structured commands
* Interface with APIs or software tools via formatted prompt templates
* Serve as translation layers between LLM outputs and real-world applications

### Key Concepts:

* **AI Shim = Prompt-based Middleware**
* Abstracts away implementation details of external tools
* Encourages decoupled, prompt-driven integration with systems

### Example Use Case

```python
unstructured_text = "Hey, can you get the weather for tomorrow in New York?"
prompt = "Convert the following request into a weather API call: {{unstructured_text}}"
api_call = agent.invoke(prompt)
```

This pattern simplifies tool usage by letting the agent use natural language to generate tool-friendly inputs—closing the gap between data ambiguity and actionable logic.

---

## Task 3: AI Agent Structured Data Extraction

LLM-powered agents can be prompted to extract structured data—such as JSON, tables, or key-value pairs—from unstructured input like emails, PDFs, or chat logs. This enables:

* Seamless data transformation
* Automation of data entry and analysis
* Integration with downstream tools and databases

### Key Concepts:

* Use clear, format-enforcing prompts to guide output structure
* Validate and post-process extracted data for reliability

### Example Structured Prompt

```python
prompt = """
Extract the following fields from this email:
- Sender
- Subject
- Date
- Action requested
Return the result as JSON.

Email:
{{email_text}}
"""
structured_output = agent.invoke(prompt)
```

This capability turns LLMs into powerful parsers that reduce manual effort and speed up data operations.

---

## Task 4: An Invoice Processing Agent

This task showcases how an agent can process real-world documents like invoices using prompt engineering. By designing targeted prompts, an agent can:

* Parse invoice data from PDFs or text
* Extract structured information like vendor, date, total amount
* Validate and transform the data for business processes

### Process Overview:

1. Receive invoice as input (text or OCR output)
2. Use a prompt to extract key-value pairs
3. Output JSON suitable for databases or APIs

### Example Prompt for Invoice Extraction

```python
prompt = """
Extract the following fields from the invoice below:
- Invoice Number
- Date
- Vendor Name
- Total Amount

Return the output in JSON format.

Invoice Text:
{{invoice_text}}
"""
data = agent.invoke(prompt)
```

### Example: Agent Code for Invoice Processing

```python
def create_invoice_agent():
    # Create action registry with our invoice tools
    action_registry = PythonActionRegistry()

    # Create our base environment
    environment = PythonEnvironment()

    # Define our invoice processing goals
    goals = [
        Goal(
            name="Persona",
            description="You are an Invoice Processing Agent, specialized in handling and storing invoice data."
        ),
        Goal(
            name="Process Invoices",
            description="""
            Your goal is to process invoices by extracting their data and storing it properly.
            For each invoice:
            1. Extract all important information including numbers, dates, amounts, and line items
            2. Store the extracted data indexed by invoice number
            3. Provide confirmation of successful processing
            4. Handle any errors appropriately
            """
        )
    ]

    # Create the agent
    return Agent(
        goals=goals,
        agent_language=AgentFunctionCallingActionLanguage(),
        action_registry=action_registry,
        generate_response=generate_response,
        environment=environment
    )
```

This practical use case ties together structured data extraction and AI shims for automation.

---

## Task 5: The Persona Pattern and Reasoning

Personas are an efficient programming abstraction that help structure agent behavior around specific roles or expert capabilities. Instead of relying on a monolithic agent, multiple personas can be designed with distinct responsibilities and reasoning models.

### Why Use Personas?

* They modularize reasoning and task execution
* Improve clarity, reusability, and specialization
* Make agents more interpretable and debuggable

### Key Concepts:

* **Persona = Specialized Role** (e.g., lawyer, accountant, researcher)
* Each persona is designed with a goal and expected reasoning approach
* Personas can be reused across tasks and agents

### Example Persona Definition Prompt

```python
prompt = """
You are a legal advisor. Your role is to assess contractual documents and highlight clauses related to termination rights, payment obligations, and liability.
"""
legal_persona_response = agent.invoke(prompt)
```

This approach lets developers build explainable and role-driven systems where each sub-agent focuses on specific decision contexts.

---

## Task 6: Simple Multi-Agent Systems with Personas

Using multiple personas enables the creation of lightweight multi-agent systems where each persona plays a clearly defined role. These personas can interact with each other to delegate subtasks, validate decisions, or simulate expert consultations.

### Benefits:

* Facilitates scalable and modular architectures
* Supports collaborative reasoning across personas
* Enables parallel processing of subtasks

### Example: Two-Persona System

```python
# Researcher Persona
prompt_researcher = """
You are a research assistant. Summarize the main findings from the following scientific article.
"""
summary = agent.invoke(prompt_researcher)

# Reviewer Persona
prompt_reviewer = f"""
You are a scientific reviewer. Critically evaluate the following summary:
{summary}
"""
review = agent.invoke(prompt_reviewer)
```

This structure simulates internal dialogue, validation, and collaboration—making reasoning processes transparent and verifiable.

---

## Task 7: Prompting for Expertise with the Persona Pattern

In this task, multiple expert personas are registered as tools, each specializing in a distinct domain (documentation, testing, code quality, communication). The pattern allows dynamic consultation with these experts using prompt-based APIs. This modular and declarative approach enables scalable multi-expert agent design.

### Example: Registered Tools Using Expert Personas

```python
@register_tool(tags=["documentation"])
def generate_technical_documentation(action_context: ActionContext, code_or_feature: str) -> str:
    return prompt_expert(
        action_context=action_context,
        description_of_expert="""
        You are a senior technical writer with 15 years of experience...
        """,
        prompt=f"""
        Please create comprehensive technical documentation for:
        {code_or_feature}
        """
    )

@register_tool(tags=["testing"])
def design_test_suite(action_context: ActionContext, feature_description: str) -> str:
    return prompt_expert(
        action_context=action_context,
        description_of_expert="""
        You are a senior QA engineer with 12 years of experience...
        """,
        prompt=f"""
        Please design a comprehensive test suite for:
        {feature_description}
        """
    )

@register_tool(tags=["code_quality"])
def perform_code_review(action_context: ActionContext, code: str) -> str:
    return prompt_expert(
        action_context=action_context,
        description_of_expert="""
        You are a senior software architect with 20 years of experience...
        """,
        prompt=f"""
        Please review the following code:
        {code}
        """
    )

@register_tool(tags=["communication"])
def write_feature_announcement(action_context: ActionContext, feature_details: str, audience: str) -> str:
    return prompt_expert(
        action_context=action_context,
        description_of_expert="""
        You are a senior product marketing manager...
        """,
        prompt=f"""
        Please write a feature announcement for:
        {feature_details}
        Audience: {audience}
        """
    )
```

### Key Ideas:

* Each tool encapsulates a "persona" with domain expertise.
* Prompts act as delegations to specialists for improved output quality.
* Facilitates clearer reasoning separation and testable logic blocks.

---
