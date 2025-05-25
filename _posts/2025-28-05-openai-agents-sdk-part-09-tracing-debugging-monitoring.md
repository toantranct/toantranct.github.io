---
title: "Tracing và Debugging: Monitoring Agent Performance"
date: 2025-05-24 09:00:00 +0700
categories: [AI, OpenAI]
tags: [openai, agents, tracing, debugging, monitoring, observability]
author: toantranct
description: Hướng dẫn chi tiết về Tracing và Debugging trong OpenAI Agents SDK. Từ built-in tracing đến custom processors - giám sát và tối ưu hiệu suất agents trong production.
---

Khi phát triển AI agents, việc hiểu được chuyện gì đang diễn ra "bên trong" agent là vô cùng quan trọng. Tại sao agent lại chọn tool này thay vì tool khác? Handoff xảy ra như thế nào? Performance có bị bottleneck ở đâu không? **Tracing và Debugging** chính là chìa khóa để trả lời những câu hỏi này.

Tracing không chỉ là việc "ghi log" đơn thuần mà là nghệ thuật **quan sát và hiểu rõ hành vi** của agent systems. Giống như một bác sĩ dùng X-ray để nhìn vào cơ thể, tracing giúp chúng ta nhìn vào "tâm trí" của agents để tối ưu hóa và cải thiện performance.

Hôm nay chúng ta sẽ khám phá cách biến agents từ những "hộp đen bí ẩn" thành những "hệ thống trong suốt" mà bạn có thể hiểu, giám sát và tối ưu hóa một cách hiệu quả.

## Tracing Là Gì Và Tại Sao Quan Trọng?

### Khái Niệm Cơ Bản

**Tracing** trong OpenAI Agents SDK là hệ thống thu thập và ghi lại tất cả hoạt động của agents - từ LLM calls, tool executions, handoffs, cho đến guardrails checks. Mỗi hoạt động được ghi thành **spans**, và tập hợp các spans tạo thành một **trace** hoàn chỉnh.

```python
# Ví dụ một trace đơn giản
Trace: "Customer Support Query"
├── Span: Agent Run (Triage Agent) - 2.3s
│   ├── Span: LLM Generation - 1.2s  
│   ├── Span: Tool Call (search_knowledge_base) - 0.8s
│   └── Span: Handoff to Billing Agent - 0.3s
└── Span: Agent Run (Billing Agent) - 1.7s
    ├── Span: LLM Generation - 0.9s
    └── Span: Tool Call (check_account_status) - 0.8s
```

### Tại Sao Cần Tracing?

🔍 **Visibility** - Biết agents đang làm gì và tại sao  
🐛 **Debugging** - Tìm ra nguyên nhân của bugs và unexpected behaviors  
⚡ **Performance** - Identify bottlenecks và optimize response times  
📊 **Analytics** - Hiểu usage patterns và user behaviors  
🛡️ **Monitoring** - Detect issues trước khi chúng affect users  

### Built-in Tracing Architecture

OpenAI Agents SDK có tracing được enable mặc định với architecture như sau:

```python
from agents import Agent, Runner, trace, enable_verbose_stdout_logging

# Enable detailed logging để xem tracing info
enable_verbose_stdout_logging()

async def demo_built_in_tracing():
    """Demo built-in tracing capabilities"""
    
    # Trace sẽ tự động được tạo cho mỗi Runner.run() call
    agent = Agent(
        name="Demo Agent",
        instructions="You are a helpful assistant for demonstration purposes."
    )
    
    print("🔍 **Built-in Tracing Demo**")
    print("Tracing is automatically enabled for all agent runs.\n")
    
    # Simple agent run - tracing happens automatically
    result = await Runner.run(agent, "Explain what tracing is in simple terms")
    
    print(f"\n📊 **Trace Info:**")
    print(f"• Raw Responses: {len(result.raw_responses)}")
    print(f"• New Items: {len(result.new_items)}")
    print(f"• Final Output Length: {len(result.final_output)} characters")
    
    return result

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_built_in_tracing())
```

## Custom Tracing Processors

### Tạo Custom Trace Processor

Để có control tốt hơn về tracing data, bạn có thể tạo custom trace processors:

