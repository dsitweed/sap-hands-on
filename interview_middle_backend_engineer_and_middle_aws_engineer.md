# Danh sách câu hỏi phỏng vấn chuyên sâu - Backend Engineer & AWS Engineer

---

## PHẦN 1: BACKEND ENGINEER - CÂU HỎI CHUYÊN SÂU

### 1.1. Kiến trúc hệ thống (System Design)

**Câu 1: Hãy thiết kế hệ thống URL Shortener như bit.ly. Hãy đi sâu vào các thành phần, lưu lượng, và cách mở rộng.**

**Trả lời:**
- **Yêu cầu**: Tạo URL ngắn từ URL dài, chuyển hướng nhanh, xử lý hàng triệu lượt truy cập/ngày.
- **Components**:
  - **API Gateway**: Nhận request, xác thực, rate limiting
  - **Application Server**: Xử lý logic tạo ngắn, lưu vào DB
  - **Database**: PostgreSQL/MySQL lưu mapping (short_url → long_url)
  - **Cache**: Redis cho hot URLs (đa số traffic truy cập vài URL phổ biến)
  - **CDN**: Phân phối redirect ở biên
- **Tạo short URL**: Sử dụng Base62 encoding của auto-increment ID hoặc hash (MD5/SHA) của URL gốc. Ví dụ: ID 123456 → "abc123"
- **Scaling**: Sharding DB theo prefix của short URL, read replicas cho read-heavy workload
- **KPI**: 99.99% uptime, latency <50ms cho redirect
- **Trade-off**: Eventual consistency chấp nhận được cho URL shortener

**Câu 2: Hãy thiết kế hệ thống notification (push, email, SMS) cho một nền tảng thương mại điện tử.**

**Trả lời:**
- **Kiến trúc Event-Driven**: User action → Event Producer (Kafka/RabbitMQ) → Notification Service → Consumer → Delivery
- **Components**:
  - **Event Bus**: Apache Kafka để xử lý hàng triệu events/giây
  - **Notification Service**: Consumer, quyết định channel (push/email/SMS)
  - **Template Engine**: Handlebars/Mustache cho email/SMS templates
  - **Delivery Service**: AWS SNS (push), SES (email), SQS + Twilio (SMS)
  - **Dashboard**: Theo dõi delivery rate, open rate, click rate
- **Retry Logic**: Exponential backoff cho failed deliveries
- **Deduplication**: Sử dụng message ID để tránh gửi trùng
- **Priority**: Transactional notifications (order confirm) > Marketing notifications
- **Scalability**: Horizontal scaling của consumer groups dựa trên lag của Kafka topic

**Câu 3: Giải thích sự khác nhau giữa Database Sharding, Replication, và Partitioning. Khi nào sử dụng cái nào?**

**Trả lời:**
- **Sharding**: Phân chia dữ liệu ngang hàng (horizontal) qua nhiều DB server. Mỗi shard là một DB riêng biệt. Key-based sharding (consistent hashing) hoặc range-based sharding.
- **Replication**: Tạo bản sao dữ liệu trên nhiều server. Master-slave (read replicas) hoặc multi-master. Mục tiêu: availability, read scalability, disaster recovery.
- **Partitioning**: Chia bảng thành các phần nhỏ hơn trong cùng một DB instance. PostgreSQL declarative partitioning, MySQL partitioning.
- **Khi nào dùng gì**:
  - Sharding: Khi dữ liệu vượt quá capacity của single DB
  - Replication: Khi cần read scalability và HA
  - Partitioning: Khi cần manage large tables tốt hơn (dễ archive, backup)
- **Challenge của Sharding**: Cross-shard queries, rebalancing, distributed transactions

---

### 1.2. Concurrency & Multithreading

**Câu 4: Hãy giải thích sự khác nhau giữa Race Condition, Deadlock, Livelock, và Starvation. Cho ví dụ thực tế.**

**Trả lời:**
- **Race Condition**: Kết quả phụ thuộc vào thứ tự thực thi của threads. Ví dụ: hai threads cùng cập nhật balance → kết quả sai. Giải pháp: mutex, semaphore, atomic operations.
- **Deadlock**: Hai hoặc nhiều threads chờ nhau giải phóng resource. 4 điều kiện cần: Mutual Exclusion, Hold and Wait, No Preemption, Circular Wait. Giải pháp: timeout, lock ordering, deadlock detection.
- **Livelock**: Threads liên tục thay đổi state để phản hồi lẫn nhau nhưng không tiến triển. Ví dụ: hai người trong hành lang đều nhường nhau qua lại. Giải pháp: random backoff.
- **Starvation**: Thread không bao giờ được cấp resource vì các thread khác chiếm hết. Ví dụ: priority inversion. Giải pháp: fair locking, aging.

**Câu 5: Giải thích cách hoạt động của Java Memory Model và tại sao `volatile` quan trọng.**

**Trả lời:**
- **Java Memory Model (JMM)**: Định nghĩa cách các thread tương tác với memory. Có hai types của memory: Main Memory (heap) và Working Memory (thread-local cache/registers).
- **Problem**: Thread A cập nhật biến trong working memory, Thread B đọc từ main memory → giá trị cũ.
- **`volatile`**: Đảm bảo:
  1. **Visibility**: Khi một thread ghi `volatile` biến, giá trị được flush ngay vào main memory. Khi đọc, load từ main memory.
  2. **Ordering**: Ngăn compiler và CPU reorder các instruction xung quanh volatile (memory barrier/fence).
- **Không đủ `volatile` cho atomicity**: `volatile` không đảm bảo atomic increment (read-modify-write). Cần `AtomicInteger` hoặc `synchronized`.
- **Happens-Before**: `volatile` write happens-before `volatile` read.

**Câu 6: Explain the difference between Process, Thread, and Coroutine. When would you use each?**

**Trả lời:**
- **Process**: Đơn vị cấp phát resource độc lập. Có memory space riêng. Giao tiếp qua IPC (pipe, socket, shared memory). Heavy context switch.
- **Thread**: Đơn vị thực thi trong process. Chia sẻ memory space với các threads khác. Lighter context switch. Giao tiếp qua shared memory (cần synchronization).
- **Coroutine (User-level thread)**: Lightweight thread managed by runtime, không bởi OS. Context switch ở user space (rất nhanh). Không cần OS scheduling. Async/await pattern.
- **When to use**:
  - Process: Isolation, CPU-bound tasks needing multiple cores
  - Thread: I/O-bound tasks sharing data within same process
  - Coroutine: Massive I/O concurrency (e.g., 10k+ connections) with minimal overhead
  - Example: Go goroutines, Python asyncio, Kotlin coroutines

---

### 1.3. Database & SQL

**Câu 7: Hãy giải thích chi tiết về Database Indexes - B-Tree vs Hash Index vs Full-Text Index. Khi nào dùng loại nào?**

**Trả lời:**
- **B-Tree Index**:
  - Cấu trúc cây cân bằng, dữ liệu lưu sorted
  - Hỗ trợ range queries (`WHERE age > 25 AND age < 40`)
  - Độ phức tạp: O(log n)
  - Phù hợp: hầu hết các trường hợp, đặc biệt range queries, ORDER BY
  - Ví dụ: PostgreSQL default, MySQL InnoDB default
- **Hash Index**:
  - Sử dụng hash table, chỉ hỗ trợ equality search (`WHERE id = 123`)
  - O(1) lookup nhưng không hỗ trợ range queries
  - Không hỗ trợ ORDER BY, không hỗ trợ partial match
  - Ví dụ: PostgreSQL hash index, MySQL MEMORY tables
  - Crash-safe hash indexes có trong PostgreSQL 10+
