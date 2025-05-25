---
title: "Context Management: Quản Lý Dữ Liệu và Dependencies Trong Agents"
date: 2025-05-22 09:00:00 +0700
categories: [AI, OpenAI SDK]
tags: [openai, agents, context, data-management, dependencies]
author: toantranct
description: Hướng dẫn chi tiết về Context Management trong OpenAI Agents SDK. Từ Local Context đến LLM Context, Dynamic Instructions và Dependency Injection patterns.
---

Trong hai bài trước, chúng ta đã học cách tạo agents và trang bị tools cho chúng. Nhưng để xây dựng những ứng dụng thực sự mạnh mẽ, agents cần có khả năng **nhớ thông tin**, **truy cập dữ liệu** và **quản lý dependencies** một cách thông minh.

Context Management chính là chìa khóa để biến agents từ những "stateless chatbots" thành những "intelligent assistants" có thể duy trì conversations, truy cập user data, và thích ứng với từng tình huống cụ thể.

## Hiểu Về Context - Hai Thế Giới Song Song

### Local Context vs LLM Context

Một trong những khái niệm quan trọng nhất trong OpenAI Agents SDK là sự phân biệt giữa hai loại context:

**🔧 Local Context (RunContextWrapper)**
- Dữ liệu và dependencies có sẵn cho **code của bạn**
- Không được gửi đến LLM
- Dùng cho database connections, user sessions, API keys
- Quản lý state và shared data giữa các tool calls

**🧠 LLM Context (Conversation History)**  
- Dữ liệu mà **LLM có thể thấy** khi generate responses
- Bao gồm conversation history, instructions, tool results
- Cách duy nhất để LLM "biết" về thông tin mới
- Phải được inject thông qua instructions, input, hoặc tool results

```python
from dataclasses import dataclass
from typing import Dict, Any
from agents import Agent, RunContextWrapper, function_tool

@dataclass
class UserSession:
    """Local Context - LLM không thấy được"""
    user_id: str
    session_id: str
    database_connection: Any  # Database connection
    api_keys: Dict[str, str]  # Sensitive API keys
    user_preferences: Dict[str, Any]
    
    def get_user_data(self) -> Dict[str, Any]:
        """Fetch user data from database"""
        # LLM không thể trực tiếp access method này
        return {
            "name": "Alice Johnson",
            "email": "alice@example.com",
            "subscription": "premium"
        }

@function_tool
async def get_user_info(ctx: RunContextWrapper[UserSession]) -> str:
    """Tool để bridge Local Context với LLM Context"""
    user_data = ctx.context.get_user_data()
    
    # Transform Local Context data thành LLM-readable format
    return f"""
Current user information:
- Name: {user_data['name']}
- Email: {user_data['email']}
- Subscription: {user_data['subscription']}
- Session ID: {ctx.context.session_id}
"""

# Agent with context awareness
def create_personalized_instructions(
    ctx: RunContextWrapper[UserSession], 
    agent: Agent
) -> str:
    """Dynamic instructions dựa trên user context"""
    user_data = ctx.context.get_user_data()
    
    return f"""
You are a personalized assistant for {user_data['name']}.

User Profile:
- Subscription Level: {user_data['subscription']}
- Session: {ctx.context.session_id}

Adapt your responses based on their subscription level:
- Premium users: Provide detailed, advanced features
- Basic users: Offer upgrade suggestions when appropriate

Always address the user by name and be contextually aware.
"""

personalized_agent = Agent[UserSession](
    name="Personal Assistant",
    instructions=create_personalized_instructions,
    tools=[get_user_info]
)
```

## Local Context Patterns

### 1. Dependency Injection Pattern

