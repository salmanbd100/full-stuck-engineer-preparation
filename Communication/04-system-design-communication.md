# System Design Communication

Master the art of articulating system design decisions, trade-offs, and architectures in senior-level technical interviews.

## 📚 Overview

System design interviews test:
- **Architectural thinking:** How you structure complex systems
- **Trade-off analysis:** How you evaluate different approaches
- **Communication clarity:** How you explain technical decisions
- **Collaboration:** How you incorporate feedback
- **Practical knowledge:** Real-world system building experience

**Duration:** 45-60 minutes
**Focus:** 70% communication + 30% technical knowledge

## 🎯 Communication Framework: RADIO

Use the **RADIO framework** to structure your system design discussion:

### R - Requirements Clarification
**A - Architecture (High-level design)**
**D - Data Model (Database design)**
**I - Interface (API design)**
**O - Optimization (Scale & performance)**

---

## 📖 Phase 1: Requirements (5-10 minutes)

### What to Clarify

**Functional Requirements:**
- "What are the core features we need to support?"
- "Should we prioritize X over Y?"
- "Are there any features out of scope for this discussion?"

**Non-Functional Requirements:**
- "What's the expected scale? Daily active users?"
- "What's the read-to-write ratio?"
- "What are the latency requirements? <200ms? <1s?"
- "Do we need strong consistency or is eventual consistency acceptable?"
- "What's the uptime requirement? 99.9%? 99.99%?"

**Technical Constraints:**
- "Are there any specific technologies we must use?"
- "What's the budget constraint?"
- "Are we building from scratch or improving an existing system?"

### Example: Design Twitter

**Strong Clarification:**
```
Interviewer: "Design Twitter."

Candidate: "Great! Let me clarify the requirements to ensure we're aligned.

Functional Requirements:
- Core features: Should we focus on posting tweets, following users, and
  viewing timelines? Or should I also include features like direct
  messages, trending topics, and search?

Interviewer: "Focus on posting tweets and viewing timelines."

Candidate: "Got it. For timelines, should I design both:
- Home timeline (tweets from people you follow)
- User timeline (tweets from a specific user)
Both?"

Interviewer: "Yes, both."

Candidate: "Perfect. Now for non-functional requirements:

Scale:
- How many daily active users? 100M? 500M?
Interviewer: "Let's say 300M DAU."

- How many tweets per day? Let's estimate... if 10% of users tweet, and
  they tweet 2 times on average, that's 60M tweets/day or ~700
  tweets/second. Does that sound right?
Interviewer: "Yes, that's a good estimate."

Read-to-Write Ratio:
- I assume viewing timelines happens much more than posting. Is a 100:1
  read-to-write ratio reasonable?
Interviewer: "Yes."

Latency:
- For timeline loading, should it be under 200ms? 500ms?
Interviewer: "Under 500ms is acceptable."

Consistency:
- If someone posts a tweet, is it okay if followers see it with a few
  seconds delay? Or does it need to appear instantly?
Interviewer: "A few seconds delay is fine - eventual consistency."

Okay, let me summarize what we're building:
- Support 300M daily active users
- Handle ~700 tweets/second
- Provide home timeline (following) and user timeline
- Target <500ms latency for timeline loads
- Eventual consistency is acceptable
- Read-heavy workload (100:1 ratio)

Is that correct?"

Interviewer: "Perfect. Proceed with the design."
```

**Why This Works:**
✅ Asks specific, relevant questions
✅ Makes reasonable assumptions with justification
✅ Confirms understanding before proceeding
✅ Shows estimation skills (back-of-envelope calculations)
✅ Summarizes for alignment

---

## 📖 Phase 2: Architecture (15-20 minutes)

### Communication Structure

1. **Start with high-level overview** (2 min)
2. **Draw main components** (5 min)
3. **Explain data flow** (5 min)
4. **Justify key decisions** (5 min)
5. **Handle questions** (3 min)

### Example: Twitter Architecture

**Step 1: High-Level Overview**
```
"I'll design a scalable tweet and timeline service. At a high level, we need:

1. Client Layer: Web and mobile apps
2. API Gateway: Entry point for all requests
3. Application Services: Tweet Service, Timeline Service, User Service
4. Data Layer: Databases for tweets, user relationships, and caching
5. Storage: Object storage for media (images, videos)

The architecture will be distributed and horizontally scalable to handle
300M users. I'll use microservices for independent scaling and deployment.

Let me draw this out... [draws diagram]"
```