- **Full-Text Index**:
  - Tạo inverted index cho text search
  - Hỗ trợ search từ khóa, ranking, stemming
  - Sử dụng TF-IDF hoặc BM25 algorithm
  - Ví dụ: MySQL FULLTEXT, PostgreSQL `tsvector`, Elasticsearch (external)
- **Composite Index**: Nhiều cột, tuân theo leftmost prefix rule
- **Covering Index**: Index chứa tất cả columns cần thiết cho query → không cần lookup vào table
- **Clustered vs Non-Clustered**: Clustered index (data stored in index order) vs Non-clustered (pointer to data)

**Câu 8: Giải thích về ACID, BASE, và sự trade-off giữa chúng.**

**Trả lời:**
- **ACID** (Traditional RDBMS):
  - **Atomicity**: Transaction all-or-nothing
  - **Consistency**: Database luôn ở valid state
  - **Isolation**: Concurrent transactions không ảnh hưởng nhau (levels: Read Uncommitted, Read Committed, Repeatable Read, Serializable)
  - **Durability**: Committed data persists even after crash
- **BASE** (NoSQL):
  - **Basically Available**: System always responds
  - **Soft State**: State may change over time without input
  - **Eventually Consistent**: Data will become consistent eventually
- **Trade-off**:
  - ACID → Consistency, nhưng availability và partition tolerance thấp hơn
  - BASE → Availability và scalability cao hơn, nhưng đánh đổi consistency
  - **CAP Theorem**: Trong distributed system, chỉ chọn được 2 trong 3: Consistency, Availability, Partition Tolerance
  - **Practical approach**: Most modern systems use hybrid - strong consistency cho critical data, eventual consistency cho non-critical

**Câu 9: Query Optimization - Hãy giải thích cách đọc và tối ưu một query慢 (slow query) trong MySQL/PostgreSQL.**

**Trả lời:**
- **Bước 1: Phân tích với EXPLAIN/EXPLAIN ANALYZE**
  - Xem execution plan: type (ALL, index, range, ref, eq_ref), rows scanned, key used, filtered %, Extra
  - `type: ALL` = Full table scan (bad), `type: ref` hoặc `range` = good
  - `rows` = estimated rows examined
- **Bước 2: Xác định vấn đề**
  - Full table scan → thiếu index
  - Using filesort → ORDER BY không dùng index
  - Using temporary → GROUP BY/DISTINCT cần temporary table
  - High rows examined → index không optimal hoặc query viết sai
- **Bước 3: Tối ưu**
  - Thêm composite index cho WHERE + ORDER BY + SELECT columns
  - Rewrite query để tránh subquery (dùng JOIN thay thế)
  - Sử dụng covering index
  - Phân trang với cursor-based pagination thay vì OFFSET (dễ degraded với large offsets)
  - Denormalize cho read-heavy queries
  - Use connection pooling
- **Bước 4: Monitor**
  - Slow query log
  - Performance Schema (MySQL) / pg_stat_statements (PostgreSQL)

---

### 1.4. API Design & Microservices

**Câu 10: Hãy giải thích sự khác nhau giữa REST, GraphQL, gRPC, và WebSocket. Khi nào dùng cái nào?**

**Trả lời:**
- **REST (Representational State Transfer)**:
  - Stateless, resource-based, HTTP methods (GET/POST/PUT/DELETE)
  - JSON/XML payloads
  - Easy to cache, simple, widely adopted
  - Over-fetching/under-fetching problem (client gets too much or too little data)
  - Best for: CRUD operations, public APIs, simple client-server interactions
- **GraphQL**:
  - Query language, single endpoint, client-defined queries
  - No over/under-fetching, strong typing (schema)
  - N+1 problem (solved with DataLoader/batching)
  - Complex caching (no HTTP caching built-in)
  - Best for: Complex frontend with diverse data needs, mobile apps with limited bandwidth
- **gRPC**:
  - Uses Protocol Buffers (binary), HTTP/2, strongly typed
  - Bidirectional streaming, low latency, high throughput
  - Code generation (client/server stubs)
  - Not browser-friendly (needs gRPC-web proxy)
  - Best for: Service-to-service communication, microservices internal APIs, real-time streaming
- **WebSocket**:
  - Full-duplex, persistent connection over TCP
  - Real-time bidirectional communication
  - No request-response pattern (server can push anytime)
  - Best for: Chat, live updates, gaming, stock tickers
- **Practical choice**: REST cho external APIs, gRPC cho internal microservices, GraphQL cho complex frontend data needs, WebSocket cho real-time features

**Câu 11: Giải thích cách thiết kế API versioning tốt. Các phương pháp và trade-off.**

**Trả lời:**
- **URL Path Versioning**: `/api/v1/users` - Simple, explicit, easy to debug. Downside: URL pollution, hard to deprecate.
- **Header Versioning**: `Accept: application/vnd.myapi.v1+json` - Clean URL, RESTful. Downside: Not visible in URL, harder to test.
- **Query Parameter Versioning**: `/api/users?version=1` - Simple but violates REST principles.
- **Best practices**:
  - Semantic versioning (MAJOR.MINOR.PATCH)
  - MAJOR version change = breaking changes
  - MINOR version = backward-compatible additions
  - PATCH = bug fixes
  - Deprecation policy: Announce 6-12 months before sunset
  - Maintain at least 2 versions simultaneously
  - Use API gateway to route versions

---

### 1.5. Caching & Performance

**Câu 12: Giải thích các loại cache invalidate strategies và cache invalidation challenge.**

**Trả lời:**
- **Cache Invalidation Strategies**:
  - **Write-through**: Write to cache and DB simultaneously. Data always consistent. Slower writes.
  - **Write-back (Write-behind)**: Write to cache only, async flush to DB. Faster writes, risk data loss.
  - **Write-around**: Write to DB only, cache populated on read. Simple, but cache miss on first read after write.
  - **Cache-aside (Lazy loading)**: Application checks cache first, if miss then load from DB and populate cache. Most common.
- **Challenges**:
  - **Cache Stampede/Thundering Herd**: Multiple requests miss cache simultaneously, all hit DB. Solution: Lock/mutex on cache miss, background refresh.
  - **Cache Penetration**: Request for non-existent data. Solution: Bloom filter, cache null values with short TTL.
  - **Cache Avalanche**: Many keys expire simultaneously. Solution: Randomized TTL, distributed cache with different TTLs.
  - **Stale Data**: Cache has old data. Solution: TTL, event-driven invalidation.
- **Advanced**: Read-through, Write-behind with write queue

**Câu 13: Giải thích về CDN (Content Delivery Network) và cách nó hoạt động với edge caching.**

**Trả lời:**
- **CDN Architecture**:
  - **Origin Server**: Nơi chứa dữ liệu gốc
  - **Edge Server/Point of Presence (PoP)**: Máy chủ gần người dùng nhất, lưu cache副本
  - **DNS Layer**: Route user request to nearest PoP
- **How it works**:
  1. User requests resource → DNS resolves to nearest edge server
  2. Edge server checks cache → HIT (return cached) or MISS (fetch from origin)
  3. On miss, edge server fetches from origin, caches, and returns to user
  4. Subsequent requests for same resource served from edge
- **Cache Control**: `Cache-Control` headers (max-age, s-maxage, no-cache, no-store), `ETag`, `Last-Modified`
- **Benefits**: Reduced latency, reduced origin load, improved availability, DDoS protection
- **Dynamic content**: Can also cache dynamic content using edge compute (CloudWorkers, Lambda@Edge)

---

### 1.6. Security

