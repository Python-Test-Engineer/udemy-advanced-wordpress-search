# Agentic RAG:

## What is RAG First?

**RAG stands for Retrieval-Augmented Generation**

Think of it like an open-book exam:
- **Traditional AI**: Answers only from what it memorized (like a closed-book exam)
- **RAG**: Can look up information in documents before answering (like an open-book exam)

### Basic RAG Flow:
1. **User asks a question**: "What was our Q3 revenue?"
2. **Retrieval**: System searches documents and finds relevant information
3. **Augmentation**: Adds the retrieved info to the AI's context
4. **Generation**: AI generates an answer using both its knowledge AND the retrieved documents

## What is Agentic RAG?

**Agentic RAG** = RAG + **Agency** (autonomous decision-making)

Instead of just one simple retrieve-and-answer step, the AI acts like an intelligent **agent** that:
- Plans its approach
- Decides which sources to check
- Determines if it needs more information
- Iteratively refines its search
- Validates its findings

### The Key Difference:

**Basic RAG (Passive):**
```
Question → Retrieve docs → Generate answer
```

**Agentic RAG (Active):**
```
Question → Plan approach → Retrieve → Evaluate → Need more? → Retrieve again → Synthesize → Verify → Answer
```

## Real-World Example

### Scenario: Student asks "Compare the economic policies of the last three presidents"

**Basic RAG would:**
1. Search for "economic policies presidents"
2. Retrieve top 5 documents
3. Generate answer from those documents
4. Done ✓

**Agentic RAG would:**
1. **Plan**: "I need info on 3 different presidents, so I should search for each separately"
2. **Execute searches**:
   - Search "Biden economic policy"
   - Search "Trump economic policy"
   - Search "Obama economic policy"
3. **Evaluate**: "Do I have enough detail on fiscal policy? What about trade?"
4. **Refine searches**:
   - Search "Biden fiscal policy tax"
   - Search "Trump trade tariffs"
5. **Synthesize**: Organize findings into coherent comparison
6. **Verify**: Check if any claims conflict, cross-reference sources
7. **Generate**: Produce comprehensive answer

## Core Components of Agentic RAG

### 1. **Planning & Reasoning**
The agent breaks down complex questions into sub-tasks

```python
Question: "How has remote work affected tech company productivity?"

Agent's Plan:
- Sub-task 1: Find data on remote work adoption in tech
- Sub-task 2: Find productivity metrics before/after
- Sub-task 3: Find expert opinions on causation
- Sub-task 4: Synthesize findings
```

### 2. **Tool Use**
The agent can use multiple tools, not just simple search

**Available Tools:**
- Semantic search (find similar concepts)
- Keyword search (find exact terms)
- SQL queries (structured data)
- Web search (current info)
- Calculator (compute statistics)
- Code execution (analyze data)

### 3. **Iterative Retrieval**
The agent can retrieve multiple times based on what it learns

```
Initial query → Find gap in knowledge → Targeted follow-up query → Still missing something? → Another query
```

### 4. **Self-Reflection**
The agent evaluates its own answers

**Questions the agent asks itself:**
- "Did I answer the full question?"
- "Do I have enough evidence?"
- "Are there contradictions I need to resolve?"
- "Should I search for more recent information?"

### 5. **Source Validation**
The agent checks credibility and consistency

**Agent might:**
- Cross-reference multiple sources
- Prioritize peer-reviewed sources
- Flag conflicting information
- Note when sources are outdated

## Architecture Comparison

### Traditional RAG Architecture:
```
┌─────────────┐
│ User Query  │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  Embed Query    │
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│ Retrieve Docs   │ (One-shot retrieval)
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│ Generate Answer │
└─────────────────┘
```

### Agentic RAG Architecture:
```
┌─────────────┐
│ User Query  │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  Agent Planner  │ ← Plans multi-step approach
└──────┬──────────┘
       │
       ▼
┌─────────────────────────────────┐
│     Reasoning Loop              │
│  ┌─────────────────────┐       │
│  │ 1. Choose Tool      │       │
│  │ 2. Execute Action   │◄──┐   │
│  │ 3. Evaluate Result  │   │   │
│  │ 4. Need more? ──────┘   │   │
│  └─────────────────────┘   │   │
│                             │   │
│  Tools Available:           │   │
│  • Semantic Search          │   │
│  • Keyword Search           │   │
│  • Web Search              │   │
│  • SQL Query               │   │
│  • Calculator              │   │
└─────────────┬───────────────┘
              │
              ▼
     ┌─────────────────┐
     │ Synthesize &    │
     │ Generate Answer │
     └─────────────────┘
```

## Key Advantages of Agentic RAG

### 1. **Handles Complex Queries**
Can break down multi-part questions and answer them systematically

**Example:**
- Question: "What are the pros and cons of our competitor's pricing strategy, and how should we respond?"
- Agent: Searches for competitor pricing → Analyzes pros/cons → Searches for our pricing → Formulates recommendations

### 2. **Reduces Hallucinations**
Can verify its own answers and search for contradictory evidence

### 3. **Adapts to Context**
Chooses different strategies based on the question type

- Factual question → Quick keyword search
- Analytical question → Multiple searches + synthesis
- Recent events → Web search first
- Technical question → Look in specific documentation

### 4. **Provides Better Transparency**
Shows its reasoning process and which sources it used

### 5. **Self-Corrects**
If initial retrieval doesn't answer the question, it can try different approaches

## Common Agentic RAG Patterns

### Pattern 1: ReAct (Reasoning + Acting)
The agent alternates between reasoning about what to do next and taking actions

