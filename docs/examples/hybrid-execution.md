# Hybrid Execution

This page demonstrates how to implement hybrid execution strategies with Oblix, intelligently routing between local and cloud models based on system resources, connectivity, and other factors.

## What is Hybrid Execution?

Hybrid execution refers to the ability to seamlessly switch between:

- **Local models** (via Ollama) running on your own hardware
- **Cloud models** (via OpenAI, Claude) running on remote servers

This approach combines the advantages of both:

- **Local models**: Privacy, offline capability, no usage fees
- **Cloud models**: Higher capabilities, larger context windows, specialized features

## Basic Hybrid Setup

This example shows how to set up a basic hybrid system:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def main():
    # Initialize client
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook a local model
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2"
    )
    
    # Hook a cloud model
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-3.5-turbo",
        api_key="your_openai_api_key"
    )
    
    # Add resource monitoring
    client.hook_agent(ResourceMonitor())
    
    # Add connectivity monitoring
    client.hook_agent(ConnectivityAgent())
    
    # Execute a prompt (Oblix will automatically choose the appropriate model)
    response = await client.execute("Explain quantum computing in simple terms")
    
    print(f"Response from {response['model_id']}:")
    print(response["response"])
    
    # Print agent check results
    print("\nAgent check results:")
    for agent_name, check_result in response["agent_checks"].items():
        print(f"- {agent_name}: {check_result.get('state')}, target: {check_result.get('target')}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Multi-Tier Model Strategy

For more sophisticated applications, you can implement a multi-tier model strategy:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def main():
    # Initialize client
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Tier 1: Small local model (fast, low resources)
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="phi"  # or other small model
    )
    
    # Tier 2: Medium local model (better quality, more resources)
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2"
    )
    
    # Tier 3: Standard cloud model (good quality, cost-effective)
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-3.5-turbo",
        api_key="your_openai_api_key"
    )
    
    # Tier 4: Advanced cloud model (highest quality, more expensive)
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-4",
        api_key="your_openai_api_key"
    )
    
    # Add monitoring agents
    client.hook_agent(ResourceMonitor())
    client.hook_agent(ConnectivityAgent())
    
    # Execute a simple prompt (likely routes to a lower tier)
    simple_response = await client.execute("What is the capital of France?")
    print(f"Simple query used: {simple_response['model_id']}")
    
    # Execute a complex prompt (likely routes to a higher tier)
    complex_response = await client.execute(
        "Analyze the implications of quantum computing on modern cryptography, "
        "including potential vulnerabilities in RSA and elliptic curve systems."
    )
    print(f"Complex query used: {complex_response['model_id']}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Offline-First Implementation

This example prioritizes offline capability while falling back to cloud when necessary:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def offline_first_app():
    # Initialize client
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Primary model: Local
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2"
    )
    
    # Fallback model: Cloud (only used when necessary)
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-3.5-turbo",
        api_key="your_openai_api_key"
    )
    
    # Add connectivity monitoring
    connectivity_agent = ConnectivityAgent(
        # Stricter thresholds to prefer local models
        latency_threshold=100.0,       # Default is 200.0
        packet_loss_threshold=5.0,     # Default is 10.0
        bandwidth_threshold=10.0       # Default is 5.0
    )
    client.hook_agent(connectivity_agent)
    
    # Add resource monitoring
    resource_monitor = ResourceMonitor(
        custom_thresholds={
            # More lenient resource thresholds to prefer local models
            "cpu_threshold": 90.0,     # Default is 80.0
            "memory_threshold": 90.0,  # Default is 85.0
        }
    )
    client.hook_agent(resource_monitor)
    
    # Function to process user input
    async def process_user_query(query):
        try:
            response = await client.execute(query)
            print(f"\nUsing model: {response['model_id']}")
            print(f"Response: {response['response']}")
            
            # Check if we're offline
            connectivity = response["agent_checks"].get("connectivity_monitor", {})
            if connectivity.get("state") == "disconnected":
                print("\n[OFFLINE MODE ACTIVE]")
        
        except Exception as e:
            print(f"Error: {e}")
            # Even if everything fails, provide minimal functionality
            print("Unable to process request. Working in emergency mode.")
    
    # Simple interactive loop
    print("Offline-First AI Assistant (type 'exit' to quit)")
    while True:
        user_input = input("\nYou: ")
        if user_input.lower() == 'exit':
            break
        await process_user_query(user_input)