```python
import json
import time
from typing import Dict, Any, List
from datetime import datetime
from agents.tracing import TraceProcessor, Trace, Span
from agents import Agent, Runner, add_trace_processor

class DetailedTraceProcessor(TraceProcessor):
    """Custom trace processor để collect detailed analytics"""
    
    def __init__(self):
        self.traces: List[Dict[str, Any]] = []
        self.performance_metrics: Dict[str, List[float]] = {}
        self.error_logs: List[Dict[str, Any]] = []
    
    def process_trace(self, trace: Trace):
        """Process completed trace"""
        
        trace_data = {
            "trace_id": trace.trace_id,
            "workflow_name": trace.workflow_name or "Unknown",
            "started_at": trace.started_at.isoformat() if trace.started_at else None,
            "ended_at": trace.ended_at.isoformat() if trace.ended_at else None,
            "duration_ms": self._calculate_duration(trace),
            "spans": [],
            "metadata": trace.metadata or {}
        }
        
        # Process all spans trong trace
        for span in trace.spans:
            span_data = self._process_span(span)
            trace_data["spans"].append(span_data)
        
        # Store trace
        self.traces.append(trace_data)
        
        # Update performance metrics
        self._update_metrics(trace_data)
        
        # Log trace summary
        self._log_trace_summary(trace_data)
    
    def _process_span(self, span: Span) -> Dict[str, Any]:
        """Process individual span"""
        
        span_data = {
            "span_id": span.span_id,
            "parent_id": span.parent_id,
            "operation_name": span.operation_name,
            "started_at": span.started_at.isoformat() if span.started_at else None,
            "ended_at": span.ended_at.isoformat() if span.ended_at else None,
            "duration_ms": self._calculate_span_duration(span),
            "tags": span.tags or {},
            "logs": span.logs or []
        }
        
        # Extract specific information based on span type
        if span.operation_name == "agent_run":
            span_data["agent_info"] = {
                "agent_name": span.tags.get("agent.name"),
                "model": span.tags.get("agent.model"),
                "tools_count": span.tags.get("agent.tools_count", 0)
            }
        
        elif span.operation_name == "llm_generation":
            span_data["llm_info"] = {
                "model": span.tags.get("llm.model"),
                "input_tokens": span.tags.get("llm.input_tokens"),
                "output_tokens": span.tags.get("llm.output_tokens"),
                "total_tokens": span.tags.get("llm.total_tokens")
            }
        
        elif span.operation_name == "tool_call":
            span_data["tool_info"] = {
                "tool_name": span.tags.get("tool.name"),
                "tool_type": span.tags.get("tool.type"),
                "success": span.tags.get("tool.success", True)
            }
        
        # Check for errors
        if span.tags and span.tags.get("error"):
            error_info = {
                "trace_id": span.trace_id,
                "span_id": span.span_id,
                "error": span.tags.get("error"),
                "timestamp": datetime.now().isoformat()
            }
            self.error_logs.append(error_info)
            span_data["has_error"] = True
        
        return span_data
    
    def _calculate_duration(self, trace: Trace) -> float:
        """Calculate trace duration in milliseconds"""
        if trace.started_at and trace.ended_at:
            return (trace.ended_at - trace.started_at).total_seconds() * 1000
        return 0
    
    def _calculate_span_duration(self, span: Span) -> float:
        """Calculate span duration in milliseconds"""
        if span.started_at and span.ended_at:
            return (span.ended_at - span.started_at).total_seconds() * 1000
        return 0
    
    def _update_metrics(self, trace_data: Dict[str, Any]):
        """Update performance metrics"""
        
        workflow = trace_data["workflow_name"]
        duration = trace_data["duration_ms"]
        
        if workflow not in self.performance_metrics:
            self.performance_metrics[workflow] = []
        
        self.performance_metrics[workflow].append(duration)
        
        # Keep only last 100 measurements per workflow
        if len(self.performance_metrics[workflow]) > 100:
            self.performance_metrics[workflow] = self.performance_metrics[workflow][-100:]
    
    def _log_trace_summary(self, trace_data: Dict[str, Any]):
        """Log trace summary cho monitoring"""
        
        total_spans = len(trace_data["spans"])
        duration = trace_data["duration_ms"]
        workflow = trace_data["workflow_name"]
        
        print(f"📈 Trace completed: {workflow} | {duration:.1f}ms | {total_spans} spans")
        
        # Log performance warnings
        if duration > 5000:  # More than 5 seconds
            print(f"⚠️  Slow trace detected: {workflow} took {duration:.1f}ms")
        
        # Log errors
        errors_in_trace = [span for span in trace_data["spans"] if span.get("has_error")]
        if errors_in_trace:
            print(f"❌ Errors detected in trace: {len(errors_in_trace)} spans with errors")
    
    def get_analytics_report(self) -> Dict[str, Any]:
        """Generate comprehensive analytics report"""
        
        if not self.traces:
            return {"message": "No traces collected yet"}
        
        # Calculate summary statistics
        total_traces = len(self.traces)
        total_errors = len(self.error_logs)
        
        # Performance statistics
        all_durations = [trace["duration_ms"] for trace in self.traces]
        avg_duration = sum(all_durations) / len(all_durations) if all_durations else 0
        
        # Workflow statistics
        workflow_stats = {}
        for trace in self.traces:
            workflow = trace["workflow_name"]
            if workflow not in workflow_stats:
                workflow_stats[workflow] = {"count": 0, "total_duration": 0}
            
            workflow_stats[workflow]["count"] += 1
            workflow_stats[workflow]["total_duration"] += trace["duration_ms"]
        
        # Calculate averages cho mỗi workflow
        for workflow, stats in workflow_stats.items():
            stats["avg_duration"] = stats["total_duration"] / stats["count"]
        
        # Most common operations
        all_operations = []
        for trace in self.traces:
            for span in trace["spans"]:
                all_operations.append(span["operation_name"])
        
        operation_counts = {}
        for op in all_operations:
            operation_counts[op] = operation_counts.get(op, 0) + 1
        
        return {
            "summary": {
                "total_traces": total_traces,
                "total_errors": total_errors,
                "error_rate": total_errors / total_traces if total_traces > 0 else 0,
                "avg_duration_ms": avg_duration
            },
            "workflow_performance": workflow_stats,
            "operation_frequency": operation_counts,
            "recent_errors": self.error_logs[-10:],  # Last 10 errors
            "generated_at": datetime.now().isoformat()
        }

# Setup custom tracing
def setup_custom_tracing():
    """Setup custom trace processor"""
    
    detailed_processor = DetailedTraceProcessor()
    add_trace_processor(detailed_processor)
    
    return detailed_processor

# Demo custom tracing
async def demo_custom_tracing():
    """Demo custom tracing với detailed analytics"""
    
    print("🔧 **Custom Tracing Demo**\n")
    
    # Setup custom processor
    trace_processor = setup_custom_tracing()
    
    # Create agent với tools để generate interesting traces
    from agents import function_tool
    
    @function_tool
    def calculate_tax(amount: float, rate: float = 0.1) -> float:
        """Calculate tax on an amount"""
        return amount * rate
    
    @function_tool
    def format_currency(amount: float, currency: str = "VND") -> str:
        """Format amount as currency"""
        if currency == "VND":
            return f"{amount:,.0f} ₫"
        elif currency == "USD":
            return f"${amount:,.2f}"
        else:
            return f"{amount:.2f} {currency}"
    
    tax_agent = Agent(
        name="Tax Calculator",
        instructions="""
        Bạn là một chuyên gia tính thuế.
        
        Capabilities:
        - Tính thuế cho các khoản thu nhập
        - Format currency theo different formats
        - Provide tax advice và explanations
        
        Always:
        - Use tools để tính toán chính xác
        - Explain calculations step by step  
        - Format results clearly
        """,
        tools=[calculate_tax, format_currency]
    )
    
    # Test scenarios với different complexity levels
    test_scenarios = [
        {
            "name": "Simple Tax Calculation",
            "query": "Tính thuế cho thu nhập 50,000,000 VND với tax rate 10%"
        },
        {
            "name": "Multi-step Calculation",
            "query": "Tôi có thu nhập $5000 USD. Tính thuế 15% và convert sang VND (tỷ giá 24,000)"
        },
        {
            "name": "Complex Scenario",
            "query": "So sánh tax burden giữa thu nhập 100 triệu VND và $4000 USD với tax rates khác nhau"
        }
    ]
    
    for scenario in test_scenarios:
        print(f"💼 **Scenario:** {scenario['name']}")
        print(f"📝 **Query:** {scenario['query']}")
        
        # Run với custom workflow name để tracking
        with trace(workflow_name=scenario['name']):
            result = await Runner.run(tax_agent, scenario['query'])
            print(f"💡 **Result:** {result.final_output[:200]}...")
        
        print("-" * 60 + "\n")
    
    # Generate analytics report
    await asyncio.sleep(1)  # Wait for traces to be processed
    
    analytics = trace_processor.get_analytics_report()
    
    print("📊 **Analytics Report:**")
    print(f"📈 Total Traces: {analytics['summary']['total_traces']}")
    print(f"⏱️  Average Duration: {analytics['summary']['avg_duration_ms']:.1f}ms")
    print(f"❌ Error Rate: {analytics['summary']['error_rate']:.1%}")
    
    print(f"\n🎯 **Workflow Performance:**")
    for workflow, stats in analytics['workflow_performance'].items():
        print(f"   • {workflow}: {stats['count']} runs, avg {stats['avg_duration']:.1f}ms")
    
    print(f"\n🔧 **Operation Frequency:**")
    for operation, count in analytics['operation_frequency'].items():
        print(f"   • {operation}: {count} times")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_custom_tracing())
```

