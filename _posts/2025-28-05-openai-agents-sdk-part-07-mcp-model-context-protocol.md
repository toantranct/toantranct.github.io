---
title: "Model Context Protocol (MCP): Extending Agent Capabilities"
date: 2025-05-24 09:00:00 +0700
categories: [AI, OpenAI, MCP]
tags: [openai, agents, mcp, protocols, extensibility]
author: toantranct
description: Hướng dẫn chi tiết về Model Context Protocol (MCP) trong OpenAI Agents SDK. Từ cơ bản đến nâng cao - kết nối agents với filesystem, databases và enterprise systems.
---

Trong những bài trước, chúng ta đã khám phá cách agents sử dụng function tools và hosted tools để tương tác với thế giới bên ngoài. Nhưng khi muốn kết nối với các hệ thống phức tạp như databases, enterprise software, hay cloud services thì sao? Đây chính là lúc **Model Context Protocol (MCP)** phát huy sức mạnh.

MCP như một "cái cầu nối thông minh" - cho phép agents kết nối với hàng ngàn dịch vụ và hệ thống khác nhau mà không cần viết lại code từ đầu. Hôm nay chúng ta sẽ tìm hiểu cách MCP biến agents từ những "công cụ đơn lẻ" thành những "trung tâm tích hợp" có thể làm việc với toàn bộ hạ tầng công nghệ của bạn.

## MCP Là Gì Và Tại Sao Quan Trọng?

### Định Nghĩa Đơn Giản

**Model Context Protocol (MCP)** là một chuẩn mở cho phép các ứng dụng AI kết nối với nguồn dữ liệu và công cụ bên ngoài một cách chuẩn hóa. Hiểu đơn giản:

> MCP giống như **USB-C cho ứng dụng AI** - một chuẩn kết nối thống nhất cho phép bạn "cắm" agents vào bất kỳ hệ thống nào có hỗ trợ MCP.

### Vấn Đề Mà MCP Giải Quyết

![Without MCP: Fragmented AI Development](/assets/img/posts/open-ai-sdk/without-mcp.png)
*Hình: Phát triển AI phân mảnh khi không có MCP*

**🔴 Trước Khi Có MCP:**
- Mỗi ứng dụng AI phải tự viết code riêng để kết nối với databases, APIs
- Không có chuẩn chung → duplicate work, khó maintain
- Khi thay đổi hệ thống → phải viết lại code cho tất cả ứng dụng
- Developers mất thời gian viết integration thay vì focus vào logic AI

**✅ Với MCP:**

![With MCP: Standardized AI Development](/assets/img/posts/open-ai-sdk/with-mcp.png)
*Hình: Phát triển AI chuẩn hóa với MCP*

- Một lần implement MCP server → tất cả AI apps có thể sử dụng
- Chuẩn hóa cách AI tương tác với external systems
- Tái sử dụng code và effort
- Dễ dàng thêm/thay đổi data sources

### Lợi Ích Thực Tế

🔌 **Plug-and-Play Integration** - Kết nối nhanh với existing systems  
🔄 **Reusability** - Một MCP server phục vụ nhiều AI applications  
🛠️ **Standardization** - Consistent interface cho mọi integration  
⚡ **Development Speed** - Giảm thời gian implement integrations  
🎯 **Focus** - Developers tập trung vào AI logic thay vì plumbing code  

## Kiến Trúc MCP

### Client-Server Architecture

![Client-Server Architecture](/assets/img/posts/open-ai-sdk/mcp-client-server-architecture.png)
*Hình: Kiến trúc Client-Server của MCP*

MCP hoạt động theo mô hình client-server:

**🏠 Host (AI Application):**
- Ứng dụng chính chứa LLM agents (ví dụ: Claude Desktop, IDEs, AI agents)
- Muốn truy cập dữ liệu thông qua MCP

**👤 MCP Client:**
- Component trong Host application
- Duy trì kết nối 1:1 với MCP Servers
- Gửi requests và nhận responses

**🗄️ MCP Server:**
- Lightweight programs expose specific capabilities
- Mỗi server chuyên về một loại dữ liệu/dịch vụ cụ thể
- Có thể là local programs hoặc remote services

### Transport Methods

![Transports for Remote Servers](/assets/img/posts/open-ai-sdk/mcp-transport-for-remote-servers.png)
*Hình: Các phương thức transport cho remote servers*

MCP hỗ trợ nhiều cách kết nối:

**📡 STDIO (Standard Input/Output):**
- MCP server chạy như subprocess của application
- Giao tiếp qua stdin/stdout
- Thích hợp cho local tools và scripts

**🌐 HTTP + SSE (Server-Sent Events):**
- MCP server chạy remote, giao tiếp qua HTTP
- Sử dụng SSE cho real-time updates
- Stateful connection

**⚡ Streamable HTTP:**
- Phiên bản mới hơn của HTTP transport
- Hỗ trợ cả stateless và stateful connections
- Tối ưu hơn cho performance

## Bắt Đầu Với MCP

### Filesystem MCP Server

Hãy bắt đầu với ví dụ đơn giản - kết nối với filesystem:

```python
import asyncio
from agents import Agent, Runner, MCPServerStdio

# Tạo filesystem MCP server
async def setup_filesystem_server():
    """Setup MCP server cho filesystem access"""
    
    # Sử dụng official filesystem server từ npm
    filesystem_server = MCPServerStdio(
        params={
            "command": "npx",
            "args": [
                "-y", 
                "@modelcontextprotocol/server-filesystem", 
                "/path/to/your/documents"  # Thư mục muốn access
            ]
        }
    )
    
    return filesystem_server

# Agent với filesystem access
async def demo_filesystem_integration():
    """Demo tích hợp với filesystem qua MCP"""
    
    # Setup MCP server
    fs_server = await setup_filesystem_server()
    
    # Tạo agent với MCP server
    file_manager_agent = Agent(
        name="File Manager Assistant",
        instructions="""
        Bạn là một trợ lý quản lý tập tin thông minh.
        
        Khả năng của bạn:
        - Đọc và ghi files
        - Tạo, xóa, di chuyển files/folders
        - Tìm kiếm nội dung trong files
        - Phân tích structure của thư mục
        
        Cách làm việc:
        1. Hiểu rõ yêu cầu của user
        2. Sử dụng MCP tools để thao tác với filesystem
        3. Báo cáo kết quả một cách rõ ràng
        4. Đề xuất các hành động tiếp theo nếu cần
        
        Lưu ý an toàn:
        - Luôn xác nhận trước khi xóa files
        - Backup quan trọng data trước khi thay đổi
        - Báo cáo nếu có lỗi xảy ra
        """,
        mcp_servers=[fs_server]
    )
    
    # Test các tác vụ file management
    tasks = [
        "Liệt kê tất cả files trong thư mục hiện tại",
        "Tạo một file mới tên 'hello.txt' với nội dung 'Hello from MCP!'",
        "Tìm tất cả files .py trong thư mục và subdirectories",
        "Đọc nội dung file 'hello.txt' vừa tạo"
    ]
    
    async with fs_server:
        for task in tasks:
            print(f"📁 **Task:** {task}")
            
            try:
                result = await Runner.run(file_manager_agent, task)
                print(f"✅ **Kết quả:** {result.final_output}")
            except Exception as e:
                print(f"❌ **Lỗi:** {str(e)}")
            
            print("-" * 60 + "\n")

if __name__ == "__main__":
    asyncio.run(demo_filesystem_integration())
```

