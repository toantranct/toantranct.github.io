---
title: "Xây Dựng Agent Đầu Tiên: Instructions, Tools và Function Calling"
date: 2025-05-21 09:00:00 +0700
categories: [AI, OpenAI]
tags: [openai, agents, tools, function-calling, python]
author: toantranct
description: Hướng dẫn chi tiết cách viết instructions hiệu quả và tạo function tools cho OpenAI Agents. Từ basic function calling đến advanced tool patterns.
---

Trong bài trước, chúng ta đã làm quen với những khái niệm cơ bản của OpenAI Agents SDK. Nhưng để tạo ra những agents thực sự hữu ích, chúng ta cần trang bị cho chúng khả năng **thực hiện hành động** thông qua tools và **hiểu rõ nhiệm vụ** thông qua instructions hiệu quả.

Hôm nay, chúng ta sẽ khám phá cách biến agents từ những "chatbot đơn thuần" thành những "digital workers" có thể tương tác với APIs, xử lý dữ liệu, và thực hiện các tác vụ phức tạp trong thế giới thực.

## Khái niệm cơ bản

### Instructions

**Instructions (System prompt)** là tập hợp các hướng dẫn định nghĩa nhiệm vụ, vai trò, và giới hạn cho một Agent. Instructions giúp Agent hiểu rõ mục tiêu và cách thức phản hồi, từ đó đảm bảo kết quả nhất quán và chính xác hơn.

### Tools

**Tools** là các chức năng hoặc phương thức được cung cấp để Agent tương tác với thế giới bên ngoài như lấy thông tin, xử lý dữ liệu, hay thực hiện các hành động cụ thể. Ví dụ: lấy thông tin thời tiết, gửi email, tính toán dữ liệu…

## Tại Sao Instructions và Tools Quan Trọng?

### Instructions - Bộ Não Của Agent

Instructions (hay system prompt) không chỉ là "hướng dẫn sử dụng" mà còn là **DNA** của agent - định nghĩa personality, capabilities, và behavior patterns. Một instruction tốt có thể:

- **Định hướng hành vi**: Agent sẽ phản ứng như thế nào trong các tình huống khác nhau
- **Thiết lập context**: Vai trò, expertise, và domain knowledge
- **Đặt ra boundaries**: Những gì agent nên và không nên làm
- **Cải thiện accuracy**: Giảm hallucination và tăng tính nhất quán

### Tools - Đôi Tay Của Agent

Tools cho phép agents:
- **Truy cập dữ liệu real-time**: Weather, news, stock prices
- **Tương tác với services**: Send emails, create calendar events
- **Xử lý files**: Read/write documents, analyze spreadsheets  
- **Thực hiện calculations**: Complex math, data analysis
- **Control systems**: Automate workflows, manage infrastructure

## Viết Instructions Hiệu Quả

### Cấu Trúc Instructions Tối Ưu

```python
def create_expert_instructions(domain: str, role: str, constraints: list = None) -> str:
    """Template tạo instructions chuyên nghiệp"""
    
    constraints = constraints or []
    constraints_text = "\n".join([f"- {c}" for c in constraints])
    
    return f"""
# Role & Identity
You are a {role} specializing in {domain}.

# Core Responsibilities  
- Provide accurate, actionable advice
- Ask clarifying questions when requests are ambiguous
- Explain your reasoning step-by-step
- Admit when you're uncertain about something

# Communication Style
- Be professional yet approachable
- Use clear, concise language
- Provide examples when helpful
- Structure responses with headers and bullet points

# Constraints & Guidelines
{constraints_text}

# Quality Standards
- Always double-check facts and calculations
- Cite sources when making specific claims
- Focus on practical, implementable solutions
- Consider potential risks and alternatives
"""

# Ví dụ sử dụng
financial_advisor_instructions = create_expert_instructions(
    domain="personal finance and investment",
    role="Senior Financial Advisor",
    constraints=[
        "Never provide specific stock recommendations",
        "Always mention risks associated with investments",
        "Suggest consulting with certified professionals for major decisions",
        "Focus on educational content rather than direct advice"
    ]
)
```