**Step 2: Draw Components** (while narrating)
```
[Draws boxes and arrows on whiteboard]

"Starting from the top:

1. Clients (Web/Mobile) make requests via HTTPS

2. API Gateway (AWS API Gateway or Kong):
   - Routes requests to appropriate services
   - Handles rate limiting (prevent abuse)
   - Authentication/authorization

3. Application Services (Node.js or Go):
   - Tweet Service: Handles posting, editing, deleting tweets
   - Timeline Service: Generates user timelines
   - User Service: Manages user profiles, relationships (following)
   - Each service is independently scalable

4. Load Balancers (AWS ALB):
   - Distribute traffic across service instances
   - Health checks for automatic failover

5. Data Layer:
   - PostgreSQL: User data, relationships
   - Cassandra: Tweet storage (write-optimized, high availability)
   - Redis: Caching for hot timelines
   - Elasticsearch: Tweet search (if we add search later)

6. Object Storage (AWS S3):
   - Media files (images, videos)
   - Served via CloudFront CDN

7. Message Queue (Kafka):
   - Asynchronous processing (timeline generation, notifications)

Does this high-level structure make sense so far?"
```

**Step 3: Explain Data Flow**
```
"Let me walk through two key flows:

Flow 1: Posting a Tweet

1. User posts via mobile app → API Gateway
2. API Gateway authenticates and routes to Tweet Service
3. Tweet Service:
   a. Validates the tweet (length, content)
   b. Writes to Cassandra (tweet storage)
   c. Publishes event to Kafka: "New tweet posted"
4. Timeline Service (consuming Kafka):
   a. Retrieves the user's followers (from User Service)
   b. For each follower, updates their timeline
      - If < 10K followers: Push to Redis cache (fan-out on write)
      - If > 10K followers: Mark timeline as stale (fan-out on read)
5. Returns 201 Created to client

Flow 2: Viewing Timeline

1. User opens app → API Gateway → Timeline Service
2. Timeline Service checks Redis cache:
   - Cache HIT: Return cached timeline (fast path)
   - Cache MISS: Query Cassandra for recent tweets from followed users
3. For each tweet, fetch metadata (likes, retweets) from counters
4. Return paginated response (20 tweets per page)
5. Cache the result in Redis (TTL: 5 minutes)

The key optimization here is that we cache timelines for fast reads, which
aligns with our 100:1 read-to-write ratio."
```

**Step 4: Justify Key Decisions**
```
"Let me explain some architectural decisions:

Decision 1: Cassandra over PostgreSQL for tweets
✅ Pro: Write-optimized (handles 700 tweets/sec easily)
✅ Pro: Horizontal scalability (add nodes as traffic grows)
✅ Pro: High availability (no single point of failure)
❌ Con: Eventual consistency (acceptable per our requirements)
❌ Con: More complex to operate than PostgreSQL
Verdict: Worth it for scale

Decision 2: Hybrid timeline generation (push for most, pull for celebrities)
✅ Pro: Fast reads for 99% of users (pre-generated timelines)
✅ Pro: Doesn't overload system when celebrity tweets (100M followers)
❌ Con: More complex logic
Verdict: Industry standard (used by Twitter, Instagram)

Decision 3: Redis for caching
✅ Pro: Sub-millisecond latency for cache hits
✅ Pro: Reduces database load by 90%+
✅ Pro: TTL support for automatic cache invalidation
❌ Con: Extra infrastructure to manage
❌ Con: Potential for cache/database inconsistency
Verdict: Necessary for 500ms latency requirement

I'd be happy to dive deeper into any of these."
```

---

## 📖 Phase 3: Data Model (10 minutes)

### Database Schema Communication

**Structure:**
1. Identify main entities
2. Show relationships
3. Discuss indexes and partitioning
4. Explain query patterns

### Example: Twitter Data Model

