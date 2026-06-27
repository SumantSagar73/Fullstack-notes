# Ultimate AWS for Backend Developers — Practical Guide with Hands-On Exercises

> **Who is this for?** Backend developers who know how to build APIs but have never deployed them to the cloud, or who use AWS at work but don't understand WHY things are set up the way they are. This guide teaches you AWS the way a backend developer needs it — not as a cloud architect, but as someone who ships, scales, and maintains production backends.

> **Prerequisites:** An AWS account (free tier works for most exercises), AWS CLI installed, basic Docker knowledge (from your Docker notes), and basic terminal skills.

---

## Table of Contents

**Part 1 — AWS Foundations**

1. [AWS Mental Model — How to Think About the Cloud](#1-aws-mental-model)
2. [AWS CLI Setup & Your First Commands](#2-aws-cli-setup--your-first-commands)
3. [IAM — Who Can Do What](#3-iam--who-can-do-what)
4. [Regions, Availability Zones & Edge Locations](#4-regions-availability-zones--edge-locations)

**Part 2 — Compute (Where Your Code Runs)**

5. [EC2 — Virtual Servers](#5-ec2--virtual-servers)
6. [Lambda — Functions Without Servers](#6-lambda--functions-without-servers)
7. [ECS & Fargate — Running Docker Containers](#7-ecs--fargate--running-docker-containers)
8. [When to Use EC2 vs Lambda vs ECS](#8-when-to-use-ec2-vs-lambda-vs-ecs)

**Part 3 — Storage & Databases**

9. [S3 — Store Anything, Serve Anything](#9-s3--store-anything-serve-anything)
10. [RDS — Managed Relational Databases](#10-rds--managed-relational-databases)
11. [DynamoDB — NoSQL at Any Scale](#11-dynamodb--nosql-at-any-scale)
12. [ElastiCache — Managed Redis/Memcached](#12-elasticache--managed-redismemcached)

**Part 4 — Networking & Traffic**

13. [VPC — Your Private Network in the Cloud](#13-vpc--your-private-network-in-the-cloud)
14. [ALB — Application Load Balancer](#14-alb--application-load-balancer)
15. [Route 53 — DNS & Domain Management](#15-route-53--dns--domain-management)
16. [CloudFront — CDN](#16-cloudfront--cdn)
17. [API Gateway — Managed API Layer](#17-api-gateway--managed-api-layer)

**Part 5 — Messaging & Async**

18. [SQS — Simple Queue Service](#18-sqs--simple-queue-service)
19. [SNS — Simple Notification Service](#19-sns--simple-notification-service)
20. [EventBridge — Event Bus](#20-eventbridge--event-bus)

**Part 6 — Monitoring, Security & DevOps**

21. [CloudWatch — Logs, Metrics & Alarms](#21-cloudwatch--logs-metrics--alarms)
22. [Secrets Manager & Parameter Store](#22-secrets-manager--parameter-store)
23. [ECR — Container Registry](#23-ecr--container-registry)

**Part 7 — Real Architectures**

24. [Architecture 1: Simple API Backend](#24-architecture-1-simple-api-backend)
25. [Architecture 2: Scalable Microservices](#25-architecture-2-scalable-microservices)
26. [Architecture 3: Serverless Backend](#26-architecture-3-serverless-backend)
27. [Architecture 4: Event-Driven Processing](#27-architecture-4-event-driven-processing)
28. [Cost Optimization — Don't Go Bankrupt](#28-cost-optimization--dont-go-bankrupt)

**Part 8 — Practical Exercises**

29. [Exercise 1: Deploy a Node.js API to EC2](#29-exercise-1-deploy-a-nodejs-api-to-ec2)
30. [Exercise 2: Serverless REST API with Lambda + API Gateway + DynamoDB](#30-exercise-2-serverless-rest-api-with-lambda--api-gateway--dynamodb)
31. [Exercise 3: Dockerized App on ECS Fargate](#31-exercise-3-dockerized-app-on-ecs-fargate)
32. [Exercise 4: File Upload System with S3 + Pre-signed URLs](#32-exercise-4-file-upload-system-with-s3--pre-signed-urls)
33. [Exercise 5: Background Job Processing with SQS](#33-exercise-5-background-job-processing-with-sqs)
34. [Exercise 6: Full Production Setup (VPC + ALB + ECS + RDS + ElastiCache)](#34-exercise-6-full-production-setup)
35. [Exercise 7: Event-Driven Architecture (SNS + SQS + Lambda)](#35-exercise-7-event-driven-architecture)
36. [Cleanup — Don't Get Surprise Bills](#36-cleanup--dont-get-surprise-bills)

**Part 9 — Interview Questions**

37. [AWS Interview Questions (60+ Questions)](#37-aws-interview-questions-60-questions)

---

# PART 1 — AWS FOUNDATIONS

---

## 1. AWS Mental Model

### The Analogy: AWS Is a Massive Building Supplies Store

Building a backend application is like building a house. You could buy land, make bricks, build everything yourself (on-premise). Or you could go to a building supplies store that has everything — pre-made walls, plumbing, electricity, security systems — and you just assemble what you need.

AWS is that store. Every service is a building block:

| What You Need | AWS Service | Analogy |
|---------------|-------------|---------|
| A server to run my code | EC2 | Renting an apartment |
| Run code without a server | Lambda | Hiring a freelancer per task |
| Run Docker containers | ECS / Fargate | Managed container apartments |
| Store files (images, PDFs) | S3 | A warehouse with infinite shelves |
| Relational database | RDS | Managed database with a DBA included |
| NoSQL database | DynamoDB | A key-value filing cabinet that scales infinitely |
| Cache (Redis) | ElastiCache | A whiteboard with sticky notes for quick lookups |
| Message queue | SQS | A to-do list that multiple workers pull from |
| Pub/sub notifications | SNS | A broadcast system — one message, many receivers |
| Load balancer | ALB / NLB | A receptionist directing visitors to available offices |
| DNS | Route 53 | The address book of the internet |
| CDN | CloudFront | Warehouses in every city for faster delivery |
| API management | API Gateway | A front door with a security guard |
| Container registry | ECR | A library of Docker images |
| Secrets storage | Secrets Manager | A locked safe for passwords and API keys |
| Monitoring | CloudWatch | Security cameras + performance dashboards |

### The Three Rules of AWS for Backend Devs

**Rule 1: Managed > Self-managed.** Don't run your own PostgreSQL on EC2. Use RDS. Don't manage your own Redis cluster. Use ElastiCache. AWS managing it means automatic backups, patching, failover, and monitoring. You focus on your application.

**Rule 2: Serverless when possible, containers when needed, VMs as last resort.** Lambda (no servers to manage) > ECS Fargate (containers without managing servers) > EC2 (full VM you manage). Each step adds operational burden.

**Rule 3: Everything fails. Design for it.** Use multiple Availability Zones. Add health checks. Use auto-scaling. If one server dies, another takes over automatically.

---

## 2. AWS CLI Setup & Your First Commands

### Install AWS CLI

```bash
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify
aws --version
```

### Configure Credentials

```bash
aws configure
# AWS Access Key ID: AKIA...     (from IAM console)
# AWS Secret Access Key: wJal... (from IAM console)
# Default region name: ap-south-1 (Mumbai — closest to you)
# Default output format: json
```

### Your First Commands

```bash
# Who am I?
aws sts get-caller-identity

# List S3 buckets
aws s3 ls

# List EC2 instances
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType]' --output table

# List Lambda functions
aws lambda list-functions --query 'Functions[].FunctionName'
```

---

## 3. IAM — Who Can Do What

### The Analogy

IAM is the security system of your AWS account. Think of it as key cards in an office building. The CEO (root account) can access everything. A developer gets a key card (IAM user) that opens specific doors (permissions). A contractor gets a temporary visitor pass (IAM role).

### Key Concepts

| Concept | What It Is | Analogy |
|---------|-----------|---------|
| **User** | A person or application with credentials | An employee with an ID card |
| **Group** | A collection of users | The "Engineering" department |
| **Policy** | A JSON document defining permissions | The list of doors a key card can open |
| **Role** | A set of permissions that can be assumed by services/users | A visitor badge anyone can wear temporarily |

### The Most Important Rule

**Never use the root account for daily work.** Create an IAM user with admin permissions and use that. If the root credentials leak, your entire AWS account is compromised.

### Practical: Create Your First IAM User

```bash
# Create a user
aws iam create-user --user-name backend-dev

# Attach the PowerUserAccess policy (everything except IAM management)
aws iam attach-user-policy \
  --user-name backend-dev \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Create access keys (for CLI access)
aws iam create-access-key --user-name backend-dev
```

### IAM Policies — How Permissions Work

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-app-uploads/*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteBucket",
      "Resource": "*"
    }
  ]
}
```

Read it as: "Allow reading and writing to the my-app-uploads bucket. Deny deleting any bucket."

### IAM Roles — The One You'll Use Most

EC2 instances, Lambda functions, and ECS containers don't use access keys. They use **IAM Roles**. You attach a role to the service, and AWS automatically provides temporary credentials.

```
Your Lambda function needs to read from DynamoDB?
→ Create a role with DynamoDB read permissions
→ Attach the role to the Lambda function
→ The function can now read DynamoDB — no hardcoded keys
```

This is the most secure approach — no credentials in your code, no keys to rotate.

---

## 4. Regions, Availability Zones & Edge Locations

```
AWS Global Infrastructure:

Region (ap-south-1 = Mumbai)
├── Availability Zone 1 (ap-south-1a) — a separate data center
├── Availability Zone 2 (ap-south-1b) — another data center, miles away
└── Availability Zone 3 (ap-south-1c) — yet another

If AZ 1 loses power → AZ 2 and 3 keep running
```

**Region:** A geographic location (Mumbai, Singapore, US East). Choose the region closest to your users. For India, `ap-south-1` (Mumbai).

**Availability Zone (AZ):** Independent data centers within a region. Deploy across multiple AZs for high availability. If one data center floods, burns, or loses power, the others keep running.

**Edge Location:** CDN points of presence. CloudFront has 400+ edge locations worldwide for fast content delivery.

**Rule:** For production, always deploy across at least 2 AZs. This is how you get high availability without multi-region complexity.

---

# PART 2 — COMPUTE

---

## 5. EC2 — Virtual Servers

### What It Is

EC2 (Elastic Compute Cloud) is a virtual machine in the cloud. You pick the CPU, RAM, disk, and OS. It's like renting a computer in a data center.

### Instance Types (What Backend Devs Need to Know)

| Family | Optimised For | Use Case |
|--------|-------------|----------|
| **t3/t3a** | General purpose, burstable | Development, small APIs, low traffic |
| **m5/m6i** | Balanced CPU + RAM | Production APIs, medium traffic |
| **c5/c6i** | CPU-intensive | Heavy computation, video transcoding |
| **r5/r6i** | Memory-intensive | In-memory caching, large datasets |

**For most backend APIs:** Start with `t3.micro` (free tier) for development, `t3.small` or `t3.medium` for staging, `m5.large` or `m6i.large` for production.

### Key Concepts

**Security Groups:** Firewall rules for your EC2 instance. "Allow traffic on port 3000 from anywhere" or "Allow port 5432 only from my app servers."

```bash
# Allow HTTP and SSH
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp --port 22 --cidr 0.0.0.0/0      # SSH (restrict in production!)

aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp --port 3000 --cidr 0.0.0.0/0     # Your API port
```

**Key Pairs:** SSH keys for connecting to the instance. Create once, use for all instances.

**Elastic IP:** A static public IP that stays the same even if you stop and restart the instance.

**User Data:** A startup script that runs when the instance launches — use it to install dependencies, pull code, and start your app.

---

## 6. Lambda — Functions Without Servers

### What It Is

Lambda runs your code in response to events — without provisioning or managing servers. You upload a function, AWS runs it when triggered. You pay only for the compute time you consume.

**Analogy:** EC2 is renting an apartment (you pay monthly whether you're home or not). Lambda is a hotel room (you pay only for the nights you stay).

### When to Use Lambda (Backend Dev Perspective)

| Good For | Bad For |
|----------|---------|
| API endpoints with variable traffic | Long-running processes (> 15 min) |
| Webhooks | WebSocket connections |
| Scheduled tasks (cron) | High-throughput, low-latency APIs |
| Event processing (S3 upload → process image) | Applications needing persistent connections |
| Background jobs | GPU-intensive work |

### Cold Start — The One Downside

When a Lambda function hasn't been called recently, AWS needs to initialise the environment (~200ms to 2s). This is called a **cold start**. Subsequent calls are fast ("warm"). For latency-sensitive APIs, this matters.

**Mitigations:** Provisioned Concurrency (pre-warm instances), use lighter runtimes (Node.js/Python cold starts are faster than Java), keep functions small.

### Lambda with Node.js

```javascript
// handler.js — a simple Lambda function
export const handler = async (event) => {
  const { name } = JSON.parse(event.body);

  return {
    statusCode: 200,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      message: `Hello, ${name}!`,
      timestamp: new Date().toISOString(),
    }),
  };
};
```

---

## 7. ECS & Fargate — Running Docker Containers

### What It Is

ECS (Elastic Container Service) runs Docker containers on AWS. Fargate is the "serverless" mode — you don't manage the underlying servers. You just say "run this Docker image with 1 CPU and 2GB RAM" and AWS handles the rest.

### ECS Concepts

| Concept | What It Is | Analogy |
|---------|-----------|---------|
| **Task Definition** | A blueprint for your container (image, CPU, memory, ports, env vars) | A Dockerfile on steroids |
| **Task** | A running instance of a task definition | A running container |
| **Service** | Ensures N tasks are always running, handles scaling and load balancing | A supervisor that restarts containers if they crash |
| **Cluster** | A logical grouping of services and tasks | A project namespace |

### EC2 Launch Type vs Fargate

| | EC2 Launch Type | Fargate |
|-|----------------|---------|
| **You manage** | EC2 instances + containers | Only containers |
| **Scaling** | You scale EC2 instances | AWS scales automatically |
| **Cost** | Cheaper for steady workloads | Cheaper for variable workloads |
| **Use when** | Predictable, high volume | Variable traffic, less ops |

**For most backend devs:** Start with Fargate. It's simpler and you don't manage servers.

---

## 8. When to Use EC2 vs Lambda vs ECS

```
"I need a long-running server with full control"
  → EC2

"I need to run code in response to events, it runs for < 15 min"
  → Lambda

"I have a Dockerized app and want AWS to manage the infrastructure"
  → ECS Fargate

"I have a Dockerized app with steady, high traffic and want to save money"
  → ECS on EC2

"I need a simple API with variable traffic"
  → Lambda + API Gateway (cheapest, simplest)

"I need a production API with WebSockets, background jobs, and persistent connections"
  → ECS Fargate (or EC2 for cost optimization)
```

---

# PART 3 — STORAGE & DATABASES

---

## 9. S3 — Store Anything, Serve Anything

### What It Is

S3 (Simple Storage Service) is object storage — store any file (images, videos, PDFs, backups, logs) with virtually unlimited capacity. Each file (object) lives in a bucket, identified by a key (path).

### S3 for Backend Developers

```
Your API receives a file upload:
  → Store the file in S3
  → Save the S3 URL/key in your database
  → Serve the file via CloudFront CDN (or pre-signed URL for private files)
```

### Key Concepts

**Bucket:** A container for objects. Globally unique name. One bucket per project/purpose.

**Object:** A file + metadata. Max size 5TB. Identified by a key (path-like string).

**Pre-signed URLs:** Temporary URLs that allow upload/download without exposing your credentials.

```javascript
// Generate a pre-signed upload URL (Node.js)
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');

const s3 = new S3Client({ region: 'ap-south-1' });

async function getUploadUrl(fileName, contentType) {
  const command = new PutObjectCommand({
    Bucket: 'my-app-uploads',
    Key: `uploads/${Date.now()}-${fileName}`,
    ContentType: contentType,
  });

  const url = await getSignedUrl(s3, command, { expiresIn: 300 }); // 5 min
  return url;
}

// API endpoint
app.post('/api/upload-url', async (req, res) => {
  const url = await getUploadUrl(req.body.fileName, req.body.contentType);
  res.json({ uploadUrl: url });
  // Client uploads directly to S3 using this URL — your server never touches the file
});
```

### Storage Classes (Cost Optimization)

| Class | Use Case | Cost |
|-------|----------|------|
| **Standard** | Frequently accessed files | $$$  |
| **Infrequent Access (IA)** | Backups, files accessed < 1x/month | $$ |
| **Glacier** | Archives, accessed < 1x/year | $ |
| **Intelligent-Tiering** | Auto-moves between classes based on access | Auto |

### CLI Commands

```bash
# Create a bucket
aws s3 mb s3://my-backend-app-uploads-2024

# Upload a file
aws s3 cp ./local-file.pdf s3://my-backend-app-uploads-2024/documents/

# List files
aws s3 ls s3://my-backend-app-uploads-2024/documents/

# Download a file
aws s3 cp s3://my-backend-app-uploads-2024/documents/file.pdf ./

# Sync a directory
aws s3 sync ./static-assets s3://my-backend-app-uploads-2024/assets/

# Delete a file
aws s3 rm s3://my-backend-app-uploads-2024/documents/file.pdf
```

---

## 10. RDS — Managed Relational Databases

### What It Is

RDS (Relational Database Service) manages PostgreSQL, MySQL, MariaDB, Oracle, and SQL Server for you. Automatic backups, patching, replication, failover — you just use the database.

### What You Get (That You'd Have to Do Yourself on EC2)

| Feature | On EC2 | On RDS |
|---------|--------|--------|
| Install database | You | AWS |
| Security patches | You | AWS (automatic) |
| Backups | You (set up cron + S3) | AWS (automatic, point-in-time recovery) |
| Read replicas | You (complex setup) | AWS (one click) |
| Failover | You (complex setup) | AWS (automatic with Multi-AZ) |
| Monitoring | You (install tools) | AWS (CloudWatch integration) |
| Scaling | You (painful) | AWS (change instance type) |

### Key Concepts for Backend Devs

**Multi-AZ Deployment:** RDS automatically maintains a standby replica in a different AZ. If the primary fails, AWS automatically switches to the standby. Your connection string doesn't change.

**Read Replicas:** Create copies of your database that handle read queries. Your write traffic goes to the primary; reads go to replicas. Great for read-heavy workloads.

```
Application → primary-db.xxx.rds.amazonaws.com (writes)
            → replica-db.xxx.rds.amazonaws.com (reads)
```

**Parameter Groups:** Database configuration (max_connections, work_mem, etc.) managed as a named configuration set. Apply different configs to different environments.

### CLI: Create an RDS Instance

```bash
aws rds create-db-instance \
  --db-instance-identifier my-api-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version 16.1 \
  --master-username admin \
  --master-user-password YourSecurePassword123 \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-xxx \
  --db-subnet-group-name my-db-subnet-group \
  --no-publicly-accessible \
  --backup-retention-period 7 \
  --multi-az
```

**Critical:** `--no-publicly-accessible` — your database should NEVER be accessible from the internet. It should only accept connections from your app servers within the same VPC.

---

## 11. DynamoDB — NoSQL at Any Scale

### When to Use DynamoDB (Not PostgreSQL)

| Use DynamoDB When | Use RDS/PostgreSQL When |
|-------------------|------------------------|
| Simple access patterns (get by ID, query by partition key) | Complex joins and relationships |
| Massive scale (millions of reads/writes per second) | Complex queries (GROUP BY, subqueries) |
| Key-value or document storage | Transactions across multiple tables |
| Consistent single-digit millisecond latency | Ad-hoc reporting and analytics |
| Serverless architecture (pairs perfectly with Lambda) | Your team knows SQL well |

### Key Concepts

**Partition Key:** The primary lookup key. DynamoDB uses it to distribute data across servers. Must be chosen carefully — high cardinality (like userId) distributes evenly.

**Sort Key:** An optional secondary key for ordering and querying within a partition.

```
Table: Orders
Partition Key: userId
Sort Key: orderDate

This allows:
- Get all orders for user 42 → query by partition key
- Get user 42's orders in January 2024 → query by partition + sort key range
- Get user 42's latest order → query by partition + sort key DESC, limit 1
```

### Node.js with DynamoDB

```javascript
const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, PutCommand, GetCommand, QueryCommand } = require('@aws-sdk/lib-dynamodb');

const client = new DynamoDBClient({ region: 'ap-south-1' });
const dynamo = DynamoDBDocumentClient.from(client);

// Write
await dynamo.send(new PutCommand({
  TableName: 'Orders',
  Item: {
    userId: 'user-42',
    orderDate: '2024-01-15T10:30:00Z',
    total: 1500,
    status: 'confirmed',
    items: [{ productId: 'p-1', quantity: 2 }],
  },
}));

// Read by key
const result = await dynamo.send(new GetCommand({
  TableName: 'Orders',
  Key: { userId: 'user-42', orderDate: '2024-01-15T10:30:00Z' },
}));

// Query all orders for a user
const orders = await dynamo.send(new QueryCommand({
  TableName: 'Orders',
  KeyConditionExpression: 'userId = :uid',
  ExpressionAttributeValues: { ':uid': 'user-42' },
  ScanIndexForward: false,  // DESC order
  Limit: 10,
}));
```

---

## 12. ElastiCache — Managed Redis/Memcached

### What It Is

ElastiCache manages Redis or Memcached clusters for you. Same Redis commands you know, but AWS handles replication, failover, patching, and backups.

### Use Cases for Backend Devs

- **Session storage** — store user sessions in Redis instead of server memory
- **API response caching** — cache database queries, reduce DB load
- **Rate limiting** — Redis INCR with TTL
- **Leaderboards** — Redis sorted sets
- **Real-time data** — pub/sub, recent activity feeds

### Connecting Your App

```javascript
const Redis = require('ioredis');

// Connect to ElastiCache Redis
const redis = new Redis({
  host: 'my-cache.xxx.cache.amazonaws.com',
  port: 6379,
  // ElastiCache is NOT accessible from the internet
  // Your app MUST be in the same VPC
});

// Same Redis commands you already know
await redis.set('user:42:session', JSON.stringify(sessionData), 'EX', 3600);
const session = await redis.get('user:42:session');
```

---

# PART 4 — NETWORKING & TRAFFIC

---

## 13. VPC — Your Private Network in the Cloud

### The Analogy

A VPC (Virtual Private Cloud) is your own private office building in AWS. You control who can enter (security groups), which floors are accessible (subnets), and which floors can see the internet (public vs private subnets).

```
VPC (10.0.0.0/16) — Your private building
├── Public Subnet (10.0.1.0/24) — Lobby (accessible from outside)
│   ├── Load Balancer (front door)
│   └── NAT Gateway (lets private resources access internet)
│
├── Private Subnet (10.0.2.0/24) — Offices (NOT accessible from outside)
│   ├── App Servers (EC2/ECS)
│   └── Lambda functions
│
└── Private Subnet (10.0.3.0/24) — Vault (most restricted)
    ├── RDS Database
    └── ElastiCache
```

### The Rules

1. **Load balancers** → public subnets (they receive internet traffic)
2. **App servers** → private subnets (only accessible via load balancer)
3. **Databases** → private subnets (only accessible from app servers)
4. **NAT Gateway** → lets private resources access the internet (for downloading packages, calling external APIs) without being accessible FROM the internet

### Why This Matters

Without a VPC, your database is on the public internet. Anyone with the connection string can try to connect. With a VPC, the database is in a private subnet — the only way in is through your app servers, which are only accessible through the load balancer.

---

## 14. ALB — Application Load Balancer

### What It Is

ALB (Application Load Balancer) distributes incoming HTTP/HTTPS traffic across your app servers. It's Layer 7 — it understands HTTP and can route based on URL paths, headers, and hostnames.

### Path-Based Routing (Microservices)

```
api.myapp.com/users/*    → User Service (Target Group 1)
api.myapp.com/orders/*   → Order Service (Target Group 2)
api.myapp.com/products/* → Product Service (Target Group 3)
```

One load balancer, multiple services. Each service can scale independently.

### Health Checks

ALB pings each server's `/health` endpoint every 30 seconds. If a server fails 3 consecutive health checks, ALB removes it from the pool. When it recovers, ALB adds it back.

---

## 15-17. Route 53, CloudFront & API Gateway

### Route 53 (DNS)
Maps domain names to AWS resources. `api.myapp.com → ALB`. Supports health-check-based failover, geolocation routing, and weighted routing (send 10% of traffic to the new version).

### CloudFront (CDN)
Caches your content at 400+ edge locations. Put your S3 static files or API responses behind CloudFront. Users in Delhi get content from the Mumbai edge location (~5ms) instead of your US server (~200ms).

### API Gateway
A fully managed API layer for Lambda-based backends. Handles authentication, throttling, request/response transformation, and caching. Pairs with Lambda to create serverless APIs without any server management.

---

# PART 5 — MESSAGING & ASYNC

---

## 18. SQS — Simple Queue Service

### What It Is

SQS is a fully managed message queue. Producers send messages, consumers poll and process them. Messages are stored reliably until processed. If a consumer crashes, the message becomes visible again for another consumer.

### Two Types

| | Standard Queue | FIFO Queue |
|-|---------------|-----------|
| **Ordering** | Best-effort (mostly ordered) | Strictly ordered |
| **Duplicates** | Possible (rare) | Exactly-once processing |
| **Throughput** | Unlimited | 3,000 messages/sec |
| **Use when** | Email sending, image processing | Financial transactions, order processing |

### Node.js with SQS

```javascript
const { SQSClient, SendMessageCommand, ReceiveMessageCommand, DeleteMessageCommand } = require('@aws-sdk/client-sqs');

const sqs = new SQSClient({ region: 'ap-south-1' });
const QUEUE_URL = 'https://sqs.ap-south-1.amazonaws.com/123456789/my-queue';

// Producer: Send a message
await sqs.send(new SendMessageCommand({
  QueueUrl: QUEUE_URL,
  MessageBody: JSON.stringify({
    type: 'send_email',
    to: 'ravi@example.com',
    subject: 'Order Confirmed',
    orderId: 'ORD-123',
  }),
}));

// Consumer: Poll and process messages
async function processMessages() {
  while (true) {
    const response = await sqs.send(new ReceiveMessageCommand({
      QueueUrl: QUEUE_URL,
      MaxNumberOfMessages: 10,
      WaitTimeSeconds: 20,  // Long polling (efficient — waits for messages)
    }));

    for (const message of response.Messages || []) {
      const body = JSON.parse(message.Body);

      // Process the message
      await sendEmail(body.to, body.subject, body.orderId);

      // Delete after successful processing
      await sqs.send(new DeleteMessageCommand({
        QueueUrl: QUEUE_URL,
        ReceiptHandle: message.ReceiptHandle,
      }));
    }
  }
}
```

### Dead Letter Queue (DLQ)

If a message fails processing 3 times, SQS automatically moves it to a DLQ. You can inspect failed messages, fix the issue, and replay them.

---

## 19. SNS — Simple Notification Service

### SNS vs SQS

| | SQS | SNS |
|-|-----|-----|
| **Model** | Queue (1 producer → 1 consumer) | Pub/Sub (1 producer → many consumers) |
| **Analogy** | A to-do list | A broadcast speaker |
| **Use when** | One worker processes each message | Multiple services react to one event |

### SNS + SQS = Fan-Out Pattern

```
Order Service → SNS Topic "order-placed"
                    │
                    ├── SQS Queue → Email Service
                    ├── SQS Queue → Inventory Service
                    ├── SQS Queue → Analytics Service
                    └── SQS Queue → Notification Service
```

One event, multiple consumers, each with their own queue (so they process independently at their own pace).

---

## 20. EventBridge — Event Bus

EventBridge is a serverless event bus for building event-driven architectures. It can filter events based on content and route them to different targets.

```
Source: "order-service"
Event: { "type": "OrderPlaced", "total": 5000 }

Rule 1: If total > 1000 → trigger fraud-check Lambda
Rule 2: All OrderPlaced events → send to analytics SQS queue
Rule 3: If type = "OrderCancelled" → trigger refund Lambda
```

---

# PART 6 — MONITORING, SECURITY & DEVOPS

---

## 21. CloudWatch — Logs, Metrics & Alarms

### Logs

Every AWS service can send logs to CloudWatch. Your Lambda functions, ECS containers, and API Gateway automatically send logs there.

```javascript
// In Lambda / ECS, just use console.log — it goes to CloudWatch automatically
console.log(JSON.stringify({
  event: 'order_placed',
  orderId: 'ORD-123',
  userId: 'user-42',
  total: 1500,
}));
```

### Metrics & Alarms

```bash
# Create an alarm: notify if API errors exceed 10 in 5 minutes
aws cloudwatch put-metric-alarm \
  --alarm-name "HighErrorRate" \
  --metric-name "5XXError" \
  --namespace "AWS/ApplicationELB" \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:ap-south-1:123456789:alerts-topic
```

---

## 22. Secrets Manager & Parameter Store

**Never hardcode secrets in your code or environment variables in task definitions.**

```bash
# Store a secret
aws secretsmanager create-secret \
  --name "prod/my-api/database-url" \
  --secret-string "postgresql://admin:password@my-db.xxx.rds.amazonaws.com:5432/myapp"

# Retrieve in your app
```

```javascript
const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');

const client = new SecretsManagerClient({ region: 'ap-south-1' });

async function getSecret(secretName) {
  const response = await client.send(
    new GetSecretValueCommand({ SecretId: secretName })
  );
  return response.SecretString;
}

// On app startup
const dbUrl = await getSecret('prod/my-api/database-url');
```

---

## 23. ECR — Container Registry

ECR (Elastic Container Registry) stores your Docker images. It's like Docker Hub but private, within your AWS account.

```bash
# Login to ECR
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 123456789.dkr.ecr.ap-south-1.amazonaws.com

# Create a repository
aws ecr create-repository --repository-name my-api

# Tag and push your image
docker tag my-api:latest 123456789.dkr.ecr.ap-south-1.amazonaws.com/my-api:latest
docker push 123456789.dkr.ecr.ap-south-1.amazonaws.com/my-api:latest
```

---

# PART 7 — REAL ARCHITECTURES

---

## 24. Architecture 1: Simple API Backend

```
Route 53 (DNS)
    ↓
CloudFront (CDN + SSL)
    ↓
ALB (Load Balancer)
    ↓
ECS Fargate (2 containers, auto-scaling)
    ↓
RDS PostgreSQL (Multi-AZ) + ElastiCache Redis

Static files → S3 + CloudFront
Secrets → Secrets Manager
Logs → CloudWatch
```

**Cost:** ~$100-200/month for low-medium traffic. This handles thousands of requests per second.

---

## 25. Architecture 2: Scalable Microservices

```
Route 53 → CloudFront → ALB (path-based routing)
                            │
                ┌───────────┼───────────┐
                ↓           ↓           ↓
          User Service  Order Service  Product Service
          (ECS Fargate)  (ECS Fargate)  (ECS Fargate)
                ↓           ↓           ↓
          RDS (users)   RDS (orders)   DynamoDB (products)
                            │
                       SNS → SQS → Email Worker (Lambda)
                                 → Analytics Worker (Lambda)
```

---

## 26. Architecture 3: Serverless Backend

```
Route 53 → API Gateway → Lambda Functions
                              ↓
                         DynamoDB
                              ↓
                    S3 (file uploads)
                              ↓
                    SQS → Lambda Workers (background jobs)
```

**Cost:** Near zero at low traffic (pay per request). Scales automatically to millions of requests. No servers to manage.

---

## 27. Architecture 4: Event-Driven Processing

```
API → SNS Topic "order-events"
          │
          ├── SQS → Lambda: Send confirmation email
          ├── SQS → Lambda: Update inventory
          ├── SQS → Lambda: Process payment
          ├── SQS → Lambda: Send analytics
          └── SQS → Lambda: Generate invoice (→ S3)
```

---

## 28. Cost Optimization

### The Rules That Save Money

1. **Right-size instances.** Don't use m5.xlarge when t3.small handles your traffic.
2. **Use Reserved Instances or Savings Plans** for steady workloads (up to 72% savings).
3. **Use Spot Instances** for batch processing (up to 90% savings, but can be interrupted).
4. **Delete what you don't use.** Unused EBS volumes, old snapshots, idle load balancers — they all cost money.
5. **Use S3 lifecycle policies** to move old files to cheaper storage classes.
6. **Set billing alerts.** Always.

```bash
# Set a billing alarm at $50
aws cloudwatch put-metric-alarm \
  --alarm-name "BillingAlarm-50USD" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --threshold 50 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789:billing-alerts
```

---

# PART 8 — PRACTICAL EXERCISES

---

## 29. Exercise 1: Deploy a Node.js API to EC2

### What You'll Learn
Launching an EC2 instance, SSHing in, installing Node.js, running your API, and making it accessible.

### Steps

```bash
# 1. Create a key pair (for SSH access)
aws ec2 create-key-pair --key-name my-api-key --query 'KeyMaterial' --output text > my-api-key.pem
chmod 400 my-api-key.pem

# 2. Create a security group
aws ec2 create-security-group --group-name my-api-sg --description "API server SG"
SG_ID=$(aws ec2 describe-security-groups --group-names my-api-sg --query 'SecurityGroups[0].GroupId' --output text)

# 3. Allow SSH (port 22) and API (port 3000)
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 3000 --cidr 0.0.0.0/0

# 4. Launch an EC2 instance (Amazon Linux 2, t3.micro = free tier)
aws ec2 run-instances \
  --image-id ami-0f5ee92e2d63afc18 \
  --instance-type t3.micro \
  --key-name my-api-key \
  --security-group-ids $SG_ID \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=my-api-server}]' \
  --user-data '#!/bin/bash
    yum update -y
    curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
    yum install -y nodejs git
    cd /home/ec2-user
    cat > app.js << EOF
const http = require("http");
const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ message: "Hello from EC2!", timestamp: new Date() }));
});
server.listen(3000, () => console.log("Server running on port 3000"));
EOF
    node app.js &'

# 5. Get the public IP
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=my-api-server" \
  --query 'Reservations[].Instances[].PublicIpAddress' --output text

# 6. Test it (wait 1-2 minutes for the instance to boot)
curl http://<PUBLIC_IP>:3000
# {"message":"Hello from EC2!","timestamp":"2024-01-15T10:30:00.000Z"}
```

### Cleanup
```bash
# Find and terminate the instance
INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=my-api-server" --query 'Reservations[].Instances[].InstanceId' --output text)
aws ec2 terminate-instances --instance-ids $INSTANCE_ID
```

---

## 30. Exercise 2: Serverless REST API with Lambda + API Gateway + DynamoDB

### What You'll Learn
Building a complete CRUD API without any servers.

### Steps

```bash
# 1. Create DynamoDB table
aws dynamodb create-table \
  --table-name Todos \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# 2. Create the Lambda execution role
aws iam create-role \
  --role-name lambda-todo-role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": { "Service": "lambda.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy --role-name lambda-todo-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess
aws iam attach-role-policy --role-name lambda-todo-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Wait 10 seconds for role propagation
sleep 10
```

```javascript
// 3. Create index.mjs (the Lambda function)
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand, GetCommand, ScanCommand, DeleteCommand } from '@aws-sdk/lib-dynamodb';
import { randomUUID } from 'crypto';

const client = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const TABLE = 'Todos';

export const handler = async (event) => {
  const method = event.httpMethod;
  const path = event.path;
  const body = event.body ? JSON.parse(event.body) : {};
  const id = event.pathParameters?.id;

  try {
    if (method === 'GET' && !id) {
      const result = await client.send(new ScanCommand({ TableName: TABLE }));
      return respond(200, result.Items);
    }

    if (method === 'GET' && id) {
      const result = await client.send(new GetCommand({ TableName: TABLE, Key: { id } }));
      if (!result.Item) return respond(404, { error: 'Not found' });
      return respond(200, result.Item);
    }

    if (method === 'POST') {
      const item = { id: randomUUID(), title: body.title, completed: false, createdAt: new Date().toISOString() };
      await client.send(new PutCommand({ TableName: TABLE, Item: item }));
      return respond(201, item);
    }

    if (method === 'DELETE' && id) {
      await client.send(new DeleteCommand({ TableName: TABLE, Key: { id } }));
      return respond(204, null);
    }

    return respond(405, { error: 'Method not allowed' });
  } catch (error) {
    return respond(500, { error: error.message });
  }
};

function respond(statusCode, body) {
  return { statusCode, headers: { 'Content-Type': 'application/json' }, body: body ? JSON.stringify(body) : '' };
}
```

```bash
# 4. Package and deploy
zip function.zip index.mjs

ROLE_ARN=$(aws iam get-role --role-name lambda-todo-role --query 'Role.Arn' --output text)

aws lambda create-function \
  --function-name todo-api \
  --runtime nodejs20.x \
  --handler index.handler \
  --role $ROLE_ARN \
  --zip-file fileb://function.zip

# 5. Create API Gateway and connect to Lambda
# (Use the AWS console for this — it's faster for API Gateway setup)
# Or use the AWS SAM/CDK for a more automated approach

# 6. Test
curl https://xxxx.execute-api.ap-south-1.amazonaws.com/prod/todos
curl -X POST https://xxxx.execute-api.ap-south-1.amazonaws.com/prod/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn AWS"}'
```

### Cleanup
```bash
aws lambda delete-function --function-name todo-api
aws dynamodb delete-table --table-name Todos
```

---

## 31. Exercise 3: Dockerized App on ECS Fargate

### What You'll Learn
Pushing a Docker image to ECR and running it on Fargate.

```bash
# 1. Create ECR repository
aws ecr create-repository --repository-name my-api

# 2. Build and push Docker image
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com

docker build -t my-api .
docker tag my-api:latest <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/my-api:latest
docker push <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/my-api:latest

# 3. Create ECS cluster
aws ecs create-cluster --cluster-name my-cluster

# 4. Register task definition (create task-def.json first)
aws ecs register-task-definition --cli-input-json file://task-def.json

# 5. Create service
aws ecs create-service \
  --cluster my-cluster \
  --service-name my-api-service \
  --task-definition my-api-task \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}"
```

---

## 32. Exercise 4: File Upload with S3 + Pre-signed URLs

### What You'll Build
An API that generates pre-signed upload URLs so clients upload directly to S3.

```javascript
// server.js
const express = require('express');
const { S3Client, PutObjectCommand, GetObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');

const app = express();
const s3 = new S3Client({ region: 'ap-south-1' });
const BUCKET = 'my-app-uploads-2024';

// Get upload URL
app.post('/api/upload-url', express.json(), async (req, res) => {
  const key = `uploads/${Date.now()}-${req.body.fileName}`;

  const url = await getSignedUrl(s3, new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: req.body.contentType,
  }), { expiresIn: 300 });

  res.json({ uploadUrl: url, fileKey: key });
});

// Get download URL
app.get('/api/download-url/:key', async (req, res) => {
  const url = await getSignedUrl(s3, new GetObjectCommand({
    Bucket: BUCKET,
    Key: req.params.key,
  }), { expiresIn: 3600 });

  res.json({ downloadUrl: url });
});

app.listen(3000);
```

```bash
# Create the S3 bucket
aws s3 mb s3://my-app-uploads-2024 --region ap-south-1

# Test: Get an upload URL, then upload a file using curl
curl -X POST http://localhost:3000/api/upload-url \
  -H "Content-Type: application/json" \
  -d '{"fileName": "test.txt", "contentType": "text/plain"}'

# Use the returned uploadUrl to upload directly to S3:
curl -X PUT "<PRESIGNED_URL>" \
  -H "Content-Type: text/plain" \
  -d "Hello from S3!"
```

---

## 33. Exercise 5: Background Job Processing with SQS

### What You'll Build
An API that queues email jobs, and a separate worker that processes them.

```javascript
// producer.js (your API server)
const { SQSClient, SendMessageCommand } = require('@aws-sdk/client-sqs');
const sqs = new SQSClient({ region: 'ap-south-1' });

app.post('/api/orders', async (req, res) => {
  const order = await saveOrder(req.body);

  // Queue the email job — don't send synchronously
  await sqs.send(new SendMessageCommand({
    QueueUrl: process.env.EMAIL_QUEUE_URL,
    MessageBody: JSON.stringify({
      type: 'order_confirmation',
      orderId: order.id,
      email: req.body.email,
    }),
  }));

  res.status(201).json(order);  // Respond immediately
});

// worker.js (separate process/container)
const { SQSClient, ReceiveMessageCommand, DeleteMessageCommand } = require('@aws-sdk/client-sqs');
const sqs = new SQSClient({ region: 'ap-south-1' });

async function poll() {
  while (true) {
    const response = await sqs.send(new ReceiveMessageCommand({
      QueueUrl: process.env.EMAIL_QUEUE_URL,
      MaxNumberOfMessages: 10,
      WaitTimeSeconds: 20,
    }));

    for (const msg of response.Messages || []) {
      const job = JSON.parse(msg.Body);
      console.log(`Processing: ${job.type} for ${job.email}`);

      await sendEmail(job);  // Actually send the email

      await sqs.send(new DeleteMessageCommand({
        QueueUrl: process.env.EMAIL_QUEUE_URL,
        ReceiptHandle: msg.ReceiptHandle,
      }));
    }
  }
}

poll();
```

```bash
# Create the queue
aws sqs create-queue --queue-name email-jobs

# Create a dead letter queue for failed messages
aws sqs create-queue --queue-name email-jobs-dlq
```

---

## 34. Exercise 6: Full Production Setup

### What You'll Build
VPC + ALB + ECS Fargate + RDS + ElastiCache — a production-ready architecture.

This exercise is best done with **AWS CDK or Terraform** (infrastructure as code). Here's the conceptual flow:

```bash
# 1. Create VPC with public and private subnets
# 2. Create security groups (ALB → public, ECS → private, RDS → private)
# 3. Create RDS PostgreSQL in private subnet
# 4. Create ElastiCache Redis in private subnet
# 5. Create ECR repository, push Docker image
# 6. Create ECS cluster, task definition, service
# 7. Create ALB in public subnet, target group pointing to ECS service
# 8. Create Route 53 record pointing to ALB
# 9. Test: curl https://api.myapp.com/health
```

**Tip:** Use AWS CDK (TypeScript) or Terraform to define this infrastructure as code. Don't click through the console for production setups — you'll forget what you configured.

---

## 35. Exercise 7: Event-Driven Architecture

### What You'll Build
SNS + SQS fan-out — one order event triggers multiple independent processors.

```bash
# 1. Create SNS topic
aws sns create-topic --name order-events

# 2. Create SQS queues for each consumer
aws sqs create-queue --queue-name order-email-queue
aws sqs create-queue --queue-name order-inventory-queue
aws sqs create-queue --queue-name order-analytics-queue

# 3. Subscribe each queue to the SNS topic
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:123456789:order-events \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:ap-south-1:123456789:order-email-queue

aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:123456789:order-events \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:ap-south-1:123456789:order-inventory-queue

# 4. Publish an event
aws sns publish \
  --topic-arn arn:aws:sns:ap-south-1:123456789:order-events \
  --message '{"orderId": "ORD-123", "userId": "user-42", "total": 1500}'

# 5. Check each queue — all three should have the message
aws sqs receive-message --queue-url https://sqs.ap-south-1.amazonaws.com/123456789/order-email-queue
aws sqs receive-message --queue-url https://sqs.ap-south-1.amazonaws.com/123456789/order-inventory-queue
aws sqs receive-message --queue-url https://sqs.ap-south-1.amazonaws.com/123456789/order-analytics-queue
```

---

## 36. Cleanup — Don't Get Surprise Bills

**Run this after every exercise session:**

```bash
# Terminate EC2 instances
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name]' --output table
aws ec2 terminate-instances --instance-ids <INSTANCE_IDs>

# Delete Lambda functions
aws lambda list-functions --query 'Functions[].FunctionName'
aws lambda delete-function --function-name <NAME>

# Delete DynamoDB tables
aws dynamodb list-tables
aws dynamodb delete-table --table-name <NAME>

# Delete SQS queues
aws sqs list-queues
aws sqs delete-queue --queue-url <URL>

# Delete SNS topics
aws sns list-topics
aws sns delete-topic --topic-arn <ARN>

# Delete S3 buckets (must empty first)
aws s3 rm s3://bucket-name --recursive
aws s3 rb s3://bucket-name

# Delete ECR repositories
aws ecr delete-repository --repository-name <NAME> --force

# Delete ECS services and clusters
aws ecs update-service --cluster my-cluster --service my-api-service --desired-count 0
aws ecs delete-service --cluster my-cluster --service my-api-service
aws ecs delete-cluster --cluster my-cluster

# Delete RDS instances (takes a few minutes)
aws rds delete-db-instance --db-instance-identifier <NAME> --skip-final-snapshot

# Delete ElastiCache clusters
aws elasticache delete-cache-cluster --cache-cluster-id <NAME>

# Check for NAT Gateways (these cost ~$32/month!)
aws ec2 describe-nat-gateways --query 'NatGateways[].[NatGatewayId,State]' --output table
aws ec2 delete-nat-gateway --nat-gateway-id <ID>

# Check for Elastic IPs (cost money when not attached)
aws ec2 describe-addresses
aws ec2 release-address --allocation-id <ID>

# Final check: Go to AWS Billing console and verify no unexpected charges
```

**Set a billing alarm (do this FIRST, before any exercise):**
```bash
# Get notified if charges exceed $10
aws cloudwatch put-metric-alarm \
  --alarm-name "BillingAlarm" \
  --namespace AWS/Billing \
  --metric-name EstimatedCharges \
  --statistic Maximum \
  --period 21600 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --dimensions Name=Currency,Value=USD \
  --alarm-actions arn:aws:sns:us-east-1:123456789:billing-alerts \
  --region us-east-1
```

---

# PART 9 — INTERVIEW QUESTIONS

---

## 37. AWS Interview Questions (60+ Questions)

### Fundamentals

**Q1: What is the difference between a Region and an Availability Zone?**
A Region is a geographic location (Mumbai, Singapore, US East) containing multiple AZs. An AZ is an independent data center within a region. AZs are connected via low-latency links but are physically separate, so a natural disaster affecting one AZ doesn't affect others. Deploy across multiple AZs for high availability.

**Q2: What is IAM? What are roles, policies, and users?**
IAM (Identity and Access Management) controls who can do what in your AWS account. A User is a person or application with credentials. A Policy is a JSON document defining permissions (which services, which actions). A Role is a set of permissions that services or users can temporarily assume — no permanent credentials. Best practice: use Roles for services (Lambda, EC2), Users for humans, Policies for granular permissions.

**Q3: What is a VPC and why does it matter?**
A Virtual Private Cloud is your own isolated network in AWS. It controls which resources can communicate with each other and what's accessible from the internet. Without a VPC, your database would be on the public internet. With a VPC, you put databases in private subnets, app servers in private subnets accessible only via load balancers, and load balancers in public subnets.

**Q4: What is the difference between a public subnet and a private subnet?**
A public subnet has a route to the Internet Gateway — resources there can receive traffic from the internet. A private subnet has no direct internet access — resources are only accessible from within the VPC. Databases and app servers go in private subnets. Load balancers and NAT Gateways go in public subnets.

**Q5: What is a security group?**
A virtual firewall for AWS resources. It controls inbound and outbound traffic by rules (protocol, port, source). Example: allow TCP port 3000 from the ALB security group, deny everything else. Security groups are stateful — if you allow inbound traffic, the response is automatically allowed out.

---

### Compute

**Q6: When would you choose Lambda over EC2?**
Lambda for event-driven, short-running tasks (< 15 min) with variable traffic — API endpoints, webhooks, scheduled jobs, file processing. EC2 for long-running processes, WebSocket connections, applications needing full OS control, GPU workloads, or steady high-traffic apps where Lambda's cost-per-invocation becomes expensive.

**Q7: What is a cold start in Lambda? How do you mitigate it?**
A cold start is the initialization delay when Lambda creates a new execution environment (~200ms to 2s depending on runtime and package size). Mitigations: use lighter runtimes (Node.js over Java), keep deployment packages small, use Provisioned Concurrency (pre-warm instances), initialise database connections outside the handler function.

**Q8: What is ECS Fargate? How is it different from running containers on EC2?**
Fargate runs containers without managing the underlying servers. You specify CPU and memory per container, and AWS handles the rest. With EC2 launch type, you manage the EC2 instances (OS patches, scaling, capacity). Choose Fargate for simplicity, EC2 for cost optimization with steady workloads.

**Q9: What is an Auto Scaling Group?**
A group of EC2 instances that automatically scales based on demand. You set min, max, and desired count, plus scaling policies (scale up when CPU > 70%, scale down when CPU < 30%). Used behind an ALB to handle variable traffic.

**Q10: What is the difference between vertical and horizontal scaling in AWS?**
Vertical: change the instance type (t3.small → m5.xlarge). Requires downtime. Has a ceiling. Horizontal: add more instances behind a load balancer. No downtime. No ceiling. AWS Auto Scaling Groups handle horizontal scaling automatically.

---

### Storage & Databases

**Q11: When would you use S3 vs EBS vs EFS?**
S3: object storage for files (images, videos, backups, logs). Accessed via HTTP/SDK. EBS: block storage attached to a single EC2 instance, like a hard drive. For databases and application data. EFS: shared file system mountable by multiple EC2 instances simultaneously. For shared application data.

**Q12: What are S3 pre-signed URLs? When would you use them?**
Temporary URLs that grant time-limited access to upload or download S3 objects without making the bucket public. Use them for file uploads (client uploads directly to S3, bypassing your server) and private file downloads (generate a URL that expires in 1 hour). This offloads bandwidth from your servers.

**Q13: What is RDS Multi-AZ? How does failover work?**
Multi-AZ maintains a synchronous standby replica in a different Availability Zone. If the primary fails, AWS automatically promotes the standby to primary within ~60 seconds. The connection endpoint (DNS name) stays the same — your app reconnects automatically. You get high availability without changing your application code.

**Q14: When would you choose DynamoDB over RDS?**
DynamoDB for simple access patterns (get by key, query by partition + sort key), extreme scale (millions of RPS), predictable single-digit millisecond latency, and serverless architectures (pairs naturally with Lambda). RDS for complex queries, joins, transactions across multiple tables, ad-hoc reporting, and when your team knows SQL.

**Q15: What is ElastiCache and when would you use it?**
Managed Redis or Memcached. Use for caching database queries (reduce DB load), session storage (share sessions across multiple servers), rate limiting (Redis INCR), leaderboards (sorted sets), and real-time data (pub/sub). Choose Redis over Memcached for persistence, data structures, and replication.

---

### Networking & Traffic

**Q16: What is the difference between ALB and NLB?**
ALB (Application Load Balancer) operates at Layer 7 (HTTP). It can route based on URL paths, hostnames, and headers. Supports WebSocket and HTTP/2. Use for web APIs and microservices. NLB (Network Load Balancer) operates at Layer 4 (TCP/UDP). Ultra-low latency, handles millions of connections. Use for TCP-based protocols, gaming, IoT.

**Q17: What is Route 53? What routing policies does it support?**
AWS's DNS service. Routing policies: Simple (one record), Weighted (send 90% to v1, 10% to v2), Latency-based (route to closest region), Failover (primary/standby), Geolocation (different servers for different countries). Used for domain management and traffic routing.

**Q18: What is CloudFront and when would you use it?**
AWS's CDN. Use for static file delivery (images, JS, CSS from S3), API acceleration (cache responses at edge locations), SSL termination, and DDoS protection. It has 400+ edge locations globally. Reduces latency and offloads traffic from your origin servers.

**Q19: What is API Gateway? How does it work with Lambda?**
API Gateway is a managed API service. You define routes (GET /users, POST /orders), and API Gateway handles authentication, throttling, CORS, and request/response transformation. With Lambda integration, each route triggers a Lambda function. Together they create a serverless API with zero server management.

---

### Messaging

**Q20: What is the difference between SQS and SNS?**
SQS is a queue (point-to-point) — one producer, one consumer per message. Messages are pulled by consumers and deleted after processing. SNS is pub/sub (one-to-many) — one message published to a topic is delivered to all subscribers. They're often used together: SNS fans out to multiple SQS queues.

**Q21: What is the SNS + SQS fan-out pattern?**
A producer publishes to an SNS topic. Multiple SQS queues subscribe to the topic. Each queue receives a copy of every message. Each queue has its own consumer processing independently. Example: "order placed" event → email queue, inventory queue, analytics queue. Each processes at its own pace without affecting the others.

**Q22: What is a dead letter queue?**
A queue where messages go after failing processing multiple times (exceeding the retry limit). Instead of losing the message, it's preserved for debugging. You inspect the DLQ, fix the issue, and replay the messages. Essential for reliability — you never lose data.

**Q23: What is EventBridge? How is it different from SNS?**
EventBridge is a serverless event bus with content-based filtering. SNS routes messages to all subscribers. EventBridge can filter events by content and route different events to different targets. Example: route "OrderPlaced" events with total > 1000 to a fraud-detection Lambda, and all events to an analytics queue.

---

### Monitoring & Security

**Q24: What is CloudWatch? What do you use it for?**
CloudWatch collects logs, metrics, and enables alarms. Logs from Lambda, ECS, and API Gateway are automatically sent there. You create custom metrics (API response time, business events) and alarms (alert if error rate > 5%). For backend devs, CloudWatch is your primary observability tool in AWS.

**Q25: What is the difference between Secrets Manager and Parameter Store?**
Both store configuration values. Secrets Manager is for secrets (database passwords, API keys) with automatic rotation and encryption. Parameter Store is for general configuration (feature flags, URLs) and can store secrets too. Secrets Manager costs per secret per month. Parameter Store's standard tier is free. Use Secrets Manager for actual secrets; Parameter Store for everything else.

---

### Architecture

**Q26: How would you deploy a production backend on AWS?**
VPC with public and private subnets across 2 AZs. ALB in public subnets for HTTPS termination and routing. ECS Fargate (or EC2 with Auto Scaling) in private subnets running the application. RDS PostgreSQL (Multi-AZ) in private subnets. ElastiCache Redis in private subnets. S3 for file storage. CloudFront for static assets. Secrets Manager for credentials. CloudWatch for monitoring. Route 53 for DNS.

**Q27: How would you build a serverless backend on AWS?**
API Gateway for HTTP routing → Lambda functions for business logic → DynamoDB for data storage → S3 for file storage → SQS for background jobs → SNS for event notifications → CloudWatch for monitoring → Secrets Manager for credentials. Zero servers to manage, pay-per-request pricing, automatic scaling.

**Q28: How do you handle secrets in AWS?**
Never hardcode secrets in code or environment variables in plain text. Use Secrets Manager or Parameter Store. Access secrets at runtime via the SDK. For ECS, use the `secrets` property in the task definition to inject Secrets Manager values as environment variables. For Lambda, read from Secrets Manager in the handler. IAM roles control which services can access which secrets.

**Q29: How do you handle file uploads at scale?**
Don't route files through your API server. Generate pre-signed S3 upload URLs. The client uploads directly to S3. S3 triggers a Lambda function (via S3 event notification) to process the file (validate, resize, extract metadata). Store the file metadata in your database. Serve files via CloudFront for fast global access.

**Q30: How do you set up a CI/CD pipeline on AWS?**
CodePipeline orchestrates the flow: source (GitHub/CodeCommit) → build (CodeBuild — run tests, build Docker image, push to ECR) → deploy (CodeDeploy or ECS rolling update). Alternatively, use GitHub Actions with AWS credentials to build, push to ECR, and update ECS service. The pipeline triggers on every git push to main.

---

### Scenario-Based

**Q31: Your API is getting slow under load. How do you diagnose and fix it on AWS?**
Check CloudWatch metrics: CPU, memory, request count, latency. If CPU is high → scale horizontally (add instances via Auto Scaling). If database is the bottleneck → add ElastiCache for caching, add read replicas. If specific endpoints are slow → check CloudWatch Logs for slow queries. If network is the bottleneck → add CloudFront for caching. Use X-Ray for distributed tracing to find which service is slow.

**Q32: Your database is running out of connections. What do you do?**
Immediate: increase max_connections in RDS parameter group. Better: use RDS Proxy — it pools and shares database connections across your Lambda functions or ECS tasks. Without a proxy, 100 Lambda invocations create 100 connections. With RDS Proxy, they share a pool of 20 connections. Also review your application's connection pooling settings.

**Q33: You need to process 10,000 images uploaded daily. How?**
S3 event notification triggers a Lambda function on each upload. Lambda resizes the image, generates thumbnails, extracts metadata, and stores results back in S3 and DynamoDB. If processing takes > 15 minutes, use SQS + ECS Fargate workers instead of Lambda. Use S3 lifecycle policies to move original images to Glacier after 30 days.

**Q34: Your Lambda function is timing out when connecting to RDS. Why?**
Lambda runs in a VPC but needs a NAT Gateway to reach the internet, and RDS must be in the same VPC. Common causes: Lambda not in the same VPC as RDS, security group not allowing traffic from Lambda to RDS on port 5432, Lambda cold start initialising connection too slowly. Fix: use RDS Proxy, initialise connection outside the handler, ensure security group rules allow Lambda → RDS communication.

**Q35: How do you handle a traffic spike (10x normal load) on AWS?**
If using Auto Scaling: it automatically adds instances (set max high enough). If using Fargate: auto-scaling adds tasks. If using Lambda: automatically scales. Add CloudFront to cache responses and reduce origin load. Use SQS to buffer and smooth out spikes for background processing. Ensure RDS can handle the load — scale up instance type or add read replicas. Pre-warm the ALB if you know the spike is coming.

---

### Quick-Fire

**Q36: What is the shared responsibility model?** AWS manages security OF the cloud (physical infrastructure, hypervisor, network). You manage security IN the cloud (your data, IAM, security groups, encryption, application code).

**Q37: What is an Elastic IP?** A static public IP address that you can assign to EC2 instances. Unlike normal public IPs, it persists across stops and starts. Free when attached to a running instance, charged when unused.

**Q38: What is a NAT Gateway?** Allows resources in private subnets to access the internet (for downloading packages, calling APIs) while preventing the internet from initiating connections to them. Costs ~$32/month — delete when not needed.

**Q39: What is S3 versioning?** Keeping every version of every object. If you overwrite a file, the old version is preserved. Useful for backup and recovery. Increases storage costs.

**Q40: What is the free tier?** AWS offers 12 months of free usage for many services: 750 hours/month of t3.micro EC2, 5GB S3, 25GB DynamoDB, 1 million Lambda invocations/month, and more. Check aws.amazon.com/free for current limits.

**Q41: What is CloudFormation?** Infrastructure as Code for AWS. Define your entire infrastructure in YAML/JSON templates. AWS creates, updates, and deletes resources based on the template. Reproducible, version-controlled infrastructure.

**Q42: What is AWS CDK?** Cloud Development Kit — write infrastructure as code in TypeScript, Python, or Java instead of YAML. More developer-friendly than CloudFormation. Compiles to CloudFormation templates.

**Q43: What is X-Ray?** Distributed tracing service for AWS. Traces requests across Lambda, API Gateway, ECS, and other services. Shows where time is spent, where errors occur. Essential for debugging microservices.

**Q44: What is WAF?** Web Application Firewall. Protects your API from SQL injection, XSS, DDoS, and bot traffic. Attaches to ALB, API Gateway, or CloudFront. Define rules to block or allow requests based on IP, headers, body content.

**Q45: What is the difference between Spot, On-Demand, and Reserved instances?** On-Demand: full price, pay by the hour, no commitment. Reserved: 1 or 3 year commitment, up to 72% discount. Spot: use spare AWS capacity at up to 90% discount, but can be interrupted with 2 minutes notice. Use On-Demand for development, Reserved for production steady loads, Spot for batch processing.

---

### Cost & Optimization

**Q46: How do you reduce AWS costs?**
Right-size instances (don't over-provision). Use Reserved Instances or Savings Plans for steady workloads. Use Spot Instances for batch processing. Use S3 lifecycle policies to archive old data. Delete unused resources (EBS volumes, Elastic IPs, old snapshots). Use Lambda for infrequent workloads. Monitor with Cost Explorer.

**Q47: What is the most expensive mistake people make on AWS?**
Leaving resources running that they've forgotten about. A NAT Gateway, an idle RDS instance, unused EBS volumes, and a forgotten ECS cluster can cost hundreds of dollars per month. Always tag resources, set billing alerts, and clean up after experiments.

**Q48: How does Lambda pricing work?**
You pay for number of requests ($0.20 per 1 million) plus duration (compute time × memory allocated). A function using 128MB for 100ms costs about $0.000000208. At 1 million invocations/month with 200ms average: ~$0.63. At 100 million invocations: ~$63. Much cheaper than running EC2 24/7 for variable traffic.

---

### Advanced

**Q49: What is AWS Step Functions?** A serverless orchestration service that coordinates multiple Lambda functions (or other AWS services) into workflows. Handles retries, error handling, parallel execution, and conditional logic. Use for complex multi-step processes like order processing or ETL pipelines.

**Q50: What is Amazon SES?** Simple Email Service — send transactional and marketing emails at scale. Cheaper than SendGrid or Mailgun for high volumes. Integrates with Lambda and SQS for async email processing.

**Q51: What is Amazon Cognito?** Managed user authentication service. Handles signup, login, password reset, MFA, and social login (Google, Facebook). Generates JWTs. Integrates with API Gateway for automatic authentication. Saves you from building auth yourself.

**Q52: What is AWS SAM?** Serverless Application Model — a framework for building serverless applications. Define Lambda functions, API Gateway, DynamoDB tables in a simplified YAML template. Includes local testing (`sam local invoke`). Extends CloudFormation with serverless-specific syntax.

**Q53: What is ElasticSearch (OpenSearch) on AWS?** Managed search and analytics engine. Use for full-text search, log analytics, and real-time dashboards. Pair with your application for search features or with CloudWatch for log aggregation and analysis.

**Q54: What is Kinesis?** Real-time data streaming service (like Kafka but managed). Use for processing real-time event streams — clickstream analytics, IoT sensor data, real-time dashboards. Kinesis Data Streams for custom processing, Kinesis Firehose for automatic delivery to S3/Redshift/Elasticsearch.

**Q55: What is the difference between SQS and Kinesis?** SQS is a message queue — messages are consumed and deleted, no ordering guarantee (standard), max 14 days retention. Kinesis is a data stream — messages are retained for up to 365 days, ordered within a shard, multiple consumers can read the same data. SQS for task queues, Kinesis for event streaming and replay.

---

> **Final Advice for AWS Interviews:**
>
> Companies don't expect backend developers to be AWS Solutions Architects. They expect you to know how to deploy, scale, and maintain a production backend on AWS. Focus on: choosing the right compute (Lambda vs ECS vs EC2), connecting services securely (VPC, security groups, IAM roles), handling data (RDS vs DynamoDB, S3, ElastiCache), building async architectures (SQS, SNS), monitoring (CloudWatch), and managing costs. If you can explain why you'd put your database in a private subnet, use pre-signed URLs for file uploads, and SQS for background jobs — you'll do great.