### Dynamic Instructions - Thông Minh Hơn Với Context

```python
from datetime import datetime
from agents import Agent, RunContextWrapper

def dynamic_market_instructions(
    context: RunContextWrapper, 
    agent: Agent
) -> str:
    """Instructions thay đổi theo thời gian và context"""
    
    current_time = datetime.now()
    market_hours = 9 <= current_time.hour <= 16
    is_weekend = current_time.weekday() >= 5
    
    base_instructions = """
    You are a Financial Market Assistant with real-time market awareness.
    
    Your capabilities:
    - Analyze market trends and patterns
    - Explain financial concepts clearly
    - Provide educational market insights
    """
    
    # Thêm context về thời gian
    time_context = f"""
    
    # Current Context
    - Current time: {current_time.strftime('%Y-%m-%d %H:%M %Z')}
    - Market status: {'OPEN' if market_hours and not is_weekend else 'CLOSED'}
    """
    
    if market_hours and not is_weekend:
        time_context += """
    - Focus on real-time market movements
    - Mention that data might be delayed
    - Emphasize volatility during trading hours
    """
    else:
        time_context += """
    - Focus on market analysis and education
    - Mention markets are currently closed
    - Discuss preparation for next trading session
    """
        
    return base_instructions + time_context

# Sử dụng dynamic instructions
market_agent = Agent(
    name="Market Assistant",
    instructions=dynamic_market_instructions,
    # tools sẽ được thêm sau
)
```

### Best Practices Cho Instructions

**✅ DO - Những Điều Nên Làm:**

```python
good_instructions = """
# Role Definition
You are a Code Review Assistant for Python projects.

# Specific Tasks
1. Analyze code for bugs, security issues, and performance problems
2. Suggest improvements following PEP 8 and best practices
3. Explain the reasoning behind each suggestion
4. Provide code examples for recommended changes

# Response Format
For each file reviewed:
## Summary
- Overall code quality: [Score/10]
- Main issues found: [count]

## Detailed Feedback
### Critical Issues (Security/Bugs)
- [List critical issues with line numbers]

### Improvements (Performance/Style)  
- [List improvement suggestions]

### Positive Aspects
- [Highlight good practices found]

# Tone & Style
- Be constructive and encouraging
- Focus on education, not just criticism
- Provide rationale for suggestions
- Use technical terms appropriately
"""
```

**❌ DON'T - Những Điều Tránh:**

```python
bad_instructions = """
Help with code.  # Too vague
Be smart.         # Meaningless
Don't be wrong.   # Negative framing
Do everything.    # Unrealistic scope
"""
```

## Function Tools - Sức Mạnh Thật Sự

### Cách Hoạt Động Của Function Tools

OpenAI Agents SDK tự động:
1. **Extract function signature** từ Python function
2. **Generate JSON schema** cho parameters
3. **Parse docstring** để lấy descriptions
4. **Handle type validation** với Pydantic
5. **Execute function** khi LLM gọi tool

### Basic Function Tools

