# Automated Email Inbox Handler

This example demonstrates how Atomic can be used to build an intelligent email processing system that automatically categorizes, prioritizes, and responds to incoming emails.

## Flow Diagram

```
┌───────────────────────────────┐
│ 1. Incoming Email             │
│    (Inbox Event Trigger)      │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 2. Preprocessing & Routing    │
│    (Parse email content)      │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 3. Model Bidding System       │
│    (Microtask bids for:       │
│    categorization, sentiment, │
│    spam detection, etc.)      │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 4. Inference-Time Fine-Tuning │
│    (Adaptive response setup)  │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 5. Model Execution Engine     │
│    (Generate structured       │
│    response via BAML)         │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 6. Observability & Logging    │
│    (Trace reasons for model   │
│    selection, final response) │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ 7. Automated Reply / Action   │
│    (Send response, tag email, │
│    trigger further workflows) │
└───────────────────────────────┘
```

## Implementation Walkthrough

### 1. Configuration and Setup

First, we define the schema for our email processing pipeline using BAML (BoundaryML) to ensure type-safe outputs:

```python
# Define schema for email processing outputs
from atomic import Schema, Field, Enum

class EmailCategory(Enum):
    CUSTOMER_SUPPORT = "customer_support"
    SALES_INQUIRY = "sales_inquiry"
    PARTNERSHIP = "partnership"
    TECHNICAL_ISSUE = "technical_issue"
    BILLING = "billing"
    OTHER = "other"

class SentimentLevel(Enum):
    POSITIVE = "positive"
    NEUTRAL = "neutral"
    NEGATIVE = "negative"
    URGENT_NEGATIVE = "urgent_negative"

class EmailAnalysisResult(Schema):
    category: EmailCategory
    sentiment: SentimentLevel
    priority: int  # 1-5 scale
    contains_question: bool
    requires_response: bool
    summary: str
    suggested_response: str = None  # Optional
    tags: list[str]
    next_actions: list[str]
    confidence: float  # 0-1 scale
```

### 2. Defining Microtasks

Next, we break down email processing into microtasks that models can bid on:

```python
from atomic import Microtask, TaskManager

# Define microtasks for email processing
email_categorization = Microtask(
    name="email_categorization",
    input_schema={"email_body": str, "email_subject": str, "sender": str},
    output_schema={"category": EmailCategory, "confidence": float}
)

sentiment_analysis = Microtask(
    name="sentiment_analysis",
    input_schema={"email_body": str, "email_subject": str},
    output_schema={"sentiment": SentimentLevel, "confidence": float}
)

priority_scoring = Microtask(
    name="priority_scoring",
    input_schema={"email_body": str, "email_subject": str, "sender": str, 
                 "sentiment": SentimentLevel, "category": EmailCategory},
    output_schema={"priority": int, "confidence": float}
)

response_generation = Microtask(
    name="response_generation",
    input_schema={"email_body": str, "email_subject": str, "sender": str,
                 "category": EmailCategory, "sentiment": SentimentLevel},
    output_schema={"suggested_response": str, "confidence": float}
)

# Register microtasks with task manager
task_manager = TaskManager()
task_manager.register([
    email_categorization,
    sentiment_analysis,
    priority_scoring,
    response_generation
])
```

### 3. Registering Models for Bidding

We register multiple models that can bid on these microtasks:

```python
from atomic import ModelRegistry, AIModel

# Register models with their capabilities and bidding strategies
model_registry = ModelRegistry()

# General-purpose models
model_registry.register(
    AIModel(
        name="gpt-4o",
        provider="openai",
        capabilities=["email_categorization", "sentiment_analysis", 
                    "priority_scoring", "response_generation"],
        bidding_strategy="confidence_based",
        cost_per_token=0.00007
    )
)

model_registry.register(
    AIModel(
        name="claude-3-opus",
        provider="anthropic",
        capabilities=["email_categorization", "sentiment_analysis", 
                    "priority_scoring", "response_generation"],
        bidding_strategy="confidence_based",
        cost_per_token=0.00008
    )
)

# Specialized models
model_registry.register(
    AIModel(
        name="sentiment-bert",
        provider="huggingface",
        capabilities=["sentiment_analysis"],
        bidding_strategy="specialization",
        cost_per_token=0.00001
    )
)

model_registry.register(
    AIModel(
        name="customer-support-llama",
        provider="local",
        capabilities=["email_categorization", "response_generation"],
        specializations=["customer_support", "technical_issue"],
        bidding_strategy="domain_specialization",
        cost_per_token=0.00002
    )
)
```

