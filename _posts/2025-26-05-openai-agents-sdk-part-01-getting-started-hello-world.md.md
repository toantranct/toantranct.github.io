---
title: "Khởi Đầu với OpenAI Agents SDK: Hello World và Những Điều Cần Biết"
date: 2025-05-20 09:00:00 +0700
categories: [AI, OpenAI]
tags: [openai, agents, sdk, python, ai, getting-started]
author: toantranct
description: Hướng dẫn chi tiết cách bắt đầu với OpenAI Agents SDK từ cài đặt đến tạo agent đầu tiên. Tìm hiểu các khái niệm cơ bản và viết code Hello World.
---

Thế giới AI đang phát triển với tốc độ chóng mặt, và việc xây dựng các ứng dụng AI thông minh không còn là điều xa vời. **OpenAI Agents SDK** chính là công cụ giúp bạn biến ý tưởng thành hiện thực một cách đơn giản và hiệu quả. Trong bài viết này, chúng ta sẽ cùng nhau khám phá từ những bước đầu tiên - từ cài đặt đến việc tạo ra agent AI đầu tiên của bạn.

Nếu bạn đã từng tò mò về cách các chatbot thông minh hoạt động, hay muốn tự tay xây dựng một assistant AI riêng, thì đây chính là điểm khởi đầu hoàn hảo!

## Tại Sao Chọn OpenAI Agents SDK?

### Triết Lý Thiết Kế

OpenAI Agents SDK được xây dựng dựa trên hai nguyên tắc cốt lõi:

1. **Đủ tính năng để đáng sử dụng, nhưng đủ đơn giản để học nhanh**
2. **Hoạt động tuyệt vời ngay từ đầu, nhưng có thể tùy chỉnh theo ý muốn**

Điều này có nghĩa là bạn có thể bắt đầu với những ví dụ đơn giản và nhanh chóng thấy kết quả, nhưng vẫn có khả năng mở rộng để xây dựng những ứng dụng phức tạp trong production.

### Những Tính Năng Nổi Bật

- **Agent Loop Tự Động**: SDK tự động xử lý việc gọi tools, gửi kết quả cho LLM, và lặp lại cho đến khi hoàn thành
- **Python-First**: Sử dụng các tính năng built-in của Python thay vì phải học thêm abstractions mới
- **Handoffs**: Tính năng mạnh mẽ cho phép các agents phối hợp và ủy quyền cho nhau
- **Guardrails**: Chạy input validation song song với agents
- **Function Tools**: Biến bất kỳ Python function nào thành tool với schema tự động
- **Tracing**: Built-in tracing để debug, visualize và monitor workflows

### So Sánh Với Các Giải Pháp Khác

OpenAI Agents SDK là bản nâng cấp production-ready của **Swarm**, với kiến trúc ổn định hơn và nhiều tính năng enterprise hơn. So với LangChain hay LlamaIndex, SDK này tập trung vào sự đơn giản và hiệu quả thay vì cung cấp quá nhiều abstractions.

## Cài Đặt và Thiết Lập Môi Trường

### Yêu Cầu Hệ Thống

