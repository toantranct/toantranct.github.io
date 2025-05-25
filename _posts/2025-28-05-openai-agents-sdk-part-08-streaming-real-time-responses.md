---
title: "Streaming và Real-time Responses: Tối Ưu User Experience"
date: 2025-05-24 09:00:00 +0700
categories: [AI, OpenAI SDK]
tags: [openai, agents, streaming, real-time, ux]
author: toantranct
description: Hướng dẫn chi tiết về Streaming trong OpenAI Agents SDK. Từ raw response events đến building responsive UIs - tạo trải nghiệm người dùng mượt mà và real-time.
---

Trong thời đại mà người dùng quen với việc nhận thông tin tức thì - từ typing indicators trên WhatsApp đến live updates trên social media - việc chờ đợi agents "suy nghĩ" mà không có feedback nào có thể tạo cảm giác khó chịu. Đây chính là lúc **Streaming** phát huy tác dụng.

Streaming không chỉ là việc "hiển thị từng từ một" mà là nghệ thuật tạo ra **trải nghiệm tương tác tự nhiên** - cho người dùng cảm giác như đang trò chuyện với một con người thật sự. Hôm nay chúng ta sẽ khám phá cách biến agents từ những "black boxes" thành những "đối tác tương tác" responsive và engaging.

## Tại Sao Cần Streaming?

### Vấn Đề Với Non-Streaming Responses

Hãy tưởng tượng bạn hỏi một câu phức tạp và phải chờ 30 giây mới có phản hồi:

```python
# ❌ Non-streaming - User experience kém
async def poor_ux_example():
    agent = Agent(name="Slow Agent", instructions="You provide detailed analysis...")
    
    print("User: Phân tích thị trường crypto trong 6 tháng qua")
    print("Agent: ⏳ Đang suy nghĩ...")
    
    # User phải chờ 30 giây không biết gì đang diễn ra
    await asyncio.sleep(30)  # Simulate long processing
    
    print("Agent: [Suddenly dumps a huge response]")
```

**🔴 Problems:**
- User không biết agent có đang hoạt động hay không
- Không có indication về progress hay completion time
- Cảm giác "đứng hình", không responsive
- Không thể cancel hay interrupt nếu cần

### Lợi Ích Của Streaming

```python
# ✅ Streaming - User experience tuyệt vời
async def great_ux_example():
    print("User: Phân tích thị trường crypto trong 6 tháng qua")
    print("Agent: 💭 Đang phân tích...")
    
    # Real-time feedback
    print("Agent: 📊 Đang thu thập dữ liệu thị trường...")
    await asyncio.sleep(2)
    
    print("Agent: 🔍 Đang phân tích trends...")
    await asyncio.sleep(2)
    
    print("Agent: ✍️ Đang tạo báo cáo...")
    # Then stream the actual response word by word
    response = "Thị trường cryptocurrency trong 6 tháng qua..."
    
    for word in response.split():
        print(word, end=" ", flush=True)
        await asyncio.sleep(0.1)  # Simulate streaming
```

**✅ Benefits:**
- **Immediate feedback** - User biết ngay agent đang xử lý
- **Progress indication** - Shows what's happening at each step  
- **Natural flow** - Giống như conversation thật
- **Interruptible** - User có thể stop nếu cần
- **Perceived performance** - Feels faster even if total time is same

## Streaming Architecture trong OpenAI Agents SDK

### Stream Event Types

OpenAI Agents SDK cung cấp hai loại streaming events:

```python
from agents import Agent, Runner
from openai.types.responses import ResponseTextDeltaEvent

# 1. 🔧 RAW RESPONSE EVENTS - Low-level, token-by-token
# 2. 📦 RUN ITEM EVENTS - High-level, structured updates
```

**🔧 Raw Response Events:**
- Token-by-token output từ LLM
- Real-time text generation
- Giống như typing indicator
- Best cho text display

**📦 Run Item Events:**  
- High-level progress updates
- "Message generated", "Tool called", "Agent switched"
- Best cho UI state management
- Structured information về workflow

### Basic Streaming Implementation

```python
import asyncio
from openai.types.responses import ResponseTextDeltaEvent
from agents import Agent, Runner

async def basic_streaming_demo():
    """Demo basic streaming với token-by-token output"""
    
    agent = Agent(
        name="Storyteller",
        instructions="""
        Bạn là một kể chuyện tài ba.
        
        Phong cách:
        - Kể chuyện hấp dẫn, cuốn hút
        - Sử dụng ngôn ngữ sinh động
        - Tạo suspense và drama
        - Kết thúc có ý nghĩa
        
        Hãy kể những câu chuyện ngắn nhưng đầy cảm xúc.
        """
    )
    
    # Request a story
    user_request = "Kể cho tôi một câu chuyện ngắn về tình bạn"
    print(f"👤 **User:** {user_request}")
    print("🤖 **Agent:** ", end="", flush=True)
    
    # Stream the response
    result = Runner.run_streamed(agent, user_request)
    
    async for event in result.stream_events():
        # Handle raw text streaming
        if event.type == "raw_response_event" and isinstance(event.data, ResponseTextDeltaEvent):
            # Print each token as it arrives
            print(event.data.delta, end="", flush=True)
    
    print("\n")  # New line after story completion
    
    # Get final result
    final_result = await result
    print(f"\n📊 **Metadata:** Hoàn thành trong {len(final_result.raw_responses)} API calls")

if __name__ == "__main__":
    asyncio.run(basic_streaming_demo())
```

### Advanced Streaming với Run Item Events

