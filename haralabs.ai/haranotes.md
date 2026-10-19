workshop notes

https://github.com/google-research/harness2
https://github.com/n8n-io

demo is Module 5’s interactive Deep Agent Builder, not a general-purpose automation tool like n8n. It gives you a UI to choose a model, instructions, and tools, then chat with the agent and inspect its tool calls.

Its React frontend talks to a FastAPI backend that runs the deepagents agent. You can enable tools such as file access, shell commands, web search, and RAG over the workshop’s IT knowledge base. The UI asks for approval before potentially consequential actions. It also demonstrates an optional Docker sandbox for execution; that’s a restricted container, not a separate machine.

The demo is mainly a hands-on interface for the agent concepts in Module 5. It requires setup before it runs: backend dependencies and server, plus npm install for the frontend. The demo README has the setup steps; start_client.sh shows how the JupyterLab launcher handles setup that hasn’t been completed yet.