## Advanced Debug Techniques

### Debug Agent Decision Making

```python
from agents import Agent, Runner, trace, function_tool
from typing import Dict, Any, List
import json

class AgentDecisionDebugger:
    """Debug agent decision making process"""
    
    def __init__(self):
        self.decision_log: List[Dict[str, Any]] = []
        self.tool_usage_stats: Dict[str, int] = {}
        self.handoff_patterns: List[Dict[str, Any]] = []
    
    def log_decision(self, decision_type: str, context: Dict[str, Any]):
        """Log agent decision với context"""
        
        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "decision_type": decision_type, 
            "context": context,
            "session_id": context.get("session_id", "unknown")
        }
        
        self.decision_log.append(log_entry)
        
        # Update statistics
        if decision_type == "tool_selection":
            tool_name = context.get("tool_name")
            if tool_name:
                self.tool_usage_stats[tool_name] = self.tool_usage_stats.get(tool_name, 0) + 1
        
        elif decision_type == "handoff":
            self.handoff_patterns.append({
                "from_agent": context.get("from_agent"),
                "to_agent": context.get("to_agent"),
                "reason": context.get("reason"),
                "timestamp": log_entry["timestamp"]
            })
    
    def analyze_decision_patterns(self) -> Dict[str, Any]:
        """Analyze decision making patterns"""
        
        if not self.decision_log:
            return {"message": "No decisions logged yet"}
        
        # Decision type frequency
        decision_types = {}
        for decision in self.decision_log:
            dtype = decision["decision_type"]
            decision_types[dtype] = decision_types.get(dtype, 0) + 1
        
        # Tool selection patterns
        tool_analysis = {}
        if self.tool_usage_stats:
            total_tool_calls = sum(self.tool_usage_stats.values())
            for tool, count in self.tool_usage_stats.items():
                tool_analysis[tool] = {
                    "usage_count": count,
                    "usage_percentage": (count / total_tool_calls) * 100
                }
        
        # Handoff analysis
        handoff_analysis = {}
        if self.handoff_patterns:
            for handoff in self.handoff_patterns:
                pattern = f"{handoff['from_agent']} -> {handoff['to_agent']}"
                handoff_analysis[pattern] = handoff_analysis.get(pattern, 0) + 1
        
        return {
            "total_decisions": len(self.decision_log),
            "decision_type_frequency": decision_types,
            "tool_usage_analysis": tool_analysis,
            "handoff_patterns": handoff_analysis,
            "most_used_tool": max(self.tool_usage_stats.items(), key=lambda x: x[1])[0] if self.tool_usage_stats else None,
            "analysis_timestamp": datetime.now().isoformat()
        }

# Debug-enabled agents
class DebugAgent:
    """Agent wrapper với debugging capabilities"""
    
    def __init__(self, base_agent: Agent, debugger: AgentDecisionDebugger):
        self.base_agent = base_agent
        self.debugger = debugger
        self.session_id = f"debug_session_{int(time.time())}"
    
    async def run_with_debug(self, user_input: str) -> Dict[str, Any]:
        """Run agent với comprehensive debugging"""
        
        print(f"🐛 **Debug Session:** {self.session_id}")
        print(f"📝 **Input:** {user_input}")
        
        # Log initial decision context
        self.debugger.log_decision("input_processing", {
            "session_id": self.session_id,
            "input_length": len(user_input),
            "agent_name": self.base_agent.name,
            "tools_available": len(self.base_agent.tools or [])
        })
        
        # Run với tracing
        with trace(workflow_name=f"Debug: {self.base_agent.name}"):
            start_time = time.time()
            
            try:
                result = await Runner.run(self.base_agent, user_input)
                
                execution_time = time.time() - start_time
                
                # Log successful completion
                self.debugger.log_decision("execution_complete", {
                    "session_id": self.session_id,
                    "execution_time": execution_time,
                    "output_length": len(result.final_output),
                    "tools_called": len([item for item in result.new_items if item.type == "tool_call_item"]),
                    "success": True
                })
                
                # Analyze tools used
                for item in result.new_items:
                    if item.type == "tool_call_item":
                        self.debugger.log_decision("tool_selection", {
                            "session_id": self.session_id,
                            "tool_name": item.tool_name if hasattr(item, 'tool_name') else "unknown",
                            "execution_order": len([i for i in result.new_items[:result.new_items.index(item)] if i.type == "tool_call_item"]) + 1
                        })
                
                print(f"✅ **Success:** {execution_time:.2f}s")
                
                return {
                    "success": True,
                    "result": result,
                    "debug_info": {
                        "session_id": self.session_id,
                        "execution_time": execution_time,
                        "tools_used": len([item for item in result.new_items if item.type == "tool_call_item"])
                    }
                }
                
            except Exception as e:
                execution_time = time.time() - start_time
                
                # Log error
                self.debugger.log_decision("execution_error", {
                    "session_id": self.session_id,
                    "error": str(e),
                    "error_type": type(e).__name__,
                    "execution_time": execution_time
                })
                
                print(f"❌ **Error:** {str(e)}")
                
                return {
                    "success": False,
                    "error": str(e),
                    "debug_info": {
                        "session_id": self.session_id,
                        "execution_time": execution_time,
                        "error_type": type(e).__name__
                    }
                }

# Demo debug techniques
async def demo_debug_techniques():
    """Demo advanced debugging techniques"""
    
    print("🔍 **Advanced Agent Debugging Demo**\n")
    
    # Setup debugger
    debugger = AgentDecisionDebugger()
    
    # Create agents với different behaviors
    @function_tool
    def search_product_database(query: str, category: str = "all") -> str:
        """Search for products in database"""
        # Simulate database search
        products = {
            "laptop": ["Dell XPS 13", "MacBook Pro", "ThinkPad X1"],
            "phone": ["iPhone 15", "Samsung Galaxy S24", "Google Pixel 8"],
            "headphones": ["AirPods Pro", "Sony WH-1000XM5", "Bose QuietComfort"]
        }
        
        if category in products:
            return f"Found products in {category}: {', '.join(products[category])}"
        else:
            return f"Search results for '{query}': Found 12 products across all categories"
    
    @function_tool
    def check_inventory(product_name: str) -> str:
        """Check product inventory status"""
        # Simulate inventory check
        import random
        stock = random.randint(0, 50)
        status = "In Stock" if stock > 5 else "Low Stock" if stock > 0 else "Out of Stock"
        return f"{product_name}: {status} ({stock} units available)"
    
    @function_tool
    def get_price_info(product_name: str) -> str:
        """Get product pricing information"""
        # Simulate price lookup
        import random
        base_price = random.randint(500, 5000) * 1000  # VND
        discount = random.choice([0, 5, 10, 15, 20])
        
        if discount > 0:
            discounted_price = base_price * (1 - discount/100)
            return f"{product_name}: {base_price:,} VND (Sale: {discounted_price:,} VND - {discount}% off)"
        else:
            return f"{product_name}: {base_price:,} VND"
    
    # Create product assistant
    product_assistant = Agent(
        name="Product Research Assistant",
        instructions="""
        Bạn là chuyên gia tư vấn sản phẩm thông minh.
        
        Process:
        1. Hiểu requirements của customer
        2. Search relevant products
        3. Check inventory status
        4. Get pricing information
        5. Provide comprehensive recommendation
        
        Decision making:
        - Always search products first
        - Check inventory cho popular items
        - Get pricing cho final recommendations
        - Consider customer budget và needs
        
        Be helpful, thorough, and data-driven.
        """,
        tools=[search_product_database, check_inventory, get_price_info]
    )
    
    # Wrap với debug capabilities
    debug_agent = DebugAgent(product_assistant, debugger)
    
    # Test scenarios với increasing complexity
    debug_scenarios = [
        "Tôi cần một laptop cho học lập trình, budget khoảng 30 triệu",
        "Tìm tai nghe bluetooth tốt nhất hiện tại, không quan tâm giá",
        "So sánh iPhone 15 và Samsung Galaxy S24, cả về giá và tính năng",
        "Recommend smartphone tầm trung, dưới 15 triệu, pin tốt"
    ]
    
    for i, scenario in enumerate(debug_scenarios, 1):
        print(f"🎯 **Debug Scenario {i}:**")
        
        debug_result = await debug_agent.run_with_debug(scenario)
        
        if debug_result["success"]:
            result = debug_result["result"]
            debug_info = debug_result["debug_info"]
            
            print(f"📊 **Debug Info:**")
            print(f"   • Session: {debug_info['session_id']}")
            print(f"   • Execution Time: {debug_info['execution_time']:.2f}s")
            print(f"   • Tools Used: {debug_info['tools_used']}")
            
            print(f"💡 **Response:** {result.final_output[:300]}...")
        
        else:
            print(f"❌ **Failed:** {debug_result['error']}")
        
        print("=" * 70 + "\n")
    
    # Analyze decision patterns
    analysis = debugger.analyze_decision_patterns()
    
    print("📈 **Decision Analysis:**")
    print(f"📊 Total Decisions: {analysis['total_decisions']}")
    
    if analysis.get('tool_usage_analysis'):
        print(f"\n🔧 **Tool Usage Patterns:**")
        for tool, stats in analysis['tool_usage_analysis'].items():
            print(f"   • {tool}: {stats['usage_count']} times ({stats['usage_percentage']:.1f}%)")
        
        if analysis.get('most_used_tool'):
            print(f"\n⭐ **Most Used Tool:** {analysis['most_used_tool']}")
    
    if analysis.get('decision_type_frequency'):
        print(f"\n🎯 **Decision Type Frequency:**")
        for dtype, count in analysis['decision_type_frequency'].items():
            print(f"   • {dtype}: {count} times")

if __name__ == "__main__":
    import time
    from datetime import datetime
    import asyncio
    asyncio.run(demo_debug_techniques())
```

