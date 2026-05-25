# LLM Gateway

A comprehensive LLM gateway built with LiteLLM and LangChain that provides multi-provider support, intelligent routing, caching, cost tracking, and load balancing for Large Language Models.

## Features

### 🔀 Multi-Provider Support
- **OpenAI** - GPT-4o Mini
- **Groq** - Llama 3.3 70B Versatile
- **Google Gemini** - Gemini 2.0 Flash & 1.5 Flash

Seamlessly switch between providers or use multiple providers in parallel.

### 💰 Cost Tracking
Track token usage and cost per request in multiple currencies (USD, CAD, etc.)
- Monitor prompt and completion tokens
- Real-time cost calculation
- Per-request cost analytics

### ⚡ Intelligent Caching
Reduce API costs by caching responses for identical queries
- Local caching support
- Significant speedup for repeated queries
- Transparent caching with minimal code changes

### 🔄 Load Balancing & Routing
Distribute requests across multiple models and providers
- Automatic fallback on quota limits or errors
- Task-aware routing (coding, summarization, general queries)
- Model list configuration with custom parameters
- Dynamic model selection based on task type

### 🔗 LangChain Integration
Build RAG pipelines and AI agents with integrated LLM support
- ChatLiteLLM wrapper for seamless LangChain compatibility
- Prompt templates and chain composition
- Output parsing

### 🛡️ Guardrails & Callbacks
Monitor and control request/response flow
- Input validation callbacks
- Success and failure callbacks
- Async callback support

## Installation

```bash
# Clone the repository
git clone <repository-url>
cd llmgateway

# Create virtual environment
python -m venv .venv
.venv\Scripts\activate  # On Windows

# Install dependencies
pip install litellm langchain-community python-dotenv google-generativeai langchain-litellm
```

## Setup

### Environment Variables

Create a `.env` file with your API keys:

```bash
OPENAI_API_KEY=your_openai_key_here
GROQ_API_KEY=your_groq_key_here
GOOGLE_API_KEY=your_google_api_key_here
```

Load environment variables in your code:

```python
from dotenv import load_dotenv
load_dotenv()
```

## Usage

### Basic Single Model Request

```python
from litellm import completion

response = completion(
    model="groq/llama-3.3-70b-versatile",
    messages=[
        {"role": "user", "content": "What is the capital of France?"}
    ]
)

print(response.choices[0].message.content)
```

### Multi-Provider Fallback

```python
from litellm import completion

models = [
    "gemini/gemini-2.0-flash",
    "groq/llama-3.3-70b-versatile",
    "openai/gpt-4o-mini"
]

for model in models:
    try:
        response = completion(
            model=model,
            messages=[
                {"role": "user", "content": "Your query here"}
            ]
        )
        print(f"✓ {model}: {response.choices[0].message.content}")
        break
    except Exception as e:
        print(f"✗ {model} failed: {e}")
```

### Cost Tracking

```python
from litellm import completion, completion_cost

response = completion(
    model="groq/llama-3.3-70b-versatile",
    messages=[
        {"role": "user", "content": "What is machine learning?"}
    ]
)

cost_usd = completion_cost(
    completion_response=response,
    model="groq/llama-3.3-70b-versatile"
)

print(f"Response: {response.choices[0].message.content}")
print(f"Cost (USD): ${cost_usd:.6f}")
print(f"Tokens: {response.usage.total_tokens}")
```

### Intelligent Caching

```python
import litellm
from litellm.caching import Cache

# Enable local cache
litellm.cache = Cache(type="local")

# First call hits API
response1 = completion(
    model="groq/llama-3.3-70b-versatile",
    messages=[{"role": "user", "content": "What is AI?"}],
    caching=True
)

# Second identical call returns cached result (much faster!)
response2 = completion(
    model="groq/llama-3.3-70b-versatile",
    messages=[{"role": "user", "content": "What is AI?"}],
    caching=True
)
```

### Task-Aware Routing

```python
from litellm import completion, completion_cost
import time

def classify_task(content):
    content = content.lower()
    if "code" in content or "function" in content:
        return "coding"
    elif "summarize" in content:
        return "summary"
    return "general"

def smart_chat(user_query):
    task = classify_task(user_query)
    
    if task == "coding":
        model = "groq/llama-3.3-70b-versatile"
    elif task == "summary":
        model = "openai/gpt-4o-mini"
    else:
        model = "gemini/gemini-2.0-flash"
    
    start = time.time()
    response = completion(
        model=model,
        messages=[{"role": "user", "content": user_query}]
    )
    latency = time.time() - start
    cost = completion_cost(completion_response=response, model=model)
    
    print(f"Task: {task}")
    print(f"Model: {model}")
    print(f"Latency: {latency:.2f}s")
    print(f"Cost: ${cost:.6f}")
    return response.choices[0].message.content

# Use it
answer = smart_chat("Write a Python function to sort a list")
```

### Router for Load Balancing

```python
from litellm import Router

model_list = [
    {
        "model_name": "fast-cheap",
        "litellm_params": {
            "model": "openai/gpt-4o-mini",
            "max_tokens": 1000
        }
    },
    {
        "model_name": "powerful",
        "litellm_params": {
            "model": "groq/llama-3.3-70b-versatile",
            "max_tokens": 2000
        }
    }
]

router = Router(model_list=model_list)

response = router.completion(
    model="fast-cheap",
    messages=[{"role": "user", "content": "Your query"}]
)
```

### LangChain Integration

```python
from langchain_litellm import ChatLiteLLM
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm = ChatLiteLLM(model="groq/llama-3.3-70b-versatile")

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "{question}")
])

chain = prompt | llm | StrOutputParser()

response = chain.invoke({"question": "What is machine learning?"})
print(response)
```

## Project Structure

```
llmgateway/
├── llm_gateway.ipynb       # Main notebook with examples
├── README.md               # This file
├── .env                    # Environment variables (not committed)
└── .venv/                  # Virtual environment
```

## Key Concepts

### Providers
Different LLM providers with different pricing, speed, and capabilities:
- **OpenAI**: High quality, higher cost
- **Groq**: Fast inference, good value
- **Google Gemini**: Latest models, competitive pricing

### Caching
Store responses for identical queries to reduce API calls and costs. Ideal for production systems where multiple users might ask the same questions.

### Load Balancing
Distribute requests across providers to:
- Reduce quota limits impact
- Optimize costs
- Improve response time
- Provide fallback options

### Task Routing
Automatically select the best model for the task:
- Coding tasks → Specialized models
- Summarization → Cost-effective models
- General queries → Balanced models

## Performance Tips

1. **Enable caching** for frequently asked questions
2. **Use cheaper models** for simple tasks (GPT-4o Mini, Llama)
3. **Implement fallback chains** for reliability
4. **Monitor costs** with completion_cost tracking
5. **Batch similar requests** to maximize cache hits

## Troubleshooting

### Quota Exceeded Errors
Use the multi-provider fallback pattern to automatically retry with alternative providers.

### High Costs
Enable caching and use cheaper models for simple queries.

### Slow Responses
Check provider latency with timing measurements and route to faster providers.

## References

- [LiteLLM Documentation](https://docs.litellm.ai/)
- [LangChain Documentation](https://python.langchain.com/)
- [OpenAI API](https://platform.openai.com/docs)
- [Groq API](https://console.groq.com/)
- [Google Gemini API](https://ai.google.dev/)