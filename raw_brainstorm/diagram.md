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
