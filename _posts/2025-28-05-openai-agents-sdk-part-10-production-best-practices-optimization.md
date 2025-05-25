---
title: "Production Best Practices và Performance Optimization"
date: 2025-05-24 09:00:00 +0700
categories: [AI, OpenAI SDK]
tags: [openai, agents, production, performance, optimization, best-practices]
author: toantranct
description: Hướng dẫn toàn diện về triển khai OpenAI Agents trong môi trường production. Từ performance optimization, scaling strategies đến cost management và security.
---

Sau khi đã thành thạo việc xây dựng agents với đầy đủ tính năng từ tools, handoffs đến tracing, giờ là lúc đưa chúng vào **thế giới thực** - môi trường production nơi mà hàng nghìn người dùng sẽ tương tác, nơi mà downtime có nghĩa là mất tiền, và nơi mà performance kém có thể làm hỏng trải nghiệm người dùng.

Production không chỉ là việc "chạy code trên server" mà là cả một nghệ thuật **tối ưu hóa, giám sát và bảo vệ** hệ thống AI. Giống như việc điều hành một nhà hàng - bạn không chỉ cần món ăn ngon mà còn phải đảm bảo phục vụ nhanh, giá cả hợp lý, vệ sinh an toàn và có thể phục vụ được đông khách.

Hôm nay chúng ta sẽ khám phá cách biến những agents "hoạt động tốt trên máy dev" thành những **production-grade systems** có thể xử lý traffic thực tế, tối ưu chi phí và đảm bảo độ tin cậy cao.

## Chiến Lược Deployment

### Environment Management

Việc đầu tiên khi chuẩn bị production là thiết lập các environment riêng biệt và rõ ràng:

```python
import os
from enum import Enum
from typing import Dict, Any, Optional
from dataclasses import dataclass
from agents import Agent, set_default_openai_key, set_default_openai_client
from openai import AsyncOpenAI

class Environment(Enum):
    DEVELOPMENT = "development"
    STAGING = "staging"
    PRODUCTION = "production"

@dataclass
class EnvironmentConfig:
    """Cấu hình cho từng environment"""
    name: Environment
    openai_api_key: str
    openai_base_url: Optional[str] = None
    max_concurrent_requests: int = 10
    request_timeout: int = 30
    rate_limit_per_minute: int = 100
    enable_tracing: bool = True
    log_level: str = "INFO"
    cost_limit_per_day: float = 100.0  # USD
    monitoring_enabled: bool = True

class ProductionEnvironmentManager:
    """Quản lý cấu hình cho các environment khác nhau"""
    
    def __init__(self):
        self.configs = self._load_environment_configs()
        self.current_env = self._detect_current_environment()
        
    def _load_environment_configs(self) -> Dict[Environment, EnvironmentConfig]:
        """Load cấu hình từ environment variables"""
        
        configs = {}
        
        # Development config
        configs[Environment.DEVELOPMENT] = EnvironmentConfig(
            name=Environment.DEVELOPMENT,
            openai_api_key=os.getenv("OPENAI_API_KEY_DEV", ""),
            max_concurrent_requests=5,
            request_timeout=60,  # Dài hơn để debug
            rate_limit_per_minute=50,
            enable_tracing=True,
            log_level="DEBUG",
            cost_limit_per_day=10.0,
            monitoring_enabled=False
        )
        
        # Staging config
        configs[Environment.STAGING] = EnvironmentConfig(
            name=Environment.STAGING,
            openai_api_key=os.getenv("OPENAI_API_KEY_STAGING", ""),
            max_concurrent_requests=20,
            request_timeout=30,
            rate_limit_per_minute=200,
            enable_tracing=True,
            log_level="INFO",
            cost_limit_per_day=50.0,
            monitoring_enabled=True
        )
        
        # Production config
        configs[Environment.PRODUCTION] = EnvironmentConfig(
            name=Environment.PRODUCTION,
            openai_api_key=os.getenv("OPENAI_API_KEY_PROD", ""),
            max_concurrent_requests=100,
            request_timeout=20,  # Ngắn hơn cho UX tốt
            rate_limit_per_minute=1000,
            enable_tracing=True,
            log_level="WARNING",  # Chỉ log những gì quan trọng
            cost_limit_per_day=500.0,
            monitoring_enabled=True
        )
        
        return configs
    
    def _detect_current_environment(self) -> Environment:
        """Tự động detect environment hiện tại"""
        
        env_name = os.getenv("ENVIRONMENT", "development").lower()
        
        if env_name in ["prod", "production"]:
            return Environment.PRODUCTION
        elif env_name in ["stage", "staging"]:
            return Environment.STAGING
        else:
            return Environment.DEVELOPMENT
    
    def setup_environment(self, env: Optional[Environment] = None):
        """Setup OpenAI client và các cấu hình cho environment"""
        
        target_env = env or self.current_env
        config = self.configs[target_env]
        
        print(f"🚀 Đang setup environment: {target_env.value}")
        
        # Setup OpenAI client
        if config.openai_base_url:
            client = AsyncOpenAI(
                api_key=config.openai_api_key,
                base_url=config.openai_base_url,
                timeout=config.request_timeout
            )
            set_default_openai_client(client)
        else:
            set_default_openai_key(config.openai_api_key)
        
        # Setup logging
        import logging
        logging.basicConfig(level=getattr(logging, config.log_level))
        
        # Setup tracing
        if not config.enable_tracing:
            from agents import set_tracing_disabled
            set_tracing_disabled(True)
        
        print(f"✅ Environment {target_env.value} đã được setup thành công")
        return config
    
    def get_config(self, env: Optional[Environment] = None) -> EnvironmentConfig:
        """Lấy config cho environment cụ thể"""
        target_env = env or self.current_env
        return self.configs[target_env]

# Production-ready agent factory
class ProductionAgentFactory:
    """Factory để tạo agents tối ưu cho production"""
    
    def __init__(self, env_manager: ProductionEnvironmentManager):
        self.env_manager = env_manager
        self.config = env_manager.get_config()
    
    def create_customer_service_agent(self) -> Agent:
        """Tạo customer service agent tối ưu cho production"""
        
        # Instructions được tối ưu cho performance và consistency
        optimized_instructions = """
        Bạn là chuyên viên hỗ trợ khách hàng chuyên nghiệp của công ty.
        
        NGUYÊN TẮC PHẢN HỒI:
        - Luôn thân thiện, lịch sự và chuyên nghiệp
        - Trả lời ngắn gọn nhưng đầy đủ thông tin
        - Ưu tiên giải pháp thực tế có thể thực hiện ngay
        - Hỏi thêm thông tin nếu câu hỏi không rõ ràng
        
        QUY TRÌNH XỬ LÝ:
        1. Xác định loại yêu cầu (thông tin, khiếu nại, hỗ trợ kỹ thuật)
        2. Cung cấp giải pháp phù hợp hoặc hướng dẫn tiếp theo
        3. Xác nhận khách hàng hiểu và hài lòng
        
        LƯU Ý QUAN TRỌNG:
        - Không đưa ra cam kết về thời gian cụ thể mà chưa được xác nhận
        - Luôn đề xuất escalate cho supervisor nếu vấn đề phức tạp
        - Ghi nhận thông tin khách hàng để theo dõi sau này
        
        Mục tiêu: Giải quyết vấn đề của khách hàng nhanh chóng và hiệu quả.
        """
        
        return Agent(
            name="Customer Service Pro",
            instructions=optimized_instructions,
            model="gpt-4o-mini",  # Cân bằng giữa performance và cost
            model_settings={
                "temperature": 0.3,  # Ít creative hơn để consistent
                "max_tokens": 500,   # Giới hạn để tránh phản hồi quá dài
            }
        )
    
    def create_sales_assistant_agent(self) -> Agent:
        """Tạo sales assistant tối ưu cho conversion"""
        
        sales_instructions = """
        Bạn là chuyên viên tư vấn bán hàng thông minh và thấu hiểu khách hàng.
        
        CHIẾN LƯỢC BÁN HÀNG:
        - Lắng nghe và hiểu nhu cầu thực sự của khách hàng
        - Tư vấn sản phẩm phù hợp nhất, không ép buộc
        - Làm nổi bật lợi ích cụ thể cho từng khách hàng
        - Xây dựng lòng tin qua việc cung cấp thông tin chính xác
        
        CÔNG CỤ PERSUASION:
        - Sử dụng social proof (đánh giá, testimonials)
        - Tạo urgency hợp lý (khuyến mãi có thời hạn)
        - Giải quyết objections một cách tự nhiên
        - Đề xuất upsell/cross-sell phù hợp
        
        QUY TRÌNH CHỐT ĐƠN:
        1. Khảo sát nhu cầu và budget
        2. Giới thiệu sản phẩm phù hợp
        3. Xử lý thắc mắc và lo ngại
        4. Tạo động lực mua hàng
        5. Hướng dẫn thực hiện giao dịch
        
        KPI: Tối ưu conversion rate và customer satisfaction.
        """
        
        return Agent(
            name="Sales Assistant Pro",
            instructions=sales_instructions,
            model="gpt-4o",  # Model tốt hơn cho sales conversations
            model_settings={
                "temperature": 0.7,  # Creative hơn để engaging
                "max_tokens": 600,
            }
        )

# Demo production setup
async def demo_production_setup():
    """Demo việc setup production environment"""
    
    print("🏭 **Production Environment Setup Demo**\n")
    
    # Setup environment manager
    env_manager = ProductionEnvironmentManager()
    
    # Test cấu hình cho các environments khác nhau
    environments = [Environment.DEVELOPMENT, Environment.STAGING, Environment.PRODUCTION]
    
    for env in environments:
        print(f"📋 **Environment: {env.value.upper()}**")
        
        config = env_manager.get_config(env)
        
        print(f"   • Max Concurrent: {config.max_concurrent_requests}")
        print(f"   • Timeout: {config.request_timeout}s")
        print(f"   • Rate Limit: {config.rate_limit_per_minute}/min")
        print(f"   • Cost Limit: ${config.cost_limit_per_day}/day")
        print(f"   • Tracing: {config.enable_tracing}")
        print(f"   • Monitoring: {config.monitoring_enabled}")
        print(f"   • Log Level: {config.log_level}")
        print()
    
    # Setup production environment
    prod_config = env_manager.setup_environment(Environment.PRODUCTION)
    
    # Tạo production agents
    agent_factory = ProductionAgentFactory(env_manager)
    
    customer_agent = agent_factory.create_customer_service_agent()
    sales_agent = agent_factory.create_sales_assistant_agent()
    
    print("🤖 **Production Agents Created:**")
    print(f"   • {customer_agent.name} - Optimized for support")
    print(f"   • {sales_agent.name} - Optimized for conversion")
    
    return env_manager, agent_factory

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_production_setup())
```