```python
from agents import Agent, Runner, ItemHelpers, function_tool
import random
import asyncio

@function_tool
def roll_dice(sides: int = 6) -> int:
    """Roll a dice với số mặt tùy chọn"""
    return random.randint(1, sides)

@function_tool
def flip_coin() -> str:
    """Tung đồng xu"""
    return random.choice(["Ngửa", "Sấp"])

@function_tool
def magic_answer() -> str:
    """Câu trả lời thần bí"""
    answers = [
        "Có thể", "Chắc chắn", "Không chắc", 
        "Hỏi lại sau", "Không", "Tuyệt đối"
    ]
    return random.choice(answers)

async def advanced_streaming_demo():
    """Demo advanced streaming với structured events"""
    
    game_master = Agent(
        name="Game Master",
        instructions="""
        Bạn là một Game Master tài ba cho trò chơi role-playing.
        
        Khả năng của bạn:
        - Roll dice để xác định kết quả hành động
        - Tung coin để quyết định ngẫu nhiên
        - Đưa ra magic answers cho câu hỏi khó
        
        Phong cách:
        1. Luôn explain tại sao cần dùng tool
        2. Interpret kết quả tool theo context
        3. Tạo narrative hấp dẫn xung quanh results
        4. Encourage players tiếp tục adventure
        
        Make mỗi interaction thành một story moment!
        """,
        tools=[roll_dice, flip_coin, magic_answer]
    )
    
    scenarios = [
        "Tôi muốn leo lên vách đá cao 20 mét. Tôi có thành công không?",
        "Có 2 con đường trước mặt. Tôi nên chọn đường nào?",
        "Liệu tôi có tìm được kho báu trong hang động này không?"
    ]
    
    for scenario in scenarios:
        print(f"\n🎲 **Scenario:** {scenario}")
        print("🎭 **Game Master:** ", end="", flush=True)
        
        # Stream với detailed event handling
        result = Runner.run_streamed(game_master, scenario)
        
        current_text = ""
        tool_calls_made = 0
        
        async for event in result.stream_events():
            
            # Raw text streaming
            if event.type == "raw_response_event" and isinstance(event.data, ResponseTextDeltaEvent):
                print(event.data.delta, end="", flush=True)
                current_text += event.data.delta
            
            # Structured event handling
            elif event.type == "run_item_stream_event":
                
                if event.item.type == "tool_call_item":
                    tool_calls_made += 1
                    print(f"\n🔧 **[Tool {tool_calls_made}]** Đang sử dụng {event.item.tool_name}...")
                
                elif event.item.type == "tool_call_output_item":
                    tool_output = event.item.output
                    print(f"⚡ **[Result]** {tool_output}")
                    print("🎭 **GM continues:** ", end="", flush=True)
                
                elif event.item.type == "message_output_item":
                    # Message completed - có thể add special formatting
                    pass
        
        print("\n" + "="*60)
    
    print(f"\n🎉 **Adventure complete!** Total tools used across all scenarios: {tool_calls_made}")

if __name__ == "__main__":
    asyncio.run(advanced_streaming_demo())
```

## Building Responsive User Interfaces

### Console-based Chat Interface

