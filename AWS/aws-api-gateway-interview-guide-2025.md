# AWS API Gateway Interview Guide 2025

---

## 1. What is AWS API Gateway?

A fully managed service to create, publish, maintain, monitor, and secure APIs at any scale.

**Key Features:**
- RESTful APIs and WebSocket APIs
- Serverless (no infrastructure to manage)
- Handles traffic management, authorization, throttling
- Integrates with Lambda, EC2, DynamoDB, etc.

---

## 2. Types of API Gateway

| Type | Use Case | Protocol |
|------|----------|----------|
| **REST API** | Full-featured REST APIs | HTTPS |
| **HTTP API** | Low-latency, cost-effective | HTTPS |
| **WebSocket API** | Real-time two-way communication | WebSocket |

### REST API vs HTTP API

| Feature | REST API | HTTP API |
|---------|----------|----------|
| Cost | Higher | ~70% cheaper |
| Latency | Higher | Lower |
| Features | Full (caching, validation, WAF) | Basic |
| Authorization | IAM, Cognito, Lambda, API Keys | IAM, Cognito, JWT |
| Use when | Need full features | Simple proxy, cost-sensitive |

---

## 3. API Gateway Components

```
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway Structure                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  API                                                         │
│   └── Resources (/users, /orders)                           │
│        └── Methods (GET, POST, PUT, DELETE)                 │
│             └── Integration (Lambda, HTTP, AWS Service)     │
│                  └── Integration Response                   │
│                       └── Method Response                   │
│                                                              │
│  Stage: dev, staging, prod (deployment snapshot)            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Integration Types

| Type | Description |
|------|-------------|
| **Lambda** | Invokes Lambda function |
| **HTTP** | Forwards to HTTP endpoint |
| **AWS Service** | Directly calls AWS services (S3, DynamoDB, SNS) |
| **Mock** | Returns static response (no backend) |
| **VPC Link** | Access private resources in VPC |

### Lambda Proxy vs Lambda Custom Integration

| Feature | Proxy Integration | Custom Integration |
|---------|-------------------|-------------------|
| Request mapping | ❌ Not needed | ✅ Required |
| Response mapping | ❌ Not needed | ✅ Required |
| Flexibility | Less | More |
| Setup | Simple | Complex |
| Full request passed | ✅ Yes | ❌ No (mapped) |

```
Proxy: API Gateway passes entire request to Lambda
Custom: You define request/response mapping templates
```

---

## 5. Authentication & Authorization

| Method | Description |
|--------|-------------|
| **IAM** | AWS credentials (SigV4 signing) |
| **Cognito** | User pools for authentication |
| **Lambda Authorizer** | Custom auth logic (JWT, OAuth) |
| **API Keys** | Simple key-based access (not for auth, use for usage tracking) |

### Lambda Authorizer Types

| Type | Input | Use Case |
|------|-------|----------|
| **Token-based** | Authorization header (Bearer token) | JWT, OAuth tokens |
| **Request-based** | Headers, query strings, path params | Multiple parameters |

```
Client → API Gateway → Lambda Authorizer → (Allow/Deny) → Backend
```

---

## 6. Stages and Deployment

**Stage:** Named reference to a deployment (dev, staging, prod)

```
https://api-id.execute-api.region.amazonaws.com/{stage}/resource