## Performance Optimization Strategies

### Request Processing Optimization

```python
import asyncio
import time
from typing import List, Dict, Any, Optional
from dataclasses import dataclass
from concurrent.futures import ThreadPoolExecutor
import aiohttp
from agents import Agent, Runner

@dataclass
class RequestMetrics:
    """Metrics cho từng request"""
    request_id: str
    start_time: float
    end_time: Optional[float] = None
    processing_time: Optional[float] = None
    tokens_used: int = 0
    success: bool = False
    error_message: Optional[str] = None

class HighPerformanceRequestProcessor:
    """Xử lý requests với performance cao"""
    
    def __init__(self, max_concurrent: int = 50):
        self.max_concurrent = max_concurrent
        self.semaphore = asyncio.Semaphore(max_concurrent)
        self.request_queue = asyncio.Queue()
        self.active_requests: Dict[str, RequestMetrics] = {}
        self.completed_requests: List[RequestMetrics] = []
        
        # Performance tracking
        self.total_requests = 0
        self.successful_requests = 0
        self.average_response_time = 0.0
        
        # Start background workers
        self.workers = []
        for i in range(min(max_concurrent, 10)):  # Tối đa 10 workers
            worker = asyncio.create_task(self._request_worker(f"worker_{i}"))
            self.workers.append(worker)
    
    async def _request_worker(self, worker_id: str):
        """Background worker xử lý requests từ queue"""
        
        print(f"🔧 Worker {worker_id} started")
        
        while True:
            try:
                # Lấy request từ queue
                request_data = await self.request_queue.get()
                
                if request_data is None:  # Shutdown signal
                    break
                
                # Xử lý request
                await self._process_single_request(request_data)
                
                # Mark task done
                self.request_queue.task_done()
                
            except Exception as e:
                print(f"❌ Worker {worker_id} error: {str(e)}")
    
    async def _process_single_request(self, request_data: Dict[str, Any]):
        """Xử lý một request cụ thể"""
        
        request_id = request_data["id"]
        agent = request_data["agent"]
        user_input = request_data["input"]
        
        # Tạo metrics tracking
        metrics = RequestMetrics(
            request_id=request_id,
            start_time=time.time()
        )
        
        self.active_requests[request_id] = metrics
        
        async with self.semaphore:  # Limit concurrent requests
            try:
                # Xử lý request với timeout
                result = await asyncio.wait_for(
                    Runner.run(agent, user_input),
                    timeout=30.0  # 30 second timeout
                )
                
                # Update metrics
                metrics.end_time = time.time()
                metrics.processing_time = metrics.end_time - metrics.start_time
                metrics.tokens_used = len(result.final_output.split())  # Approximate
                metrics.success = True
                
                # Store result
                request_data["result"] = result.final_output
                request_data["success"] = True
                
                self.successful_requests += 1
                
            except asyncio.TimeoutError:
                metrics.end_time = time.time()
                metrics.processing_time = metrics.end_time - metrics.start_time
                metrics.error_message = "Request timeout"
                request_data["error"] = "Request timeout after 30 seconds"
                request_data["success"] = False
                
            except Exception as e:
                metrics.end_time = time.time()
                metrics.processing_time = metrics.end_time - metrics.start_time
                metrics.error_message = str(e)
                request_data["error"] = str(e)
                request_data["success"] = False
            
            finally:
                # Update global metrics
                self.total_requests += 1
                
                # Calculate running average
                if metrics.processing_time:
                    self.average_response_time = (
                        (self.average_response_time * (self.total_requests - 1) + metrics.processing_time) 
                        / self.total_requests
                    )
                
                # Move to completed
                self.completed_requests.append(metrics)
                del self.active_requests[request_id]
                
                # Keep only last 1000 completed requests
                if len(self.completed_requests) > 1000:
                    self.completed_requests = self.completed_requests[-1000:]
    
    async def submit_request(self, agent: Agent, user_input: str) -> str:
        """Submit request và chờ kết quả"""
        
        request_id = f"req_{int(time.time() * 1000)}_{len(self.active_requests)}"
        
        # Tạo request data với callback mechanism
        request_data = {
            "id": request_id,
            "agent": agent,
            "input": user_input,
            "future": asyncio.Future()
        }
        
        # Add to queue
        await self.request_queue.put(request_data)
        
        # Wait for completion (polling approach)
        while request_id in self.active_requests or request_data.get("result") is None:
            if request_data.get("success") is not None:  # Completed
                break
            await asyncio.sleep(0.1)  # Poll every 100ms
        
        if request_data.get("success"):
            return request_data["result"]
        else:
            raise Exception(request_data.get("error", "Unknown error"))
    
    def get_performance_stats(self) -> Dict[str, Any]:
        """Lấy thống kê performance"""
        
        success_rate = (self.successful_requests / self.total_requests * 100) if self.total_requests > 0 else 0
        
        # Recent performance (last 100 requests)
        recent_requests = self.completed_requests[-100:]
        recent_avg_time = sum(r.processing_time for r in recent_requests if r.processing_time) / len(recent_requests) if recent_requests else 0
        
        return {
            "total_requests": self.total_requests,
            "successful_requests": self.successful_requests,
            "success_rate": success_rate,
            "average_response_time": self.average_response_time,
            "recent_average_response_time": recent_avg_time,
            "active_requests": len(self.active_requests),
            "queue_size": self.request_queue.qsize(),
            "max_concurrent": self.max_concurrent
        }
    
    async def shutdown(self):
        """Graceful shutdown"""
        print("🛑 Shutting down request processor...")
        
        # Stop workers
        for _ in self.workers:
            await self.request_queue.put(None)  # Shutdown signal
        
        # Wait for workers to finish
        await asyncio.gather(*self.workers, return_exceptions=True)
        
        print("✅ Request processor shutdown complete")

# Load testing system
class LoadTester:
    """Công cụ load testing cho agents"""
    
    def __init__(self, processor: HighPerformanceRequestProcessor):
        self.processor = processor
    
    async def run_load_test(
        self, 
        agent: Agent, 
        test_queries: List[str], 
        concurrent_users: int = 10,
        requests_per_user: int = 5
    ) -> Dict[str, Any]:
        """Chạy load test với concurrent users"""
        
        print(f"🚀 Starting load test:")
        print(f"   • Concurrent users: {concurrent_users}")
        print(f"   • Requests per user: {requests_per_user}")
        print(f"   • Total requests: {concurrent_users * requests_per_user}")
        
        start_time = time.time()
        
        # Tạo tasks cho concurrent users
        user_tasks = []
        
        for user_id in range(concurrent_users):
            user_task = self._simulate_user_requests(
                agent, test_queries, requests_per_user, f"user_{user_id}"
            )
            user_tasks.append(user_task)
        
        # Chạy tất cả user tasks đồng thời
        user_results = await asyncio.gather(*user_tasks, return_exceptions=True)
        
        end_time = time.time()
        total_duration = end_time - start_time
        
        # Tổng hợp kết quả
        total_requests = 0
        successful_requests = 0
        
        for result in user_results:
            if isinstance(result, dict):
                total_requests += result["requests"]
                successful_requests += result["successful"]
        
        # Performance stats
        final_stats = self.processor.get_performance_stats()
        
        return {
            "load_test_summary": {
                "concurrent_users": concurrent_users,
                "requests_per_user": requests_per_user,
                "total_duration": total_duration,
                "requests_per_second": total_requests / total_duration,
                "success_rate": (successful_requests / total_requests * 100) if total_requests > 0 else 0
            },
            "performance_stats": final_stats,
            "individual_user_results": [r for r in user_results if isinstance(r, dict)]
        }
    
    async def _simulate_user_requests(
        self, 
        agent: Agent, 
        queries: List[str], 
        num_requests: int, 
        user_id: str
    ) -> Dict[str, Any]:
        """Mô phỏng requests của một user"""
        
        import random
        
        successful = 0
        failed = 0
        response_times = []
        
        for i in range(num_requests):
            try:
                # Random query
                query = random.choice(queries)
                
                # Measure response time
                start = time.time()
                result = await self.processor.submit_request(agent, query)
                end = time.time()
                
                response_times.append(end - start)
                successful += 1
                
                # Random delay giữa requests (0.1-1s)
                await asyncio.sleep(random.uniform(0.1, 1.0))
                
            except Exception as e:
                failed += 1
                print(f"❌ {user_id} request {i+1} failed: {str(e)}")
        
        return {
            "user_id": user_id,
            "requests": num_requests,
            "successful": successful,
            "failed": failed,
            "avg_response_time": sum(response_times) / len(response_times) if response_times else 0,
            "min_response_time": min(response_times) if response_times else 0,
            "max_response_time": max(response_times) if response_times else 0
        }

# Demo performance optimization
async def demo_performance_optimization():
    """Demo hệ thống performance optimization"""
    
    print("⚡ **Performance Optimization Demo**\n")
    
    # Setup high-performance processor
    processor = HighPerformanceRequestProcessor(max_concurrent=20)
    
    # Tạo test agent
    test_agent = Agent(
        name="Performance Test Agent",
        instructions="""
        Bạn là một AI assistant được tối ưu cho performance testing.
        
        Quy tắc phản hồi:
        - Trả lời ngắn gọn và súc tích
        - Tập trung vào thông tin quan trọng
        - Tránh các câu văn dài dòng
        - Sử dụng bullet points khi phù hợp
        
        Mục tiêu: Cung cấp thông tin hữu ích với thời gian phản hồi tối ưu.
        """,
        model="gpt-4o-mini",  # Model nhanh cho testing
        model_settings={"temperature": 0.3, "max_tokens": 200}
    )
    
    # Test queries
    test_queries = [
        "Machine learning là gì?",
        "Lợi ích của cloud computing",
        "Cách tối ưu database performance",
        "Best practices cho API design",
        "Sự khác biệt giữa REST và GraphQL",
        "Microservices vs Monolith",
        "Docker container hoạt động như thế nào?",
        "CI/CD pipeline là gì?",
        "Cách secure một web application",
        "Performance monitoring tools nào tốt?"
    ]
    
    # Chạy load test
    load_tester = LoadTester(processor)
    
    print("🔥 Bắt đầu load test...")
    
    load_test_results = await load_tester.run_load_test(
        agent=test_agent,
        test_queries=test_queries,
        concurrent_users=8,
        requests_per_user=3
    )
    
    # Hiển thị kết quả
    summary = load_test_results["load_test_summary"]
    print(f"\n📊 **Load Test Results:**")
    print(f"   • Total Duration: {summary['total_duration']:.2f}s")
    print(f"   • Requests/Second: {summary['requests_per_second']:.2f}")
    print(f"   • Success Rate: {summary['success_rate']:.1f}%")
    
    perf_stats = load_test_results["performance_stats"]
    print(f"\n⚡ **Performance Stats:**")
    print(f"   • Total Requests: {perf_stats['total_requests']}")
    print(f"   • Average Response Time: {perf_stats['average_response_time']:.3f}s")
    print(f"   • Recent Average: {perf_stats['recent_average_response_time']:.3f}s")
    print(f"   • Max Concurrent: {perf_stats['max_concurrent']}")
    
    # User performance breakdown
    user_results = load_test_results["individual_user_results"]
    print(f"\n👥 **Per-User Performance:**")
    for user_result in user_results[:5]:  # Show first 5 users
        print(f"   • {user_result['user_id']}: {user_result['successful']}/{user_result['requests']} success, avg {user_result['avg_response_time']:.3f}s")
    
    # Cleanup
    await processor.shutdown()

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_performance_optimization())
```

