---
title: "Handoffs: Orchestrating Multiple Agents như một Dàn Nhạc"
date: 2025-05-23 09:00:00 +0700
categories: [AI, OpenAI]
tags: [openai, agents, handoffs, multi-agent, orchestration]
author: toantranct
description: Hướng dẫn chi tiết về Handoffs trong OpenAI Agents SDK. Từ basic delegation đến advanced multi-agent orchestration patterns với real-world examples.
---

Trong những bài trước, chúng ta đã học cách tạo agents với tools và quản lý context. Nhưng trong thế giới thực, một agent duy nhất - dù có mạnh mẽ đến đâu - cũng không thể giải quyết mọi vấn đề phức tạp. Đó là lúc chúng ta cần **Handoffs** - khả năng cho phép các agents phối hợp và ủy quyền cho nhau như một dàn nhạc hoàn hảo.

Handoffs không chỉ là việc "chuyển giao công việc", mà là nghệ thuật **orchestration** - biết khi nào cần chuyên gia, làm thế nào để truyền đạt context, và đảm bảo workflow mượt mà. Hôm nay, chúng ta sẽ khám phá cách xây dựng những hệ thống multi-agent thông minh và hiệu quả.

## Tại Sao Cần Handoffs?

### Thế Giới Thực Hoạt Động Như Thế Nào

Hãy tưởng tượng bạn gọi vào một công ty lớn:

1. **Tổng đài** tiếp nhận và phân loại yêu cầu
2. **Chuyên viên kỹ thuật** xử lý vấn đề technical
3. **Quản lý** xử lý các vấn đề phức tạp
4. **Chuyên viên thanh toán** xử lý billing issues

Mỗi người có expertise riêng, nhưng họ phối hợp để giải quyết vấn đề của bạn một cách hiệu quả nhất.

### Single Agent vs Multi-Agent Architecture

**🤖 Single Agent Approach:**
```
User Request → Generalist Agent → Response
```

**Vấn đề:**
- Agent phải "biết tất cả" → performance kém
- Context switching giữa các domains khác nhau
- Khó maintain và improve từng area cụ thể
- Không tận dụng được specialized knowledge

**🎭 Multi-Agent với Handoffs:**
```
User Request → Triage Agent → Specialist Agent A → Response
                           → Specialist Agent B → Response
                           → Escalation Agent → Response
```

**Lợi ích:**
- Mỗi agent focus vào một domain cụ thể
- Better performance và accuracy
- Easier maintenance và scalability
- Natural division of concerns

### Use Cases Phổ Biến

🏢 **Customer Service System:**
- Triage Agent → Booking Agent / Refund Agent / FAQ Agent

🔍 **Research Platform:**
- Research Coordinator → Data Collector / Analyst / Report Writer

⚕️ **Healthcare Assistant:**
- Intake Agent → Symptom Checker / Appointment Scheduler / Emergency Handler

💼 **Business Process Automation:**
- Process Manager → Document Processor / Email Handler / Database Manager

## Handoffs Cơ Bản

### Cách Hoạt Động

Handoffs trong OpenAI Agents SDK hoạt động bằng cách biến chúng thành **tools** mà LLM có thể gọi. Nếu có handoff đến agent tên "Refund Agent", tool sẽ có tên `transfer_to_refund_agent`.

### Tạo Handoff Đơn Giản