```
"Let me design the data model for our three main entities: Users, Tweets,
and Relationships.

1. Users Table (PostgreSQL):

users
  id          : UUID (primary key)
  username    : VARCHAR(15) (unique, indexed)
  email       : VARCHAR(255) (unique)
  created_at  : TIMESTAMP
  bio         : TEXT
  follower_count  : INT
  following_count : INT

Why PostgreSQL for users?
- Strong consistency needed (can't have duplicate usernames)
- Relatively small data size (<1GB for 300M users)
- ACID transactions for critical operations

2. Tweets Table (Cassandra):

tweets
  id          : UUID (primary key)
  user_id     : UUID
  content     : TEXT
  media_urls  : LIST<TEXT>
  created_at  : TIMESTAMP
  like_count  : INT
  retweet_count : INT

Partitioning Strategy:
- Partition key: user_id
- Clustering key: created_at (descending)
- This allows efficient queries like 'get tweets by user, ordered by time'

Why Cassandra?
- Write-optimized (700 tweets/sec)
- Massive data volume (billions of tweets)
- High availability (multi-datacenter replication)

3. Relationships Table (PostgreSQL):

relationships
  follower_id : UUID (indexed)
  followee_id : UUID (indexed)
  created_at  : TIMESTAMP
  PRIMARY KEY (follower_id, followee_id)

Composite primary key prevents duplicate follows.
Two indexes:
- follower_id: Get 'who am I following'
- followee_id: Get 'who follows me'

For users with >1M followers, we might denormalize this into a separate
'celebrity_followers' table for performance.

4. Timelines Cache (Redis):

Key: user_id:timeline
Value: List of tweet IDs (ordered by time)
TTL: 5 minutes

Structure:
timeline:user123 → [tweet789, tweet788, tweet785, ...]

We store only tweet IDs, then batch-fetch full tweet details from Cassandra.
This keeps cache size small while maintaining fast access.

Query Patterns:
- Get user timeline: SELECT * FROM tweets WHERE user_id = ? ORDER BY
  created_at DESC LIMIT 20
  (Efficient: uses partition key + clustering key)

- Get home timeline: Fetch following list, then batch query tweets from
  each followed user
  (Optimized with Redis cache)

Does this data model make sense?"
```

---

## 📖 Phase 4: Interface (API Design) (5-10 minutes)

### RESTful API Communication

**Structure:**
1. Define main endpoints
2. Specify request/response format
3. Discuss pagination, filtering, errors
4. Explain rate limiting

### Example: Twitter APIs

```
"Let me design the core APIs for our tweet service.

1. POST /api/v1/tweets
   Purpose: Create a new tweet

   Request:
   {
     "content": "Hello world!",
     "media_urls": ["https://cdn.example.com/image1.jpg"]
   }

   Response (201 Created):
   {
     "tweet": {
       "id": "tweet123",
       "user_id": "user456",
       "content": "Hello world!",
       "media_urls": ["..."],
       "created_at": "2024-12-07T10:30:00Z",
       "like_count": 0,
       "retweet_count": 0
     }
   }

   Rate Limit: 50 tweets per hour per user (prevent spam)
   Authentication: Required (OAuth 2.0)

2. GET /api/v1/timeline/home
   Purpose: Get tweets from followed users (home feed)

   Query Parameters:
   - page_size: int (default 20, max 100)
   - cursor: string (for pagination)

   Response (200 OK):
   {
     "tweets": [
       {
         "id": "tweet789",
         "user": { "id": "...", "username": "...", "avatar": "..." },
         "content": "...",
         "created_at": "...",
         "like_count": 42,
         "retweet_count": 10,
         "is_liked_by_me": false
       },
       ...
     ],
     "next_cursor": "eyJpZCI6InR3ZWV0Nzg4IiwidGltZSI6MTYzOTAw...",
     "has_more": true
   }

   Pagination Strategy: Cursor-based (not offset-based)
   Why? Consistent results even when new tweets are added

   Rate Limit: 15 requests per 15 minutes (Twitter's actual limit)

3. GET /api/v1/users/:username/timeline
   Purpose: Get tweets from a specific user

   Response: Similar to home timeline

   Rate Limit: 900 requests per 15 minutes (read-heavy operation)

4. POST /api/v1/tweets/:id/like
   Purpose: Like a tweet

   Response (200 OK):
   {
     "tweet_id": "tweet123",
     "like_count": 43
   }

   Rate Limit: 1000 requests per 15 minutes

Error Handling:
- 400 Bad Request: Invalid input (e.g., tweet too long)
- 401 Unauthorized: Missing/invalid authentication
- 403 Forbidden: Blocked user, suspended account
- 404 Not Found: Tweet doesn't exist
- 429 Too Many Requests: Rate limit exceeded
- 500 Internal Server Error: Server-side issue

All errors return:
{
  "error": {
    "code": "TWEET_TOO_LONG",
    "message": "Tweet exceeds 280 characters",
    "details": {...}
  }
}

API Versioning:
- Version in URL path (/api/v1/, /api/v2/)
- Allows breaking changes without affecting existing clients
- Deprecation policy: 6 months notice before sunsetting old versions

Does this API design make sense?"
```

---

## 📖 Phase 5: Optimization (10-15 minutes)

### Scaling Discussion

**Topics to Cover:**
1. Bottleneck identification
2. Caching strategies
3. Database optimization
4. Load balancing
5. CDN usage
6. Monitoring & metrics

### Example: Scaling to 300M Users

