---
title: "Advanced Tools: Web Search, File Search và Computer Use"
date: 2025-05-24 09:00:00 +0700
categories: [AI, OpenAI SDK]
tags: [openai, agents, tools, web-search, file-search, computer-use]
author: toantranct
description: Hướng dẫn chi tiết về Advanced Tools trong OpenAI Agents SDK. Từ Web Search, File Search đến Computer Use - mở rộng khả năng agents với thế giới bên ngoài.
---

Trong những bài trước, chúng ta đã học cách tạo agents với function tools tự định nghĩa. Nhưng OpenAI Agents SDK còn cung cấp một bộ **hosted tools** cực kỳ mạnh mẽ - những công cụ chạy trực tiếp trên infrastructure của OpenAI, cho phép agents tương tác với internet, files, và thậm chí là operating systems.

Hôm nay chúng ta sẽ khám phá những **siêu năng lực** này: từ việc search thông tin real-time trên web, đến phân tích documents phức tạp, và thậm chí là automation toàn bộ computer workflows. Đây chính là những tools biến agents từ "chatbots thông minh" thành "digital workers" thực sự.

## Tổng Quan Advanced Tools

### Hosted Tools vs Function Tools

**🔧 Function Tools (Custom):**
- Chạy trên infrastructure của bạn
- Tự định nghĩa logic và capabilities
- Hoàn toàn kiểm soát được
- Performance phụ thuộc vào system của bạn

**☁️ Hosted Tools (OpenAI):**  
- Chạy trên OpenAI servers
- Pre-built với optimized performance
- Integrated với LLM models
- Maintained và updated bởi OpenAI

### Danh Sách Hosted Tools

🌐 **WebSearchTool** - Search real-time information trên internet  
📁 **FileSearchTool** - Retrieve từ OpenAI Vector Stores  
💻 **ComputerTool** - Automate computer use tasks  
🐍 **CodeInterpreterTool** - Execute code trong sandboxed environment  
🖼️ **ImageGenerationTool** - Generate images từ prompts  
🌐 **HostedMCPTool** - Access remote MCP servers  
⚡ **LocalShellTool** - Run shell commands locally  

## Web Search Tool - Kết Nối Với Thế Giới

### Cách Hoạt Động

WebSearchTool cho phép agents search và retrieve real-time information từ internet. Tool này đặc biệt hữu ích cho:
- Current events và breaking news
- Real-time data (stock prices, weather, sports scores)
- Product information và reviews
- Research với up-to-date sources

### Basic Web Search

```python
from agents import Agent, Runner, WebSearchTool

# Tạo agent với web search capability
research_agent = Agent(
    name="Research Assistant",
    instructions="""
    You are a research assistant with access to real-time web information.
    
    Your capabilities:
    - Search for current information on any topic
    - Find multiple sources and perspectives
    - Synthesize information from various sources
    - Provide citations and source links
    
    Research approach:
    1. Search for relevant, recent information
    2. Cross-reference multiple sources
    3. Present balanced, well-sourced findings
    4. Include source URLs for verification
    
    Always mention when information is current as of your search date.
    """,
    tools=[WebSearchTool()]
)

# Demo basic web search
async def demo_web_search():
    queries = [
        "What are the latest developments in AI technology in 2025?",
        "Current stock price of Tesla and recent news about the company",
        "Best restaurants in Hanoi, Vietnam with recent reviews"
    ]
    
    for query in queries:
        print(f"**Research Query:** {query}")
        result = await Runner.run(research_agent, query)
        print(f"**Findings:** {result.final_output[:500]}...")
        print("-" * 70 + "\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_web_search())
```

### Advanced Research Agent