```python
import asyncio
import sys
from datetime import datetime
from typing import List, Dict, Any
from agents import Agent, Runner
from openai.types.responses import ResponseTextDeltaEvent

class StreamingChatInterface:
    """Interactive chat interface với streaming support"""
    
    def __init__(self):
        self.conversation_history: List[Dict[str, str]] = []
        
        self.chat_agent = Agent(
            name="Friendly Assistant",
            instructions="""
            Bạn là một trợ lý AI thân thiện và hữu ích.
            
            Tính cách:
            - Nhiệt tình và tích cực
            - Thông minh nhưng không khô khan
            - Có sense of humor nhẹ nhàng
            - Quan tâm đến user's needs
            
            Communication style:
            - Sử dụng emojis một cách tự nhiên
            - Hỏi follow-up questions khi cần
            - Provide examples và context
            - Keep responses concise nhưng informative
            
            Nếu không chắc chắn về thông tin, hãy thành thật admit.
            """
        )
    
    def display_header(self):
        """Display chat interface header"""
        print("=" * 60)
        print("🤖 **STREAMING CHAT INTERFACE** 🤖")
        print("=" * 60)
        print("💡 Tips:")
        print("   • Type '/help' for commands")
        print("   • Type '/history' to see conversation")
        print("   • Type '/clear' to clear history")
        print("   • Type '/quit' to exit")
        print("=" * 60)
    
    def get_user_input(self) -> str:
        """Get user input với prompt"""
        try:
            return input("\n👤 **You:** ").strip()
        except KeyboardInterrupt:
            print("\n\n👋 Chat interrupted. Goodbye!")
            sys.exit(0)
    
    async def stream_agent_response(self, user_message: str) -> str:
        """Stream agent response với visual indicators"""
        
        print("🤖 **Assistant:** ", end="", flush=True)
        
        # Add user message to history
        self.conversation_history.append({
            "role": "user", 
            "content": user_message,
            "timestamp": datetime.now().isoformat()
        })
        
        # Build context từ conversation history
        if len(self.conversation_history) > 1:
            # Include recent conversation context
            context_messages = []
            for msg in self.conversation_history[-6:]:  # Last 3 exchanges
                context_messages.append(f"{msg['role']}: {msg['content']}")
            
            contextual_input = f"""
Previous conversation:
{chr(10).join(context_messages)}

Current user message: {user_message}

Please respond naturally, considering the conversation context.
"""
        else:
            contextual_input = user_message
        
        # Stream the response
        result = Runner.run_streamed(self.chat_agent, contextual_input)
        
        response_text = ""
        typing_indicator_shown = False
        
        async for event in result.stream_events():
            
            # Handle raw text streaming
            if event.type == "raw_response_event" and isinstance(event.data, ResponseTextDeltaEvent):
                if not typing_indicator_shown:
                    typing_indicator_shown = True
                
                delta = event.data.delta
                print(delta, end="", flush=True)
                response_text += delta
            
            # Handle structured events for additional context
            elif event.type == "run_item_stream_event":
                if event.item.type == "tool_call_item":
                    print(f"\n   🔧 [Using tool: {event.item.tool_name}]", end="", flush=True)
                elif event.item.type == "tool_call_output_item":
                    print(f"\n   ✅ [Tool completed]", end="", flush=True)
                    print("\n🤖 **Assistant:** ", end="", flush=True)
        
        print()  # New line after response
        
        # Add assistant response to history
        self.conversation_history.append({
            "role": "assistant",
            "content": response_text,
            "timestamp": datetime.now().isoformat()
        })
        
        return response_text
    
    def handle_command(self, command: str) -> bool:
        """Handle special commands. Returns True if should continue chat."""
        
        if command == "/help":
            print("""
📋 **Available Commands:**
   • /help     - Show this help message
   • /history  - Show conversation history
   • /clear    - Clear conversation history
   • /stats    - Show chat statistics
   • /quit     - Exit the chat
            """)
        
        elif command == "/history":
            if not self.conversation_history:
                print("📝 No conversation history yet.")
            else:
                print("\n📖 **Conversation History:**")
                for i, msg in enumerate(self.conversation_history, 1):
                    role_icon = "👤" if msg["role"] == "user" else "🤖"
                    timestamp = datetime.fromisoformat(msg["timestamp"]).strftime("%H:%M")
                    print(f"{i:2d}. {role_icon} [{timestamp}] {msg['content'][:80]}{'...' if len(msg['content']) > 80 else ''}")
        
        elif command == "/clear":
            self.conversation_history.clear()
            print("🗑️ Conversation history cleared.")
        
        elif command == "/stats":
            total_messages = len(self.conversation_history)
            user_messages = len([m for m in self.conversation_history if m["role"] == "user"])
            assistant_messages = len([m for m in self.conversation_history if m["role"] == "assistant"])
            
            print(f"""
📊 **Chat Statistics:**
   • Total messages: {total_messages}
   • Your messages: {user_messages}
   • Assistant messages: {assistant_messages}
   • Average response length: {sum(len(m['content']) for m in self.conversation_history if m['role'] == 'assistant') // max(assistant_messages, 1)} characters
            """)
        
        elif command == "/quit":
            print("👋 Thanks for chatting! Goodbye!")
            return False
        
        else:
            print(f"❓ Unknown command: {command}. Type '/help' for available commands.")
        
        return True
    
    async def run(self):
        """Main chat loop"""
        self.display_header()
        
        try:
            while True:
                user_input = self.get_user_input()
                
                if not user_input:
                    continue
                
                # Handle commands
                if user_input.startswith("/"):
                    should_continue = self.handle_command(user_input)
                    if not should_continue:
                        break
                    continue
                
                # Process regular message
                try:
                    await self.stream_agent_response(user_input)
                    
                except KeyboardInterrupt:
                    print("\n\n⏸️ Response interrupted by user.")
                    continue
                    
                except Exception as e:
                    print(f"\n❌ **Error:** {str(e)}")
                    print("🔄 Please try again or type '/help' for assistance.")
        
        except Exception as e:
            print(f"\n💥 **Unexpected error:** {str(e)}")
        
        finally:
            print("\n👋 Chat session ended.")

# Demo the streaming chat interface
async def demo_streaming_chat():
    """Demo interactive streaming chat"""
    chat = StreamingChatInterface()
    await chat.run()

if __name__ == "__main__":
    asyncio.run(demo_streaming_chat())
```

### Web-based Streaming Interface