## Production Monitoring và Alerting

### Comprehensive Monitoring System

```python
import asyncio
import time
from typing import Dict, Any, List, Optional
from datetime import datetime, timedelta
from dataclasses import dataclass, field
from enum import Enum
import statistics

class AlertLevel(Enum):
    INFO = "info"
    WARNING = "warning"  
    ERROR = "error"
    CRITICAL = "critical"

@dataclass
class Alert:
    level: AlertLevel
    message: str
    metric_name: str
    current_value: float
    threshold: float
    timestamp: datetime = field(default_factory=datetime.now)
    
    def to_dict(self) -> Dict[str, Any]:
        return {
            "level": self.level.value,
            "message": self.message,
            "metric_name": self.metric_name,
            "current_value": self.current_value,
            "threshold": self.threshold,
            "timestamp": self.timestamp.isoformat()
        }

class ProductionMonitor:
    """Comprehensive production monitoring system"""
    
    def __init__(self):
        # Metrics storage
        self.metrics: Dict[str, List[float]] = {}
        self.metric_timestamps: Dict[str, List[datetime]] = {}
        
        # Alert configuration
        self.alert_thresholds = {
            "response_time_ms": {"warning": 2000, "error": 5000, "critical": 10000},
            "error_rate": {"warning": 0.05, "error": 0.10, "critical": 0.20},
            "token_throughput": {"warning": 5, "error": 2, "critical": 1},  # tokens/sec
            "concurrent_requests": {"warning": 50, "error": 100, "critical": 150}
        }
        
        # Alert history
        self.alerts: List[Alert] = []
        self.alert_cooldown: Dict[str, datetime] = {}
        
        # System health
        self.last_health_check = datetime.now()
        self.health_status = "healthy"
        
        print("🚨 Production Monitor initialized")
    
    def record_metric(self, metric_name: str, value: float):
        """Record a metric value"""
        
        if metric_name not in self.metrics:
            self.metrics[metric_name] = []
            self.metric_timestamps[metric_name] = []
        
        self.metrics[metric_name].append(value)
        self.metric_timestamps[metric_name].append(datetime.now())
        
        # Keep only last 1000 data points
        if len(self.metrics[metric_name]) > 1000:
            self.metrics[metric_name] = self.metrics[metric_name][-1000:]
            self.metric_timestamps[metric_name] = self.metric_timestamps[metric_name][-1000:]
        
        # Check for alerts
        self._check_alerts(metric_name, value)
    
    def _check_alerts(self, metric_name: str, current_value: float):
        """Check if metric value triggers any alerts"""
        
        if metric_name not in self.alert_thresholds:
            return
        
        thresholds = self.alert_thresholds[metric_name]
        
        # Check alert levels (từ critical xuống info)
        alert_level = None
        threshold_value = None
        
        if "critical" in thresholds and current_value >= thresholds["critical"]:
            alert_level = AlertLevel.CRITICAL
            threshold_value = thresholds["critical"]
        elif "error" in thresholds and current_value >= thresholds["error"]:
            alert_level = AlertLevel.ERROR  
            threshold_value = thresholds["error"]
        elif "warning" in thresholds and current_value >= thresholds["warning"]:
            alert_level = AlertLevel.WARNING
            threshold_value = thresholds["warning"]
        
        if alert_level:
            # Check cooldown để avoid spam alerts
            cooldown_key = f"{metric_name}_{alert_level.value}"
            now = datetime.now()
            
            if cooldown_key in self.alert_cooldown:
                if now - self.alert_cooldown[cooldown_key] < timedelta(minutes=5):
                    return  # Still in cooldown
            
            # Create alert
            alert = Alert(
                level=alert_level,
                message=f"{metric_name} is {current_value:.2f}, exceeds {alert_level.value} threshold of {threshold_value}",
                metric_name=metric_name,
                current_value=current_value,
                threshold=threshold_value
            )
            
            self.alerts.append(alert)
            self.alert_cooldown[cooldown_key] = now
            
            # Trigger alert notification
            self._trigger_alert(alert)
    
    def _trigger_alert(self, alert: Alert):
        """Trigger alert notification"""
        
        # Print alert (trong production sẽ gửi đến Slack, email, etc.)
        icon = {
            AlertLevel.INFO: "ℹ️",
            AlertLevel.WARNING: "⚠️", 
            AlertLevel.ERROR: "❌",
            AlertLevel.CRITICAL: "🚨"
        }
        
        print(f"\n{icon[alert.level]} **ALERT [{alert.level.value.upper()}]**")
        print(f"   📊 {alert.message}")
        print(f"   🕐 {alert.timestamp.strftime('%H:%M:%S')}")
        
        # Update system health status
        if alert.level in [AlertLevel.ERROR, AlertLevel.CRITICAL]:
            self.health_status = "degraded" if alert.level == AlertLevel.ERROR else "critical"
    
    def get_metric_summary(self, metric_name: str, minutes: int = 60) -> Dict[str, Any]:
        """Get metric summary for specified time window"""
        
        if metric_name not in self.metrics:
            return {"error": f"Metric {metric_name} not found"}
        
        # Filter data within time window
        now = datetime.now()
        cutoff_time = now - timedelta(minutes=minutes)
        
        values = []
        timestamps = []
        
        for i, timestamp in enumerate(self.metric_timestamps[metric_name]):
            if timestamp >= cutoff_time:
                values.append(self.metrics[metric_name][i])
                timestamps.append(timestamp)
        
        if not values:
            return {"message": f"No data for {metric_name} in last {minutes} minutes"}
        
        return {
            "metric_name": metric_name,
            "time_window_minutes": minutes,
            "data_points": len(values),
            "latest_value": values[-1],
            "min_value": min(values),
            "max_value": max(values),
            "avg_value": statistics.mean(values),
            "median_value": statistics.median(values),
            "std_dev": statistics.stdev(values) if len(values) > 1 else 0,
            "trend": self._calculate_trend(values),
            "last_updated": timestamps[-1].isoformat() if timestamps else None
        }
    
    def _calculate_trend(self, values: List[float]) -> str:
        """Calculate trend direction"""
        if len(values) < 2:
            return "insufficient_data"
        
        # Simple trend calculation
        first_half = values[:len(values)//2]
        second_half = values[len(values)//2:]
        
        first_avg = statistics.mean(first_half)
        second_avg = statistics.mean(second_half)
        
        diff_percent = ((second_avg - first_avg) / first_avg) * 100 if first_avg != 0 else 0
        
        if abs(diff_percent) < 5:
            return "stable"
        elif diff_percent > 0:
            return "increasing"
        else:
            return "decreasing"
    
    def get_system_health(self) -> Dict[str, Any]:
        """Get overall system health status"""
        
        recent_alerts = [a for a in self.alerts if datetime.now() - a.timestamp < timedelta(hours=1)]
        critical_alerts = [a for a in recent_alerts if a.level == AlertLevel.CRITICAL]
        error_alerts = [a for a in recent_alerts if a.level == AlertLevel.ERROR]
        
        # Calculate health score
        health_score = 100
        health_score -= len(critical_alerts) * 30
        health_score -= len(error_alerts) * 15
        health_score -= len([a for a in recent_alerts if a.level == AlertLevel.WARNING]) * 5
        health_score = max(0, health_score)
        
        return {
            "status": self.health_status,
            "health_score": health_score,
            "last_check": self.last_health_check.isoformat(),
            "alerts_last_hour": {
                "critical": len(critical_alerts),
                "error": len(error_alerts),
                "warning": len([a for a in recent_alerts if a.level == AlertLevel.WARNING]),
                "total": len(recent_alerts)
            },
            "metrics_tracked": len(self.metrics),
            "uptime_status": "operational"  # Trong production sẽ track actual uptime
        }
    
    def generate_dashboard_data(self) -> Dict[str, Any]:
        """Generate data cho monitoring dashboard"""
        
        dashboard = {
            "timestamp": datetime.now().isoformat(),
            "system_health": self.get_system_health(),
            "key_metrics": {},
            "recent_alerts": [alert.to_dict() for alert in self.alerts[-10:]],
            "metric_summaries": {}
        }
        
        # Get summaries cho key metrics
        key_metrics = ["response_time_ms", "error_rate", "token_throughput"]
        
        for metric in key_metrics:
            if metric in self.metrics:
                summary = self.get_metric_summary(metric, minutes=60)
                dashboard["metric_summaries"][metric] = summary
                
                # Add current value to key metrics
                if "latest_value" in summary:
                    dashboard["key_metrics"][metric] = summary["latest_value"]
        
        return dashboard

# Production monitoring integration
class MonitoredAgent:
    """Agent wrapper với production monitoring"""
    
    def __init__(self, agent: Agent, monitor: ProductionMonitor):
        self.agent = agent
        self.monitor = monitor
        self.request_count = 0
        self.concurrent_requests = 0
    
    async def run_with_monitoring(self, user_input: str) -> Dict[str, Any]:
        """Run agent với comprehensive monitoring"""
        
        self.request_count += 1
        self.concurrent_requests += 1
        
        # Record concurrent requests
        self.monitor.record_metric("concurrent_requests", self.concurrent_requests)
        
        start_time = time.time()
        tokens_generated = 0
        errors_occurred = 0
        
        try:
            # Run agent với monitoring
            result = await Runner.run(self.agent, user_input)
            
            # Calculate metrics
            response_time = (time.time() - start_time) * 1000  # ms
            tokens_generated = len(result.final_output.split())  # Approximate token count
            token_throughput = tokens_generated / (response_time / 1000) if response_time > 0 else 0
            
            # Record metrics
            self.monitor.record_metric("response_time_ms", response_time)
            self.monitor.record_metric("token_throughput", token_throughput)
            self.monitor.record_metric("error_rate", 0)  # Success
            
            return {
                "success": True,
                "result": result,
                "metrics": {
                    "response_time_ms": response_time,
                    "tokens_generated": tokens_generated,
                    "token_throughput": token_throughput
                }
            }
            
        except Exception as e:
            errors_occurred = 1
            response_time = (time.time() - start_time) * 1000
            
            # Record error metrics
            self.monitor.record_metric("response_time_ms", response_time)
            self.monitor.record_metric("error_rate", 1)  # Error occurred
            
            return {
                "success": False,
                "error": str(e),
                "metrics": {
                    "response_time_ms": response_time,
                    "errors": errors_occurred
                }
            }
            
        finally:
            self.concurrent_requests -= 1

# Demo production monitoring
async def demo_production_monitoring():
    """Demo comprehensive production monitoring"""
    
    print("📊 **Production Monitoring Demo**\n")
    
    # Setup monitoring
    monitor = ProductionMonitor()
    
    # Create monitored agent
    production_agent = Agent(
        name="Production Assistant",
        instructions="""
        Bạn là một production assistant được monitor 24/7.
        
        Provide helpful, accurate responses while maintaining high performance.
        Be concise but informative.
        """,
        tools=[]
    )
    
    monitored_agent = MonitoredAgent(production_agent, monitor)
    
    # Simulate production traffic
    print("🚀 Simulating production traffic...\n")
    
    test_queries = [
        "Explain machine learning briefly",
        "What is cloud computing?", 
        "How does blockchain work?",
        "Benefits of microservices architecture",
        "Database optimization techniques"
    ] * 3  # Repeat to generate more data
    
    # Process requests với varying delays để simulate real traffic
    for i, query in enumerate(test_queries):
        print(f"🔄 Request {i+1}: {query[:50]}...")
        
        # Simulate variable response times
        if i % 7 == 0:  # Occasionally slow request
            await asyncio.sleep(0.5)  # Add artificial delay
        
        result = await monitored_agent.run_with_monitoring(query)
        
        if result["success"]:
            metrics = result["metrics"]
            print(f"   ✅ {metrics['response_time_ms']:.0f}ms | {metrics['tokens_generated']} tokens | {metrics['token_throughput']:.1f} t/s")
        else:
            print(f"   ❌ Error: {result['error']}")
        
        # Small delay between requests
        await asyncio.sleep(0.1)
    
    print("\n" + "="*60 + "\n")
    
    # Generate dashboard
    dashboard = monitor.generate_dashboard_data()
    
    print("📈 **Production Dashboard:**")
    
    # System health
    health = dashboard["system_health"]
    print(f"🏥 **System Health:** {health['status'].upper()}")
    print(f"   • Health Score: {health['health_score']}/100")
    print(f"   • Metrics Tracked: {health['metrics_tracked']}")
    
    # Recent alerts
    recent_alerts = dashboard["recent_alerts"]
    if recent_alerts:
        print(f"\n🚨 **Recent Alerts:** ({len(recent_alerts)} alerts)")
        for alert in recent_alerts[-3:]:  # Show last 3
            print(f"   • {alert['level'].upper()}: {alert['message']}")
    else:
        print(f"\n✅ **No Recent Alerts**")
    
    # Key metrics
    key_metrics = dashboard["key_metrics"]
    if key_metrics:
        print(f"\n📊 **Key Metrics:**")
        for metric, value in key_metrics.items():
            if metric == "response_time_ms":
                print(f"   • Response Time: {value:.0f}ms")
            elif metric == "token_throughput":
                print(f"   • Token Throughput: {value:.1f} tokens/sec")
            elif metric == "error_rate":
                print(f"   • Error Rate: {value:.1%}")
    
    # Metric summaries
    summaries = dashboard["metric_summaries"]
    if summaries:
        print(f"\n📈 **Metric Trends (Last Hour):**")
        for metric, summary in summaries.items():
            trend_icon = {"increasing": "📈", "decreasing": "📉", "stable": "➡️"}.get(summary["trend"], "❓")
            print(f"   • {metric}: {summary['avg_value']:.1f} avg, {summary['trend']} {trend_icon}")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_production_monitoring())
```

