Được. Với mục tiêu của bạn, mình **không khuyên học AWS theo kiểu học từng service rồi làm đề SAP**. Trong 2 tháng, nên làm theo mô hình:

> **Học kiến thức → Hands-on → tự thiết kế hệ thống → xử lý incident → trả lời interview → làm câu hỏi SAP**

Như vậy 1 lần học phục vụ **3 mục tiêu cùng lúc**: đi phỏng vấn, đi làm AWS thực tế và thi SAP-C02.

AWS hiện xác định SAP-C02 theo 4 domain: **Organizational Complexity 26%, New Solutions 29%, Continuous Improvement 25%, Migration & Modernization 20%**. Nội dung xoay quanh compute, database, DR, HA, networking, security, migration, serverless, storage, cost... ([AWS ドキュメント][1])

## 🎯 Mục tiêu sau 2 tháng

Mình đề xuất mục tiêu thực tế:

**Sau 8 tuần bạn phải có khả năng:**

1. Tự dựng một hệ thống AWS hoàn chỉnh.
2. Hiểu sâu:

   * VPC
   * EC2
   * ALB
   * Auto Scaling
   * IAM
   * S3
   * RDS
   * DynamoDB
   * Lambda
   * API Gateway
   * CloudFront
   * Route 53
   * CloudWatch
   * SQS/SNS
   * ECS/Fargate
   * ECR
   * WAF
   * KMS
3. Hiểu HA / scalability / security / cost / DR.
4. Biết **tại sao chọn service A thay vì B**.
5. Có khả năng debug một hệ thống AWS đang lỗi.
6. Có 1–2 project để nói trong interview.
7. Có nền tảng để bắt đầu luyện đề SAP-C02 nghiêm túc.

Đặc biệt, SAP không đơn thuần hỏi "service này dùng làm gì". AWS mô tả SAP-C02 là chứng chỉ xác nhận kỹ năng thiết kế các giải pháp AWS tối ưu dựa trên **Well-Architected Framework**. ([AWS ドキュメント][2])

---

# 🗓️ Lộ trình 9 tuần (8 tuần học + 1 tuần buffer/exam)

> ✅ **Với commit 3-4h/ngày, lộ trình này 100% khả thi**

---

## Tuần 1 — AWS Foundation + IAM + VPC

### Học

**AWS fundamentals**

* Region / AZ
* Account
* IAM
* IAM User / Role
* Policy
* STS (Security Token Service)
* Organizations
* CloudTrail
* CloudWatch

**Networking**

* VPC
* Subnet
* Route table
  * Có định nghĩa Destination và Target. Khi gửi 1 request tới 1 IP, check Destination → thoả mãn thì forward tới Target
* Internet Gateway
* NAT Gateway
* Security Group
* NACL
* Public / Private subnet
* VPC Endpoint
* VPC Peering

### Bức tranh tổng thể
```
AWS
│
├── Region
│   ├── AZ
│   │   ├── VPC
│   │   │   ├── Public Subnet
│   │   │   └── Private Subnet
│   │   │
│   │   └── ...
│   └── AZ
│
├── Account
│   ├── IAM
│   ├── EC2
│   ├── S3
│   └── ...
│
└── Organizations
    ├── Management Account
    ├── Production Account
    ├── Development Account
    └── ...
```

### 4 câu hỏi lớn 

1. AWS đặt server ở đâu? → Region / AZ
2. Tài nguyên thuộc về ai? → Account / Organizations
3. Ai được phép làm gì? → IAM / Policy / STS
4. Server giao tiếp với Internet và nhau thế nào? → VPC / Subnet / Route / Gateway

### Hands-on

Tự dựng:

```text
Internet
   ↓
Internet Gateway
   ↓
Public Subnet
   ↓
EC2
```

Sau đó:

```text
Internet
   ↓
ALB
   ↓
Private Subnet
   ↓
EC2
```

Rồi thử:

* EC2 private không có public IP
* NAT Gateway
* Bastion / SSM
* Security Group
* VPC Endpoint

### Interview

Bạn phải trả lời được:

> Security Group vs NACL?

> Public subnet là gì?

> Private subnet làm sao ra Internet?

> NAT Gateway hoạt động thế nào?

> Internet Gateway khác NAT Gateway thế nào?

> IAM Role khác IAM User?

### 📊 Project Deliverable - Tuần 1

**Mục tiêu**: AWS Foundation setup + VPC architecture + IAM + Security groups

✅ Commit items:
- [x] AWS Account setup (create account, enable billing alerts)
- [x] IAM users + roles (create 2-3 IAM users, 1 admin role, 1 developer role)
- [x] CloudTrail enabled (log all API calls to S3)
- [ ] VPC creation:
  - [ ] 1 VPC (10.0.0.0/16)
  - [ ] 2 Public Subnets (10.0.1.0/24, 10.0.2.0/24 in different AZs)
  - [ ] 2 Private Subnets (10.0.11.0/24, 10.0.12.0/24 in different AZs)
- [ ] Internet Gateway (attach to VPC)
- [ ] NAT Gateway (in public subnet, test from private EC2)
- [ ] Route tables:
  - [ ] Public route table (default route → IGW)
  - [ ] Private route table (default route → NAT Gateway)
- [ ] Security Groups:
  - [ ] Public SG (SSH from your IP, HTTP/HTTPS from anywhere)
  - [ ] Private SG (all traffic from public SG)
- [ ] EC2 instances:
  - [ ] 1 instance in public subnet (with public IP)
  - [ ] 1 instance in private subnet (no public IP)
- [ ] Test connectivity:
  - [ ] SSH into public EC2
  - [ ] From public EC2, SSH into private EC2
  - [ ] From private EC2, curl to public internet (via NAT)
- [ ] CloudWatch basic monitoring:
  - [ ] CPU/Network metrics enabled
  - [ ] Create 1 simple alarm (CPU > 70%)
- [ ] Documentation:
  - [ ] vpc-architecture.md (diagram + explanation of all components)
  - [ ] iam-setup.md (users, roles, policies)
  - [ ] security-groups.md (rules documentation)
- [ ] GitHub repo initialized with infrastructure diagrams
- [ ] Cost estimation: ~$20-30/month (mostly NAT Gateway + small EC2s)

---

# Tuần 2 — Compute + Storage + Database

### Compute

* EC2
* AMI
* EBS
* Instance type
* Auto Scaling
* Launch Template
* ALB
* Target Group

### Storage

* S3
* Storage Class
* Lifecycle
* Versioning
* Encryption
* Presigned URL
* S3 replication

### Database

* RDS
* Multi-AZ
* Read Replica
* Aurora
* DynamoDB
* ElastiCache

### Hands-on

Dựng:

```text
                 ┌── EC2
Internet → ALB ──┤
                 └── EC2
                     ↓
                    RDS
```

Sau đó:

```text
EC2 × N
   ↓
 RDS
   ↓
Read Replica
```

Thử:

* EC2 chết
* RDS failover
* Auto Scaling
* ALB health check
* S3 lifecycle

### 📊 Project Deliverable - Tuần 2

**Mục tiêu**: Hoàn thành ALB + Auto Scaling + RDS

✅ Commit items:
- [ ] ALB + 2 EC2 instances (different AZs) - health check working
- [ ] Auto Scaling Group (min 1, max 3, scale at 70% CPU)
- [ ] RDS Multi-AZ (PostgreSQL, automated backups)
- [ ] Basic S3 bucket (for logs)
- [ ] Documentation: architecture.md (with diagrams)
- [ ] Cost estimation: $250-350/month
- [ ] Terraform/CloudFormation IaC (nên bắt đầu từ tuần này)

---

# Tuần 3 — Serverless + Event Driven

Học:

* Lambda
* API Gateway
* SQS
* SNS
* EventBridge
* Step Functions
* DynamoDB
* Kinesis cơ bản