### Database MCP Server

Tiếp theo, hãy tạo MCP server cho database access:

```python
import json
import sqlite3
from typing import Dict, List, Any
from pydantic import BaseModel

class DatabaseMCPServer:
    """Custom MCP Server cho database operations"""
    
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.connection = None
    
    async def initialize(self):
        """Khởi tạo database connection"""
        self.connection = sqlite3.connect(self.db_path)
        self.connection.row_factory = sqlite3.Row  # Return rows as dicts
        
        # Tạo sample tables nếu chưa có
        await self._create_sample_data()
    
    async def _create_sample_data(self):
        """Tạo sample data để demo"""
        cursor = self.connection.cursor()
        
        # Tạo bảng customers
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS customers (
                id INTEGER PRIMARY KEY,
                name TEXT NOT NULL,
                email TEXT UNIQUE,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        
        # Tạo bảng orders
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS orders (
                id INTEGER PRIMARY KEY,
                customer_id INTEGER,
                product_name TEXT,
                amount DECIMAL(10,2),
                order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                FOREIGN KEY (customer_id) REFERENCES customers (id)
            )
        """)
        
        # Insert sample data nếu tables trống
        cursor.execute("SELECT COUNT(*) FROM customers")
        if cursor.fetchone()[0] == 0:
            sample_customers = [
                ("Nguyễn Văn An", "an@example.com"),
                ("Trần Thị Bình", "binh@example.com"),
                ("Lê Văn Cường", "cuong@example.com")
            ]
            
            cursor.executemany(
                "INSERT INTO customers (name, email) VALUES (?, ?)",
                sample_customers
            )
            
            sample_orders = [
                (1, "Laptop Dell XPS", 25000000),
                (1, "Mouse Logitech", 500000),
                (2, "iPhone 15", 30000000),
                (3, "Headphones Sony", 2000000)
            ]
            
            cursor.executemany(
                "INSERT INTO orders (customer_id, product_name, amount) VALUES (?, ?, ?)",
                sample_orders
            )
        
        self.connection.commit()
    
    async def list_tools(self) -> List[Dict[str, Any]]:
        """Liệt kê các tools có sẵn"""
        return [
            {
                "name": "query_database",
                "description": "Thực hiện SELECT query trên database",
                "inputSchema": {
                    "type": "object",
                    "properties": {
                        "query": {
                            "type": "string",
                            "description": "SQL SELECT query"
                        }
                    },
                    "required": ["query"]
                }
            },
            {
                "name": "get_table_schema",
                "description": "Lấy schema của table",
                "inputSchema": {
                    "type": "object", 
                    "properties": {
                        "table_name": {
                            "type": "string",
                            "description": "Tên table cần xem schema"
                        }
                    },
                    "required": ["table_name"]
                }
            },
            {
                "name": "insert_record",
                "description": "Thêm record mới vào table",
                "inputSchema": {
                    "type": "object",
                    "properties": {
                        "table_name": {"type": "string"},
                        "data": {
                            "type": "object",
                            "description": "Dữ liệu để insert"
                        }
                    },
                    "required": ["table_name", "data"]
                }
            }
        ]
    
    async def call_tool(self, name: str, arguments: Dict[str, Any]) -> Dict[str, Any]:
        """Thực hiện tool call"""
        
        if name == "query_database":
            return await self._query_database(arguments["query"])
        
        elif name == "get_table_schema":
            return await self._get_table_schema(arguments["table_name"])
        
        elif name == "insert_record":
            return await self._insert_record(arguments["table_name"], arguments["data"])
        
        else:
            return {"error": f"Unknown tool: {name}"}
    
    async def _query_database(self, query: str) -> Dict[str, Any]:
        """Thực hiện SELECT query"""
        try:
            # Chỉ cho phép SELECT queries để đảm bảo an toàn
            if not query.strip().upper().startswith("SELECT"):
                return {"error": "Chỉ được phép thực hiện SELECT queries"}
            
            cursor = self.connection.cursor()
            cursor.execute(query)
            
            rows = cursor.fetchall()
            
            # Convert rows thành list of dicts
            result = []
            for row in rows:
                result.append(dict(row))
            
            return {
                "success": True,
                "data": result,
                "row_count": len(result)
            }
            
        except Exception as e:
            return {"error": f"Database error: {str(e)}"}
    
    async def _get_table_schema(self, table_name: str) -> Dict[str, Any]:
        """Lấy schema của table"""
        try:
            cursor = self.connection.cursor()
            cursor.execute(f"PRAGMA table_info({table_name})")
            
            schema_info = cursor.fetchall()
            
            columns = []
            for col in schema_info:
                columns.append({
                    "name": col[1],
                    "type": col[2],
                    "not_null": bool(col[3]),
                    "default_value": col[4],
                    "primary_key": bool(col[5])
                })
            
            return {
                "success": True,
                "table_name": table_name,
                "columns": columns
            }
            
        except Exception as e:
            return {"error": f"Schema error: {str(e)}"}
    
    async def _insert_record(self, table_name: str, data: Dict[str, Any]) -> Dict[str, Any]:
        """Thêm record mới"""
        try:
            columns = list(data.keys())
            values = list(data.values())
            placeholders = ", ".join(["?"] * len(values))
            
            query = f"""
                INSERT INTO {table_name} ({', '.join(columns)}) 
                VALUES ({placeholders})
            """
            
            cursor = self.connection.cursor()
            cursor.execute(query, values)
            self.connection.commit()
            
            return {
                "success": True,
                "message": f"Đã thêm record vào {table_name}",
                "inserted_id": cursor.lastrowid
            }
            
        except Exception as e:
            return {"error": f"Insert error: {str(e)}"}
    
    async def close(self):
        """Đóng database connection"""
        if self.connection:
            self.connection.close()

# Demo database integration
async def demo_database_integration():
    """Demo tích hợp với database qua custom MCP server"""
    
    # Khởi tạo database MCP server
    db_server = DatabaseMCPServer("demo.db")
    await db_server.initialize()
    
    # Tạo agent có thể làm việc với database
    database_agent = Agent(
        name="Database Analysis Assistant",
        instructions="""
        Bạn là một chuyên gia phân tích dữ liệu với khả năng truy vấn database.
        
        Khả năng của bạn:
        - Truy vấn dữ liệu từ database
        - Phân tích trends và patterns
        - Tạo reports từ raw data
        - Thêm records mới khi cần
        
        Database hiện tại có:
        - Bảng customers: thông tin khách hàng
        - Bảng orders: thông tin đơn hàng
        
        Quy trình làm việc:
        1. Hiểu requirements từ user
        2. Xem schema của tables liên quan
        3. Viết và thực hiện appropriate queries
        4. Phân tích kết quả và đưa ra insights
        5. Trình bày findings một cách rõ ràng
        
        Luôn giải thích SQL queries và kết quả để user hiểu.
        """,
        # Note: Trong thực tế, bạn sẽ setup db_server như MCP server thật
        # Ở đây chúng ta simulate bằng cách tạo custom tools
    )
    
    # Simulate MCP integration bằng function tools
    from agents import function_tool, RunContextWrapper
    
    @function_tool
    async def query_database(ctx: RunContextWrapper, query: str) -> str:
        """Query database và trả về kết quả"""
        result = await db_server.call_tool("query_database", {"query": query})
        
        if "error" in result:
            return f"❌ Lỗi: {result['error']}"
        
        data = result["data"]
        if not data:
            return "📋 Không tìm thấy dữ liệu nào."
        
        # Format output
        output = f"📊 **Kết quả query:** ({result['row_count']} rows)\n\n"
        
        # Show first few rows
        for i, row in enumerate(data[:5]):
            output += f"**Row {i+1}:** {json.dumps(row, ensure_ascii=False, indent=2)}\n"
        
        if len(data) > 5:
            output += f"\n... và {len(data) - 5} rows khác."
        
        return output
    
    @function_tool 
    async def get_table_schema(ctx: RunContextWrapper, table_name: str) -> str:
        """Lấy schema của table"""
        result = await db_server.call_tool("get_table_schema", {"table_name": table_name})
        
        if "error" in result:
            return f"❌ Lỗi: {result['error']}"
        
        columns = result["columns"]
        output = f"📋 **Schema của table '{table_name}':**\n\n"
        
        for col in columns:
            pk = " (PRIMARY KEY)" if col["primary_key"] else ""
            nn = " NOT NULL" if col["not_null"] else ""
            default = f" DEFAULT {col['default_value']}" if col["default_value"] else ""
            
            output += f"• **{col['name']}**: {col['type']}{pk}{nn}{default}\n"
        
        return output
    
    # Add tools to agent
    database_agent.tools = [query_database, get_table_schema]
    
    # Test database operations
    analysis_tasks = [
        "Cho tôi xem schema của bảng customers và orders",
        "Hiển thị tất cả khách hàng và email của họ",
        "Tìm khách hàng nào có tổng giá trị đơn hàng cao nhất",
        "Thống kê số lượng đơn hàng theo customer",
        "Tính tổng doanh thu từ tất cả đơn hàng"
    ]
    
    for task in analysis_tasks:
        print(f"💾 **Analysis Task:** {task}")
        
        try:
            result = await Runner.run(database_agent, task)
            print(f"📈 **Phân tích:** {result.final_output}")
        except Exception as e:
            print(f"❌ **Lỗi:** {str(e)}")
        
        print("=" * 70 + "\n")
    
    # Cleanup
    await db_server.close()

if __name__ == "__main__":
    asyncio.run(demo_database_integration())
```