## Cost Optimization và Resource Management

### Intelligent Cost Control

```python
import time
from typing import Dict, Any, List, Optional
from datetime import datetime, timedelta
from dataclasses import dataclass, field
from enum import Enum

class CostTier(Enum):
    """Các mức giá khác nhau cho different models"""
    BUDGET = "budget"      # gpt-4o-mini
    STANDARD = "standard"  # gpt-4o
    PREMIUM = "premium"    # gpt-4-turbo, o1

@dataclass
class ModelCostConfig:
    """Cấu hình cost cho từng model"""
    model_name: str
    cost_per_1k_input_tokens: float
    cost_per_1k_output_tokens: float
    tier: CostTier
    performance_score: int  # 1-10, cao hơn = performance tốt hơn

class IntelligentCostManager:
    """Quản lý chi phí thông minh cho OpenAI API"""
    
    def __init__(self, daily_budget: float = 100.0):
        self.daily_budget = daily_budget
        self.current_spend = 0.0
        self.spend_history: List[Dict[str, Any]] = []
        self.last_reset = datetime.now().date()
        
        # Model cost configurations (giá tính theo $)
        self.model_costs = {
            "gpt-4o-mini": ModelCostConfig(
                model_name="gpt-4o-mini",
                cost_per_1k_input_tokens=0.00015,
                cost_per_1k_output_tokens=0.0006,
                tier=CostTier.BUDGET,
                performance_score=7
            ),
            "gpt-4o": ModelCostConfig(
                model_name="gpt-4o",
                cost_per_1k_input_tokens=0.0025,
                cost_per_1k_output_tokens=0.01,
                tier=CostTier.STANDARD,
                performance_score=9
            ),
            "gpt-4-turbo": ModelCostConfig(
                model_name="gpt-4-turbo",
                cost_per_1k_input_tokens=0.01,
                cost_per_1k_output_tokens=0.03,
                tier=CostTier.PREMIUM,
                performance_score=10
            )
        }
        
        # Thresholds cho intelligent switching
        self.cost_thresholds = {
            "warning": 0.7,   # 70% of budget
            "critical": 0.9,  # 90% of budget
            "emergency": 0.95 # 95% of budget
        }
    
    def _reset_daily_budget_if_needed(self):
        """Reset budget nếu qua ngày mới"""
        today = datetime.now().date()
        
        if today > self.last_reset:
            print(f"📅 New day detected, resetting budget")
            print(f"   Yesterday's spend: ${self.current_spend:.4f}")
            
            # Archive yesterday's data
            self.spend_history.append({
                "date": self.last_reset.isoformat(),
                "total_spend": self.current_spend,
                "budget": self.daily_budget,
                "utilization": self.current_spend / self.daily_budget
            })
            
            # Reset for new day
            self.current_spend = 0.0
            self.last_reset = today
    
    def calculate_request_cost(
        self, 
        model: str, 
        input_tokens: int, 
        output_tokens: int
    ) -> float:
        """Tính cost cho một request cụ thể"""
        
        if model not in self.model_costs:
            print(f"⚠️ Unknown model {model}, using default pricing")
            model = "gpt-4o"  # Default fallback
        
        config = self.model_costs[model]
        
        input_cost = (input_tokens / 1000) * config.cost_per_1k_input_tokens
        output_cost = (output_tokens / 1000) * config.cost_per_1k_output_tokens
        
        return input_cost + output_cost
    
    def recommend_model(
        self, 
        complexity_score: int,  # 1-10, cao hơn = phức tạp hơn
        priority: str = "balanced"  # "cost", "performance", "balanced"
    ) -> str:
        """Recommend model dựa trên complexity và priority"""
        
        self._reset_daily_budget_if_needed()
        
        # Check budget status
        budget_utilization = self.current_spend / self.daily_budget
        
        # Emergency mode - chỉ dùng model rẻ nhất
        if budget_utilization >= self.cost_thresholds["emergency"]:
            print(f"🚨 Emergency mode: Budget {budget_utilization:.1%} used, forcing budget model")
            return "gpt-4o-mini"
        
        # Critical mode - prefer budget models
        elif budget_utilization >= self.cost_thresholds["critical"]:
            print(f"⚠️ Critical budget mode: {budget_utilization:.1%} used")
            if complexity_score <= 6:
                return "gpt-4o-mini"
            else:
                return "gpt-4o"  # Minimum acceptable for complex tasks
        
        # Warning mode - be more conservative
        elif budget_utilization >= self.cost_thresholds["warning"]:
            print(f"⚠️ Budget warning: {budget_utilization:.1%} used")
            if priority == "cost" or complexity_score <= 4:
                return "gpt-4o-mini"
            elif complexity_score <= 8:
                return "gpt-4o"
            else:
                return "gpt-4-turbo"
        
        # Normal mode - optimize based on priority
        else:
            if priority == "cost":
                return "gpt-4o-mini" if complexity_score <= 6 else "gpt-4o"
            elif priority == "performance":
                return "gpt-4-turbo" if complexity_score >= 7 else "gpt-4o"
            else:  # balanced
                if complexity_score <= 4:
                    return "gpt-4o-mini"
                elif complexity_score <= 8:
                    return "gpt-4o"
                else:
                    return "gpt-4-turbo"
    
    def record_usage(
        self, 
        model: str, 
        input_tokens: int, 
        output_tokens: int,
        request_metadata: Dict[str, Any] = None
    ):
        """Ghi nhận usage và update budget tracking"""
        
        cost = self.calculate_request_cost(model, input_tokens, output_tokens)
        self.current_spend += cost
        
        # Log usage
        usage_record = {
            "timestamp": datetime.now().isoformat(),
            "model": model,
            "input_tokens": input_tokens,
            "output_tokens": output_tokens,
            "cost": cost,
            "cumulative_spend": self.current_spend,
            "metadata": request_metadata or {}
        }
        
        print(f"💰 Usage recorded: {model} - ${cost:.4f} (Total today: ${self.current_spend:.4f})")
        
        return usage_record
    
    def get_budget_status(self) -> Dict[str, Any]:
        """Lấy trạng thái budget hiện tại"""
        
        self._reset_daily_budget_if_needed()
        
        utilization = self.current_spend / self.daily_budget
        remaining = self.daily_budget - self.current_spend
        
        # Determine status
        if utilization >= self.cost_thresholds["emergency"]:
            status = "emergency"
        elif utilization >= self.cost_thresholds["critical"]:
            status = "critical"
        elif utilization >= self.cost_thresholds["warning"]:
            status = "warning"
        else:
            status = "healthy"
        
        return {
            "date": self.last_reset.isoformat(),
            "daily_budget": self.daily_budget,
            "current_spend": self.current_spend,
            "remaining_budget": remaining,
            "utilization_percentage": utilization * 100,
            "status": status,
            "recommended_action": self._get_budget_recommendation(status, utilization)
        }
    
    def _get_budget_recommendation(self, status: str, utilization: float) -> str:
        """Đề xuất hành động dựa trên budget status"""
        
        if status == "emergency":
            return "Switch to budget models only, consider increasing daily budget"
        elif status == "critical":
            return "Prefer budget models, monitor usage closely"
        elif status == "warning":
            return "Be more selective with premium models"
        else:
            return "Budget healthy, can use models as needed"
    
    def get_cost_analytics(self) -> Dict[str, Any]:
        """Phân tích chi phí và đưa ra insights"""
        
        if not self.spend_history:
            return {"message": "Insufficient historical data"}
        
        # Recent performance
        recent_days = self.spend_history[-7:] if len(self.spend_history) >= 7 else self.spend_history
        
        avg_daily_spend = sum(day["total_spend"] for day in recent_days) / len(recent_days)
        avg_utilization = sum(day["utilization"] for day in recent_days) / len(recent_days)
        
        # Trends
        if len(recent_days) >= 2:
            recent_trend = recent_days[-1]["total_spend"] - recent_days[-2]["total_spend"]
            trend_direction = "increasing" if recent_trend > 0 else "decreasing" if recent_trend < 0 else "stable"
        else:
            trend_direction = "insufficient_data"
        
        return {
            "analytics_period": f"Last {len(recent_days)} days",
            "average_daily_spend": avg_daily_spend,
            "average_utilization": avg_utilization * 100,
            "trend_direction": trend_direction,
            "total_historical_spend": sum(day["total_spend"] for day in self.spend_history),
            "budget_efficiency": "good" if avg_utilization < 0.8 else "needs_optimization",
            "recommendations": self._generate_cost_recommendations(avg_utilization, trend_direction)
        }
    
    def _generate_cost_recommendations(self, avg_utilization: float, trend: str) -> List[str]:
        """Generate cost optimization recommendations"""
        
        recommendations = []
        
        if avg_utilization > 0.9:
            recommendations.append("Consider increasing daily budget or optimizing model usage")
        
        if trend == "increasing":
            recommendations.append("Monitor increasing cost trend, review model selection strategy")
        
        if avg_utilization < 0.5:
            recommendations.append("Budget utilization is low, could afford higher performance models")
        
        recommendations.append("Regularly review model performance vs cost trade-offs")
        recommendations.append("Consider implementing request batching for similar queries")
        
        return recommendations

# Cost-optimized agent wrapper
class CostOptimizedAgent:
    """Agent wrapper với intelligent cost optimization"""
    
    def __init__(self, base_agent: Agent, cost_manager: IntelligentCostManager):
        self.base_agent = base_agent
        self.cost_manager = cost_manager
        
    def _assess_query_complexity(self, query: str) -> int:
        """Đánh giá độ phức tạp của query (1-10)"""
        
        complexity_indicators = {
            "simple_keywords": ["gì là", "là gì", "nghĩa là", "how to", "what is"],
            "complex_keywords": ["phân tích", "so sánh", "đánh giá", "analyze", "compare", "evaluate"],
            "very_complex_keywords": ["strategy", "chiến lược", "optimization", "tối ưu", "comprehensive"]
        }
        
        query_lower = query.lower()
        
        # Base complexity từ length
        base_complexity = min(len(query) // 50 + 1, 5)  # 1-5 based on length
        
        # Adjust dựa trên keywords
        if any(keyword in query_lower for keyword in complexity_indicators["very_complex_keywords"]):
            complexity = max(base_complexity + 4, 8)  # 8-10
        elif any(keyword in query_lower for keyword in complexity_indicators["complex_keywords"]):
            complexity = max(base_complexity + 2, 6)  # 6-8
        elif any(keyword in query_lower for keyword in complexity_indicators["simple_keywords"]):
            complexity = min(base_complexity, 4)  # 1-4
        else:
            complexity = base_complexity
        
        return min(complexity, 10)
    
    async def run_cost_optimized(
        self, 
        query: str, 
        priority: str = "balanced"
    ) -> Dict[str, Any]:
        """Run agent với cost optimization"""
        
        # Assess complexity
        complexity = self._assess_query_complexity(query)
        
        # Get recommended model
        recommended_model = self.cost_manager.recommend_model(complexity, priority)
        
        print(f"🤖 Query complexity: {complexity}/10")
        print(f"💡 Recommended model: {recommended_model}")
        
        # Update agent model
        original_model = self.base_agent.model
        self.base_agent.model = recommended_model
        
        try:
            # Run agent
            start_time = time.time()
            result = await Runner.run(self.base_agent, query)
            end_time = time.time()
            
            # Estimate token usage (rough approximation)
            input_tokens = len(query.split()) * 1.3  # Approximate tokens
            output_tokens = len(result.final_output.split()) * 1.3
            
            # Record usage
            usage_record = self.cost_manager.record_usage(
                model=recommended_model,
                input_tokens=int(input_tokens),
                output_tokens=int(output_tokens),
                request_metadata={
                    "query_complexity": complexity,
                    "priority": priority,
                    "response_time": end_time - start_time
                }
            )
            
            return {
                "success": True,
                "result": result.final_output,
                "model_used": recommended_model,
                "complexity_score": complexity,
                "cost_info": {
                    "estimated_cost": usage_record["cost"],
                    "input_tokens": usage_record["input_tokens"],
                    "output_tokens": usage_record["output_tokens"]
                },
                "budget_status": self.cost_manager.get_budget_status()
            }
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e),
                "model_used": recommended_model,
                "complexity_score": complexity
            }
        
        finally:
            # Restore original model
            self.base_agent.model = original_model

# Demo cost optimization
async def demo_cost_optimization():
    """Demo intelligent cost optimization"""
    
    print("💰 **Cost Optimization Demo**\n")
    
    # Setup cost manager với budget nhỏ để demo
    cost_manager = IntelligentCostManager(daily_budget=5.0)  # $5/day for demo
    
    # Tạo base agent
    base_agent = Agent(
        name="Cost-Optimized Assistant",
        instructions="""
        Bạn là một AI assistant được tối ưu về cost và performance.
        
        Nguyên tắc:
        - Trả lời chính xác và hữu ích
        - Tối ưu độ dài phản hồi phù hợp với độ phức tạp câu hỏi
        - Ưu tiên thông tin quan trọng nhất
        - Tránh dài dòng không cần thiết
        """,
        model="gpt-4o"  # Default model
    )
    
    # Wrap với cost optimization
    cost_optimized_agent = CostOptimizedAgent(base_agent, cost_manager)
    
    # Test queries với độ phức tạp khác nhau
    test_scenarios = [
        {
            "query": "Python là gì?",
            "priority": "cost",
            "expected_complexity": "low"
        },
        {
            "query": "So sánh performance giữa React và Vue.js, phân tích ưu nhược điểm của từng framework",
            "priority": "balanced", 
            "expected_complexity": "medium"
        },
        {
            "query": "Xây dựng chiến lược comprehensive để optimize database performance cho enterprise application với millions of users",
            "priority": "performance",
            "expected_complexity": "high"
        },
        {
            "query": "Cách install Docker?",
            "priority": "cost",
            "expected_complexity": "low"
        }
    ]
    
    total_cost = 0.0
    
    for i, scenario in enumerate(test_scenarios, 1):
        print(f"🎯 **Test {i}: {scenario['expected_complexity'].title()} Complexity**")
        print(f"📝 Query: {scenario['query'][:80]}...")
        print(f"🎖️ Priority: {scenario['priority']}")
        
        result = await cost_optimized_agent.run_cost_optimized(
            scenario["query"],
            scenario["priority"]
        )
        
        if result["success"]:
            cost_info = result["cost_info"]
            total_cost += cost_info["estimated_cost"]
            
            print(f"✅ Model: {result['model_used']}")
            print(f"💰 Cost: ${cost_info['estimated_cost']:.4f}")
            print(f"📊 Tokens: {cost_info['input_tokens']:.0f} in, {cost_info['output_tokens']:.0f} out")
            print(f"🎯 Response: {result['result'][:150]}...")
            
            # Budget status
            budget_status = result["budget_status"]
            print(f"💳 Budget: ${budget_status['current_spend']:.4f}/${budget_status['daily_budget']:.2f} ({budget_status['utilization_percentage']:.1f}%)")
            
        else:
            print(f"❌ Error: {result['error']}")
        
        print("-" * 80 + "\n")
    
    # Final cost analytics
    analytics = cost_manager.get_cost_analytics()
    
    print("📈 **Cost Analytics:**")
    if "message" not in analytics:
        print(f"💵 Total Demo Cost: ${total_cost:.4f}")
        print(f"📊 Average Daily Spend: ${analytics['average_daily_spend']:.4f}")
        print(f"📈 Budget Efficiency: {analytics['budget_efficiency']}")
        
        print(f"\n💡 **Recommendations:**")
        for rec in analytics["recommendations"]:
            print(f"   • {rec}")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_cost_optimization())
```