**Câu 14: Giải thích chi tiết về JWT (JSON Web Token) - cách hoạt động, lợi ích, và lỗ hổng bảo mật.**

**Trả lời:**
- **Structure**: Header (algorithm, token type) + Payload (claims) + Signature
  - Header: `{"alg": "HS256", "typ": "JWT"}`
  - Payload: `{"sub": "1234567890", "name": "John", "iat": 1516239022}`
  - Signature: `HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)`
- **How it works**:
  1. User login → Server creates JWT, signs with secret key
  2. Client stores token (localStorage or cookie)
  3. Client sends token in `Authorization: Bearer <token>` header
  4. Server verifies signature, checks expiration, grants access
- **Vulnerabilities**:
  - **XSS**: If stored in localStorage, attacker can steal via XSS. Mitigation: HttpOnly cookies, CSP
  - **CSRF**: If stored in cookie. Mitigation: CSRF tokens, SameSite cookie attribute
  - **Token theft**: No built-in revocation mechanism. Mitigation: Short-lived access tokens + refresh tokens, blacklist
  - **Algorithm confusion**: Accepting `none` algorithm or switching RS256→HS256. Mitigation: Explicitly specify expected algorithm
- **Best practices**:
  - Short expiration (15 min for access token)
  - Refresh token stored securely in DB, rotated on use
  - Don't store sensitive data in payload (it's base64 encoded, not encrypted)
  - Use strong secret (HS256) or asymmetric keys (RS256)

**Câu 15: Giải thích về OAuth 2.0 và OpenID Connect (OIDC). Sự khác nhau và khi nào dùng gì.**

**Trả lời:**
- **OAuth 2.0**: Protocol for **authorization** (cho phép ứng dụng truy cập tài nguyên thay mặt user). Không xử lý authentication.
  - **Grant Types**: Authorization Code, Client Credentials, Implicit, Resource Owner Password Credentials, Refresh Token
  - **Roles**: Resource Owner, Client, Authorization Server, Resource Server
  - **Flow**: User → Authorization Server (login/consent) → Authorization Code → Client exchanges code for Access Token → Access Resource
- **OpenID Connect (OIDC)**: Authentication layer trên OAuth 2.0. Thêm ID Token (JWT) chứa user identity info.
  - ID Token contains: sub (user ID), name, email, picture, auth_time, etc.
  - Uses the same authorization flow but returns an ID Token in addition to Access Token
- **Difference**: OAuth = "Who can access what?" Authorization. OIDC = "Who are you?" Authentication.
- **Use cases**: OAuth for API access (third-party app accessing user data), OIDC for user login (SSO, "Login with Google")
- **Flows in OIDC**: Authorization Code Flow (web), PKCE Extension (mobile/SPA), Implicit Flow (deprecated), Hybrid Flow

---

### 1.7. Message Queue & Asynchronous Processing

**Câu 16: Giải thích sự khác nhau giữa Message Queue và Event Streaming. Khi nào dùng cái nào?**

**Trả lời:**
- **Message Queue (e.g., RabbitMQ, AWS SQS)**:
  - Messages consumed once (load balancing)
  - Message deleted after consumption
  - Point-to-point or pub/sub with single delivery
  - Good for: Task queues, job processing, reliable delivery guarantees
  - No message replay (unless using DLQ)
- **Event Streaming (e.g., Apache Kafka)**:
  - Event log with retention (messages persisted for configurable period)
  - Multiple consumers can read independently at different offsets
  - Message replay possible
  - Event sourcing, CQRS patterns
  - Good for: Event sourcing, audit trails, real-time analytics, data pipelines
- **Decision matrix**:
  - Need message replay? → Kafka
  - Simple queue with guaranteed delivery? → SQS/RabbitMQ
  - High throughput (millions/sec)? → Kafka
  - Complex routing? → RabbitMQ
  - Need exactly-once semantics? → Kafka with idempotent producers

**Câu 17: Giải thích về Dead Letter Queue (DLQ) và retry mechanisms.**

**Trả lời:**
- **DLQ**: Queue lưu các messages không thể xử lý thành công sau nhiều lần retry.
- **Retry strategies**:
  - **Immediate retry**: Simple but can overwhelm system
  - **Exponential backoff**: Wait time doubles with each retry (1s, 2s, 4s, 8s...)
  - **Exponential backoff with jitter**: Add randomness to prevent thundering herd
  - **Dead letter handling**: After max retries, move to DLQ for manual inspection
- **Dead letter process**:
  1. Message sent to main queue
  2. Consumer fails to process
  3. Message retried N times (configurable)
  4. After N failures, message routed to DLQ
  5. Admin inspects DLQ, fixes issue, reprocesses or discards
- **Poison messages**: Messages causing persistent failures. Identify and isolate.
- **Monitoring**: DLQ depth alert, retry rate metrics, processing time P99

---

### 1.8. Kubernetes & Container Orchestration

**Câu 18: Giải thích sự khác nhau giữa Deployment, StatefulSet, DaemonSet, và Job trong Kubernetes.**

**Trả lời:**
- **Deployment**:
  - Manages stateless applications
  - ReplicaSet ensures desired number of replicas
  - Rolling updates, rollback support
  - Use for: Web servers, API services, any stateless workload
- **StatefulSet**:
  - Manages stateful applications
  - Each pod has stable, unique network identity (pod-0, pod-1)
  - Persistent volume claims (PVCs) tied to pod identity
  - Ordered deployment/scaling/deletion
  - Use for: Databases (MySQL, PostgreSQL), distributed systems (ZooKeeper, Kafka)
- **DaemonSet**:
  - Ensures one (or some) pod runs on each node
  - Automatically schedules pod on new nodes
  - Use for: Monitoring agents (Prometheus node exporter), log collectors (Fluentd), network plugins (CNI)
- **Job/CronJob**:
  - Runs to completion (not long-running)
  - Job: Run to completion once
  - CronJob: Schedule-based execution (cron syntax)
  - Use for: Batch processing, data migration, scheduled reports
- **Key decisions**:
  - Stateless + multiple replicas → Deployment
  - Stateful with stable identity → StatefulSet
  - One per node → DaemonSet
  - One-time/scheduled → Job/CronJob

---

### 1.9. Design Patterns

**Câu 19: Giải thích Repository Pattern và Unit of Work Pattern. Tại sao và khi nào dùng?**

**Trả lời:**
- **Repository Pattern**:
  - Abstracts data access layer
  - Provides collection-like interface for domain objects
  - Decouples business logic from data access technology (EF Core, Entity Framework, etc.)
  - Enables unit testing with mock repositories
  - Example: `IUserRepository.GetById(id)`, `IUserRepository.Add(user)`
- **Unit of Work Pattern**:
  - Tracks all changes made during a business transaction
  - Ensures all changes are committed or rolled back atomically
  - Maintains a list of dirty objects, coordinates write-back
  - In EF Core: `DbContext` acts as Unit of Work
  - `_context.SaveChanges()` commits all changes in single transaction
- **Together**: Repository handles data access, UoW manages transactions
- **When to use**: Complex applications with multiple data sources, need for testability, DDD (Domain-Driven Design)

**Câu 20: Giải thích Circuit Breaker Pattern và ứng dụng thực tế.**

**Trả lời:**
- **Problem**: When a service is down, all requests to it will timeout, consuming resources and causing cascading failures (avalanche effect).
- **Circuit Breaker States**:
  - **Closed**: Normal operation, requests flow through
  - **Open**: Failure threshold reached, requests fail fast (no call to downstream service)
  - **Half-Open**: After timeout, allow a few requests to test if service recovered
