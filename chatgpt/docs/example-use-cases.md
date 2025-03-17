# Example Use Cases

Below are hypothetical scenarios illustrating how Atomic's bidding and inference-time 
fine-tuning might be used in practice.

1. **Email Routing & Automated Replies**  
   - **Microtasks**: Spam detection, intent classification, sentiment analysis.
   - **Bidding**: Basic language models vs. specialized spam filters. System picks 
     the highest-confidence or fastest models, balancing cost with accuracy.

2. **Customer Support Chatbot**  
   - **Microtasks**: Entity extraction, user intent, recommended solutions.
   - **Bidding**: Different large language models or smaller domain-focused models 
     may submit bids. The orchestrator might prefer a less expensive model if 
     confidence is above a threshold.

3. **Document Classification for Enterprises**  
   - **Microtasks**: Topic classification, access control tagging, summarization.
   - **Bidding**: Domain-specific models might have higher accuracy but cost more. 
     Atomic chooses them only for specialized documents; cheaper general-purpose 
     models handle simpler tasks.

4. **Financial Data Analysis**  
   - **Microtasks**: Fraud detection, risk scoring, anomaly detection.
   - **Bidding**: Legacy rule-based engines vs. advanced machine learning. The system 
     might prefer the advanced model if the stakes (risk) justify higher compute costs.

5. **Health Care Triage**  
   - **Microtasks**: Symptom classification, urgency detection, recommended next steps.
   - **Bidding**: Strict reliability or regulation might require high-trust medical 
     models. The orchestrator logs each selection for compliance audits.

---

In all cases, **inference-time fine-tuning** allows these models to adjust on the fly, 
adapting to new data, changes in user input, or updated business logic without 
extensive retraining.