```python
from typing import List, Dict, Any
from pydantic import BaseModel

class ResearchTopic(BaseModel):
    topic: str
    subtopics: List[str]
    search_keywords: List[str]
    required_sources: int

class ResearchFindings(BaseModel):
    topic: str
    key_points: List[str]
    sources: List[Dict[str, str]]
    confidence_level: str
    last_updated: str

# Comprehensive research system
class ComprehensiveResearcher:
    def __init__(self):
        self.search_agent = Agent(
            name="Web Search Specialist",
            instructions="""
            You are a web search specialist focused on finding high-quality, relevant information.
            
            Search Strategy:
            - Use specific, targeted search queries
            - Look for authoritative sources (news sites, academic papers, official websites)
            - Search for recent information (within last 6 months when possible)
            - Find diverse perspectives on controversial topics
            
            Source Quality Guidelines:
            - Prefer primary sources over secondary
            - Academic and institutional sources have high credibility
            - Recent news from reputable outlets
            - Official company/government websites for factual data
            
            When searching:
            1. Start with broad queries, then get specific
            2. Look for contradicting information to ensure balance
            3. Note the publication date and source credibility
            4. Search for expert opinions and analysis
            """,
            tools=[WebSearchTool()]
        )
        
        self.synthesis_agent = Agent(
            name="Research Synthesizer",
            instructions="""
            You synthesize research findings into comprehensive, balanced reports.
            
            Synthesis Process:
            1. Identify key themes and patterns across sources
            2. Note areas of consensus and disagreement
            3. Evaluate source credibility and recency
            4. Present findings in logical, structured format
            5. Highlight gaps or areas needing more research
            
            Report Structure:
            - Executive Summary (2-3 sentences)
            - Key Findings (bullet points with sources)
            - Different Perspectives (if applicable)
            - Source Assessment (credibility notes)
            - Research Limitations
            - Recommendations for further investigation
            """,
            output_type=ResearchFindings
        )
    
    async def conduct_research(self, topic: str, depth: str = "standard") -> Dict[str, Any]:
        """Conduct comprehensive research on a topic"""
        
        print(f"🔍 Starting research on: {topic}")
        
        # Phase 1: Initial broad search
        broad_search_query = f"""
        Research this topic comprehensively: {topic}
        
        Search for:
        1. General overview and background
        2. Recent developments and news
        3. Expert opinions and analysis
        4. Statistical data and trends
        
        Depth level: {depth}
        Find at least 5-7 diverse, credible sources.
        """
        
        initial_results = await Runner.run(self.search_agent, broad_search_query)
        
        # Phase 2: Targeted follow-up searches based on initial findings
        if depth in ["deep", "comprehensive"]:
            follow_up_query = f"""
            Based on the initial research about {topic}, conduct targeted follow-up searches on:
            
            Previous findings: {initial_results.final_output[:1000]}
            
            Focus on:
            1. Any controversial or debated aspects mentioned
            2. Recent developments or breaking news
            3. Expert criticisms or alternative viewpoints  
            4. Quantitative data and statistics
            
            Find additional authoritative sources to provide complete picture.
            """
            
            detailed_results = await Runner.run(self.search_agent, follow_up_query)
            
            # Combine results
            combined_research = f"""
            Initial Research:
            {initial_results.final_output}
            
            Detailed Follow-up:
            {detailed_results.final_output}
            """
        else:
            combined_research = initial_results.final_output
        
        # Phase 3: Synthesize all findings
        synthesis_query = f"""
        Synthesize this research about {topic} into a comprehensive report:
        
        Research Data:
        {combined_research}
        
        Create a balanced, well-structured analysis with proper source attribution.
        """
        
        final_report = await Runner.run(self.synthesis_agent, synthesis_query)
        
        return {
            "topic": topic,
            "research_depth": depth,
            "raw_research": combined_research,
            "synthesized_report": final_report.final_output,
            "research_timestamp": datetime.now().isoformat()
        }

# Demo comprehensive research
async def demo_comprehensive_research():
    researcher = ComprehensiveResearcher()
    
    # Research complex topics
    topics = [
        ("Impact of AI on software development jobs", "deep"),
        ("Electric vehicle adoption trends in Southeast Asia", "standard"),
        ("Cryptocurrency regulation updates 2025", "comprehensive")
    ]
    
    for topic, depth in topics:
        print(f"📚 **Researching:** {topic} (Depth: {depth})")
        
        research_result = await researcher.conduct_research(topic, depth)
        
        print(f"**Synthesized Report:**")
        print(research_result["synthesized_report"])
        print("=" * 80 + "\n")

if __name__ == "__main__":
    from datetime import datetime
    import asyncio
    asyncio.run(demo_comprehensive_research())
```

## File Search Tool - Truy Cập Tri Thức Riêng

### Vector Store Integration

FileSearchTool cho phép agents search thông tin từ OpenAI Vector Stores - nơi bạn có thể upload và index documents riêng của mình.

```python
from agents import Agent, Runner, FileSearchTool

# Tải documents lên Vector Store trước (qua OpenAI API hoặc dashboard)
# Giả sử bạn đã có vector_store_id từ việc upload docs

knowledge_agent = Agent(
    name="Company Knowledge Assistant",
    instructions="""
    You are an intelligent assistant with access to our company's knowledge base.
    
    Knowledge Sources:
    - Company policies and procedures
    - Technical documentation  
    - Product specifications
    - Training materials
    - Meeting notes and decisions
    
    Your approach:
    1. Search relevant documents for user questions
    2. Provide accurate, cited information
    3. If information isn't in knowledge base, say so clearly
    4. Reference specific documents and sections when possible
    
    Always distinguish between:
    - Information from company documents (cite sources)
    - General knowledge (mention it's general info)
    - Information you're uncertain about (be honest)
    """,
    tools=[
        FileSearchTool(
            max_num_results=5,  # Return top 5 most relevant results
            vector_store_ids=["vs-abc123xyz"]  # Your vector store ID
        )
    ]
)

# Demo knowledge base queries
async def demo_knowledge_base():
    company_queries = [
        "What is our remote work policy?",
        "How do we handle customer data privacy?", 
        "What are the requirements for the new project approval process?",
        "Can you find information about our API rate limiting?"
    ]
    
    for query in company_queries:
        print(f"**Knowledge Query:** {query}")
        result = await Runner.run(knowledge_agent, query)
        print(f"**Response:** {result.final_output}")
        print("-" * 60 + "\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_knowledge_base())
```

### Advanced Document Analysis