```python
from typing import AsyncGenerator, Dict, Any
import json
import asyncio
from datetime import datetime

class WebStreamingService:
    """Service for web-based streaming interfaces"""
    
    def __init__(self):
        self.active_sessions: Dict[str, Dict[str, Any]] = {}
        
        self.web_agent = Agent(
            name="Web Assistant",
            instructions="""
            Bạn là một AI assistant được tối ưu cho web interface.
            
            Web-specific behaviors:
            - Responses should be well-structured for HTML rendering
            - Use markdown formatting when appropriate
            - Be concise but informative
            - Include relevant emojis for visual appeal
            - Suggest follow-up actions when helpful
            
            Response format:
            - Short paragraphs for better readability
            - Use bullet points for lists
            - Bold important information
            - Include examples when explaining concepts
            """
        )
    
    async def create_session(self, session_id: str, user_context: Dict[str, Any] = None) -> Dict[str, Any]:
        """Create new streaming session"""
        
        self.active_sessions[session_id] = {
            "created_at": datetime.now().isoformat(),
            "user_context": user_context or {},
            "message_count": 0,
            "last_activity": datetime.now().isoformat()
        }
        
        return {
            "session_id": session_id,
            "status": "created",
            "timestamp": datetime.now().isoformat()
        }
    
    async def stream_response(
        self, 
        session_id: str, 
        user_message: str
    ) -> AsyncGenerator[Dict[str, Any], None]:
        """Stream response for web interface"""
        
        if session_id not in self.active_sessions:
            yield {
                "type": "error",
                "data": {
                    "message": "Session not found",
                    "code": "SESSION_NOT_FOUND"
                }
            }
            return
        
        # Update session activity
        session = self.active_sessions[session_id]
        session["last_activity"] = datetime.now().isoformat()
        session["message_count"] += 1
        
        # Yield session update
        yield {
            "type": "session_update",
            "data": {
                "session_id": session_id,
                "message_count": session["message_count"],
                "timestamp": datetime.now().isoformat()
            }
        }
        
        # Yield typing indicator
        yield {
            "type": "typing_start",
            "data": {
                "message": "Assistant is thinking...",
                "timestamp": datetime.now().isoformat()
            }
        }
        
        # Stream the actual response
        try:
            result = Runner.run_streamed(self.web_agent, user_message)
            
            response_buffer = ""
            word_count = 0
            
            async for event in result.stream_events():
                
                # Handle text streaming
                if event.type == "raw_response_event" and isinstance(event.data, ResponseTextDeltaEvent):
                    delta = event.data.delta
                    response_buffer += delta
                    
                    # Send periodic updates (every few words)
                    if " " in delta:
                        word_count += delta.count(" ")
                        
                        if word_count % 5 == 0:  # Every 5 words
                            yield {
                                "type": "text_delta",
                                "data": {
                                    "delta": delta,
                                    "full_text": response_buffer,
                                    "word_count": word_count,
                                    "timestamp": datetime.now().isoformat()
                                }
                            }
                    else:
                        yield {
                            "type": "text_delta", 
                            "data": {
                                "delta": delta,
                                "timestamp": datetime.now().isoformat()
                            }
                        }
                
                # Handle structured events
                elif event.type == "run_item_stream_event":
                    
                    if event.item.type == "tool_call_item":
                        yield {
                            "type": "tool_start",
                            "data": {
                                "tool_name": event.item.tool_name,
                                "message": f"Using {event.item.tool_name}...",
                                "timestamp": datetime.now().isoformat()
                            }
                        }
                    
                    elif event.item.type == "tool_call_output_item":
                        yield {
                            "type": "tool_complete",
                            "data": {
                                "tool_output": str(event.item.output)[:200] + "..." if len(str(event.item.output)) > 200 else str(event.item.output),
                                "message": "Tool execution completed",
                                "timestamp": datetime.now().isoformat()
                            }
                        }
            
            # Final response completion
            yield {
                "type": "response_complete",
                "data": {
                    "final_response": response_buffer,
                    "word_count": len(response_buffer.split()),
                    "character_count": len(response_buffer),
                    "session_id": session_id,
                    "timestamp": datetime.now().isoformat()
                }
            }
            
        except Exception as e:
            yield {
                "type": "error",
                "data": {
                    "message": f"Stream error: {str(e)}",
                    "code": "STREAM_ERROR",
                    "timestamp": datetime.now().isoformat()
                }
            }
    
    def get_session_stats(self, session_id: str) -> Dict[str, Any]:
        """Get session statistics"""
        
        if session_id not in self.active_sessions:
            return {"error": "Session not found"}
        
        session = self.active_sessions[session_id]
        
        return {
            "session_id": session_id,
            "created_at": session["created_at"],
            "last_activity": session["last_activity"],
            "message_count": session["message_count"],
            "duration_minutes": (
                datetime.now() - datetime.fromisoformat(session["created_at"])
            ).total_seconds() / 60,
            "status": "active"
        }

# Demo web streaming service
async def demo_web_streaming():
    """Demo web streaming service"""
    
    print("🌐 **Web Streaming Service Demo**\n")
    
    streaming_service = WebStreamingService()
    
    # Create session
    session_id = "web_session_123"
    session_info = await streaming_service.create_session(
        session_id, 
        {"user_agent": "Demo Browser", "ip": "127.0.0.1"}
    )
    
    print(f"📱 Session created: {session_info}")
    
    # Test messages
    test_messages = [
        "Giải thích cho tôi về machine learning một cách đơn giản",
        "Tôi có nên học Python hay JavaScript trước?",
        "Làm thế nào để trở thành một developer giỏi?"
    ]
    
    for i, message in enumerate(test_messages, 1):
        print(f"\n💬 **Message {i}:** {message}")
        print("📡 **Streaming events:**")
        
        event_count = 0
        async for event in streaming_service.stream_response(session_id, message):
            event_count += 1
            
            if event["type"] == "typing_start":
                print(f"   ⏳ {event['data']['message']}")
            
            elif event["type"] == "text_delta":
                if "delta" in event["data"]:
                    print(event["data"]["delta"], end="", flush=True)
            
            elif event["type"] == "tool_start":
                print(f"\n   🔧 {event['data']['message']}")
            
            elif event["type"] == "tool_complete":
                print(f"\n   ✅ {event['data']['message']}")
            
            elif event["type"] == "response_complete":
                data = event["data"]
                print(f"\n\n📊 Response complete:")
                print(f"   • Words: {data['word_count']}")
                print(f"   • Characters: {data['character_count']}")
        
        print(f"\n   📈 Total events: {event_count}")
        print("=" * 60)
    
    # Show session stats
    stats = streaming_service.get_session_stats(session_id)
    print(f"\n📈 **Session Statistics:**")
    print(f"   • Duration: {stats['duration_minutes']:.1f} minutes")
    print(f"   • Messages: {stats['message_count']}")
    print(f"   • Status: {stats['status']}")

if __name__ == "__main__":
    from openai.types.responses import ResponseTextDeltaEvent
    asyncio.run(demo_web_streaming())
```

## Error Handling và Recovery

### Robust Streaming với Error Recovery