## Advanced MCP Integration

### Enterprise Systems Integration

```python
from typing import Optional, Dict, Any, List
from datetime import datetime
import httpx
import asyncio

class CRMMCPServer:
    """MCP Server tích hợp với CRM system"""
    
    def __init__(self, crm_api_url: str, api_key: str):
        self.api_url = crm_api_url
        self.api_key = api_key
        self.client = httpx.AsyncClient(
            headers={"Authorization": f"Bearer {api_key}"}
        )
    
    async def list_tools(self) -> List[Dict[str, Any]]:
        """Available CRM tools"""
        return [
            {
                "name": "search_contacts",
                "description": "Tìm kiếm contacts trong CRM",
                "inputSchema": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string", "description": "Search query"},
                        "limit": {"type": "integer", "default": 10}
                    },
                    "required": ["query"]
                }
            },
            {
                "name": "get_contact_details",
                "description": "Lấy chi tiết contact theo ID",
                "inputSchema": {
                    "type": "object",
                    "properties": {
                        "contact_id": {"type": "string"}
                    },
                    "required": ["contact_id"]
                }
            },
            {
                "name": "create_opportunity",
                "description": "Tạo opportunity mới",
                "inputSchema": {
                    "type": "object",
                    "properties": {
                        "contact_id": {"type": "string"},
                        "title": {"type": "string"},
                        "value": {"type": "number"},
                        "stage": {"type": "string", "default": "prospecting"}
                    },
                    "required": ["contact_id", "title", "value"]
                }
            },
            {
                "name": "update_contact_notes",
                "description": "Cập nhật notes cho contact",
                "inputSchema": {
                    "type": "object",
                    "properties": {
                        "contact_id": {"type": "string"},
                        "notes": {"type": "string"}
                    },
                    "required": ["contact_id", "notes"]
                }
            }
        ]
    
    async def call_tool(self, name: str, arguments: Dict[str, Any]) -> Dict[str, Any]:
        """Execute CRM tool"""
        
        try:
            if name == "search_contacts":
                return await self._search_contacts(
                    arguments["query"], 
                    arguments.get("limit", 10)
                )
            
            elif name == "get_contact_details":
                return await self._get_contact_details(arguments["contact_id"])
            
            elif name == "create_opportunity":
                return await self._create_opportunity(arguments)
            
            elif name == "update_contact_notes":
                return await self._update_contact_notes(
                    arguments["contact_id"], 
                    arguments["notes"]
                )
            
            else:
                return {"error": f"Unknown tool: {name}"}
                
        except Exception as e:
            return {"error": f"CRM API error: {str(e)}"}
    
    async def _search_contacts(self, query: str, limit: int) -> Dict[str, Any]:
        """Tìm kiếm contacts"""
        # Simulate CRM API call
        # Trong thực tế sẽ gọi real CRM API
        
        mock_contacts = [
            {
                "id": "contact_1",
                "name": "Nguyễn Văn Anh",
                "email": "anh@company.com",
                "company": "Tech Solutions",
                "status": "active",
                "last_contact": "2025-01-20"
            },
            {
                "id": "contact_2", 
                "name": "Trần Thị Bình",
                "email": "binh@startup.vn",
                "company": "Innovation Hub",
                "status": "prospect",
                "last_contact": "2025-01-18"
            }
        ]
        
        # Filter based on query
        filtered = [
            c for c in mock_contacts 
            if query.lower() in c["name"].lower() or 
               query.lower() in c["company"].lower()
        ]
        
        return {
            "success": True,
            "contacts": filtered[:limit],
            "total_found": len(filtered)
        }
    
    async def _get_contact_details(self, contact_id: str) -> Dict[str, Any]:
        """Lấy chi tiết contact"""
        # Mock detailed contact info
        mock_detail = {
            "id": contact_id,
            "name": "Nguyễn Văn Anh",
            "email": "anh@company.com",
            "phone": "+84901234567",
            "company": "Tech Solutions",
            "position": "CTO",
            "status": "active",
            "created_date": "2024-12-01",
            "last_contact": "2025-01-20",
            "notes": "Quan tâm đến AI solutions. Scheduled demo for next week.",
            "opportunities": [
                {
                    "id": "opp_1",
                    "title": "AI Consulting Project",
                    "value": 50000,
                    "stage": "proposal",
                    "probability": 75
                }
            ]
        }
        
        return {
            "success": True,
            "contact": mock_detail
        }
    
    async def _create_opportunity(self, data: Dict[str, Any]) -> Dict[str, Any]:
        """Tạo opportunity mới"""
        new_opp = {
            "id": f"opp_{datetime.now().timestamp()}",
            "contact_id": data["contact_id"],
            "title": data["title"],
            "value": data["value"],
            "stage": data.get("stage", "prospecting"),
            "created_date": datetime.now().isoformat(),
            "probability": 25  # Default for new opportunities
        }
        
        return {
            "success": True,
            "opportunity": new_opp,
            "message": f"Đã tạo opportunity '{data['title']}' với giá trị ${data['value']:,}"
        }
    
    async def _update_contact_notes(self, contact_id: str, notes: str) -> Dict[str, Any]:
        """Cập nhật notes"""
        return {
            "success": True,
            "message": f"Đã cập nhật notes cho contact {contact_id}",
            "updated_at": datetime.now().isoformat()
        }

# Multi-system integration agent
class EnterpriseIntegrationSystem:
    """Hệ thống tích hợp doanh nghiệp với multiple MCP servers"""
    
    def __init__(self):
        # Initialize different MCP servers
        self.crm_server = CRMMCPServer("https://api.crm.com", "crm_api_key")
        
        # Trong thực tế sẽ có thêm:
        # self.email_server = EmailMCPServer(...)
        # self.calendar_server = CalendarMCPServer(...)
        # self.accounting_server = AccountingMCPServer(...)
        
        self.integration_agent = Agent(
            name="Enterprise Integration Assistant",
            instructions="""
            Bạn là trợ lý tích hợp doanh nghiệp với khả năng làm việc với nhiều hệ thống:
            
            Hệ thống có sẵn:
            - CRM: Quản lý contacts, opportunities, sales pipeline
            - Email: Gửi/nhận emails, templates, campaigns
            - Calendar: Lên lịch meetings, events, reminders
            - Accounting: Invoices, payments, financial reports
            
            Capabilities:
            - Tìm kiếm và quản lý customer data
            - Tạo opportunities và track sales progress
            - Schedule meetings và follow-ups
            - Generate reports từ multiple systems
            - Automate workflows across platforms
            
            Workflow approach:
            1. Hiểu business context và requirements
            2. Identify các systems cần tương tác
            3. Execute operations theo proper sequence
            4. Consolidate results và provide insights
            5. Suggest next steps và automation opportunities
            
            Luôn đảm bảo data consistency across systems.
            """,
            # Trong implementation thật sẽ add MCP servers:
            # mcp_servers=[self.crm_server, self.email_server, ...]
        )
    
    async def business_workflow(self, workflow_description: str) -> Dict[str, Any]:
        """Execute complex business workflow"""
        
        print(f"🏢 **Enterprise Workflow:** {workflow_description}")
        
        # Simulate MCP integration với function tools
        from agents import function_tool, RunContextWrapper
        
        @function_tool
        async def search_crm_contacts(ctx: RunContextWrapper, query: str) -> str:
            """Tìm kiếm contacts trong CRM"""
            result = await self.crm_server.call_tool("search_contacts", {"query": query})
            
            if "error" in result:
                return f"❌ CRM Error: {result['error']}"
            
            contacts = result["contacts"]
            if not contacts:
                return f"📋 Không tìm thấy contacts cho query: {query}"
            
            output = f"👥 **Tìm thấy {len(contacts)} contacts:**\n\n"
            for contact in contacts:
                output += f"• **{contact['name']}** ({contact['company']})\n"
                output += f"  Email: {contact['email']} | Status: {contact['status']}\n"
                output += f"  Last Contact: {contact['last_contact']}\n\n"
            
            return output
        
        @function_tool
        async def get_contact_details(ctx: RunContextWrapper, contact_id: str) -> str:
            """Lấy chi tiết contact"""
            result = await self.crm_server.call_tool("get_contact_details", {"contact_id": contact_id})
            
            if "error" in result:
                return f"❌ Error: {result['error']}"
            
            contact = result["contact"]
            output = f"👤 **Chi tiết Contact:**\n\n"
            output += f"**Tên:** {contact['name']}\n"
            output += f"**Company:** {contact['company']} - {contact['position']}\n"
            output += f"**Liên hệ:** {contact['email']} | {contact['phone']}\n"
            output += f"**Status:** {contact['status']}\n"
            output += f"**Notes:** {contact['notes']}\n\n"
            
            if contact.get("opportunities"):
                output += "💰 **Opportunities:**\n"
                for opp in contact["opportunities"]:
                    output += f"• {opp['title']}: ${opp['value']:,} ({opp['stage']}) - {opp['probability']}%\n"
            
            return output
        
        @function_tool
        async def create_opportunity(
            ctx: RunContextWrapper, 
            contact_id: str, 
            title: str, 
            value: float
        ) -> str:
            """Tạo opportunity mới"""
            result = await self.crm_server.call_tool("create_opportunity", {
                "contact_id": contact_id,
                "title": title,
                "value": value
            })
            
            if "error" in result:
                return f"❌ Error: {result['error']}"
            
            return f"✅ {result['message']}"
        
        # Add tools to agent
        self.integration_agent.tools = [
            search_crm_contacts, 
            get_contact_details, 
            create_opportunity
        ]
        
        # Execute workflow
        try:
            result = await Runner.run(self.integration_agent, workflow_description)
            
            return {
                "success": True,
                "workflow": workflow_description,
                "result": result.final_output,
                "timestamp": datetime.now().isoformat()
            }
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e),
                "workflow": workflow_description
            }

# Demo enterprise integration
async def demo_enterprise_integration():
    integration_system = EnterpriseIntegrationSystem()
    
    # Complex business workflows
    workflows = [
        """
        Tôi cần chuẩn bị cho meeting với potential client về AI consulting project:
        1. Tìm thông tin contact có tên 'Anh' hoặc company 'Tech'
        2. Xem chi tiết contact và opportunities hiện tại
        3. Nếu chưa có opportunity nào, tạo một opportunity mới 'AI Strategy Consulting' với giá trị $75,000
        4. Tóm tắt thông tin để chuẩ bị meeting
        """,
        
        """
        Phân tích sales pipeline hiện tại:
        1. Tìm tất cả contacts có status 'prospect'
        2. Xem chi tiết từng prospect và opportunities
        3. Tính tổng giá trị potential revenue
        4. Đề xuất actions để convert prospects thành customers
        """,
        
        """
        Follow-up với existing clients:
        1. Tìm contacts có last_contact > 1 tuần trước
        2. Xem opportunities đang pending
        3. Tạo plan để follow-up và move opportunities forward
        """
    ]
    
    for workflow in workflows:
        print("🔄 " + "="*80)
        
        result = await integration_system.business_workflow(workflow)
        
        if result["success"]:
            print(f"✅ **Workflow completed successfully**")
            print(f"**Result:**\n{result['result']}")
        else:
            print(f"❌ **Workflow failed:** {result['error']}")
        
        print("\n" + "="*80 + "\n")

if __name__ == "__main__":
    asyncio.run(demo_enterprise_integration())
```

