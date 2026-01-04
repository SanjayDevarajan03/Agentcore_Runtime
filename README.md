# AWS Bedrock AgentCore FAQ Assistant

A production-ready FAQ assistant built with AWS Bedrock AgentCore, LangChain, and LangGraph. This project demonstrates how to deploy intelligent agents with RAG (Retrieval-Augmented Generation) capabilities and persistent memory on AWS infrastructure.

## 🚀 Features

- **Vector-based FAQ Search**: Semantic search over FAQ knowledge base using FAISS and HuggingFace embeddings
- **Multi-Agent Architecture**: Two deployment variants - basic runtime and memory-enabled runtime
- **Persistent Memory**: AgentCore memory integration for short-term and long-term memory (STM & LTM)
- **Tool-Based Agents**: LangChain agents with custom tools for FAQ search and query reformulation
- **AWS Bedrock Integration**: Deployed on AWS Bedrock AgentCore runtime with container-based deployment
- **Observability**: CloudWatch integration for logs and monitoring

## 📁 Project Structure

```
AgentCore/
├── agentcore_runtime.py       # Basic runtime without memory
├── agentcore_memory.py        # Runtime with AgentCore memory integration
├── langgraph_agent.py         # Local testing script
├── lauki_qna.csv              # FAQ knowledge base (question-answer pairs)
├── .bedrock_agentcore.yaml    # AgentCore deployment configuration
├── pyproject.toml             # Python dependencies
└── README.md                  # This file
```

## 🏗️ Architecture

### Agent Variants

#### 1. `agentcore_runtime.py` - Basic Runtime
- Simple FAQ assistant without persistent memory
- Stateless agent for basic question-answering
- Uses Groq API for LLM inference
- FAISS vector store for semantic search

#### 2. `agentcore_memory.py` - Memory-Enabled Runtime
- Enhanced version with AgentCore memory integration
- Supports short-term memory (STM) and long-term memory (LTM)
- Session-aware with actor_id and thread_id tracking
- Custom middleware for memory management

### Components

- **LLM**: Groq API with `openai/gpt-oss-20b` model
- **Embeddings**: HuggingFace `sentence-transformers/all-MiniLM-L6-v2`
- **Vector Store**: FAISS for efficient similarity search
- **Agent Framework**: LangChain agents with tool calling
- **Deployment**: AWS Bedrock AgentCore runtime (container-based)

## 🛠️ Tools

The agents have access to three tools:

1. **`search_faq`**: Basic FAQ search (returns top 3 results)
2. **`search_detailed_faq`**: Extended search (configurable number of results)
3. **`reformulate_query`**: Query reformulation for multi-aspect searches

## 📋 Prerequisites

- Python 3.13+
- AWS Account with Bedrock AgentCore access
- AWS CLI configured with appropriate credentials
- Docker (for container builds)
- Groq API key

## ⚙️ Setup

### 1. Install Dependencies

```bash
# Using uv (recommended)
uv sync

# Or using pip
pip install -e .
```

### 2. Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
```

### 3. Prepare FAQ Data

Ensure `lauki_qna.csv` exists with the following format:

```csv
question,answer
"What is roaming activation?","Roaming activation allows..."
"How do I activate roaming?","To activate roaming..."
```

### 4. AWS Configuration

The project uses `.bedrock_agentcore.yaml` for deployment configuration. Key settings:

- **Region**: us-east-1
- **Deployment Type**: Container (Docker)
- **Platform**: linux/arm64
- **Network Mode**: PUBLIC
- **Observability**: Enabled (CloudWatch logs)

## 🚀 Deployment

### Deploy to AWS Bedrock AgentCore

```bash
# Deploy the basic runtime
agentcore deploy agentcore_runtime

# Deploy the memory-enabled runtime
agentcore deploy agentcore_memory
```

### Invoke the Agent

```bash
# Basic invocation
agentcore invoke '{"prompt": "Tell me about roaming activations"}'

# With memory context
agentcore invoke '{"prompt": "What did we discuss earlier?", "actor_id": "user123", "thread_id": "session456"}'
```

## 📊 Usage Examples

### Basic FAQ Query

```python
payload = {
    "prompt": "How do I activate roaming?"
}

response = agentcore.invoke(payload)
print(response["result"])
```

### Memory-Enabled Query

```python
payload = {
    "prompt": "What are the pricing details?",
    "actor_id": "customer-123",
    "thread_id": "session-456"
}

response = agentcore.invoke(payload)
# The agent will remember previous interactions in this session
```

## 🔧 Configuration

### Memory Configuration (agentcore_memory.py)

- **Memory Mode**: STM_AND_LTM (Short-Term and Long-Term Memory)
- **Memory ID**: Configured in the runtime file
- **Event Expiry**: 30 days (configurable)

### Agent Configuration

- **Model**: Groq `openai/gpt-oss-20b`
- **Temperature**: 0 (deterministic responses)
- **Embedding Model**: `sentence-transformers/all-MiniLM-L6-v2`
- **Chunk Size**: 500 characters
- **Search Results**: 3 (basic), 5 (detailed)

## 📝 API Response Format

### Basic Runtime Response

```json
{
  "result": "Answer to the user's question..."
}
```

### Memory Runtime Response

```json
{
  "result": "Answer to the user's question...",
  "actor_id": "user123",
  "thread_id": "session456"
}
```

## 🔍 Monitoring & Logs

View CloudWatch logs:

```bash
# Tail logs in real-time
aws logs tail /aws/bedrock-agentcore/runtimes/agentcore_memory-0w5gDqFoTJ-DEFAULT \
  --log-stream-name-prefix "2025/12/30/[runtime-logs]" \
  --follow

# View recent logs (last hour)
aws logs tail /aws/bedrock-agentcore/runtimes/agentcore_memory-0w5gDqFoTJ-DEFAULT \
  --log-stream-name-prefix "2025/12/30/[runtime-logs]" \
  --since 1h
```

Access the GenAI Dashboard:
- [AWS Console - GenAI Observability](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#gen-ai-observability/agent-core)

## 🧪 Local Testing

Test the agent locally using `langgraph_agent.py`:

```bash
python langgraph_agent.py
```

This runs a simple test query without deploying to AWS.

## 📚 Key Technologies

- **AWS Bedrock AgentCore**: Managed agent runtime platform
- **LangChain**: Framework for building LLM applications
- **LangGraph**: Stateful agent framework
- **FAISS**: Facebook AI Similarity Search for vector retrieval
- **HuggingFace Transformers**: Embedding models
- **Groq**: High-performance LLM inference API

## 🔐 Security Considerations

- API keys stored in environment variables (never commit `.env`)
- IAM roles configured for secure AWS access
- Network configuration set to PUBLIC (adjust for production)
- Container-based deployment for isolation

## 🐛 Troubleshooting

### Runtime Startup Errors

1. Check CloudWatch logs for detailed error messages
2. Verify environment variables are set correctly
3. Ensure FAQ CSV file exists and is readable
4. Check Docker is running for container builds

### Memory Issues

1. Verify memory ID is correct in configuration
2. Check IAM permissions for AgentCore memory access
3. Ensure memory mode is correctly set (STM_AND_LTM)

### Invocation Errors

1. Verify payload format matches expected structure
2. Check AWS credentials are configured
3. Ensure runtime is deployed and active