```python
import asyncio
from typing import Optional, Dict, Any, List
from datetime import datetime, timedelta
from agents import Agent, Runner
from agents.exceptions import AgentsException

class ResilientStreamingSystem:
    """Streaming system với comprehensive error handling"""
    
    def __init__(self):
        self.retry_config = {
            "max_retries": 3,
            "base_delay": 1.0,
            "max_delay": 10.0,
            "backoff_multiplier": 2.0
        }
        
        self.error_statistics = {
            "total_errors": 0,
            "network_errors": 0,
            "timeout_errors": 0,
            "agent_errors": 0,
            "recovery_successes": 0
        }
        
        self.resilient_agent = Agent(
            name="Resilient Assistant",
            instructions="""
            Bạn là một AI assistant được thiết kế để robust và reliable.
            
            Characteristics:
            - Always try to provide helpful responses
            - Gracefully handle interruptions
            - Acknowledge when recovering from errors
            - Be transparent about limitations
            - Maintain conversation context even after errors
            
            If you detect that a previous response was interrupted:
            - Acknowledge the interruption
            - Offer to continue or restart
            - Ask if the user needs clarification
            """
        )
    
    async def stream_with_recovery(
        self, 
        user_message: str,
        context: Optional[str] = None,
        timeout: float = 30.0
    ) -> Dict[str, Any]:
        """Stream response với automatic error recovery"""
        
        attempt = 0
        last_error = None
        partial_response = ""
        
        while attempt < self.retry_config["max_retries"]:
            try:
                print(f"🔄 Attempt {attempt + 1}/{self.retry_config['max_retries']}")
                
                # Build full context including any partial response
                full_context = self._build_recovery_context(user_message, context, partial_response, attempt)
                
                # Stream với timeout
                result = await asyncio.wait_for(
                    self._stream_response(full_context, partial_response),
                    timeout=timeout
                )
                
                # Success!
                if attempt > 0:
                    self.error_statistics["recovery_successes"] += 1
                    print(f"✅ Recovered successfully after {attempt} attempts")
                
                return {
                    "success": True,
                    "response": result["response"],
                    "attempts": attempt + 1,
                    "recovered": attempt > 0,
                    "partial_content": partial_response
                }
                
            except asyncio.TimeoutError as e:
                self.error_statistics["timeout_errors"] += 1
                last_error = f"Timeout after {timeout}s"
                print(f"⏰ Timeout error: {last_error}")
                
            except AgentsException as e:
                self.error_statistics["agent_errors"] += 1
                last_error = f"Agent error: {str(e)}"
                print(f"🤖 Agent error: {last_error}")
                
            except Exception as e:
                self.error_statistics["network_errors"] += 1
                last_error = f"Network error: {str(e)}"
                print(f"🌐 Network error: {last_error}")
            
            attempt += 1
            self.error_statistics["total_errors"] += 1
            
            if attempt < self.retry_config["max_retries"]:
                # Calculate delay với exponential backoff
                delay = min(
                    self.retry_config["base_delay"] * (self.retry_config["backoff_multiplier"] ** attempt),
                    self.retry_config["max_delay"]
                )
                
                print(f"⏳ Retrying in {delay:.1f} seconds...")
                await asyncio.sleep(delay)
        
        # All retries failed
        print(f"❌ All {self.retry_config['max_retries']} attempts failed")
        
        return {
            "success": False,
            "error": last_error,
            "attempts": attempt,
            "partial_content": partial_response,
            "recovery_failed": True
        }
    
    def _build_recovery_context(
        self, 
        original_message: str, 
        context: Optional[str], 
        partial_response: str, 
        attempt: int
    ) -> str:
        """Build context cho recovery attempts"""
        
        recovery_context = f"User message: {original_message}\n"
        
        if context:
            recovery_context += f"Additional context: {context}\n"
        
        if attempt > 0:
            recovery_context += f"\nNote: This is attempt #{attempt + 1} due to previous errors."
            
            if partial_response:
                recovery_context += f"\nPartial response was generated: {partial_response[:200]}..."
                recovery_context += "\nPlease continue from where it left off or provide a complete response."
            else:
                recovery_context += "\nPrevious attempt failed before generating response. Please try again."
        
        return recovery_context
    
    async def _stream_response(self, message: str, existing_partial: str = "") -> Dict[str, Any]:
        """Internal streaming method"""
        
        result = Runner.run_streamed(self.resilient_agent, message)
        
        response_text = existing_partial
        events_processed = 0
        
        async for event in result.stream_events():
            events_processed += 1
            
            if event.type == "raw_response_event":
                from openai.types.responses import ResponseTextDeltaEvent
                if isinstance(event.data, ResponseTextDeltaEvent):
                    delta = event.data.delta
                    response_text += delta
                    print(delta, end="", flush=True)
            
            # Simulate random errors for testing
            if events_processed > 10 and random.random() < 0.1:  # 10% chance
                raise Exception("Simulated network interruption")
        
        print()  # New line after response
        
        return {
            "response": response_text,
            "events_processed": events_processed
        }
    
    def get_error_statistics(self) -> Dict[str, Any]:
        """Get comprehensive error statistics"""
        
        total_errors = self.error_statistics["total_errors"]
        
        if total_errors == 0:
            error_rate = 0.0
            recovery_rate = 0.0
        else:
            error_rate = total_errors / (total_errors + self.error_statistics["recovery_successes"])
            recovery_rate = self.error_statistics["recovery_successes"] / total_errors
        
        return {
            **self.error_statistics,
            "error_rate": error_rate,
            "recovery_rate": recovery_rate,
            "last_updated": datetime.now().isoformat()
        }

# Demo resilient streaming
async def demo_resilient_streaming():
    """Demo resilient streaming system"""
    
    print("🛡️ **Resilient Streaming System Demo**\n")
    
    streaming_system = ResilientStreamingSystem()
    
    test_scenarios = [
        {
            "message": "Giải thích về quantum computing một cách đơn giản",
            "context": "User is a beginner in technology"
        },
        {
            "message": "Tạo một plan học lập trình trong 6 tháng",
            "context": "User có background về toán học"
        },
        {
            "message": "So sánh các framework JavaScript phổ biến",
            "context": "User đang tìm hiểu về web development"
        }
    ]
    
    for i, scenario in enumerate(test_scenarios, 1):
        print(f"💬 **Scenario {i}:** {scenario['message']}")
        print(f"📝 **Context:** {scenario['context']}")
        print("🤖 **Response:**")
        
        result = await streaming_system.stream_with_recovery(
            scenario["message"],
            scenario["context"],
            timeout=20.0
        )
        
        if result["success"]:
            print(f"\n✅ Success (attempts: {result['attempts']})")
            if result["recovered"]:
                print("🔄 Response was recovered from error")
        else:
            print(f"\n❌ Failed: {result['error']}")
            if result.get("partial_content"):
                print(f"📝 Partial content: {result['partial_content'][:100]}...")
        
        print("=" * 70 + "\n")
    
    # Show error statistics
    stats = streaming_system.get_error_statistics()
    print("📊 **Error Statistics:**")
    print(f"   • Total errors: {stats['total_errors']}")
    print(f"   • Network errors: {stats['network_errors']}")
    print(f"   • Timeout errors: {stats['timeout_errors']}")  
    print(f"   • Agent errors: {stats['agent_errors']}")
    print(f"   • Recovery successes: {stats['recovery_successes']}")
    print(f"   • Recovery rate: {stats['recovery_rate']:.1%}")

if __name__ == "__main__":
    import random
    asyncio.run(demo_resilient_streaming())
```