## Production MCP Deployment

### Caching và Performance Optimization

```python
import time
from typing import Dict, Any, Optional
import json
import hashlib

class OptimizedMCPClient:
    """MCP Client với performance optimization"""
    
    def __init__(self, cache_ttl: int = 300):  # 5 minutes default
        self.cache_ttl = cache_ttl
        self.tools_cache: Dict[str, Dict[str, Any]] = {}
        self.response_cache: Dict[str, Dict[str, Any]] = {}
        
    def _generate_cache_key(self, server_id: str, operation: str, params: Dict = None) -> str:
        """Generate cache key cho operation"""
        key_data = f"{server_id}:{operation}:{json.dumps(params or {}, sort_keys=True)}"
        return hashlib.md5(key_data.encode()).hexdigest()
    
    async def get_cached_tools(self, server_id: str, server) -> List[Dict[str, Any]]:
        """Get tools with caching"""
        current_time = time.time()
        
        # Check cache
        if server_id in self.tools_cache:
            cached_entry = self.tools_cache[server_id]
            if current_time - cached_entry["timestamp"] < self.cache_ttl:
                print(f"📦 Using cached tools for {server_id}")
                return cached_entry["tools"]
        
        # Fetch fresh tools
        print(f"🔄 Fetching fresh tools for {server_id}")
        tools = await server.list_tools()
        
        # Cache the result
        self.tools_cache[server_id] = {
            "tools": tools,
            "timestamp": current_time
        }
        
        return tools
    
    async def call_with_cache(
        self, 
        server_id: str, 
        server, 
        tool_name: str, 
        params: Dict[str, Any],
        cache_enabled: bool = True
    ) -> Dict[str, Any]:
        """Call tool với caching support"""
        
        if not cache_enabled:
            return await server.call_tool(tool_name, params)
        
        # Generate cache key
        cache_key = self._generate_cache_key(server_id, tool_name, params)
        current_time = time.time()
        
        # Check response cache
        if cache_key in self.response_cache:
            cached_entry = self.response_cache[cache_key]
            if current_time - cached_entry["timestamp"] < self.cache_ttl:
                print(f"📦 Using cached response for {tool_name}")
                return cached_entry["response"]
        
        # Execute tool call
        print(f"🔄 Executing fresh call: {tool_name}")
        start_time = time.time()
        response = await server.call_tool(tool_name, params)
        execution_time = time.time() - start_time
        
        # Cache successful responses only
        if "error" not in response:
            self.response_cache[cache_key] = {
                "response": response,
                "timestamp": current_time,
                "execution_time": execution_time
            }
        
        return response
    
    def invalidate_cache(self, server_id: str = None):
        """Invalidate cache"""
        if server_id:
            # Invalidate specific server cache
            self.tools_cache.pop(server_id, None)
            
            # Remove response cache entries for this server
            keys_to_remove = [
                key for key in self.response_cache.keys() 
                if key.startswith(server_id + ":")
            ]
            
            for key in keys_to_remove:
                self.response_cache.pop(key, None)
        else:
            # Clear all cache
            self.tools_cache.clear()
            self.response_cache.clear()
        
        print(f"🗑️ Cache invalidated for {server_id or 'all servers'}")
    
    def get_cache_stats(self) -> Dict[str, Any]:
        """Get cache statistics"""
        return {
            "tools_cache_size": len(self.tools_cache),
            "response_cache_size": len(self.response_cache),
            "cache_ttl": self.cache_ttl,
            "servers_cached": list(self.tools_cache.keys())
        }

# Production-ready MCP integration
class ProductionMCPSystem:
    """Production-ready MCP system với monitoring và error handling"""
    
    def __init__(self):
        self.mcp_client = OptimizedMCPClient(cache_ttl=600)  # 10 minutes
        self.server_health: Dict[str, Dict[str, Any]] = {}
        self.error_counts: Dict[str, int] = {}
        self.performance_metrics: Dict[str, List[float]] = {}
    
    async def health_check_server(self, server_id: str, server) -> Dict[str, Any]:
        """Check health của MCP server"""
        start_time = time.time()
        
        try:
            # Simple health check - try to list tools
            tools = await server.list_tools()
            response_time = time.time() - start_time
            
            health_status = {
                "status": "healthy",
                "response_time": response_time,
                "tools_count": len(tools),
                "last_check": datetime.now().isoformat(),
                "error": None
            }
            
            # Reset error count on successful health check
            self.error_counts[server_id] = 0
            
        except Exception as e:
            response_time = time.time() - start_time
            self.error_counts[server_id] = self.error_counts.get(server_id, 0) + 1
            
            health_status = {
                "status": "unhealthy",
                "response_time": response_time,
                "tools_count": 0,
                "last_check": datetime.now().isoformat(),
                "error": str(e)
            }
        
        self.server_health[server_id] = health_status
        
        # Track performance metrics
        if server_id not in self.performance_metrics:
            self.performance_metrics[server_id] = []
        
        self.performance_metrics[server_id].append(response_time)
        
        # Keep only last 100 measurements
        if len(self.performance_metrics[server_id]) > 100:
            self.performance_metrics[server_id] = self.performance_metrics[server_id][-100:]
        
        return health_status
    
    async def execute_with_monitoring(
        self, 
        server_id: str, 
        server, 
        tool_name: str, 
        params: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Execute tool call với comprehensive monitoring"""
        
        # Pre-execution health check
        health = await self.health_check_server(server_id, server)
        
        if health["status"] != "healthy":
            return {
                "error": f"Server {server_id} is unhealthy: {health['error']}",
                "server_status": health
            }
        
        # Execute with caching
        start_time = time.time()
        
        try:
            result = await self.mcp_client.call_with_cache(
                server_id, server, tool_name, params
            )
            
            execution_time = time.time() - start_time
            
            # Log successful execution
            print(f"✅ {server_id}.{tool_name} executed in {execution_time:.3f}s")
            
            return {
                "success": True,
                "result": result,
                "execution_time": execution_time,
                "server_id": server_id,
                "tool_name": tool_name
            }
            
        except Exception as e:
            execution_time = time.time() - start_time
            self.error_counts[server_id] = self.error_counts.get(server_id, 0) + 1
            
            print(f"❌ {server_id}.{tool_name} failed in {execution_time:.3f}s: {str(e)}")
            
            return {
                "success": False,
                "error": str(e),
                "execution_time": execution_time,
                "server_id": server_id,
                "tool_name": tool_name
            }
    
    def get_system_status(self) -> Dict[str, Any]:
        """Get comprehensive system status"""
        
        # Calculate average response times
        avg_response_times = {}
        for server_id, times in self.performance_metrics.items():
            if times:
                avg_response_times[server_id] = sum(times) / len(times)
        
        # Identify problematic servers
        problematic_servers = [
            server_id for server_id, count in self.error_counts.items()
            if count > 5  # More than 5 errors
        ]
        
        return {
            "timestamp": datetime.now().isoformat(),
            "servers_health": self.server_health,
            "error_counts": self.error_counts,
            "avg_response_times": avg_response_times,
            "problematic_servers": problematic_servers,
            "cache_stats": self.mcp_client.get_cache_stats(),
            "total_servers": len(self.server_health),
            "healthy_servers": len([
                h for h in self.server_health.values() 
                if h["status"] == "healthy"
            ])
        }
    
    async def maintenance_routine(self):
        """Routine maintenance tasks"""
        print("🔧 Running MCP system maintenance...")
        
        # Clear old cache entries
        self.mcp_client.invalidate_cache()
        
        # Reset error counts for recovered servers
        for server_id in list(self.error_counts.keys()):
            if (server_id in self.server_health and 
                self.server_health[server_id]["status"] == "healthy"):
                self.error_counts[server_id] = 0
        
        # Clean old performance metrics
        for server_id in self.performance_metrics:
            self.performance_metrics[server_id] = self.performance_metrics[server_id][-50:]
        
        print("✅ Maintenance completed")

# Demo production MCP system
async def demo_production_mcp():
    """Demo production-ready MCP system"""
    
    production_system = ProductionMCPSystem()
    
    # Mock MCP servers for testing
    db_server = DatabaseMCPServer("production.db")
    await db_server.initialize()
    
    crm_server = CRMMCPServer("https://api.crm.com", "prod_api_key")
    
    # Test system với different scenarios
    test_scenarios = [
        {
            "server_id": "database",
            "server": db_server,
            "tool": "query_database",
            "params": {"query": "SELECT COUNT(*) FROM customers"}
        },
        {
            "server_id": "crm",
            "server": crm_server,
            "tool": "search_contacts",
            "params": {"query": "tech", "limit": 5}
        }
    ]
    
    print("🚀 **Production MCP System Demo**\n")
    
    # Execute scenarios multiple times để test caching
    for round_num in range(3):
        print(f"📊 **Round {round_num + 1}:**")
        
        for scenario in test_scenarios:
            result = await production_system.execute_with_monitoring(
                scenario["server_id"],
                scenario["server"],
                scenario["tool"],
                scenario["params"]
            )
            
            if result["success"]:
                print(f"  ✅ {scenario['server_id']}.{scenario['tool']}: {result['execution_time']:.3f}s")
            else:
                print(f"  ❌ {scenario['server_id']}.{scenario['tool']}: {result['error']}")
        
        print()
    
    # Show system status
    status = production_system.get_system_status()
    print("📈 **System Status:**")
    print(f"  Healthy Servers: {status['healthy_servers']}/{status['total_servers']}")
    print(f"  Cache Hit Rate: {status['cache_stats']['response_cache_size']} cached responses")
    print(f"  Average Response Times: {status['avg_response_times']}")
    
    # Run maintenance
    await production_system.maintenance_routine()
    
    # Cleanup
    await db_server.close()

if __name__ == "__main__":
    from datetime import datetime
    asyncio.run(demo_production_mcp())
```