## Security và Compliance

### Production Security Framework

```python
import hashlib
import secrets
import jwt
from typing import Dict, Any, List, Optional, Set
from datetime import datetime, timedelta
from dataclasses import dataclass
from enum import Enum
import re

class SecurityLevel(Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    CRITICAL = "critical"

@dataclass
class SecurityPolicy:
    """Chính sách bảo mật cho agents"""
    max_input_length: int = 10000
    allowed_file_types: Set[str] = None
    rate_limit_per_user: int = 100  # requests per hour
    session_timeout_minutes: int = 30
    require_authentication: bool = True
    log_all_interactions: bool = True
    content_filtering_enabled: bool = True
    pii_detection_enabled: bool = True
    
    def __post_init__(self):
        if self.allowed_file_types is None:
            self.allowed_file_types = {'.txt', '.pdf', '.doc', '.docx'}

class ProductionSecurityManager:
    """Quản lý bảo mật cho production environment"""
    
    def __init__(self, secret_key: str, security_level: SecurityLevel = SecurityLevel.HIGH):
        self.secret_key = secret_key
        self.security_level = security_level
        self.security_policy = self._create_security_policy()
        
        # Tracking security events
        self.security_events: List[Dict[str, Any]] = []
        self.blocked_ips: Set[str] = set()
        self.rate_limits: Dict[str, List[datetime]] = {}
        
        # PII patterns
        self.pii_patterns = {
            'email': r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
            'phone': r'(\+84|0)[0-9]{9,10}',
            'credit_card': r'\b(?:\d{4}[-\s]?){3}\d{4}\b',
            'ssn': r'\b\d{3}-?\d{2}-?\d{4}\b'
        }
        
        print(f"🔐 Security Manager initialized with {security_level.value} security level")
    
    def _create_security_policy(self) -> SecurityPolicy:
        """Tạo security policy dựa trên security level"""
        
        if self.security_level == SecurityLevel.CRITICAL:
            return SecurityPolicy(
                max_input_length=5000,
                rate_limit_per_user=50,
                session_timeout_minutes=15,
                require_authentication=True,
                log_all_interactions=True,
                content_filtering_enabled=True,
                pii_detection_enabled=True
            )
        elif self.security_level == SecurityLevel.HIGH:
            return SecurityPolicy(
                max_input_length=8000,
                rate_limit_per_user=100,
                session_timeout_minutes=30,
                require_authentication=True,
                log_all_interactions=True,
                content_filtering_enabled=True,
                pii_detection_enabled=True
            )
        elif self.security_level == SecurityLevel.MEDIUM:
            return SecurityPolicy(
                max_input_length=10000,
                rate_limit_per_user=200,
                session_timeout_minutes=60,
                require_authentication=True,
                log_all_interactions=True,
                content_filtering_enabled=True,
                pii_detection_enabled=False
            )
        else:  # LOW
            return SecurityPolicy(
                max_input_length=15000,
                rate_limit_per_user=500,
                session_timeout_minutes=120,
                require_authentication=False,
                log_all_interactions=False,
                content_filtering_enabled=False,
                pii_detection_enabled=False
            )
    
    def generate_secure_token(self, user_id: str, permissions: List[str]) -> str:
        """Tạo secure JWT token"""
        
        payload = {
            'user_id': user_id,
            'permissions': permissions,
            'iat': datetime.utcnow(),
            'exp': datetime.utcnow() + timedelta(hours=24),
            'nonce': secrets.token_hex(16)
        }
        
        token = jwt.encode(payload, self.secret_key, algorithm='HS256')
        
        # Log token generation
        self._log_security_event(
            event_type="token_generated",
            user_id=user_id,
            details={"permissions": permissions}
        )
        
        return token
    
    def validate_token(self, token: str) -> Optional[Dict[str, Any]]:
        """Validate JWT token"""
        
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=['HS256'])
            
            # Check if token is not expired
            if datetime.utcnow() > datetime.fromisoformat(payload['exp'].replace('Z', '+00:00')):
                self._log_security_event(
                    event_type="token_expired",
                    user_id=payload.get('user_id'),
                    details={"token_hash": hashlib.sha256(token.encode()).hexdigest()[:16]}
                )
                return None
            
            return payload
            
        except jwt.InvalidTokenError as e:
            self._log_security_event(
                event_type="invalid_token",
                details={"error": str(e), "token_hash": hashlib.sha256(token.encode()).hexdigest()[:16]}
            )
            return None
    
    def check_rate_limit(self, user_id: str, ip_address: str) -> bool:
        """Kiểm tra rate limiting"""
        
        if ip_address in self.blocked_ips:
            self._log_security_event(
                event_type="blocked_ip_access",
                user_id=user_id,
                details={"ip": ip_address}
            )
            return False
        
        now = datetime.now()
        hour_ago = now - timedelta(hours=1)
        
        # Clean old requests
        if user_id in self.rate_limits:
            self.rate_limits[user_id] = [
                req_time for req_time in self.rate_limits[user_id]
                if req_time > hour_ago
            ]
        else:
            self.rate_limits[user_id] = []
        
        # Check rate limit
        current_requests = len(self.rate_limits[user_id])
        
        if current_requests >= self.security_policy.rate_limit_per_user:
            self._log_security_event(
                event_type="rate_limit_exceeded",
                user_id=user_id,
                details={"requests_in_hour": current_requests, "limit": self.security_policy.rate_limit_per_user}
            )
            return False
        
        # Record this request
        self.rate_limits[user_id].append(now)
        return True
    
    def validate_input(self, user_input: str, user_id: str) -> Dict[str, Any]:
        """Validate user input cho security threats"""
        
        validation_result = {
            "valid": True,
            "issues": [],
            "risk_level": "low",
            "sanitized_input": user_input
        }
        
        # Check input length
        if len(user_input) > self.security_policy.max_input_length:
            validation_result["valid"] = False
            validation_result["issues"].append(f"Input too long: {len(user_input)} > {self.security_policy.max_input_length}")
            validation_result["risk_level"] = "medium"
        
        # Check for potential injection attacks
        injection_patterns = [
            r'<script[^>]*>.*?</script>',  # XSS
            r'javascript:',
            r'on\w+\s*=',  # Event handlers
            r'union\s+select',  # SQL injection
            r'drop\s+table',
            r'insert\s+into',
            r'delete\s+from'
        ]
        
        for pattern in injection_patterns:
            if re.search(pattern, user_input, re.IGNORECASE):
                validation_result["valid"] = False
                validation_result["issues"].append(f"Potential injection attack detected: {pattern}")
                validation_result["risk_level"] = "high"
        
        # PII detection
        if self.security_policy.pii_detection_enabled:
            pii_found = self._detect_pii(user_input)
            if pii_found:
                validation_result["issues"].append(f"PII detected: {', '.join(pii_found.keys())}")
                validation_result["risk_level"] = "medium"
                
                # Sanitize PII
                sanitized = user_input
                for pii_type, matches in pii_found.items():
                    for match in matches:
                        sanitized = sanitized.replace(match, f"[{pii_type.upper()}_REDACTED]")
                validation_result["sanitized_input"] = sanitized
        
        # Content filtering
        if self.security_policy.content_filtering_enabled:
            if self._contains_inappropriate_content(user_input):
                validation_result["valid"] = False
                validation_result["issues"].append("Inappropriate content detected")
                validation_result["risk_level"] = "high"
        
        # Log security validation
        if not validation_result["valid"] or validation_result["issues"]:
            self._log_security_event(
                event_type="input_validation_failed",
                user_id=user_id,
                details={
                    "issues": validation_result["issues"],
                    "risk_level": validation_result["risk_level"],
                    "input_length": len(user_input)
                }
            )
        
        return validation_result
    
    def _detect_pii(self, text: str) -> Dict[str, List[str]]:
        """Phát hiện PII trong text"""
        
        pii_found = {}
        
        for pii_type, pattern in self.pii_patterns.items():
            matches = re.findall(pattern, text)
            if matches:
                pii_found[pii_type] = matches
        
        return pii_found
    
    def _contains_inappropriate_content(self, text: str) -> bool:
        """Kiểm tra inappropriate content"""
        
        inappropriate_keywords = [
            'hack', 'exploit', 'malware', 'phishing',
            'illegal', 'fraud', 'scam'
        ]
        
        text_lower = text.lower()
        return any(keyword in text_lower for keyword in inappropriate_keywords)
    
    def _log_security_event(
        self, 
        event_type: str, 
        user_id: str = None, 
        details: Dict[str, Any] = None
    ):
        """Log security events"""
        
        event = {
            "timestamp": datetime.now().isoformat(),
            "event_type": event_type,
            "user_id": user_id,
            "details": details or {},
            "security_level": self.security_level.value
        }
        
        self.security_events.append(event)
        
        # Keep only last 1000 events
        if len(self.security_events) > 1000:
            self.security_events = self.security_events[-1000:]
        
        # Print critical events
        if event_type in ["blocked_ip_access", "rate_limit_exceeded", "input_validation_failed"]:
            print(f"🚨 Security Event: {event_type} - User: {user_id}")
    
    def get_security_report(self) -> Dict[str, Any]:
        """Tạo báo cáo bảo mật"""
        
        if not self.security_events:
            return {"message": "No security events recorded"}
        
        # Event statistics
        event_types = {}
        for event in self.security_events:
            event_type = event["event_type"]
            event_types[event_type] = event_types.get(event_type, 0) + 1
        
        # Recent events (last 24 hours)
        day_ago = datetime.now() - timedelta(hours=24)
        recent_events = [
            event for event in self.security_events
            if datetime.fromisoformat(event["timestamp"]) > day_ago
        ]
        
        # Risk assessment
        high_risk_events = [
            event for event in recent_events
            if event["event_type"] in ["blocked_ip_access", "rate_limit_exceeded", "input_validation_failed"]
        ]
        
        risk_level = "high" if len(high_risk_events) > 10 else "medium" if len(high_risk_events) > 3 else "low"
        
        return {
            "report_generated": datetime.now().isoformat(),
            "security_level": self.security_level.value,
            "total_events": len(self.security_events),
            "events_last_24h": len(recent_events),
            "event_type_breakdown": event_types,
            "current_risk_level": risk_level,
            "blocked_ips_count": len(self.blocked_ips),
            "active_rate_limits": len(self.rate_limits),
            "recent_high_risk_events": len(high_risk_events),
            "security_recommendations": self._generate_security_recommendations(risk_level, event_types)
        }
    
    def _generate_security_recommendations(
        self, 
        risk_level: str, 
        event_types: Dict[str, int]
    ) -> List[str]:
        """Generate security recommendations"""
        
        recommendations = []
        
        if risk_level == "high":
            recommendations.append("Immediate review of security events required")
            
        if event_types.get("rate_limit_exceeded", 0) > 5:
            recommendations.append("Consider implementing IP-based blocking for repeat offenders")
            
        if event_types.get("input_validation_failed", 0) > 10:
            recommendations.append("Review and strengthen input validation rules")
            
        if event_types.get("invalid_token", 0) > 3:
            recommendations.append("Monitor for potential token brute force attempts")
        
        recommendations.extend([
            "Regularly rotate security keys and tokens",
            "Monitor security events dashboard daily",
            "Keep security policies updated with latest threats",
            "Conduct periodic security audits"
        ])
        
        return recommendations

# Secure agent wrapper
class SecureAgent:
    """Agent wrapper với comprehensive security"""
    
    def __init__(self, agent: Agent, security_manager: ProductionSecurityManager):
        self.agent = agent
        self.security_manager = security_manager
        
    async def secure_run(
        self, 
        user_input: str, 
        user_id: str,
        auth_token: str,
        ip_address: str = "127.0.0.1"
    ) -> Dict[str, Any]:
        """Run agent với full security checks"""
        
        # 1. Token validation
        if self.security_manager.security_policy.require_authentication:
            token_payload = self.security_manager.validate_token(auth_token)
            if not token_payload:
                return {
                    "success": False,
                    "error": "Invalid or expired authentication token",
                    "error_code": "AUTH_FAILED"
                }
            
            # Check if user_id matches token
            if token_payload.get("user_id") != user_id:
                return {
                    "success": False,
                    "error": "User ID mismatch",
                    "error_code": "AUTH_MISMATCH"
                }
        
        # 2. Rate limiting
        if not self.security_manager.check_rate_limit(user_id, ip_address):
            return {
                "success": False,
                "error": "Rate limit exceeded",
                "error_code": "RATE_LIMIT"
            }
        
        # 3. Input validation
        validation_result = self.security_manager.validate_input(user_input, user_id)
        
        if not validation_result["valid"]:
            return {
                "success": False,
                "error": "Input validation failed",
                "error_code": "INVALID_INPUT",
                "validation_issues": validation_result["issues"]
            }
        
        # 4. Run agent với sanitized input
        try:
            result = await Runner.run(self.agent, validation_result["sanitized_input"])
            
            # Log successful interaction
            if self.security_manager.security_policy.log_all_interactions:
                self.security_manager._log_security_event(
                    event_type="agent_interaction_success",
                    user_id=user_id,
                    details={
                        "input_length": len(user_input),
                        "output_length": len(result.final_output),
                        "sanitized": validation_result["sanitized_input"] != user_input
                    }
                )
            
            return {
                "success": True,
                "result": result.final_output,
                "security_info": {
                    "input_sanitized": validation_result["sanitized_input"] != user_input,
                    "risk_level": validation_result["risk_level"]
                }
            }
            
        except Exception as e:
            # Log error
            self.security_manager._log_security_event(
                event_type="agent_execution_error",
                user_id=user_id,
                details={"error": str(e)}
            )
            
            return {
                "success": False,
                "error": "Agent execution failed",
                "error_code": "EXECUTION_ERROR"
            }

# Demo production security
async def demo_production_security():
    """Demo comprehensive production security"""
    
    print("🔐 **Production Security Demo**\n")
    
    # Setup security manager
    security_manager = ProductionSecurityManager(
        secret_key="production_secret_key_2025",
        security_level=SecurityLevel.HIGH
    )
    
    # Tạo secure agent
    base_agent = Agent(
        name="Secure Production Agent",
        instructions="""
        Bạn là một AI assistant được triển khai trong môi trường production bảo mật cao.
        
        Nguyên tắc bảo mật:
        - Không bao giờ tiết lộ thông tin nhạy cảm
        - Từ chối các yêu cầu có thể gây hại
        - Báo cáo các hoạt động đáng ngờ
        - Tuân thủ nghiêm ngặt các chính sách bảo mật
        
        Phong cách phản hồi:
        - Chuyên nghiệp và cẩn thận
        - Minh bạch về giới hạn bảo mật
        - Hướng dẫn người dùng sử dụng an toàn
        """
    )
    
    secure_agent = SecureAgent(base_agent, security_manager)
    
    # Generate token cho test user
    test_user_id = "user_12345"
    auth_token = security_manager.generate_secure_token(
        user_id=test_user_id,
        permissions=["chat", "query"]
    )
    
    print(f"🎫 Generated auth token for {test_user_id}")
    
    # Test scenarios
    security_test_cases = [
        {
            "name": "Normal Query",
            "input": "Giải thích machine learning cơ bản",
            "should_succeed": True
        },
        {
            "name": "PII in Input",
            "input": "Tôi tên Nguyễn Văn An, email an@company.com, SĐT 0901234567. Hỏi về AI.",
            "should_succeed": True  # Sẽ sanitize PII
        },
        {
            "name": "Potential XSS Attack",
            "input": "<script>alert('hack')</script> Explain JavaScript",
            "should_succeed": False
        },
        {
            "name": "SQL Injection Attempt",
            "input": "'; DROP TABLE users; -- Explain databases",
            "should_succeed": False
        },
        {
            "name": "Very Long Input",
            "input": ("A" * 12000) + " What is AI?",  # Vượt quá limit
            "should_succeed": False
        }
    ]
    
    for i, test_case in enumerate(security_test_cases, 1):
        print(f"🧪 **Test {i}: {test_case['name']}**")
        
        result = await secure_agent.secure_run(
            user_input=test_case["input"],
            user_id=test_user_id,
            auth_token=auth_token,
            ip_address="192.168.1.100"
        )
        
        success_expected = test_case["should_succeed"]
        actual_success = result["success"]
        
        if actual_success == success_expected:
            status = "✅ PASS"
        else:
            status = "❌ FAIL"
        
        print(f"   {status} - Expected: {success_expected}, Got: {actual_success}")
        
        if result["success"]:
            print(f"   📝 Response: {result['result'][:100]}...")
            if result.get("security_info", {}).get("input_sanitized"):
                print(f"   🧹 Input was sanitized for security")
        else:
            print(f"   🚫 Blocked: {result['error']} ({result.get('error_code')})")
        
        print()
    
    # Test rate limiting
    print("🚦 **Testing Rate Limiting**")
    
    # Make multiple requests to trigger rate limit
    for i in range(5):
        result = await secure_agent.secure_run(
            user_input=f"Test request {i+1}",
            user_id=test_user_id,
            auth_token=auth_token,
            ip_address="192.168.1.100"
        )
        
        if result["success"]:
            print(f"   ✅ Request {i+1}: Success")
        else:
            print(f"   🚫 Request {i+1}: {result['error']}")
    
    # Security report
    print("\n📊 **Security Report**")
    
    security_report = security_manager.get_security_report()
    
    print(f"🔐 Security Level: {security_report['security_level']}")
    print(f"📈 Total Events: {security_report['total_events']}")
    print(f"⏰ Events (24h): {security_report['events_last_24h']}")
    print(f"🚨 Risk Level: {security_report['current_risk_level']}")
    
    print(f"\n📋 **Event Breakdown:**")
    for event_type, count in security_report['event_type_breakdown'].items():
        print(f"   • {event_type}: {count}")
    
    print(f"\n💡 **Security Recommendations:**")
    for rec in security_report['security_recommendations'][:5]:
        print(f"   • {rec}")

if __name__ == "__main__":
    from agents import Runner
    import asyncio
    asyncio.run(demo_production_security())
```