## Khắc Phục Sự Cố Thường Gặp

### Common Issues và Solutions

```python
from typing import List, Dict, Any, Optional
import re
import json

class TroubleshootingGuide:
    """Hướng dẫn khắc phục sự cố thường gặp"""
    
    def __init__(self):
        self.issue_patterns = {
            "slow_response": {
                "symptoms": ["response_time > 5000ms", "timeout errors", "user complaints about speed"],
                "causes": ["complex prompts", "too many tools", "inefficient tool implementations", "network latency"],
                "solutions": [
                    "Optimize agent instructions",
                    "Reduce number of tools",
                    "Implement tool caching",
                    "Use streaming responses",
                    "Add performance monitoring"
                ]
            },
            
            "high_error_rate": {
                "symptoms": ["error_rate > 10%", "frequent exceptions", "incomplete responses"],
                "causes": ["API rate limits", "invalid tool schemas", "network issues", "malformed inputs"],
                "solutions": [
                    "Implement retry logic with exponential backoff",
                    "Validate tool schemas",
                    "Add input sanitization",
                    "Improve error handling",
                    "Monitor API quotas"
                ]
            },
            
            "unexpected_behavior": {
                "symptoms": ["wrong tool selection", "poor handoff decisions", "irrelevant responses"],
                "causes": ["unclear instructions", "insufficient context", "tool description ambiguity"],
                "solutions": [
                    "Refine agent instructions",
                    "Improve tool descriptions",
                    "Add more context to prompts",
                    "Use examples in instructions",
                    "Implement decision logging"
                ]
            },
            
            "resource_exhaustion": {
                "symptoms": ["memory errors", "CPU spikes", "concurrent request failures"],
                "causes": ["memory leaks", "inefficient algorithms", "too many concurrent requests"],
                "solutions": [
                    "Implement connection pooling",
                    "Add request queuing",
                    "Optimize memory usage",
                    "Scale horizontally",
                    "Add resource monitoring"
                ]
            }
        }
    
    def diagnose_issue(self, symptoms: List[str], metrics: Dict[str, float] = None) -> Dict[str, Any]:
        """Chẩn đoán vấn đề dựa trên symptoms và metrics"""
        
        potential_issues = []
        
        # Check từng issue pattern
        for issue_type, pattern in self.issue_patterns.items():
            match_score = 0
            
            # Check symptoms
            for symptom in symptoms:
                if any(pattern_symptom in symptom.lower() for pattern_symptom in pattern["symptoms"]):
                    match_score += 1
            
            # Check metrics nếu có
            if metrics:
                if issue_type == "slow_response" and metrics.get("response_time_ms", 0) > 5000:
                    match_score += 2
                elif issue_type == "high_error_rate" and metrics.get("error_rate", 0) > 0.1:
                    match_score += 2
            
            if match_score > 0:
                potential_issues.append({
                    "issue_type": issue_type,
                    "match_score": match_score,
                    "confidence": min(match_score / len(pattern["symptoms"]), 1.0),
                    "pattern": pattern
                })
        
        # Sort by confidence
        potential_issues.sort(key=lambda x: x["confidence"], reverse=True)
        
        return {
            "diagnosis": potential_issues,
            "top_issue": potential_issues[0] if potential_issues else None,
            "recommendations": self._generate_recommendations(potential_issues)
        }
    
    def _generate_recommendations(self, issues: List[Dict[str, Any]]) -> List[str]:
        """Generate actionable recommendations"""
        
        if not issues:
            return ["No specific issues detected. Continue monitoring system health."]
        
        recommendations = []
        
        # Get top 2 issues
        for issue in issues[:2]:
            issue_type = issue["issue_type"]
            solutions = issue["pattern"]["solutions"]
            
            recommendations.append(f"For {issue_type.replace('_', ' ')}:")
            for solution in solutions[:3]:  # Top 3 solutions
                recommendations.append(f"  • {solution}")
        
        return recommendations
    
    def get_troubleshooting_checklist(self, issue_type: str) -> List[str]:
        """Get step-by-step troubleshooting checklist"""
        
        checklists = {
            "slow_response": [
                "Check average response time metrics",
                "Review agent instructions for complexity",
                "Count number of tools available to agent",
                "Test individual tool performance",
                "Monitor network latency",
                "Enable streaming if not already active",
                "Review tracing data for bottlenecks"
            ],
            
            "high_error_rate": [
                "Check error logs for common patterns",
                "Verify API keys and authentication",
                "Test tool schemas with sample data",
                "Review input validation logic",
                "Check rate limit status",
                "Verify network connectivity",
                "Test error recovery mechanisms"
            ],
            
            "unexpected_behavior": [
                "Review agent instructions for clarity",
                "Test with sample inputs",
                "Check tool descriptions accuracy",
                "Verify context is being passed correctly",
                "Review handoff logic",
                "Test edge cases",
                "Enable decision logging"
            ],
            
            "resource_exhaustion": [
                "Monitor memory usage patterns",
                "Check CPU utilization",
                "Review concurrent request limits",
                "Analyze connection pool usage",
                "Check for memory leaks",
                "Review garbage collection metrics",
                "Consider horizontal scaling"
            ]
        }
        
        return checklists.get(issue_type, ["No specific checklist available for this issue type."])

# Demo troubleshooting system
async def demo_troubleshooting():
    """Demo troubleshooting system"""
    
    print("🔧 **Agent Troubleshooting Demo**\n")
    
    troubleshooter = TroubleshootingGuide()
    
    # Test scenarios
    scenarios = [
        {
            "name": "Slow Performance Issue",
            "symptoms": ["responses taking over 10 seconds", "users complaining about speed", "timeout errors"],
            "metrics": {"response_time_ms": 8500, "error_rate": 0.02}
        },
        {
            "name": "High Error Rate",
            "symptoms": ["many failed requests", "API errors", "incomplete responses"],
            "metrics": {"response_time_ms": 2000, "error_rate": 0.15}
        },
        {
            "name": "Unexpected Agent Behavior", 
            "symptoms": ["wrong tool selections", "irrelevant responses", "poor handoff decisions"],
            "metrics": {"response_time_ms": 1500, "error_rate": 0.05}
        },
        {
            "name": "Resource Issues",
            "symptoms": ["memory errors", "CPU spikes", "concurrent request failures"],
            "metrics": {"response_time_ms": 3000, "error_rate": 0.08, "memory_usage": 0.95}
        }
    ]
    
    for scenario in scenarios:
        print(f"🚨 **Scenario:** {scenario['name']}")
        print(f"📋 **Symptoms:** {', '.join(scenario['symptoms'])}")
        print(f"📊 **Metrics:** {scenario['metrics']}")
        
        # Diagnose issue
        diagnosis = troubleshooter.diagnose_issue(scenario['symptoms'], scenario['metrics'])
        
        if diagnosis['top_issue']:
            top_issue = diagnosis['top_issue']
            print(f"\n🎯 **Primary Issue:** {top_issue['issue_type'].replace('_', ' ').title()}")
            print(f"📈 **Confidence:** {top_issue['confidence']:.0%}")
            
            # Show recommendations
            print(f"\n💡 **Recommendations:**")
            for rec in diagnosis['recommendations']:
                print(f"   {rec}")
            
            # Show troubleshooting checklist
            checklist = troubleshooter.get_troubleshooting_checklist(top_issue['issue_type'])
            print(f"\n✅ **Troubleshooting Checklist:**")
            for i, step in enumerate(checklist, 1):
                print(f"   {i}. {step}")
        
        else:
            print(f"\n❓ **No specific issues detected**")
        
        print("\n" + "="*70 + "\n")
    
    # Show general best practices
    print("📋 **General Best Practices:**")
    best_practices = [
        "Monitor key metrics continuously (response time, error rate, throughput)",
        "Implement comprehensive logging and tracing",
        "Set up automated alerts for critical thresholds",
        "Regular performance testing under load",
        "Keep agent instructions clear and specific",
        "Validate all tool schemas and implementations",
        "Implement graceful error handling and recovery",
        "Document known issues and their solutions"
    ]
    
    for i, practice in enumerate(best_practices, 1):
        print(f"   {i}. {practice}")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_troubleshooting())
```