if __name__ == "__main__":
    asyncio.run(offline_first_app())
```

## Performance-Optimized Hybrid

This example optimizes for performance in a hybrid setup:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def performance_optimized_app():
    # Initialize client
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Fast local model for quick responses
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="phi"  # Small, fast model
    )
    
    # Powerful cloud model for complex tasks
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-3.5-turbo",
        api_key="your_openai_api_key"
    )
    
    # Add resource monitoring
    resource_monitor = ResourceMonitor()
    client.hook_agent(resource_monitor)
    
    # Add connectivity monitoring with performance-oriented thresholds
    connectivity_agent = ConnectivityAgent(
        latency_threshold=100.0,  # Lower threshold for better performance
        check_interval=15         # More frequent checks
    )
    client.hook_agent(connectivity_agent)
    
    # Example task classification function
    def classify_task_complexity(prompt):
        # Simple heuristic: longer prompts or certain keywords indicate complexity
        complexity_keywords = [
            "analyze", "compare", "explain", "discuss", "evaluate", 
            "synthesize", "recommend", "critique"
        ]
        
        # Check prompt length and keywords
        is_complex = len(prompt.split()) > 20 or any(kw in prompt.lower() for kw in complexity_keywords)
        return "complex" if is_complex else "simple"
    
    # Process tasks based on complexity
    async def process_task(prompt):
        task_type = classify_task_complexity(prompt)
        
        if task_type == "simple":
            # For simple tasks, explicitly use the faster local model
            print("Simple task detected, using local model for speed")
            response = await client.execute(prompt, model_id="ollama:phi")
        else:
            # For complex tasks, let Oblix decide based on current conditions
            print("Complex task detected, using intelligent routing")
            response = await client.execute(prompt)
        
        print(f"Response from {response['model_id']}:")
        print(response["response"])
        
        # Performance metrics
        metrics = response.get("metrics", {})
        if metrics:
            latency = metrics.get("total_latency")
            if latency:
                print(f"Response time: {latency:.2f} seconds")
    
    # Example tasks
    tasks = [
        "What is 15 + 27?",
        "What's the capital of Japan?",
        "Explain the theory of relativity and its implications for modern physics, including experimental evidence.",
        "Recommend a strategy for implementing a microservice architecture in a legacy monolithic application."
    ]
    
    # Process all tasks
    for task in tasks:
        print(f"\nTask: {task}")
        await process_task(task)

if __name__ == "__main__":
    asyncio.run(performance_optimized_app())
```

## Cost-Optimized Hybrid

This example optimizes for cost in a hybrid setup:

```python
import asyncio
import time
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent
from oblix.agents.base import BaseAgent
from typing import Dict, Any

# Custom agent for cost optimization
class CostOptimizationAgent(BaseAgent):
    def __init__(self, name: str = "cost_optimizer", 
                 daily_budget: float = 5.0):
        super().__init__(name)
        self.daily_budget = daily_budget
        self.daily_spend = 0.0
        self.reset_time = time.time() + 86400  # 24 hours from now
        
        # Estimated cost per 1K tokens
        self.cost_per_model = {
            "openai:gpt-3.5-turbo": 0.0015,
            "openai:gpt-4": 0.03,
            "claude:claude-3-opus-20240229": 0.015,
            "claude:claude-3-sonnet-20240229": 0.003,
            "ollama:llama2": 0.0,
            "ollama:phi": 0.0
        }
    
    async def initialize(self) -> bool:
        self.is_active = True
        return True
    
    def _estimate_cost(self, model_id: str, input_tokens: int, output_tokens: int) -> float:
        """Estimate cost for a model execution"""
        if model_id not in self.cost_per_model:
            return 0.0
        
        cost_per_1k = self.cost_per_model[model_id]
        total_tokens = input_tokens + output_tokens
        return (total_tokens / 1000) * cost_per_1k
    
    def _update_spend(self, model_id: str, input_tokens: int, output_tokens: int):
        """Update running spend total"""
        # Reset daily budget if needed
        if time.time() > self.reset_time:
            self.daily_spend = 0.0
            self.reset_time = time.time() + 86400
        
        # Add estimated cost
        self.daily_spend += self._estimate_cost(model_id, input_tokens, output_tokens)
    
    async def check(self, **kwargs) -> Dict[str, Any]:
        # Get prompt info if available
        prompt = kwargs.get("prompt", "")
        
        # Check budget status
        budget_percentage = (self.daily_spend / self.daily_budget) * 100
        
        if budget_percentage > 90:
            # Critical budget: force local models
            return {
                "proceed": True,
                "state": "budget_critical",
                "target": "local",
                "reason": f"Budget nearly depleted: ${self.daily_spend:.2f}/${self.daily_budget:.2f}",
                "metrics": {
                    "budget_percentage": budget_percentage,
                    "daily_spend": self.daily_spend,
                    "daily_budget": self.daily_budget
                }
            }
        elif budget_percentage > 70:
            # High budget use: prefer local but allow cloud for complex tasks
            return {
                "proceed": True,
                "state": "budget_constrained",
                "target": "hybrid",
                "reason": f"Budget usage high: ${self.daily_spend:.2f}/${self.daily_budget:.2f}",
                "metrics": {
                    "budget_percentage": budget_percentage,
                    "daily_spend": self.daily_spend,
                    "daily_budget": self.daily_budget
                }
            }
        else:
            # Budget available: make recommendations based on estimated complexity
            # Simple heuristic: longer prompts may need more powerful models
            is_complex = len(prompt.split()) > 25
            
            return {
                "proceed": True,
                "state": "budget_available",
                "target": "cloud" if is_complex else "local",
                "reason": f"Budget available: ${self.daily_spend:.2f}/${self.daily_budget:.2f}",
                "metrics": {
                    "budget_percentage": budget_percentage,
                    "daily_spend": self.daily_spend,
                    "daily_budget": self.daily_budget,
                    "estimated_complexity": "complex" if is_complex else "simple"
                }
            }
    
    async def shutdown(self) -> None:
        self.is_active = False

async def cost_optimized_app():
    # Initialize client
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook models
    # Tier 1: Free local model (no cost)
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2"
    )
    
    # Tier 2: Economy cloud model (low cost)
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-3.5-turbo",
        api_key="your_openai_api_key"
    )
    
    # Tier 3: Premium cloud model (higher cost, only used when necessary)
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-4",
        api_key="your_openai_api_key"
    )
    
    # Add standard monitoring agents
    client.hook_agent(ResourceMonitor())
    client.hook_agent(ConnectivityAgent())
    
    # Add cost optimization agent
    cost_agent = CostOptimizationAgent(daily_budget=5.0)  # $5 daily budget
    client.hook_agent(cost_agent)
    
    # Test with various queries
    test_queries = [
        "What time is it?",  # Very simple
        "Explain how photosynthesis works",  # Moderate
        "Write a detailed analysis of sustainable energy solutions for developing nations",  # Complex
    ]
    
    for query in test_queries:
        print(f"\n--- Query: {query}")
        
        # Execute the query
        response = await client.execute(query)
        
        # Print model used and cost info
        model_id = response["model_id"]
        print(f"Model used: {model_id}")
        
        # Update cost tracker with token usage
        metrics = response.get("metrics", {})
        input_tokens = metrics.get("input_tokens", len(query.split()) * 1.3)  # Rough estimate if not available
        output_tokens = metrics.get("output_tokens", len(response["response"].split()) * 1.3)
        
        cost_agent._update_spend(model_id, input_tokens, output_tokens)
        
        # Print budget status
        budget_percentage = (cost_agent.daily_spend / cost_agent.daily_budget) * 100
        print(f"Budget status: ${cost_agent.daily_spend:.4f} spent (of ${cost_agent.daily_budget:.2f} total)")
        print(f"Budget usage: {budget_percentage:.1f}%")
        
        # Abbreviated response
        response_text = response["response"]
        print(f"Response: {response_text[:100]}..." if len(response_text) > 100 else response_text)
    
    print("\n--- Final Budget Status ---")
    print(f"Total spend: ${cost_agent.daily_spend:.4f}")
    print(f"Remaining budget: ${cost_agent.daily_budget - cost_agent.daily_spend:.4f}")

if __name__ == "__main__":
    asyncio.run(cost_optimized_app())
```

## Conclusion

Hybrid execution with Oblix provides several key benefits:

1. **Resilience**: Continue operation even without internet connectivity
2. **Cost optimization**: Use free local models when possible, cloud models when necessary
3. **Performance optimization**: Route to the most appropriate model based on the task
4. **Privacy**: Keep sensitive prompts on local models
5. **Adaptability**: Automatically adjust to changing conditions

By implementing these patterns, you can build AI applications that are more robust, cost-effective, and performant than those using only local or only cloud models.