## Những Điều Cần Biết Khi Đưa Vào Sản Xuất

### Kiến Thức Cốt Lõi

✅ **Environment management** - Phân chia rõ ràng dev, staging và production  
✅ **Performance optimization** - Request processing với high concurrency  
✅ **Cost management** - Intelligent model selection dựa trên complexity  
✅ **Security framework** - Authentication, validation và monitoring toàn diện  
✅ **Monitoring và alerting** - Theo dõi liên tục để phát hiện vấn đề sớm  

### Kinh Nghiệm Từ Thực Tế

🎯 **Về Trải Nghiệm Người Dùng:**
- Luôn prioritize response time và availability
- Implement graceful degradation khi hệ thống quá tải
- Cung cấp feedback rõ ràng khi có lỗi xảy ra
- Design cho mobile-first nếu có nhiều user mobile
- Test thoroughly trên các devices và browsers khác nhau

⚡ **Về Hiệu Suất:**
- Monitor response time như metric quan trọng nhất
- Implement caching ở nhiều layers khác nhau
- Use connection pooling để optimize database connections
- Consider CDN cho static assets và responses
- Load test với realistic traffic patterns

🔧 **Về Triển Khai:**
- Automate deployment process hoàn toàn
- Implement blue-green deployment để minimize downtime
- Have rollback strategy sẵn sàng
- Monitor system health ngay sau mỗi deployment
- Keep deployment logs để troubleshoot khi cần