```python
import asyncio
import aiohttp
from dataclasses import dataclass
from typing import Protocol
from agents import Agent, RunContextWrapper, function_tool

# Định nghĩa interfaces
class DatabaseService(Protocol):
    async def get_user(self, user_id: str) -> Dict[str, Any]: ...
    async def save_user(self, user_id: str, data: Dict[str, Any]) -> bool: ...

class NotificationService(Protocol):
    async def send_email(self, to: str, subject: str, body: str) -> bool: ...
    async def send_sms(self, phone: str, message: str) -> bool: ...

# Concrete implementations
class PostgreSQLService:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
    
    async def get_user(self, user_id: str) -> Dict[str, Any]:
        # Simulate database query
        return {
            "id": user_id,
            "name": "Alice Johnson",
            "email": "alice@example.com",
            "phone": "+84901234567"
        }
    
    async def save_user(self, user_id: str, data: Dict[str, Any]) -> bool:
        print(f"Saving user {user_id}: {data}")
        return True

class EmailService:
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    async def send_email(self, to: str, subject: str, body: str) -> bool:
        print(f"Sending email to {to}: {subject}")
        return True
    
    async def send_sms(self, phone: str, message: str) -> bool:
        print(f"Sending SMS to {phone}: {message}")
        return True

@dataclass
class ServiceContext:
    """Context chứa tất cả dependencies"""
    user_id: str
    db: DatabaseService
    notifications: NotificationService
    session_data: Dict[str, Any]
    
    async def log_activity(self, activity: str):
        """Helper method for logging"""
        print(f"User {self.user_id}: {activity}")

# Tools sử dụng injected dependencies
@function_tool
async def update_user_profile(
    ctx: RunContextWrapper[ServiceContext],
    field: str,
    value: str
) -> str:
    """Update user profile field.
    
    Args:
        field: Field to update (name, email, phone)
        value: New value for the field
    """
    try:
        # Get current user data
        user_data = await ctx.context.db.get_user(ctx.context.user_id)
        
        # Update field
        user_data[field] = value
        success = await ctx.context.db.save_user(ctx.context.user_id, user_data)
        
        if success:
            await ctx.context.log_activity(f"Updated {field} to {value}")
            
            # Send confirmation notification
            if field == "email":
                await ctx.context.notifications.send_email(
                    to=value,
                    subject="Profile Updated",
                    body=f"Your {field} has been successfully updated."
                )
            
            return f"✅ Successfully updated {field} to: {value}"
        else:
            return f"❌ Failed to update {field}"
            
    except Exception as e:
        return f"❌ Error updating profile: {str(e)}"

@function_tool
async def get_user_profile(ctx: RunContextWrapper[ServiceContext]) -> str:
    """Get current user profile information."""
    try:
        user_data = await ctx.context.db.get_user(ctx.context.user_id)
        await ctx.context.log_activity("Viewed profile")
        
        return f"""
📋 **Your Profile Information**

• **Name:** {user_data.get('name', 'Not set')}
• **Email:** {user_data.get('email', 'Not set')}  
• **Phone:** {user_data.get('phone', 'Not set')}
• **User ID:** {user_data.get('id', 'Unknown')}

Use the update_user_profile tool to modify any of these fields.
"""
    except Exception as e:
        return f"❌ Error retrieving profile: {str(e)}"

# Agent với dependency injection
profile_agent = Agent[ServiceContext](
    name="Profile Manager",
    instructions="""
    You are a user profile management assistant.
    
    Capabilities:
    - View current profile information
    - Update profile fields (name, email, phone)
    - Send confirmation notifications
    
    Always:
    - Confirm changes before making them
    - Validate email formats and phone numbers
    - Provide clear success/error messages
    - Log all activities for audit purposes
    """,
    tools=[get_user_profile, update_user_profile]
)

# Usage example
async def demo_dependency_injection():
    # Setup dependencies
    db_service = PostgreSQLService("postgresql://localhost/mydb")
    email_service = EmailService("email-api-key-123")
    
    # Create context with injected dependencies
    context = ServiceContext(
        user_id="user_456",
        db=db_service,
        notifications=email_service,
        session_data={"logged_in_at": "2025-01-27T10:00:00Z"}
    )
    
    # Test the agent
    queries = [
        "Show me my current profile",
        "Update my email to alice.new@example.com",
        "Change my name to Alice Smith"
    ]
    
    for query in queries:
        print(f"\n**Query:** {query}")
        result = await Runner.run(profile_agent, query, context=context)
        print(f"**Response:** {result.final_output}")

if __name__ == "__main__":
    from agents import Runner
    asyncio.run(demo_dependency_injection())
```

### 2. Session Management Pattern