```python
from agents import Agent, Runner, FileSearchTool, function_tool
from typing import Dict, List

class DocumentAnalysisSystem:
    def __init__(self, vector_store_ids: List[str]):
        self.document_retriever = Agent(
            name="Document Retriever",
            instructions="""
            You specialize in finding and retrieving relevant documents from our knowledge base.
            
            Search Strategy:
            - Use multiple search approaches (broad and specific keywords)
            - Look for documents from different perspectives
            - Find both policy documents and practical examples
            - Retrieve background/context documents when needed
            
            When searching:
            1. Start with user's exact query
            2. Try related keywords and synonyms
            3. Look for process documents, examples, templates
            4. Find any related policies or guidelines
            """,
            tools=[
                FileSearchTool(
                    max_num_results=10,
                    vector_store_ids=vector_store_ids
                )
            ]
        )
        
        self.document_analyzer = Agent(
            name="Document Analyzer",
            instructions="""
            You analyze retrieved documents to answer user questions comprehensively.
            
            Analysis Process:
            1. Review all retrieved documents for relevance
            2. Extract key information that answers the user's question
            3. Identify any conflicting information between documents
            4. Note if information seems outdated or incomplete
            5. Provide clear, actionable answers with document citations
            
            Citation Format:
            - Reference specific document names/sections
            - Note document dates when available
            - Highlight any uncertainties or conflicts
            - Suggest who to contact for clarification if needed
            
            If documents don't contain enough information:
            - Clearly state what's missing
            - Suggest where to find additional information
            - Recommend next steps for the user
            """
        )
    
    async def analyze_query(self, user_query: str) -> Dict[str, Any]:
        """Comprehensive document analysis for user queries"""
        
        print(f"📄 Analyzing query: {user_query}")
        
        # Step 1: Retrieve relevant documents
        retrieval_query = f"""
        Find all relevant documents for this query: {user_query}
        
        Search thoroughly using different keyword approaches:
        - Direct keywords from the query
        - Related terms and synonyms  
        - Process/procedure documents
        - Policy and guideline documents
        - Examples or templates if applicable
        
        Cast a wide net to ensure comprehensive coverage.
        """
        
        retrieval_result = await Runner.run(self.document_retriever, retrieval_query)
        
        # Step 2: Analyze retrieved documents
        analysis_query = f"""
        User Question: {user_query}
        
        Retrieved Documents and Information:
        {retrieval_result.final_output}
        
        Analyze these documents to provide a comprehensive answer:
        1. Direct answer to the user's question
        2. Relevant details and context
        3. Any important caveats or exceptions
        4. Next steps or action items if applicable
        5. Document sources and citations
        
        If the documents don't fully answer the question, clearly explain what's missing.
        """
        
        analysis_result = await Runner.run(self.document_analyzer, analysis_query)
        
        return {
            "query": user_query,
            "retrieved_documents": retrieval_result.final_output,
            "comprehensive_answer": analysis_result.final_output,
            "timestamp": datetime.now().isoformat()
        }

# Demo advanced document analysis
async def demo_document_analysis():
    # Initialize with your vector store IDs
    doc_system = DocumentAnalysisSystem(["vs-policies", "vs-procedures", "vs-technical"])
    
    complex_queries = [
        "What's the complete process for onboarding a new remote employee?",
        "How should we handle a data breach incident from start to finish?",
        "What are all the approvals needed to launch a new product feature?",
        "Can you explain our customer support escalation procedures?"
    ]
    
    for query in complex_queries:
        result = await doc_system.analyze_query(query)
        
        print(f"**Query:** {result['query']}")
        print(f"**Comprehensive Answer:**")
        print(result['comprehensive_answer'])
        print("=" * 80 + "\n")

if __name__ == "__main__":
    from datetime import datetime
    import asyncio
    asyncio.run(demo_document_analysis())
```

## Computer Tool - Automation Thực Sự

### Desktop Automation

ComputerTool cho phép agents tương tác trực tiếp với desktop environment - click, type, screenshot, và automation workflows.

```python
from agents import Agent, Runner, ComputerTool

# Desktop automation agent
automation_agent = Agent(
    name="Desktop Automation Assistant",
    instructions="""
    You are a desktop automation specialist capable of controlling computers.
    
    Capabilities:
    - Take screenshots to see current state
    - Click on UI elements (buttons, links, menus)
    - Type text into fields and applications
    - Navigate through applications and websites
    - Perform multi-step workflows
    
    Automation Approach:
    1. Take screenshot to understand current state
    2. Plan the sequence of actions needed
    3. Execute actions step by step
    4. Verify results with screenshots
    5. Handle errors and unexpected states gracefully
    
    Safety Guidelines:
    - Always confirm before destructive actions
    - Take screenshots before and after important actions
    - Be careful with sensitive information
    - Stop if anything looks unexpected
    """,
    tools=[ComputerTool()]
)

# Demo automation workflows
async def demo_desktop_automation():
    automation_tasks = [
        "Take a screenshot of the current desktop",
        "Open a web browser and navigate to google.com",
        "Create a new text document and write 'Hello from AI automation'",
        "Take a screenshot showing the completed task"
    ]
    
    for task in automation_tasks:
        print(f"**Automation Task:** {task}")
        try:
            result = await Runner.run(automation_agent, task)
            print(f"**Result:** {result.final_output}")
        except Exception as e:
            print(f"**Error:** {str(e)}")
        print("-" * 60 + "\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_desktop_automation())
```

### Advanced Workflow Automation