## Performance Optimization

### Streaming Performance Monitoring

```python
import time
import statistics
from typing import List, Dict, Any
from dataclasses import dataclass, field
from datetime import datetime, timedelta

@dataclass
class StreamingMetrics:
    """Metrics for streaming performance"""
    session_id: str
    start_time: float = field(default_factory=time.time)
    first_token_time: Optional[float] = None
    completion_time: Optional[float] = None
    total_tokens: int = 0
    total_events: int = 0
    error_count: int = 0
    interruption_count: int = 0
    
    def calculate_latencies(self) -> Dict[str, float]:
        """Calculate various latency metrics"""
        current_time = time.time()
        
        return {
            "time_to_first_token": self.first_token_time - self.start_time if self.first_token_time else None,
            "total_response_time": self.completion_time - self.start_time if self.completion_time else current_time - self.start_time,
            "tokens_per_second": self.total_tokens / (current_time - self.start_time) if current_time > self.start_time else 0,
            "events_per_second": self.total_events / (current_time - self.start_time) if current_time > self.start_time else 0
        }

class StreamingPerformanceMonitor:
    """Monitor và optimize streaming performance"""
    
    def __init__(self):
        self.active_sessions: Dict[str, StreamingMetrics] = {}
        self.completed_sessions: List[StreamingMetrics] = []
        self.performance_thresholds = {
            "max_time_to_first_token": 2.0,  # seconds
            "min_tokens_per_second": 10,     # tokens/sec
            "max_error_rate": 0.05           # 5%
        }
    
    def start_session(self, session_id: str) -> StreamingMetrics:
        """Start monitoring a new streaming session"""
        
        metrics = StreamingMetrics(session_id=session_id)
        self.active_sessions[session_id] = metrics
        
        return metrics
    
    def record_first_token(self, session_id: str):
        """Record when first token was received"""
        if session_id in self.active_sessions:
            self.active_sessions[session_id].first_token_time = time.time()
    
    def record_token(self, session_id: str, token: str):
        """Record a new token"""
        if session_id in self.active_sessions:
            self.active_sessions[session_id].total_tokens += 1
    
    def record_event(self, session_id: str, event_type: str):
        """Record a streaming event"""
        if session_id in self.active_sessions:
            metrics = self.active_sessions[session_id]
            metrics.total_events += 1
            
            if event_type == "error":
                metrics.error_count += 1
            elif event_type == "interruption":
                metrics.interruption_count += 1
    
    def complete_session(self, session_id: str) -> Dict[str, Any]:
        """Complete a streaming session và calculate final metrics"""
        
        if session_id not in self.active_sessions:
            return {"error": "Session not found"}
        
        metrics = self.active_sessions[session_id]
        metrics.completion_time = time.time()
        
        # Calculate final metrics
        latencies = metrics.calculate_latencies()
        
        # Move to completed sessions
        self.completed_sessions.append(metrics)
        del self.active_sessions[session_id]
        
        # Performance assessment
        performance_issues = self._assess_performance(metrics, latencies)
        
        return {
            "session_id": session_id,
            "metrics": metrics,
            "latencies": latencies,
            "performance_issues": performance_issues,
            "status": "completed"
        }
    
    def _assess_performance(self, metrics: StreamingMetrics, latencies: Dict[str, float]) -> List[str]:
        """Assess performance và identify issues"""
        
        issues = []
        
        # Check time to first token
        if latencies["time_to_first_token"] and latencies["time_to_first_token"] > self.performance_thresholds["max_time_to_first_token"]:
            issues.append(f"Slow first token: {latencies['time_to_first_token']:.2f}s > {self.performance_thresholds['max_time_to_first_token']}s")
        
        # Check tokens per second
        if latencies["tokens_per_second"] < self.performance_thresholds["min_tokens_per_second"]:
            issues.append(f"Low throughput: {latencies['tokens_per_second']:.1f} tokens/s < {self.performance_thresholds['min_tokens_per_second']}")
        
        # Check error rate
        if metrics.total_events > 0:
            error_rate = metrics.error_count / metrics.total_events
            if error_rate > self.performance_thresholds["max_error_rate"]:
                issues.append(f"High error rate: {error_rate:.1%} > {self.performance_thresholds['max_error_rate']:.1%}")
        
        return issues
    
    def get_global_stats(self) -> Dict[str, Any]:
        """Get global performance statistics"""
        
        if not self.completed_sessions:
            return {"message": "No completed sessions yet"}
        
        # Aggregate metrics
        all_sessions = self.completed_sessions
        
        ttft_times = [s.first_token_time - s.start_time for s in all_sessions if s.first_token_time]
        total_times = [s.completion_time - s.start_time for s in all_sessions if s.completion_time]
        token_rates = [s.total_tokens / (s.completion_time - s.start_time) for s in all_sessions if s.completion_time and s.completion_time > s.start_time]
        
        return {
            "total_sessions": len(all_sessions),
            "average_time_to_first_token": statistics.mean(ttft_times) if ttft_times else 0,
            "median_time_to_first_token": statistics.median(ttft_times) if ttft_times else 0,
            "average_total_time": statistics.mean(total_times) if total_times else 0,
            "average_tokens_per_second": statistics.mean(token_rates) if token_rates else 0,
            "total_tokens_processed": sum(s.total_tokens for s in all_sessions),
            "total_errors": sum(s.error_count for s in all_sessions),
            "sessions_with_issues": len([s for s in all_sessions if s.error_count > 0 or s.interruption_count > 0]),
            "last_updated": datetime.now().isoformat()
        }

# Optimized streaming agent với performance monitoring
class OptimizedStreamingAgent:
    """High-performance streaming agent với monitoring"""
    
    def __init__(self):
        self.performance_monitor = StreamingPerformanceMonitor()
        
        self.optimized_agent = Agent(
            name="High Performance Assistant",
            instructions="""
            Bạn là một AI assistant được tối ưu hóa cho performance.
            
            Performance characteristics:
            - Respond quickly and efficiently
            - Use concise but informative language
            - Structure responses for optimal streaming
            - Minimize unnecessary processing
            
            Response strategy:
            - Start with direct answer
            - Add details progressively
            - Use short sentences for better streaming flow
            - Include examples when helpful but keep them brief
            """
        )
    
    async def optimized_stream_response(self, session_id: str, user_message: str) -> Dict[str, Any]:
        """Stream response với performance optimization"""
        
        # Start monitoring
        metrics = self.performance_monitor.start_session(session_id)
        
        try:
            # Stream response
            result = Runner.run_streamed(self.optimized_agent, user_message)
            
            response_text = ""
            first_token_recorded = False
            
            async for event in result.stream_events():
                
                # Record events
                self.performance_monitor.record_event(session_id, event.type)
                
                if event.type == "raw_response_event":
                    from openai.types.responses import ResponseTextDeltaEvent
                    if isinstance(event.data, ResponseTextDeltaEvent):
                        
                        # Record first token
                        if not first_token_recorded:
                            self.performance_monitor.record_first_token(session_id)
                            first_token_recorded = True
                        
                        # Record token
                        delta = event.data.delta
                        self.performance_monitor.record_token(session_id, delta)
                        response_text += delta
                        
                        # Stream to output
                        print(delta, end="", flush=True)
                
                elif event.type == "run_item_stream_event":
                    # Handle tool calls efficiently  
                    if event.item.type == "tool_call_item":
                        print(f"\n🔧 [{event.item.tool_name}]", end="", flush=True)
            
            print()  # New line after completion
            
            # Complete session monitoring
            session_result = self.performance_monitor.complete_session(session_id)
            
            return {
                "success": True,
                "response": response_text,
                "performance": session_result
            }
            
        except Exception as e:
            self.performance_monitor.record_event(session_id, "error")
            session_result = self.performance_monitor.complete_session(session_id)
            
            return {
                "success": False,
                "error": str(e),
                "performance": session_result
            }

# Demo optimized streaming
async def demo_optimized_streaming():
    """Demo optimized streaming với performance monitoring"""
    
    print("⚡ **Optimized Streaming Performance Demo**\n")
    
    streaming_agent = OptimizedStreamingAgent()
    
    # Test different types of queries
    test_queries = [
        "Giải thích nhanh về React hooks",
        "So sánh Python và JavaScript cho beginners",
        "Tạo một TODO list đơn giản bằng HTML/CSS/JS",
        "Explain machine learning trong 2 phút",
        "Best practices cho database design"
    ]
    
    for i, query in enumerate(test_queries, 1):
        session_id = f"perf_session_{i}"
        
        print(f"🚀 **Query {i}:** {query}")
        print("📡 **Streaming response:**")
        
        start_time = time.time()
        result = await streaming_agent.optimized_stream_response(session_id, query)
        end_time = time.time()
        
        if result["success"]:
            perf = result["performance"]
            latencies = perf["latencies"]
            
            print(f"\n📊 **Performance Metrics:**")
            print(f"   • Time to First Token: {latencies['time_to_first_token']:.3f}s")
            print(f"   • Total Response Time: {latencies['total_response_time']:.3f}s")
            print(f"   • Tokens per Second: {latencies['tokens_per_second']:.1f}")
            print(f"   • Total Tokens: {perf['metrics'].total_tokens}")
            
            if perf["performance_issues"]:
                print(f"   ⚠️ Issues: {', '.join(perf['performance_issues'])}")
            else:
                print(f"   ✅ Performance: Good")
        
        else:
            print(f"\n❌ Error: {result['error']}")
        
        print("=" * 70 + "\n")
    
    # Show global performance stats
    global_stats = streaming_agent.performance_monitor.get_global_stats()
    
    print("📈 **Global Performance Statistics:**")
    print(f"   • Total Sessions: {global_stats['total_sessions']}")
    print(f"   • Avg Time to First Token: {global_stats['average_time_to_first_token']:.3f}s")
    print(f"   • Avg Tokens/Second: {global_stats['average_tokens_per_second']:.1f}")
    print(f"   • Total Tokens Processed: {global_stats['total_tokens_processed']}")
    print(f"   • Sessions with Issues: {global_stats['sessions_with_issues']}")

if __name__ == "__main__":
    asyncio.run(demo_optimized_streaming())
```