```python
from agents import Agent, Runner

# Tạo specialized agents
billing_agent = Agent(
    name="Billing Agent",
    instructions="""
    You are a billing specialist for TechCorp.
    
    Expertise:
    - Invoice questions and disputes
    - Payment processing issues
    - Subscription management
    - Billing history inquiries
    
    Always:
    - Ask for account number or email for verification
    - Provide clear explanations of charges
    - Offer solutions, not just explanations
    - Be empathetic with billing concerns
    """
)

refund_agent = Agent(
    name="Refund Agent", 
    instructions="""
    You are a refund specialist for TechCorp.
    
    Expertise:
    - Processing refund requests
    - Determining refund eligibility
    - Explaining refund policies
    - Handling special cases and escalations
    
    Refund Policy Guidelines:
    - Full refund within 30 days for unused services
    - Partial refund for partially used services
    - No refund for services used > 30 days (exceptions apply)
    - Always explain the reasoning behind decisions
    """
)

# Triage agent với handoffs
triage_agent = Agent(
    name="Customer Service Triage",
    instructions="""
    You are the first point of contact for customer service.
    
    Your role:
    - Understand customer inquiries
    - Route to appropriate specialists
    - Handle simple questions yourself when possible
    
    Routing Guidelines:
    - Billing questions, payment issues → Billing Agent
    - Refund requests, return policies → Refund Agent
    - General questions → Handle yourself
    
    Always be friendly and explain why you're transferring them.
    """,
    handoffs=[billing_agent, refund_agent]  # Basic handoff setup
)

# Test basic handoffs
async def test_basic_handoffs():
    queries = [
        "I have a question about my last invoice",
        "I want to request a refund for my subscription", 
        "What are your business hours?"
    ]
    
    for query in queries:
        print(f"**Customer:** {query}")
        result = await Runner.run(triage_agent, query)
        print(f"**Response:** {result.final_output}")
        print(f"**Last Agent:** {result.last_agent.name}\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(test_basic_handoffs())
```

## Advanced Handoffs với Customization

### Handoff với Input Data

Thường thì khi handoff, chúng ta muốn truyền thêm thông tin cụ thể:

```python
from pydantic import BaseModel
from agents import Agent, handoff, RunContextWrapper

class EscalationData(BaseModel):
    reason: str
    customer_priority: str
    issue_summary: str
    attempted_solutions: list[str]

class TicketData(BaseModel):
    ticket_id: str
    category: str
    urgency: str
    customer_info: str

async def on_escalation_handoff(ctx: RunContextWrapper[None], input_data: EscalationData):
    """Callback khi escalation xảy ra"""
    print(f"🚨 ESCALATION ALERT:")
    print(f"   Reason: {input_data.reason}")
    print(f"   Priority: {input_data.customer_priority}")
    print(f"   Summary: {input_data.issue_summary}")
    
    # Có thể gửi notification, log, etc.
    # await send_slack_notification(input_data)
    # await create_high_priority_ticket(input_data)

async def on_technical_handoff(ctx: RunContextWrapper[None], input_data: TicketData):
    """Callback cho technical handoffs"""
    print(f"🔧 Technical Support Handoff:")
    print(f"   Ticket: {input_data.ticket_id}")
    print(f"   Category: {input_data.category}")
    print(f"   Urgency: {input_data.urgency}")

# Advanced agents với custom handoffs
escalation_agent = Agent(
    name="Escalation Manager",
    instructions="""
    You are a senior customer service manager handling escalated issues.
    
    Expertise:
    - Complex problem resolution
    - Policy exceptions and special cases
    - Customer retention strategies
    - Interdepartmental coordination
    
    When you receive an escalation:
    1. Acknowledge the customer's frustration
    2. Review all attempted solutions
    3. Take ownership of the resolution
    4. Provide timeline for resolution
    5. Follow up personally
    """
)

technical_agent = Agent(
    name="Technical Support Specialist", 
    instructions="""
    You are a technical support specialist.
    
    Expertise:
    - Software troubleshooting
    - API integration issues
    - Performance optimization
    - System configuration
    
    Approach:
    1. Gather detailed technical information
    2. Reproduce the issue when possible
    3. Provide step-by-step solutions
    4. Verify resolution with customer
    5. Document solution for future reference
    """
)

# Advanced triage với custom handoffs
advanced_triage_agent = Agent(
    name="Advanced Customer Service",
    instructions="""
    You are an intelligent customer service coordinator.
    
    Capabilities:
    - Assess issue complexity and urgency
    - Route to most appropriate specialist
    - Gather necessary information before handoff
    - Handle simple issues independently
    
    Handoff Decision Matrix:
    - Technical issues (API, integration, bugs) → Technical Support
    - Billing disputes, complex refunds → Escalation Manager
    - Standard questions → Handle yourself
    
    Always gather relevant details before handoff to make the specialist's job easier.
    """,
    handoffs=[
        handoff(
            agent=escalation_agent,
            on_handoff=on_escalation_handoff,
            input_type=EscalationData,
            tool_name_override="escalate_to_manager",
            tool_description_override="Escalate complex issues to senior management"
        ),
        handoff(
            agent=technical_agent,
            on_handoff=on_technical_handoff,
            input_type=TicketData,
            tool_name_override="transfer_to_technical_support",
            tool_description_override="Transfer technical issues to specialized support"
        )
    ]
)

# Demo advanced handoffs
async def demo_advanced_handoffs():
    scenarios = [
        "Our API integration has been failing for 3 days and it's affecting our production system. We've tried restarting services but nothing works.",
        "I've been trying to get a refund for 2 weeks and nobody has helped me. I want to speak to a manager immediately.",
        "What's the difference between your Pro and Enterprise plans?"
    ]
    
    for scenario in scenarios:
        print(f"**Customer Issue:** {scenario}")
        result = await Runner.run(advanced_triage_agent, scenario)
        print(f"**Agent Response:** {result.final_output}")
        print(f"**Final Handler:** {result.last_agent.name}")
        print("-" * 80 + "\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_advanced_handoffs())
```

