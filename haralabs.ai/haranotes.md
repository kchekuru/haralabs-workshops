workshop notes

https://github.com/google-research/harness2
https://github.com/n8n-io

demo is Module 5’s interactive Deep Agent Builder, not a general-purpose automation tool like n8n. It gives you a UI to choose a model, instructions, and tools, then chat with the agent and inspect its tool calls.

Its React frontend talks to a FastAPI backend that runs the deepagents agent. You can enable tools such as file access, shell commands, web search, and RAG over the workshop’s IT knowledge base. The UI asks for approval before potentially consequential actions. It also demonstrates an optional Docker sandbox for execution; that’s a restricted container, not a separate machine.

The demo is mainly a hands-on interface for the agent concepts in Module 5. It requires setup before it runs: backend dependencies and server, plus npm install for the frontend. The demo README has the setup steps; start_client.sh shows how the JupyterLab launcher handles setup that hasn’t been completed yet.

-- architecture for super intelligence --

"anatomy of aGENtic systems "
intro_to_agents.ipynb
part # 1 model
part # 2 tools
part # 3 memory
part # 4 routing


"reflection loop" 
intro_to_agents.ipynb
model requests a tool, code adds its result to memory, then calls the model again with that updated history.
# Add tool observations to the conversation history
for result in tool_results:
    memory.append({
        "role": "tool",
        "tool_call_id": ...,  # fill in from the result
        "name": ...,          # fill in from the result
        "content": str(...),  # fill in from the result
    })

# Ask the model to respond to the updated history
if tool_results:
    llm_response = call_llm(
        model_client=...,
        model_name=...,
        message_history=...,
        tool_list=...,
    )
    memory.append(llm_response)
key idea: the tool result is the observation; the next model call lets the model decide(aka reflect upon) whether it can answer or needs another tool (reflection loop)

"self correction workflow"
intro_to_agents.ipynb
harness_lab_answer.py
for _ in range(max_turns):
    response = invoke_with_retry(model, messages) //model self corrects
    messages.append(response)
    if not response.tool_calls:
        return response.content

    for call in response.tool_calls:
        try:
            result = registry[call["name"]].invoke(call["args"])
        except Exception as exc:
            result = f"ERROR: {type(exc).__name__}: {str(exc)[:500]}"

        messages.append(ToolMessage(content=str(result), tool_call_id=call["id"]))

"tool calling aka tool pipeline"
docgen_agent.py
tools is the list of functions the agent is allowed to call. It’s passed into create_agent here:
# Create the tool collection
tools = [search_tavily]

# Create the agent with the model, tools, and instructions
agent = create_agent(
    model=llm,
    tools=tools,
    system_prompt=system_prompt,
)

"dynamic task routing"
deep_agent.py
dynamic task routing is not hard-coded 'if/else' but **model-driven delegation** router: the prompt tells the model when to delegate, and the haralabs agents library provides the `task` tool.

The prompt (aka builtin) describes the routing behavior in `deep_agent.py`:

```python
builtin = [
    "- Planning (write_todos): organize tasks",
    "- Delegation (task): ask a subagent to handle a focused subtask with the same permissions",
]
```

It also tells the model when to use a particular tool

```text
1. Answer simple questions directly. For independent subtasks, use task and check its result.
```

The `task` tool is included in the allowed tool names by `capabilities.py`:

```python
def enabled_tools(skill_ids):
    names = {"write_todos", "task"}
    ...
    return names
```
Then `CapabilityMiddleware` limits which tools the model can see and call. The agent is wired with that middleware in `create_agent`.

"runtime inference"
server.py
The runtime inference call is in : session.agent.astream_events(...). 
That sends the user’s message into the agent, which calls the model( api call); the model may respond directly or request the tool.

The model is configured and the agent assembled in create_agent(), but the inference itself is handled inside the extended framework library (server.py - async call / event based)

"multi-agent collaboration with state"
deep_agent.py

notice line 224, 197 for 'task'

the prompt describes the 'task' tool for delegating focused subtasks. Its instruction to use 'task' for independent subtasks is at line 224
'langgraph'.MemorySaver() is configured as the agent’s checkpointer.
that checkpointer is passed into 'deepagent'.create_deep_agent(...), so conversation state is persisted across calls. [ making it stateful - as api call to model is stateless]
In server.py, each session has a 'thread_id'; the config property uses 'it/thread_id' to identify that session’s state when calling the agent.
The subagent execution itself is handled inside the "deepagents==0.3.11" when the model invokes 'task'


"core libraries"
deepagents==0.3.11
langchain-nvidia-ai-endpoints>=1.4.3,<2
langchain_mcp_adapters~=0.2.1
langgraph-cli[inmem]
langsmith~=0.3.0