### Hướng Dẫn Triển Khai Production

📊 **Thiết Lập Monitoring:**
- Track key metrics: response time, error rate, throughput
- Set up alerts cho các thresholds quan trọng
- Monitor cost và usage patterns daily
- Implement health checks cho tất cả components
- Create dashboards cho different stakeholders

🔒 **Bảo Mật Production:**
- Implement defense-in-depth strategy
- Regular security audits và penetration testing
- Keep all dependencies updated
- Monitor cho suspicious activities
- Have incident response plan ready

💰 **Quản Lý Chi Phí:**
- Set up budget alerts và automatic controls
- Monitor usage patterns để optimize model selection
- Implement request batching khi có thể
- Regular cost analysis và optimization
- Consider reserved instances cho predictable workloads

### Những Sai Lầm Thường Gặp

❌ **Thiếu monitoring** → Không biết khi nào system có vấn đề  
❌ **Không có backup plan** → Không thể recover khi có sự cố  
❌ **Over-engineering từ đầu** → Waste time và resources  
❌ **Ignore security** → Vulnerabilities có thể được exploit  
❌ **Không test với real data** → Surprise khi go-live  

### Chiến Lược Scaling

**🏗️ Horizontal Scaling:**
- Design stateless services từ đầu cho easy replication
- Sử dụng load balancers để distribute traffic evenly
- Implement database sharding strategies khi single DB không đủ
- Consider microservices architecture cho large systems
- Plan cho multi-region deployment để serve global users