### Input Filters - Kiểm Soát Context Flow

Input filters cho phép bạn kiểm soát thông tin nào được truyền đến agent tiếp theo:

```python
from agents import Agent, handoff
from agents.extensions import handoff_filters
from agents.handoffs import HandoffInputData

def custom_security_filter(input_data: HandoffInputData) -> HandoffInputData:
    """Remove sensitive information before handoff"""
    filtered_input = input_data.copy()
    
    # Remove any messages containing sensitive keywords
    sensitive_keywords = ['password', 'ssn', 'credit card', 'api key']
    
    filtered_messages = []
    for item in filtered_input.input:
        if isinstance(item, dict) and 'content' in item:
            content = item['content'].lower()
            if not any(keyword in content for keyword in sensitive_keywords):
                filtered_messages.append(item)
            else:
                # Replace with sanitized message
                sanitized_item = item.copy()
                sanitized_item['content'] = "[SENSITIVE INFORMATION REMOVED FOR SECURITY]"
                filtered_messages.append(sanitized_item)
        else:
            filtered_messages.append(item)
    
    filtered_input.input = filtered_messages
    return filtered_input

def context_summarizer_filter(input_data: HandoffInputData) -> HandoffInputData:
    """Summarize long conversations before handoff"""
    if len(input_data.input) > 10:  # If conversation is too long
        # Keep first message (context) and last 3 messages (recent context)
        filtered_input = input_data.copy()
        filtered_input.input = (
            input_data.input[:1] +  # Original context
            [{"role": "system", "content": "[CONVERSATION SUMMARY: Customer has been discussing billing issues for 15+ messages]"}] +
            input_data.input[-3:]   # Recent messages
        )
        return filtered_input
    return input_data

# Agents với input filters
secure_billing_agent = Agent(
    name="Secure Billing Specialist",
    instructions="""
    You are a billing specialist with strict security protocols.
    
    Security Guidelines:
    - Never ask for sensitive information like SSN or full credit card numbers
    - Use last 4 digits for verification only
    - Redirect sensitive requests to secure channels
    - Log all security-related interactions
    """
)

# Handoff với security filter
secure_triage_agent = Agent(
    name="Security-Conscious Triage",
    instructions="""
    You are a security-conscious customer service agent.
    Route billing issues to billing specialists with appropriate security measures.
    """,
    handoffs=[
        handoff(
            agent=secure_billing_agent,
            input_filter=custom_security_filter,
            tool_description_override="Transfer to secure billing specialist"
        )
    ]
)
```

## Real-World Example: Smart E-commerce Support

Hãy xây dựng một hệ thống customer support hoàn chỉnh cho e-commerce:

```python
import json
from datetime import datetime, timedelta
from typing import Dict, Any, List, Optional
from pydantic import BaseModel
from agents import Agent, handoff, function_tool, RunContextWrapper

# Data models
class OrderInfo(BaseModel):
    order_id: str
    status: str
    items: List[Dict[str, Any]]
    total: float
    shipping_address: str

class CustomerProfile(BaseModel):
    customer_id: str
    name: str
    email: str
    tier: str  # bronze, silver, gold, platinum
    order_history: List[str]

# Mock data storage
ORDERS_DB = {
    "ORD-12345": {
        "order_id": "ORD-12345",
        "status": "shipped",
        "items": [{"name": "Wireless Headphones", "price": 99.99, "qty": 1}],
        "total": 99.99,
        "shipping_address": "123 Main St, City, State",
        "tracking_number": "TRK-789456"
    }
}

CUSTOMERS_DB = {
    "CUST-789": {
        "customer_id": "CUST-789",
        "name": "Alice Johnson",
        "email": "alice@example.com", 
        "tier": "gold",
        "order_history": ["ORD-12345", "ORD-11111"]
    }
}

# Shared context for all agents
class EcommerceContext:
    def __init__(self):
        self.current_customer: Optional[CustomerProfile] = None
        self.current_order: Optional[OrderInfo] = None
        self.session_notes: List[str] = []
    
    def add_note(self, note: str):
        self.session_notes.append(f"[{datetime.now().strftime('%H:%M')}] {note}")

# Tools for order management
@function_tool
async def lookup_order(
    ctx: RunContextWrapper[EcommerceContext], 
    order_id: str
) -> str:
    """Look up order information by order ID."""
    order_data = ORDERS_DB.get(order_id)
    
    if not order_data:
        return f"❌ Order {order_id} not found. Please check the order ID and try again."
    
    ctx.context.current_order = OrderInfo(**order_data)
    ctx.context.add_note(f"Retrieved order {order_id}")
    
    return f"""
📦 **Order Information: {order_id}**

• **Status:** {order_data['status'].title()}
• **Items:** {', '.join([f"{item['name']} (${item['price']})" for item in order_data['items']])}
• **Total:** ${order_data['total']}
• **Shipping Address:** {order_data['shipping_address']}
• **Tracking:** {order_data.get('tracking_number', 'Not available')}
"""

@function_tool
async def lookup_customer(
    ctx: RunContextWrapper[EcommerceContext],
    customer_email: str
) -> str:
    """Look up customer profile by email."""
    # Simple lookup by email
    customer_data = None
    for customer in CUSTOMERS_DB.values():
        if customer['email'].lower() == customer_email.lower():
            customer_data = customer
            break
    
    if not customer_data:
        return f"❌ Customer with email {customer_email} not found."
    
    ctx.context.current_customer = CustomerProfile(**customer_data)
    ctx.context.add_note(f"Retrieved customer profile for {customer_data['name']}")
    
    return f"""
👤 **Customer Profile**

• **Name:** {customer_data['name']}
• **Email:** {customer_data['email']}
• **Tier:** {customer_data['tier'].title()}
• **Order History:** {len(customer_data['order_history'])} orders
"""

# Specialized agents
order_tracking_agent = Agent[EcommerceContext](
    name="Order Tracking Specialist",
    instructions="""
    You are an order tracking specialist for ShopSmart e-commerce.
    
    Expertise:
    - Order status inquiries
    - Shipping and delivery updates
    - Tracking number assistance
    - Delivery issue resolution
    
    Process:
    1. Get order ID from customer
    2. Look up order information
    3. Provide detailed status update
    4. Offer proactive solutions for delays
    5. Set expectations for delivery
    
    Be proactive about potential issues and always provide tracking numbers when available.
    """,
    tools=[lookup_order, lookup_customer]
)

returns_agent = Agent[EcommerceContext](
    name="Returns & Exchanges Specialist",
    instructions="""
    You are a returns and exchanges specialist for ShopSmart.
    
    Return Policy:
    - 30-day return window for most items
    - Items must be unused and in original packaging
    - Free returns for defective items
    - Customer pays return shipping for non-defective items
    
    Process:
    1. Verify order and customer information
    2. Check return eligibility
    3. Explain return process clearly
    4. Generate return authorization if approved
    5. Provide return shipping instructions
    
    Always be understanding about customer concerns and look for win-win solutions.
    """,
    tools=[lookup_order, lookup_customer]
)

account_agent = Agent[EcommerceContext](
    name="Account Management Specialist", 
    instructions="""
    You are an account management specialist for ShopSmart.
    
    Responsibilities:
    - Account settings and preferences
    - Subscription management
    - Loyalty program questions
    - Account security issues
    - Profile updates
    
    Customer Tiers & Benefits:
    - Bronze: Standard support, regular shipping
    - Silver: Priority support, free shipping over $50
    - Gold: Priority support, free shipping, 5% discount
    - Platinum: VIP support, free express shipping, 10% discount
    
    Always highlight tier benefits and upgrade opportunities when appropriate.
    """,
    tools=[lookup_customer]
)

# Handoff data models
class OrderHandoffData(BaseModel):
    order_id: str
    issue_type: str
    customer_tier: str

class ReturnHandoffData(BaseModel):
    order_id: str
    return_reason: str
    customer_tier: str

class AccountHandoffData(BaseModel):
    customer_email: str
    issue_category: str
    customer_tier: str

# Handoff callbacks
async def on_order_handoff(ctx: RunContextWrapper[EcommerceContext], data: OrderHandoffData):
    ctx.context.add_note(f"Handoff to Order Tracking: {data.issue_type} for order {data.order_id}")
    print(f"📊 Order tracking handoff: {data.issue_type} ({data.customer_tier} customer)")

async def on_return_handoff(ctx: RunContextWrapper[EcommerceContext], data: ReturnHandoffData):
    ctx.context.add_note(f"Handoff to Returns: {data.return_reason} for order {data.order_id}")
    print(f"📊 Returns handoff: {data.return_reason} ({data.customer_tier} customer)")

async def on_account_handoff(ctx: RunContextWrapper[EcommerceContext], data: AccountHandoffData):
    ctx.context.add_note(f"Handoff to Account Management: {data.issue_category}")
    print(f"📊 Account management handoff: {data.issue_category} ({data.customer_tier} customer)")

# Main customer service agent
customer_service_agent = Agent[EcommerceContext](
    name="ShopSmart Customer Service",
    instructions="""
    You are the main customer service agent for ShopSmart e-commerce.
    
    Your role:
    - First point of contact for all customer inquiries
    - Intelligent routing to appropriate specialists
    - Handle simple questions independently
    - Gather necessary information before handoffs
    
    Routing Logic:
    - Order status, shipping, tracking → Order Tracking Specialist
    - Returns, exchanges, refunds → Returns & Exchanges Specialist  
    - Account issues, subscriptions, profile → Account Management Specialist
    
    Before handoffs:
    - Gather customer email and/or order ID
    - Understand the specific issue
    - Set customer expectations about the handoff
    
    Customer Tier Priority:
    - Platinum/Gold: Immediate attention, proactive solutions
    - Silver: Standard priority, helpful service
    - Bronze: Standard service, suggest upgrades when appropriate
    """,
    handoffs=[
        handoff(
            agent=order_tracking_agent,
            on_handoff=on_order_handoff,
            input_type=OrderHandoffData,
            tool_name_override="transfer_to_order_tracking"
        ),
        handoff(
            agent=returns_agent,
            on_handoff=on_return_handoff, 
            input_type=ReturnHandoffData,
            tool_name_override="transfer_to_returns"
        ),
        handoff(
            agent=account_agent,
            on_handoff=on_account_handoff,
            input_type=AccountHandoffData,
            tool_name_override="transfer_to_account_management"
        )
    ],
    tools=[lookup_order, lookup_customer]
)

# Demo the complete e-commerce support system
async def demo_ecommerce_support():
    from agents import Runner
    
    print("🛍️ **ShopSmart Customer Service Simulation**\n")
    
    context = EcommerceContext()
    
    scenarios = [
        {
            "query": "Hi, I want to check the status of my order ORD-12345",
            "description": "Order tracking inquiry"
        },
        {
            "query": "I received my wireless headphones but they're defective. I want to return them. My email is alice@example.com",
            "description": "Return request"
        },
        {
            "query": "I can't log into my account and I want to update my shipping address",
            "description": "Account management issue"
        },
        {
            "query": "What are the benefits of upgrading to Gold tier?",
            "description": "General inquiry (no handoff needed)"
        }
    ]
    
    for i, scenario in enumerate(scenarios, 1):
        print(f"**Scenario {i}: {scenario['description']}**")
        print(f"Customer: {scenario['query']}")
        
        result = await Runner.run(customer_service_agent, scenario['query'], context=context)
        
        print(f"Final Handler: {result.last_agent.name}")
        print(f"Response: {result.final_output}")
        
        # Show session notes
        if context.session_notes:
            print(f"Session Notes: {'; '.join(context.session_notes[-2:])}")
        
        print("-" * 80 + "\n")
        
        # Reset context for next scenario
        context = EcommerceContext()

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_ecommerce_support())
```