```python
import uuid
from datetime import datetime, timedelta
from typing import Optional, List
from dataclasses import dataclass, field
from agents import Agent, RunContextWrapper, function_tool

@dataclass
class ConversationHistory:
    messages: List[Dict[str, Any]] = field(default_factory=list)
    topics: List[str] = field(default_factory=list)
    sentiment: str = "neutral"
    
    def add_message(self, role: str, content: str, metadata: Dict = None):
        self.messages.append({
            "role": role,
            "content": content,
            "timestamp": datetime.now().isoformat(),
            "metadata": metadata or {}
        })
    
    def get_recent_context(self, limit: int = 5) -> str:
        """Get recent conversation context for LLM"""
        recent = self.messages[-limit:] if self.messages else []
        if not recent:
            return "No previous conversation history."
            
        context = "Recent conversation context:\n"
        for msg in recent:
            context += f"- {msg['role']}: {msg['content'][:100]}...\n"
        return context

@dataclass  
class UserSession:
    session_id: str
    user_id: str
    created_at: datetime
    last_activity: datetime
    conversation: ConversationHistory = field(default_factory=ConversationHistory)
    user_preferences: Dict[str, Any] = field(default_factory=dict)
    temporary_data: Dict[str, Any] = field(default_factory=dict)
    
    def update_activity(self):
        self.last_activity = datetime.now()
    
    def is_expired(self, timeout_minutes: int = 30) -> bool:
        return datetime.now() - self.last_activity > timedelta(minutes=timeout_minutes)
    
    def get_session_summary(self) -> str:
        duration = datetime.now() - self.created_at
        return f"""
Session Summary:
- Session ID: {self.session_id}
- Duration: {duration}
- Messages: {len(self.conversation.messages)}
- Topics: {', '.join(self.conversation.topics)}
- Sentiment: {self.conversation.sentiment}
"""

# Global session storage (trong production dùng Redis/Database)
sessions: Dict[str, UserSession] = {}

@function_tool
async def remember_preference(
    ctx: RunContextWrapper[UserSession],
    category: str,
    preference: str
) -> str:
    """Remember user preference for future interactions.
    
    Args:
        category: Category of preference (communication_style, interests, etc.)
        preference: The preference to remember
    """
    ctx.context.user_preferences[category] = preference
    ctx.context.update_activity()
    
    return f"✅ I'll remember that you prefer {preference} for {category}."

@function_tool
async def recall_conversation(
    ctx: RunContextWrapper[UserSession],
    topic: str = ""
) -> str:
    """Recall previous conversation about a specific topic.
    
    Args:
        topic: Topic to recall (optional)
    """
    if not ctx.context.conversation.messages:
        return "We haven't had any previous conversations in this session."
    
    if topic:
        # Simple topic matching
        relevant_messages = [
            msg for msg in ctx.context.conversation.messages
            if topic.lower() in msg['content'].lower()
        ]
        
        if relevant_messages:
            context = f"Previous discussion about '{topic}':\n"
            for msg in relevant_messages[-3:]:  # Last 3 relevant messages
                context += f"- {msg['role']}: {msg['content'][:150]}...\n"
            return context
        else:
            return f"I don't recall discussing '{topic}' in our conversation."
    else:
        return ctx.context.conversation.get_recent_context()

@function_tool
async def set_temporary_note(
    ctx: RunContextWrapper[UserSession],
    key: str,
    note: str
) -> str:
    """Set a temporary note for this session.
    
    Args:
        key: Key to store the note under
        note: The note content
    """
    ctx.context.temporary_data[key] = {
        "content": note,
        "created_at": datetime.now().isoformat()
    }
    ctx.context.update_activity()
    
    return f"📝 I've noted: {note} (under key: {key})"

@function_tool
async def get_session_info(ctx: RunContextWrapper[UserSession]) -> str:
    """Get current session information and preferences."""
    session = ctx.context
    
    info = f"""
🔍 **Session Information**

{session.get_session_summary()}

**User Preferences:**
"""
    
    if session.user_preferences:
        for category, preference in session.user_preferences.items():
            info += f"• {category}: {preference}\n"
    else:
        info += "• No preferences set yet\n"
    
    info += "\n**Temporary Notes:**\n"
    if session.temporary_data:
        for key, data in session.temporary_data.items():
            info += f"• {key}: {data['content']}\n"
    else:
        info += "• No temporary notes\n"
        
    return info

# Dynamic instructions với session context
def create_session_aware_instructions(
    ctx: RunContextWrapper[UserSession], 
    agent: Agent
) -> str:
    """Instructions thay đổi dựa trên session context"""
    session = ctx.context
    
    base_instructions = """
You are a helpful assistant with memory capabilities.

Core abilities:
- Remember user preferences across the conversation
- Recall previous topics we've discussed  
- Take temporary notes during our session
- Provide personalized responses based on learned preferences
"""
    
    # Add personalized context
    if session.user_preferences:
        preferences_context = "\n\nKnown user preferences:\n"
        for category, preference in session.user_preferences.items():
            preferences_context += f"- {category}: {preference}\n"
        base_instructions += preferences_context
    
    # Add conversation context
    if session.conversation.messages:
        conversation_context = f"\n\nConversation context:\n"
        conversation_context += f"- We've exchanged {len(session.conversation.messages)} messages\n"
        conversation_context += f"- Current topics: {', '.join(session.conversation.topics)}\n"
        conversation_context += f"- Conversation tone: {session.conversation.sentiment}\n"
        base_instructions += conversation_context
    
    # Add session info
    base_instructions += f"\n\nSession info:\n- Session ID: {session.session_id}\n- Duration: {datetime.now() - session.created_at}\n"
    
    return base_instructions

# Session-aware agent
memory_agent = Agent[UserSession](
    name="Memory Assistant",
    instructions=create_session_aware_instructions,
    tools=[remember_preference, recall_conversation, set_temporary_note, get_session_info]
)

# Session manager
class SessionManager:
    @staticmethod
    def create_session(user_id: str) -> UserSession:
        session_id = str(uuid.uuid4())
        session = UserSession(
            session_id=session_id,
            user_id=user_id,
            created_at=datetime.now(),
            last_activity=datetime.now()
        )
        sessions[session_id] = session
        return session
    
    @staticmethod
    def get_session(session_id: str) -> Optional[UserSession]:
        session = sessions.get(session_id)
        if session and not session.is_expired():
            session.update_activity()
            return session
        elif session:
            # Clean up expired session
            del sessions[session_id]
        return None
    
    @staticmethod
    def cleanup_expired_sessions():
        expired = [
            sid for sid, session in sessions.items() 
            if session.is_expired()
        ]
        for sid in expired:
            del sessions[sid]
        return len(expired)

# Demo session management
async def demo_session_management():
    from agents import Runner
    
    # Create new session
    session = SessionManager.create_session("user_789")
    print(f"Created session: {session.session_id}\n")
    
    # Simulate conversation with memory
    queries = [
        "I prefer detailed explanations over brief answers",
        "Remember that I'm working on a Python project",
        "Set a note to follow up on the database optimization we discussed",
        "What do you remember about my preferences?",
        "Can you recall what project I'm working on?",
        "Show me my session information"
    ]
    
    for i, query in enumerate(queries, 1):
        print(f"**Turn {i}:** {query}")
        
        # Add user message to conversation history
        session.conversation.add_message("user", query)
        
        result = await Runner.run(memory_agent, query, context=session)
        
        # Add assistant response to conversation history
        session.conversation.add_message("assistant", result.final_output)
        
        print(f"**Response:** {result.final_output}\n")
        print("-" * 60 + "\n")

if __name__ == "__main__":
    asyncio.run(demo_session_management())
```