- **Implementation**:
  - Track failure count, timeout duration, success threshold
  - Libraries: Polly (.NET), Resilience4j (Java), Hystrix (deprecated)
- **Example**: Payment service timeout 5s, failure threshold 5, timeout 30s
  - If 5 failures in 30s → Open circuit
  - After 30s → Half-Open → 1 request succeeds → Close circuit
  - If 1 request fails → Open again for another 30s
- **Complementary patterns**:
  - **Fallback**: Return cached data or default response when circuit is open
  - **Bulkhead**: Isolate thread pools per service to prevent resource exhaustion
  - **Rate Limiter**: Limit requests per second to downstream services

---

## PHẦN 2: AWS ENGINEER - CÂU HỎI CHUYÊN SÂU

### 2.1. Compute Services

**Câu 21: Giải thích sự khác nhau giữa EC2, Lambda, Fargate, và ECS/EKS. Khi nào dùng cái nào?**

**Trả lời:**
- **Amazon EC2**:
  - Virtual machines (IaaS), full control over OS, networking, security
  - Pay for compute capacity by the hour/second
  - Best for: Long-running processes, custom OS requirements, stateful workloads, legacy applications
  - Management overhead: Patch OS, scale manually or with Auto Scaling
- **AWS Lambda**:
  - Serverless functions (FaaS), pay per execution
  - No server management, auto-scales to zero
  - Max execution time: 15 minutes
  - Best for: Event-driven tasks, API backends, data processing, cron jobs
  - Cold start latency (100ms-2s depending on runtime/language)
  - Limitations: Stateless, 10-minute max, vendor lock-in concerns
- **AWS Fargate**:
  - Serverless compute engine for containers (ECS/EKS)
  - No need to manage EC2 instances
  - Define CPU/memory requirements, Fargate handles infrastructure
  - Best for: Containerized applications without EC2 management overhead
  - Pay per vCPU/memory consumed per second
- **Amazon ECS (Elastic Container Service)**:
  - Container orchestration service (AWS native)
  - Can use EC2 launch type (manage EC2) or Fargate (serverless)
  - Task definitions, services, task scheduling
- **Amazon EKS (Elastic Kubernetes Service)**:
  - Managed Kubernetes (K8s) control plane
  - Industry standard for container orchestration
  - Best for: Organizations already invested in K8s ecosystem
- **Decision**:
  - Simple event-driven, unpredictable traffic → Lambda
  - Containers without managing servers → Fargate
  - Kubernetes expertise, complex orchestration → EKS
  - Full control, custom networking → EC2
  - Need persistent state on compute → EC2

**Câu 22: Giải thích cách Auto Scaling hoạt động trên EC2 với target tracking, step scaling, và predictive scaling.**

**Trả lời:**
- **Target Tracking Scaling**:
  - Maintain a specific metric at a target value
  - Example: Keep CPU utilization at 50%
  - If CPU > 50% → add instances, if CPU < 50% → remove instances
  - Simple, automatic adjustment
- **Step Scaling**:
  - Define scaling adjustments based on CloudWatch alarm thresholds
  - Example: CPU > 70% → add 2 instances, CPU > 85% → add 4 instances
  - More granular control than target tracking
  - Can define multiple step adjustments
- **Predictive Scaling**:
  - Uses ML to forecast traffic based on historical patterns
  - Proactively scales before traffic increases
  - Combines with dynamic scaling for immediate response
  - Best for: Predictable traffic patterns (daily/weekly cycles)
- **Scaling Policies Details**:
  - **Cooldown period**: Prevent rapid scaling fluctuations (default 300s)
  - **Min/Max capacity**: Safety bounds
  - **Scaling adjustments**: Absolute count or percentage change
  - **Instance warm-up**: Time before new instance counted toward metrics
- **Practical considerations**:
  - Use multiple metrics (CPU + request count + queue depth)
  - Consider scaling based on custom application metrics
  - Use mixed instances policy for cost optimization (spot + on-demand)

---

### 2.2. Networking

**Câu 23: Giải thích chi tiết về VPC (Virtual Private Cloud) - subnets, route tables, NACLs, Security Groups. Khi nào dùng public vs private subnet.**

**Trả lời:**
- **VPC**: Logically isolated virtual network in AWS. Custom IP range (CIDR block, e.g., 10.0.0.0/16).
- **Subnets**:
  - **Public Subnet**: Has route to Internet Gateway (IGW). Resources can have public IP. Use for: Load balancers, NAT gateways, bastion hosts, public-facing web servers.
  - **Private Subnet**: No direct route to IGW. Resources accessed via NAT gateway/instance or VPN. Use for: Application servers, databases, internal services.
  - Best practice: Multi-AZ deployment across at least 2 public + 2 private subnets
- **Internet Gateway (IGW)**: Horizontally scalable, redundant connection between VPC and Internet. Attach to VPC for public subnets.
- **NAT Gateway**: Allows resources in private subnet to access internet (for updates, downloads) without exposing them. One per AZ (or cross-AZ NAT gateway).
- **Route Tables**:
  - Rules-based routing (destination CIDR → target)
  - Main route table (automatically associated with all subnets not explicitly associated)
  - Each subnet can be associated with only one route table
  - Example: `0.0.0.0/0 → igw-xxxx` (public), `0.0.0.0/0 → nat-xxxx` (private)
- **Security Groups (SG)**:
  - Stateful firewall at instance level
  - Allow rules only (no explicit deny)
  - Return traffic automatically allowed
  - Apply to ENI/network interfaces
  - Can reference other SGs
- **Network ACLs (NACL)**:
  - Stateless firewall at subnet level
  - Both allow AND deny rules
  - Rules evaluated by number (lowest first)
  - Must explicitly allow return traffic
  - Best practice: Use SG for fine-grained, NACL for coarse-grained subnet-level control
- **Classic Architecture**:
  - Public subnets: ALB/NLB → Web servers (public)
  - Private subnets: Application servers → Databases (private)
  - NAT Gateway in public subnet for outbound internet from private subnets

**Câu 24: Giải thích VPC Peering, Transit Gateway, và VPN CloudHub. Khi nào dùng từng loại?**