## Best Practices và Patterns

### 1. Handoff Decision Matrix

```python
# Pattern: Decision matrix trong instructions
TRIAGE_DECISION_MATRIX = """
# Handoff Decision Matrix

## Order-Related Issues
- Order status, tracking → Order Tracking Agent
- Shipping delays, delivery issues → Order Tracking Agent
- Order modifications, cancellations → Order Management Agent

## Product & Returns
- Product defects, quality issues → Returns Agent
- Return requests, exchanges → Returns Agent
- Warranty claims → Returns Agent

## Billing & Payments
- Payment failures, billing disputes → Billing Agent
- Refund requests → Billing Agent (if simple) or Returns Agent (if product-related)
- Subscription management → Account Agent

## Technical Issues
- Website bugs, app crashes → Technical Support Agent
- API integration problems → Technical Support Agent
- Performance issues → Technical Support Agent

## Escalation Triggers
- Customer mentions "manager", "supervisor" → Escalation Agent
- Unresolved after 3+ interactions → Escalation Agent
- Threats, legal language → Escalation Agent
"""

specialized_triage_agent = Agent(
    name="Expert Triage Agent",
    instructions=f"""
    You are an expert customer service triage agent.
    
    {TRIAGE_DECISION_MATRIX}
    
    Your process:
    1. Listen carefully to understand the core issue
    2. Ask clarifying questions if needed
    3. Consult the decision matrix above
    4. Route to the most appropriate specialist
    5. Provide context to the specialist
    """,
    handoffs=[...] # All specialized agents
)
```