```python
import requests
from typing import Dict, Any
from agents import Agent, Runner, function_tool

@function_tool
def get_weather(city: str, country_code: str = "US") -> str:
    """Get current weather information for a specific city.
    
    Args:
        city: Name of the city (e.g., "New York", "London")
        country_code: ISO country code (e.g., "US", "UK", "VN")
        
    Returns:
        Weather information as a formatted string
    """
    # Demo implementation - trong thực tế sẽ call real weather API
    try:
        # Mock weather data
        weather_data = {
            "temperature": 22,
            "condition": "Sunny",
            "humidity": 65,
            "wind_speed": 10
        }
        
        return f"""
Weather in {city}, {country_code}:
🌡️ Temperature: {weather_data['temperature']}°C
☀️ Condition: {weather_data['condition']}
💧 Humidity: {weather_data['humidity']}%
💨 Wind Speed: {weather_data['wind_speed']} km/h
        """.strip()
        
    except Exception as e:
        return f"Sorry, I couldn't fetch weather data for {city}. Error: {str(e)}"

@function_tool
def calculate_compound_interest(
    principal: float, 
    annual_rate: float, 
    years: int,
    compound_frequency: int = 12
) -> Dict[str, Any]:
    """Calculate compound interest over time.
    
    Args:
        principal: Initial investment amount in dollars
        annual_rate: Annual interest rate as decimal (e.g., 0.05 for 5%)
        years: Number of years to calculate
        compound_frequency: How many times per year interest compounds (default: 12 for monthly)
        
    Returns:
        Dictionary with calculation results
    """
    if principal <= 0 or annual_rate < 0 or years <= 0:
        return {"error": "Invalid input parameters"}
        
    # A = P(1 + r/n)^(nt)
    final_amount = principal * (1 + annual_rate/compound_frequency) ** (compound_frequency * years)
    total_interest = final_amount - principal
    
    return {
        "principal": principal,
        "annual_rate": annual_rate * 100,  # Convert to percentage
        "years": years,
        "compound_frequency": compound_frequency,
        "final_amount": round(final_amount, 2),
        "total_interest": round(total_interest, 2),
        "growth_percentage": round((total_interest / principal) * 100, 2)
    }

# Tạo agent với tools
weather_finance_agent = Agent(
    name="Weather & Finance Assistant",
    instructions="""
    You are a helpful assistant that can provide weather information and financial calculations.
    
    Capabilities:
    - Get current weather for any city
    - Calculate compound interest for investments
    
    When providing financial calculations:
    - Always explain the assumptions used
    - Mention that this is for educational purposes
    - Suggest consulting with financial advisors for actual investment decisions
    
    Format your responses clearly with appropriate emojis and structure.
    """,
    tools=[get_weather, calculate_compound_interest]
)

# Test the agent
async def test_tools():
    # Weather query
    result1 = await Runner.run(
        weather_finance_agent,
        "What's the weather like in Hanoi, Vietnam?"
    )
    print("Weather Query Result:")
    print(result1.final_output)
    print("\n" + "="*50 + "\n")
    
    # Finance query  
    result2 = await Runner.run(
        weather_finance_agent,
        "If I invest $10,000 at 7% annual interest for 20 years, how much will I have?"
    )
    print("Finance Query Result:")
    print(result2.final_output)

if __name__ == "__main__":
    import asyncio
    asyncio.run(test_tools())
```

### Advanced Function Tools với Context