```
"Now let's discuss how to optimize and scale this system.

1. Bottleneck Analysis

The main bottleneck is timeline generation. If a celebrity with 100M
followers tweets, we can't fan-out to all 100M timelines synchronously.

Solution: Hybrid Approach
- Regular users (<10K followers): Fan-out on write (pre-generate timelines)
- Celebrities (>10K followers): Fan-out on read (generate timeline on
  request)

This is how Twitter actually does it. It balances write cost with read cost.

2. Caching Strategy (Multi-Level)

Layer 1: Browser Cache
- Static assets (JS, CSS, images): 1 year cache
- API responses: 30 seconds (for non-real-time data)

Layer 2: CDN (CloudFront)
- Profile pictures, media files: 1 day cache
- Geo-distributed for low latency worldwide

Layer 3: Application Cache (Redis)
- Hot timelines: 5 minutes TTL
- User profiles: 1 hour TTL
- Follower counts: 15 minutes TTL

Layer 4: Database Query Cache
- Cassandra has built-in row cache
- PostgreSQL query cache for frequently accessed data

Expected Impact:
- 90% of timeline requests served from Redis (sub-10ms)
- 9% served from database cache (50-100ms)
- 1% served from cold database queries (200-300ms)
- Avg latency: <50ms (well under 500ms requirement)

3. Database Optimization

PostgreSQL (Users, Relationships):
- Read replicas (3 replicas for read scaling)
- Connection pooling (PgBouncer)
- Indexes on username, email (fast lookups)
- Partitioning relationships table by follower_id (if needed)

Cassandra (Tweets):
- Replication factor: 3 (across 3 data centers)
- Consistency level: Quorum reads, ONE write (eventual consistency)
- SSD storage for low latency
- Compression to reduce storage costs

4. Horizontal Scaling

Application Services:
- Stateless services behind load balancers
- Auto-scaling based on CPU/memory/request rate
  - Normal: 20 instances
  - Peak: 100+ instances
- Kubernetes for orchestration

Database Scaling:
- Cassandra: Add nodes to cluster (linear scalability)
- PostgreSQL: Read replicas + sharding if needed
- Redis: Redis Cluster (sharded across nodes)

5. Load Balancing

Layer 7 Load Balancer (Application LB):
- Distributes requests based on user_id (consistent hashing)
- Health checks: Remove unhealthy instances
- Sticky sessions: Not needed (stateless services)

Geographic Distribution:
- Regions: US-East, US-West, EU-West, Asia-Pacific
- Route users to nearest region (latency optimization)
- Cross-region replication for disaster recovery

6. Monitoring & Observability

Key Metrics:
- Request rate (requests/second)
- Error rate (% 5xx errors)
- Latency (p50, p95, p99)
- Database query time
- Cache hit rate (target: >90%)
- Queue depth (Kafka lag)

Tools:
- Prometheus + Grafana: Metrics and dashboards
- Datadog or New Relic: APM (application performance monitoring)
- ELK Stack: Log aggregation and analysis
- Sentry: Error tracking and alerting

Alerts:
- p99 latency > 1 second: Page on-call engineer
- Error rate > 1%: Page on-call engineer
- Cache hit rate < 80%: Warning
- Database CPU > 80%: Warning

7. Failure Scenarios & Resilience

Scenario 1: Database Failure (PostgreSQL)
- Auto-failover to read replica (promoted to primary)
- Downtime: <30 seconds
- Mitigation: Multi-AZ deployment

Scenario 2: Redis Cache Failure
- Requests fall back to database (degraded performance)
- Auto-recovery: Rebuild cache from database
- Mitigation: Redis Sentinel for automatic failover

Scenario 3: Service Instance Crash
- Load balancer detects failure (health check)
- Routes traffic to healthy instances
- Auto-scaling spins up replacement instance
- Downtime: 0 seconds (no user impact)

Scenario 4: Entire Data Center Failure
- Route traffic to other regions (increased latency)
- Cassandra handles multi-datacenter replication
- Downtime: Minimal (automatic failover)

8. Cost Optimization

Estimated Monthly Cost (AWS):

Compute:
- Application servers (EC2): 50 instances * $200 = $10K
- Auto-scaling peak: Additional $5K during high traffic

Database:
- RDS PostgreSQL (replicas): $5K
- Cassandra cluster (10 nodes): $15K
- Redis cluster: $3K

Storage:
- S3 (media files): $10K (growing over time)
- CloudFront (CDN): $8K

Other:
- Load balancers: $2K
- Monitoring tools: $2K

Total: ~$60K/month

Optimization strategies:
- Use spot instances for non-critical services (50% cost savings)
- Archive old tweets to cheaper storage (S3 Glacier)
- Compress images/videos (reduce S3 costs by 40%)
- Right-size instances (monitoring shows we over-provisioned by 20%)

Estimated optimized cost: ~$45K/month

For 300M DAU, that's $0.00015 per user per month - very reasonable.

Does this cover the optimization and scaling aspects?"
```