## LLM Context Management

### 1. Strategic Information Injection

```python
from agents import Agent, function_tool
from datetime import datetime

# Strategy 1: Static Instructions
DOMAIN_EXPERT_INSTRUCTIONS = """
You are a Senior Software Architect with 15+ years of experience.

Domain Expertise:
- Microservices architecture and distributed systems
- Cloud-native technologies (AWS, Kubernetes, Docker)
- Database design and optimization
- Performance tuning and scalability

Current Technology Stack Context:
- Primary languages: Python, Go, TypeScript  
- Cloud: AWS with EKS for container orchestration
- Databases: PostgreSQL, Redis, MongoDB
- Message queues: RabbitMQ, Apache Kafka
- Monitoring: Prometheus, Grafana, ELK stack

Communication Style:
- Provide both high-level strategy and implementation details
- Include code examples and architecture diagrams when relevant
- Consider cost, maintainability, and team expertise in recommendations
- Always mention potential trade-offs and alternatives
"""

# Strategy 2: Dynamic Context Injection
def create_context_aware_instructions(
    project_context: Dict[str, Any],
    team_context: Dict[str, Any],
    current_challenges: List[str]
) -> str:
    """Generate context-rich instructions dynamically"""
    
    instructions = f"""
You are the Technical Lead for the {project_context['name']} project.

# Project Context
- Project: {project_context['name']}
- Stage: {project_context['stage']}  
- Timeline: {project_context['timeline']}
- Budget: {project_context['budget']}

# Team Context
- Team size: {team_context['size']} developers
- Experience level: {team_context['experience_level']}
- Available technologies: {', '.join(team_context['tech_stack'])}

# Current Focus Areas
"""
    
    for challenge in current_challenges:
        instructions += f"- {challenge}\n"
    
    instructions += """
# Decision-Making Framework
Consider these factors in your recommendations:
1. Team capability and learning curve
2. Budget and timeline constraints  
3. Maintenance and operational overhead
4. Scalability and future requirements
5. Integration with existing systems

Tailor all advice to the specific project and team context provided.
"""
    
    return instructions

# Strategy 3: Context Through Tool Results
@function_tool
async def get_system_status() -> str:
    """Get current system status and metrics"""
    # Simulate system monitoring
    return """
🔍 **Current System Status**

**Performance Metrics:**
- API Response Time: 145ms (avg)
- Database Query Time: 23ms (avg)  
- Error Rate: 0.02%
- CPU Utilization: 45%
- Memory Usage: 67%

**Recent Issues:**
- 3 slow queries identified in user_analytics table
- Redis connection pool approaching limits
- Disk space at 78% on primary database server

**Traffic Patterns:**
- Current RPS: 1,247
- Peak today: 2,891 RPS at 14:30
- Geographic distribution: 60% Asia, 25% US, 15% Europe
"""

@function_tool 
async def get_team_velocity() -> str:
    """Get current team performance metrics"""
    return """
📊 **Team Velocity Dashboard**

**Sprint Metrics (Current Sprint 23):**
- Story Points Completed: 34/40
- Tasks Completed: 28/35
- Bugs Fixed: 12
- Code Review Average Time: 4.2 hours

**Quality Metrics:**
- Test Coverage: 87%
- Build Success Rate: 94%
- Deployment Frequency: 2.3 times/week
- Mean Time to Recovery: 1.8 hours

**Team Health:**
- Team Satisfaction: 8.2/10
- Workload Distribution: Balanced
- Knowledge Sharing Score: 7.8/10
"""

# Context-rich agent
tech_lead_agent = Agent(
    name="Technical Lead Assistant", 
    instructions=create_context_aware_instructions(
        project_context={
            "name": "E-commerce Platform Redesign",
            "stage": "Implementation Phase 2",
            "timeline": "Q2 2025 Launch",
            "budget": "$500K"
        },
        team_context={
            "size": 8,
            "experience_level": "Mid to Senior",
            "tech_stack": ["Python", "React", "PostgreSQL", "AWS", "Docker"]
        },
        current_challenges=[
            "Database performance optimization",
            "API rate limiting implementation", 
            "Third-party integration stability",
            "Frontend state management complexity"
        ]
    ),
    tools=[get_system_status, get_team_velocity]
)
```