## Những Điều Quan Trọng Cần Nhớ

### Kiến Thức Cốt Lõi

✅ **Tracing tự động được bật** - Tất cả agent runs đều được theo dõi mặc định  
✅ **Hai loại events chính** - Raw response events và run item events  
✅ **Custom processors mạnh mẽ** - Có thể tạo monitoring systems phức tạp  
✅ **Production monitoring cần thiết** - Không thể thiếu cho hệ thống thực tế  
✅ **Debug techniques đa dạng** - Từ decision logging đến performance analysis  

### Kinh Nghiệm Thực Tiễn

🎯 **Trải Nghiệm Người Dùng:**
- Hiển thị feedback ngay lập tức khi có thể
- Dùng các chỉ báo tiến độ rõ ràng  
- Cho phép người dùng dừng các phản hồi dài
- Cung cấp thông tin về những gì đang xảy ra
- Xử lý lỗi một cách nhẹ nhàng với các tùy chọn khôi phục

⚡ **Hiệu Suất:**
- Theo dõi thời gian đến token đầu tiên như một chỉ số quan trọng
- Tối ưu hóa để có tốc độ token ổn định
- Triển khai các chiến lược đệm thông minh
- Lưu cache các phản hồi được yêu cầu thường xuyên
- Sử dụng connection pooling để duy trì hiệu suất