Dựng:

```text
Client
  ↓
API Gateway
  ↓
Lambda
  ↓
DynamoDB
```

Sau đó:

```text
API
 ↓
SQS
 ↓
Lambda
 ↓
DynamoDB
```

Mục tiêu là hiểu **decoupling**.

Ví dụ interview:

> Một hệ thống có 10.000 request/giây, backend không xử lý kịp. Bạn làm gì?

Không được trả lời ngay:

> "Tăng EC2."

Phải suy nghĩ:

```text
ALB
 ↓
Auto Scaling
 ↓
SQS
 ↓
Worker
```

và phân tích trade-off.

### 📊 Project Deliverable - Tuần 3

**Mục tiêu**: Lambda + API Gateway + SQS + DynamoDB integration

✅ Commit items:
- [ ] API Gateway + Lambda (REST API)
- [ ] DynamoDB table (with TTL)
- [ ] SQS queue + Lambda consumer
- [ ] SNS topic (async notifications)
- [ ] X-Ray tracing (for debugging)
- [ ] Unit test 1-2 Lambda functions
- [ ] Cost estimation: +$100-150/month (serverless pattern)
- [ ] Document: decoupling.md (SQS vs SNS use cases)

---

# Tuần 4 — Security + SAP Prep Start ⭐

Đây là phần **cực kỳ quan trọng cho SAP và interview**.

### ⭐ SAP Prep Start!

**Từ tuần này, bắt đầu làm 5-10 SAP questions/tuần**

- Ít nhất 1-2 câu/ngày
- Không chỉ xem đáp án, phải hiểu **tại sao**
- Ghi chú các câu sai vào \"SAP_mistakes.md\"
- Focus: Security, IAM, Permission, Data Protection

Học:

* IAM
* IAM Policy
* Role
* STS
* KMS
* Secrets Manager
* Parameter Store
* Cognito
* WAF
* Shield
* Security Hub
* GuardDuty
* CloudTrail
* AWS Config

Hands-on:

### Hands-on: Project security

```text
CloudFront
    ↓
   WAF
    ↓
  ALB
    ↓
Private EC2
    ↓
   RDS
```

Không public RDS.

Không cho EC2 SSH trực tiếp từ Internet.

Secret không hard-code (dùng Secrets Manager).

Database encryption (KMS).

S3 bucket private + versioning + MFA delete.

Đây là lúc bắt đầu tư duy:

> **"Nếu hệ thống này bị attack thì sao?"**

### 📊 Project Deliverable - Tuần 4

**Mục tiêu**: Security hardening + IAM + KMS

✅ Commit items:
- [ ] IAM roles & policies (least privilege)
- [ ] CloudFront + WAF (block specific countries/IPs)
- [ ] KMS key (for RDS encryption)
- [ ] Secrets Manager (store DB password, API keys)
- [ ] VPC Security Groups (ingress/egress rules documented)
- [ ] S3 bucket policies (private, versioning, MFA delete)
- [ ] CloudTrail logging (all API calls)
- [ ] Cost estimation: +$50-80/month
- [ ] Document: security.md (threat model + mitigations)
- [ ] SAP Notes: Ghi chú 5-10 SAP questions về Security/IAM

---

# Tuần 5 — High Availability + Disaster Recovery

Đây là tuần rất quan trọng cho SAP.

Học:

### HA

```text
AZ-a                 AZ-b

EC2                   EC2
 ↓                     ↓
 └─────── ALB ─────────┘
            ↓
           RDS
```

### DR

Phải phân biệt:

* Backup & Restore
* Pilot Light
* Warm Standby
* Multi-site Active/Active

Hiểu:

* RTO
* RPO

Ví dụ:

> Business yêu cầu RPO 5 phút, RTO 15 phút.

Bạn phải biết lựa chọn architecture nào.

AWS Well-Architected hiện có **6 pillars**:

* Operational Excellence
* Security
* Reliability
* Performance Efficiency
* Cost Optimization
* Sustainability. ([AWS ドキュメント][3])

Từ tuần 5 trở đi, **mọi architecture bạn thiết kế đều phải tự hỏi 6 pillar này**.

### 📊 Project Deliverable - Tuần 5

**Mục tiêu**: Multi-AZ HA + RDS Failover + Backup Strategy

✅ Commit items:
- [ ] RDS Multi-AZ + failover testing (manual trigger failover)
- [ ] Read Replicas (in different AZ)
- [ ] Automated backup (daily snapshots)
- [ ] Test RDS restore from snapshot
- [ ] ElastiCache (Redis) in Multi-AZ
- [ ] Test cache failover
- [ ] Document: ha-dr.md (RTO/RPO analysis)
- [ ] Create mock incident: "RDS primary down" → test failover time
- [ ] SAP Questions: 8-10 questions on HA/DR
- [ ] Cost estimation: +$200-250/month (Multi-AZ overhead)
- [ ] Well-Architected Review: self-assess 6 pillars

---

# Tuần 6 — Migration + Hybrid + Large Scale

SAP-C02 dành **20%** cho Migration & Modernization. ([AWS ドキュメント][1])

Học:

* AWS Migration Hub
* Application Migration Service
* Database Migration Service (DMS)
* Storage Gateway
* DataSync
* Direct Connect
* Site-to-Site VPN
* Transit Gateway
* VPC Peering
* AWS Organizations
* Control Tower
* Landing Zone

### Hands-on Scenario

Công ty có:

```text
On-premise (simulated via EC2 in different VPC)
   ↓
10 GB database (RDS)
   ↓
AWS Production VPC
```

Bạn phải thiết kế:

* DMS vs DataSync?
* Direct Connect hay VPN?
* downtime strategy?
* CDC (Change Data Capture)?
* RPO/RTO requirements?
* rollback plan?

Đây chính là kiểu tư duy SAP.

### 📊 Project Deliverable - Tuần 6

**Mục tiêu**: Hybrid networking + DMS simulation

✅ Commit items:
- [ ] VPC Peering setup (on-prem VPC ↔ prod VPC)
- [ ] VPN connection (site-to-site simulation)
- [ ] Test connectivity between VPCs
- [ ] Plan DMS migration (design, not execute)
- [ ] Document: migration-plan.md (cut-over, rollback, validation)
- [ ] Cost estimation: DMS = $150-200/month for 3 months
- [ ] SAP Questions: 8-10 questions on Migration/Hybrid
- [ ] Interview prep: "Design migration for legacy system" (practice)
- [ ] Test cross-VPC RDS access from Lambda

---

# Tuần 7 — Architecture + Cost + Troubleshooting + IAC

Đây sẽ là tuần **rất quan trọng cho phỏng vấn + SAP**.

Không học thêm quá nhiều service.

Thay vào đó:

### 📚 Daily Tasks

**Mỗi ngày:**
- [ ] 2 architecture problems (system design)
- [ ] 2-3 SAP questions (full practice mode)
- [ ] 1-2 mock incident scenarios (troubleshoot)
- [ ] 30 min Terraform/CloudFormation IaC coding

## Ví dụ mỗi ngày giải 2 architecture problem

Ví dụ:

### Problem 1

> E-commerce có 1 triệu users/ngày. Thiết kế AWS architecture.

Bạn tự vẽ:

```text
Route53
   ↓
CloudFront
   ↓
WAF
   ↓
ALB
   ↓
ECS
   ↓
ElastiCache
   ↓
Aurora
```

---

### Problem 2

> Website hiện tại chạy EC2. Traffic tăng 10 lần.

Bạn phải phân tích:

```text
EC2
 ↓
ALB
 ↓
Auto Scaling
```

rồi:

* cache?
* CDN?
* database bottleneck?
* asynchronous processing?
* SQS?
* read replica?

---

### Problem 3

