# Multi-Agent Data Analytics System (MAS)

## 1. Implementation Plan

### Use Case Overview
A coordinated set of agents will ingest raw data, clean it, analyze trends, and visualize key findings. The system aims to streamline analytics tasks like summarization, forecasting, and dashboard creation.

### Agent Roles
1. **Data Extractor** – collects data from external sources. Utilizes the **Browser Tool** for web extraction or custom API targets via the Gateway.
2. **Data Cleaner** – uses the **Code Interpreter** to sanitize and structure data.
3. **Data Analyst** – generates summaries and forecasts. Leverages models like Anthropic Claude for reasoning and Amazon Titan for numerical tasks.
4. **Data Visualizer** – produces charts and graphs using the Code Interpreter.
5. **Orchestrator** – manages task flow and stores intermediate results in **AgentCore Memory**.

### Communication Strategy
- **Gateway** exposes APIs and Lambda functions as tools so each agent can invoke external services via the [Model Context Protocol](https://docs.aws.amazon.com/bedrock/latest/userguide/model-context-protocol.html).
- **Memory** preserves conversation history and extracted insights, enabling agents to share context.

### Tool Usage
- **Browser Tool** – acquire web data or download files.
- **Code Interpreter Tool** – perform data transformations, run Python libraries like pandas or matplotlib, and generate visuals.

### Foundation Model Choices
| Step            | Suggested Model                            |
|-----------------|--------------------------------------------|
| Extraction      | Amazon Titan Text                         |
| Cleaning        | Amazon Titan or Claude 3 Haiku            |
| Analysis        | Anthropic Claude 3 Sonnet/Haiku           |
| Visualization   | Claude model with Code Interpreter        |

## 2. Design & Coding Guide

### Agent Structure Template
```python
from bedrock_agentcore.runtime import BedrockAgentCoreApp
from pydantic import BaseModel

app = BedrockAgentCoreApp()

class InputSchema(BaseModel):
    prompt: str

@app.entrypoint
async def handler(payload: InputSchema, context):
    # agent logic here
    return {"result": "output"}
```
- Define intents, inputs, and outputs with Pydantic schemas.
- Decorate an entry function with `@app.entrypoint` as shown in the [hosting agent tutorial](01-tutorials/01-AgentCore-runtime/01-hosting-agent/README.md).

### Orchestration Pattern
- The Orchestrator agent invokes sub-agents sequentially and stores results in Memory.
- Use Gateway tools for data access, similar to the architecture described in the [Bedrock Agent integration example](03-integrations/bedrock-agent/README.md).

### Integration with Frameworks
- Adapt existing frameworks such as LangGraph or CrewAI by wrapping them with the AgentCore SDK (see `03-integrations/agentic-frameworks/langgraph/README.md`).
- Ensure all agents communicate via MCP-compatible endpoints.

## 3. Programmatic Deployment Workflow

1. **Environment Setup**
   ```bash
   uv venv --python 3.10
   source .venv/bin/activate
   uv pip install -r requirements.txt
   ```
2. **Configure Tools and Memory**
   ```bash
   python scripts/agentcore_gateway.py create --name analytics-gateway
   python scripts/agentcore_memory.py create --name analytics-memory
   ```
3. **Register Agents**
   ```bash
   agentcore configure --entrypoint orchestrator.py -er <IAM role ARN> --name analytics-orchestrator
   agentcore launch
   ```
4. **Deploy Sub-Agents** in a similar fashion, specifying their entrypoints and roles.
5. **Observability** – enable metrics and tracing via AgentCore Observability during deployment.
6. **Invoke MAS**
   ```bash
   agentcore invoke '{"prompt": "Analyze sales data from last quarter"}'
   ```

### Embedding in Cloud Environment
- Use the Gateway endpoint to expose agent APIs to other AWS services.
- Store credentials in AWS Secrets Manager and manage configuration with AWS Systems Manager Parameter Store.

## 4. Task Breakdown Checklist
1. Review sample agent logic in `02-use-cases` for reference.
2. Define the workflow for extraction → cleaning → analysis → visualization.
3. Design individual agents with clear input/output schemas.
4. Configure tools (Browser, Code Interpreter) and set up Memory.
5. Write orchestration logic to call each agent in sequence.
6. Deploy using the AgentCore SDK commands listed above.
7. Integrate Observability for debugging and tracing.
8. Test end-to-end, iterate on prompts, and scale as needed.

---
This plan references the repository tutorials and integration examples for creating a multi-agent analytics solution on Amazon Bedrock AgentCore.
