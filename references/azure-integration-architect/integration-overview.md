## The Azure Integration Stack

### When to Use What
```
Need to orchestrate multi-step workflows?           → Logic Apps
Need reliable async messaging between services?     → Service Bus
Need event notifications / reactive pub-sub?         → Event Grid
Need an API gateway / façade for backends?           → API Management
Need lightweight event-driven compute?               → Azure Functions
Need large-scale data movement / ETL?                → Data Factory
```

### Integration Architecture Patterns
1. **Basic enterprise integration**: APIM → Logic Apps → Backend (synchronous, simple)
2. **Message broker + events**: APIM → Logic Apps → Service Bus/Event Grid → Backend (async, reliable, scalable)
3. **Event-driven microservices**: Event Grid for reactive, Service Bus for reliable delivery
4. **Hybrid integration**: Logic Apps with on-prem data gateway + Service Bus for bridging cloud/on-prem

The four core integration technologies: **orchestration** (Logic Apps), **messaging** (Service Bus), **events** (Event Grid), and **APIs** (APIM).
