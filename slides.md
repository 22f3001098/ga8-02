---
marp: true
theme: custom-tech
paginate: true
header: 'Product Documentation'
footer: '22f3001098@ds.study.iitm.ac.in'
style: |
  @import 'default';
  
  section {
    background-color: #f5f5f5;
    color: #333;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }
  
  h1 {
    color: #2c3e50;
    border-bottom: 4px solid #3498db;
    padding-bottom: 10px;
  }
  
  h2 {
    color: #34495e;
    border-left: 5px solid #3498db;
    padding-left: 15px;
  }
  
  code {
    background-color: #ecf0f1;
    padding: 2px 6px;
    border-radius: 3px;
    color: #e74c3c;
  }
  
  pre {
    background-color: #2c3e50;
    color: #ecf0f1;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  }
  
  blockquote {
    border-left: 5px solid #3498db;
    padding-left: 20px;
    font-style: italic;
    color: #555;
  }
  
  .columns {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
  }
  
  table {
    border-collapse: collapse;
    width: 100%;
  }
  
  th {
    background-color: #3498db;
    color: white;
    padding: 12px;
  }
  
  td {
    padding: 10px;
    border: 1px solid #ddd;
  }
  
  section.lead {
    text-align: center;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
  }
  
  section.lead h1 {
    color: white;
    border: none;
    font-size: 3em;
  }
---

<!-- _class: lead -->

# Product Documentation
## API Gateway Service v2.0

**Technical Writer**
Email: 22f3001098@ds.study.iitm.ac.in

---

## Table of Contents

1. **System Architecture Overview**
2. **Performance Characteristics**
3. **API Endpoints**
4. **Configuration Guide**
5. **Deployment Instructions**

---

## System Architecture Overview

Our API Gateway provides a unified entry point for microservices with the following features:

- **Rate Limiting**: Token bucket algorithm implementation
- **Load Balancing**: Round-robin with health checks
- **Authentication**: OAuth 2.0 and JWT support
- **Caching**: Distributed Redis-based caching

<div class="columns">

**Benefits:**
- Improved security
- Better performance
- Simplified client integration

**Technologies:**
- Node.js
- Redis
- PostgreSQL
- Docker

</div>

---

## Performance Characteristics

### Algorithmic Complexity

The rate limiting algorithm uses a **Token Bucket** approach:

$$
\text{tokens}_{\text{available}} = \min(\text{capacity}, \text{tokens}_{\text{current}} + \frac{t \times r}{1000})
$$

Where:
- $t$ = time elapsed (ms)
- $r$ = refill rate (tokens/sec)
- Complexity: $O(1)$ per request

### Time Complexity Analysis

| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| Route lookup | $O(\log n)$ | $O(n)$ |
| Cache hit | $O(1)$ | $O(1)$ |
| Auth validation | $O(1)$ | $O(n)$ |

---

<!-- _backgroundImage: url('https://images.unsplash.com/photo-1451187580459-43490279c0fa?w=1200') -->
<!-- _color: white -->

## API Endpoints

### Core Routes
```javascript
// User Management
GET    /api/v2/users
POST   /api/v2/users
PUT    /api/v2/users/:id
DELETE /api/v2/users/:id

// Authentication
POST   /api/v2/auth/login
POST   /api/v2/auth/refresh
POST   /api/v2/auth/logout
```

---

## Configuration Guide

### Environment Variables
```bash
# Server Configuration
PORT=3000
NODE_ENV=production

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=api_gateway

# Redis Cache
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_TTL=3600

# Rate Limiting
RATE_LIMIT_WINDOW=60000
RATE_LIMIT_MAX_REQUESTS=100
```

---

## Rate Limiting Configuration

> **Important**: Configure rate limits based on your traffic patterns and infrastructure capacity.

### Default Settings

| Tier | Requests/min | Burst Size |
|------|--------------|------------|
| Free | 60 | 10 |
| Basic | 600 | 50 |
| Premium | 6000 | 200 |

**Formula for token bucket capacity:**

$$
C = R \times W + B
$$

Where $C$ = capacity, $R$ = rate, $W$ = window, $B$ = burst

---

## Authentication Flow
```python
def authenticate_request(token):
    """
    Validates JWT token and returns user context
    Time Complexity: O(1) average case
    """
    try:
        # Decode and verify token
        payload = jwt.decode(token, SECRET_KEY, 
                            algorithms=['HS256'])
        
        # Check expiration
        if payload['exp'] < time.time():
            raise TokenExpiredError()
        
        return payload['user_id']
    except Exception as e:
        log_error(e)
        return None
```

---

## Deployment Instructions

### Docker Deployment
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
EXPOSE 3000

CMD ["node", "server.js"]
```

### Container Orchestration

- Use **Kubernetes** for production deployments
- Minimum 3 replicas for high availability
- Configure horizontal pod autoscaling

---

## Monitoring & Observability

### Key Metrics to Track

<div class="columns">

**Performance Metrics:**
- Request latency (p50, p95, p99)
- Throughput (req/sec)
- Error rates
- Cache hit ratio

**Business Metrics:**
- API usage by endpoint
- User authentication success rate
- Rate limit violations
- Geographic distribution

</div>

### Mathematical Model for Capacity Planning

$$
\text{Required\_Instances} = \frac{T \times L}{C \times U}
$$

Where $T$ = target throughput, $L$ = latency, $C$ = CPU cores, $U$ = utilization target

---

## Security Best Practices

1. **Enable TLS 1.3** for all communications
2. **Implement JWT rotation** with refresh tokens
3. **Use environment variables** for secrets
4. **Enable CORS** with specific origins only
5. **Implement request signing** for sensitive endpoints
```javascript
// Example: Request signature verification
const crypto = require('crypto');

function verifySignature(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}
```

---

## Troubleshooting Guide

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| High latency | p95 > 1s | Check database queries, enable caching |
| Memory leak | RAM usage climbing | Review event listeners, use profiling tools |
| Rate limit errors | 429 responses | Increase limits or implement backoff |

### Debug Mode

Enable verbose logging:
```bash
DEBUG=api-gateway:* npm start
```

---

<!-- _class: lead -->

# Questions?

## Contact Information

**Email**: 22f3001098@ds.study.iitm.ac.in
**Documentation**: https://docs.example.com
**Support**: https://support.example.com

---

## Appendix: Performance Benchmarks

### Load Test Results

Benchmark configuration: 1000 concurrent users, 60-second duration
```
Requests per second:    2,450 [#/sec] (mean)
Time per request:       408 [ms] (mean)
Time per request:       0.408 [ms] (mean, across concurrent)
Transfer rate:          1,250 [Kbytes/sec]

Connection Times (ms)
              min  mean[+/-sd] median   max
Total:         45  408  125.3    395  1250

Percentage of requests served within time (ms)
  50%    395
  66%    448
  75%    485
  80%    512
  90%    598
  95%    685
  99%    892
 100%   1250
```

---

<!-- _class: lead -->

# Thank You!

**Keep documentation up to date in your repository**

Repository structure:
```
docs/
├── slides.md          # This presentation
├── api-reference.md
├── setup-guide.md
└── troubleshooting.md
```
