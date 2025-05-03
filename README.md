# AI Agents and Agentic AI Architecture in Python

**Offered by:** Vanderbilt University via Coursera
**Instructor:** Dr. Jules White
**Start Date:** May 3
**License:** © Coursera & Vanderbilt University (educational purpose only)
Personal learning notes and summaries compiled by Noor Uddin ([noor.cs2@yahoo.com](mailto:noor.cs2@yahoo.com)) for study and reference

---

## Course Overview

This repository includes structured notes, key takeaways, and practical insights from the Coursera course **"AI Agents and Agentic AI Architecture in Python"**. The course focuses on the practical implementation of agent-based systems using Python. It highlights how to design autonomous AI agents capable of reasoning, decision-making, and executing tasks in a modular and reusable manner.

**Disclaimer:** All content is derived from course lectures and resources provided via Coursera. This repository is intended strictly for educational use and does not claim any rights over the original course material.

---

## Modules

### Module 1: Extending AI Agents with Self-Prompting

* **Prompts as Computation**: Introduction to using prompts as functional components in agent design.
* **Self-Prompting & Clean Separation of AI Agent Reasoning**: How agents self-generate prompts and separate core reasoning from task execution.
* **Bridging Computer Tools & Unstructured Data with Prompting - the AI Shim**: Demonstrates using prompting as an interface to blend tool execution and unstructured data inputs.
* **AI Agent Structured Data Extraction**: Teaches agents to extract structured output from unstructured prompts.
* **An Invoice Processing Agent**: A case study of building a document-processing agent using prompts.
* **The Persona Pattern and Reasoning**: Introduction to using "personas" as design abstractions for agent reasoning.
* **The Persona Pattern**: Deep dive into structuring personas as modular components in agent design.
* **Format of the Persona Pattern**: Reading material explaining standardized persona formats.
* **Simple Multi-Agent Systems with Personas**: Building lightweight agent ecosystems using specialized personas.
* **Consulting Experts or Simulating with the Persona Pattern**: Simulating expert reasoning using persona prompts.
* **The Persona Abstraction & Agents**: Conceptualizing personas as reusable, programmable units.
* **Invoice Processing with Experts**: Extending agent workflows with simulated expert personas.
* **Using Human Policies for Document-as-Implementation**: Emphasizes policy-driven reasoning as a programming technique.
* **Persona & Self-Prompting Review (Assessment)**: A graded review to consolidate learning.

### Module 2: AI Agent Design Principles & Safety

* **The MATE Design Principles for AI Agents**: Introduces the MATE framework (Monitor, Act, Think, and Explain) for building safe and explainable agents.
* **MATE Design Principles in Code**: Demonstrates how MATE principles are implemented in Python-based agent design.
* **AI Agents & Environment Safety**: Discusses how to ensure agents interact safely with their environment using policy and context-aware safeguards.

### Module 3: Multi-Agent Systems

* **Introduction to Multi-Agent Systems**: Explains the foundational concept of systems composed of multiple interacting agents.
* **Building Multi-Agent Systems: Agent-to-Agent Communication**: Covers protocols and techniques for enabling agent collaboration.
* **Agent Interaction & Memory**: Discusses how memory mechanisms enhance agent cooperation and context tracking.
* **Agent Interaction Patterns with Memory**: Implementation strategies for memory-augmented interactions.
* **Removing Noise: Focusing Agent Attention**: Techniques to filter distractions and direct focus in communication.
* **Advanced Agent Interaction**: Advanced patterns for scalable and robust multi-agent collaboration.
* **Providing Agentic AI Information About the World**: Methods to supply real-world context and data to agents.
* **Agent Interaction Architectures (Assessment)**: A graded evaluation of architecture strategies in multi-agent systems.

### Module 4: Dependency Injection for Tools

* **Isolating Agents from Accidental Complexity**: Explains how to design agents that remain clean and focused by isolating them from unnecessary tool complexity.
* **Clean AI Tools with Dependency Injection**: Introduces dependency injection as a pattern to simplify AI tool integration while promoting modular design.
* **Clean Tool Dependency Injection with the Environment**: Extends the injection pattern to handle external environmental inputs, improving flexibility and maintainability.

### Module 5: Approaches to Improving AI Agent Reasoning

* **Improving AI Agent Reasoning with In-Context Learning**: Uses in-context examples to guide agent decisions without altering model parameters.
* **Improving AI Agent Reasoning with Up-front Planning & Chain of Thought**: Emphasizes explicit planning steps and logical chains to structure agent responses.
* **The Capability Architectural Pattern**: Introduces a modular pattern for grouping reasoning capabilities in agents.
* **Ahead of Time Planning for Improving Agent Reasoning**: Describes how proactive planning boosts reliability and clarity in agent behavior.
* **Improving AI Agent Reasoning with In-loop Planning**: Focuses on adaptive, iterative planning methods during agent execution.
* **Intermediate Planning: Tracking Progress in the Agent Loop**: Covers progress monitoring within agents to enable dynamic plan adjustment.
* **The Great Agent Trade-off: Ahead of Time vs. Dynamic**: Discusses trade-offs between static planning and flexible, real-time decision-making.
---

## Notes Repository Structure (Suggested)

```
AI-Agents-Agentic-Python/
├── module1_self_prompting.md
├── module2_design_principles_safety.md
├── module3_multi_agent_systems.md
├── module4_dependency_injection.md
├── module5_agent_reasoning.md
├── README.md
├── LICENSE
```

Let me know when you're ready to proceed with Module 2!