```python
from pydantic import BaseModel
from typing import List, Dict, Any

class AutomationStep(BaseModel):
    action: str
    description: str
    expected_result: str
    screenshot_needed: bool = True

class WorkflowPlan(BaseModel):
    workflow_name: str
    steps: List[AutomationStep]
    estimated_duration: str
    risk_level: str

class WorkflowAutomationSystem:
    def __init__(self):
        self.planner_agent = Agent(
            name="Workflow Planner",
            instructions="""
            You are a workflow automation planner that breaks down complex tasks into detailed steps.
            
            Planning Process:
            1. Understand the overall goal
            2. Identify all required applications/websites
            3. Break down into specific, actionable steps
            4. Consider potential error states and recovery
            5. Estimate time and risk level
            
            Step Details Should Include:
            - Specific UI elements to interact with
            - Text to type or data to enter
            - Verification steps to confirm success
            - Screenshots at key decision points
            
            Risk Assessment:
            - LOW: Simple tasks, no data modification
            - MEDIUM: Multiple applications, some data entry
            - HIGH: System settings, file operations, sensitive data
            """,
            output_type=WorkflowPlan
        )
        
        self.executor_agent = Agent(
            name="Workflow Executor",
            instructions="""
            You execute planned automation workflows step by step.
            
            Execution Principles:
            1. Follow the plan exactly unless problems arise
            2. Take screenshots at each major step
            3. Verify each action succeeded before continuing
            4. Pause and report if unexpected issues occur
            5. Provide detailed feedback on each step
            
            Error Handling:
            - If UI element not found, take screenshot and describe what you see
            - If action fails, try alternative approaches
            - If critical error, stop and report immediately
            - Always explain what went wrong and suggest solutions
            
            Safety Checks:
            - Confirm before any destructive actions
            - Validate data before entering sensitive information
            - Take screenshot evidence of completion
            """,
            tools=[ComputerTool()]
        )
    
    async def automate_workflow(self, workflow_description: str) -> Dict[str, Any]:
        """Plan and execute complete workflow automation"""
        
        print(f"🤖 Planning workflow: {workflow_description}")
        
        # Phase 1: Create detailed plan
        planning_query = f"""
        Create a detailed automation plan for this workflow:
        
        Task: {workflow_description}
        
        Consider:
        - What applications/websites are needed?
        - What are the specific steps to complete this task?
        - Where might errors occur and how to handle them?
        - What screenshots are needed for verification?
        
        Provide a comprehensive, step-by-step plan.
        """
        
        plan_result = await Runner.run(self.planner_agent, planning_query)
        workflow_plan = plan_result.final_output
        
        print(f"📋 Plan created: {workflow_plan.workflow_name}")
        print(f"   Steps: {len(workflow_plan.steps)}")
        print(f"   Risk Level: {workflow_plan.risk_level}")
        
        # Phase 2: Execute the workflow
        execution_query = f"""
        Execute this automation workflow:
        
        Workflow Plan:
        {workflow_plan}
        
        Execute each step carefully:
        1. Take initial screenshot to see current state
        2. Follow each step in the plan
        3. Verify success after each step
        4. Take final screenshot when complete
        5. Report on overall success
        
        Stop immediately if any critical errors occur.
        """
        
        print(f"⚡ Executing workflow...")
        execution_result = await Runner.run(self.executor_agent, execution_query)
        
        return {
            "workflow_description": workflow_description,
            "plan": workflow_plan,
            "execution_result": execution_result.final_output,
            "success": "completed successfully" in execution_result.final_output.lower(),
            "timestamp": datetime.now().isoformat()
        }

# Demo advanced workflow automation
async def demo_workflow_automation():
    automation_system = WorkflowAutomationSystem()
    
    workflows = [
        "Create a PowerPoint presentation with 3 slides about 'AI in Business'",
        "Set up a new email account and send a test email to myself", 
        "Download a file from the internet and organize it in a new folder",
        "Take a screenshot of the current screen and save it to Desktop"
    ]
    
    for workflow in workflows:
        print(f"🚀 **Workflow:** {workflow}")
        
        try:
            result = await automation_system.automate_workflow(workflow)
            
            print(f"**Planning:** {result['plan'].workflow_name}")
            print(f"**Steps:** {len(result['plan'].steps)}")
            print(f"**Execution:** {'Success' if result['success'] else 'Failed'}")
            print(f"**Details:** {result['execution_result'][:200]}...")
            
        except Exception as e:
            print(f"**Error:** {str(e)}")
        
        print("=" * 80 + "\n")

if __name__ == "__main__":
    from datetime import datetime
    import asyncio
    asyncio.run(demo_workflow_automation())
```

## Code Interpreter Tool - Xử Lý Dữ Liệu Thông Minh

### Data Analysis và Visualization