**Trả lời:**
- **VPC Peering**:
  - Direct connection between 2 VPCs using private IP
  - Transitive peering NOT supported (A↔B, B↔C doesn't mean A↔C)
  - Max 125 peering connections per VPC
  - No bandwidth limit (uses AWS backbone)
  - Use for: Simple connections between 2 VPCs in same/different regions
  - Cost: Per-hour per connection + data transfer
- **Transit Gateway (TGW)**:
  - Hub-and-spoke model - central router connecting multiple VPCs, VPN, Direct Connect
  - Transitive routing: All attached networks can communicate
  - Scales to thousands of attachments
  - Supports cross-region peering
  - Use for: Complex multi-VPC architectures, hybrid cloud, hub-and-spoke patterns
  - Best for: Large organizations with many VPCs
- **VPN CloudHub**:
  - Uses Virtual Private Gateway with multiple Site-to-Site VPN tunnels
  - Spoke-to-spoke communication via hub VPC
  - Each branch office connects via VPN to central hub
  - Use for: Branch office connectivity when dedicated lines aren't available
- **Decision**:
  - 2 VPCs simple connection → VPC Peering
  - Many VPCs, central management → Transit Gateway
  - On-premises branch offices → VPN CloudHub or Direct Connect

---

### 2.3. Storage

**Câu 25: Giải thích toàn diện về Amazon S3 - storage classes, lifecycle policies, encryption, và versioning.**

**Trả lời:**
- **Storage Classes**:
  - **S3 Standard**: Frequently accessed data, low latency, high throughput. Same-region replication.
  - **S3 Intelligent-Tiering**: Automatically moves objects between frequent and infrequent access tiers. Best for unknown access patterns.
  - **S3 Standard-IA (Infrequent Access)**: Lower storage cost, higher retrieval cost. 30-day minimum storage duration.
  - **S3 One Zone-IA**: Same as Standard-IA but stored in single AZ. Lower cost, but no cross-AZ durability.
  - **S3 Glacier Instant Retrieval**: Archived data needing millisecond retrieval. 90-day minimum.
  - **S3 Glacier Flexible Retrieval**: Archive with retrieval in minutes to hours. 90-day minimum.
  - **S3 Glacier Deep Archive**: Lowest cost, retrieval in 12-48 hours. 180-day minimum.
- **Lifecycle Policies**: Automate transitions between storage classes.
  - Example: Move to Standard-IA after 30 days → Glacier after 90 days → Delete after 365 days
  - Can define rules based on object age, prefix, tags
- **Encryption**:
  - **SSE-S3**: Amazon S3 manages keys (AES-256). Simplest.
  - **SSE-KMS**: AWS Key Management Service manages keys. Audit trail, fine-grained access control.
  - **SSE-C**: Customer provides encryption keys. S3 never stores your key.
  - **Client-Side Encryption**: Encrypt before uploading (envelope encryption with KMS or custom).
  - **Bucket Policy**: Enforce encryption in-transit (`aws:SecureTransport`) and at-rest (`s3:x-amz-server-side-encryption`)
- **Versioning**:
  - Keeps all versions of every object
  - Protects against accidental overwrite/delete
  - MFA delete option for additional protection
  - Lifecycle policies can expire non-current versions
  - **Delete markers**: Soft delete when versioning is enabled (restore by deleting the delete marker)
- **Important**: S3 is 11 nines (99.999999999%) durability, 99.99% availability

**Câu 26: Giải thích sự khác nhau giữa EFS, FSx, EBS, và Instance Store. Khi nào dùng gì?**

**Trả lời:**
- **Amazon EBS (Elastic Block Store)**:
  - Block-level storage for EC2 instances
  - Volume types: gp3 (general purpose), io2 (provisioned IOPS), sc1/c1 (throughput optimized)
  - Single-attach (one instance) or multi-attach (io1/io2, same AZ only)
  - Snapshots to S3 for backup
  - Persistent across instance stop/start
  - Use for: Boot volumes, databases, file systems on single instance
- **Amazon EFS (Elastic File System)**:
  - NFS file system, shared across multiple EC2 instances
  - Automatically scales up and down
  - Multi-AZ (data replicated across AZs)
  - Performance modes: General Purpose (latency optimized), Max I/O (throughput optimized)
  - Life Cycle Management: Infrequent Access tier
  - Use for: Shared file storage, CMS, web serving, content management
  - Not suitable for Windows (use FSx for Windows)
- **Amazon FSx**:
  - **FSx for Lustre**: High-performance file system for ML, HPC, video processing. Integrates with S3.
  - **FSx for Windows File Server**: Fully managed Windows file server with Active Directory integration.
  - **FSx for ONTAP**: NetApp ONTAP-based, supports SMB/NFS, deduplication, compression.
  - **FSx for OpenZFS**: OpenZFS-based, compatible with ZFS.
- **Instance Store**:
  - Temporary storage physically attached to host machine
  - Extremely fast (NVMe/SSD)
  - Data lost on instance stop/terminate
  - Use for: Buffer, cache, scratch data, temporary processing
  - Never use for persistent data
- **Decision matrix**:
  - Single EC2 persistent block storage → EBS
  - Shared file system across EC2 instances → EFS
  - High-performance computing → FSx for Lustre
  - Windows file sharing → FSx for Windows
  - Temporary, ultra-fast storage → Instance Store

---

### 2.4. Security & Identity

**Câu 27: Giải thích IAM Policies - JSON policy structure, actions, resources, conditions, và evaluation logic.**

**Trả lời:**
- **JSON Policy Structure**:
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "UniqueIdentifier",
        "Effect": "Allow|Deny",
        "Principal": { ... },
        "Action": "s3:GetObject|s3:PutObject",
        "Resource": "arn:aws:s3:::my-bucket/*",
        "Condition": {
          "StringEquals": {
            "aws:SourceVpc": "vpc-123456"
          }
        }
      }
    ]
  }
  ```
- **Key Elements**:
  - **Effect**: Allow or Deny (DENY always overrides ALLOW)
  - **Action**: API operations (can use wildcards like `s3:*`)
  - **Resource**: ARN (Amazon Resource Name) of the resource
  - **Principal**: Who the policy applies to (user, role, service, account)
  - **Condition**: Optional constraints (time, IP, MFA, etc.)
- **IAM Policy Types**:
  - **Identity-based**: Attached to users/roles/groups (who can do what)
  - **Resource-based**: Attached to resources (who can access this resource) - e.g., S3 bucket policy, IAM role trust policy
  - **Scoping policies**: Limit permissions to specific conditions
- **Evaluation Logic**:
  1. Explicit Deny → DENY (always wins)
  2. Explicit Allow → ALLOW
  3. Implicit Deny (no matching allow) → DENY
  - All applicable policies evaluated together (identity + resource + org + SCP)
- **Best practices**:
  - Principle of least privilege (grant minimum necessary permissions)
  - Use conditions to restrict by IP, VPC, time, MFA
  - Use roles instead of sharing credentials
  - Use policy variables (`${aws:username}`) for dynamic policies
  - Regularly audit with IAM Access Analyzer

**Câu 28: Giải thích Cross-Account Access - Role-Based Access vs Resource-Based Policies.**

**Trả lời:**
- **Cross-Account Access**: Allow users/roles from one AWS account to access resources in another AWS account.
- **Method 1: IAM Role (Recommended)**:
  - Account B creates IAM Role with trust policy allowing Account A to assume it
  - Trust Policy example:
    ```json
    {
      "Principal": {"AWS": "arn:aws:iam::111111111111:root"},
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
    ```
  - User in Account A calls `sts:AssumeRole` → gets temporary credentials
  - Role permissions define what user can do in Account B
  - **Advantages**: Centralized permission management, supports MFA, supports external ID for third parties
- **Method 2: Resource-Based Policy**:
  - Resource (e.g., S3 bucket) in Account B has policy allowing Account A
  - Example: S3 bucket policy with `Principal: {"AWS": "arn:aws:iam::111111111111:role/role-name"}`
  - Directly attached to the resource
  - **Advantages**: Simpler for specific resources (S3, SQS, Lambda)
  - **Limitations**: Not all services support resource-based policies
- **SCP (Service Control Policies)**:
  - Organization-level policies that set permission boundaries for all accounts in OU
  - Apply at master/root account level
  - Can only deny (no allow in SCP)
  - Use for: Guardrails (e.g., "no EC2 instances in us-east-1", "MFA required")
- **Use case**:
  - Centralized logging account → Other accounts write logs to central account via role assumption
  - Multi-account strategy (production/staging/development) → Central security account

---

### 2.5. Database Services

**Câu 29: Giải thích sự khác nhau giữa Amazon RDS, Aurora, DynamoDB, và ElastiCache. Kiến trúc và trade-off.**

**Trả lời:**
- **Amazon RDS (Relational Database Service)**:
  - Managed relational database (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server)
  - Single-AZ or Multi-AZ (synchronous standby in different AZ)
  - Read replicas for read scaling (asynchronous replication)
  - Automated backups, snapshots, patching
  - Max storage: 64TB (depending on engine)
  - **Limitation**: Single writer, vertical scaling limit, manual sharding needed
- **Amazon Aurora**:
  - AWS-native relational database (Aurora MySQL, Aurora PostgreSQL)
  - **Storage architecture**: Data replicated across 3 AZs automatically (6 copies)
  - Storage auto-scales from 10GB to 128TB
  - **Performance**: Up to 5x faster than MySQL, 3x faster than PostgreSQL
  - **Serverless Aurora**: Auto-scales compute capacity (Aurora Serverless v2)
  - **Global Database**: Low-latency reads across regions, RPO < 1 second
  - **Multi-master**: Multiple read-write instances (Aurora PostgreSQL only)
- **Amazon DynamoDB**:
  - Fully managed NoSQL key-value/document database
  - Single-digit millisecond latency at any scale
  - Provisioned or on-demand capacity mode
  - Global Tables for multi-region active-active
  - DAX (DynamoDB Accelerator) for read caching
  - Streams for change data capture
  - **Limitations**: No joins, no complex queries, item size limit (400KB)
- **Amazon ElastiCache**:
  - Managed in-memory cache (Redis or Memcached)
  - Redis: Advanced data structures, persistence, replication, cluster mode
  - Memcached: Simple key-value, multi-threaded, simpler
  - Use for: Session management, leaderboards, caching database queries
  - **Not a database replacement**: Cache layer for performance
- **Architecture Decision**:
  - Complex queries, ACID transactions → RDS or Aurora
  - Massive scale, simple access patterns, high throughput → DynamoDB
  - Caching layer → ElastiCache (Redis for complex data, Memcached for simple)
  - Hybrid: DynamoDB as primary, ElastiCache as cache layer

**Câu 30: Giải thích về database migration strategies trên AWS (AWS DMS, Schema Conversion Tool).**

**Trả lời:**
- **AWS Database Migration Service (DMS)**:
  - Migrates databases to AWS with minimal downtime
  - **Migration types**:
    - **Full load**: One-time migration of all data
    - **CDC (Change Data Capture)**: Continuous replication of changes
    - **Full load + CDC**: Initial migration + ongoing sync
  - **Source/Target support**: On-premises or cloud, RDS, Aurora, DynamoDB, Redshift, S3
  - **Transformation**: Data type mapping, column mapping, table mapping
  - **Monitoring**: CloudWatch metrics, migration task events
- **AWS Schema Conversion Tool (SCT)**:
  - Converts database schema from one engine to another (e.g., Oracle → Aurora PostgreSQL)
  - Converts SQL code (stored procedures, functions, triggers, views)
  - Provides assessment report showing convertibility percentage
  - Manual conversion needed for unsupported features
- **Migration Strategies (The 7 R's from AWS)**:
  - **Rehost (Lift and Shift)**: Move as-is to EC2
  - **Replatform**: Move to managed service with minimal changes (RDS)
  - **Refactor**: Re-architect for cloud-native (DynamoDB, Aurora)
  - **Replatform**: Move to managed service with minimal changes (RDS)
  - **Repurchase**: Switch to different product (e.g., on-prem SQL → SaaS)
  - **Retain**: Keep on-premises
  - **Retire**: Decommission
- **Best practices**:
  - Use SCT assessment first to understand migration complexity
  - Start with full load, then enable CDC for zero-downtime migration
  - Test data integrity after migration (row counts, checksums)
  - Set up monitoring and rollback plan
  - Consider data residency and compliance requirements

---

### 2.6. Monitoring & Observability

**Câu 31: Giải thích CloudWatch, X-Ray, và Trusted Advisor. Tích hợp và best practices cho observability.**

**Trả lời:**
- **Amazon CloudWatch**:
  - **Metrics**: Time-series data (CPU, memory, request count, custom metrics)
  - **Alarms**: Trigger actions when metric exceeds threshold (SNS notification, Auto Scaling, EC2 actions)
  - **Logs**: Centralized log management, log insights for querying
  - **Dashboards**: Visualize metrics across services
  - **Custom Metrics**: Publish application-specific metrics via PutMetricData
  - **Events**: Rule-based response to state changes (e.g., EC2 state change → Lambda)
- **AWS X-Ray**:
  - **Distributed tracing**: Track requests across microservices
  - **Service map**: Visualize architecture and dependencies
  - **Annotations/Metadata**: Add custom data to traces
  - **Insights**: Automatically detect anomalies (latency, errors)
  - **Sampling**: Configurable sampling rules to control cost
  - **Integration**: Native support for Lambda, EC2, ECS, EKS, API Gateway, Spring Boot, etc.
  - **Trace ID**: Propagated through all service calls in a request chain
- **AWS Trusted Advisor**:
  - **5 categories**: Cost optimization, Performance, Security, Fault tolerance, Service limits
  - Provides recommendations with actionable steps
  - Full access requires Business/Enterprise Support plan
  - Checks: Underutilized EC2 instances, open security groups, MFA on root account, S3 public access, etc.
- **Observability Best Practices**:
  - **Three Pillars**: Metrics (what), Logs (why), Traces (how)
  - Use centralized logging with CloudWatch Log Groups and Metric Filters
  - Define SLOs/SLIs and create alarms on them
  - Use X-Ray for distributed system debugging
  - Create structured log format (JSON) for easy querying
  - Use CloudWatch Synthetics for uptime monitoring
  - Implement anomaly detection (CloudWatch Anomaly Detection)

---

### 2.7. Serverless & Event-Driven Architecture

**Câu 32: Giải thích kiến trúc serverless với Lambda + API Gateway + DynamoDB + SQS, và các pattern.**

**Trả lời:**
- **Basic Serverless Architecture**:
  ```
  Client → API Gateway → Lambda → DynamoDB
  Client → API Gateway → Lambda → SQS → Lambda → Database
  ```
- **Component Details**:
  - **API Gateway**: REST or HTTP API, request validation, throttling, CORS, custom domain
  - **Lambda**: Business logic, auto-scaling, pay-per-execution
  - **DynamoDB**: Low-latency data store, single-digit ms
  - **SQS**: Decouple components, buffer traffic, async processing
- **Common Patterns**:
  - **Lambda Microservices**: Each Lambda handles one business capability, communicates via API Gateway
  - **Fan-out**: S3 upload event → Lambda → SQS → multiple consumers
  - **Saga Pattern**: Distributed transaction across multiple Lambda functions using Step Functions or SQS
  - **CQRS (Command Query Responsibility Segregation)**: Write model (Lambda → DynamoDB) separated from read model (DynamoDB Streams → Elasticsearch)
  - **Event Sourcing**: All changes stored as events (DynamoDB Streams → Lambda → event store)
- **Cold Start Optimization**:
  - Provisioned Concurrency: Keep functions warm
  - Use lightweight runtimes (Node.js, Python vs Java, .NET)
  - Minimize package size (tree shaking, layers)
  - Keep initialization code minimal
  - Use Lambda SnapStart (Java)
- **Best practices**:
  - Idempotent Lambda functions (handle duplicate events)
  - Set appropriate memory/timeout
  - Use environment variables (encrypted with KMS)
  - Implement retry logic with exponential backoff
  - Use VPC only when necessary (adds cold start time)

---

### 2.8. CI/CD & DevOps

**Câu 33: Giải thích pipeline CI/CD trên AWS sử dụng CodePipeline, CodeBuild, CodeDeploy.**

**Trả lời:**
- **AWS CodePipeline**:
  - Orchestrates CI/CD workflow as a pipeline (visual pipeline editor)
  - Stages: Source → Build → Test → Deploy → Approval → Deploy
  - Source providers: CodeCommit, GitHub, S3, ECR
  - Integrates with all AWS deployment services
  - Manual approval gates between stages
- **AWS CodeBuild**:
  - Managed build service, runs in Docker containers
  - `buildspec.yml` defines build commands
  - Caches dependencies for faster builds
  - Can pull from ECR (build container images)
  - Concurrency limits, batch builds
  - Artifacts output to S3, ECR, or CodePipeline
- **AWS CodeDeploy**:
  - Automates code deployment to EC2, Lambda, ECS, on-premises
  - **Deployment types**:
    - **In-place**: Updates existing instances
    - **Blue/Green**: Deploys new version to new instances, swaps traffic
    - **Canary**: Routes small percentage of traffic to new version, gradually shifts
    - **Linear**: Gradually shifts traffic in equal increments
  - **AppSpec file**: Defines deployment steps, hooks (BeforeInstall, AfterInstall, ApplicationStart)
  - **Rollback**: Automatic rollback on deployment failure
- **Complete Pipeline Example**:
  1. **Source**: CodeCommit/GitHub webhook triggers pipeline
  2. **Build**: CodeBuild builds application, runs unit tests, creates Docker image, pushes to ECR
  3. **Test**: CodeBuild runs integration tests
  4. **Deploy**: CodeDeploy deploys to ECS (blue/green) or EC2
  5. **Verify**: CloudWatch alarms monitor health, automatic rollback if degraded

---

### 2.9. High Availability & Disaster Recovery

**Câu 34: Giải thích các mô hình DR (Disaster Recovery) trên AWS: Backup & Restore, Pilot Light, Warm Standby, Multi-Site.**

**Trả lời:**
- **Backup & Restore (RTO: hours-days, RPO: hours)**:
  - Regular backups to S3 (cross-region replication)
  - Restore from backups when disaster occurs
  - Lowest cost, highest RTO
  - Use for: Non-critical workloads, development environments
  - Tools: AWS Backup, AMIs, RDS snapshots, S3 cross-region replication
- **Pilot Light (RTO: minutes-hours, RPO: minutes)**:
  - Core services always running in secondary region (small instance, database in standby)
  - Scale up when disaster occurs
  - Reduced costs vs full standby
  - Use for: Business-critical applications with moderate RTO requirements
  - Example: EC2 minimum instance running, RDS standby in secondary region
- **Warm Standby (RTO: minutes, RPO: seconds)**:
  - Scaled-down version of full production in secondary region
  - Can handle reduced traffic until full scaling
  - Almost fully operational, just waiting to scale up
  - Use for: Applications requiring fast recovery with some traffic reduction acceptable
  - Example: Auto Scaling min = 1, RDS read replica promoted to writer
- **Multi-Site / Active-Active (RTO: seconds-minutes, RPO: seconds)**:
  - Fully operational in multiple regions simultaneously
  - Traffic routed to healthy region (Route 53 failover, latency-based routing)
  - Highest cost, lowest RTO/RPO
  - Use for: Mission-critical applications, zero downtime requirements
  - Example: Global DynamoDB tables, multi-region Aurora, Route 53 health checks + failover
- **Decision factors**:
  - Budget, RTO (Recovery Time Objective), RPO (Recovery Point Objective), business criticality

---

### 2.10. Cost Optimization

**Câu 35: Giải thích chi tiết các strategies tối ưu chi phí trên AWS với Reserved Instances, Savings Plans, Spot Instances.**

**Trả lời:**
- **On-Demand Instances**:
  - Pay by the second/hour, no commitment
  - Highest cost, maximum flexibility
  - Use for: Short-term, unpredictable workloads, testing
- **Reserved Instances (RI)**:
  - 1-year or 3-year commitment for specific instance type in specific AZ
  - Up to 72% discount vs On-Demand
  - **Types**:
    - **Standard RI**: Max discount, limited flexibility (instance type, AZ fixed)
    - **Convertible RI**: Can exchange for different instance type, family, OS (up to 54% discount)
    - **Scheduled RI**: Reserve for specific time windows (daily/weekly/monthly)
  - **Payment options**: All upfront (max discount), Partial upfront, No upfront
  - Apply to EC2, RDS, Redshift, DynamoDB (reserved capacity)
- **Savings Plans**:
  - Flexible alternative to RIs, commit to specific $/hour usage
  - **Compute Savings Plans**: Up to 66% discount, apply across instance families, regions, OS, EC2/Lambda/Fargate
  - **EC2 Instance Savings Plans**: Up to 72% discount, apply to specific instance family in region
  - **SageMaker Savings Plans**: For SageMaker instances
  - More flexible than RIs, easier to manage
- **Spot Instances**:
  - Use unused EC2 capacity, up to 90% discount
  - Can be interrupted with 2-minute warning when AWS needs capacity back
  - Best for: Fault-tolerant, flexible, stateless workloads
  - Use cases: Batch processing, CI/CD, data analysis, rendering, web scraping
  - **Spot Fleet**: Mix of Spot and On-Demand instances
  - **Spot Instance Pools**: Multiple instance types in same region for availability
- **Other optimization strategies**:
  - Right-sizing instances (use Trusted Advisor, Compute Optimizer)
  - Use Graviton (ARM-based) instances for 20% better price-performance
  - S3 lifecycle policies to move old data to cheaper tiers
  - Delete unattached EBS volumes, unused Elastic IPs
  - Use Auto Scaling to match capacity to demand
  - Consolidated billing with AWS Organizations for volume discounts

---

### 2.11. Advanced Topics

**Câu 36: Giải thích about Infrastructure as Code (IaC) với CloudFormation, CDK, Terraform - comparison.**

**Trả lời:**
- **AWS CloudFormation**:
  - Native AWS IaC service
  - JSON/YAML templates defining AWS resources
  - Stack-based (all resources as one unit, create/update/delete together)
  - Change sets for preview before execution
  - Drift detection (compare deployed vs template)
  - **Limitations**: Verbose JSON, slow iteration, limited cross-stack references, AWS-only
- **AWS CDK (Cloud Development Kit)**:
  - Define infrastructure using familiar programming languages (TypeScript, Python, Java, C#, Go)
  - Synthesizes to CloudFormation templates
  - **Constructs**: Reusable, composable cloud components
  - **Benefits**: Code reuse, logic in infrastructure, faster development, less boilerplate
  - Example: `new ec2.Vpc(this, 'MyVpc', { cidr: '10.0.0.0/16' })`
- **Terraform**:
  - Multi-cloud IaC tool (HashiCorp)
  - HCL (HashiCorp Configuration Language) declarative
  - State file (.tfstate) tracks real-world resources
  - `terraform plan` shows execution plan
  - **Providers**: AWS, Azure, GCP, and hundreds of services
  - **Modules**: Reusable, shareable configurations
  - Works across cloud providers, great for multi-cloud
  - **Limitations**: CloudFormation drift detection not equivalent, state management complexity
- **Comparison**:
  - **Native AWS support**: CloudFormation/CDK > Terraform
  - **Multi-cloud**: Terraform > CloudFormation > CDK
  - **Developer experience**: CDK > Terraform > CloudFormation (JSON)
  - **Community/modules**: Terraform > CDK > CloudFormation
  - **Maturity**: CloudFormation (oldest), Terraform (mature), CDK (growing)
- **Best practice**: Use CDK for AWS-only environments, Terraform for multi-cloud

---

### 2.12. AWS Well-Architected Framework

**Câu 37: Giải thích 6 trụ cột của AWS Well-Architected Framework với ví dụ thực tế.**

**Trả lời:**
- **1. Operational Excellence**:
  - Run and monitor systems to deliver business value
  - Code infrastructure as code, automate processes
  - Make frequent, small, reversible changes
  - **Practices**: CI/CD pipelines, infrastructure automation, runbooks, event-driven operations
  - Example: Use CloudFormation/CDK to deploy, automated rollback on failures

- **2. Security**:
  - Protect data, systems, and assets
  - Shared Responsibility Model (AWS secures infra, you secure data/apps)
  - Apply least privilege, encrypt data, detect threats
  - **Practices**: IAM with MFA, security groups as firewalls, KMS encryption, GuardDuty for threat detection, CloudTrail for auditing
  - Example: Use IAM roles for Lambda (not hardcoded credentials), enable S3 bucket encryption, use WAF for web app protection

- **3. Reliability**:
  - Recover from infrastructure failures
  - Design for failure, test recovery procedures
  - **Practices**: Multi-AZ deployments, Auto Scaling, backup and restore, chaos engineering (AWS Fault Injection Simulator), circuit breakers
  - Example: RDS Multi-AZ for database HA, multiple EC2 instances across AZs behind ALB

- **4. Performance Efficiency**:
  - Use computing resources efficiently to meet requirements
  - **Practices**: Serverless (Lambda), auto-scaling, right-sizing, Graviton instances, caching (ElastiCache/CDN)
  - Example: Use Lambda instead of always-on EC2 for sporadic workloads, use CloudFront for content caching

- **5. Cost Optimization**:
  - No money wasted
  - **Practices**: Right-sizing, Savings Plans, Spot Instances, serverless, delete unused resources, consolidated billing
  - Example: Use Compute Optimizer to identify underutilized EC2, implement S3 lifecycle policies

- **6. Sustainability**:
  - Reduce environmental impact
  - **Practices**: Serverless reduces idle resources, choose regions with lower carbon footprint, use Graviton (more energy efficient), optimize data transfer
  - Example: Use serverless architectures to eliminate idle compute, use AWS regions powered by renewable energy

---

## PHẦN 3: CÂU HỎI PHỎNG VẤN COMBINED

**Câu 38: Thiết kế hệ thống notification service sử dụng AWS serverless với yêu cầu: 10 million events/day, delivery guarantee, retry logic, and monitoring.**

**Giải pháp:**
```
Components:
1. API Gateway (HTTP API) → Receives notification requests
2. Lambda (validation) → Validates input, enqueues to SQS
3. SQS Standard Queue → Buffers messages, decouples producer/consumer
4. Lambda (consumer) → Processes messages, decides channel
5. SNS → Fan-out to multiple delivery services
6. DynamoDB → Store message state, delivery status, templates
7. CloudWatch → Monitoring, alarms, dashboards
8. S3 → Store failed messages for DLQ analysis

Flow:
User Request → API Gateway → Lambda (validate & enrich) → SQS → Lambda (process) → SNS → [SES/FCM/Twilio] → DynamoDB (update status)

Retry: Lambda retries on failure (2 retries), then SQS DLQ after max receive count
Monitoring: CloudWatch metrics for queue depth, error rate, delivery latency
DLQ: Failed messages after max retries → SQS DLQ → Analysis Lambda → S3
```

- **Scalability**: SQS scales automatically, Lambda scales with SQS events
- **Delivery guarantee**: SQS at-least-once delivery, idempotent Lambda processing
- **Cost**: Pay-per-use, no servers to manage
- **Monitoring**: X-Ray for tracing, CloudWatch for metrics, SNS alerts

---

**Câu 39: Migration strategy cho hệ thống monolith đang chạy trên EC2 sang microservices trên EKS với DynamoDB.**

**Giải pháp:**
1. **Phase 1 - Analysis**:
   - Analyze monolith: Identify bounded contexts (Domain-Driven Design)
   - Database dependency mapping: Shared tables → Service boundaries
   - Traffic analysis: Which modules have highest traffic/isolation potential
   - Use AWS Application Discovery Service

2. **Phase 2 - Strangler Fig Pattern**:
   - Deploy API Gateway as facade in front of monolith
   - Route new features to microservices, old features to monolith
   - Gradually extract services one by one
   - Use DynamoDB Streams for event-driven data sync during transition

3. **Phase 3 - Data Migration**:
   - Use AWS DMS (Database Migration Service) for CDC
   - Extract service-specific data to DynamoDB
   - Keep shared data in RDS, or create read replicas
   - Implement saga pattern for distributed transactions

4. **Phase 4 - Full Migration**:
   - Decommission monolith
   - Full EKS deployment with HPA (Horizontal Pod Autoscaler)
   - Multi-AZ EKS cluster with managed node groups
   - Blue/green deployments with ArgoCD or CodeDeploy for EKS

5. **Monitoring throughout**:
   - X-Ray tracing across services
   - CloudWatch container insights for EKS
   - RUM (Real User Monitoring) for end-to-end experience

---

**Câu 40: Security incident response trên AWS - từ detection đến remediation.**

**Giải pháp:**
1. **Detection**:
   - Amazon GuardDuty: Threat detection (unauthorized API calls, unusual IP, crypto-mining)
   - AWS CloudTrail: Audit trail of all API calls
   - Amazon Macie: Sensitive data discovery (PII in S3)
   - Amazon Security Hub: Centralized security findings, compliance checks
   - VPC Flow Logs: Network traffic analysis

2. **Analysis**:
   - Security Hub aggregates findings from GuardDuty, Inspector, Macie
   - Severity scoring and prioritization
   - Correlate findings across services
   - CloudWatch Events trigger automated response

3. **Containment**:
   - Isolate compromised EC2 instance (change security group to deny all)
   - Revoke compromised IAM credentials (disable user, rotate keys)
   - Block malicious IPs with Network ACL or WAF
   - Snapshot EBS volumes for forensics

4. **Remediation**:
   - AWS Systems Manager Automation for predefined remediation playbooks
   - Lambda functions for automated response (e.g., isolate instance, block IP)
   - Patch vulnerable systems via SSM Patch Manager
   - Restore from backups if data compromised

5. **Recovery**:
   - Restore services from clean backups
   - Verify system integrity
   - Update security policies
   - Document incident, post-mortem analysis

6. **Prevention**:
   - Implement AWS Config rules for compliance
   - Enable MFA on all accounts, especially root
   - Implement least privilege with IAM
   - Regular security assessments with Inspector

---

## GHI CHÚ CUỐI

### Các câu hỏi thường gặp phỏng vấn middleware/senior level:

1. **Rate Limiting**: Token bucket vs Leaky bucket vs Fixed window vs Sliding window
2. **Load Balancing**: Round Robin vs Least Connections vs Consistent Hashing vs IP Hash
3. **Service Discovery**: DNS-based (Route 53), Client-side (Eureka), Server-side (ALB/NLB), Service Mesh (Istio, Linkerd)
4. **Circuit Breaker**: Implementation patterns, half-open state management
5. **Database Connection Pooling**: HikariCP, PgBouncer, proxy-level pooling
6. **GraphQL vs REST**: When to choose, performance trade-offs, N+1 problem solutions
7. **Event Sourcing vs CQRS**: Patterns, when to use, how they complement each other
8. **Kubernetes Networking**: CNI plugins (Calico, Cilium), Service mesh, Ingress controllers (Nginx, Traefik)
9. **Observability Stack**: OpenTelemetry, Prometheus + Grafana, ELK/EFK stack
10. **GitOps**: ArgoCD, Flux, declarative deployments, Git as source of truth

### Resources tham khảo:
- AWS Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html
- AWS Certified Solutions Architect Pro Study Guide
- Designing Data-Intensive Applications (Martin Kleppmann)
- System Design Interview (Alex Xu)
- AWS Official Documentation