### 2. Recommended Prompts Pattern

```python
from agents.extensions.handoff_prompt import prompt_with_handoff_instructions

# Sử dụng recommended prompts cho better handoff understanding
enhanced_agent = Agent(
    name="Enhanced Agent",
    instructions=prompt_with_handoff_instructions("""
    Your core instructions here...
    """),
    handoffs=[other_agent]
)

# Hoặc manual inclusion
HANDOFF_INSTRUCTIONS = """
# Handoff Guidelines

When you need to transfer a conversation:
1. Explain to the user why you're transferring them
2. Summarize what you've learned so far
3. Set expectations about what the specialist will help with
4. Use the appropriate transfer tool

Available transfers:
- transfer_to_billing_agent: For billing, payments, and financial issues
- transfer_to_technical_agent: For technical problems and integrations
- escalate_to_manager: For complex issues requiring senior attention

Example: "I can see you're having a billing issue. Let me transfer you to our billing specialist who can access your account details and resolve this quickly."
"""
```

### 3. Context Preservation Pattern

```python
def create_context_preserving_filter(important_keys: List[str]):
    """Create filter that preserves important context"""
    def context_filter(input_data: HandoffInputData) -> HandoffInputData:
        # Preserve conversation context
        filtered_input = input_data.copy()
        
        # Add context summary
        context_summary = []
        for key in important_keys:
            if hasattr(input_data, key):
                context_summary.append(f"{key}: {getattr(input_data, key)}")
        
        if context_summary:
            context_message = {
                "role": "system",
                "content": f"Context from previous conversation: {'; '.join(context_summary)}"
            }
            filtered_input.input.insert(0, context_message)
        
        return filtered_input
    
    return context_filter

# Usage
context_preserving_handoff = handoff(
    agent=specialist_agent,
    input_filter=create_context_preserving_filter(['customer_tier', 'order_id', 'issue_type'])
)
```