```python
from agents import Agent, Runner, CodeInterpreterTool

# Data analysis agent
data_scientist_agent = Agent(
    name="Data Science Assistant",
    instructions="""
    You are a data scientist with the ability to execute Python code for analysis.
    
    Capabilities:
    - Data loading, cleaning, and preprocessing
    - Statistical analysis and hypothesis testing
    - Data visualization with matplotlib, seaborn, plotly
    - Machine learning with scikit-learn, pandas
    - Generate insights and recommendations
    
    Analysis Approach:
    1. Understand the data and objectives
    2. Explore and clean the data
    3. Perform relevant statistical analysis
    4. Create meaningful visualizations
    5. Draw conclusions and provide recommendations
    
    Code Standards:
    - Write clean, well-commented code
    - Use appropriate libraries for each task
    - Create informative plots with proper labels
    - Handle errors and edge cases gracefully
    - Explain your methodology and findings
    """,
    tools=[CodeInterpreterTool()]
)

# Demo data analysis
async def demo_data_analysis():
    analysis_requests = [
        """
        Create a dataset of 1000 random sales records with columns: date, product, sales_amount, region.
        Then analyze:
        1. Sales trends over time
        2. Performance by region
        3. Top-selling products
        4. Create visualizations for each analysis
        """,
        
        """
        Generate sample customer data (age, income, spending_score) and:
        1. Perform customer segmentation using K-means clustering
        2. Visualize the segments
        3. Analyze characteristics of each segment
        4. Provide business recommendations
        """,
        
        """
        Create a simple linear regression model to predict house prices based on size and location.
        Include:
        1. Generate synthetic real estate data
        2. Build and train the model
        3. Evaluate model performance
        4. Visualize predictions vs actual values
        """
    ]
    
    for i, request in enumerate(analysis_requests, 1):
        print(f"**Data Analysis Task {i}:**")
        print(request.strip())
        
        result = await Runner.run(data_scientist_agent, request)
        print(f"**Analysis Results:**")
        print(result.final_output[:500] + "..." if len(result.final_output) > 500 else result.final_output)
        print("=" * 80 + "\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_data_analysis())
```

## Multi-Tool Integration - Sức Mạnh Kết Hợp

### Research + Analysis Workflow

```python
from agents import Agent, Runner, WebSearchTool, CodeInterpreterTool, FileSearchTool

class IntegratedResearchSystem:
    def __init__(self, vector_store_ids: List[str] = None):
        self.research_agent = Agent(
            name="Integrated Research Analyst",
            instructions="""
            You are an advanced research analyst with access to multiple powerful tools.
            
            Available Tools:
            - Web search for current information and data
            - Code interpreter for data analysis and visualization  
            - File search for internal knowledge base (if configured)
            
            Research Methodology:
            1. Define research scope and objectives
            2. Gather data from multiple sources
            3. Analyze and process data programmatically
            4. Create visualizations and insights
            5. Synthesize findings into actionable recommendations
            
            Multi-tool Approach:
            - Use web search for current data and trends
            - Use code interpreter to analyze numerical data
            - Use file search for company-specific context
            - Combine insights from all sources
            
            Always provide:
            - Data sources and methodology
            - Visual representations of key findings
            - Confidence levels and limitations
            - Actionable recommendations
            """,
            tools=[
                WebSearchTool(),
                CodeInterpreterTool(),
                FileSearchTool(vector_store_ids=vector_store_ids or [])
            ] if vector_store_ids else [
                WebSearchTool(),
                CodeInterpreterTool()
            ]
        )
    
    async def comprehensive_research(self, research_topic: str) -> Dict[str, Any]:
        """Conduct comprehensive research using multiple tools"""
        
        research_query = f"""
        Conduct comprehensive research and analysis on: {research_topic}
        
        Research Process:
        1. INFORMATION GATHERING:
           - Search for current data, trends, and expert opinions
           - Find relevant statistics and quantitative data
           - Look for different perspectives and analyses
        
        2. DATA ANALYSIS:
           - If you find numerical data, analyze it programmatically
           - Create visualizations to illustrate key trends
           - Perform statistical analysis where appropriate
           - Calculate relevant metrics and comparisons
        
        3. SYNTHESIS:
           - Combine web research with data analysis
           - Identify key insights and patterns
           - Note any contradictions or uncertainties
           - Provide balanced, evidence-based conclusions
        
        4. RECOMMENDATIONS:
           - Based on your analysis, provide actionable insights
           - Include confidence levels for your findings
           - Suggest areas for further investigation
        
        Please be thorough and use all available tools to provide the most comprehensive analysis possible.
        """
        
        print(f"🔬 Starting comprehensive research on: {research_topic}")
        
        result = await Runner.run(self.research_agent, research_query)
        
        return {
            "topic": research_topic,
            "comprehensive_analysis": result.final_output,
            "timestamp": datetime.now().isoformat(),
            "tools_used": ["WebSearch", "CodeInterpreter", "FileSearch"]
        }

# Demo integrated research
async def demo_integrated_research():
    research_system = IntegratedResearchSystem()
    
    research_topics = [
        "Global electric vehicle market growth trends and forecasts for 2025-2030",
        "Impact of remote work on commercial real estate prices in major cities",
        "Cryptocurrency adoption rates and regulatory changes affecting market performance"
    ]
    
    for topic in research_topics:
        print(f"📊 **Comprehensive Research:** {topic}")
        
        result = await research_system.comprehensive_research(topic)
        
        print(f"**Analysis:**")
        print(result['comprehensive_analysis'][:800] + "..." if len(result['comprehensive_analysis']) > 800 else result['comprehensive_analysis'])
        print("\n" + "=" * 100 + "\n")

if __name__ == "__main__":
    from datetime import datetime
    import asyncio
    asyncio.run(demo_integrated_research())
```

### Complete Business Intelligence System