```python
from dataclasses import dataclass
from typing import List, Optional
from agents import RunContextWrapper, function_tool

@dataclass
class UserProfile:
    user_id: str
    name: str
    email: str
    preferences: Dict[str, Any]
    subscription_level: str = "basic"

@function_tool
async def get_user_preferences(
    ctx: RunContextWrapper[UserProfile], 
    category: str
) -> Dict[str, Any]:
    """Get user preferences for a specific category.
    
    Args:
        category: Category of preferences (e.g., 'notifications', 'display', 'privacy')
        
    Returns:
        User preferences for the specified category
    """
    user = ctx.context
    
    # Simulate database lookup
    all_preferences = {
        "notifications": {
            "email": user.subscription_level != "basic",
            "push": True,
            "frequency": "daily" if user.subscription_level == "premium" else "weekly"
        },
        "display": {
            "theme": "dark",
            "language": "en",
            "timezone": "Asia/Ho_Chi_Minh"
        },
        "privacy": {
            "analytics": user.subscription_level == "premium",
            "data_sharing": False
        }
    }
    
    return {
        "user_id": user.user_id,
        "category": category,
        "preferences": all_preferences.get(category, {}),
        "subscription_level": user.subscription_level
    }

@function_tool
async def update_user_preferences(
    ctx: RunContextWrapper[UserProfile],
    category: str,
    updates: Dict[str, Any]
) -> str:
    """Update user preferences for a specific category.
    
    Args:
        category: Category to update
        updates: Dictionary of preference updates
        
    Returns:
        Confirmation message
    """
    user = ctx.context
    
    # Validate subscription level for premium features
    if category == "notifications" and updates.get("frequency") == "realtime":
        if user.subscription_level != "premium":
            return "Real-time notifications require a Premium subscription. Please upgrade to access this feature."
    
    # Simulate database update
    print(f"Updating {category} preferences for user {user.name}")
    print(f"Updates: {updates}")
    
    return f"Successfully updated {category} preferences for {user.name}. Changes will take effect immediately."

# Context-aware agent
personalized_agent = Agent[UserProfile](
    name="Personal Settings Assistant",
    instructions="""
    You are a personal settings assistant that helps users manage their account preferences.
    
    You have access to the user's profile and can:
    - Retrieve current preferences by category
    - Update preferences based on user requests
    - Explain subscription-level restrictions
    
    Always:
    - Address the user by name
    - Explain what changes were made
    - Mention subscription limitations when relevant
    - Ask for confirmation before making changes
    """,
    tools=[get_user_preferences, update_user_preferences]
)

# Usage example
async def demo_context_tools():
    # Create user profile
    user = UserProfile(
        user_id="user_123",
        name="Alice",
        email="alice@example.com",
        preferences={},
        subscription_level="basic"
    )
    
    # Test the agent
    result = await Runner.run(
        personalized_agent,
        "Can you show me my notification settings and help me enable real-time notifications?",
        context=user
    )
    
    print(result.final_output)

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_context_tools())
```

### Custom Function Tools - Kiểm Soát Hoàn Toàn

```python
from pydantic import BaseModel
from agents import FunctionTool, RunContextWrapper

class DatabaseQuery(BaseModel):
    table: str
    where_clause: Optional[str] = None
    limit: int = 10

async def execute_safe_query(ctx: RunContextWrapper, args: str) -> str:
    """Execute a database query with safety checks"""
    try:
        query_params = DatabaseQuery.model_validate_json(args)
        
        # Safety checks
        dangerous_keywords = ['DROP', 'DELETE', 'UPDATE', 'INSERT', 'TRUNCATE']
        if query_params.where_clause:
            if any(keyword in query_params.where_clause.upper() for keyword in dangerous_keywords):
                return "Error: Dangerous SQL keywords detected. Only SELECT queries are allowed."
        
        # Simulate database query
        mock_results = [
            {"id": 1, "name": "Product A", "price": 29.99},
            {"id": 2, "name": "Product B", "price": 49.99},
        ]
        
        return f"Query Results from {query_params.table}:\n" + \
               "\n".join([str(row) for row in mock_results[:query_params.limit]])
               
    except Exception as e:
        return f"Query execution error: {str(e)}"

# Create custom tool
database_tool = FunctionTool(
    name="query_database",
    description="Execute safe SELECT queries on the database",
    params_json_schema=DatabaseQuery.model_json_schema(),
    on_invoke_tool=execute_safe_query,
)

# Agent with custom tool
database_agent = Agent(
    name="Database Assistant",
    instructions="""
    You are a database assistant that can help users query data safely.
    
    Security restrictions:
    - Only SELECT queries are allowed
    - No modification operations (INSERT, UPDATE, DELETE, DROP)
    - Query results are limited to prevent overload
    
    Always explain what data you're retrieving and why.
    """,
    tools=[database_tool]
)
```

### Error Handling trong Function Tools