## Tổng Kết và Best Practices

### Những điểm chính cần nhớ

✅ **Ưu tiên trải nghiệm người dùng** - Streaming cải thiện đáng kể cảm giác phản hồi nhanh và tự nhiên
✅ **Hai loại sự kiện** - Raw Response Events cho từng từ hiển thị, Run Item Events cho thông tin cấu trúc
✅ **Xử lý lỗi thông minh** - Chủ động xử lý gián đoạn mạng và lỗi hệ thống
✅ **Giám sát hiệu suất** - Đo lường các chỉ số streaming để tối ưu trải nghiệm
✅ **Giao diện phản hồi nhanh** - Thích hợp cho cả ứng dụng console và web

### Những best practice khi triển khai Streaming

🎯 **Về trải nghiệm người dùng:**

* Luôn hiển thị phản hồi ngay lập tức khi bắt đầu xử lý
* Sử dụng các chỉ báo trạng thái và tiến độ rõ ràng
* Cho phép người dùng ngắt các phản hồi quá dài
* Luôn cung cấp ngữ cảnh rõ ràng về những gì đang diễn ra
* Xử lý lỗi một cách mượt mà và đề xuất các phương án khắc phục

⚡ **Về hiệu suất:**

* Theo dõi sát sao độ trễ của token đầu tiên (time-to-first-token)
* Đảm bảo tốc độ token ổn định và nhanh chóng
* Thực hiện buffering một cách thông minh và hiệu quả
* Lưu cache những phản hồi thường xuyên lặp lại
* Sử dụng kỹ thuật connection pooling để duy trì hiệu suất