### 2. Multi-Turn Context Preservation

```python
from agents import Agent, Runner
from typing import List, Dict, Any

class ConversationManager:
    """Manage multi-turn conversations with context preservation"""
    
    def __init__(self, agent: Agent):
        self.agent = agent
        self.conversation_history: List[Dict[str, Any]] = []
        self.context_summary = ""
        self.important_facts: List[str] = []
    
    async def chat_turn(self, user_input: str, context: Any = None) -> str:
        """Execute a single conversation turn with context"""
        
        # Build context-aware input
        if self.conversation_history:
            # Inject conversation context
            contextual_input = self._build_contextual_input(user_input)
        else:
            contextual_input = user_input
            
        # Run the agent
        result = await Runner.run(
            self.agent,
            contextual_input,
            context=context
        )
        
        # Store conversation turn
        self.conversation_history.append({
            "user": user_input,
            "assistant": result.final_output,
            "timestamp": datetime.now().isoformat()
        })
        
        # Update context summary periodically
        if len(self.conversation_history) % 3 == 0:
            await self._update_context_summary()
            
        return result.final_output
    
    def _build_contextual_input(self, current_input: str) -> List[Dict[str, str]]:
        """Build input with conversation context"""
        
        messages = []
        
        # Add context summary if available
        if self.context_summary:
            messages.append({
                "role": "system",
                "content": f"""
Previous conversation context:
{self.context_summary}

Important facts established:
{chr(10).join(['- ' + fact for fact in self.important_facts])}

Continue the conversation considering this context.
"""
            })
        
        # Add recent conversation history (last 3 turns)
        recent_history = self.conversation_history[-3:] if self.conversation_history else []
        
        for turn in recent_history:
            messages.extend([
                {"role": "user", "content": turn["user"]},
                {"role": "assistant", "content": turn["assistant"]}
            ])
        
        # Add current user input
        messages.append({"role": "user", "content": current_input})
        
        return messages
    
    async def _update_context_summary(self):
        """Generate summary of conversation context"""
        if len(self.conversation_history) < 2:
            return
            
        # Create summarization agent
        summarizer = Agent(
            name="Context Summarizer",
            instructions="""
            Summarize the key points and context from this conversation.
            
            Focus on:
            - Main topics discussed
            - Decisions made or preferences expressed
            - Important facts or requirements mentioned
            - Current state of any ongoing tasks or projects
            
            Keep the summary concise but comprehensive.
            """
        )
        
        # Prepare conversation text for summarization
        conversation_text = ""
        for turn in self.conversation_history:
            conversation_text += f"User: {turn['user']}\n"
            conversation_text += f"Assistant: {turn['assistant']}\n\n"
        
        # Generate summary
        summary_result = await Runner.run(
            summarizer,
            f"Summarize this conversation:\n\n{conversation_text}"
        )
        
        self.context_summary = summary_result.final_output
    
    def add_important_fact(self, fact: str):
        """Add an important fact to remember"""
        if fact not in self.important_facts:
            self.important_facts.append(fact)
    
    def get_conversation_stats(self) -> Dict[str, Any]:
        """Get conversation statistics"""
        return {
            "total_turns": len(self.conversation_history),
            "important_facts": len(self.important_facts),
            "has_context_summary": bool(self.context_summary),
            "start_time": self.conversation_history[0]["timestamp"] if self.conversation_history else None,
            "last_activity": self.conversation_history[-1]["timestamp"] if self.conversation_history else None
        }

# Demo multi-turn conversation
async def demo_multi_turn_context():
    # Create agent for project planning
    planning_agent = Agent(
        name="Project Planning Assistant",
        instructions="""
        You are a project planning assistant helping with software development projects.
        
        Your role:
        - Help break down requirements into tasks
        - Estimate effort and timelines
        - Identify risks and dependencies
        - Suggest best practices and methodologies
        
        Remember information from previous conversation turns and build upon it.
        Ask clarifying questions when needed.
        """
    )
    
    # Create conversation manager
    conversation = ConversationManager(planning_agent)
    
    # Simulate multi-turn planning session
    planning_queries = [
        "I need to plan a new mobile app for food delivery. What should we start with?",
        "The app needs to support iOS and Android, with real-time order tracking. What technology stack would you recommend?",
        "Great suggestions! For the backend, we prefer Python. How should we structure the development phases?",
        "We have a team of 4 developers and a 6-month timeline. Is this realistic?",
        "What are the main risks we should plan for in this project?",
        "Can you create a high-level milestone timeline based on everything we've discussed?"
    ]
    
    print("🚀 **Multi-Turn Project Planning Session**\n")
    
    for i, query in enumerate(planning_queries, 1):
        print(f"**Turn {i}:** {query}")
        response = await conversation.chat_turn(query)
        print(f"**Assistant:** {response}\n")
        print("-" * 70 + "\n")
        
        # Add important facts occasionally
        if i == 2:
            conversation.add_important_fact("Technology: iOS/Android mobile app with Python backend")
        elif i == 4:
            conversation.add_important_fact("Team: 4 developers, Timeline: 6 months")
    
    # Show conversation stats
    stats = conversation.get_conversation_stats()
    print("📊 **Conversation Statistics:**")
    for key, value in stats.items():
        print(f"• {key}: {value}")

if __name__ == "__main__":
    asyncio.run(demo_multi_turn_context())
```