```
Thought: I need to find the latest sales data
Action: search_documents("Q4 2024 sales")
Observation: Found revenue numbers but not profit margins
Thought: I need profit data too
Action: search_documents("Q4 2024 profit margin")
Observation: Found profit margins
Thought: Now I can calculate the answer
Action: calculate(revenue, margin)
Answer: [Final response]
```

### Pattern 2: Plan-and-Execute
Agent makes a complete plan first, then executes it

```
Plan:
  Step 1: Get historical data (2020-2024)
  Step 2: Get current quarter data
  Step 3: Calculate growth rate
  Step 4: Compare to industry average

Execute each step...
```

### Pattern 3: Self-Ask
Agent generates and answers sub-questions

```
Main Question: "Why did our customer churn increase?"

Sub-Q1: What was the churn rate? → Search & Answer
Sub-Q2: When did it start increasing? → Search & Answer  
Sub-Q3: What changed around that time? → Search & Answer
Sub-Q4: What do customers say? → Search reviews → Answer

Final Answer: [Synthesized from all sub-answers]
```

## Building a Simple Agentic RAG (Conceptual)

Here's a simplified Python-like pseudocode:

```python
class AgenticRAG:
    def __init__(self):
        self.tools = {
            'search': self.search_documents,
            'web_search': self.search_web,
            'calculate': self.calculator
        }
    
    def answer_question(self, question):
        # 1. Plan
        plan = self.create_plan(question)
        
        # 2. Execute with reasoning loop
        context = []
        for step in plan:
            # Reason about what to do
            thought = self.reason(step, context)
            
            # Choose and use a tool
            tool = self.choose_tool(thought)
            result = self.tools[tool](step)
            
            # Evaluate if we have enough
            context.append(result)
            if self.evaluate_sufficiency(question, context):
                break
            
            # Otherwise, refine and continue
            plan = self.refine_plan(plan, context)
        
        # 3. Synthesize final answer
        answer = self.generate_answer(question, context)
        
        # 4. Verify answer quality
        if not self.verify_answer(answer, context):
            # Try a different approach
            return self.answer_question(question)  # Retry
        
        return answer
    
    def reason(self, step, context):
        """Think about what information we need"""
        return f"To complete {step}, I need to..."
    
    def choose_tool(self, thought):
        """Decide which tool to use"""
        if "recent" in thought:
            return 'web_search'
        elif "calculate" in thought:
            return 'calculate'
        else:
            return 'search'
    
    def evaluate_sufficiency(self, question, context):
        """Do we have enough information?"""
        # Check if all parts of question are covered
        return self.coverage_check(question, context) > 0.9
```

## When to Use Agentic RAG vs. Basic RAG

### Use **Basic RAG** when:
- Simple factual lookups ("What is the capital of France?")
- Single-source answers ("What does page 5 of the manual say?")
- Speed is critical
- Questions are straightforward

### Use **Agentic RAG** when:
- Complex, multi-part questions
- Comparative analysis needed
- Answer requires synthesis from multiple sources
- Query is ambiguous and needs clarification
- Need to verify conflicting information
- Research-style questions

## Real-World Applications

### 1. **Customer Support**
- Agentic RAG can search FAQs, product docs, ticket history
- Self-corrects if first answer doesn't match the issue
- Escalates if it can't find solution

### 2. **Research Assistant**
- Breaks down research questions
- Searches multiple databases
- Cross-references findings
- Produces literature review

### 3. **Legal Document Analysis**
- Searches case law, statutes, precedents
- Identifies contradictions
- Builds comprehensive legal argument

### 4. **Business Intelligence**
- Queries multiple data sources
- Performs calculations
- Generates insights with supporting evidence

## Challenges & Limitations

### 1. **Latency**
Multiple retrieval steps take more time than single retrieval

**Solution**: Balance between thoroughness and speed

### 2. **Cost**
More LLM calls = higher API costs

**Solution**: Set limits on reasoning iterations

### 3. **Complexity**
Harder to debug and predict behavior

**Solution**: Log agent's reasoning process

### 4. **Looping**
Agent might get stuck in repetitive searches

**Solution**: Set maximum iterations, detect loops

## Future of Agentic RAG

**Emerging trends:**
- Multi-agent systems (multiple specialized agents working together)
- Memory systems (agents remember past interactions)
- Fine-tuned models for specific reasoning tasks
- Better evaluation metrics for agent performance
- Integration with external APIs and tools

## Key Takeaways

1. **Basic RAG** = Retrieve once, generate answer
2. **Agentic RAG** = Plan, retrieve iteratively, reason, verify, then answer
3. **Agency** means the AI makes autonomous decisions about how to find information
4. **Best for** complex questions requiring multi-step reasoning
5. **Trade-off** between answer quality and speed/cost

## Practice Questions for Students

1. **Scenario Analysis**: Given the question "Compare our product reviews to competitors", describe how an agentic RAG system would approach this vs. basic RAG.

2. **Design Challenge**: Design an agentic RAG system for a medical diagnosis assistant. What tools would it need? What safety measures?

3. **Critical Thinking**: When might agentic RAG make things worse instead of better? What questions are better suited for basic RAG?

4. **Implementation**: Sketch out the agent's reasoning loop for: "Why did our website traffic drop last month?"

---

## Additional Resources

- **LangChain**: Popular framework for building agentic RAG systems
- **LlamaIndex**: Another framework with built-in agent capabilities  
- **Papers**: "ReAct: Synergizing Reasoning and Acting in Language Models"
- **Tools**: OpenAI Assistants API, Anthropic's Claude with tools


<br>