**📈 Vertical Scaling:**
- Monitor resource utilization patterns để identify bottlenecks
- Upgrade hardware khi cost-effective và necessary
- Optimize memory usage và CPU utilization
- Consider GPU instances cho AI-intensive workloads
- Profile application performance để identify optimization opportunities

### Kế Hoạch Disaster Recovery

🚨 **Chuẩn Bị Cho Sự Cố:**
- Document tất cả procedures và runbooks chi tiết
- Thực hiện regular backup và recovery testing
- Maintain multiple communication channels
- Train team members về incident response procedures
- Keep updated emergency contact lists và escalation paths

🔄 **Business Continuity:**
- Define clear RTO (Recovery Time Objective) và RPO (Recovery Point Objective)
- Implement redundancy ở critical system components
- Have alternative service providers sẵn sàng
- Conduct regular disaster recovery drills
- Maintain updated incident response và communication plans

### Tối Ưu Chi Phí Trong Dài Hạn

💡 **Chiến Lược Cost Optimization:**
- Analyze usage patterns để predict và optimize costs
- Implement smart caching để reduce unnecessary API calls
- Sử dụng cheaper models cho simple, routine queries
- Consider training custom models cho specific, high-volume use cases
- Regular review và cleanup unused resources và subscriptions

📊 **Cost Tracking:**
- Implement detailed cost attribution và tracking
- Track ROI của different features và use cases
- Setup cost allocation tags cho different projects/departments
- Conduct regular cost review meetings với stakeholders
- Forecast future costs based on growth projections và plans