🔧 **Triển Khai:**
- Xử lý cả raw và structured streaming events
- Triển khai khôi phục lỗi toàn diện
- Thêm giám sát hiệu suất từ đầu
- Thiết kế cho cả giao diện console và web
- Kiểm tra với các điều kiện mạng khác nhau

### Hướng Dẫn Triển Khai Sản Xuất

📊 **Thiết Lập Giám Sát:**
- Theo dõi thời gian đến token đầu tiên cho tất cả yêu cầu
- Giám sát tốc độ token và tỷ lệ lỗi
- Thiết lập cảnh báo cho sự suy giảm hiệu suất
- Ghi lại các gián đoạn streaming và thành công khôi phục
- Đo lường sự tương tác của người dùng với streaming vs non-streaming

🛡️ **Xử Lý Lỗi:**
- Triển khai exponential backoff cho việc thử lại
- Bảo tồn các phản hồi một phần trong khi có lỗi
- Cung cấp thông báo lỗi có ý nghĩa cho người dùng
- Cho phép các tùy chọn thử lại thủ công
- Theo dõi các mẫu lỗi để xác định các vấn đề hệ thống

🔄 **Chiến Lược Tối Ưu:**
- Làm ấm trước các kết nối để giảm độ trễ
- Triển khai caching thông minh cho các truy vấn lặp lại
- Sử dụng connection multiplexing khi có thể
- Tối ưu hóa cài đặt model cho hiệu suất streaming
- Xem xét triển khai edge để giảm độ trễ mạng