## Best Practices và Deployment Guidelines

### MCP Security Best Practices

```python
import secrets
import jwt
from typing import Dict, Any, Optional
from datetime import datetime, timedelta

class SecureMCPServer:
    """MCP Server với security enhancements"""
    
    def __init__(self, server_id: str, secret_key: str):
        self.server_id = server_id
        self.secret_key = secret_key
        self.active_tokens: Dict[str, Dict[str, Any]] = {}
    
    def generate_access_token(self, client_id: str, permissions: List[str]) -> str:
        """Generate JWT access token"""
        payload = {
            "client_id": client_id,
            "server_id": self.server_id,
            "permissions": permissions,
            "issued_at": datetime.now().timestamp(),
            "expires_at": (datetime.now() + timedelta(hours=1)).timestamp()
        }
        
        token = jwt.encode(payload, self.secret_key, algorithm="HS256")
        
        # Store active token
        self.active_tokens[token] = payload
        
        return token
    
    def validate_token(self, token: str) -> Optional[Dict[str, Any]]:
        """Validate JWT token"""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=["HS256"])
            
            # Check expiration
            if datetime.now().timestamp() > payload["expires_at"]:
                return None
            
            return payload
            
        except jwt.InvalidTokenError:
            return None
    
    def check_permission(self, token: str, required_permission: str) -> bool:
        """Check if token has required permission"""
        payload = self.validate_token(token)
        
        if not payload:
            return False
        
        return required_permission in payload.get("permissions", [])
    
    async def secure_call_tool(
        self, 
        token: str, 
        tool_name: str, 
        arguments: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Secure tool execution với authentication"""
        
        # Validate token
        if not self.validate_token(token):
            return {"error": "Invalid or expired token"}
        
        # Check permissions
        if not self.check_permission(token, f"tool:{tool_name}"):
            return {"error": f"Insufficient permissions for {tool_name}"}
        
        # Input validation
        validated_args = self._validate_inputs(tool_name, arguments)
        if "error" in validated_args:
            return validated_args
        
        # Rate limiting check
        if not self._check_rate_limit(token):
            return {"error": "Rate limit exceeded"}
        
        # Execute tool
        try:
            result = await self._execute_tool(tool_name, validated_args)
            
            # Audit log
            await self._audit_log(token, tool_name, arguments, "success")
            
            return result
            
        except Exception as e:
            await self._audit_log(token, tool_name, arguments, "error", str(e))
            return {"error": f"Tool execution failed: {str(e)}"}
    
    def _validate_inputs(self, tool_name: str, arguments: Dict[str, Any]) -> Dict[str, Any]:
        """Validate and sanitize inputs"""
        
        # Define validation rules cho từng tool
        validation_rules = {
            "query_database": {
                "query": {"type": str, "max_length": 1000, "no_sql_injection": True}
            },
            "search_contacts": {
                "query": {"type": str, "max_length": 100},
                "limit": {"type": int, "min": 1, "max": 100}
            }
        }
        
        if tool_name not in validation_rules:
            return {"error": f"Unknown tool: {tool_name}"}
        
        rules = validation_rules[tool_name]
        validated = {}
        
        for field, value in arguments.items():
            if field not in rules:
                continue  # Skip unknown fields
            
            rule = rules[field]
            
            # Type check
            if not isinstance(value, rule["type"]):
                return {"error": f"Invalid type for {field}"}
            
            # String validations
            if rule["type"] == str:
                if "max_length" in rule and len(value) > rule["max_length"]:
                    return {"error": f"{field} too long"}
                
                if rule.get("no_sql_injection") and self._detect_sql_injection(value):
                    return {"error": f"Potentially dangerous SQL in {field}"}
            
            # Integer validations
            if rule["type"] == int:
                if "min" in rule and value < rule["min"]:
                    return {"error": f"{field} below minimum"}
                if "max" in rule and value > rule["max"]:
                    return {"error": f"{field} above maximum"}
            
            validated[field] = value
        
        return validated
    
    def _detect_sql_injection(self, query: str) -> bool:
        """Simple SQL injection detection"""
        dangerous_patterns = [
            "DROP TABLE", "DELETE FROM", "INSERT INTO", "UPDATE SET",
            "UNION SELECT", "'; --", "' OR '1'='1"
        ]
        
        query_upper = query.upper()
        return any(pattern in query_upper for pattern in dangerous_patterns)
    
    def _check_rate_limit(self, token: str) -> bool:
        """Simple rate limiting"""
        # Implement rate limiting logic
        # For demo, always return True
        return True
    
    async def _execute_tool(self, tool_name: str, arguments: Dict[str, Any]) -> Dict[str, Any]:
        """Execute the actual tool"""
        # Delegate to actual tool implementation
        pass
    
    async def _audit_log(
        self, 
        token: str, 
        tool_name: str, 
        arguments: Dict[str, Any], 
        status: str, 
        error: str = None
    ):
        """Log tool execution for audit"""
        payload = self.validate_token(token)
        
        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "client_id": payload.get("client_id") if payload else "unknown",
            "server_id": self.server_id,
            "tool_name": tool_name,
            "arguments_hash": hashlib.md5(str(arguments).encode()).hexdigest(),
            "status": status,
            "error": error
        }
        
        # In production: send to logging service
        print(f"🔒 AUDIT: {log_entry}")

# Demo secure MCP
async def demo_secure_mcp():
    """Demo secure MCP implementation"""
    
    print("🔐 **Secure MCP Demo**\n")
    
    # Create secure server
    secure_server = SecureMCPServer("secure_db", "super_secret_key_123")
    
    # Generate tokens với different permissions
    admin_token = secure_server.generate_access_token(
        "admin_client", 
        ["tool:query_database", "tool:search_contacts", "tool:create_user"]
    )
    
    readonly_token = secure_server.generate_access_token(
        "readonly_client",
        ["tool:search_contacts"]
    )
    
    print(f"🎫 Admin Token: {admin_token[:50]}...")
    print(f"🎫 Readonly Token: {readonly_token[:50]}...\n")
    
    # Test different scenarios
    test_cases = [
        {
            "name": "Admin querying database",
            "token": admin_token,
            "tool": "query_database",
            "args": {"query": "SELECT COUNT(*) FROM users"}
        },
        {
            "name": "Readonly searching contacts",
            "token": readonly_token,
            "tool": "search_contacts", 
            "args": {"query": "john", "limit": 10}
        },
        {
            "name": "Readonly trying to query database (should fail)",
            "token": readonly_token,
            "tool": "query_database",
            "args": {"query": "SELECT * FROM users"}
        },
        {
            "name": "SQL injection attempt (should fail)",
            "token": admin_token,
            "tool": "query_database",
            "args": {"query": "SELECT * FROM users; DROP TABLE users;"}
        }
    ]
    
    for test in test_cases:
        print(f"🧪 **Test:** {test['name']}")
        
        result = await secure_server.secure_call_tool(
            test["token"],
            test["tool"],
            test["args"]
        )
        
        if "error" in result:
            print(f"   ❌ {result['error']}")
        else:
            print(f"   ✅ Success: {result}")
        
        print()

if __name__ == "__main__":
    import hashlib
    asyncio.run(demo_secure_mcp())
```