> AWS bill tăng từ $500 → $5.000/tháng.

Bạn phải biết điều tra:

```text
Cost Explorer
    ↓
service
    ↓
resource
    ↓
CloudWatch
    ↓
optimization
```

**Quick tips:**
- Tắt NAT Gateway nếu không cần (chi phí cao)
- RDS chuyển sang graviton2 instances (30% rẻ hơn)
- Xem có EC2 nào idle không
- ElastiCache đang sử dụng bao nhiêu

---

### Problem 4

> EC2 vẫn running nhưng application không truy cập được.

Debug:

```text
DNS
 ↓
Route
 ↓
ALB
 ↓
Security Group
 ↓
NACL
 ↓
EC2
 ↓
Application
 ↓
Database
```

Đây mới là kiến thức **đi làm thực tế**.

### 📊 Project Deliverable - Tuần 7

**Mục tiêu**: Cost optimization + Terraform IaC + Troubleshooting guide

✅ Commit items:
- [ ] Infrastructure-as-Code: Convert all manual resources to Terraform
- [ ] Terraform modules (vpc, rds, ec2, etc.)
- [ ] terraform plan/apply/destroy automation
- [ ] Cost optimization report:
  - [ ] Identify waste (idle resources, oversized instances)
  - [ ] Reserved Instances analysis
  - [ ] Spot Instances consideration
  - [ ] Target: 20-30% cost reduction
- [ ] Troubleshooting guide (troubleshooting.md):
  - [ ] Networking issues (DNS, Route, SG, NACL)
  - [ ] Application down (ALB health, target group)
  - [ ] Database issues (slow queries, connection pool)
  - [ ] Permission denied (IAM policies)
- [ ] CloudWatch dashboard (custom metrics)
- [ ] CloudWatch alarms (CPU, memory, errors)
- [ ] SAP Questions: 10-12 questions on Cost/Optimization
- [ ] Interview mock: 1 full system design problem (1 hour)

---

# Tuần 8 — Final Sprint: SAP-C02 + Architecture + Interview

Tuần cuối chuyển trọng tâm (last week before exam):

**45% SAP Practice Exams**

**35% Architecture Problems**

**20% Mock Interviews**

SAP-C02 hiện phân bố:

| Domain                    | Weight |
| ------------------------- | -----: |
| Organizational Complexity |    26% |
| New Solutions             |    29% |
| Continuous Improvement    |    25% |
| Migration & Modernization |    20% |

([AWS ドキュメント][1])

Tức là **Domain 1 + 2 + 3 chiếm 80%**.

Do đó đừng dành cả tuần chỉ học migration.

### 📊 Project Deliverable - Tuần 8

**Mục tiêu**: Complete project documentation + Full SAP exam simulation

✅ Commit items:
- [ ] **Project Documentation Complete:**
  - [ ] architecture.md (with diagrams, 6 pillars review)
  - [ ] security.md (threat model, IAM policies, encryption)
  - [ ] ha-dr.md (RTO/RPO, failover testing, backup strategy)
  - [ ] cost.md (monthly breakdown, optimization suggestions)
  - [ ] troubleshooting.md (debugging guide, incident examples)
  - [ ] migration.md (how to move existing system here)
  - [ ] infrastructure-as-code.md (Terraform guide)
  - [ ] incident-response.md (3 real scenarios: RDS down, DDoS, leaked secrets)

- [ ] **SAP Exam Prep:**
  - [ ] 2 full-length practice exams (150 questions each) = 3-4 hours
  - [ ] Review all incorrect answers
  - [ ] Consolidate into "SAP_final_review.md" (key concepts by domain)
  - [ ] Target: 85%+ on practice exams

- [ ] **Mock Interviews (3 total):**
  - [ ] System Design Interview #1 (60 min)
  - [ ] Behavioral Interview (30 min) - project story
  - [ ] AWS-specific technical questions (45 min)
  - [ ] Get feedback from peer or mentor