### 4. Monitoring và Analytics

```python
import time
from typing import Dict
from collections import defaultdict

class HandoffAnalytics:
    def __init__(self):
        self.handoff_counts: Dict[str, int] = defaultdict(int)
        self.handoff_times: Dict[str, List[float]] = defaultdict(list)
        self.success_rates: Dict[str, List[bool]] = defaultdict(list)
    
    async def track_handoff(self, from_agent: str, to_agent: str, success: bool, duration: float):
        handoff_key = f"{from_agent} → {to_agent}"
        self.handoff_counts[handoff_key] += 1
        self.handoff_times[handoff_key].append(duration)
        self.success_rates[handoff_key].append(success)
    
    def get_analytics_report(self) -> str:
        report = "📊 **Handoff Analytics Report**\n\n"
        
        for handoff_key, count in self.handoff_counts.items():
            avg_time = sum(self.handoff_times[handoff_key]) / len(self.handoff_times[handoff_key])
            success_rate = sum(self.success_rates[handoff_key]) / len(self.success_rates[handoff_key]) * 100
            
            report += f"**{handoff_key}:**\n"
            report += f"• Count: {count}\n"
            report += f"• Avg Duration: {avg_time:.2f}s\n" 
            report += f"• Success Rate: {success_rate:.1f}%\n\n"
        
        return report

# Global analytics tracker
analytics = HandoffAnalytics()

async def tracked_handoff_callback(ctx: RunContextWrapper, data: BaseModel):
    start_time = time.time()
    
    # Your handoff logic here
    
    duration = time.time() - start_time
    await analytics.track_handoff(
        from_agent="Triage Agent",
        to_agent="Specialist Agent", 
        success=True,  # Determine based on your criteria
        duration=duration
    )
```

## Production Considerations

### 1. Error Handling và Fallbacks

```python
from agents.exceptions import AgentsException

class RobustAgent(Agent):
    def __init__(self, fallback_agent: Agent, **kwargs):
        super().__init__(**kwargs)
        self.fallback_agent = fallback_agent
    
    async def run_with_fallback(self, input_data, context=None):
        try:
            result = await Runner.run(self, input_data, context=context)
            return result
        except AgentsException as e:
            print(f"Primary agent failed: {e}. Falling back...")
            return await Runner.run(self.fallback_agent, input_data, context=context)

# Usage
primary_agent = Agent(name="Primary", instructions="...")
fallback_agent = Agent(name="Fallback", instructions="Handle any general inquiries...")

robust_agent = RobustAgent(
    fallback_agent=fallback_agent,
    name="Robust Primary Agent",
    instructions="...",
    handoffs=[...]
)
```

### 2. Rate Limiting và Resource Management

```python
import asyncio
from typing import Dict
from datetime import datetime, timedelta

class HandoffRateLimiter:
    def __init__(self, max_handoffs_per_minute: int = 10):
        self.max_handoffs = max_handoffs_per_minute
        self.handoff_timestamps: Dict[str, List[datetime]] = defaultdict(list)
    
    async def check_rate_limit(self, agent_name: str) -> bool:
        now = datetime.now()
        recent_handoffs = [
            ts for ts in self.handoff_timestamps[agent_name]
            if now - ts < timedelta(minutes=1)
        ]
        
        self.handoff_timestamps[agent_name] = recent_handoffs
        
        if len(recent_handoffs) >= self.max_handoffs:
            return False
        
        self.handoff_timestamps[agent_name].append(now)
        return True

# Global rate limiter
rate_limiter = HandoffRateLimiter(max_handoffs_per_minute=5)

async def rate_limited_handoff_callback(ctx: RunContextWrapper, data: BaseModel):
    if not await rate_limiter.check_rate_limit("specialist_agent"):
        raise Exception("Rate limit exceeded. Please try again later.")
    
    # Continue with handoff
    print(f"Handoff approved: {data}")
```

## Tổng Kết và Những Điều Cần Nhớ