## Advanced Context Patterns

### 1. Hierarchical Context Management

```python
from dataclasses import dataclass
from typing import Optional, Dict, Any, List
from enum import Enum

class ContextScope(Enum):
    GLOBAL = "global"      # Across all conversations
    SESSION = "session"    # Within current session  
    THREAD = "thread"      # Within conversation thread
    TURN = "turn"         # Single interaction

@dataclass
class LayeredContext:
    """Hierarchical context with multiple scopes"""
    global_context: Dict[str, Any]    # User profile, permissions
    session_context: Dict[str, Any]   # Session-specific data
    thread_context: Dict[str, Any]    # Conversation thread data
    turn_context: Dict[str, Any]      # Current turn data
    
    def get_context_for_scope(self, scope: ContextScope) -> Dict[str, Any]:
        """Get merged context up to specified scope"""
        merged = {}
        
        if scope in [ContextScope.GLOBAL, ContextScope.SESSION, ContextScope.THREAD, ContextScope.TURN]:
            merged.update(self.global_context)
        
        if scope in [ContextScope.SESSION, ContextScope.THREAD, ContextScope.TURN]:
            merged.update(self.session_context)
            
        if scope in [ContextScope.THREAD, ContextScope.TURN]:
            merged.update(self.thread_context)
            
        if scope == ContextScope.TURN:
            merged.update(self.turn_context)
            
        return merged
    
    def set_context(self, scope: ContextScope, key: str, value: Any):
        """Set context value at specific scope"""
        if scope == ContextScope.GLOBAL:
            self.global_context[key] = value
        elif scope == ContextScope.SESSION:
            self.session_context[key] = value
        elif scope == ContextScope.THREAD:
            self.thread_context[key] = value
        elif scope == ContextScope.TURN:
            self.turn_context[key] = value

@function_tool
async def set_context_value(
    ctx: RunContextWrapper[LayeredContext],
    scope: str,
    key: str, 
    value: str
) -> str:
    """Set a context value at specified scope.
    
    Args:
        scope: Context scope (global, session, thread, turn)
        key: Context key
        value: Context value
    """
    try:
        context_scope = ContextScope(scope.lower())
        ctx.context.set_context(context_scope, key, value)
        
        return f"✅ Set {key} = {value} at {scope} scope"
    except ValueError:
        return f"❌ Invalid scope: {scope}. Use: global, session, thread, or turn"

@function_tool
async def get_context_info(
    ctx: RunContextWrapper[LayeredContext],
    scope: str = "all"
) -> str:
    """Get context information for specified scope."""
    
    if scope == "all":
        info = "🔍 **Complete Context Hierarchy**\n\n"
        
        for scope_enum in ContextScope:
            scope_data = ctx.context.get_context_for_scope(scope_enum)
            info += f"**{scope_enum.value.title()} Context:**\n"
            
            if scope_data:
                for key, value in scope_data.items():
                    info += f"• {key}: {value}\n"
            else:
                info += f"• (empty)\n"
            info += "\n"
                
        return info
    else:
        try:
            context_scope = ContextScope(scope.lower())
            scope_data = ctx.context.get_context_for_scope(context_scope)
            
            info = f"🔍 **{scope.title()} Context**\n\n"
            if scope_data:
                for key, value in scope_data.items():
                    info += f"• {key}: {value}\n"
            else:
                info += "• (empty)\n"
                
            return info
        except ValueError:
            return f"❌ Invalid scope: {scope}"

# Hierarchical context agent
layered_agent = Agent[LayeredContext](
    name="Context-Aware Assistant",
    instructions="""
    You are an assistant with hierarchical context awareness.
    
    Context Scopes (from broad to specific):
    - Global: User profile, permanent preferences  
    - Session: Current session data, temporary preferences
    - Thread: Conversation-specific information
    - Turn: Current interaction data
    
    Use context appropriately:
    - Remember user preferences across conversations (Global)
    - Track session-specific settings (Session)  
    - Maintain conversation flow (Thread)
    - Handle current request details (Turn)
    
    Always consider the most specific relevant context for responses.
    """,
    tools=[set_context_value, get_context_info]
)

# Demo hierarchical context
async def demo_hierarchical_context():
    from agents import Runner
    
    # Initialize layered context
    context = LayeredContext(
        global_context={"user_name": "Alice", "role": "developer"},
        session_context={"session_start": datetime.now().isoformat()},
        thread_context={},
        turn_context={}
    )
    
    queries = [
        "Set my preferred programming language to Python at global scope",
        "Set current project to 'Mobile App' at thread scope", 
        "Set today's focus to 'API development' at session scope",
        "Show me all my context information",
        "What do you know about my preferences and current work?"
    ]
    
    for query in queries:
        print(f"**Query:** {query}")
        
        # Clear turn context for each new turn
        context.turn_context = {"current_query": query}
        
        result = await Runner.run(layered_agent, query, context=context)
        print(f"**Response:** {result.final_output}\n")
        print("-" * 50 + "\n")

if __name__ == "__main__":
    asyncio.run(demo_hierarchical_context())
```