- [ ] **Project Demo Ready:**
  - [ ] Live demo of entire system (5-10 min)
  - [ ] GitHub repo with clean code, README, architecture diagrams
  - [ ] Can show: deployment, scaling, security features, monitoring

---

# 🔥 Cách học mỗi ngày (3-4 tiếng/ngày)

Với **3-4 tiếng/ngày**, phân bổ như sau:

### ⏰ Daily Schedule (3-4h)

**Tuần 1-3: Foundation Phase**
```
30 phút  → Theory (VPC, EC2, RDS, etc.)
90 phút  → Hands-on (AWS Console)
45 phút  → Architecture problem (1 scenario)
15 phút  → Break
```

**Tuần 4-6: Integration Phase**
```
30 phút  → Theory + SAP Review (5 SAP questions)
90 phút  → Hands-on + Project deliverables
45 phút  → Architecture problem (1-2 scenarios)
15 phút  → SAP notes + break
```

**Tuần 7: Heavy Sprint Phase**
```
20 phút  → Theory (Terraform, cost optimization)
60 phút  → IAC coding (Terraform)
60 phút  → 2 Architecture problems
30 phút  → 3 SAP questions + review mistakes
10 phút  → Break
```

**Tuần 8: Exam Prep Phase**
```
90 phút  → SAP practice exam (50 questions)
60 phút  → Mock interview OR full system design
45 phút  → Troubleshooting scenarios
15 phút  → Review + consolidate notes
```

### 🎯 Learning Framework

**❌ Sai cách:**
> Xem video 2h → xem đáp án SAP → "OK hiểu rồi"

**✅ Đúng cách:**
```
Theory (đọc doc)
  ↓
Hands-on (tự làm, không copy-paste)
  ↓
Test yourself (làm 1 architecture problem)
  ↓
SAP questions (giải thích tại sao A đúng, B sai)
  ↓
Ghi chú lỗi (để review tuần sau)
```

### 💡 SAP Study Tips

❌ Không chỉ xem đáp án.

✅ Phải giải thích:
- Tại sao **A đúng**?
- Tại sao **B sai**?
- **Khi nào B lại đúng**? (edge cases)
- Liên hệ với **project thực tế** bạn đang làm

Đây là cách học **rất hiệu quả cho SAP**.

---

# 🧠 Một nguyên tắc mình muốn bạn nhớ

Đừng học AWS kiểu:

> EC2 = virtual machine
> S3 = object storage
> RDS = database

Cách đó **không đủ để đi interview AWS**.

Hãy học theo:

> **Requirement → Constraint → Architecture → Service → Trade-off**

Ví dụ:

### Requirement

"Database phải chịu được lượng read lớn."

Không hỏi:

> "RDS là gì?"

Mà hỏi:

> "Read Replica hay ElastiCache?"

> "Aurora hay RDS?"

> "Multi-AZ có giải quyết read scaling không?"

> "DynamoDB có phù hợp không?"

Đó chính là tư duy Solutions Architect.

---

# 🎯 Project Tracking

Thay vì làm 10 project nhỏ, bạn sẽ làm **1 project lớn kéo dài 8 tuần**.

## 「AWS Production-like System」

```text
Week 2:   Route 53 → ALB → EC2 × N → RDS
            (Basic HA setup)

Week 3:   + Lambda → API Gateway → SQS → Worker
            (Serverless + decoupling)

Week 4:   + WAF → CloudFront → KMS → Secrets Manager
            (Security hardening)

Week 5:   + RDS Multi-AZ → Read Replicas → Backup
            (HA + DR)

Week 6:   + VPC Peering → DMS Plan → Cross-VPC access
            (Hybrid setup)

Week 7:   + Terraform IaC → CloudWatch → Cost optimization
            (Production-ready)

Week 8:   Complete documentation + demo ready
```

### 📋 Project GitHub Structure