### Những Lỗi Thường Gặp Và Cách Khắc Phục

❌ **Bỏ Qua Độ Trễ Token Đầu Tiên** → Giám sát và tối ưu thời gian đến token đầu tiên  
❌ **Không Có Khôi Phục Lỗi** → Triển khai cơ chế thử lại mạnh mẽ  
❌ **Chỉ Báo Tiến Độ Kém** → Hiển thị tiến độ và cập nhật trạng thái rõ ràng  
❌ **Làm Choáng Ngợp Người Dùng** → Cân bằng thông tin với khả năng sử dụng  
❌ **Không Giám Sát Hiệu Suất** → Theo dõi các chỉ số từ ngày đầu tiên  

### Các Mẫu Streaming Nâng Cao

🎨 **Cải Tiến Giao Diện Người Dùng:**
- Tiết lộ tiến dần - hiển thị tóm tắt trước, chi tiết theo yêu cầu
- Gián đoạn thông minh - cho phép người dùng đặt câu hỏi tiếp theo giữa chừng
- Bảo tồn ngữ cảnh - duy trì trạng thái cuộc trò chuyện qua các lỗi
- Streaming thích ứng - điều chỉnh tốc độ dựa trên tốc độ đọc của người dùng
- Phản hồi đa phương thức - kết hợp văn bản, hình ảnh và các yếu tố tương tác

🔧 **Tối Ưu Kỹ Thuật:**
- Phân đoạn phản hồi - chia các phản hồi dài thành các phần logic
- Xử lý song song - xử lý nhiều luồng đồng thời
- Đệm thông minh - tối ưu hóa việc sử dụng mạng
- Nén - giảm việc sử dụng băng thông cho các phản hồi lớn
- Tích hợp CDN - phân phối tải streaming trên toàn cầu

### Bước Tiếp Theo

Trong **bài tiếp theo**, chúng ta sẽ khám phá:

🏭 **Production Best Practices và Performance Optimization** - Triển khai và tối ưu hóa toàn diện  
🔧 **Advanced Architecture Patterns** - Kiến trúc có thể mở rộng cho enterprise systems  
📈 **Cost Optimization Strategies** - Tối ưu chi phí trong production environments  

### Thử Thách Dành Cho Bạn

1. **Xây dựng hệ thống giám sát toàn diện** với dashboard thời gian thực
2. **Triển khai intelligent error recovery** với bảo tồn ngữ cảnh
3. **Tạo performance analysis tools** để tối ưu hóa agent behavior
4. **Thiết kế troubleshooting automation** cho các vấn đề thường gặp
5. **Phát triển monitoring alerts** thích ứng với các mẫu sử dụng khác nhau

---

*Bài tiếp theo: [**"Production Best Practices và Performance Optimization"**](../openai-agents-sdk-part-10-production-best-practices-optimization/) - Chúng ta sẽ học cách triển khai và tối ưu hóa agent systems cho production environments với performance cao và reliability.*