**## Tổng Kết và Hướng Dẫn Triển Khai**

### Những điểm cần ghi nhớ

* **Phân biệt Local vs. LLM Context**: Hiểu rõ dữ liệu nào agent “nhìn thấy” và dữ liệu nào chỉ dùng nội bộ.
* **Dependency Injection**: Quản lý các service, kết nối và tài nguyên một cách có tổ chức.
* **Session Management**: Giữ trạng thái xuyên suốt cuộc trò chuyện, đảm bảo agent nhớ được lịch sử.
* **Strategic Context Injection**: Chỉ bơm thông tin phù hợp vào LLM khi thật sự cần.
* **Hierarchical Context**: Xây dựng ngữ cảnh theo nhiều tầng (global, session, thread, turn) để quản lý linh hoạt.

---

### Kinh nghiệm thực tiễn

1. **Bảo mật & Quyền riêng tư**

   * Không bao giờ đưa API key, mật khẩu vào prompt gửi đến LLM.
   * Luôn sanitize đầu vào trước khi lưu trữ.
   * Áp dụng kiểm soát truy cập và ghi lại log cho mọi thao tác với context.

2. **Hiệu năng**

   * Cache dữ liệu context được truy vấn thường xuyên.
   * Thiết lập timeout/expiration và cơ chế dọn dẹp context.
   * Gom nhóm cập nhật để giảm số lần gọi database.
   * Giám sát độ lớn của context để tránh chạm giới hạn token.