---

## 🎤 Key Communication Techniques

### 1. Think Out Loud
❌ Silent for 2 minutes while thinking
✅ "Let me think about the database choice. We need high write throughput
    and horizontal scalability, so I'm leaning towards Cassandra over
    PostgreSQL..."

### 2. Use Signposting
"I'll break this into three parts: first, the API design; second, the
 database schema; and third, how we'll scale it."

### 3. Engage the Interviewer
"Does this approach make sense? Should I dive deeper into the caching
 strategy, or move on to the API design?"

### 4. Acknowledge Trade-offs
"We could use approach A, which gives us X benefit but costs Y. Alternatively,
 approach B is simpler but has Z limitation. For our requirements, I'd
 choose A because..."

### 5. Back with Numbers
❌ "This is fast enough."
✅ "This reduces latency from 800ms to 200ms, well under our 500ms target."

### 6. Use Industry Examples
"This is similar to how Netflix handles video recommendations..."
"Twitter uses a hybrid fan-out approach for this exact reason..."

### 7. Be Honest About Knowledge Gaps
❌ Pretending to know something you don't
✅ "I haven't worked with Kafka directly, but I understand it's a
    distributed event streaming platform. For our use case, I'd use it
    for asynchronous timeline generation. I'd need to research the
    specific configuration for 100K messages/second."

---

## 📊 Practice Exercises

### Exercise 1: Design Instagram
**Functional Requirements:**
- Post photos/videos
- Follow users
- View feed
- Like, comment

**Practice:** Explain for 45 minutes, record yourself

**Evaluation:**
- [ ] Did I clarify requirements first?
- [ ] Did I draw a clear architecture diagram?
- [ ] Did I discuss trade-offs?
- [ ] Did I cover scaling and optimization?
- [ ] Did I engage the interviewer?

### Exercise 2: Design WhatsApp
**Functional Requirements:**
- Send/receive messages (text, media)
- Group chats
- Online/offline status
- Message delivery confirmation

**Practice:** Explain focusing on real-time communication

### Exercise 3: Design YouTube
**Functional Requirements:**
- Upload videos
- Watch videos
- Recommendations
- Comments, likes

**Practice:** Focus on video streaming and CDN

### Exercise 4: Design Uber
**Functional Requirements:**
- Request ride
- Driver matching
- Real-time location tracking
- Payment

**Practice:** Focus on geospatial indexing and matching

---

## ⚠️ Common Mistakes

### Mistake 1: Jumping to Solutions
❌ Immediately drawing architecture without clarifying requirements
✅ Spend 5-10 minutes on requirements first

### Mistake 2: Over-Engineering
❌ "We'll use Kubernetes, Kafka, Redis, Cassandra, Elasticsearch..."
✅ Start simple, scale incrementally: "Let's start with a monolith and
    PostgreSQL, then split into microservices as we scale."

### Mistake 3: No Numbers
❌ "We need a fast database."
✅ "We need to handle 10,000 writes/second with <50ms latency."

### Mistake 4: Ignoring Trade-offs
❌ "Cassandra is the best choice."
✅ "Cassandra handles high writes well but has eventual consistency, which
    is acceptable for our use case."

### Mistake 5: Not Checking Understanding
❌ Monologuing for 15 minutes without pause
✅ Pause every 5 minutes: "Does this make sense? Any questions?"

---

## ✅ Pre-Interview Checklist

**2 Weeks Before:**
- [ ] Practice 5-10 common system designs
- [ ] Study scaling patterns (caching, sharding, replication)
- [ ] Learn database trade-offs (SQL vs NoSQL)
- [ ] Understand CAP theorem, consistency models

**1 Week Before:**
- [ ] Do 3-5 mock system design interviews
- [ ] Review common architectures (Twitter, Instagram, YouTube)
- [ ] Practice drawing clean diagrams quickly
- [ ] Prepare questions about company's architecture

**Day Before:**
- [ ] Review RADIO framework
- [ ] Practice back-of-envelope calculations
- [ ] Prepare your own questions for the interviewer
- [ ] Get good sleep!

---

**Related:** [Technical Communication](./01-technical-communication.md) | [Problem-Solving Communication](./05-problem-solving-communication.md) | [Presentation Skills](./07-presentation-skills.md)

---

**Think out loud. Draw clearly. Justify decisions.** 🎯