## Tổng Kết và Production Guidelines

### Những Điều Cần Nhớ

✅ **MCP = USB-C cho AI** - Chuẩn kết nối thống nhất cho external system integration  
✅ **Nhiều Phương Thức Transport** - STDIO, HTTP+SSE, Streamable HTTP để phù hợp mọi use case  
✅ **Lợi Ích Hệ Sinh Thái** - Servers có thể tái sử dụng, interfaces chuẩn hóa  
✅ **Sẵn Sàng Production** - Caching, monitoring, security, error handling toàn diện  
✅ **Tích Hợp Enterprise** - CRM, databases, business systems một cách seamless  

### MCP vs Custom Integration - Bảng So Sánh Chi Tiết

| Khía Cạnh | Custom Integration | MCP Integration |
|-----------|-------------------|-----------------|
| **Thời Gian Phát Triển** | Cao (mỗi app riêng biệt) | Thấp (tái sử dụng existing servers) |
| **Bảo Trì** | Phức tạp (nhiều codebases) | Đơn giản (centralized servers) |
| **Chuẩn Hóa** | Không nhất quán | Unified interface |
| **Hệ Sinh Thái** | Chia sẻ hạn chế | Rich server ecosystem |
| **Tính Bền Vững** | Rủi ro vendor lock-in | Open standard |
| **Debugging** | Khó khăn | Standardized tooling |
| **Community Support** | Hạn chế | Growing ecosystem |