```python
from agents import function_tool

@function_tool(failure_error_function=None)  # Re-raise errors for custom handling
def risky_api_call(endpoint: str) -> str:
    """Call external API with proper error handling.
    
    Args:
        endpoint: API endpoint to call
        
    Returns:
        API response data
    """
    try:
        # Simulate API call
        if "invalid" in endpoint:
            raise ValueError("Invalid endpoint format")
            
        if "timeout" in endpoint:
            raise TimeoutError("API request timed out")
            
        return f"Success: Data from {endpoint}"
        
    except ValueError as e:
        return f"Validation Error: {str(e)}"
    except TimeoutError as e:
        return f"Timeout Error: {str(e)}. Please try again later."
    except Exception as e:
        return f"Unexpected Error: {str(e)}"

def custom_error_handler(error: Exception, tool_name: str, args: str) -> str:
    """Custom error handler for tools"""
    return f"The {tool_name} tool encountered an issue. Please try rephrasing your request or contact support if the problem persists."

@function_tool(failure_error_function=custom_error_handler)
def another_tool(data: str) -> str:
    """Another tool with custom error handling"""
    if not data:
        raise ValueError("Data cannot be empty")
    return f"Processed: {data}"
```

## Real-World Example: Smart Task Manager

Hãy xây dựng một task manager thông minh với đầy đủ tính năng:

