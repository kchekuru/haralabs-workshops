workshop notes

https://github.com/google-research/harness2
https://github.com/n8n-io

demo is Module 5’s interactive Deep Agent Builder, not a general-purpose automation tool like n8n. It gives you a UI to choose a model, instructions, and tools, then chat with the agent and inspect its tool calls.

Its React frontend talks to a FastAPI backend that runs the deepagents agent. You can enable tools such as file access, shell commands, web search, and RAG over the workshop’s IT knowledge base. The UI asks for approval before potentially consequential actions. It also demonstrates an optional Docker sandbox for execution; that’s a restricted container, not a separate machine.

The demo is mainly a hands-on interface for the agent concepts in Module 5. It requires setup before it runs: backend dependencies and server, plus npm install for the frontend. The demo README has the setup steps; start_client.sh shows how the JupyterLab launcher handles setup that hasn’t been completed yet.

-- newer intelligence constructs and paradigm review --
"multi-agent collaboration"
deep_agent.py

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
key idea: the tool result is the observation; the next model call lets the model decide whether it can answer or needs another tool (reflection loop)

"tool calling"
docgen_agent.py, tools is the list of functions the agent is allowed to call. It’s passed into create_agent here:
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
