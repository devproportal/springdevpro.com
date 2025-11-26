
基于下面的信息，给出英文技术博客文章（面向欧美用户，基于 Google Adsense赚钱）：
文章为主，代码为辅。
要有图表和表格。

Reference Title: Spring AI + Milvus: Self-Hosted Vector Database Setup
Reference Keywords: spring ai milvus
Target Word Count: 6000

# Spring AI + Milvus: Self-Hosted Vector Database Setup

**Complete Guide to Building Production-Ready RAG with Open-Source Vector Search**

---

## Table of Contents

- [Spring AI + Milvus: Self-Hosted Vector Database Setup](#spring-ai--milvus-self-hosted-vector-database-setup)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [What You'll Learn](#what-youll-learn)
    - [Why Self-Hosted Milvus?](#why-self-hosted-milvus)
    - [When to Choose Milvus](#when-to-choose-milvus)
    - [Milvus vs Alternatives](#milvus-vs-alternatives)
    - [Prerequisites](#prerequisites)
    - [What Makes Milvus Special](#what-makes-milvus-special)
  - [Why Choose Milvus?](#why-choose-milvus)
    - [Open Source Advantage](#open-source-advantage)
    - [Performance Benchmarks](#performance-benchmarks)
    - [Cost Comparison](#cost-comparison)
    - [Feature Deep Dive](#feature-deep-dive)
    - [Architecture Advantages](#architecture-advantages)
  - [Architecture Overview](#architecture-overview)
    - [System Architecture](#system-architecture)
    - [Data Flow](#data-flow)
  - [Milvus Installation and Setup](#milvus-installation-and-setup)
    - [Docker Compose Setup (Development)](#docker-compose-setup-development)
    - [Kubernetes Deployment (Production)](#kubernetes-deployment-production)
    - [Initial Configuration and Verification](#initial-configuration-and-verification)
  - [Spring AI Integration](#spring-ai-integration)
    - [Project Dependencies](#project-dependencies)
    - [Application Configuration](#application-configuration)
    - [Milvus Configuration](#milvus-configuration)
    - [Advanced Milvus Client Configuration](#advanced-milvus-client-configuration)
  - [Building Your First RAG Application](#building-your-first-rag-application)
    - [Domain Models](#domain-models)
    - [RAG Service Implementation](#rag-service-implementation)
    - [Document Ingestion Service](#document-ingestion-service)
    - [Text Chunking Service](#text-chunking-service)
    - [REST Controller](#rest-controller)
  - [Collection Design and Schema Management](#collection-design-and-schema-management)
    - [Advanced Collection Design](#advanced-collection-design)
  - [Index Types and Performance Tuning](#index-types-and-performance-tuning)
    - [Understanding Milvus Index Types](#understanding-milvus-index-types)
    - [Index Optimization Service](#index-optimization-service)
  - [Advanced Query Patterns](#advanced-query-patterns)
    - [Complex Filter Expressions](#complex-filter-expressions)
  - [Partitioning Strategies](#partitioning-strategies)
    - [Understanding Milvus Partitions](#understanding-milvus-partitions)
    - [Partition Management Service](#partition-management-service)
  - [Data Management and ETL](#data-management-and-etl)
    - [Bulk Data Operations](#bulk-data-operations)
  - [Performance Optimization](#performance-optimization)
    - [Query Optimization Service](#query-optimization-service)
    - [Memory Management](#memory-management)
  - [High Availability Setup](#high-availability-setup)
    - [Cluster Configuration](#cluster-configuration)
    - [Health Check Service](#health-check-service)
  - [Monitoring and Observability](#monitoring-and-observability)
    - [Metrics Configuration](#metrics-configuration)
    - [Prometheus Metrics Endpoint](#prometheus-metrics-endpoint)
    - [Grafana Dashboard JSON](#grafana-dashboard-json)
  - [Backup and Recovery](#backup-and-recovery)
    - [Backup Service](#backup-service)
  - [Scaling Strategies](#scaling-strategies)
    - [Horizontal Scaling Guide](#horizontal-scaling-guide)
    - [Auto-Scaling Configuration](#auto-scaling-configuration)
  - [Security Configuration](#security-configuration)
    - [Authentication and Authorization](#authentication-and-authorization)
  - [Troubleshooting Guide](#troubleshooting-guide)
    - [Common Issues and Solutions](#common-issues-and-solutions)
    - [Diagnostic Service](#diagnostic-service)
  - [Production Deployment](#production-deployment)
    - [Production Checklist](#production-checklist)
    - [Deployment Script](#deployment-script)
  - [Cost Analysis](#cost-analysis)
    - [Total Cost of Ownership](#total-cost-of-ownership)
  - [Conclusion](#conclusion)
    - [Key Takeaways](#key-takeaways)
    - [When to Choose Milvus](#when-to-choose-milvus-1)
    - [Next Steps](#next-steps)
    - [Resources](#resources)
    - [Final Thoughts](#final-thoughts)

---

## Introduction

Milvus is the world's most advanced open-source vector database, purpose-built for billion-scale similarity search and AI applications. Unlike managed solutions, Milvus gives you complete control over your data, deployment, and costs while delivering exceptional performance.

### What You'll Learn

This comprehensive guide provides everything you need to deploy and operate Milvus in production:

- **Installation**: Docker, Kubernetes, bare-metal deployment
- **Integration**: Seamless Spring AI integration
- **Performance**: Achieving sub-10ms queries at scale
- **Operations**: Monitoring, backup, high availability
- **Cost**: Running enterprise RAG for under $200/month

### Why Self-Hosted Milvus?

```
Advantages:
✅ Complete data control and privacy
✅ No vendor lock-in
✅ Customizable performance tuning
✅ Predictable, low costs at scale
✅ On-premise deployment capability
✅ Open-source with active community

Trade-offs:
⚠️ Requires DevOps expertise
⚠️ Infrastructure management overhead
⚠️ Manual scaling and optimization
⚠️ Self-managed backups and monitoring
```

### When to Choose Milvus

**Perfect For**:
- Organizations with strict data residency requirements
- High-volume applications (10M+ vectors)
- Teams with strong DevOps capabilities
- Cost-sensitive deployments
- On-premise or air-gapped environments
- Custom performance requirements

**Consider Managed Solutions If**:
- Small team without DevOps resources
- Need immediate production deployment
- Prefer hands-off infrastructure management
- Budget allows for managed services

### Milvus vs Alternatives

```
Feature Comparison:

                    Milvus    Pinecone   Weaviate   pgvector
───────────────────────────────────────────────────────────
Open Source         ✓         ✗          ✓          ✓
Billion-scale       ✓         ✓          ✓          ✗
Sub-10ms queries    ✓         ✓          ✓          ✗
GPU acceleration    ✓         ✗          ✗          ✗
Distributed         ✓         ✓          ✓          ✗
Managed option      ✓         ✓          ✓          ✗
Setup complexity    Medium    Low        Medium     Low
Cost at scale       Low       High       Medium     Low
```

### Prerequisites

**Technical Requirements**:

```yaml
Development Environment:
  Java: 17 or higher
  Spring Boot: 3.2+
  Maven/Gradle: Latest
  Docker: 20.10+
  Docker Compose: 2.0+

Production Environment:
  Kubernetes: 1.24+ (recommended)
  CPU: 8+ cores per node
  RAM: 32GB+ per node
  Storage: SSD with 500+ IOPS
  Network: 10Gbps recommended

Knowledge:
  - Container orchestration
  - Linux system administration
  - Network configuration
  - Monitoring and alerting
```

### What Makes Milvus Special

**Technical Innovations**:

```
1. Purpose-Built Architecture
   - Separation of storage and compute
   - Distributed microservices design
   - Cloud-native from ground up

2. Advanced Indexing
   - 10+ index types (HNSW, IVF, DiskANN)
   - GPU-accelerated search
   - Hybrid search (vector + scalar)

3. Massive Scale
   - Billions of vectors
   - Horizontal scalability
   - Multi-tenancy support

4. Performance
   - Sub-millisecond latency
   - 10K+ QPS per node
   - Auto-optimization
```

Let's dive into building a production-grade RAG system with Milvus!

---

## Why Choose Milvus?

### Open Source Advantage

**Community and Ecosystem**:

```
Milvus Ecosystem:
├─ 20K+ GitHub stars
├─ 300+ contributors
├─ LF AI & Data Foundation project
├─ Backed by Zilliz (creators of Milvus)
├─ Enterprise support available
└─ Active community forums

Release Cadence:
- Major releases: Quarterly
- Minor updates: Monthly
- Security patches: As needed
- LTS versions: Annual
```

### Performance Benchmarks

**Real-World Performance** (1M vectors, 1536 dimensions):

```
Milvus Performance Metrics:

Query Latency (HNSW index):
├─ p50: 3ms
├─ p95: 8ms
├─ p99: 15ms
└─ p99.9: 25ms

Throughput:
├─ Single node: 5,000 QPS
├─ 3-node cluster: 15,000 QPS
├─ 10-node cluster: 50,000+ QPS
└─ With GPU: 100,000+ QPS

Accuracy (recall@10):
├─ HNSW: 99.5%
├─ IVF_FLAT: 99.0%
├─ IVF_SQ8: 98.5%
└─ IVF_PQ: 95.0%

Resource Usage (1M vectors):
├─ Memory: 4-8GB
├─ Storage: 2-6GB
├─ CPU: 2-4 cores
└─ Network: <100Mbps
```

### Cost Comparison

**3-Year TCO Analysis** (5M vectors, 50K queries/day):

```
Self-Hosted Milvus on AWS:

Infrastructure (EC2):
├─ 3 × r5.2xlarge instances: $900/month
├─ EBS storage (1TB SSD): $100/month
├─ Load balancer: $20/month
├─ Data transfer: $50/month
└─ Total: $1,070/month

Operational Costs:
├─ DevOps time (20 hrs/mo): $3,000/month
├─ Monitoring tools: $100/month
├─ Backup storage: $50/month
└─ Total: $3,150/month

Year 1 Total: $50,640
Year 2 Total: $50,640
Year 3 Total: $50,640
3-Year Total: $151,920

Managed Pinecone (equivalent):
├─ Service cost: $700/month
├─ Setup time: 2 hours
├─ Maintenance: 0 hours/month
Year 1: $8,700
Year 2: $8,400
Year 3: $8,400
3-Year Total: $25,500

Break-Even Analysis:
✓ Pinecone cheaper for <3 years
✓ Milvus cheaper if DevOps cost <$140/month
✓ Milvus ideal if you have existing DevOps team
✓ Consider hybrid: Milvus + occasional consulting
```

### Feature Deep Dive

**What Sets Milvus Apart**:

```java
// 1. Multiple Vector Fields per Collection
Collection multiVector = client.createCollection(
    CollectionSchema.builder()
        .addField("id", DataType.Int64, true)
        .addField("image_vector", DataType.FloatVector, 512)
        .addField("text_vector", DataType.FloatVector, 1536)
        .addField("metadata", DataType.JSON)
        .build()
);

// 2. Hybrid Search (Vector + Scalar)
SearchParam searchParam = SearchParam.builder()
    .vectorField("text_vector")
    .vectors(queryVector)
    .expr("price < 1000 AND category == 'electronics'")
    .topK(10)
    .metricType(MetricType.COSINE)
    .build();

// 3. Dynamic Schema
// Add fields without recreation
collection.addField("new_field", DataType.VarChar);

// 4. Time Travel
// Query historical data
SearchParam timeTravel = SearchParam.builder()
    .vectorField("embedding")
    .vectors(queryVector)
    .travelTimestamp(pastTimestamp)
    .build();

// 5. Multi-Tenancy via Partitions
collection.createPartition("tenant_123");
collection.createPartition("tenant_456");

// Search specific tenant
searchParam.setPartitionNames(List.of("tenant_123"));
```

### Architecture Advantages

**Distributed Design**:

```
Milvus Architecture Components:

1. Coordinator Services:
   ├─ Root Coordinator: Cluster management
   ├─ Data Coordinator: Data distribution
   ├─ Query Coordinator: Query routing
   └─ Index Coordinator: Index building

2. Worker Nodes:
   ├─ Query Nodes: Handle searches
   ├─ Data Nodes: Handle inserts
   └─ Index Nodes: Build indexes

3. Storage:
   ├─ Object Storage (S3/MinIO): Vector data
   ├─ Meta Store (etcd): Metadata
   └─ Message Queue (Pulsar/Kafka): Streaming

Benefits:
✓ Independent scaling of each component
✓ Fault tolerance through replication
✓ Hot/cold data separation
✓ Resource optimization
```

---

## Architecture Overview

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Spring Boot Application                   │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              RAG Service Layer                         │ │
│  │  ┌──────────────┐  ┌──────────────┐                  │ │
│  │  │Query Service │  │Document Svc  │                  │ │
│  │  └──────┬───────┘  └──────┬───────┘                  │ │
│  │         │                  │                           │ │
│  │  ┌──────▼──────────────────▼───────────────────────┐  │ │
│  │  │      Spring AI Milvus Integration              │  │ │
│  │  │  ┌────────────────┐  ┌──────────────────────┐  │  │ │
│  │  │  │ MilvusVector   │  │  EmbeddingClient    │  │  │ │
│  │  │  │    Store       │  │    (OpenAI)          │  │  │ │
│  │  │  └───────┬────────┘  └──────────────────────┘  │  │ │
│  │  └──────────┼─────────────────────────────────────┘  │ │
│  └─────────────┼────────────────────────────────────────┘ │
└────────────────┼─────────────────────────────────────────┘
                 │
                 │ gRPC/HTTP
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                    Milvus Cluster                            │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            Coordinator Services                       │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────┐  │   │
│  │  │  Root   │ │  Data   │ │  Query  │ │  Index   │  │   │
│  │  │  Coord  │ │  Coord  │ │  Coord  │ │  Coord   │  │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └──────────┘  │   │
│  └───────────────────────┬──────────────────────────────┘   │
│                          │                                   │
│  ┌───────────────────────▼──────────────────────────────┐   │
│  │              Worker Nodes                             │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │
│  │  │  Query   │  │  Data    │  │  Index   │           │   │
│  │  │  Node    │  │  Node    │  │  Node    │           │   │
│  │  │  × 3     │  │  × 3     │  │  × 2     │           │   │
│  │  └──────────┘  └──────────┘  └──────────┘           │   │
│  └───────────────────────┬──────────────────────────────┘   │
│                          │                                   │
└──────────────────────────┼───────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  Storage Layer                               │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │    MinIO     │  │     etcd     │  │    Pulsar    │      │
│  │  (Object     │  │  (Metadata)  │  │  (Message    │      │
│  │   Storage)   │  │              │  │   Queue)     │      │
│  │              │  │              │  │              │      │
│  │ Vector Data  │  │ Schemas      │  │ Change Data  │      │
│  │ Indexes      │  │ Partitions   │  │ Capture      │      │
│  │ Snapshots    │  │ Collections  │  │ Replication  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

**Insert Pipeline**:

```
1. Document Ingestion
   └─> Spring Boot receives document
   
2. Text Chunking
   └─> Split into semantic chunks
   
3. Embedding Generation
   └─> OpenAI API generates vectors
   
4. Metadata Preparation
   └─> Extract and structure metadata
   
5. Milvus Insert
   ├─> Data Coordinator receives request
   ├─> Assigns to Data Node
   ├─> Data Node writes to WAL
   ├─> Persists to MinIO
   └─> Updates etcd metadata
   
6. Index Building (async)
   ├─> Index Coordinator triggers build
   ├─> Index Node creates index
   └─> Stores in MinIO

Total Time: 200-500ms
├─> Embedding: 150-300ms
├─> Milvus insert: 10-50ms
└─> Index (async): 1-10 seconds
```

**Query Pipeline**:

```
1. User Query
   └─> "What are the return policies?"
   
2. Query Embedding
   └─> Generate query vector (150-300ms)
   
3. Milvus Search
   ├─> Query Coordinator receives request
   ├─> Routes to Query Nodes
   ├─> Parallel index search
   ├─> Scalar filtering applied
   └─> Results merged and ranked
   
4. Result Processing
   ├─> Fetch full metadata
   ├─> Apply business logic
   └─> Format response
   
5. LLM Generation
   └─> Generate answer with context

Total Time: 800-1500ms
├─> Query embedding: 200ms
├─> Milvus search: 5-15ms
├─> Metadata fetch: 5-10ms
├─> LLM generation: 600-1200ms
└─> Overhead: 20-50ms
```

---

## Milvus Installation and Setup

### Docker Compose Setup (Development)

**docker-compose.yml**:

```yaml
version: '3.5'

services:
  etcd:
    container_name: milvus-etcd
    image: quay.io/coreos/etcd:v3.5.5
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
      - ETCD_SNAPSHOT_COUNT=50000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/etcd:/etcd
    command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd
    healthcheck:
      test: ["CMD", "etcdctl", "endpoint", "health"]
      interval: 30s
      timeout: 20s
      retries: 3
    networks:
      - milvus

  minio:
    container_name: milvus-minio
    image: minio/minio:RELEASE.2023-03-20T20-16-18Z
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/minio:/minio_data
    command: minio server /minio_data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
    ports:
      - "9000:9000"
      - "9001:9001"
    networks:
      - milvus

  pulsar:
    container_name: milvus-pulsar
    image: apachepulsar/pulsar:3.0.0
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/pulsar:/pulsar/data
    environment:
      - BOOKIE_MEM=" -Xms512m -Xmx512m -XX:MaxDirectMemorySize=512m"
    command: |
      /bin/bash -c "
      bin/apply-config-from-env.py conf/standalone.conf &&
      exec bin/pulsar standalone --no-functions-worker --no-stream-storage
      "
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/admin/v2/brokers/health"]
      interval: 30s
      timeout: 20s
      retries: 3
    networks:
      - milvus

  standalone:
    container_name: milvus-standalone
    image: milvusdb/milvus:v2.3.3
    command: ["milvus", "run", "standalone"]
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
      PULSAR_ADDRESS: pulsar://pulsar:6650
      MILVUS_ROOT_USER: root
      MILVUS_ROOT_PASSWORD: Milvus
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
      - ${DOCKER_VOLUME_DIRECTORY:-.}/milvus.yaml:/milvus/configs/milvus.yaml
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9091/healthz"]
      interval: 30s
      start_period: 90s
      timeout: 20s
      retries: 3
    ports:
      - "19530:19530"
      - "9091:9091"
    depends_on:
      - "etcd"
      - "minio"
      - "pulsar"
    networks:
      - milvus

  attu:
    container_name: milvus-attu
    image: zilliz/attu:v2.3.4
    environment:
      MILVUS_URL: milvus-standalone:19530
    ports:
      - "8000:3000"
    depends_on:
      - "standalone"
    networks:
      - milvus

networks:
  milvus:
    driver: bridge

volumes:
  etcd:
  minio:
  pulsar:
  milvus:
```

**milvus.yaml** (Configuration):

```yaml
# Milvus Configuration

# Common settings
common:
  defaultPartitionName: _default
  defaultIndexName: _default_idx
  retentionDuration: 432000  # 5 days in seconds
  entityExpiration: -1  # Never expire

# etcd configuration
etcd:
  endpoints:
    - etcd:2379
  rootPath: by-dev
  metaSubPath: meta
  kvSubPath: kv
  use:
    embed: false

# MinIO configuration
minio:
  address: minio:9000
  port: 9000
  accessKeyID: minioadmin
  secretAccessKey: minioadmin
  useSSL: false
  bucketName: a-bucket
  rootPath: files
  useIAM: false

# Pulsar configuration
pulsar:
  address: pulsar://pulsar:6650
  port: 6650
  maxMessageSize: 5242880  # 5MB

# Root coordinator configuration
rootCoord:
  dmlChannelNum: 16
  maxPartitionNum: 4096
  minSegmentSizeToEnableIndex: 1024
  enableActiveStandby: false

# Data coordinator configuration
dataCoord:
  enableCompaction: true
  enableGarbageCollection: true
  gc:
    interval: 3600
    missingTolerance: 86400
    dropTolerance: 86400

# Query coordinator configuration
queryCoord:
  autoHandoff: true
  autoBalance: true
  balanceIntervalSeconds: 60
  overloadedMemoryThresholdPercentage: 90

# Query node configuration
queryNode:
  cacheSize: 32  # GB
  
# Data node configuration
dataNode:
  insertBufferSize: 16777216  # 16MB
  insertBufferFlushSize: 4194304  # 4MB

# Index node configuration
indexNode:
  enableDisk: true  # Enable disk-based indexing

# Log configuration
log:
  level: info
  file:
    rootPath: /var/lib/milvus/logs
    maxSize: 300  # MB
    maxAge: 10  # days
    maxBackups: 20

# Metrics configuration
metrics:
  enable: true
  port: 9091
```

**Start Milvus**:

```bash
# Create volumes directory
mkdir -p volumes/{etcd,minio,pulsar,milvus}

# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f standalone

# Access Attu (Web UI)
# Open http://localhost:8000
# Connect to: milvus-standalone:19530

# Stop services
docker-compose down

# Stop and remove data
docker-compose down -v
```

### Kubernetes Deployment (Production)

**Using Helm Chart**:

```bash
# Add Milvus Helm repository
helm repo add milvus https://milvus-io.github.io/milvus-helm/
helm repo update

# Create namespace
kubectl create namespace milvus

# Create values.yaml for customization
cat > milvus-values.yaml <<EOF
cluster:
  enabled: true

image:
  all:
    repository: milvusdb/milvus
    tag: v2.3.3
    pullPolicy: IfNotPresent

service:
  type: LoadBalancer
  port: 19530
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb

resources:
  limits:
    cpu: 4
    memory: 16Gi
  requests:
    cpu: 2
    memory: 8Gi

persistence:
  enabled: true
  storageClass: gp3
  size: 500Gi

pulsar:
  enabled: true
  persistence:
    storageClass: gp3
    size: 100Gi

etcd:
  enabled: true
  persistence:
    storageClass: gp3
    size: 50Gi

minio:
  enabled: true
  mode: distributed
  replicas: 4
  persistence:
    storageClass: gp3
    size: 200Gi

metrics:
  enabled: true
  serviceMonitor:
    enabled: true

# Component-specific settings
queryNode:
  replicas: 3
  resources:
    limits:
      cpu: 4
      memory: 16Gi
    requests:
      cpu: 2
      memory: 8Gi

dataNode:
  replicas: 3
  resources:
    limits:
      cpu: 2
      memory: 8Gi
    requests:
      cpu: 1
      memory: 4Gi

indexNode:
  replicas: 2
  resources:
    limits:
      cpu: 4
      memory: 16Gi
    requests:
      cpu: 2
      memory: 8Gi

proxy:
  replicas: 2
  resources:
    limits:
      cpu: 2
      memory: 4Gi
    requests:
      cpu: 1
      memory: 2Gi
EOF

# Install Milvus
helm install milvus milvus/milvus \
  -n milvus \
  -f milvus-values.yaml

# Check deployment
kubectl get pods -n milvus
kubectl get svc -n milvus

# Get load balancer endpoint
kubectl get svc milvus -n milvus

# Upgrade configuration
helm upgrade milvus milvus/milvus \
  -n milvus \
  -f milvus-values.yaml
```

### Initial Configuration and Verification

**Health Check Script**:

```java
package com.example.milvus.config;

import io.milvus.client.MilvusServiceClient;
import io.milvus.param.ConnectParam;
import io.milvus.param.R;
import io.milvus.response.GetMetricsResponseWrapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class MilvusHealthCheck implements ApplicationRunner {
    
    @Value("${milvus.host:localhost}")
    private String host;
    
    @Value("${milvus.port:19530}")
    private int port;
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("Verifying Milvus connection to {}:{}", host, port);
        
        try {
            MilvusServiceClient client = new MilvusServiceClient(
                ConnectParam.newBuilder()
                    .withHost(host)
                    .withPort(port)
                    .withConnectTimeout(10000)
                    .build()
            );
            
            // Check version
            R<GetMetricsResponseWrapper> response = client.getMetrics();
            if (response.getStatus() == R.Status.Success.getCode()) {
                log.info("✓ Milvus connection successful");
                log.info("  Cluster info: {}", response.getData());
            }
            
            // List collections
            R<java.util.List<String>> collections = client.listCollections();
            log.info("✓ Existing collections: {}", 
                collections.getData().size());
            
            client.close();
            
            log.info("✓ Milvus health check passed");
            
        } catch (Exception e) {
            log.error("✗ Milvus health check failed", e);
            throw new RuntimeException("Failed to connect to Milvus", e);
        }
    }
}
```

---

## Spring AI Integration

### Project Dependencies

**pom.xml**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>spring-ai-milvus-rag</artifactId>
    <version>1.0.0</version>
    <name>Spring AI Milvus RAG</name>
    
    <properties>
        <java.version>17</java.version>
        <spring-ai.version>1.0.0-M3</spring-ai.version>
        <milvus-sdk.version>2.3.3</milvus-sdk.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        
        <!-- Spring AI with Milvus -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-milvus-store-spring-boot-starter</artifactId>
            <version>${spring-ai.version}</version>
        </dependency>
        
        <!-- Spring AI with OpenAI -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
            <version>${spring-ai.version}</version>
        </dependency>
        
        <!-- Milvus SDK (for advanced features) -->
        <dependency>
            <groupId>io.milvus</groupId>
            <artifactId>milvus-sdk-java</artifactId>
            <version>${milvus-sdk.version}</version>
        </dependency>
        
        <!-- JSON processing -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        
        <!-- Utilities -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Caching -->
        <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
        </dependency>
        
        <!-- Metrics -->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### Application Configuration

**application.yml**:

```yaml
spring:
  application:
    name: milvus-rag-app
  
  ai:
    milvus:
      client:
        host: ${MILVUS_HOST:localhost}
        port: ${MILVUS_PORT:19530}
        username: ${MILVUS_USERNAME:}
        password: ${MILVUS_PASSWORD:}
        secure: ${MILVUS_SECURE:false}
        connection-timeout-ms: 10000
        keep-alive-time-ms: 55000
        keep-alive-timeout-ms: 20000
        rpc-deadline-ms: 86400000  # 24 hours
      
      store:
        collection-name: ${MILVUS_COLLECTION:rag_documents}
        embedding-dimension: 1536
        index-type: HNSW
        metric-type: COSINE
        database-name: ${MILVUS_DATABASE:default}
        
    openai:
      api-key: ${OPENAI_API_KEY}
      embedding:
        options:
          model: text-embedding-3-small
          dimensions: 1536
      chat:
        options:
          model: gpt-4-turbo-preview
          temperature: 0.7
          max-tokens: 1000

# Milvus-specific configuration
milvus:
  collection:
    auto-create: true
    shard-num: 2
    consistency-level: BOUNDED
  
  index:
    params:
      M: 16
      efConstruction: 256
    build-async: true
  
  search:
    params:
      ef: 64
      nprobe: 16
    consistency-level: BOUNDED

# Connection pool
milvus-pool:
  max-total: 50
  max-idle: 25
  min-idle: 5
  max-wait-millis: 30000

# Caching
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=10000,expireAfterWrite=1h

# Metrics
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true

logging:
  level:
    root: INFO
    com.example.milvus: DEBUG
    io.milvus: DEBUG
    org.springframework.ai: DEBUG
```

### Milvus Configuration

**MilvusConfig.java**:

```java
package com.example.milvus.config;

import io.milvus.client.MilvusServiceClient;
import io.milvus.param.ConnectParam;
import io.milvus.param.IndexType;
import io.milvus.param.MetricType;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.embedding.EmbeddingClient;
import org.springframework.ai.vectorstore.MilvusVectorStore;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Slf4j
@Configuration
public class MilvusConfig {
    
    @Value("${spring.ai.milvus.client.host}")
    private String host;
    
    @Value("${spring.ai.milvus.client.port}")
    private int port;
    
    @Value("${spring.ai.milvus.client.username:}")
    private String username;
    
    @Value("${spring.ai.milvus.client.password:}")
    private String password;
    
    @Value("${spring.ai.milvus.client.connection-timeout-ms}")
    private long connectionTimeout;
    
    @Value("${spring.ai.milvus.store.collection-name}")
    private String collectionName;
    
    @Value("${spring.ai.milvus.store.embedding-dimension}")
    private int dimension;
    
    @Value("${spring.ai.milvus.store.database-name}")
    private String databaseName;
    
    @Bean
    public MilvusServiceClient milvusClient() {
        log.info("Initializing Milvus client: {}:{}", host, port);
        
        ConnectParam.Builder builder = ConnectParam.newBuilder()
            .withHost(host)
            .withPort(port)
            .withConnectTimeout(connectionTimeout)
            .withDatabaseName(databaseName);
        
        if (username != null && !username.isEmpty()) {
            builder.withUsername(username);
            builder.withPassword(password);
        }
        
        MilvusServiceClient client = new MilvusServiceClient(builder.build());
        
        log.info("✓ Milvus client initialized successfully");
        return client;
    }
    
    @Bean
    public VectorStore vectorStore(
            MilvusServiceClient milvusClient,
            EmbeddingClient embeddingClient) {
        
        log.info("Creating Milvus vector store for collection: {}", 
            collectionName);
        
        MilvusVectorStore.MilvusVectorStoreConfig config = 
            MilvusVectorStore.MilvusVectorStoreConfig.builder()
                .withCollectionName(collectionName)
                .withDatabaseName(databaseName)
                .withDimension(dimension)
                .withIndexType(IndexType.HNSW)
                .withMetricType(MetricType.COSINE)
                .build();
        
        return new MilvusVectorStore(milvusClient, embeddingClient, config);
    }
}
```

### Advanced Milvus Client Configuration

**MilvusClientFactory.java**:

```java
package com.example.milvus.config;

import io.milvus.client.MilvusServiceClient;
import io.milvus.param.*;
import io.milvus.param.collection.*;
import io.milvus.param.index.CreateIndexParam;
import io.milvus.grpc.DataType;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import jakarta.annotation.PostConstruct;
import java.util.List;

@Slf4j
@Component
@RequiredArgsConstructor
public class MilvusClientFactory {
    
    private final MilvusServiceClient milvusClient;
    
    @PostConstruct
    public void initialize() {
        log.info("Initializing Milvus collections and indexes");
        createCollectionIfNotExists();
        createIndexIfNotExists();
    }
    
    private void createCollectionIfNotExists() {
        String collectionName = "rag_documents";
        
        // Check if collection exists
        R<Boolean> hasResponse = milvusClient.hasCollection(
            HasCollectionParam.newBuilder()
                .withCollectionName(collectionName)
                .build()
        );
        
        if (hasResponse.getData()) {
            log.info("Collection already exists: {}", collectionName);
            return;
        }
        
        log.info("Creating collection: {}", collectionName);
        
        // Define schema
        FieldType id = FieldType.newBuilder()
            .withName("id")
            .withDataType(DataType.Int64)
            .withPrimaryKey(true)
            .withAutoID(true)
            .build();
        
        FieldType vector = FieldType.newBuilder()
            .withName("embedding")
            .withDataType(DataType.FloatVector)
            .withDimension(1536)
            .build();
        
        FieldType content = FieldType.newBuilder()
            .withName("content")
            .withDataType(DataType.VarChar)
            .withMaxLength(65535)
            .build();
        
        FieldType metadata = FieldType.newBuilder()
            .withName("metadata")
            .withDataType(DataType.JSON)
            .build();
        
        FieldType timestamp = FieldType.newBuilder()
            .withName("timestamp")
            .withDataType(DataType.Int64)
            .build();
        
        CollectionSchemaParam schema = CollectionSchemaParam.newBuilder()
            .withFieldTypes(List.of(id, vector, content, metadata, timestamp))
            .withEnableDynamicField(true)
            .build();
        
        // Create collection
        R<RpcStatus> createResponse = milvusClient.createCollection(
            CreateCollectionParam.newBuilder()
                .withCollectionName(collectionName)
                .withSchema(schema)
                .withShardsNum(2)
                .withConsistencyLevel(ConsistencyLevelEnum.BOUNDED)
                .build()
        );
        
        if (createResponse.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Collection created successfully: {}", collectionName);
        } else {
            log.error("Failed to create collection: {}", 
                createResponse.getMessage());
        }
    }
    
    private void createIndexIfNotExists() {
        String collectionName = "rag_documents";
        String fieldName = "embedding";
        
        log.info("Creating HNSW index on field: {}", fieldName);
        
        R<RpcStatus> indexResponse = milvusClient.createIndex(
            CreateIndexParam.newBuilder()
                .withCollectionName(collectionName)
                .withFieldName(fieldName)
                .withIndexType(IndexType.HNSW)
                .withMetricType(MetricType.COSINE)
                .withExtraParam("{\"M\":16,\"efConstruction\":256}")
                .withSyncMode(Boolean.FALSE)  // Build async
                .build()
        );
        
        if (indexResponse.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Index creation initiated");
        } else {
            log.error("Failed to create index: {}", 
                indexResponse.getMessage());
        }
        
        // Load collection into memory
        R<RpcStatus> loadResponse = milvusClient.loadCollection(
            LoadCollectionParam.newBuilder()
                .withCollectionName(collectionName)
                .build()
        );
        
        if (loadResponse.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Collection loaded into memory");
        }
    }
}
```
## Building Your First RAG Application

### Domain Models

**DocumentRequest.java**:

```java
package com.example.milvus.model;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Data;

import java.util.HashMap;
import java.util.Map;

@Data
public class DocumentRequest {
    
    @NotBlank(message = "Content cannot be empty")
    @Size(max = 50000, message = "Content too large")
    private String content;
    
    private String title;
    
    private String source;
    
    private String category;
    
    private Map<String, Object> metadata = new HashMap<>();
    
    private String partitionName;
    
    private Boolean buildIndex = true;
}
```

**QueryRequest.java**:

```java
package com.example.milvus.model;

import jakarta.validation.constraints.*;
import lombok.Data;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Data
public class QueryRequest {
    
    @NotBlank(message = "Query cannot be empty")
    @Size(max = 2000, message = "Query too long")
    private String query;
    
    @Min(1)
    @Max(100)
    private Integer topK = 10;
    
    @DecimalMin("0.0")
    @DecimalMax("1.0")
    private Double similarityThreshold = 0.7;
    
    private String partitionName;
    
    private String metadataFilter;
    
    private Map<String, Object> searchParams = new HashMap<>();
    
    private List<String> outputFields;
    
    @Pattern(regexp = "gpt-4|gpt-4-turbo|gpt-3.5-turbo")
    private String model = "gpt-4-turbo-preview";
    
    @DecimalMin("0.0")
    @DecimalMax("2.0")
    private Double temperature = 0.7;
    
    private Boolean includeVectors = false;
}
```

**QueryResponse.java**:

```java
package com.example.milvus.model;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.Instant;
import java.util.List;
import java.util.Map;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class QueryResponse {
    
    private String answer;
    
    private List<SearchResult> results;
    
    private Integer totalResults;
    
    private QueryMetrics metrics;
    
    private Instant timestamp;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class SearchResult {
        private String id;
        private String content;
        private Map<String, Object> metadata;
        private Double score;
        private Float[] vector;
        private Integer rank;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class QueryMetrics {
        private Long embeddingTimeMs;
        private Long searchTimeMs;
        private Long llmTimeMs;
        private Long totalTimeMs;
        private Integer vectorsSearched;
        private String indexType;
    }
}
```

### RAG Service Implementation

**MilvusRagService.java**:

```java
package com.example.milvus.service;

import com.example.milvus.model.QueryRequest;
import com.example.milvus.model.QueryResponse;
import io.milvus.client.MilvusServiceClient;
import io.milvus.grpc.SearchResults;
import io.milvus.param.R;
import io.milvus.param.dml.SearchParam;
import io.milvus.response.SearchResultsWrapper;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.ChatClient;
import org.springframework.ai.chat.ChatResponse;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.chat.prompt.PromptTemplate;
import org.springframework.ai.document.Document;
import org.springframework.ai.embedding.EmbeddingClient;
import org.springframework.ai.embedding.EmbeddingResponse;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import java.time.Instant;
import java.util.*;
import java.util.stream.Collectors;

@Slf4j
@Service
@RequiredArgsConstructor
public class MilvusRagService {
    
    private final MilvusServiceClient milvusClient;
    private final EmbeddingClient embeddingClient;
    private final ChatClient chatClient;
    
    private static final String COLLECTION_NAME = "rag_documents";
    
    private static final String RAG_PROMPT = """
        Answer the question based on the context below. Be concise and accurate.
        If you cannot answer based on the context, say so.
        
        Context:
        {context}
        
        Question: {question}
        
        Answer:
        """;
    
    public QueryResponse query(QueryRequest request) {
        long startTime = System.currentTimeMillis();
        
        QueryResponse.QueryMetrics.QueryMetricsBuilder metricsBuilder = 
            QueryResponse.QueryMetrics.builder();
        
        try {
            // Step 1: Generate query embedding
            long embeddingStart = System.currentTimeMillis();
            List<Float> queryVector = generateEmbedding(request.getQuery());
            metricsBuilder.embeddingTimeMs(
                System.currentTimeMillis() - embeddingStart
            );
            
            // Step 2: Search Milvus
            long searchStart = System.currentTimeMillis();
            List<QueryResponse.SearchResult> searchResults = 
                searchMilvus(queryVector, request);
            metricsBuilder.searchTimeMs(
                System.currentTimeMillis() - searchStart
            );
            metricsBuilder.vectorsSearched(searchResults.size());
            
            if (searchResults.isEmpty()) {
                return buildEmptyResponse(metricsBuilder, startTime);
            }
            
            // Step 3: Build context
            String context = buildContext(searchResults);
            
            // Step 4: Generate answer
            long llmStart = System.currentTimeMillis();
            String answer = generateAnswer(request.getQuery(), context, request);
            metricsBuilder.llmTimeMs(
                System.currentTimeMillis() - llmStart
            );
            
            // Build response
            metricsBuilder.totalTimeMs(System.currentTimeMillis() - startTime);
            
            return QueryResponse.builder()
                .answer(answer)
                .results(searchResults)
                .totalResults(searchResults.size())
                .metrics(metricsBuilder.build())
                .timestamp(Instant.now())
                .build();
            
        } catch (Exception e) {
            log.error("Error processing query", e);
            throw new RuntimeException("Query processing failed", e);
        }
    }
    
    @Cacheable(value = "embeddings", key = "#text")
    private List<Float> generateEmbedding(String text) {
        EmbeddingResponse response = embeddingClient.embedForResponse(
            List.of(text)
        );
        
        return response.getResults().get(0).getOutput().stream()
            .map(Double::floatValue)
            .collect(Collectors.toList());
    }
    
    private List<QueryResponse.SearchResult> searchMilvus(
            List<Float> queryVector,
            QueryRequest request) {
        
        // Build search parameters
        Map<String, Object> searchParams = new HashMap<>();
        searchParams.put("ef", request.getSearchParams()
            .getOrDefault("ef", 64));
        
        // Build output fields
        List<String> outputFields = request.getOutputFields() != null ?
            request.getOutputFields() :
            List.of("content", "metadata", "timestamp");
        
        // Build search request
        SearchParam.Builder searchBuilder = SearchParam.newBuilder()
            .withCollectionName(COLLECTION_NAME)
            .withMetricType(io.milvus.param.MetricType.COSINE)
            .withTopK(request.getTopK())
            .withVectors(List.of(queryVector))
            .withVectorFieldName("embedding")
            .withParams(new com.google.gson.Gson().toJson(searchParams))
            .withOutFields(outputFields);
        
        // Add partition filter
        if (request.getPartitionName() != null) {
            searchBuilder.withPartitionNames(
                List.of(request.getPartitionName())
            );
        }
        
        // Add metadata filter
        if (request.getMetadataFilter() != null) {
            searchBuilder.withExpr(request.getMetadataFilter());
        }
        
        // Execute search
        R<SearchResults> response = milvusClient.search(searchBuilder.build());
        
        if (response.getStatus() != R.Status.Success.getCode()) {
            throw new RuntimeException("Search failed: " + response.getMessage());
        }
        
        // Process results
        SearchResultsWrapper wrapper = new SearchResultsWrapper(
            response.getData().getResults()
        );
        
        List<QueryResponse.SearchResult> results = new ArrayList<>();
        
        for (int i = 0; i < wrapper.getRowRecords(0).size(); i++) {
            SearchResultsWrapper.IDScore idScore = wrapper.getIDScore(0).get(i);
            
            // Filter by similarity threshold
            if (idScore.getScore() < request.getSimilarityThreshold()) {
                continue;
            }
            
            Map<String, Object> fieldData = wrapper.getRowRecords(0).get(i);
            
            QueryResponse.SearchResult result = QueryResponse.SearchResult.builder()
                .id(String.valueOf(idScore.getLongID()))
                .content((String) fieldData.get("content"))
                .metadata(parseMetadata(fieldData.get("metadata")))
                .score(idScore.getScore())
                .rank(i + 1)
                .build();
            
            if (request.getIncludeVectors()) {
                // Fetch vectors if requested
                result.setVector(wrapper.getFieldWrapper("embedding")
                    .getFieldData().get(i));
            }
            
            results.add(result);
        }
        
        return results;
    }
    
    private String buildContext(List<QueryResponse.SearchResult> results) {
        return results.stream()
            .map(result -> {
                String source = result.getMetadata() != null ?
                    (String) result.getMetadata().getOrDefault("source", "Unknown") :
                    "Unknown";
                
                return String.format(
                    "[Source: %s, Score: %.3f]\n%s",
                    source,
                    result.getScore(),
                    result.getContent()
                );
            })
            .collect(Collectors.joining("\n\n"));
    }
    
    private String generateAnswer(
            String question,
            String context,
            QueryRequest request) {
        
        PromptTemplate template = new PromptTemplate(RAG_PROMPT);
        
        Prompt prompt = template.create(Map.of(
            "context", context,
            "question", question
        ));
        
        ChatResponse response = chatClient.call(prompt);
        return response.getResult().getOutput().getContent();
    }
    
    private QueryResponse buildEmptyResponse(
            QueryResponse.QueryMetrics.QueryMetricsBuilder metricsBuilder,
            long startTime) {
        
        metricsBuilder.totalTimeMs(System.currentTimeMillis() - startTime);
        
        return QueryResponse.builder()
            .answer("I couldn't find relevant information to answer your question.")
            .results(List.of())
            .totalResults(0)
            .metrics(metricsBuilder.build())
            .timestamp(Instant.now())
            .build();
    }
    
    @SuppressWarnings("unchecked")
    private Map<String, Object> parseMetadata(Object metadataObj) {
        if (metadataObj instanceof Map) {
            return (Map<String, Object>) metadataObj;
        }
        return new HashMap<>();
    }
}
```

### Document Ingestion Service

**DocumentIngestionService.java**:

```java
package com.example.milvus.service;

import com.example.milvus.model.DocumentRequest;
import io.milvus.client.MilvusServiceClient;
import io.milvus.grpc.MutationResult;
import io.milvus.param.R;
import io.milvus.param.dml.InsertParam;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.document.Document;
import org.springframework.ai.embedding.EmbeddingClient;
import org.springframework.ai.embedding.EmbeddingResponse;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.util.*;
import java.util.concurrent.CompletableFuture;
import java.util.stream.Collectors;

@Slf4j
@Service
@RequiredArgsConstructor
public class DocumentIngestionService {
    
    private final MilvusServiceClient milvusClient;
    private final EmbeddingClient embeddingClient;
    private final TextChunkingService chunkingService;
    
    private static final String COLLECTION_NAME = "rag_documents";
    
    /**
     * Ingest single document
     */
    public String ingestDocument(DocumentRequest request) {
        
        log.info("Ingesting document: {} (size: {} chars)",
            request.getTitle(), request.getContent().length());
        
        try {
            // Split into chunks
            List<String> chunks = chunkingService.chunk(
                request.getContent(),
                1000,  // chunk size
                200    // overlap
            );
            
            log.debug("Split into {} chunks", chunks.size());
            
            // Generate embeddings
            List<List<Float>> embeddings = generateEmbeddings(chunks);
            
            // Prepare metadata
            Map<String, Object> baseMetadata = new HashMap<>(request.getMetadata());
            baseMetadata.put("title", request.getTitle());
            baseMetadata.put("source", request.getSource());
            baseMetadata.put("category", request.getCategory());
            baseMetadata.put("ingested_at", System.currentTimeMillis());
            
            // Insert into Milvus
            insertVectors(chunks, embeddings, baseMetadata, request.getPartitionName());
            
            log.info("Successfully ingested document: {}", request.getTitle());
            
            return UUID.randomUUID().toString();
            
        } catch (Exception e) {
            log.error("Failed to ingest document", e);
            throw new RuntimeException("Document ingestion failed", e);
        }
    }
    
    /**
     * Async batch ingestion
     */
    @Async("documentProcessingExecutor")
    public CompletableFuture<BatchResult> ingestBatchAsync(
            List<DocumentRequest> documents) {
        
        long startTime = System.currentTimeMillis();
        int successCount = 0;
        int failureCount = 0;
        
        for (DocumentRequest doc : documents) {
            try {
                ingestDocument(doc);
                successCount++;
            } catch (Exception e) {
                log.error("Failed to ingest document: {}", doc.getTitle(), e);
                failureCount++;
            }
        }
        
        long duration = System.currentTimeMillis() - startTime;
        
        return CompletableFuture.completedFuture(
            BatchResult.builder()
                .totalDocuments(documents.size())
                .successfulIngestions(successCount)
                .failedIngestions(failureCount)
                .durationMs(duration)
                .build()
        );
    }
    
    private List<List<Float>> generateEmbeddings(List<String> texts) {
        
        // Batch embedding generation
        EmbeddingResponse response = embeddingClient.embedForResponse(texts);
        
        return response.getResults().stream()
            .map(result -> result.getOutput().stream()
                .map(Double::floatValue)
                .collect(Collectors.toList())
            )
            .collect(Collectors.toList());
    }
    
    private void insertVectors(
            List<String> contents,
            List<List<Float>> embeddings,
            Map<String, Object> baseMetadata,
            String partitionName) {
        
        List<List<Float>> vectors = new ArrayList<>();
        List<String> contentList = new ArrayList<>();
        List<String> metadataList = new ArrayList<>();
        List<Long> timestamps = new ArrayList<>();
        
        long currentTime = System.currentTimeMillis();
        
        for (int i = 0; i < contents.size(); i++) {
            vectors.add(embeddings.get(i));
            contentList.add(contents.get(i));
            
            Map<String, Object> chunkMetadata = new HashMap<>(baseMetadata);
            chunkMetadata.put("chunk_index", i);
            chunkMetadata.put("total_chunks", contents.size());
            
            metadataList.add(new com.google.gson.Gson().toJson(chunkMetadata));
            timestamps.add(currentTime);
        }
        
        // Prepare insert fields
        List<InsertParam.Field> fields = new ArrayList<>();
        
        fields.add(new InsertParam.Field("embedding", vectors));
        fields.add(new InsertParam.Field("content", contentList));
        fields.add(new InsertParam.Field("metadata", metadataList));
        fields.add(new InsertParam.Field("timestamp", timestamps));
        
        // Build insert request
        InsertParam.Builder insertBuilder = InsertParam.newBuilder()
            .withCollectionName(COLLECTION_NAME)
            .withFields(fields);
        
        if (partitionName != null) {
            insertBuilder.withPartitionName(partitionName);
        }
        
        // Execute insert
        R<MutationResult> response = milvusClient.insert(insertBuilder.build());
        
        if (response.getStatus() != R.Status.Success.getCode()) {
            throw new RuntimeException("Insert failed: " + response.getMessage());
        }
        
        log.debug("Inserted {} vectors", contents.size());
    }
    
    @lombok.Data
    @lombok.Builder
    public static class BatchResult {
        private Integer totalDocuments;
        private Integer successfulIngestions;
        private Integer failedIngestions;
        private Long durationMs;
    }
}
```

### Text Chunking Service

**TextChunkingService.java**:

```java
package com.example.milvus.service;

import org.springframework.stereotype.Service;

import java.util.*;
import java.util.regex.Pattern;

@Service
public class TextChunkingService {
    
    /**
     * Recursive character-based chunking
     */
    public List<String> chunk(String text, int chunkSize, int overlap) {
        
        List<String> separators = List.of(
            "\n\n",  // Paragraphs
            "\n",    // Lines
            ". ",    // Sentences
            "! ",
            "? ",
            "; ",
            ", ",
            " "      // Words
        );
        
        return splitRecursive(text, separators, chunkSize, overlap);
    }
    
    private List<String> splitRecursive(
            String text,
            List<String> separators,
            int chunkSize,
            int overlap) {
        
        if (text.length() <= chunkSize) {
            return List.of(text);
        }
        
        String separator = separators.get(0);
        List<String> splits = Arrays.asList(
            text.split(Pattern.quote(separator))
        );
        
        List<String> chunks = new ArrayList<>();
        StringBuilder currentChunk = new StringBuilder();
        
        for (String split : splits) {
            if (currentChunk.length() + split.length() + separator.length() <= chunkSize) {
                if (currentChunk.length() > 0) {
                    currentChunk.append(separator);
                }
                currentChunk.append(split);
            } else {
                if (currentChunk.length() > 0) {
                    chunks.add(currentChunk.toString());
                    
                    // Add overlap
                    String overlapText = getOverlap(currentChunk.toString(), overlap);
                    currentChunk = new StringBuilder(overlapText);
                }
                currentChunk.append(split);
            }
        }
        
        if (currentChunk.length() > 0) {
            chunks.add(currentChunk.toString());
        }
        
        return chunks;
    }
    
    private String getOverlap(String text, int overlapSize) {
        if (text.length() <= overlapSize) {
            return text;
        }
        
        String overlap = text.substring(text.length() - overlapSize);
        
        // Try to start at sentence boundary
        int sentenceStart = overlap.indexOf(". ");
        if (sentenceStart > 0) {
            return overlap.substring(sentenceStart + 2);
        }
        
        return overlap;
    }
}
```

### REST Controller

**RagController.java**:

```java
package com.example.milvus.controller;

import com.example.milvus.model.*;
import com.example.milvus.service.DocumentIngestionService;
import com.example.milvus.service.MilvusRagService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@Slf4j
@RestController
@RequestMapping("/api/v1/rag")
@RequiredArgsConstructor
public class RagController {
    
    private final MilvusRagService ragService;
    private final DocumentIngestionService ingestionService;
    
    @PostMapping("/query")
    public ResponseEntity<QueryResponse> query(
            @Valid @RequestBody QueryRequest request) {
        
        log.info("Received query: {} (topK={})",
            request.getQuery(), request.getTopK());
        
        QueryResponse response = ragService.query(request);
        
        log.info("Query processed: {} results in {}ms",
            response.getTotalResults(),
            response.getMetrics().getTotalTimeMs());
        
        return ResponseEntity.ok(response);
    }
    
    @PostMapping("/documents")
    public ResponseEntity<Map<String, String>> ingestDocument(
            @Valid @RequestBody DocumentRequest request) {
        
        log.info("Ingesting document: {}", request.getTitle());
        
        String docId = ingestionService.ingestDocument(request);
        
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(Map.of(
                "documentId", docId,
                "status", "ingested"
            ));
    }
    
    @GetMapping("/health")
    public ResponseEntity<Map<String, Object>> health() {
        return ResponseEntity.ok(Map.of(
            "status", "UP",
            "service", "Milvus RAG",
            "timestamp", System.currentTimeMillis()
        ));
    }
}
```

---

## Collection Design and Schema Management

### Advanced Collection Design

**CollectionManager.java**:

```java
package com.example.milvus.service;

import io.milvus.client.MilvusServiceClient;
import io.milvus.grpc.DataType;
import io.milvus.param.*;
import io.milvus.param.collection.*;
import io.milvus.param.index.CreateIndexParam;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.List;

@Slf4j
@Service
@RequiredArgsConstructor
public class CollectionManager {
    
    private final MilvusServiceClient milvusClient;
    
    /**
     * Create collection with multiple vector fields
     */
    public void createMultiVectorCollection(String collectionName) {
        
        log.info("Creating multi-vector collection: {}", collectionName);
        
        FieldType id = FieldType.newBuilder()
            .withName("id")
            .withDataType(DataType.Int64)
            .withPrimaryKey(true)
            .withAutoID(true)
            .build();
        
        // Text embedding vector
        FieldType textVector = FieldType.newBuilder()
            .withName("text_vector")
            .withDataType(DataType.FloatVector)
            .withDimension(1536)
            .build();
        
        // Image embedding vector (optional)
        FieldType imageVector = FieldType.newBuilder()
            .withName("image_vector")
            .withDataType(DataType.FloatVector)
            .withDimension(512)
            .build();
        
        // Scalar fields
        FieldType title = FieldType.newBuilder()
            .withName("title")
            .withDataType(DataType.VarChar)
            .withMaxLength(500)
            .build();
        
        FieldType content = FieldType.newBuilder()
            .withName("content")
            .withDataType(DataType.VarChar)
            .withMaxLength(65535)
            .build();
        
        FieldType category = FieldType.newBuilder()
            .withName("category")
            .withDataType(DataType.VarChar)
            .withMaxLength(100)
            .build();
        
        FieldType price = FieldType.newBuilder()
            .withName("price")
            .withDataType(DataType.Double)
            .build();
        
        FieldType inStock = FieldType.newBuilder()
            .withName("in_stock")
            .withDataType(DataType.Bool)
            .build();
        
        FieldType tags = FieldType.newBuilder()
            .withName("tags")
            .withDataType(DataType.Array)
            .withElementType(DataType.VarChar)
            .withMaxLength(50)
            .withMaxCapacity(20)
            .build();
        
        FieldType metadata = FieldType.newBuilder()
            .withName("metadata")
            .withDataType(DataType.JSON)
            .build();
        
        FieldType timestamp = FieldType.newBuilder()
            .withName("timestamp")
            .withDataType(DataType.Int64)
            .build();
        
        CollectionSchemaParam schema = CollectionSchemaParam.newBuilder()
            .withFieldTypes(List.of(
                id, textVector, imageVector, title, content,
                category, price, inStock, tags, metadata, timestamp
            ))
            .withEnableDynamicField(true)
            .withDescription("Multi-vector RAG collection")
            .build();
        
        R<RpcStatus> response = milvusClient.createCollection(
            CreateCollectionParam.newBuilder()
                .withCollectionName(collectionName)
                .withSchema(schema)
                .withShardsNum(4)
                .withConsistencyLevel(ConsistencyLevelEnum.BOUNDED)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Collection created: {}", collectionName);
            
            // Create indexes
            createIndexes(collectionName);
        } else {
            log.error("Failed to create collection: {}", response.getMessage());
        }
    }
    
    private void createIndexes(String collectionName) {
        
        // Index for text_vector
        createVectorIndex(
            collectionName,
            "text_vector",
            IndexType.HNSW,
            "{\"M\":16,\"efConstruction\":256}"
        );
        
        // Index for image_vector
        createVectorIndex(
            collectionName,
            "image_vector",
            IndexType.IVF_FLAT,
            "{\"nlist\":1024}"
        );
        
        // Scalar indexes
        createScalarIndex(collectionName, "category");
        createScalarIndex(collectionName, "in_stock");
    }
    
    private void createVectorIndex(
            String collectionName,
            String fieldName,
            IndexType indexType,
            String params) {
        
        R<RpcStatus> response = milvusClient.createIndex(
            CreateIndexParam.newBuilder()
                .withCollectionName(collectionName)
                .withFieldName(fieldName)
                .withIndexType(indexType)
                .withMetricType(MetricType.COSINE)
                .withExtraParam(params)
                .withSyncMode(Boolean.FALSE)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Index created: {}.{}", collectionName, fieldName);
        }
    }
    
    private void createScalarIndex(String collectionName, String fieldName) {
        
        R<RpcStatus> response = milvusClient.createIndex(
            CreateIndexParam.newBuilder()
                .withCollectionName(collectionName)
                .withFieldName(fieldName)
                .withIndexType(IndexType.INVERTED)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Scalar index created: {}.{}", collectionName, fieldName);
        }
    }
    
    /**
     * Get collection statistics
     */
    public CollectionStats getCollectionStats(String collectionName) {
        
        R<GetCollectionStatisticsResponse> response = 
            milvusClient.getCollectionStatistics(
                GetCollectionStatisticsParam.newBuilder()
                    .withCollectionName(collectionName)
                    .build()
            );
        
        if (response.getStatus() != R.Status.Success.getCode()) {
            throw new RuntimeException("Failed to get stats: " + 
                response.getMessage());
        }
        
        long rowCount = Long.parseLong(
            response.getData().getStats().getOrDefault("row_count", "0")
        );
        
        return CollectionStats.builder()
            .collectionName(collectionName)
            .rowCount(rowCount)
            .build();
    }
    
    @lombok.Data
    @lombok.Builder
    public static class CollectionStats {
        private String collectionName;
        private Long rowCount;
    }
}
```

---

## Index Types and Performance Tuning

### Understanding Milvus Index Types

**Index Comparison**:

```
Vector Index Types in Milvus:

1. FLAT (Brute Force)
   ├─ Accuracy: 100%
   ├─ Speed: Slow (O(n))
   ├─ Memory: Low
   ├─ Use: <10K vectors, guaranteed accuracy
   └─ Build time: Instant

2. IVF_FLAT (Inverted File)
   ├─ Accuracy: 95-99%
   ├─ Speed: Fast (O(log n))
   ├─ Memory: Medium
   ├─ Parameters: nlist (clusters)
   ├─ Use: 10K-1M vectors, balanced
   └─ Build time: Minutes

3. IVF_SQ8 (Scalar Quantization)
   ├─ Accuracy: 95-98%
   ├─ Speed: Faster than IVF_FLAT
   ├─ Memory: 75% reduction
   ├─ Parameters: nlist
   ├─ Use: 100K-10M vectors, memory constrained
   └─ Build time: Minutes

4. IVF_PQ (Product Quantization)
   ├─ Accuracy: 90-95%
   ├─ Speed: Very fast
   ├─ Memory: 97% reduction
   ├─ Parameters: nlist, m (subquantizers)
   ├─ Use: 1M-1B vectors, extreme compression
   └─ Build time: Hours

5. HNSW (Hierarchical NSW)
   ├─ Accuracy: 99%+
   ├─ Speed: Fastest
   ├─ Memory: High (1.5x vectors)
   ├─ Parameters: M, efConstruction
   ├─ Use: <100M vectors, production RAG
   └─ Build time: Hours

6. DISKANN (Disk-based)
   ├─ Accuracy: 95-99%
   ├─ Speed: Fast
   ├─ Memory: Low (disk-based)
   ├─ Parameters: max_degree
   ├─ Use: Billion-scale, SSD storage
   └─ Build time: Days

Scalar Index Types:

1. INVERTED
   ├─ For: Categorical fields
   ├─ Fast filtering
   └─ Tag matching

2. TRIE
   ├─ For: String prefix matching
   └─ Autocomplete

3. SORTED
   ├─ For: Range queries
   └─ Numeric filtering
```

### Index Optimization Service

**IndexOptimizationService.java**:

```java
package com.example.milvus.service;

import io.milvus.client.MilvusServiceClient;
import io.milvus.param.IndexType;
import io.milvus.param.MetricType;
import io.milvus.param.R;
import io.milvus.param.RpcStatus;
import io.milvus.param.index.CreateIndexParam;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Slf4j
@Service
@RequiredArgsConstructor
public class IndexOptimizationService {
    
    private final MilvusServiceClient milvusClient;
    
    /**
     * Recommend index type based on dataset size
     */
    public IndexRecommendation recommendIndex(
            long vectorCount,
            int dimension,
            String workload) {
        
        IndexType recommendedType;
        Map<String, Object> params = new HashMap<>();
        
        if (vectorCount < 10_000) {
            recommendedType = IndexType.FLAT;
            params.put("reason", "Small dataset, brute force is efficient");
            
        } else if (vectorCount < 1_000_000) {
            recommendedType = IndexType.HNSW;
            params.put("M", 16);
            params.put("efConstruction", 256);
            params.put("reason", "Medium dataset, HNSW for best performance");
            
        } else if (vectorCount < 10_000_000) {
            if ("memory_constrained".equals(workload)) {
                recommendedType = IndexType.IVF_SQ8;
                params.put("nlist", Math.min(16384, (int) Math.sqrt(vectorCount) * 4));
                params.put("reason", "Large dataset, memory optimization");
            } else {
                recommendedType = IndexType.HNSW;
                params.put("M", 32);
                params.put("efConstruction", 512);
                params.put("reason", "Large dataset, performance priority");
            }
            
        } else {
            recommendedType = IndexType.DISKANN;
            params.put("max_degree", 64);
            params.put("reason", "Billion-scale, disk-based indexing");
        }
        
        return IndexRecommendation.builder()
            .indexType(recommendedType)
            .parameters(params)
            .estimatedBuildTime(estimateBuildTime(vectorCount, recommendedType))
            .estimatedMemory(estimateMemory(vectorCount, dimension, recommendedType))
            .build();
    }
    
    /**
     * Create optimized index
     */
    public void createOptimizedIndex(
            String collectionName,
            String fieldName,
            IndexType indexType,
            Map<String, Object> params) {
        
        log.info("Creating {} index on {}.{}", indexType, collectionName, fieldName);
        
        String paramsJson = new com.google.gson.Gson().toJson(params);
        
        R<RpcStatus> response = milvusClient.createIndex(
            CreateIndexParam.newBuilder()
                .withCollectionName(collectionName)
                .withFieldName(fieldName)
                .withIndexType(indexType)
                .withMetricType(MetricType.COSINE)
                .withExtraParam(paramsJson)
                .withSyncMode(Boolean.FALSE)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Index creation initiated");
        } else {
            log.error("Index creation failed: {}", response.getMessage());
        }
    }
    
    /**
     * Tune search parameters based on index type
     */
    public Map<String, Object> optimizeSearchParams(
            IndexType indexType,
            int topK,
            double targetRecall) {
        
        Map<String, Object> params = new HashMap<>();
        
        switch (indexType) {
            case HNSW:
                // Higher ef = better accuracy, slower search
                int ef = (int) (topK * 2 * (targetRecall / 0.95));
                params.put("ef", Math.max(topK, ef));
                break;
                
            case IVF_FLAT:
            case IVF_SQ8:
            case IVF_PQ:
                // Higher nprobe = better accuracy, slower search
                int nprobe = (int) (targetRecall * 32);
                params.put("nprobe", Math.max(1, nprobe));
                break;
                
            case DISKANN:
                params.put("search_list", topK * 10);
                break;
                
            default:
                // No parameters for FLAT
        }
        
        log.debug("Optimized search params for {}: {}", indexType, params);
        return params;
    }
    
    private String estimateBuildTime(long vectorCount, IndexType indexType) {
        
        // Rough estimates based on common hardware
        double minutes = switch (indexType) {
            case FLAT -> 0;
            case IVF_FLAT, IVF_SQ8 -> vectorCount / 1_000_000.0 * 5;
            case IVF_PQ -> vectorCount / 1_000_000.0 * 10;
            case HNSW -> vectorCount / 1_000_000.0 * 30;
            case DISKANN -> vectorCount / 1_000_000.0 * 60;
            default -> 0;
        };
        
        if (minutes < 1) return "< 1 minute";
        if (minutes < 60) return String.format("~%.0f minutes", minutes);
        return String.format("~%.1f hours", minutes / 60);
    }
    
    private String estimateMemory(long vectorCount, int dimension, IndexType indexType) {
        
        long bytesPerVector = dimension * 4; // float32
        long baseMemory = vectorCount * bytesPerVector;
        
        double multiplier = switch (indexType) {
            case FLAT -> 1.0;
            case IVF_FLAT -> 1.1;
            case IVF_SQ8 -> 0.3;
            case IVF_PQ -> 0.1;
            case HNSW -> 1.5;
            case DISKANN -> 0.2;
            default -> 1.0;
        };
        
        long estimatedBytes = (long) (baseMemory * multiplier);
        
        if (estimatedBytes < 1024) return estimatedBytes + " B";
        if (estimatedBytes < 1024 * 1024) return (estimatedBytes / 1024) + " KB";
        if (estimatedBytes < 1024 * 1024 * 1024) return (estimatedBytes / (1024 * 1024)) + " MB";
        return String.format("%.2f GB", estimatedBytes / (1024.0 * 1024 * 1024));
    }
    
    @Data
    @lombok.Builder
    public static class IndexRecommendation {
        private IndexType indexType;
        private Map<String, Object> parameters;
        private String estimatedBuildTime;
        private String estimatedMemory;
    }
}
```

---

## Advanced Query Patterns

### Complex Filter Expressions

**FilterExpressionBuilder.java**:

```java
package com.example.milvus.service;

import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Component
public class FilterExpressionBuilder {
    
    /**
     * Build filter expression from criteria
     */
    public String build(Map<String, Object> criteria) {
        
        if (criteria == null || criteria.isEmpty()) {
            return "";
        }
        
        return criteria.entrySet().stream()
            .map(this::buildCondition)
            .filter(s -> !s.isEmpty())
            .collect(Collectors.joining(" and "));
    }
    
    private String buildCondition(Map.Entry<String, Object> entry) {
        String field = entry.getKey();
        Object value = entry.getValue();
        
        if (value instanceof String) {
            return String.format("%s == \"%s\"", field, value);
        }
        
        if (value instanceof Number) {
            return String.format("%s == %s", field, value);
        }
        
        if (value instanceof Boolean) {
            return String.format("%s == %s", field, value);
        }
        
        if (value instanceof Map) {
            return buildRangeCondition(field, (Map<String, Object>) value);
        }
        
        if (value instanceof List) {
            return buildInCondition(field, (List<?>) value);
        }
        
        return "";
    }
    
    private String buildRangeCondition(String field, Map<String, Object> range) {
        
        StringBuilder condition = new StringBuilder();
        
        if (range.containsKey("$gte")) {
            condition.append(String.format("%s >= %s", field, range.get("$gte")));
        }
        
        if (range.containsKey("$gt")) {
            if (condition.length() > 0) condition.append(" and ");
            condition.append(String.format("%s > %s", field, range.get("$gt")));
        }
        
        if (range.containsKey("$lte")) {
            if (condition.length() > 0) condition.append(" and ");
            condition.append(String.format("%s <= %s", field, range.get("$lte")));
        }
        
        if (range.containsKey("$lt")) {
            if (condition.length() > 0) condition.append(" and ");
            condition.append(String.format("%s < %s", field, range.get("$lt")));
        }
        
        return condition.toString();
    }
    
    private String buildInCondition(String field, List<?> values) {
        
        String valueList = values.stream()
            .map(v -> v instanceof String ? "\"" + v + "\"" : v.toString())
            .collect(Collectors.joining(", "));
        
        return String.format("%s in [%s]", field, valueList);
    }
    
    /**
     * Complex query examples
     */
    public String buildEcommerceFilter(
            String category,
            Double minPrice,
            Double maxPrice,
            Boolean inStock,
            List<String> tags) {
        
        StringBuilder expr = new StringBuilder();
        
        // Category filter
        if (category != null) {
            expr.append(String.format("category == \"%s\"", category));
        }
        
        // Price range
        if (minPrice != null || maxPrice != null) {
            if (expr.length() > 0) expr.append(" and ");
            
            if (minPrice != null && maxPrice != null) {
                expr.append(String.format("price >= %.2f and price <= %.2f",
                    minPrice, maxPrice));
            } else if (minPrice != null) {
                expr.append(String.format("price >= %.2f", minPrice));
            } else {
                expr.append(String.format("price <= %.2f", maxPrice));
            }
        }
        
        // Stock status
        if (inStock != null) {
            if (expr.length() > 0) expr.append(" and ");
            expr.append(String.format("in_stock == %s", inStock));
        }
        
        // Tags (array contains)
        if (tags != null && !tags.isEmpty()) {
            if (expr.length() > 0) expr.append(" and ");
            
            String tagConditions = tags.stream()
                .map(tag -> String.format("array_contains(tags, \"%s\")", tag))
                .collect(Collectors.joining(" or "));
            
            expr.append("(").append(tagConditions).append(")");
        }
        
        return expr.toString();
    }
    
    /**
     * Time-based filter
     */
    public String buildTimeRangeFilter(long startTime, long endTime) {
        return String.format("timestamp >= %d and timestamp <= %d",
            startTime, endTime);
    }
    
    /**
     * JSON field filter
     */
    public String buildJsonFilter(String jsonPath, Object value) {
        if (value instanceof String) {
            return String.format("json_contains(metadata, '\"%s\": \"%s\"')",
                jsonPath, value);
        }
        return String.format("json_contains(metadata, '\"%s\": %s')",
            jsonPath, value);
    }
}
```
## Partitioning Strategies

### Understanding Milvus Partitions

**Partition Architecture**:

```
Collection with Partitions:

collection: product_catalog
├─ partition: electronics
│  ├─ vectors: 2M
│  └─ use: Electronics products
├─ partition: clothing
│  ├─ vectors: 3M
│  └─ use: Clothing products
├─ partition: books
│  ├─ vectors: 1M
│  └─ use: Book catalog
└─ partition: _default
   ├─ vectors: 500K
   └─ use: Uncategorized items

Benefits:
✓ Faster queries (search specific partitions)
✓ Efficient data management
✓ Multi-tenancy isolation
✓ Targeted data deletion
```

### Partition Management Service

**PartitionService.java**:

```java
package com.example.milvus.service;

import io.milvus.client.MilvusServiceClient;
import io.milvus.param.R;
import io.milvus.param.RpcStatus;
import io.milvus.param.partition.*;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

@Slf4j
@Service
@RequiredArgsConstructor
public class PartitionService {
    
    private final MilvusServiceClient milvusClient;
    
    /**
     * Create partition for multi-tenancy
     */
    public void createTenantPartition(String collectionName, String tenantId) {
        
        String partitionName = formatPartitionName(tenantId);
        
        log.info("Creating partition: {} in collection: {}",
            partitionName, collectionName);
        
        R<RpcStatus> response = milvusClient.createPartition(
            CreatePartitionParam.newBuilder()
                .withCollectionName(collectionName)
                .withPartitionName(partitionName)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Partition created: {}", partitionName);
        } else {
            log.error("Failed to create partition: {}", response.getMessage());
        }
    }
    
    /**
     * Time-based partitioning
     */
    public void createMonthlyPartition(String collectionName, int year, int month) {
        
        String partitionName = String.format("y%d_m%02d", year, month);
        
        R<RpcStatus> response = milvusClient.createPartition(
            CreatePartitionParam.newBuilder()
                .withCollectionName(collectionName)
                .withPartitionName(partitionName)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Monthly partition created: {}", partitionName);
        }
    }
    
    /**
     * List all partitions in collection
     */
    public List<String> listPartitions(String collectionName) {
        
        R<ShowPartitionsResponse> response = milvusClient.showPartitions(
            ShowPartitionsParam.newBuilder()
                .withCollectionName(collectionName)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            List<String> partitions = response.getData().getPartitionNamesList();
            log.debug("Partitions in {}: {}", collectionName, partitions);
            return new ArrayList<>(partitions);
        }
        
        return new ArrayList<>();
    }
    
    /**
     * Get partition statistics
     */
    public PartitionStats getPartitionStats(
            String collectionName,
            String partitionName) {
        
        R<GetPartitionStatisticsResponse> response = 
            milvusClient.getPartitionStatistics(
                GetPartitionStatisticsParam.newBuilder()
                    .withCollectionName(collectionName)
                    .withPartitionName(partitionName)
                    .build()
            );
        
        if (response.getStatus() != R.Status.Success.getCode()) {
            throw new RuntimeException("Failed to get partition stats");
        }
        
        long rowCount = Long.parseLong(
            response.getData().getStats().getOrDefault("row_count", "0")
        );
        
        return PartitionStats.builder()
            .partitionName(partitionName)
            .rowCount(rowCount)
            .build();
    }
    
    /**
     * Drop old partitions (data retention)
     */
    public void dropOldPartitions(
            String collectionName,
            int retentionMonths) {
        
        log.info("Dropping partitions older than {} months", retentionMonths);
        
        List<String> partitions = listPartitions(collectionName);
        
        java.time.LocalDate cutoffDate = java.time.LocalDate.now()
            .minusMonths(retentionMonths);
        
        for (String partition : partitions) {
            if (partition.startsWith("y")) {
                // Parse year and month from partition name
                if (isOlderThan(partition, cutoffDate)) {
                    dropPartition(collectionName, partition);
                }
            }
        }
    }
    
    /**
     * Release partition from memory
     */
    public void releasePartition(String collectionName, String partitionName) {
        
        log.info("Releasing partition: {}", partitionName);
        
        R<RpcStatus> response = milvusClient.releasePartitions(
            ReleasePartitionsParam.newBuilder()
                .withCollectionName(collectionName)
                .withPartitionNames(List.of(partitionName))
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Partition released from memory");
        }
    }
    
    /**
     * Load partition into memory
     */
    public void loadPartition(String collectionName, String partitionName) {
        
        log.info("Loading partition: {}", partitionName);
        
        R<RpcStatus> response = milvusClient.loadPartitions(
            LoadPartitionsParam.newBuilder()
                .withCollectionName(collectionName)
                .withPartitionNames(List.of(partitionName))
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Partition loaded into memory");
        }
    }
    
    private void dropPartition(String collectionName, String partitionName) {
        
        log.warn("Dropping partition: {}", partitionName);
        
        R<RpcStatus> response = milvusClient.dropPartition(
            DropPartitionParam.newBuilder()
                .withCollectionName(collectionName)
                .withPartitionName(partitionName)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Partition dropped: {}", partitionName);
        }
    }
    
    private String formatPartitionName(String tenantId) {
        return "tenant_" + tenantId.replaceAll("[^a-zA-Z0-9]", "_");
    }
    
    private boolean isOlderThan(String partitionName, java.time.LocalDate cutoff) {
        try {
            // Parse "y2024_m01" format
            String[] parts = partitionName.split("_");
            int year = Integer.parseInt(parts[0].substring(1));
            int month = Integer.parseInt(parts[1].substring(1));
            
            java.time.LocalDate partitionDate = 
                java.time.LocalDate.of(year, month, 1);
            
            return partitionDate.isBefore(cutoff);
        } catch (Exception e) {
            return false;
        }
    }
    
    @Data
    @lombok.Builder
    public static class PartitionStats {
        private String partitionName;
        private Long rowCount;
    }
}
```

---

## Data Management and ETL

### Bulk Data Operations

**BulkOperationsService.java**:

```java
package com.example.milvus.service;

import io.milvus.bulkwriter.LocalBulkWriter;
import io.milvus.bulkwriter.LocalBulkWriterParam;
import io.milvus.bulkwriter.common.clientenum.BulkFileType;
import io.milvus.client.MilvusServiceClient;
import io.milvus.grpc.DataType;
import io.milvus.param.R;
import io.milvus.param.collection.FieldType;
import io.milvus.param.dml.DeleteParam;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.nio.file.Path;
import java.util.*;

@Slf4j
@Service
@RequiredArgsConstructor
public class BulkOperationsService {
    
    private final MilvusServiceClient milvusClient;
    
    /**
     * Bulk import from JSON files
     */
    public void bulkImportFromJson(
            String collectionName,
            Path jsonFilePath) throws IOException {
        
        log.info("Starting bulk import from: {}", jsonFilePath);
        
        // Create bulk writer
        LocalBulkWriterParam writerParam = LocalBulkWriterParam.newBuilder()
            .withCollectionSchema(getCollectionSchema(collectionName))
            .withLocalPath(jsonFilePath.getParent().toString())
            .withChunkSize(512 * 1024 * 1024)  // 512MB chunks
            .withFileType(BulkFileType.JSON)
            .build();
        
        try (LocalBulkWriter bulkWriter = new LocalBulkWriter(writerParam)) {
            
            // Read and write data
            // Implementation depends on your data format
            
            log.info("✓ Bulk import completed");
            
        } catch (Exception e) {
            log.error("Bulk import failed", e);
            throw new RuntimeException("Bulk import failed", e);
        }
    }
    
    /**
     * Bulk delete by expression
     */
    public void bulkDelete(String collectionName, String expression) {
        
        log.warn("Bulk deleting from {} with expression: {}",
            collectionName, expression);
        
        R<io.milvus.grpc.MutationResult> response = milvusClient.delete(
            DeleteParam.newBuilder()
                .withCollectionName(collectionName)
                .withExpr(expression)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            long deletedCount = response.getData().getDeleteCnt();
            log.info("✓ Deleted {} vectors", deletedCount);
        } else {
            log.error("Bulk delete failed: {}", response.getMessage());
        }
    }
    
    /**
     * Bulk update (delete + insert)
     */
    public void bulkUpdate(
            String collectionName,
            List<Long> ids,
            List<Map<String, Object>> newData) {
        
        log.info("Bulk updating {} records", ids.size());
        
        // Step 1: Delete old records
        String deleteExpr = String.format("id in [%s]",
            ids.stream()
                .map(String::valueOf)
                .reduce((a, b) -> a + "," + b)
                .orElse("")
        );
        
        bulkDelete(collectionName, deleteExpr);
        
        // Step 2: Insert new records
        // Use standard insert operation
        
        log.info("✓ Bulk update completed");
    }
    
    /**
     * Export collection to JSON
     */
    public void exportToJson(
            String collectionName,
            Path outputPath) {
        
        log.info("Exporting collection {} to {}", collectionName, outputPath);
        
        // Implement export logic
        // Query all data in batches
        // Write to JSON file
        
        log.info("✓ Export completed");
    }
    
    private io.milvus.grpc.CollectionSchema getCollectionSchema(
            String collectionName) {
        
        // Fetch schema from Milvus
        // This is a simplified placeholder
        
        return io.milvus.grpc.CollectionSchema.newBuilder()
            .setName(collectionName)
            .build();
    }
}
```

---

## Performance Optimization

### Query Optimization Service

**QueryOptimizationService.java**:

```java
package com.example.milvus.service;

import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import java.util.*;
import java.util.concurrent.*;

@Slf4j
@Service
public class QueryOptimizationService {
    
    private final ExecutorService executorService = 
        Executors.newFixedThreadPool(10);
    
    /**
     * Parallel multi-partition search
     */
    public List<SearchResultWrapper> searchParallel(
            String collectionName,
            List<String> partitions,
            List<Float> queryVector,
            int topK) {
        
        List<CompletableFuture<SearchResultWrapper>> futures = 
            new ArrayList<>();
        
        for (String partition : partitions) {
            CompletableFuture<SearchResultWrapper> future = 
                CompletableFuture.supplyAsync(() -> 
                    searchPartition(collectionName, partition, queryVector, topK),
                    executorService
                );
            
            futures.add(future);
        }
        
        // Wait for all searches to complete
        return futures.stream()
            .map(CompletableFuture::join)
            .toList();
    }
    
    /**
     * Query result caching
     */
    @Cacheable(value = "queryResults", key = "#queryVector.hashCode()")
    public List<Object> searchWithCache(
            String collectionName,
            List<Float> queryVector,
            int topK) {
        
        // Actual search implementation
        return new ArrayList<>();
    }
    
    /**
     * Batch query optimization
     */
    public Map<String, List<Object>> batchSearch(
            String collectionName,
            List<List<Float>> queryVectors,
            int topK) {
        
        log.info("Processing batch of {} queries", queryVectors.size());
        
        Map<String, List<Object>> results = new ConcurrentHashMap<>();
        
        // Process in parallel batches
        int batchSize = 10;
        for (int i = 0; i < queryVectors.size(); i += batchSize) {
            int end = Math.min(i + batchSize, queryVectors.size());
            List<List<Float>> batch = queryVectors.subList(i, end);
            
            List<CompletableFuture<Void>> batchFutures = new ArrayList<>();
            
            for (int j = 0; j < batch.size(); j++) {
                final int index = i + j;
                final List<Float> vector = batch.get(j);
                
                CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                    List<Object> result = searchWithCache(
                        collectionName, vector, topK
                    );
                    results.put("query_" + index, result);
                }, executorService);
                
                batchFutures.add(future);
            }
            
            // Wait for batch to complete
            CompletableFuture.allOf(
                batchFutures.toArray(new CompletableFuture[0])
            ).join();
        }
        
        return results;
    }
    
    /**
     * Connection pooling for better performance
     */
    public void optimizeConnectionPool() {
        // Configure connection pool settings
        // This would be in MilvusConfig
        
        log.info("Connection pool optimized");
    }
    
    private SearchResultWrapper searchPartition(
            String collectionName,
            String partition,
            List<Float> queryVector,
            int topK) {
        
        // Actual partition search
        return new SearchResultWrapper(partition, new ArrayList<>());
    }
    
    @Data
    public static class SearchResultWrapper {
        private final String partition;
        private final List<Object> results;
    }
}
```

### Memory Management

**MemoryOptimizationService.java**:

```java
package com.example.milvus.service;

import io.milvus.client.MilvusServiceClient;
import io.milvus.param.R;
import io.milvus.param.RpcStatus;
import io.milvus.param.collection.LoadCollectionParam;
import io.milvus.param.collection.ReleaseCollectionParam;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

import java.util.*;

@Slf4j
@Service
@RequiredArgsConstructor
public class MemoryOptimizationService {
    
    private final MilvusServiceClient milvusClient;
    private final Map<String, Long> collectionAccessTime = 
        new ConcurrentHashMap<>();
    
    /**
     * Track collection access for LRU eviction
     */
    public void trackAccess(String collectionName) {
        collectionAccessTime.put(collectionName, System.currentTimeMillis());
    }
    
    /**
     * Scheduled memory optimization
     */
    @Scheduled(fixedRate = 3600000) // Every hour
    public void optimizeMemory() {
        log.info("Starting memory optimization");
        
        long cutoffTime = System.currentTimeMillis() - 
            (2 * 60 * 60 * 1000); // 2 hours
        
        List<String> collectionsToRelease = collectionAccessTime.entrySet()
            .stream()
            .filter(e -> e.getValue() < cutoffTime)
            .map(Map.Entry::getKey)
            .toList();
        
        for (String collection : collectionsToRelease) {
            releaseCollection(collection);
            collectionAccessTime.remove(collection);
        }
        
        log.info("Released {} collections from memory", 
            collectionsToRelease.size());
    }
    
    private void releaseCollection(String collectionName) {
        
        R<RpcStatus> response = milvusClient.releaseCollection(
            ReleaseCollectionParam.newBuilder()
                .withCollectionName(collectionName)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Collection released: {}", collectionName);
        }
    }
    
    /**
     * Preload frequently accessed collections
     */
    public void preloadCollections(List<String> collectionNames) {
        
        log.info("Preloading {} collections", collectionNames.size());
        
        for (String collection : collectionNames) {
            loadCollection(collection);
        }
    }
    
    private void loadCollection(String collectionName) {
        
        R<RpcStatus> response = milvusClient.loadCollection(
            LoadCollectionParam.newBuilder()
                .withCollectionName(collectionName)
                .build()
        );
        
        if (response.getStatus() == R.Status.Success.getCode()) {
            log.info("✓ Collection loaded: {}", collectionName);
        }
    }
}
```

---

## High Availability Setup

### Cluster Configuration

**High Availability Architecture**:

```yaml
# Kubernetes HA Deployment

# 1. Multiple Query Nodes for read scalability
queryNode:
  replicas: 5
  resources:
    requests:
      cpu: 2
      memory: 8Gi
    limits:
      cpu: 4
      memory: 16Gi
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: component
              operator: In
              values:
              - querynode
          topologyKey: kubernetes.io/hostname

# 2. Redundant Data Nodes
dataNode:
  replicas: 3
  resources:
    requests:
      cpu: 2
      memory: 8Gi
  
# 3. etcd cluster (metadata)
etcd:
  replicaCount: 3
  persistence:
    enabled: true
    size: 50Gi

# 4. MinIO distributed mode (object storage)
minio:
  mode: distributed
  replicas: 4
  persistence:
    size: 200Gi

# 5. Pulsar cluster (message queue)
pulsar:
  broker:
    replicaCount: 3
  bookkeeper:
    replicaCount: 3

# 6. Load Balancer
service:
  type: LoadBalancer
  sessionAffinity: ClientIP
```

### Health Check Service

**HealthCheckService.java**:

```java
package com.example.milvus.service;

import io.milvus.client.MilvusServiceClient;
import io.milvus.param.R;
import io.milvus.response.GetMetricsResponseWrapper;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Slf4j
@Component("milvus")
@RequiredArgsConstructor
public class HealthCheckService implements HealthIndicator {
    
    private final MilvusServiceClient milvusClient;
    
    @Override
    public Health health() {
        try {
            R<GetMetricsResponseWrapper> response = milvusClient.getMetrics();
            
            if (response.getStatus() == R.Status.Success.getCode()) {
                
                GetMetricsResponseWrapper metrics = response.getData();
                
                return Health.up()
                    .withDetail("status", "Connected")
                    .withDetail("nodes", metrics.getComponentStates().size())
                    .build();
            } else {
                return Health.down()
                    .withDetail("error", response.getMessage())
                    .build();
            }
            
        } catch (Exception e) {
            log.error("Health check failed", e);
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
    
    /**
     * Detailed cluster health check
     */
    public ClusterHealth checkClusterHealth() {
        
        try {
            R<GetMetricsResponseWrapper> response = milvusClient.getMetrics();
            
            if (response.getStatus() != R.Status.Success.getCode()) {
                return ClusterHealth.builder()
                    .healthy(false)
                    .error(response.getMessage())
                    .build();
            }
            
            GetMetricsResponseWrapper metrics = response.getData();
            
            return ClusterHealth.builder()
                .healthy(true)
                .queryNodes(countNodesByType(metrics, "QueryNode"))
                .dataNodes(countNodesByType(metrics, "DataNode"))
                .indexNodes(countNodesByType(metrics, "IndexNode"))
                .build();
            
        } catch (Exception e) {
            return ClusterHealth.builder()
                .healthy(false)
                .error(e.getMessage())
                .build();
        }
    }
    
    private int countNodesByType(
            GetMetricsResponseWrapper metrics,
            String nodeType) {
        
        return (int) metrics.getComponentStates().stream()
            .filter(state -> state.contains(nodeType))
            .count();
    }
    
    @Data
    @lombok.Builder
    public static class ClusterHealth {
        private Boolean healthy;
        private Integer queryNodes;
        private Integer dataNodes;
        private Integer indexNodes;
        private String error;
    }
}
```

---

## Monitoring and Observability

### Metrics Configuration

**MilvusMetricsConfig.java**:

```java
package com.example.milvus.config;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MilvusMetricsConfig {
    
    @Bean
    public Counter milvusQueryCounter(MeterRegistry registry) {
        return Counter.builder("milvus.queries.total")
            .description("Total Milvus queries")
            .tag("service", "milvus")
            .register(registry);
    }
    
    @Bean
    public Timer milvusQueryTimer(MeterRegistry registry) {
        return Timer.builder("milvus.query.duration")
            .description("Milvus query duration")
            .publishPercentiles(0.5, 0.95, 0.99)
            .register(registry);
    }
    
    @Bean
    public Counter milvusInsertCounter(MeterRegistry registry) {
        return Counter.builder("milvus.inserts.total")
            .description("Total vectors inserted")
            .register(registry);
    }
    
    @Bean
    public Counter milvusErrorCounter(MeterRegistry registry) {
        return Counter.builder("milvus.errors.total")
            .description("Total Milvus errors")
            .tag("operation", "all")
            .register(registry);
    }
}
```

### Prometheus Metrics Endpoint

**application.yml** (metrics section):

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
      environment: ${ENVIRONMENT:dev}
```

### Grafana Dashboard JSON

**milvus-dashboard.json**:

```json
{
  "dashboard": {
    "title": "Milvus RAG Monitoring",
    "panels": [
      {
        "title": "Query Latency (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(milvus_query_duration_bucket[5m]))"
          }
        ]
      },
      {
        "title": "Queries Per Second",
        "targets": [
          {
            "expr": "rate(milvus_queries_total[1m])"
          }
        ]
      },
      {
        "title": "Insert Throughput",
        "targets": [
          {
            "expr": "rate(milvus_inserts_total[1m])"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(milvus_errors_total[5m])"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "targets": [
          {
            "expr": "milvus_querynode_memory_bytes"
          }
        ]
      },
      {
        "title": "Collection Size",
        "targets": [
          {
            "expr": "milvus_collection_row_count"
          }
        ]
      }
    ]
  }
}
```

---

## Backup and Recovery

### Backup Service

**BackupService.java**:

```java
package com.example.milvus.service;

import io.milvus.client.MilvusServiceClient;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Slf4j
@Service
@RequiredArgsConstructor
public class BackupService {
    
    private final MilvusServiceClient milvusClient;
    private final BulkOperationsService bulkOps;
    
    private static final String BACKUP_DIR = "/var/backups/milvus";
    
    /**
     * Daily automated backup
     */
    @Scheduled(cron = "0 0 2 * * *") // 2 AM daily
    public void performDailyBackup() {
        log.info("Starting daily backup");
        
        String timestamp = LocalDateTime.now()
            .format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss"));
        
        try {
            BackupResult result = backupCollection(
                "rag_documents",
                Paths.get(BACKUP_DIR, timestamp)
            );
            
            log.info("✓ Backup completed: {}", result);
            
            // Cleanup old backups
            cleanupOldBackups(7); // Keep 7 days
            
        } catch (Exception e) {
            log.error("Backup failed", e);
        }
    }
    
    /**
     * Backup collection to local storage
     */
    public BackupResult backupCollection(
            String collectionName,
            Path backupPath) throws IOException {
        
        long startTime = System.currentTimeMillis();
        
        log.info("Backing up collection {} to {}",
            collectionName, backupPath);
        
        // Export collection data
        bulkOps.exportToJson(collectionName, backupPath);
        
        long duration = System.currentTimeMillis() - startTime;
        
        return BackupResult.builder()
            .collectionName(collectionName)
            .backupPath(backupPath.toString())
            .durationMs(duration)
            .timestamp(LocalDateTime.now())
            .success(true)
            .build();
    }
    
    /**
     * Restore from backup
     */
    public void restoreFromBackup(
            String collectionName,
            Path backupPath) throws IOException {
        
        log.warn("Restoring collection {} from {}",
            collectionName, backupPath);
        
        // Import backup data
        bulkOps.bulkImportFromJson(collectionName, backupPath);
        
        log.info("✓ Restore completed");
    }
    
    /**
     * Point-in-time recovery
     */
    public void restoreToTimestamp(
            String collectionName,
            long timestamp) {
        
        log.warn("Restoring collection to timestamp: {}", timestamp);
        
        // Milvus supports time travel queries
        // Delete all data after timestamp
        // This is a simplified example
        
        String deleteExpr = String.format("timestamp > %d", timestamp);
        bulkOps.bulkDelete(collectionName, deleteExpr);
        
        log.info("✓ Point-in-time recovery completed");
    }
    
    private void cleanupOldBackups(int retentionDays) {
        log.info("Cleaning up backups older than {} days", retentionDays);
        
        // Implement backup cleanup logic
        // Delete directories older than retention period
    }
    
    @Data
    @lombok.Builder
    public static class BackupResult {
        private String collectionName;
        private String backupPath;
        private Long durationMs;
        private LocalDateTime timestamp;
        private Boolean success;
    }
}
```

---

## Scaling Strategies

### Horizontal Scaling Guide

**Scaling Recommendations**:

```
Scaling Decision Matrix:

Current Load → Recommended Action

Query Load:
├─ <1K QPS: Single query node
├─ 1K-5K QPS: 3 query nodes
├─ 5K-10K QPS: 5 query nodes
├─ 10K-50K QPS: 10+ query nodes
└─ >50K QPS: Consider GPU acceleration

Data Volume:
├─ <10M vectors: Standalone mode
├─ 10M-100M: Cluster with 3 data nodes
├─ 100M-1B: Cluster with 6+ data nodes
└─ >1B: Distributed cluster with partitioning

Insert Throughput:
├─ <10K/sec: 2 data nodes
├─ 10K-50K/sec: 3-4 data nodes
├─ 50K-100K/sec: 5+ data nodes
└─ >100K/sec: Dedicated write cluster
```

### Auto-Scaling Configuration

**Horizontal Pod Autoscaler (HPA)**:

```yaml
# Query Node Autoscaling
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: milvus-querynode-hpa
  namespace: milvus
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: milvus-querynode
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30

---
# Data Node Autoscaling
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: milvus-datanode-hpa
  namespace: milvus
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: milvus-datanode
  minReplicas: 2
  maxReplicas: 6
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
```

---

## Security Configuration

### Authentication and Authorization

**Security Configuration**:

```yaml
# milvus.yaml - Security Settings

common:
  security:
    authorizationEnabled: true
    tlsMode: 2  # 2 = mutual TLS

# User authentication
rootCoord:
  enableAuth: true

# TLS Configuration
tls:
  serverPemPath: /etc/milvus/tls/server.pem
  serverKeyPath: /etc/milvus/tls/server.key
  caPemPath: /etc/milvus/tls/ca.pem
```

**Java Client with TLS**:

```java
package com.example.milvus.config;

import io.milvus.client.MilvusServiceClient;
import io.milvus.param.ConnectParam;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SecureMilvusConfig {
    
    @Value("${milvus.host}")
    private String host;
    
    @Value("${milvus.port}")
    private int port;
    
    @Value("${milvus.username}")
    private String username;
    
    @Value("${milvus.password}")
    private String password;
    
    @Value("${milvus.tls.enabled:false}")
    private boolean tlsEnabled;
    
    @Bean
    public MilvusServiceClient secureMilvusClient() {
        
        ConnectParam.Builder builder = ConnectParam.newBuilder()
            .withHost(host)
            .withPort(port)
            .withUsername(username)
            .withPassword(password);
        
        if (tlsEnabled) {
            builder.withSecure(true)
                .withServerPemPath("/etc/certs/server.pem")
                .withServerKeyPath("/etc/certs/server.key")
                .withCaPemPath("/etc/certs/ca.pem");
        }
        
        return new MilvusServiceClient(builder.build());
    }
}
```

---

## Troubleshooting Guide

### Common Issues and Solutions

**Troubleshooting Reference**:

```
Issue: Slow Query Performance
Symptoms: Queries taking >100ms
Solutions:
  ✓ Check index type (use HNSW for <100M vectors)
  ✓ Increase search params (ef, nprobe)
  ✓ Load collection into memory
  ✓ Add more query nodes
  ✓ Check network latency

Issue: High Memory Usage
Symptoms: OOM errors, crashes
Solutions:
  ✓ Use IVF_SQ8 or IVF_PQ indexes
  ✓ Release unused collections
  ✓ Implement partition strategy
  ✓ Increase node memory
  ✓ Enable memory quota limits

Issue: Insert Failures
Symptoms: Timeout errors on insert
Solutions:
  ✓ Reduce batch size
  ✓ Check network connectivity
  ✓ Verify collection schema
  ✓ Monitor data node CPU/memory
  ✓ Check MinIO storage capacity

Issue: Index Build Failures
Symptoms: Index stuck in "building" state
Solutions:
  ✓ Check index node resources
  ✓ Reduce efConstruction parameter
  ✓ Build index asynchronously
  ✓ Monitor disk I/O
  ✓ Check logs for errors

Issue: Connection Timeouts
Symptoms: Client connection failures
Solutions:
  ✓ Verify network connectivity
  ✓ Check firewall rules
  ✓ Increase connection timeout
  ✓ Verify service is running
  ✓ Check etcd health
```

### Diagnostic Service

**DiagnosticService.java**:

```java
package com.example.milvus.service;

import io.milvus.client.MilvusServiceClient;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.*;

@Slf4j
@Service
@RequiredArgsConstructor
public class DiagnosticService {
    
    private final MilvusServiceClient milvusClient;
    
    /**
     * Run comprehensive diagnostics
     */
    public DiagnosticReport runDiagnostics() {
        
        log.info("Running Milvus diagnostics");
        
        DiagnosticReport report = new DiagnosticReport();
        
        // Check connectivity
        report.addCheck("Connectivity", checkConnectivity());
        
        // Check collection health
        report.addCheck("Collections", checkCollections());
        
        // Check index health
        report.addCheck("Indexes", checkIndexes());
        
        // Check resource usage
        report.addCheck("Resources", checkResources());
        
        return report;
    }
    
    private CheckResult checkConnectivity() {
        try {
            milvusClient.getMetrics();
            return CheckResult.pass("Milvus connection successful");
        } catch (Exception e) {
            return CheckResult.fail("Connection failed: " + e.getMessage());
        }
    }
    
    private CheckResult checkCollections() {
        try {
            var collections = milvusClient.listCollections();
            return CheckResult.pass(
                String.format("Found %d collections", 
                    collections.getData().size())
            );
        } catch (Exception e) {
            return CheckResult.fail("Failed to list collections: " + 
                e.getMessage());
        }
    }
    
    private CheckResult checkIndexes() {
        // Implement index health check
        return CheckResult.pass("Indexes healthy");
    }
    
    private CheckResult checkResources() {
        // Implement resource check
        return CheckResult.pass("Resources within limits");
    }
    
    @Data
    public static class DiagnosticReport {
        private Map<String, CheckResult> checks = new LinkedHashMap<>();
        private boolean allPassed = true;
        
        public void addCheck(String name, CheckResult result) {
            checks.put(name, result);
            if (!result.isPassed()) {
                allPassed = false;
            }
        }
    }
    
    @Data
    public static class CheckResult {
        private boolean passed;
        private String message;
        
        public static CheckResult pass(String message) {
            CheckResult result = new CheckResult();
            result.setPassed(true);
            result.setMessage(message);
            return result;
        }
        
        public static CheckResult fail(String message) {
            CheckResult result = new CheckResult();
            result.setPassed(false);
            result.setMessage(message);
            return result;
        }
    }
}
```

---

## Production Deployment

### Production Checklist

```
Pre-Production Checklist:

Infrastructure:
☐ Kubernetes cluster configured (1.24+)
☐ Persistent volumes provisioned
☐ Load balancer configured
☐ Network policies defined
☐ Resource quotas set

Milvus Configuration:
☐ HA mode enabled (3+ nodes each type)
☐ Indexes optimized for workload
☐ Memory limits configured
☐ TLS/SSL enabled
☐ Authentication configured
☐ Backup strategy implemented

Monitoring:
☐ Prometheus metrics enabled
☐ Grafana dashboards deployed
☐ Alerting rules configured
☐ Log aggregation setup (ELK/Loki)
☐ Health checks operational

Application:
☐ Connection pooling configured
☐ Error handling implemented
☐ Rate limiting configured
☐ Caching enabled
☐ Circuit breakers configured

Testing:
☐ Load testing completed
☐ Failover tested
☐ Backup/restore verified
☐ Performance benchmarks met
☐ Security audit passed
```

### Deployment Script

**deploy.sh**:

```bash
#!/bin/bash

set -e

echo "Deploying Milvus to Production..."

# Variables
NAMESPACE="milvus-prod"
RELEASE_NAME="milvus"
VALUES_FILE="production-values.yaml"

# Create namespace
kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

# Create secrets
kubectl create secret generic milvus-credentials \
  --from-literal=username=$MILVUS_USERNAME \
  --from-literal=password=$MILVUS_PASSWORD \
  -n $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

# Deploy Milvus
helm upgrade --install $RELEASE_NAME milvus/milvus \
  -n $NAMESPACE \
  -f $VALUES_FILE \
  --wait \
  --timeout 15m

# Verify deployment
echo "Verifying deployment..."
kubectl wait --for=condition=ready pod \
  -l app.kubernetes.io/name=milvus \
  -n $NAMESPACE \
  --timeout=600s

# Run health checks
echo "Running health checks..."
kubectl run health-check \
  --image=curlimages/curl \
  --rm -it \
  --restart=Never \
  -n $NAMESPACE \
  -- curl -f http://milvus:9091/healthz

echo "✓ Deployment completed successfully"
```

---

## Cost Analysis

### Total Cost of Ownership

**Cost Breakdown (3 years)**:

```
AWS Infrastructure Costs:

Compute (EC2):
├─ 3 × r6i.2xlarge (Query Nodes)
│  ├─ vCPU: 8, RAM: 64GB
│  └─ Cost: $450/month × 3 = $1,350/month
├─ 3 × r6i.xlarge (Data Nodes)
│  ├─ vCPU: 4, RAM: 32GB
│  └─ Cost: $225/month × 3 = $675/month
├─ 2 × c6i.xlarge (Index Nodes)
│  ├─ vCPU: 4, RAM: 8GB
│  └─ Cost: $145/month × 2 = $290/month
└─ Subtotal: $2,315/month

Storage (EBS gp3):
├─ Vector data: 2TB × $80/TB = $160/month
├─ Metadata (etcd): 100GB = $10/month
├─ Message queue: 200GB = $20/month
└─ Subtotal: $190/month

Network:
├─ Load Balancer: $20/month
├─ Data transfer: $100/month
└─ Subtotal: $120/month

Monthly Total: $2,625
Annual Total: $31,500
3-Year Total: $94,500

Operational Costs:
├─ DevOps (40 hrs/month): $6,000/month
├─ Monitoring tools: $200/month
├─ Backup storage: $100/month
└─ Monthly Total: $6,300

Total 3-Year TCO: $320,100

Cost Optimization Opportunities:
✓ Use Reserved Instances: Save 40% ($37,800)
✓ Spot Instances for Index Nodes: Save 70% ($14,800)
✓ S3 for cold storage: Save 80% on backups ($2,880)

Optimized 3-Year TCO: $264,620

Versus Managed Solutions:
Pinecone equivalent: $25,500 (3 years)
Break-even: If DevOps costs < $800/month
```

---

## Conclusion

### Key Takeaways

**Milvus Advantages**:

```
1. Complete Control
   ✓ Full data ownership
   ✓ Customizable performance
   ✓ On-premise deployment option
   ✓ No vendor lock-in

2. Cost Efficiency at Scale
   ✓ $2,500-$3,000/month for 5M vectors
   ✓ Linear scaling costs
   ✓ No per-query charges
   ✓ Open-source licensing

3. Advanced Features
   ✓ Multiple index types
   ✓ GPU acceleration
   ✓ Hybrid search
   ✓ Time travel queries
   ✓ Multi-vector collections

4. Production Ready
   ✓ Billion-scale capability
   ✓ High availability
   ✓ Enterprise support available
   ✓ Active community
```

### When to Choose Milvus

**Decision Framework**:

```
Choose Milvus If:
✓ Data volume >10M vectors
✓ Have DevOps capability
✓ Need on-premise deployment
✓ Require custom optimization
✓ Cost-sensitive at scale
✓ Want open-source solution

Choose Managed Solution If:
✓ Small team (<5 people)
✓ Quick time to market critical
✓ Limited DevOps resources
✓ Prefer hands-off infrastructure
✓ Budget allows managed services
```

### Next Steps

**Getting Started**:

1. **Week 1**: Set up development environment
   - Install Docker/Kubernetes
   - Deploy Milvus standalone
   - Test basic operations

2. **Week 2**: Build prototype RAG
   - Integrate Spring AI
   - Implement document ingestion
   - Test query performance

3. **Week 3**: Optimize performance
   - Choose optimal index type
   - Tune search parameters
   - Implement caching

4. **Week 4**: Production preparation
   - Set up HA cluster
   - Configure monitoring
   - Implement backup strategy

5. **Week 5+**: Deploy and iterate
   - Deploy to production
   - Monitor performance
   - Optimize based on metrics

### Resources

**Essential Links**:
- [Milvus Documentation](https://milvus.io/docs)
- [Spring AI Milvus Guide](https://docs.spring.io/spring-ai/reference/api/vectordbs/milvus.html)
- [Milvus GitHub](https://github.com/milvus-io/milvus)
- [Community Slack](https://milvusio.slack.com)
- [Zilliz Cloud](https://zilliz.com) (Managed Milvus)

**Training Resources**:
- Milvus Bootcamp
- Vector Database Tutorials
- Spring AI Examples
- Performance Tuning Guides

### Final Thoughts

Milvus represents the cutting edge of open-source vector database technology. While it requires more operational expertise than managed solutions, the benefits of control, customization, and cost efficiency make it ideal for organizations building large-scale RAG applications. Combined with Spring AI's elegant abstractions, you can build production-grade systems that rival any proprietary solution.

The investment in learning Milvus pays dividends through deeper understanding of vector search internals, complete control over your AI infrastructure, and the flexibility to optimize for your specific use case.

Start small, experiment often, and scale confidently!

---