3. **Độ tin cậy**

   * Bắt lỗi và fallback khi context bị hỏng hoặc thiếu.
   * Luôn kiểm tra tính toàn vẹn của dữ liệu context.
   * Cung cấp giá trị mặc định cho các thông tin chưa có.

---

### Danh sách kiểm tra trước khi triển khai

* [ ] Xác định rõ các scope context cần dùng (global, session, thread, turn)
* [ ] Thiết kế mô hình dữ liệu context phù hợp
* [ ] Quy định lifecycle và cơ chế cleanup cho mỗi scope
* [ ] Định nghĩa cách truy cập, cập nhật và inject context vào LLM
* [ ] Viết tests cho isolation, persistence, expiration và hiệu suất của context

---

### Bước tiếp theo

Trong bài kế tiếp, chúng ta sẽ cùng khám phá:

* **Handoffs** – Cách điều phối nhiều agent chuyên biệt phối hợp hiệu quả
* **Multi-Agent Workflows** – Các pattern phối hợp và best practices
* **Giao tiếp giữa các agent** – Xây dựng luồng dữ liệu và thông điệp rõ ràng

---

### Thử thách cho bạn

1. Triển khai cơ chế **ghi nhớ cuộc trò chuyện** cho agent hiện có.
2. Xây dựng hệ thống **layered context** cho một trường hợp cụ thể.
3. Tạo **session manager** với tính năng lưu trữ và tự động hết hạn.
4. Thử nghiệm nhiều chiến lược **inject context** và so sánh hiệu quả.

---

*Bài tiếp theo: [**"Handoffs: Orchestrating Multiple Agents như một Dàn Nhạc"**](../openai-agents-sdk-part-04-handoffs-multi-agent-orchestration/) - Chúng ta sẽ học cách các agents phối hợp và ủy quyền cho nhau để giải quyết những tác vụ phức tạp.*