### Checklist Triển Khai Production

**🔧 Thiết Lập Kỹ Thuật:**
- [ ] Lựa chọn transport method phù hợp (STDIO vs HTTP)
- [ ] Triển khai proper error handling và retries
- [ ] Setup caching cho frequently accessed data
- [ ] Cấu hình connection pooling và resource limits
- [ ] Thêm comprehensive logging và monitoring

**🔒 Bảo Mật:**
- [ ] Triển khai authentication và authorization
- [ ] Thêm input validation và sanitization
- [ ] Setup rate limiting để prevent abuse
- [ ] Enable audit logging cho compliance
- [ ] Sử dụng secure communication (HTTPS, encrypted channels)

**📊 Monitoring:**
- [ ] Track server health và response times
- [ ] Monitor error rates và failure patterns
- [ ] Setup alerts cho critical failures
- [ ] Implement performance metrics collection
- [ ] Tạo dashboards cho operational visibility

**🧪 Testing:**
- [ ] Unit tests cho individual MCP servers
- [ ] Integration tests cho end-to-end workflows
- [ ] Load testing under realistic conditions
- [ ] Security testing cho common vulnerabilities
- [ ] Disaster recovery testing

### Các Lỗi Thường Gặp và Giải Pháp

❌ **Over-Engineering** → Bắt đầu đơn giản, thêm complexity khi cần thiết  
❌ **No Caching** → Triển khai intelligent caching strategies  
❌ **Poor Error Handling** → Comprehensive error handling với retries  
❌ **Security Neglect** → Security-first approach từ đầu  
❌ **No Monitoring** → Built-in observability từ ngày đầu tiên  