```python
class BusinessIntelligenceSystem:
    def __init__(self):
        self.bi_agent = Agent(
            name="Business Intelligence Analyst",
            instructions="""
            You are a senior business intelligence analyst with comprehensive analytical capabilities.
            
            Your expertise:
            - Market research and competitive analysis
            - Financial data analysis and forecasting
            - Customer behavior and segmentation analysis
            - Operational efficiency optimization
            - Strategic planning and recommendations
            
            BI Methodology:
            1. DISCOVERY: Understand business context and objectives
            2. DATA GATHERING: Collect relevant internal and external data
            3. ANALYSIS: Apply statistical and analytical techniques
            4. VISUALIZATION: Create clear, actionable dashboards
            5. INSIGHTS: Generate strategic recommendations
            6. REPORTING: Present findings in executive-ready format
            
            Tool Usage Strategy:
            - Web search: Market trends, competitor info, industry data
            - Code interpreter: Statistical analysis, modeling, visualization
            - File search: Internal reports, historical data, policies
            
            Always provide executive-level insights with supporting data.
            """,
            tools=[
                WebSearchTool(),
                CodeInterpreterTool(),
                # FileSearchTool() would be added with actual vector stores
            ]
        )
    
    async def business_analysis(self, business_question: str, context: str = "") -> Dict[str, Any]:
        """Comprehensive business intelligence analysis"""
        
        analysis_query = f"""
        Conduct comprehensive business intelligence analysis:
        
        BUSINESS QUESTION: {business_question}
        
        CONTEXT: {context}
        
        ANALYSIS FRAMEWORK:
        
        1. MARKET INTELLIGENCE:
           - Research current market conditions and trends
           - Identify key competitors and their strategies
           - Analyze industry growth patterns and forecasts
           - Find relevant market size and opportunity data
        
        2. DATA-DRIVEN ANALYSIS:
           - If market data is available, perform quantitative analysis
           - Create trend visualizations and projections
           - Calculate key business metrics and ratios
           - Build simple forecasting models if appropriate
        
        3. STRATEGIC INSIGHTS:
           - Synthesize findings into actionable business insights
           - Identify opportunities and threats
           - Assess competitive positioning
           - Evaluate potential strategic options
        
        4. EXECUTIVE SUMMARY:
           - Present key findings in executive-ready format
           - Include specific recommendations with rationale
           - Provide confidence levels and risk assessments
           - Suggest next steps and metrics to monitor
        
        Focus on delivering practical, actionable intelligence that drives business decisions.
        """
        
        print(f"💼 Analyzing business question: {business_question}")
        
        result = await Runner.run(self.bi_agent, analysis_query)
        
        return {
            "business_question": business_question,
            "context": context,
            "bi_analysis": result.final_output,
            "analysis_date": datetime.now().isoformat()
        }

# Demo business intelligence
async def demo_business_intelligence():
    bi_system = BusinessIntelligenceSystem()
    
    business_scenarios = [
        {
            "question": "Should our SaaS company expand into the Southeast Asian market?",
            "context": "We currently serve US and European customers with project management software"
        },
        {
            "question": "How should we price our new AI-powered analytics feature?",
            "context": "Existing product is $50/month, new feature adds advanced ML capabilities"
        },
        {
            "question": "What's the potential ROI of implementing remote work policies permanently?",
            "context": "Mid-size company with 200 employees, currently hybrid model"
        }
    ]
    
    for scenario in business_scenarios:
        print(f"🎯 **Business Question:** {scenario['question']}")
        print(f"**Context:** {scenario['context']}")
        
        analysis = await bi_system.business_analysis(
            scenario['question'], 
            scenario['context']
        )
        
        print(f"**BI Analysis:**")
        print(analysis['bi_analysis'][:1000] + "..." if len(analysis['bi_analysis']) > 1000 else analysis['bi_analysis'])
        print("\n" + "=" * 120 + "\n")

if __name__ == "__main__":
    from datetime import datetime
    import asyncio
    asyncio.run(demo_business_intelligence())
```

## Production Best Practices

### Tool Performance Optimization