### Chuẩn Bị Cho Tương Lai

🔮 **Technology Evolution:**
- Stay updated với latest AI developments
- Plan cho model upgrades và migrations
- Consider emerging technologies như multimodal AI
- Invest in team training và development
- Build flexible architecture có thể adapt

🌱 **Business Growth:**
- Plan infrastructure scaling cho anticipated growth
- Consider international expansion requirements
- Prepare cho regulatory compliance changes
- Build partnerships với technology vendors
- Invest in automation để handle scale

### Checklist Cuối Cùng Trước Khi Go-Live

**✅ Sẵn Sàng Kỹ Thuật:**
- [ ] Load testing passed với expected traffic volumes
- [ ] Security audit completed và all vulnerabilities addressed
- [ ] Monitoring và alerting systems hoàn tất và tested
- [ ] Backup và recovery procedures tested successfully
- [ ] Performance benchmarks established và documented

**✅ Sẵn Sàng Vận Hành:**
- [ ] Team training completed cho all relevant personnel
- [ ] Documentation updated và easily accessible
- [ ] Incident response procedures ready và tested
- [ ] Customer support processes established
- [ ] Communication plans prepared cho different scenarios

**✅ Sẵn Sàng Business:**
- [ ] User acceptance testing passed với real users
- [ ] Legal và compliance requirements fully met
- [ ] Marketing materials prepared cho launch
- [ ] Customer support team ready với proper training
- [ ] Success metrics defined và tracking systems ready

---

*Chúc mừng bạn đã hoàn thành hành trình khám phá OpenAI Agents SDK! Từ những bước đầu tiên với "Hello World" đến việc triển khai production-grade systems, bạn đã trang bị đầy đủ kiến thức để xây dựng những AI applications mạnh mẽ và đáng tin cậy. Hãy áp dụng những kiến thức này vào projects thực tế và tiếp tục học hỏi từ community để không ngừng cải thiện skills của mình.*