```python
import json
from datetime import datetime, timedelta
from typing import List, Dict, Optional
from enum import Enum
from pydantic import BaseModel
from agents import Agent, Runner, function_tool

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    URGENT = "urgent"

class TaskStatus(str, Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    COMPLETED = "completed"
    CANCELLED = "cancelled"

class Task(BaseModel):
    id: str
    title: str
    description: Optional[str] = None
    priority: Priority = Priority.MEDIUM
    status: TaskStatus = TaskStatus.PENDING
    due_date: Optional[str] = None
    created_at: str
    tags: List[str] = []

# In-memory storage (trong production sẽ dùng database)
tasks_storage: Dict[str, Task] = {}

@function_tool
def create_task(
    title: str,
    description: str = "",
    priority: str = "medium",
    due_date: str = "",
    tags: str = ""
) -> str:
    """Create a new task.
    
    Args:
        title: Task title
        description: Detailed description
        priority: Task priority (low, medium, high, urgent)
        due_date: Due date in YYYY-MM-DD format
        tags: Comma-separated tags
        
    Returns:
        Confirmation message with task ID
    """
    try:
        task_id = f"task_{len(tasks_storage) + 1}"
        tag_list = [tag.strip() for tag in tags.split(",") if tag.strip()]
        
        task = Task(
            id=task_id,
            title=title,
            description=description,
            priority=Priority(priority.lower()),
            due_date=due_date if due_date else None,
            created_at=datetime.now().isoformat(),
            tags=tag_list
        )
        
        tasks_storage[task_id] = task
        
        return f"""✅ Task created successfully!
        
**Task ID:** {task_id}
**Title:** {title}
**Priority:** {priority.upper()}
**Due Date:** {due_date or 'Not set'}
**Tags:** {', '.join(tag_list) if tag_list else 'None'}

You can reference this task using ID: {task_id}"""

    except Exception as e:
        return f"❌ Error creating task: {str(e)}"

@function_tool
def list_tasks(
    status: str = "all",
    priority: str = "all",
    limit: int = 10
) -> str:
    """List tasks with optional filtering.
    
    Args:
        status: Filter by status (all, pending, in_progress, completed, cancelled)
        priority: Filter by priority (all, low, medium, high, urgent)
        limit: Maximum number of tasks to return
        
    Returns:
        Formatted list of tasks
    """
    try:
        filtered_tasks = list(tasks_storage.values())
        
        # Filter by status
        if status != "all":
            filtered_tasks = [t for t in filtered_tasks if t.status.value == status]
            
        # Filter by priority
        if priority != "all":
            filtered_tasks = [t for t in filtered_tasks if t.priority.value == priority]
            
        # Sort by priority and due date
        priority_order = {"urgent": 4, "high": 3, "medium": 2, "low": 1}
        filtered_tasks.sort(
            key=lambda t: (
                priority_order.get(t.priority.value, 0),
                t.due_date or "9999-12-31"
            ),
            reverse=True
        )
        
        if not filtered_tasks:
            return "📝 No tasks found matching your criteria."
            
        result = f"📋 **Task List** (showing {min(len(filtered_tasks), limit)} of {len(filtered_tasks)} tasks)\n\n"
        
        for task in filtered_tasks[:limit]:
            priority_emoji = {
                "urgent": "🔥",
                "high": "⚡",
                "medium": "📝",
                "low": "💤"
            }
            
            status_emoji = {
                "pending": "⏳",
                "in_progress": "🔄",
                "completed": "✅",
                "cancelled": "❌"
            }
            
            due_info = f" (Due: {task.due_date})" if task.due_date else ""
            tags_info = f" #{', #'.join(task.tags)}" if task.tags else ""
            
            result += f"{priority_emoji.get(task.priority.value, '📝')} **{task.title}** [{task.id}]\n"
            result += f"{status_emoji.get(task.status.value, '⏳')} {task.status.value.replace('_', ' ').title()}{due_info}\n"
            if task.description:
                result += f"💭 {task.description}\n"
            if tags_info:
                result += f"🏷️ {tags_info}\n"
            result += "\n"
            
        return result.strip()
        
    except Exception as e:
        return f"❌ Error listing tasks: {str(e)}"

@function_tool
def update_task_status(task_id: str, new_status: str) -> str:
    """Update task status.
    
    Args:
        task_id: Task ID to update
        new_status: New status (pending, in_progress, completed, cancelled)
        
    Returns:
        Confirmation message
    """
    try:
        if task_id not in tasks_storage:
            return f"❌ Task {task_id} not found."
            
        old_status = tasks_storage[task_id].status.value
        tasks_storage[task_id].status = TaskStatus(new_status.lower())
        
        status_emoji = {
            "pending": "⏳",
            "in_progress": "🔄", 
            "completed": "✅",
            "cancelled": "❌"
        }
        
        return f"{status_emoji.get(new_status.lower(), '📝')} Task **{tasks_storage[task_id].title}** status updated from {old_status} to {new_status}!"
        
    except Exception as e:
        return f"❌ Error updating task: {str(e)}"

@function_tool
def get_task_analytics() -> str:
    """Get task analytics and insights.
    
    Returns:
        Task statistics and insights
    """
    try:
        if not tasks_storage:
            return "📊 No tasks available for analysis."
            
        total_tasks = len(tasks_storage)
        status_counts = {}
        priority_counts = {}
        overdue_count = 0
        
        today = datetime.now().date()
        
        for task in tasks_storage.values():
            # Count by status
            status_counts[task.status.value] = status_counts.get(task.status.value, 0) + 1
            
            # Count by priority
            priority_counts[task.priority.value] = priority_counts.get(task.priority.value, 0) + 1
            
            # Check overdue
            if task.due_date and task.status != TaskStatus.COMPLETED:
                due_date = datetime.strptime(task.due_date, "%Y-%m-%d").date()
                if due_date < today:
                    overdue_count += 1
                    
        completion_rate = (status_counts.get("completed", 0) / total_tasks) * 100
        
        result = f"""📊 **Task Analytics**

**Overview:**
• Total Tasks: {total_tasks}
• Completion Rate: {completion_rate:.1f}%
• Overdue Tasks: {overdue_count}

**By Status:**
"""
        for status, count in status_counts.items():
            percentage = (count / total_tasks) * 100
            result += f"• {status.replace('_', ' ').title()}: {count} ({percentage:.1f}%)\n"
            
        result += "\n**By Priority:**\n"
        for priority, count in priority_counts.items():
            percentage = (count / total_tasks) * 100
            result += f"• {priority.title()}: {count} ({percentage:.1f}%)\n"
            
        # Insights
        result += "\n**💡 Insights:**\n"
        if overdue_count > 0:
            result += f"• ⚠️ You have {overdue_count} overdue task(s) that need attention\n"
        if priority_counts.get("urgent", 0) > 2:
            result += f"• 🔥 High number of urgent tasks - consider prioritizing\n"
        if completion_rate < 50:
            result += f"• 📈 Low completion rate - try breaking down large tasks\n"
        else:
            result += f"• 🎉 Good job! You're maintaining a healthy completion rate\n"
            
        return result
        
    except Exception as e:
        return f"❌ Error generating analytics: {str(e)}"

# Create the Smart Task Manager Agent
task_manager_agent = Agent(
    name="Smart Task Manager",
    instructions="""
    You are an intelligent task management assistant that helps users organize and track their work efficiently.
    
    # Your Capabilities
    - Create new tasks with proper categorization
    - List and filter existing tasks  
    - Update task statuses
    - Provide analytics and insights
    
    # Communication Style
    - Be proactive in suggesting task organization improvements
    - Use emojis and formatting to make responses visually appealing
    - Provide actionable insights based on task data
    - Ask clarifying questions when task details are incomplete
    
    # Best Practices
    - Suggest appropriate priorities based on due dates and descriptions
    - Recommend breaking down large tasks into smaller ones
    - Alert users about overdue tasks
    - Celebrate completed milestones
    
    # When creating tasks:
    - Ask about due dates if not provided
    - Suggest relevant tags based on task content
    - Recommend appropriate priority levels
    
    # When providing analytics:
    - Give actionable insights
    - Highlight both achievements and areas for improvement
    - Suggest specific next actions
    """,
    tools=[create_task, list_tasks, update_task_status, get_task_analytics]
)

# Demo the Smart Task Manager
async def demo_task_manager():
    print("🚀 Smart Task Manager Demo\n")
    
    # Create some sample tasks
    queries = [
        "Create a task to 'Review Q4 budget proposal' with high priority, due 2025-02-01, tags: finance, budget",
        "Add a task 'Buy groceries' with medium priority and tag: personal",
        "Create an urgent task 'Fix production bug in payment system' due tomorrow",
        "Show me all my tasks",
        "Mark the payment system bug as in progress",
        "Give me task analytics and insights"
    ]
    
    for i, query in enumerate(queries, 1):
        print(f"**Query {i}:** {query}")
        result = await Runner.run(task_manager_agent, query)
        print(f"**Response:**\n{result.final_output}\n")
        print("-" * 80 + "\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_task_manager())
```