### 4. Creating the Email Processing Pipeline

Now we set up the complete pipeline using Atomic's orchestration:

```python
from atomic import Pipeline, BiddingSystem, InferenceTimeTuner, OutputValidator

# Create email processing pipeline
email_pipeline = Pipeline(
    name="email_processor",
    description="Processes incoming emails for automated handling"
)

# Add bidding system for model selection
email_pipeline.add_component(
    BiddingSystem(
        model_registry=model_registry,
        optimization_goal="balanced",  # Options: speed, cost, accuracy, balanced
        bid_timeout_ms=200  # Maximum time to wait for bids
    )
)

# Add inference-time fine-tuning
email_pipeline.add_component(
    InferenceTimeTuner(
        adaptation_strategies=["few_shot", "prompt_engineering", "parameter_efficient_tuning"],
        context_window_size=10  # Number of recent examples to use for adaptation
    )
)

# Add output validation
email_pipeline.add_component(
    OutputValidator(
        strict_mode=True,  # Enforce schema validation
        fallback_strategy="retry_with_guidance"  # What to do if validation fails
    )
)

# Define the execution flow
email_pipeline.set_flow([
    # Step 1: Categorize the email
    {
        "task": "email_categorization",
        "inputs": {
            "email_body": "$.email.body",
            "email_subject": "$.email.subject",
            "sender": "$.email.sender"
        },
        "outputs": {
            "category": "$.analysis.category",
            "confidence": "$.analysis.category_confidence"
        }
    },
    
    # Step 2: Analyze sentiment (in parallel with step 1)
    {
        "task": "sentiment_analysis",
        "inputs": {
            "email_body": "$.email.body",
            "email_subject": "$.email.subject"
        },
        "outputs": {
            "sentiment": "$.analysis.sentiment",
            "confidence": "$.analysis.sentiment_confidence"
        }
    },
    
    # Step 3: Score priority (depends on steps 1 and 2)
    {
        "task": "priority_scoring",
        "inputs": {
            "email_body": "$.email.body",
            "email_subject": "$.email.subject",
            "sender": "$.email.sender",
            "category": "$.analysis.category",
            "sentiment": "$.analysis.sentiment"
        },
        "outputs": {
            "priority": "$.analysis.priority",
            "confidence": "$.analysis.priority_confidence"
        }
    },
    
    # Step 4: Generate response if needed
    {
        "task": "response_generation",
        "condition": "$.analysis.priority >= 3",  # Only for higher priority emails
        "inputs": {
            "email_body": "$.email.body",
            "email_subject": "$.email.subject",
            "sender": "$.email.sender",
            "category": "$.analysis.category",
            "sentiment": "$.analysis.sentiment"
        },
        "outputs": {
            "suggested_response": "$.analysis.suggested_response",
            "confidence": "$.analysis.response_confidence"
        }
    }
])
```

### 5. Setting Up API Endpoint and Email Integration

Finally, we create an API endpoint to receive incoming emails:

```python
from atomic import create_api_endpoint
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel

app = FastAPI()

class EmailPayload(BaseModel):
    sender: str
    recipient: str
    subject: str
    body: str
    attachments: list = []
    metadata: dict = {}

@app.post("/api/incoming-email")
async def process_email(email: EmailPayload, background_tasks: BackgroundTasks):
    # Process email in background to avoid blocking
    background_tasks.add_task(handle_email, email)
    return {"status": "processing"}

async def handle_email(email: EmailPayload):
    # Initialize the pipeline with the email data
    pipeline_run = email_pipeline.start_run(
        input_data={
            "email": {
                "sender": email.sender,
                "recipient": email.recipient,
                "subject": email.subject,
                "body": email.body,
                "received_at": datetime.now().isoformat()
            }
        }
    )
    
    # Execute the pipeline
    result = await pipeline_run.execute()
    
    # Take actions based on the analysis
    if result.analysis.priority >= 4:
        # High priority - notify team
        await notify_team(email, result.analysis)
    
    if result.analysis.requires_response and result.analysis.suggested_response:
        # Send automated response if confidence is high
        if result.analysis.response_confidence > 0.85:
            await send_response(email, result.analysis.suggested_response)
        else:
            # Queue for human review
            await queue_for_review(email, result.analysis)
    
    # Log the complete analysis for observability
    await log_email_processing(email, result)
    
    # Return the result
    return result

# Start the API server
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Key Benefits of Using Atomic for Email Processing

### 1. Dynamic Model Selection

The bidding system automatically selects the most appropriate model for each email processing task:

- Specialized sentiment analysis models are used for detecting customer frustration
- Domain-specific models handle technical support queries
- Cost-efficient models handle routine categorization
- More powerful models take on complex response generation

### 2. Adaptive Processing

The inference-time fine-tuning allows models to adapt to specific email contexts:

- Learns company-specific terminology over time
- Adapts to different communication styles
- Improves responses based on historical interactions with specific senders

### 3. Comprehensive Observability

Every decision in the email processing pipeline is tracked and explainable:

- Why a particular model was selected for each task
- Confidence scores for each analysis component
- Performance metrics across different email types and senders
- Cost breakdown per processing step

### 4. Type-Safe Outputs

The BAML schema ensures that all outputs conform to expected formats:

- Prevents incorrect categorizations
- Ensures priority scores are within valid ranges
- Validates that all required fields are present
- Enables reliable integration with downstream systems

### 5. Cost Optimization

The bidding system optimizes for cost-efficiency:

- Uses minimal models for routine tasks
- Deploys expensive models only when necessary
- Balances performance and cost based on email priority
- Provides detailed cost attribution for optimization

## Observability Example

Here's an example of the observability data captured for a processed email:

```json
{
  "request_id": "email-proc-7821",
  "timestamp": "2024-03-17T14:23:45Z",
  "input_metadata": {
    "email_size_bytes": 4832,
    "has_attachments": false,
    "sender_domain": "customer.com"
  },
  "task_execution": [
    {
      "task": "email_categorization",
      "bidding": {
        "participants": [
          {"model": "gpt-4o", "confidence": 0.92, "bid": 0.0035},
          {"model": "claude-3-opus", "confidence": 0.88, "bid": 0.0042},
          {"model": "customer-support-llama", "confidence": 0.95, "bid": 0.0012}
        ],
        "winner": "customer-support-llama",
        "selection_reason": "Highest confidence at lowest cost"
      },
      "execution_metrics": {
        "latency_ms": 87,
        "tokens_processed": 312,
        "cost": 0.000624
      },
      "result": {
        "category": "technical_issue",
        "confidence": 0.95
      }
    },
    {
      "task": "sentiment_analysis",
      "bidding": {
        "participants": [
          {"model": "gpt-4o", "confidence": 0.87, "bid": 0.0024},
          {"model": "claude-3-opus", "confidence": 0.91, "bid": 0.0028},
          {"model": "sentiment-bert", "confidence": 0.96, "bid": 0.0008}
        ],
        "winner": "sentiment-bert",
        "selection_reason": "Specialized model with highest confidence"
      },
      "execution_metrics": {
        "latency_ms": 42,
        "tokens_processed": 312,
        "cost": 0.000312
      },
      "result": {
        "sentiment": "negative",
        "confidence": 0.96
      }
    },
    // Additional tasks omitted for brevity
  ],
  "total_metrics": {
    "total_latency_ms": 342,
    "total_cost": 0.00283,
    "models_used": ["customer-support-llama", "sentiment-bert", "gpt-4o"],
    "success": true
  }
}
```

## Conclusion

This example demonstrates how Atomic's bidding system, inference-time fine-tuning, and structured outputs create a powerful, efficient, and observable email processing system. The architecture allows for:

1. **Dynamic adaptation** to different email types and contexts
2. **Cost optimization** through intelligent model selection
3. **High reliability** through type-safe outputs and comprehensive validation
4. **Complete observability** for debugging and optimization
5. **Flexible deployment** in cloud or on-premise environments

For more examples and implementation details, please refer to the [Atomic documentation](../docs/examples.md).