```python
import time
from typing import Dict, Any
import asyncio

class OptimizedToolUsage:
    def __init__(self):
        self.tool_performance_cache = {}
        self.tool_usage_stats = {
            "web_search": {"count": 0, "avg_time": 0, "errors": 0},
            "file_search": {"count": 0, "avg_time": 0, "errors": 0},
            "code_interpreter": {"count": 0, "avg_time": 0, "errors": 0},
            "computer_tool": {"count": 0, "avg_time": 0, "errors": 0}
        }
    
    async def optimized_tool_execution(self, agent: Agent, query: str, tool_hints: List[str] = None) -> Dict[str, Any]:
        """Execute agent with optimized tool usage"""
        
        start_time = time.time()
        
        # Add performance guidance to query
        if tool_hints:
            performance_guidance = f"""
            PERFORMANCE OPTIMIZATION HINTS:
            {'; '.join(tool_hints)}
            
            USER QUERY: {query}
            """
        else:
            performance_guidance = query
        
        try:
            result = await Runner.run(agent, performance_guidance)
            execution_time = time.time() - start_time
            
            # Update performance stats
            self._update_performance_stats("successful_execution", execution_time)
            
            return {
                "result": result.final_output,
                "execution_time": execution_time,
                "success": True,
                "performance_notes": self._get_performance_recommendations(execution_time)
            }
            
        except Exception as e:
            execution_time = time.time() - start_time
            self._update_performance_stats("failed_execution", execution_time)
            
            return {
                "result": f"Error: {str(e)}",
                "execution_time": execution_time,
                "success": False,
                "error": str(e)
            }
    
    def _update_performance_stats(self, operation: str, duration: float):
        """Update performance statistics"""
        # Implementation for tracking performance metrics
        pass
    
    def _get_performance_recommendations(self, execution_time: float) -> List[str]:
        """Provide performance optimization recommendations"""
        recommendations = []
        
        if execution_time > 30:
            recommendations.append("Consider breaking complex queries into smaller parts")
        
        if execution_time > 60:
            recommendations.append("Long execution time - consider using faster tools first")
        
        return recommendations

# Performance-optimized agents
class PerformanceOptimizedAgents:
    def __init__(self):
        self.quick_research_agent = Agent(
            name="Quick Research Agent",
            instructions="""
            You provide fast, efficient research using web search.
            
            Speed Optimization:
            - Use specific, targeted search queries
            - Focus on authoritative sources (Wikipedia, official sites, major news)
            - Limit search to 2-3 queries maximum
            - Provide concise but informative summaries
            
            Prioritize speed while maintaining accuracy.
            """,
            tools=[WebSearchTool()]
        )
        
        self.efficient_analyst = Agent(
            name="Efficient Data Analyst", 
            instructions="""
            You perform rapid data analysis with focused scope.
            
            Efficiency Guidelines:
            - Use simple, fast analytical methods first
            - Create essential visualizations only
            - Focus on key metrics and trends
            - Avoid complex modeling unless specifically requested
            - Provide quick insights with supporting data
            
            Balance speed with analytical rigor.
            """,
            tools=[CodeInterpreterTool()]
        )

# Demo performance optimization
async def demo_performance_optimization():
    optimizer = OptimizedToolUsage()
    agents = PerformanceOptimizedAgents()
    
    # Performance-sensitive queries
    quick_queries = [
        ("Current Bitcoin price and today's trend", ["use_recent_data", "limit_search_scope"]),
        ("Quick analysis of tech stock performance this week", ["focus_on_major_stocks", "simple_visualization"]),
        ("Fast summary of latest AI news", ["limit_to_3_sources", "prioritize_major_outlets"])
    ]
    
    for query, hints in quick_queries:
        print(f"⚡ **Quick Query:** {query}")
        print(f"**Hints:** {', '.join(hints)}")
        
        # Test with optimized research agent
        result = await optimizer.optimized_tool_execution(
            agents.quick_research_agent,
            query,
            hints
        )
        
        print(f"**Time:** {result['execution_time']:.2f}s")
        print(f"**Success:** {result['success']}")
        print(f"**Result:** {result['result'][:300]}...")
        
        if result.get('performance_notes'):
            print(f"**Performance Notes:** {', '.join(result['performance_notes'])}")
        
        print("-" * 70 + "\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_performance_optimization())
```

### Error Handling và Resilience

```python
class ResilientToolSystem:
    def __init__(self):
        self.retry_config = {
            "max_retries": 3,
            "backoff_factor": 2,
            "timeout": 30
        }
        
    async def resilient_tool_execution(self, agent: Agent, query: str) -> Dict[str, Any]:
        """Execute tools with comprehensive error handling"""
        
        for attempt in range(self.retry_config["max_retries"]):
            try:
                # Add timeout to prevent hanging
                result = await asyncio.wait_for(
                    Runner.run(agent, query),
                    timeout=self.retry_config["timeout"]
                )
                
                return {
                    "success": True,
                    "result": result.final_output,
                    "attempts": attempt + 1,
                    "warnings": []
                }
                
            except asyncio.TimeoutError:
                if attempt < self.retry_config["max_retries"] - 1:
                    wait_time = self.retry_config["backoff_factor"] ** attempt
                    print(f"⏰ Timeout on attempt {attempt + 1}, retrying in {wait_time}s...")
                    await asyncio.sleep(wait_time)
                    continue
                else:
                    return {
                        "success": False,
                        "error": "Operation timed out after multiple attempts",
                        "attempts": attempt + 1
                    }
                    
            except Exception as e:
                error_type = type(e).__name__
                
                # Handle specific error types
                if "rate limit" in str(e).lower():
                    if attempt < self.retry_config["max_retries"] - 1:
                        wait_time = self.retry_config["backoff_factor"] ** (attempt + 2)  # Longer wait for rate limits
                        print(f"🚫 Rate limit hit, waiting {wait_time}s...")
                        await asyncio.sleep(wait_time)
                        continue
                
                if attempt < self.retry_config["max_retries"] - 1:
                    wait_time = self.retry_config["backoff_factor"] ** attempt
                    print(f"❌ Error on attempt {attempt + 1}: {error_type}, retrying in {wait_time}s...")
                    await asyncio.sleep(wait_time)
                    continue
                else:
                    return {
                        "success": False,
                        "error": f"{error_type}: {str(e)}",
                        "attempts": attempt + 1
                    }
        
        return {
            "success": False,
            "error": "Max retries exceeded",
            "attempts": self.retry_config["max_retries"]
        }

# Demo resilient execution
async def demo_resilient_tools():
    resilient_system = ResilientToolSystem()
    
    # Create agent that might encounter various issues
    test_agent = Agent(
        name="Test Agent",
        instructions="You are a test agent that uses various tools.",
        tools=[WebSearchTool(), CodeInterpreterTool()]
    )
    
    test_queries = [
        "Search for current weather in New York",
        "Calculate the factorial of 100 and create a simple plot",
        "Find the latest news about artificial intelligence"
    ]
    
    for query in test_queries:
        print(f"🧪 **Testing:** {query}")
        
        result = await resilient_system.resilient_tool_execution(test_agent, query)
        
        if result["success"]:
            print(f"✅ **Success** (attempts: {result['attempts']})")
            print(f"**Result:** {result['result'][:200]}...")
        else:
            print(f"❌ **Failed** after {result['attempts']} attempts")
            print(f"**Error:** {result['error']}")
        
        print("-" * 60 + "\n")

if __name__ == "__main__":
    import asyncio
    asyncio.run(demo_resilient_tools())
```