```
aws-production-system/
├── terraform/
│   ├── main.tf
│   ├── vpc.tf
│   ├── rds.tf
│   ├── ecs.tf
│   ├── variables.tf
│   └── outputs.tf
├── docs/
│   ├── architecture.md
│   ├── security.md
│   ├── ha-dr.md
│   ├── cost.md
│   ├── troubleshooting.md
│   ├── migration.md
│   ├── incident-response.md
│   └── deployment-guide.md
├── lambda/
│   ├── api-handler/
│   └── worker-handler/
├── backend-code/ (Node.js/Python API)
└── README.md
```

**Đây sẽ trở thành portfolio + tài liệu ôn interview + tài liệu ôn SAP cùng lúc.**

---

### Một lưu ý về SAP

**SAP-C02 là chứng chỉ Professional**, và AWS mô tả target là người có kỹ năng nâng cao trong việc thiết kế giải pháp tối ưu theo Well-Architected Framework. ([AWS ドキュメント][2])

Vì vậy trong 2 tháng này, **đừng đặt mục tiêu "học hết AWS"**. Không thể.

Mục tiêu nên là:

> **8 tuần xây được tư duy AWS Architecture + thực chiến + bắt đầu chinh phục SAP-C02.**

Nếu học đúng cách, **2 tháng sau bạn có thể bắt đầu apply AWS/Cloud Engineer và tiếp tục luyện SAP song song**, thay vì chờ đến khi "học AWS xong" — vì thực tế sẽ không bao giờ có thời điểm AWS được học xong.

---

# Tuần 9 — Buffer + Exam Week

> Đừng lo, tuần này để "thôi miên" trước khi đi thi hoặc nếu bạn lag lộ trình

### Plan (chỉ nếu cần)

```
Nếu tuần 1-8 hoàn thành 100%:
  └─ Tuần 9: Nghỉ ngơi 2-3 ngày, rồi:
      ├─ 1-2 full SAP exams
      ├─ Final project review
      └─ Exam day!

Nếu lag 1-2 tuần:
  └─ Tuần 9: Catch-up tuần 7-8 deliverables
```

---

# ✅ Commit Checklist

**Trước khi bắt đầu, hãy confirm:**

- [ ] Có thể commit **3-4h/ngày** liên tục trong 9 tuần
- [ ] Có AWS account (free tier ok)
- [ ] Cài Terraform hoặc CloudFormation (từ tuần 2)
- [ ] Set up GitHub repo cho project
- [ ] Tải SAP study materials (Udemy course hoặc A Cloud Guru)
- [ ] Bookmark AWS Well-Architected Framework (sẽ dùng liên tục)
- [ ] Join AWS community (r/aws, AWS Forum) để hỏi khi bị stuck

---

# 🎓 Success Metrics

**Sau 9 tuần, bạn sẽ có:**

✅ 1 production-like AWS system (Terraform + docs)
✅ 8 documentation files (architecture, security, cost, etc.)
✅ 3+ mock interviews (pass grade)
✅ 85%+ on SAP practice exams
✅ Can explain any AWS service in 2 minutes
✅ Can design systems following 6 pillars
✅ Can debug AWS issues systematically
✅ Ready for AWS job + ready for SAP-C02 exam

---

# 🚀 Let's Go!

**Start date:** [2026-08-26]
**Exam date:** [2026-10-28] (9 weeks later)
**Job applications:** Ready from week 7

Bạn sẽ làm tốt! 💪

[1]: https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02.html?utm_source=chatgpt.com "AWS Certified Solutions Architect - Professional (SAP-C02) - AWS Certified Solutions Architect - Professional"
[2]: https://docs.aws.amazon.com/pdfs/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02.pdf?utm_source=chatgpt.com "AWS Certified Solutions Architect - Professional - Exam Guide (SAP-C02)"
[3]: https://docs.aws.amazon.com/wellarchitected/2025-02-25/framework/the-pillars-of-the-framework.html?utm_source=chatgpt.com "The pillars of the framework - AWS Well-Architected Framework"