### MCP Ecosystem và Tương Lai

**🌟 Hệ Sinh Thái Đang Phát Triển:**
- Filesystem, database, CRM integrations
- Cloud services (AWS, GCP, Azure) connectors  
- Developer tools (Git, IDEs, CI/CD) integrations
- Business applications (Slack, Teams, Notion) servers

**🚀 Phát Triển Tương Lai:**
- Enhanced streaming capabilities
- Better performance optimizations
- Richer metadata và discovery mechanisms
- Advanced security features
- Multi-tenant server architectures

### Kiến Trúc Production-Ready

**📈 Scalability Considerations:**
- Horizontal scaling với multiple server instances
- Load balancing cho high-availability
- Database sharding cho large datasets
- Caching layers cho performance optimization
- Auto-scaling based on demand

**🔄 Reliability Patterns:**
- Circuit breaker pattern cho external dependencies
- Retry with exponential backoff
- Graceful degradation khi services unavailable
- Health checks và monitoring
- Disaster recovery procedures

### Bước Tiếp Theo

Trong **bài tiếp theo**, chúng ta sẽ khám phá [Streaming và Real-time Responses: Tối Ưu User Experience](../openai-agents-sdk-part-08-streaming-real-time-responses/):

🎙️ **Streaming và Real-time Responses** - Tối ưu user experience với streaming  
⚡ **Performance Optimization** - Kỹ thuật nâng cao cho production systems  
🔄 **Event-driven Architectures** - Xây dựng reactive agent systems  

### Thử Thách Cho Bạn

1. **Xây dựng custom MCP server** cho domain cụ thể của bạn (e.g., project management, inventory)
2. **Triển khai production-ready features** - caching, monitoring, security
3. **Tạo multi-system integration** kết hợp filesystem + database + API servers
4. **Thiết kế enterprise workflow** sử dụng MCP để automate business processes

---

*Bài tiếp theo: [**"Streaming và Real-time Responses: Tối Ưu User Experience"**](../openai-agents-sdk-part-08-streaming-real-time-responses/) - Chúng ta sẽ học cách tạo responsive, real-time interactions với streaming responses và event-driven patterns.*