## Tổng Kết và Best Practices

### Những Điều Cần Nhớ

✅ **Sức Mạnh Hosted Tools** - WebSearch, FileSearch, ComputerTool mở rộng khả năng agents một cách đáng kể  
✅ **Tích Hợp Multi-Tool** - Kết hợp nhiều tools để tạo ra workflows toàn diện và mạnh mẽ  
✅ **Ứng Dụng Thực Tế** - Research, automation, data analysis, business intelligence  
✅ **Tối Ưu Hiệu Suất** - Caching, parallel execution, intelligent tool selection  
✅ **Tính Ổn Định Production** - Error handling, retries, monitoring, graceful degradation  

### So Sánh Hosted Tools vs Function Tools

| Đặc Điểm | Function Tools (Custom) | Hosted Tools (OpenAI) |
|----------|------------------------|----------------------|
| **Nơi Chạy** | Infrastructure của bạn | OpenAI servers |
| **Kiểm Soát** | Hoàn toàn kiểm soát | Giới hạn theo OpenAI |
| **Performance** | Phụ thuộc hệ thống của bạn | Được tối ưu bởi OpenAI |
| **Bảo Trì** | Tự bảo trì và cập nhật | OpenAI maintained |
| **Tính Năng** | Tự định nghĩa | Pre-built, feature-rich |

### Hướng Dẫn Triển Khai Production

**🚀 Chiến Lược Lựa Chọn Tools:**
- **WebSearchTool**: Thông tin real-time, dữ liệu thị trường, tin tức mới nhất
- **FileSearchTool**: Kiến thức nội bộ, tài liệu công ty, policies và procedures  
- **CodeInterpreterTool**: Phân tích dữ liệu, tính toán phức tạp, visualizations
- **ComputerTool**: Desktop automation, UI testing, system integration

**⚡ Best Practices Hiệu Suất:**
- Cache dữ liệu được truy cập thường xuyên để giảm API calls
- Sử dụng parallel tool execution khi có thể để tăng tốc độ
- Triển khai intelligent tool routing dựa trên query type
- Giám sát tool performance và tối ưu phù hợp với usage patterns
- Thiết lập timeout và retry policies hợp lý

**🛡️ Cân Nhắc Bảo Mật:**
- Validate tất cả tool inputs và outputs
- Triển khai rate limiting để prevent abuse
- Monitor tool usage cho suspicious patterns
- Secure vector stores và sensitive documents
- Sử dụng proper access controls cho computer automation

**📊 Monitoring & Analytics:**
- Track tool usage patterns và performance metrics
- Monitor error rates và failure types
- Đo lường user satisfaction với tool-enabled responses
- Phân tích cost implications của different tools
- Setup alerts cho critical tool failures

### Các Lỗi Thường Gặp và Cách Khắc Phục

❌ **Tool Overuse** → Sử dụng đúng tool cho đúng việc, không dùng tất cả tools cho mọi query  
❌ **Poor Error Handling** → Triển khai comprehensive retry logic với exponential backoff  
❌ **Performance Issues** → Profile tool usage và optimize dựa trên actual bottlenecks  
❌ **Security Gaps** → Validate inputs, secure credentials, monitor access patterns  
❌ **Cost Explosion** → Monitor usage, implement budgets, optimize tool selection  

### Patterns Nâng Cao

🔄 **Tool Chaining** - Output của tool A trở thành input của tool B để tạo workflows phức tạp  
🎯 **Conditional Tool Use** - Lựa chọn tools dựa trên query analysis và context  
🔀 **Parallel Tool Execution** - Chạy multiple tools đồng thời để tăng hiệu suất  
📈 **Progressive Enhancement** - Bắt đầu với fast tools, escalate to powerful ones khi cần  
🧠 **Tool Learning** - Thích ứng tool selection dựa trên success patterns và feedback  

### Bước Tiếp Theo

Trong **bài tiếp theo**, chúng ta sẽ khám phá [Model Context Protocol (MCP): Extending Agent Capabilities](../openai-agents-sdk-part-07-mcp-model-context-protocol/):

🌐 **Model Context Protocol (MCP)** - Mở rộng khả năng agents với external systems  
🔌 **Custom Integrations** - Kết nối với databases, APIs, enterprise systems  
🏗️ **Architecture Patterns** - Xây dựng agent systems có thể mở rộng và maintainable  

### Thử Thách Cho Bạn

1. **Xây dựng integrated research system** kết hợp web search + data analysis cho domain cụ thể
2. **Tạo automation workflows** sử dụng ComputerTool cho các tác vụ lặp đi lặp lại
3. **Thiết kế business intelligence dashboard** sử dụng multiple tools và real data
4. **Triển khai performance monitoring** cho tool usage trong production environment

---

*Bài tiếp theo: [**"Model Context Protocol (MCP): Extending Agent Capabilities"**](../openai-agents-sdk-part-07-mcp-model-context-protocol/) - Chúng ta sẽ học cách mở rộng khả năng agents thông qua MCP servers và external system integration.*