## Tổng Kết và Best Practices

### Những Điều Cần Ghi Nhớ

✅ **Instructions là foundation** - Đầu tư thời gian viết instructions chi tiết và rõ ràng  
✅ **Tools mở rộng capabilities** - Function tools cho phép agents tương tác với thế giới thực  
✅ **Type hints và docstrings** - SDK tự động generate schemas từ Python code  
✅ **Error handling** - Luôn handle errors gracefully trong tools  
✅ **Context awareness** - Sử dụng dynamic instructions và context-aware tools  

### Production Checklist

🔍 **Instructions Quality:**
- [ ] Vai trò và trách nhiệm được định nghĩa rõ ràng
- [ ] Phong cách giao tiếp được thiết lập
- [ ] Các giới hạn và hạn chế được nêu rõ
- [ ] Tiêu chuẩn chất lượng được xác định

🛠️ **Tools Development:**
- [ ] Function signatures có type hints đầy đủ
- [ ] Docstrings mô tả parameters và return values
- [ ] Error handling toàn diện
- [ ] Input validation và sanitization
- [ ] Tối ưu hiệu suất cho tools thường dùng

### Bước Tiếp Theo

Trong **bài tiếp theo**, chúng ta sẽ khám phá:

🎯 **Context Management** - Quản lý data và dependencies hiệu quả  
📊 **State Management** - Duy trì state xuyên suốt conversations  
🔄 **Data Flow Patterns** - Best practices cho data handling  

### Thử Thách Cho Bạn

Trước khi chuyển bài tiếp theo:

1. **Extend Task Manager** với reminder notifications
2. **Create domain-specific agent** cho work của bạn
3. **Experiment với different instruction styles** và measure effectiveness

---

*Bài tiếp theo: [**"Context Management: Quản Lý Dữ Liệu và Dependencies Trong Agents"**](../openai-agents-sdk-part-03-context-management-data-dependencies/) - Chúng ta sẽ học cách quản lý state, data flow, và dependencies để xây dựng agents thực sự thông minh.*