Trước khi bắt đầu, hãy đảm bảo bạn có:
- **Python 3.8+** (khuyến nghị Python 3.10+)
- **OpenAI API Key** (tạo tại [platform.openai.com](https://platform.openai.com))
- **Kiến thức Python cơ bản** (functions, async/await, type hints)

### Bước 1: Tạo Project và Virtual Environment

```bash
# Tạo thư mục project
mkdir my-openai-agents
cd my-openai-agents

# Tạo virtual environment
python -m venv .venv

# Kích hoạt virtual environment
# Trên Linux/macOS:
source .venv/bin/activate
# Trên Windows:
# .venv\Scripts\activate
```

### Bước 2: Cài Đặt OpenAI Agents SDK

```bash
# Cài đặt package chính
pip install openai-agents

# Hoặc nếu bạn sử dụng uv (recommended):
uv add openai-agents

# Để sử dụng tính năng visualization (optional):
pip install "openai-agents[viz]"
```

### Bước 3: Thiết Lập API Key

Có 3 cách để cấu hình OpenAI API key:

**Cách 1: Environment Variable (Khuyến nghị)**
```bash
export OPENAI_API_KEY=sk-your-api-key-here
```

**Cách 2: Sử dụng Code**
```python
from agents import set_default_openai_key

set_default_openai_key("sk-your-api-key-here")
```

**Cách 3: Custom Client**
```python
from openai import AsyncOpenAI
from agents import set_default_openai_client

custom_client = AsyncOpenAI(
    api_key="sk-your-api-key-here",
    base_url="https://api.openai.com/v1"  # hoặc custom endpoint
)
set_default_openai_client(custom_client)
```

## Hello World - Agent Đầu Tiên

Hãy bắt đầu với ví dụ đơn giản nhất để hiểu cách SDK hoạt động:

### Ví Dụ Cơ Bản

```python
# hello_agent.py
from agents import Agent, Runner

# Tạo agent đầu tiên
agent = Agent(
    name="Assistant", 
    instructions="You are a helpful assistant"
)

# Chạy agent
result = Runner.run_sync(
    agent, 
    "Write a haiku about recursion in programming."
)

print(result.final_output)
```

**Kết quả:**
```
Code within the code,
Functions calling themselves,
Infinite loop's dance.
```

### Phân Tích Code

Hãy cùng phân tích từng phần:

1. **Import cần thiết**: `Agent` và `Runner` là hai class chính bạn sẽ sử dụng
2. **Tạo Agent**: 
   - `name`: Tên của agent (dùng cho tracing và debugging)
   - `instructions`: Hướng dẫn cho agent (tương đương system prompt)
3. **Chạy Agent**: `Runner.run_sync()` chạy đồng bộ và trả về kết quả

### Ví Dụ Async (Khuyến Nghị)

```python
# hello_agent_async.py
import asyncio
from agents import Agent, Runner

async def main():
    agent = Agent(
        name="Friendly Assistant",
        instructions="You are a friendly and helpful assistant. Always be polite and enthusiastic."
    )
    
    result = await Runner.run(
        agent, 
        "Explain what is artificial intelligence in simple terms."
    )
    
    print(f"Agent response: {result.final_output}")

if __name__ == "__main__":
    asyncio.run(main())
```

### Tùy Chỉnh Model và Settings

```python
from agents import Agent, Runner, ModelSettings

# Tạo agent với model settings tùy chỉnh
agent = Agent(
    name="Creative Writer",
    instructions="You are a creative writer who loves to tell stories.",
    model="gpt-4o",  # Specify model
    model_settings=ModelSettings(
        temperature=0.8,  # Tăng creativity
        max_tokens=500,   # Giới hạn độ dài response
    )
)

async def main():
    result = await Runner.run(
        agent,
        "Tell me a short story about a robot learning to paint."
    )
    print(result.final_output)

if __name__ == "__main__":
    asyncio.run(main())
```

## Hiểu Về Các Thành Phần Cơ Bản

### 1. Agent - Trái Tim của Hệ Thống

`Agent` là một LLM được trang bị instructions và tools. Các thuộc tính quan trọng:

```python
agent = Agent(
    name="Customer Support",           # Tên agent
    instructions="Help customers...",  # System prompt
    model="gpt-4o",                   # Model sử dụng
    tools=[],                         # Danh sách tools
    handoffs=[],                      # Agents khác có thể handoff đến
    model_settings=ModelSettings(...) # Cài đặt model
)
```

### 2. Runner - Điều Phối Viên

`Runner` chịu trách nhiệm chạy agent loop:

- **`Runner.run()`**: Async method, trả về `RunResult`
- **`Runner.run_sync()`**: Sync wrapper của `run()`
- **`Runner.run_streamed()`**: Streaming responses

### 3. Instructions - Bộ Não của Agent

Instructions là system prompt định nghĩa behavior của agent:

```python
# Static instructions
instructions = """
You are a helpful coding assistant.
- Always provide working code examples
- Explain your reasoning step by step
- Use Python 3.10+ syntax
- Include error handling when appropriate
"""

# Dynamic instructions (advanced)
def dynamic_instructions(context, agent):
    return f"Today is {datetime.now().date()}. Help the user with their questions."

agent = Agent(
    name="Code Helper",
    instructions=instructions  # or dynamic_instructions
)
```

## Best Practices Cho Beginners

### 1. Viết Instructions Hiệu Quả

**❌ Tệ:**
```python
instructions = "Help user"
```

**✅ Tốt:**
```python
instructions = """
You are a professional customer support agent for TechCorp.

Your responsibilities:
- Answer questions about products and services
- Be polite, helpful, and professional
- If you don't know something, say so clearly
- Always ask for clarification when requests are ambiguous

Communication style:
- Keep responses concise but comprehensive
- Use bullet points for multiple items
- Provide examples when helpful
"""
```

### 2. Error Handling

```python
import asyncio
from agents import Agent, Runner
from agents.exceptions import AgentsException

async def safe_agent_run():
    agent = Agent(
        name="Safe Agent",
        instructions="You are a helpful assistant."
    )
    
    try:
        result = await Runner.run(agent, "Hello!")
        return result.final_output
    except AgentsException as e:
        print(f"Agent error: {e}")
        return "Sorry, I encountered an error."
    except Exception as e:
        print(f"Unexpected error: {e}")
        return "Something went wrong."

if __name__ == "__main__":
    response = asyncio.run(safe_agent_run())
    print(response)
```

### 3. Logging và Debug

```python
from agents import enable_verbose_stdout_logging

# Enable detailed logging để debug
enable_verbose_stdout_logging()

# Hoặc custom logging
import logging

logger = logging.getLogger("openai.agents")
logger.setLevel(logging.INFO)
logger.addHandler(logging.StreamHandler())
```

### 4. Environment Configuration

Tạo file `.env` để quản lý config:

```bash
# .env
OPENAI_API_KEY=sk-your-key-here
OPENAI_AGENTS_DISABLE_TRACING=0
OPENAI_AGENTS_DONT_LOG_MODEL_DATA=0
```

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
if not OPENAI_API_KEY:
    raise ValueError("OPENAI_API_KEY environment variable is required")
```

## Ứng Dụng Thực Tế - Personal Assistant

Hãy xây dựng một personal assistant đơn giản để thấy được sức mạnh của SDK:

```python
# personal_assistant.py
import asyncio
from datetime import datetime
from agents import Agent, Runner

class PersonalAssistant:
    def __init__(self, user_name: str):
        self.user_name = user_name
        self.agent = Agent(
            name="Personal Assistant",
            instructions=f"""
            You are {user_name}'s personal assistant.
            
            Your personality:
            - Friendly and professional
            - Proactive and helpful
            - Remember context from previous conversations
            
            Current information:
            - Today is {datetime.now().strftime('%A, %B %d, %Y')}
            - Time: {datetime.now().strftime('%I:%M %p')}
            - User name: {user_name}
            
            Always address the user by name and be contextually aware.
            """
        )
    
    async def chat(self, message: str) -> str:
        """Send a message to the assistant"""
        try:
            result = await Runner.run(self.agent, message)
            return result.final_output
        except Exception as e:
            return f"I'm sorry, I encountered an error: {str(e)}"
    
    async def interactive_session(self):
        """Start an interactive chat session"""
        print(f"Hello {self.user_name}! I'm your personal assistant.")
        print("Type 'quit' to exit the conversation.\n")
        
        while True:
            user_input = input(f"{self.user_name}: ").strip()
            
            if user_input.lower() in ['quit', 'exit', 'bye']:
                print("Assistant: Goodbye! Have a great day!")
                break
                
            if not user_input:
                continue
                
            print("Assistant: ", end="", flush=True)
            response = await self.chat(user_input)
            print(response)
            print()

async def main():
    # Tạo assistant cho user
    assistant = PersonalAssistant("Alice")
    
    # Demo một vài interactions
    print("=== Demo Conversations ===")
    
    responses = [
        "What should I focus on today?",
        "Can you help me write a professional email?",
        "What's a good way to stay motivated?",
    ]
    
    for question in responses:
        print(f"Alice: {question}")
        answer = await assistant.chat(question)
        print(f"Assistant: {answer}\n")
    
    # Uncomment để chạy interactive session
    # await assistant.interactive_session()

if __name__ == "__main__":
    asyncio.run(main())
```

### Kết Quả Dự Kiến

```
=== Demo Conversations ===
Alice: What should I focus on today?
Assistant: Good morning Alice! For today, I'd suggest focusing on:

• **Priority tasks**: Start with your most important or time-sensitive work
• **Energy management**: Tackle challenging tasks when you're most alert
• **Communication**: Check and respond to important messages
• **Planning**: Take 10 minutes to review tomorrow's schedule

What's your main goal for today? I can help you break it down into actionable steps.

Alice: Can you help me write a professional email?
Assistant: Of course, Alice! I'd be happy to help you craft a professional email.

To write the most effective email, could you tell me:
• Who is the recipient?
• What's the main purpose/request?
• What tone are you aiming for (formal, friendly-professional, etc.)?
• Any specific details you want to include?

Once I have these details, I can help you structure a clear, professional message that gets results!
```

## Tổng Kết và Bước Tiếp Theo

Chúng ta đã cùng nhau khám phá những bước đầu tiên với OpenAI Agents SDK:

### Những Gì Bạn Đã Học Được

✅ **Cài đặt và cấu hình** môi trường phát triển  
✅ **Tạo agent đầu tiên** với Hello World example  
✅ **Hiểu các thành phần cơ bản**: Agent, Runner, Instructions  
✅ **Best practices** cho beginners  
✅ **Xây dựng ứng dụng thực tế** với Personal Assistant  

### Điểm Mạnh Của SDK

- **Đơn giản**: API rõ ràng, dễ hiểu
- **Mạnh mẽ**: Đủ tính năng cho production
- **Linh hoạt**: Có thể tùy chỉnh mọi aspect
- **Python-native**: Leverage toàn bộ ecosystem Python

### Bước Tiếp Theo

Trong **bài tiếp theo**, chúng ta sẽ đi sâu vào:

🔧 **Tools và Function Calling** - Cách làm cho agents có thể thực hiện actions  
📊 **Data Integration** - Kết nối với APIs, databases, và external services  
🎯 **Advanced Instructions** - Viết prompts hiệu quả cho specific use cases  

### Thử Thách Cho Bạn

Trước khi chuyển sang bài tiếp theo, hãy thử:

1. **Modify Personal Assistant** để phù hợp với workflow của bạn
2. **Experiment với different models** (gpt-3.5-turbo, gpt-4o, etc.)
3. **Create themed agents** (Code Reviewer, Creative Writer, Data Analyst)

## Tài Liệu Tham Khảo

### Official Documentation
- [OpenAI Agents SDK GitHub](https://github.com/openai/openai-agents-python)
- [OpenAI Platform Documentation](https://platform.openai.com/docs)
- [API Reference](https://docs.anthropic.com)

### Community Resources
- [OpenAI Community Forum](https://community.openai.com)
- [Example Projects](https://github.com/openai/openai-agents-python/tree/main/examples)
- [Stack Overflow - openai-agents tag](https://stackoverflow.com/questions/tagged/openai-agents)

### Tools và Utilities
- [Tracing Dashboard](https://platform.openai.com/traces)
- [Python Type Hints Guide](https://docs.python.org/3/library/typing.html)
- [Async Programming in Python](https://docs.python.org/3/library/asyncio.html)

---

*Bài viết tiếp theo: **"Xây Dựng Agent Đầu Tiên: Instructions, Tools và Function Calling"** - Chúng ta sẽ tìm hiểu cách làm cho agents thực sự hữu ích bằng cách trang bị tools và khả năng tương tác với thế giới bên ngoài.*