Example:
https://abc123.execute-api.us-east-1.amazonaws.com/prod/users
```

**Stage Variables:** Key-value pairs for configuration
```
Use case: Different Lambda aliases per stage
${stageVariables.lambdaAlias} → dev, prod
```

**Canary Deployments:** Route % of traffic to new version
```
90% → Current version
10% → Canary (new version)
```

---

## 7. Throttling & Quotas

### Default Limits
| Limit | Value |
|-------|-------|
| Steady-state requests | 10,000 RPS (per region) |
| Burst | 5,000 requests |
| Max timeout | 29 seconds |

### Throttling Levels
```
1. Account level (regional)
2. API level
3. Stage level
4. Method level
5. Usage plan level (per client)
```

**429 Error:** Too Many Requests (throttled)

---

## 8. Caching

- Available only for **REST API**
- Cache responses at stage level
- TTL: 0-3600 seconds (default 300)
- Cache size: 0.5 GB to 237 GB
- Reduces backend calls & latency

```
Enable: Stage → Settings → Enable API Cache
Invalidate: Header "Cache-Control: max-age=0"
```

---

## 9. CORS (Cross-Origin Resource Sharing)

**When needed:** Browser calls API from different domain

**Enable CORS:**
1. Enable on resource in API Gateway
2. Returns headers: `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`

**For Lambda Proxy:** Must return CORS headers from Lambda function

```javascript
return {
    statusCode: 200,
    headers: {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "GET,POST,OPTIONS"
    },
    body: JSON.stringify(data)
};
```

---

## 10. Request/Response Transformation

**Mapping Templates:** Transform request/response using VTL (Velocity Template Language)

```velocity
## Request mapping example
{
    "userId": "$input.params('id')",
    "body": $input.json('$')
}
```

**Use Cases:**
- Change request format for backend
- Transform backend response for client
- Add/remove headers

---

## 11. Error Handling

### Common HTTP Status Codes

| Code | Meaning | Cause |
|------|---------|-------|
| 400 | Bad Request | Invalid request |
| 401 | Unauthorized | Auth failed |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Resource not found |
| 429 | Too Many Requests | Throttled |
| 500 | Internal Server Error | Backend error |
| 502 | Bad Gateway | Lambda error, timeout |
| 503 | Service Unavailable | Service overloaded |
| 504 | Gateway Timeout | Integration timeout (>29s) |

### Gateway Responses
Customize error responses for specific errors:
```
4XX errors → Custom message
5XX errors → Custom message
```

---

## 12. Usage Plans & API Keys

**Usage Plan:** Define who can access API and how much

| Setting | Description |
|---------|-------------|
| Throttle | Requests per second limit |
| Quota | Requests per day/week/month |
| API Stages | Which APIs included |

**API Keys:**
- Alphanumeric string for client identification
- Not for authentication (use with authorizers)
- Associate with usage plans

```
Client Request Header: x-api-key: abc123xyz
```

---

## 13. Monitoring & Logging

### CloudWatch Metrics
| Metric | Description |
|--------|-------------|
| Count | Total API requests |
| Latency | Time from request to response |
| IntegrationLatency | Time for backend |
| 4XXError | Client errors |
| 5XXError | Server errors |
| CacheHitCount | Cache hits |
| CacheMissCount | Cache misses |

### Logging Levels
| Level | What's logged |
|-------|--------------|
| OFF | No logging |
| ERROR | Errors only |
| INFO | Errors + info |

**X-Ray:** Enable for distributed tracing

---

## 14. Security Best Practices

1. **Use HTTPS only** (API Gateway enforces)
2. **Enable WAF** for REST APIs
3. **Use authorizers** (Cognito, Lambda, IAM)
4. **Enable CloudWatch logs** for auditing
5. **Use resource policies** for cross-account access
6. **Validate requests** using models
7. **Use private endpoints** for internal APIs
8. **Set throttling limits** to prevent abuse

---

## 15. Common Interview Questions

### Q: Difference between REST API and HTTP API?
**A:** REST API has more features (caching, WAF, request validation) but HTTP API is ~70% cheaper and lower latency. Use HTTP API for simple Lambda proxies.

### Q: How to handle 504 timeout errors?
**A:** API Gateway has max 29-second timeout. Solutions:
- Optimize backend
- Use async pattern (return 202, process in background)
- Increase Lambda timeout (if applicable)

### Q: How to secure API Gateway?
**A:** Use IAM roles, Cognito User Pools, Lambda Authorizers, API Keys with usage plans, WAF, and resource policies.

### Q: What is Lambda Proxy Integration?
**A:** API Gateway passes entire request (headers, body, path params) to Lambda. Lambda must return specific response format. No mapping templates needed.

### Q: How to implement rate limiting?
**A:** Create Usage Plans with throttle (RPS) and quota (requests/period) settings. Associate API Keys with usage plans.

### Q: How to do blue/green deployment?
**A:** Use stage variables with Lambda aliases, or canary deployments to route traffic percentage.

### Q: What is VPC Link?
**A:** Allows API Gateway to access private resources (EC2, ALB, NLB) inside a VPC securely.

### Q: How to enable CORS?
**A:** Enable CORS on resource, which creates OPTIONS method. For Lambda Proxy, also return CORS headers from Lambda.

### Q: What happens when API is throttled?
**A:** Returns 429 Too Many Requests. Client should implement exponential backoff retry.

### Q: How to share API across accounts?
**A:** Use Resource Policies to allow cross-account access, or use private API with VPC endpoint.

### Q: Does API Gateway support HTTP (non-HTTPS)?
**A:** No. All APIs expose only HTTPS endpoints. Unencrypted HTTP is not supported.

### Q: What is Mock Integration?
**A:** Returns predefined response without calling any backend. Useful for testing or static responses.

### Q: What are the 3 Endpoint Types?
**A:**
- **Edge-optimized:** Routes through CloudFront (global clients)
- **Regional:** Best for clients in same region
- **Private:** Accessible only within VPC via VPC endpoint

### Q: How to implement API versioning?
**A:** Options:
- Path-based: `/v1/users`, `/v2/users`
- Header-based: `X-API-Version: 1`
- Query parameter: `?version=1`
- Stage-based: Different stages for versions

### Q: What is the difference between Stage Variables and Environment Variables?
**A:** Stage variables are API Gateway-specific key-value pairs per stage (dev, prod). They're used in mapping templates, Lambda ARNs, or HTTP endpoints. Environment variables are Lambda-specific.

### Q: How does API Gateway handle authentication vs authorization?
**A:** 
- **Authentication:** Verifies identity (who you are) - Cognito, IAM
- **Authorization:** Verifies permissions (what you can do) - Lambda Authorizer, IAM policies

### Q: What AWS services can API Gateway directly integrate with?
**A:** Lambda, DynamoDB, S3, SNS, SQS, Step Functions, Kinesis, and any HTTP endpoint.

### Q: How to test API Gateway APIs?
**A:** 
- API Gateway console built-in test feature
- Postman or cURL
- AWS CLI
- SDK calls

### Q: What is Request Validation?
**A:** API Gateway can validate requests before sending to backend:
- Validate required parameters
- Validate request body against JSON schema
- Returns 400 Bad Request if validation fails

### Q: What is the difference between Edge-optimized and Regional endpoint?
**A:**
- **Edge-optimized:** Request goes through CloudFront → Lower latency for global users
- **Regional:** Direct to API Gateway → Better for same-region clients, lower latency for regional traffic

---

## 16. WebSocket API

**Use Case:** Real-time, two-way communication (chat apps, dashboards, notifications)

**Key Points:**
- Maintains persistent connection
- Routes messages based on route key in message body
- Callback URL for backend to send messages to client
- Max message size: 128 KB

**Routes:**
- `$connect` - When client connects
- `$disconnect` - When client disconnects
- `$default` - Fallback route
- Custom routes - Based on message content

---

## 17. API Gateway with Lambda - Response Format

For Lambda Proxy Integration, Lambda must return this format:

```javascript
{
    "statusCode": 200,
    "headers": {
        "Content-Type": "application/json"
    },
    "body": "{\"message\": \"success\"}",  // Must be string
    "isBase64Encoded": false
}
```

---

## 18. Common Errors & Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| 403 Forbidden | Missing auth, invalid API key | Check authorizer, API key |
| 429 Too Many Requests | Throttled | Implement retry with backoff |
| 502 Bad Gateway | Lambda error, wrong response format | Check Lambda logs, response format |
| 504 Gateway Timeout | Backend > 29 seconds | Optimize backend, use async |

---

## Quick Reference

```
Endpoint Types:
• Edge-optimized: CloudFront (global clients)
• Regional: Same region clients
• Private: VPC only (via VPC endpoint)

Key Limits:
• 10,000 RPS steady-state
• 5,000 burst
• 29 seconds max timeout
• 10 MB max payload

Common Headers:
• x-api-key: API key
• Authorization: Bearer token
• Content-Type: application/json

URL Format:
https://{api-id}.execute-api.{region}.amazonaws.com/{stage}/{resource}
```

---

*Good luck with your interview!*