🔧 **Về mặt kỹ thuật:**

* Xử lý đồng thời các sự kiện streaming ở cấp độ raw và cấu trúc
* Áp dụng các cơ chế khôi phục lỗi toàn diện và mạnh mẽ
* Triển khai giám sát hiệu suất ngay từ ban đầu
* Thiết kế phù hợp cho cả giao diện console và web
* Kiểm thử kỹ lưỡng trong nhiều điều kiện mạng khác nhau

### Hướng dẫn triển khai cho hệ thống production

📊 **Thiết lập giám sát:**

* Theo dõi độ trễ token đầu tiên cho mọi yêu cầu
* Kiểm soát tốc độ xử lý token và tỷ lệ lỗi
* Thiết lập cảnh báo khi hiệu suất giảm sút
* Ghi log các trường hợp gián đoạn streaming và kết quả khôi phục
* Đánh giá tương tác của người dùng khi dùng streaming so với cách truyền thống

🛡️ **Xử lý lỗi:**

* Thực hiện retry với chiến lược backoff tăng dần
* Bảo toàn các phản hồi một phần khi gặp lỗi
* Cung cấp thông báo lỗi thân thiện và dễ hiểu cho người dùng
* Cho phép người dùng thử lại một cách chủ động
* Theo dõi các mô hình lỗi để phát hiện vấn đề hệ thống

🔄 **Các chiến lược tối ưu:**

* Duy trì sẵn kết nối để giảm độ trễ ban đầu
* Thực hiện caching thông minh cho các truy vấn phổ biến
* Áp dụng kết nối đa luồng khi có thể
* Tối ưu hóa các thiết lập mô hình AI để phù hợp với streaming
* Cân nhắc triển khai ở các edge server để giảm độ trễ mạng

### Những lỗi thường gặp và cách khắc phục

❌ **Bỏ qua độ trễ token đầu tiên** → Luôn giám sát và tối ưu độ trễ ban đầu
❌ **Không xử lý lỗi** → Thực hiện retry mạnh mẽ và thông minh
❌ **Không có chỉ báo tiến độ rõ ràng** → Luôn hiển thị trạng thái xử lý rõ ràng
❌ **Hiển thị quá nhiều thông tin** → Cân bằng giữa thông tin cung cấp và trải nghiệm người dùng
❌ **Không giám sát hiệu suất** → Theo dõi các chỉ số hiệu suất ngay từ đầu

### Các mô hình streaming nâng cao

🎨 **Cải tiến về giao diện và trải nghiệm:**

* Progressive disclosure - hiển thị tóm tắt trước, chi tiết sau
* Smart interruption - cho phép người dùng đặt câu hỏi thêm trong quá trình streaming
* Bảo toàn ngữ cảnh - duy trì trạng thái trò chuyện dù xảy ra lỗi
* Adaptive streaming - điều chỉnh tốc độ hiển thị theo khả năng đọc của người dùng
* Phản hồi đa phương tiện - kết hợp văn bản, hình ảnh và các yếu tố tương tác

🔧 **Các kỹ thuật tối ưu chuyên sâu:**

* Response chunking - chia nhỏ các phản hồi dài thành các phần logic
* Xử lý song song - xử lý đồng thời nhiều luồng streaming
* Buffering thông minh - tối ưu hóa việc sử dụng mạng
* Nén dữ liệu - giảm băng thông cho các phản hồi lớn
* Tích hợp CDN - phân phối tải streaming trên các khu vực địa lý

### Bước tiếp theo

Trong **bài tiếp theo**, chúng ta sẽ khám phá:

🕵️ **Tracing và Debugging** - Giám sát hiệu năng và hành vi của Agent
📊 **Các mô hình Observability** - Giám sát toàn diện trong hệ thống production
🔍 **Phân tích hiệu suất** - Đi sâu vào các kỹ thuật tối ưu hóa

### Thử thách dành cho bạn

1. **Xây dựng giao diện chat phản hồi nhanh** với hỗ trợ streaming trên web hoặc mobile
2. **Triển khai xử lý lỗi thông minh** bảo toàn ngữ cảnh hội thoại
3. **Tạo dashboard theo dõi hiệu suất** của streaming
4. **Tối ưu hóa hiệu suất streaming** cho từng trường hợp sử dụng cụ thể
5. **Thiết kế adaptive streaming** điều chỉnh theo hành vi người dùng

---

*Bài tiếp theo: [**"Tracing và Debugging: Giám Sát Hiệu Năng Agent"**](../openai-agents-sdk-part-09-tracing-debugging-monitoring/) - Chúng ta sẽ học cách giám sát, debug và tối ưu hiệu năng của Agent với các công cụ observability toàn diện.*