### Những Gì Chúng Ta Đã Học

✅ **Handoffs Fundamentals** - Cách agents ủy quyền cho nhau hiệu quả  
✅ **Customization Patterns** - Input data, callbacks, và filters  
✅ **Real-world Architecture** - Multi-agent e-commerce support system  
✅ **Best Practices** - Decision matrices, context preservation, monitoring  
✅ **Production Patterns** - Error handling, rate limiting, analytics  

### Core Principles cho Multi-Agent Systems

🎯 **Single Responsibility** - Mỗi agent đảm nhận đúng một lĩnh vực chuyên môn.  
🔄 **Seamless Handoffs** - Context được truyền tiếp đầy đủ, không để lộ hay mất mát thông tin.  
📊 **Intelligent Routing** - Quyết định chuyển giao dựa trên ma trận nghiệp vụ và dữ liệu khách hàng.  
🛡️ **Robust Error Handling** - Graceful degradation và fallback mechanisms  
📈 **Observability** - Theo dõi số lượng, thời gian và tỉ lệ thành công của mỗi handoff.

### Hướng dẫn triển khai

1. **Giai đoạn thiết kế**

   * Xác định rõ trách nhiệm của từng agent.
   * Lập ma trận quyết định (decision matrix) cho các loại yêu cầu.
   * Định nghĩa cách bảo vệ và lọc context (input filters, security).
   * Vạch sẵn luồng fallback khi handoff không thành công.

2. **Giai đoạn phát triển**

   * Tạo các agent chuyên biệt với instructions rõ ràng.
   * Xây dựng các tool handoff với callback và input\_type tuỳ chỉnh.
   * Áp dụng filter để loại bỏ thông tin nhạy cảm và tóm tắt context dài.
   * Tích hợp monitoring/analytics để thu thập dữ liệu handoff.
   * Thêm cơ chế rate limiting, error handling và fallback agents.

3. **Giai đoạn kiểm thử**

   * Test mọi ngả handoff (đường đi chính, các trường hợp edge-case).
   * Kiểm tra tính toàn vẹn của context sau mỗi handoff.
   * Đánh giá hiệu năng và tần suất handoff; đảm bảo không quá tải.
   * Xác thực cơ chế fallback và xử lý lỗi.

4. **Giai đoạn vận hành**

   * Thiết lập dashboard giám sát handoff counts, avg time, success rate.
   * Cấu hình rate limiting phù hợp với lưu lượng thực tế.
   * Theo dõi logs để phát hiện circular handoffs hoặc context loss.
   * Cập nhật ma trận routing và nâng cấp agents theo nhu cầu.

---

### Những anti-pattern cần tránh

* **Chuyển giao quá nhiều lần** khiến khách hàng mất kiên nhẫn.
* **Mất context** làm specialist không biết chuyện gì đã xảy ra.
* **Circular handoffs** (agent này gửi lại agent kia).
* **Agents quá generic** thiếu chuyên môn sâu.
* **Không có fallback** khi handoff thất bại.

---

### Bước Tiếp Theo

Trong **bài tiếp theo**, chúng ta sẽ khám phá:

🛡️ **Guardrails** - Bảo vệ và kiểm soát agent behavior  
🔒 **Security Patterns** - Input validation, content filtering  
⚡ **Performance Optimization** - Caching, rate limiting, monitoring  

### Thử Thách Cho Bạn

1. Thiết kế hệ thống multi-agent cho một domain bạn đang quan tâm.
2. Viết filter tùy chỉnh để bảo vệ privacy/security khi handoff.
3. Xây dựng dashboard analytics cho handoff — số lượng, thời gian, tỉ lệ thành công.
4. Thử nghiệm các chiến lược phân chia responsibilities khác nhau và so sánh kết quả.

---

*Bài tiếp theo: [**"Guardrails: Bảo Vệ và Kiểm Soát Agent Behavior Hiệu Quả"**](../openai-agents-sdk-part-05-guardrails-security-validation/) - Chúng ta sẽ học cách xây dựng các lớp bảo vệ để đảm bảo agents hoạt động an toàn và đúng mục đích.*