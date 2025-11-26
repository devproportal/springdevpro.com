
基于下面的信息，给出英文技术博客文章（面向欧美用户，基于 Google Adsense赚钱）：
文章为主，代码为辅。
要有图表和表格。

Reference Title: Spring AI + PostgreSQL pgvector: Low-Cost RAG Solution
Reference Keywords: spring ai pgvector
Target Word Count: 6000

# Spring AI + PostgreSQL pgvector: Low-Cost RAG Solution

**Build Production-Ready Retrieval Augmented Generation Systems Without Breaking the Bank**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Why pgvector for RAG?](#why-pgvector-for-rag)
3. [Architecture Overview](#architecture-overview)
4. [Environment Setup](#environment-setup)
5. [PostgreSQL and pgvector Installation](#postgresql-and-pgvector-installation)
6. [Spring AI Configuration](#spring-ai-configuration)
7. [Building Your First RAG Application](#building-your-first-rag-application)
8. [Document Processing Pipeline](#document-processing-pipeline)
9. [Advanced Querying Techniques](#advanced-querying-techniques)
10. [Performance Optimization](#performance-optimization)
11. [Scaling Strategies](#scaling-strategies)
12. [Production Deployment](#production-deployment)
13. [Cost Analysis](#cost-analysis)
14. [Real-World Use Cases](#real-world-use-cases)
15. [Troubleshooting Guide](#troubleshooting-guide)
16. [Migration from Other Vector Databases](#migration-from-other-vector-databases)
17. [Best Practices](#best-practices)
18. [Conclusion](#conclusion)

---

## Introduction

Retrieval Augmented Generation (RAG) has revolutionized how we build AI applications, enabling chatbots, question-answering systems, and intelligent assistants to provide accurate, contextual responses based on your own data. However, many developers are deterred by the perceived complexity and cost of vector databases.

### What You'll Learn

This comprehensive guide demonstrates how to build a production-ready RAG system using **Spring AI** and **PostgreSQL with pgvector** - a combination that offers:

- **Low Cost**: Use your existing PostgreSQL infrastructure
- **Simplicity**: Leverage familiar SQL and JDBC
- **Reliability**: ACID transactions and proven PostgreSQL stability
- **Performance**: Sub-second searches on millions of documents
- **Flexibility**: Combine vector search with complex relational queries

### Who This Guide Is For

- **Spring Boot Developers** wanting to add AI capabilities to existing applications
- **Startups** building MVPs on a budget
- **Enterprises** already invested in PostgreSQL infrastructure
- **AI Engineers** seeking cost-effective RAG solutions

### Prerequisites

```
Required Knowledge:
- Java 17+ and Spring Boot basics
- Basic SQL and PostgreSQL familiarity
- Understanding of REST APIs
- Basic AI/LLM concepts

Tools Needed:
- JDK 17 or higher
- Maven or Gradle
- Docker (for local development)
- PostgreSQL 15+ with pgvector
- OpenAI API key (or alternative LLM provider)
```

Let's build a RAG system that's both powerful and affordable!

---

## Why pgvector for RAG?

### The RAG Challenge

Traditional LLMs have limitations:
- **Knowledge Cutoff**: Don't know about recent events or your proprietary data
- **Hallucinations**: Generate plausible but incorrect information
- **Context Window**: Limited amount of information they can process at once

**RAG solves these problems** by retrieving relevant information from your knowledge base before generating responses.

### The Vector Database Dilemma

Most RAG implementations use specialized vector databases, which introduce:

```
Common Pain Points:
❌ Additional infrastructure to manage
❌ Monthly costs ranging from $50-$500+
❌ New APIs and query languages to learn
❌ Data duplication across databases
❌ Complex backup and recovery procedures
```

### The pgvector Advantage

**pgvector** brings vector search capabilities directly to PostgreSQL:

```
Benefits:
✅ Use your existing PostgreSQL database
✅ ACID transactions ensure data consistency
✅ Familiar SQL interface
✅ Combine vector search with relational queries
✅ Proven backup and recovery tools
✅ No additional licensing costs
```

### Performance Reality Check

**Misconception**: "pgvector is slow compared to specialized databases"

**Reality**: For most applications, pgvector is more than sufficient:

```
Performance Metrics:

Dataset: 500K documents, 1536-dimensional vectors

Query Performance:
- Search latency: 30-80ms (p95)
- Throughput: 200-500 QPS
- Recall@10: 95-98%

Specialized Vector DB:
- Search latency: 10-30ms (p95)
- Throughput: 1000+ QPS
- Recall@10: 98-99%

Verdict: 
For <1M vectors, pgvector provides excellent 
performance at a fraction of the cost.
```

### Cost Comparison

**Real-World Example**: Customer support knowledge base

```
Scenario: 100K documents, 10K queries/day

Pinecone:
- Infrastructure: $70/month
- Annual cost: $840

Milvus (AWS):
- EC2 instance: $120/month
- Storage: $20/month
- Annual cost: $1,680

pgvector (existing PostgreSQL):
- Additional cost: $0
- Slight increase in RDS size: ~$50/month
- Annual cost: $600

Savings: $240-$1,080/year
```

### When pgvector Might Not Be Ideal

**Be honest about limitations**:

```
Consider specialized vector DB if:
- Dataset > 10 million vectors
- Need sub-10ms query latency
- Require advanced features (hybrid search, GPU acceleration)
- Vector workload is completely separate from relational data
- Need to scale vector operations independently

For most RAG applications with <1M documents,
pgvector is the smart choice.
```

---

## Architecture Overview

### System Components

```
┌─────────────────────────────────────────────────┐
│              User Interface                      │
│         (Web App / Mobile / API)                │
└──────────────────┬──────────────────────────────┘
                   │
                   │ HTTP/REST
                   ▼
┌─────────────────────────────────────────────────┐
│           Spring Boot Application               │
│                                                  │
│  ┌────────────────────────────────────────┐    │
│  │      RAG Controller                     │    │
│  │  - Receive user queries                 │    │
│  │  - Orchestrate RAG pipeline             │    │
│  └───────────┬────────────────────────────┘    │
│              │                                   │
│  ┌───────────▼────────────────────────────┐    │
│  │      RAG Service                        │    │
│  │  1. Generate query embedding            │    │
│  │  2. Retrieve relevant documents         │    │
│  │  3. Build context prompt                │    │
│  │  4. Generate response with LLM          │    │
│  └───┬──────────────────────────┬─────────┘    │
│      │                           │               │
│      │ Spring AI                 │               │
│      │                           │               │
│  ┌───▼─────────────┐      ┌─────▼──────────┐   │
│  │ EmbeddingClient │      │   ChatClient   │   │
│  │ (OpenAI/Ollama) │      │  (OpenAI/etc)  │   │
│  └───┬─────────────┘      └────────────────┘   │
│      │                                           │
│  ┌───▼─────────────────────────────────────┐   │
│  │      PgVectorStore                       │   │
│  │  - Store document embeddings             │   │
│  │  - Perform similarity search             │   │
│  └───┬──────────────────────────────────────┘   │
│      │ JDBC                                      │
└──────┼───────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│         PostgreSQL + pgvector                    │
│                                                  │
│  ┌────────────────────────────────────────┐    │
│  │  vector_store table                     │    │
│  │                                          │    │
│  │  - id (UUID)                             │    │
│  │  - content (TEXT)                        │    │
│  │  - metadata (JSONB)                      │    │
│  │  - embedding (vector(1536))              │    │
│  │                                          │    │
│  │  + HNSW index on embedding               │    │
│  └────────────────────────────────────────┘    │
│                                                  │
│  ┌────────────────────────────────────────┐    │
│  │  Your Application Tables                │    │
│  │  (users, orders, products, etc.)        │    │
│  └────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

### Data Flow: RAG Query Process

```java
// Step-by-step RAG flow
public String processQuery(String userQuestion) {
    
    // 1. Generate embedding for user question
    float[] questionEmbedding = embeddingClient.embed(userQuestion);
    
    // 2. Search for similar documents in pgvector
    List<Document> relevantDocs = vectorStore.similaritySearch(
        SearchRequest.query(userQuestion)
            .withTopK(5)
            .withSimilarityThreshold(0.7)
    );
    
    // 3. Build context from retrieved documents
    String context = relevantDocs.stream()
        .map(Document::getContent)
        .collect(Collectors.joining("\n\n"));
    
    // 4. Generate answer with LLM using context
    String prompt = String.format("""
        Answer the question based on the following context.
        If the answer cannot be found in the context, say so.
        
        Context:
        %s
        
        Question: %s
        
        Answer:
        """, context, userQuestion);
    
    return chatClient.prompt()
        .user(prompt)
        .call()
        .content();
}
```

### Technology Stack

```yaml
Backend:
  Framework: Spring Boot 3.2+
  Language: Java 17+
  AI Integration: Spring AI
  Database: PostgreSQL 15+
  Vector Extension: pgvector 0.5.0+

AI Services:
  Embeddings: OpenAI text-embedding-3-small
  LLM: OpenAI GPT-4 / GPT-3.5-turbo
  Alternatives: Ollama (local), Anthropic Claude

Infrastructure:
  Development: Docker Compose
  Production: AWS RDS PostgreSQL / Azure Database
  Monitoring: Spring Boot Actuator + Prometheus
```

---

## Environment Setup

### Project Structure

```
rag-pgvector-demo/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/rag/
│   │   │       ├── RagApplication.java
│   │   │       ├── config/
│   │   │       │   ├── PgVectorConfig.java
│   │   │       │   ├── OpenAIConfig.java
│   │   │       │   └── SecurityConfig.java
│   │   │       ├── controller/
│   │   │       │   └── RagController.java
│   │   │       ├── service/
│   │   │       │   ├── DocumentService.java
│   │   │       │   ├── RagService.java
│   │   │       │   └── EmbeddingService.java
│   │   │       ├── repository/
│   │   │       │   └── DocumentRepository.java
│   │   │       ├── model/
│   │   │       │   ├── QueryRequest.java
│   │   │       │   └── QueryResponse.java
│   │   │       └── util/
│   │   │           └── TextSplitter.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       └── schema.sql
│   └── test/
│       └── java/
│           └── com/example/rag/
│               └── RagServiceTest.java
├── docker/
│   └── docker-compose.yml
├── pom.xml
└── README.md
```

### Maven Dependencies (pom.xml)

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
        <relativePath/>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>rag-pgvector-demo</artifactId>
    <version>1.0.0</version>
    <name>RAG with pgvector</name>
    
    <properties>
        <java.version>17</java.version>
        <spring-ai.version>1.0.0-M3</spring-ai.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Spring AI -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
            <version>${spring-ai.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
            <version>${spring-ai.version}</version>
        </dependency>
        
        <!-- PostgreSQL -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Utilities -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Monitoring -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        
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
        
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
```

### Gradle Alternative (build.gradle)

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.example'
version = '1.0.0'
sourceCompatibility = '17'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
    maven { url 'https://repo.spring.io/milestone' }
}

ext {
    springAiVersion = '1.0.0-M3'
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jdbc'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    
    implementation "org.springframework.ai:spring-ai-openai-spring-boot-starter:${springAiVersion}"
    implementation "org.springframework.ai:spring-ai-pgvector-store-spring-boot-starter:${springAiVersion}"
    
    runtimeOnly 'org.postgresql:postgresql'
    
    implementation 'io.micrometer:micrometer-registry-prometheus'
    
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.testcontainers:postgresql'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

---

## PostgreSQL and pgvector Installation

### Local Development with Docker

**docker-compose.yml**:

```yaml
version: '3.8'

services:
  postgres:
    image: pgvector/pgvector:pg16
    container_name: rag-postgres
    environment:
      POSTGRES_DB: ragdb
      POSTGRES_USER: raguser
      POSTGRES_PASSWORD: ragpass
      POSTGRES_INITDB_ARGS: "-E UTF8"
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    command: >
      postgres
      -c shared_preload_libraries=vector
      -c max_connections=200
      -c shared_buffers=256MB
      -c effective_cache_size=1GB
      -c maintenance_work_mem=64MB
      -c checkpoint_completion_target=0.9
      -c wal_buffers=16MB
      -c default_statistics_target=100
      -c random_page_cost=1.1
      -c effective_io_concurrency=200
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U raguser -d ragdb"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

**Start PostgreSQL**:

```bash
# Start PostgreSQL with pgvector
docker-compose up -d

# Verify pgvector is available
docker exec -it rag-postgres psql -U raguser -d ragdb -c "CREATE EXTENSION IF NOT EXISTS vector;"

# Check version
docker exec -it rag-postgres psql -U raguser -d ragdb -c "SELECT extversion FROM pg_extension WHERE extname = 'vector';"
```

### Manual PostgreSQL Installation

**Ubuntu/Debian**:

```bash
# Install PostgreSQL 16
sudo apt install -y postgresql-16 postgresql-server-dev-16

# Install build dependencies
sudo apt install -y build-essential git

# Clone and build pgvector
git clone https://github.com/pgvector/pgvector.git
cd pgvector
make
sudo make install

# Restart PostgreSQL
sudo systemctl restart postgresql

# Create database
sudo -u postgres createdb ragdb
sudo -u postgres createuser raguser

# Enable extension
sudo -u postgres psql -d ragdb -c "CREATE EXTENSION vector;"
```

**macOS with Homebrew**:

```bash
# Install PostgreSQL
brew install postgresql@16

# Install pgvector
brew install pgvector

# Start PostgreSQL
brew services start postgresql@16

# Create database
createdb ragdb
psql ragdb -c "CREATE EXTENSION vector;"
```

### Database Schema Initialization

**init-scripts/01-create-schema.sql**:

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create vector store table
CREATE TABLE IF NOT EXISTS vector_store (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content TEXT NOT NULL,
    metadata JSONB DEFAULT '{}'::jsonb,
    embedding vector(1536),  -- OpenAI text-embedding-3-small dimension
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX IF NOT EXISTS vector_store_embedding_idx 
    ON vector_store 
    USING hnsw (embedding vector_cosine_ops)
    WITH (m = 16, ef_construction = 64);

CREATE INDEX IF NOT EXISTS vector_store_metadata_idx 
    ON vector_store 
    USING gin (metadata);

CREATE INDEX IF NOT EXISTS vector_store_created_at_idx 
    ON vector_store (created_at DESC);

-- Create function to update updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Create trigger
CREATE TRIGGER update_vector_store_updated_at 
    BEFORE UPDATE ON vector_store
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Create view for vector statistics
CREATE OR REPLACE VIEW vector_store_stats AS
SELECT 
    COUNT(*) as total_documents,
    COUNT(DISTINCT metadata->>'source') as unique_sources,
    AVG(length(content)) as avg_content_length,
    MIN(created_at) as oldest_document,
    MAX(created_at) as newest_document,
    pg_size_pretty(pg_total_relation_size('vector_store')) as table_size,
    pg_size_pretty(pg_indexes_size('vector_store')) as index_size
FROM vector_store;

-- Grant permissions
GRANT ALL PRIVILEGES ON DATABASE ragdb TO raguser;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO raguser;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO raguser;
```

### Verify Installation

```sql
-- Check pgvector version
SELECT extversion FROM pg_extension WHERE extname = 'vector';

-- Test vector operations
SELECT '[1,2,3]'::vector <-> '[1,2,4]'::vector AS distance;

-- Check HNSW index
SELECT 
    indexname, 
    indexdef 
FROM pg_indexes 
WHERE tablename = 'vector_store';

-- View table structure
\d vector_store

-- Check statistics view
SELECT * FROM vector_store_stats;
```

---

## Spring AI Configuration

### Application Configuration

**application.yml**:

```yaml
spring:
  application:
    name: rag-pgvector-demo
  
  datasource:
    url: jdbc:postgresql://localhost:5432/ragdb
    username: raguser
    password: ragpass
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 20000
      idle-timeout: 300000
      max-lifetime: 1200000
  
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4-turbo-preview
          temperature: 0.7
          max-tokens: 1000
      embedding:
        options:
          model: text-embedding-3-small
    
    vectorstore:
      pgvector:
        index-type: HNSW
        distance-type: COSINE_DISTANCE
        dimensions: 1536
        remove-existing-vector-store-table: false
        schema-name: public
        table-name: vector_store

logging:
  level:
    root: INFO
    com.example.rag: DEBUG
    org.springframework.ai: DEBUG

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

**application-dev.yml** (Development overrides):

```yaml
spring:
  ai:
    openai:
      chat:
        options:
          model: gpt-3.5-turbo  # Cheaper for development
          temperature: 0.7
  
  datasource:
    hikari:
      maximum-pool-size: 5

logging:
  level:
    org.springframework.jdbc.core: TRACE
```

### Core Configuration Classes

**PgVectorConfig.java**:

```java
package com.example.rag.config;

import org.springframework.ai.embedding.EmbeddingClient;
import org.springframework.ai.vectorstore.PgVectorStore;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;

@Configuration
public class PgVectorConfig {
    
    @Value("${spring.ai.vectorstore.pgvector.dimensions:1536}")
    private int dimensions;
    
    @Value("${spring.ai.vectorstore.pgvector.index-type:HNSW}")
    private String indexType;
    
    @Value("${spring.ai.vectorstore.pgvector.distance-type:COSINE_DISTANCE}")
    private String distanceType;
    
    @Bean
    public VectorStore vectorStore(
            JdbcTemplate jdbcTemplate,
            EmbeddingClient embeddingClient) {
        
        return new PgVectorStore(
            jdbcTemplate,
            embeddingClient,
            PgVectorStore.PgVectorStoreConfig.builder()
                .withSchemaName("public")
                .withTableName("vector_store")
                .withIndexType(parseIndexType(indexType))
                .withDistanceType(parseDistanceType(distanceType))
                .withDimensions(dimensions)
                .withRemoveExistingVectorStoreTable(false)
                .withSchemaValidation(true)
                .build()
        );
    }
    
    private PgVectorStore.PgIndexType parseIndexType(String type) {
        return switch (type.toUpperCase()) {
            case "HNSW" -> PgVectorStore.PgIndexType.HNSW;
            case "IVFFLAT" -> PgVectorStore.PgIndexType.IVFFLAT;
            default -> PgVectorStore.PgIndexType.HNSW;
        };
    }
    
    private PgVectorStore.PgDistanceType parseDistanceType(String type) {
        return switch (type.toUpperCase()) {
            case "COSINE_DISTANCE" -> PgVectorStore.PgDistanceType.COSINE_DISTANCE;
            case "EUCLIDEAN_DISTANCE" -> PgVectorStore.PgDistanceType.EUCLIDEAN_DISTANCE;
            case "NEGATIVE_INNER_PRODUCT" -> PgVectorStore.PgDistanceType.NEGATIVE_INNER_PRODUCT;
            default -> PgVectorStore.PgDistanceType.COSINE_DISTANCE;
        };
    }
}
```

**OpenAIConfig.java**:

```java
package com.example.rag.config;

import org.springframework.ai.chat.ChatClient;
import org.springframework.ai.embedding.EmbeddingClient;
import org.springframework.ai.openai.OpenAiChatClient;
import org.springframework.ai.openai.OpenAiEmbeddingClient;
import org.springframework.ai.openai.api.OpenAiApi;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestClient;

@Configuration
public class OpenAIConfig {
    
    @Value("${spring.ai.openai.api-key}")
    private String apiKey;
    
    @Bean
    public OpenAiApi openAiApi() {
        return new OpenAiApi(apiKey);
    }
    
    @Bean
    public ChatClient chatClient(OpenAiApi openAiApi) {
        return new OpenAiChatClient(openAiApi);
    }
    
    @Bean
    public EmbeddingClient embeddingClient(OpenAiApi openAiApi) {
        return new OpenAiEmbeddingClient(openAiApi);
    }
}
```

---

## Building Your First RAG Application

### Domain Models

**QueryRequest.java**:

```java
package com.example.rag.model;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Data;

import java.util.Map;

@Data
public class QueryRequest {
    
    @NotBlank(message = "Query cannot be empty")
    @Size(max = 1000, message = "Query too long")
    private String query;
    
    private Integer topK = 5;
    
    private Double similarityThreshold = 0.7;
    
    private Map<String, Object> filters;
    
    private Boolean includeMetadata = true;
    
    private Boolean streamResponse = false;
}
```

**QueryResponse.java**:

```java
package com.example.rag.model;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;
import java.util.Map;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class QueryResponse {
    
    private String answer;
    
    private List<SourceDocument> sources;
    
    private Integer sourcesUsed;
    
    private Double confidenceScore;
    
    private Long processingTimeMs;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class SourceDocument {
        private String content;
        private Map<String, Object> metadata;
        private Double similarity;
    }
}
```

### RAG Service Implementation

**RagService.java**:

```java
package com.example.rag.service;

import com.example.rag.model.QueryRequest;
import com.example.rag.model.QueryResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.ChatClient;
import org.springframework.ai.chat.ChatResponse;
import org.springframework.ai.chat.prompt.Prompt;
import org.springframework.ai.chat.prompt.PromptTemplate;
import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.SearchRequest;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Slf4j
@Service
@RequiredArgsConstructor
public class RagService {
    
    private final VectorStore vectorStore;
    private final ChatClient chatClient;
    
    private static final String RAG_PROMPT_TEMPLATE = """
        Answer the question based solely on the following context.
        If you cannot answer the question based on the context, say 
        "I don't have enough information to answer this question."
        
        Context:
        {context}
        
        Question: {question}
        
        Provide a comprehensive answer with specific details from the context.
        If you reference specific information, mention which source it came from.
        
        Answer:
        """;
    
    public QueryResponse query(QueryRequest request) {
        long startTime = System.currentTimeMillis();
        
        try {
            // Step 1: Retrieve relevant documents
            log.debug("Searching for documents similar to: {}", request.getQuery());
            List<Document> documents = retrieveDocuments(request);
            
            if (documents.isEmpty()) {
                return buildNoResultsResponse(startTime);
            }
            
            // Step 2: Build context from documents
            String context = buildContext(documents);
            log.debug("Built context from {} documents", documents.size());
            
            // Step 3: Generate answer
            String answer = generateAnswer(request.getQuery(), context);
            
            // Step 4: Build response
            return buildResponse(answer, documents, startTime);
            
        } catch (Exception e) {
            log.error("Error processing query: {}", request.getQuery(), e);
            throw new RuntimeException("Failed to process query", e);
        }
    }
    
    private List<Document> retrieveDocuments(QueryRequest request) {
        SearchRequest.Builder searchBuilder = SearchRequest
            .query(request.getQuery())
            .withTopK(request.getTopK())
            .withSimilarityThreshold(request.getSimilarityThreshold());
        
        // Add metadata filters if provided
        if (request.getFilters() != null && !request.getFilters().isEmpty()) {
            request.getFilters().forEach((key, value) -> {
                searchBuilder.withFilterExpression(
                    org.springframework.ai.vectorstore.filter.Filter.Expression
                        .builder()
                        .key(key)
                        .eq(value)
                        .build()
                );
            });
        }
        
        return vectorStore.similaritySearch(searchBuilder.build());
    }
    
    private String buildContext(List<Document> documents) {
        return documents.stream()
            .map(doc -> {
                String source = (String) doc.getMetadata()
                    .getOrDefault("source", "Unknown");
                return String.format("[Source: %s]\n%s", source, doc.getContent());
            })
            .collect(Collectors.joining("\n\n---\n\n"));
    }
    
    private String generateAnswer(String question, String context) {
        PromptTemplate promptTemplate = new PromptTemplate(RAG_PROMPT_TEMPLATE);
        
        Prompt prompt = promptTemplate.create(Map.of(
            "context", context,
            "question", question
        ));
        
        ChatResponse response = chatClient.call(prompt);
        return response.getResult().getOutput().getContent();
    }
    
    private QueryResponse buildResponse(
            String answer, 
            List<Document> documents, 
            long startTime) {
        
        List<QueryResponse.SourceDocument> sources = documents.stream()
            .map(doc -> QueryResponse.SourceDocument.builder()
                .content(doc.getContent())
                .metadata(doc.getMetadata())
                .similarity(extractSimilarity(doc))
                .build())
            .collect(Collectors.toList());
        
        return QueryResponse.builder()
            .answer(answer)
            .sources(sources)
            .sourcesUsed(documents.size())
            .confidenceScore(calculateConfidence(documents))
            .processingTimeMs(System.currentTimeMillis() - startTime)
            .build();
    }
    
    private QueryResponse buildNoResultsResponse(long startTime) {
        return QueryResponse.builder()
            .answer("I don't have enough information to answer this question.")
            .sources(List.of())
            .sourcesUsed(0)
            .confidenceScore(0.0)
            .processingTimeMs(System.currentTimeMillis() - startTime)
            .build();
    }
    
    private Double extractSimilarity(Document doc) {
        // pgvector stores similarity in metadata
        Object score = doc.getMetadata().get("distance");
        if (score instanceof Number) {
            // Convert distance to similarity (1 - distance for cosine)
            return 1.0 - ((Number) score).doubleValue();
        }
        return null;
    }
    
    private Double calculateConfidence(List<Document> documents) {
        if (documents.isEmpty()) {
            return 0.0;
        }
        
        // Average similarity of top documents
        return documents.stream()
            .map(this::extractSimilarity)
            .filter(sim -> sim != null)
            .mapToDouble(Double::doubleValue)
            .average()
            .orElse(0.0);
    }
}
```

### REST Controller

**RagController.java**:

```java
package com.example.rag.controller;

import com.example.rag.model.QueryRequest;
import com.example.rag.model.QueryResponse;
import com.example.rag.service.RagService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@Slf4j
@RestController
@RequestMapping("/api/v1/rag")
@RequiredArgsConstructor
public class RagController {
    
    private final RagService ragService;
    
    @PostMapping("/query")
    public ResponseEntity<QueryResponse> query(
            @Valid @RequestBody QueryRequest request) {
        
        log.info("Received query: {}", request.getQuery());
        
        QueryResponse response = ragService.query(request);
        
        log.info("Query processed in {}ms with {} sources",
            response.getProcessingTimeMs(),
            response.getSourcesUsed());
        
        return ResponseEntity.ok(response);
    }
    
    @GetMapping("/health")
    public ResponseEntity<Map<String, String>> health() {
        return ResponseEntity.ok(Map.of(
            "status", "UP",
            "service", "RAG API"
        ));
    }
}
```

### Testing the Basic RAG System

**Start the application**:

```bash
# Set environment variable
export OPENAI_API_KEY=your-api-key-here

# Run with Maven
./mvnw spring-boot:run

# Or with Gradle
./gradlew bootRun
```

**Test the API**:

```bash
# Test query
curl -X POST http://localhost:8080/api/v1/rag/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are the main features of Spring Boot?",
    "topK": 5,
    "similarityThreshold": 0.7
  }'

# Response
{
  "answer": "Based on the provided context, Spring Boot...",
  "sources": [
    {
      "content": "Spring Boot makes it easy to create...",
      "metadata": {
        "source": "spring-boot-docs.pdf",
        "page": 1
      },
      "similarity": 0.89
    }
  ],
  "sourcesUsed": 3,
  "confidenceScore": 0.85,
  "processingTimeMs": 1250
}
```

---

## Document Processing Pipeline

### Document Chunking Strategy

**TextSplitter.java**:

```java
package com.example.rag.util;

import org.springframework.ai.document.Document;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;

@Component
public class TextSplitter {
    
    private static final int DEFAULT_CHUNK_SIZE = 1000;
    private static final int DEFAULT_CHUNK_OVERLAP = 200;
    
    /**
     * Split text into chunks with overlap for better context preservation
     */
    public List<Document> splitText(
            String text,
            Map<String, Object> metadata) {
        
        return splitText(
            text, 
            metadata, 
            DEFAULT_CHUNK_SIZE, 
            DEFAULT_CHUNK_OVERLAP
        );
    }
    
    public List<Document> splitText(
            String text,
            Map<String, Object> metadata,
            int chunkSize,
            int chunkOverlap) {
        
        List<Document> chunks = new ArrayList<>();
        String[] sentences = text.split("(?<=[.!?])\\s+");
        
        StringBuilder currentChunk = new StringBuilder();
        int chunkIndex = 0;
        
        for (String sentence : sentences) {
            if (currentChunk.length() + sentence.length() > chunkSize 
                    && currentChunk.length() > 0) {
                
                // Save current chunk
                Map<String, Object> chunkMetadata = new java.util.HashMap<>(metadata);
                chunkMetadata.put("chunk_index", chunkIndex);
                chunkMetadata.put("chunk_size", currentChunk.length());
                
                chunks.add(new Document(
                    currentChunk.toString().trim(),
                    chunkMetadata
                ));
                
                // Start new chunk with overlap
                String overlapText = getOverlapText(
                    currentChunk.toString(), 
                    chunkOverlap
                );
                currentChunk = new StringBuilder(overlapText);
                chunkIndex++;
            }
            
            currentChunk.append(sentence).append(" ");
        }
        
        // Add final chunk if not empty
        if (currentChunk.length() > 0) {
            Map<String, Object> chunkMetadata = new java.util.HashMap<>(metadata);
            chunkMetadata.put("chunk_index", chunkIndex);
            chunkMetadata.put("chunk_size", currentChunk.length());
            
            chunks.add(new Document(
                currentChunk.toString().trim(),
                chunkMetadata
            ));
        }
        
        return chunks;
    }
    
    private String getOverlapText(String text, int overlapSize) {
        if (text.length() <= overlapSize) {
            return text;
        }
        
        // Get last N characters and find sentence boundary
        String overlap = text.substring(text.length() - overlapSize);
        int sentenceStart = overlap.indexOf(". ");
        
        if (sentenceStart > 0) {
            return overlap.substring(sentenceStart + 2);
        }
        
        return overlap;
    }
    
    /**
     * Split by semantic units (paragraphs)
     */
    public List<Document> splitByParagraphs(
            String text,
            Map<String, Object> metadata) {
        
        List<Document> chunks = new ArrayList<>();
        String[] paragraphs = text.split("\n\n+");
        
        for (int i = 0; i < paragraphs.length; i++) {
            String paragraph = paragraphs[i].trim();
            
            if (!paragraph.isEmpty()) {
                Map<String, Object> chunkMetadata = new java.util.HashMap<>(metadata);
                chunkMetadata.put("chunk_index", i);
                chunkMetadata.put("chunk_type", "paragraph");
                
                chunks.add(new Document(paragraph, chunkMetadata));
            }
        }
        
        return chunks;
    }
}
```

### Document Ingestion Service

**DocumentService.java**:

```java
package com.example.rag.service;

import com.example.rag.util.TextSplitter;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Slf4j
@Service
@RequiredArgsConstructor
public class DocumentService {
    
    private final VectorStore vectorStore;
    private final TextSplitter textSplitter;
    
    /**
     * Ingest a single document
     */
    @Transactional
    public void ingestDocument(
            String content,
            Map<String, Object> metadata) {
        
        log.info("Ingesting document: {}", metadata.get("source"));
        
        // Split into chunks
        List<Document> chunks = textSplitter.splitText(content, metadata);
        log.debug("Split into {} chunks", chunks.size());
        
        // Store in vector database
        vectorStore.add(chunks);
        
        log.info("Successfully ingested {} chunks", chunks.size());
    }
    
    /**
     * Ingest multiple documents in batch
     */
    @Transactional
    public void ingestDocuments(List<Document> documents) {
        log.info("Batch ingesting {} documents", documents.size());
        
        List<Document> allChunks = documents.stream()
            .flatMap(doc -> textSplitter
                .splitText(doc.getContent(), doc.getMetadata())
                .stream())
            .toList();
        
        log.debug("Total chunks to ingest: {}", allChunks.size());
        
        // Batch insert for better performance
        int batchSize = 100;
        for (int i = 0; i < allChunks.size(); i += batchSize) {
            int end = Math.min(i + batchSize, allChunks.size());
            List<Document> batch = allChunks.subList(i, end);
            
            vectorStore.add(batch);
            log.debug("Ingested batch {}-{}", i, end);
        }
        
        log.info("Successfully ingested all documents");
    }
    
    /**
     * Ingest from file
     */
    public void ingestFromFile(Path filePath) throws IOException {
        String content = Files.readString(filePath);
        
        Map<String, Object> metadata = new HashMap<>();
        metadata.put("source", filePath.getFileName().toString());
        metadata.put("file_path", filePath.toString());
        metadata.put("file_size", Files.size(filePath));
        metadata.put("ingested_at", System.currentTimeMillis());
        
        ingestDocument(content, metadata);
    }
    
    /**
     * Delete documents by metadata filter
     */
    @Transactional
    public void deleteDocuments(String metadataKey, Object metadataValue) {
        log.info("Deleting documents where {}={}", metadataKey, metadataValue);
        
        // Note: Current Spring AI PgVectorStore doesn't have delete by filter
        // This would need custom JDBC implementation
        throw new UnsupportedOperationException(
            "Delete by filter not yet implemented"
        );
    }
}
```

### Bulk Document Ingestion

**DocumentIngestionController.java**:

```java
package com.example.rag.controller;

import com.example.rag.service.DocumentService;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.HashMap;
import java.util.Map;

@Slf4j
@RestController
@RequestMapping("/api/v1/documents")
@RequiredArgsConstructor
public class DocumentIngestionController {
    
    private final DocumentService documentService;
    
    @PostMapping("/ingest")
    public ResponseEntity<Map<String, Object>> ingestDocument(
            @RequestBody DocumentRequest request) {
        
        long startTime = System.currentTimeMillis();
        
        Map<String, Object> metadata = new HashMap<>();
        metadata.put("source", request.getSource());
        metadata.put("category", request.getCategory());
        metadata.putAll(request.getAdditionalMetadata());
        
        documentService.ingestDocument(request.getContent(), metadata);
        
        return ResponseEntity.ok(Map.of(
            "status", "success",
            "message", "Document ingested successfully",
            "processingTimeMs", System.currentTimeMillis() - startTime
        ));
    }
    
    @PostMapping("/ingest/file")
    public ResponseEntity<Map<String, Object>> ingestFile(
            @RequestParam("file") MultipartFile file,
            @RequestParam(required = false) String category) throws IOException {
        
        long startTime = System.currentTimeMillis();
        
        // Save uploaded file temporarily
        Path tempFile = Files.createTempFile("upload-", ".txt");
        file.transferTo(tempFile.toFile());
        
        try {
            String content = Files.readString(tempFile);
            
            Map<String, Object> metadata = new HashMap<>();
            metadata.put("source", file.getOriginalFilename());
            metadata.put("category", category != null ? category : "general");
            metadata.put("file_size", file.getSize());
            
            documentService.ingestDocument(content, metadata);
            
            return ResponseEntity.ok(Map.of(
                "status", "success",
                "filename", file.getOriginalFilename(),
                "size", file.getSize(),
                "processingTimeMs", System.currentTimeMillis() - startTime
            ));
            
        } finally {
            Files.deleteIfExists(tempFile);
        }
    }
    
    @Data
    public static class DocumentRequest {
        private String content;
        private String source;
        private String category;
        private Map<String, Object> additionalMetadata = new HashMap<>();
    }
}
```

---

## Advanced Querying Techniques

### Metadata Filtering

```java
@Service
public class AdvancedRagService {
    
    private final VectorStore vectorStore;
    
    /**
     * Search with complex metadata filters
     */
    public List<Document> searchWithFilters(
            String query,
            String category,
            Long afterTimestamp,
            List<String> tags) {
        
        // Build filter expression
        var filterBuilder = Filter.Expression.builder();
        
        if (category != null) {
            filterBuilder.and(
                Filter.Expression.builder()
                    .key("category")
                    .eq(category)
                    .build()
            );
        }
        
        if (afterTimestamp != null) {
            filterBuilder.and(
                Filter.Expression.builder()
                    .key("ingested_at")
                    .gte(afterTimestamp)
                    .build()
            );
        }
        
        if (tags != null && !tags.isEmpty()) {
            filterBuilder.and(
                Filter.Expression.builder()
                    .key("tags")
                    .in(tags.toArray())
                    .build()
            );
        }
        
        return vectorStore.similaritySearch(
            SearchRequest.query(query)
                .withTopK(10)
                .withSimilarityThreshold(0.7)
                .withFilterExpression(filterBuilder.build())
        );
    }
    
    /**
     * Search specific source documents
     */
    public List<Document> searchInSource(String query, String source) {
        return vectorStore.similaritySearch(
            SearchRequest.query(query)
                .withTopK(10)
                .withFilterExpression(
                    Filter.Expression.builder()
                        .key("source")
                        .eq(source)
                        .build()
                )
        );
    }
}
```

### Hybrid Search (Vector + Full-Text)

```java
@Repository
public class HybridSearchRepository {
    
    private final JdbcTemplate jdbcTemplate;
    
    /**
     * Combine vector similarity with full-text search
     */
    public List<Document> hybridSearch(
            float[] queryEmbedding,
            String searchTerms,
            int topK) {
        
        String sql = """
            WITH vector_results AS (
                SELECT 
                    id,
                    content,
                    metadata,
                    1 - (embedding <=> ?::vector) as vector_score
                FROM vector_store
                WHERE embedding <=> ?::vector < 0.5
                ORDER BY embedding <=> ?::vector
                LIMIT ?
            ),
            text_results AS (
                SELECT 
                    id,
                    content,
                    metadata,
                    ts_rank(
                        to_tsvector('english', content),
                        plainto_tsquery('english', ?)
                    ) as text_score
                FROM vector_store
                WHERE to_tsvector('english', content) @@ 
                      plainto_tsquery('english', ?)
                LIMIT ?
            )
            SELECT DISTINCT
                COALESCE(v.id, t.id) as id,
                COALESCE(v.content, t.content) as content,
                COALESCE(v.metadata, t.metadata) as metadata,
                COALESCE(v.vector_score, 0) * 0.7 + 
                COALESCE(t.text_score, 0) * 0.3 as combined_score
            FROM vector_results v
            FULL OUTER JOIN text_results t ON v.id = t.id
            ORDER BY combined_score DESC
            LIMIT ?
            """;
        
        String vectorString = toPostgresVector(queryEmbedding);
        
        return jdbcTemplate.query(
            sql,
            new Object[]{
                vectorString, vectorString, vectorString, topK,
                searchTerms, searchTerms, topK,
                topK
            },
            (rs, rowNum) -> {
                Document doc = new Document(rs.getString("content"));
                // Parse JSONB metadata
                // Add to doc.getMetadata()
                return doc;
            }
        );
    }
    
    private String toPostgresVector(float[] vector) {
        return "[" + String.join(",", 
            Arrays.stream(vector)
                .mapToObj(String::valueOf)
                .toArray(String[]::new)
        ) + "]";
    }
}
```

### Contextual Compression

```java
@Service
public class ContextCompressionService {
    
    private final ChatClient chatClient;
    
    /**
     * Compress retrieved documents to most relevant excerpts
     */
    public List<Document> compressContext(
            List<Document> documents,
            String query) {
        
        String compressionPrompt = """
            Given the following document and question, extract only 
            the most relevant sentences that help answer the question.
            Return ONLY the relevant sentences, maintaining their original wording.
            
            Document:
            {document}
            
            Question: {question}
            
            Relevant excerpts:
            """;
        
        return documents.stream()
            .map(doc -> {
                String compressed = chatClient.prompt()
                    .user(compressionPrompt
                        .replace("{document}", doc.getContent())
                        .replace("{question}", query))
                    .call()
                    .content();
                
                return new Document(compressed, doc.getMetadata());
            })
            .toList();
    }
}
```

### Re-ranking Retrieved Documents

```java
@Service
public class DocumentReranker {
    
    private final EmbeddingClient embeddingClient;
    
    /**
     * Re-rank documents using cross-encoder or reciprocal rank fusion
     */
    public List<Document> rerank(
            List<Document> documents,
            String query) {
        
        float[] queryEmbedding = embeddingClient.embed(query);
        
        // Calculate relevance scores
        return documents.stream()
            .map(doc -> {
                double score = calculateRelevanceScore(
                    queryEmbedding,
                    doc.getEmbedding(),
                    query,
                    doc.getContent()
                );
                
                doc.getMetadata().put("rerank_score", score);
                return doc;
            })
            .sorted((d1, d2) -> {
                double score1 = (double) d1.getMetadata().get("rerank_score");
                double score2 = (double) d2.getMetadata().get("rerank_score");
                return Double.compare(score2, score1);
            })
            .toList();
    }
    
    private double calculateRelevanceScore(
            float[] queryEmbedding,
            float[] docEmbedding,
            String query,
            String content) {
        
        // Cosine similarity
        double cosineSim = cosineSimilarity(queryEmbedding, docEmbedding);
        
        // Keyword overlap
        double keywordScore = calculateKeywordOverlap(query, content);
        
        // BM25-like score
        double bm25Score = calculateBM25(query, content);
        
        // Weighted combination
        return 0.5 * cosineSim + 0.3 * keywordScore + 0.2 * bm25Score;
    }
    
    private double cosineSimilarity(float[] a, float[] b) {
        double dotProduct = 0.0;
        double normA = 0.0;
        double normB = 0.0;
        
        for (int i = 0; i < a.length; i++) {
            dotProduct += a[i] * b[i];
            normA += a[i] * a[i];
            normB += b[i] * b[i];
        }
        
        return dotProduct / (Math.sqrt(normA) * Math.sqrt(normB));
    }
    
    private double calculateKeywordOverlap(String query, String content) {
        Set<String> queryTerms = new HashSet<>(
            Arrays.asList(query.toLowerCase().split("\\s+"))
        );
        Set<String> contentTerms = new HashSet<>(
            Arrays.asList(content.toLowerCase().split("\\s+"))
        );
        
        Set<String> intersection = new HashSet<>(queryTerms);
        intersection.retainAll(contentTerms);
        
        return (double) intersection.size() / queryTerms.size();
    }
    
    private double calculateBM25(String query, String content) {
        // Simplified BM25 implementation
        // In production, use a proper BM25 library
        double k1 = 1.5;
        double b = 0.75;
        double avgDocLength = 1000.0; // Should be calculated from corpus
        
        int docLength = content.split("\\s+").length;
        String[] queryTerms = query.toLowerCase().split("\\s+");
        
        double score = 0.0;
        for (String term : queryTerms) {
            int termFreq = countOccurrences(content.toLowerCase(), term);
            if (termFreq > 0) {
                double idf = Math.log((1.0 + termFreq) / termFreq);
                double numerator = termFreq * (k1 + 1);
                double denominator = termFreq + k1 * 
                    (1 - b + b * (docLength / avgDocLength));
                score += idf * (numerator / denominator);
            }
        }
        
        return score;
    }
    
    private int countOccurrences(String text, String term) {
        return (text.length() - text.replace(term, "").length()) / term.length();
    }
}
```

---

## Performance Optimization

### Index Tuning

```sql
-- Drop existing index
DROP INDEX IF EXISTS vector_store_embedding_idx;

-- Optimized HNSW index for your dataset
CREATE INDEX vector_store_embedding_idx 
ON vector_store 
USING hnsw (embedding vector_cosine_ops)
WITH (
    m = 16,              -- Number of connections (higher = better recall)
    ef_construction = 64 -- Build quality (higher = slower build, better quality)
);

-- For larger datasets (>1M vectors)
CREATE INDEX vector_store_embedding_idx 
ON vector_store 
USING hnsw (embedding vector_cosine_ops)
WITH (
    m = 32,
    ef_construction = 128
);

-- Set runtime search parameter for better recall
SET hnsw.ef_search = 100;  -- Higher = better recall, slower queries
```

### Connection Pool Optimization

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      pool-name: RAGHikariPool
      
      # PostgreSQL specific
      data-source-properties:
        cachePrepStmts: true
        prepStmtCacheSize: 250
        prepStmtCacheSqlLimit: 2048
        useServerPrepStmts: true
```

### Caching Strategy

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager(
            "embeddings", "queryResults"
        );
        cacheManager.setCaffeine(
            Caffeine.newBuilder()
                .maximumSize(1000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .recordStats()
        );
        return cacheManager;
    }
}

@Service
public class CachedEmbeddingService {
    
    private final EmbeddingClient embeddingClient;
    
    @Cacheable(value = "embeddings", key = "#text")
    public float[] embed(String text) {
        return embeddingClient.embed(text);
    }
}
```

### Batch Processing

```java
@Service
public class BatchDocumentProcessor {
    
    private final VectorStore vectorStore;
    private final EmbeddingClient embeddingClient;
    
    /**
     * Process documents in parallel batches
     */
    public void processBatch(List<Document> documents) {
        int batchSize = 50;
        int threads = Runtime.getRuntime().availableProcessors();
        
        ExecutorService executor = Executors.newFixedThreadPool(threads);
        List<CompletableFuture<Void>> futures = new ArrayList<>();
        
        for (int i = 0; i < documents.size(); i += batchSize) {
            int end = Math.min(i + batchSize, documents.size());
            List<Document> batch = documents.subList(i, end);
            
            CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                // Generate embeddings in parallel
                batch.forEach(doc -> {
                    if (doc.getEmbedding() == null) {
                        float[] embedding = embeddingClient.embed(doc.getContent());
                        doc.setEmbedding(embedding);
                    }
                });
                
                // Store batch
                vectorStore.add(batch);
            }, executor);
            
            futures.add(future);
        }
        
        // Wait for all batches to complete
        CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
        executor.shutdown();
    }
}
```

### Query Optimization

```java
@Service
public class OptimizedQueryService {
    
    /**
     * Optimize query by preprocessing
     */
    public List<Document> optimizedSearch(String query) {
        // 1. Clean and normalize query
        String normalized = normalizeQuery(query);
        
        // 2. Expand query with synonyms if needed
        String expanded = expandQuery(normalized);
        
        // 3. Use appropriate topK based on query type
        int topK = determineOptimalTopK(query);
        
        // 4. Adjust similarity threshold
        double threshold = calculateAdaptiveThreshold(query);
        
        return vectorStore.similaritySearch(
            SearchRequest.query(expanded)
                .withTopK(topK)
                .withSimilarityThreshold(threshold)
        );
    }
    
    private String normalizeQuery(String query) {
        return query.trim()
            .toLowerCase()
            .replaceAll("\\s+", " ");
    }
    
    private String expandQuery(String query) {
        // Add query expansion logic
        return query;
    }
    
    private int determineOptimalTopK(String query) {
        // Short queries might need more results
        return query.split("\\s+").length < 5 ? 10 : 5;
    }
    
    private double calculateAdaptiveThreshold(String query) {
        // More specific queries can have higher thresholds
        return query.split("\\s+").length > 10 ? 0.8 : 0.7;
    }
}
```

---

## Scaling Strategies

### Horizontal Read Scaling

```yaml
# application.yml with read replicas
spring:
  datasource:
    primary:
      url: jdbc:postgresql://primary:5432/ragdb
      username: raguser
      password: ${DB_PASSWORD}
    
    replica:
      url: jdbc:postgresql://replica:5432/ragdb
      username: raguser
      password: ${DB_PASSWORD}
      read-only: true
```

```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    @ConfigurationProperties("spring.datasource.replica")
    public DataSource replicaDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    public DataSource routingDataSource(
            @Qualifier("primaryDataSource") DataSource primary,
            @Qualifier("replicaDataSource") DataSource replica) {
        
        RoutingDataSource routing = new RoutingDataSource();
        
        Map<Object, Object> sources = new HashMap<>();
        sources.put("PRIMARY", primary);
        sources.put("REPLICA", replica);
        
        routing.setTargetDataSources(sources);
        routing.setDefaultTargetDataSource(primary);
        
        return routing;
    }
}

@Service
public class RoutedVectorService {
    
    @Transactional(readOnly = true)
    @ReadOnlyRoute  // Custom annotation to route to replica
    public List<Document> search(String query) {
        // Reads from replica
        return vectorStore.similaritySearch(
            SearchRequest.query(query).withTopK(10)
        );
    }
    
    @Transactional
    public void add(List<Document> documents) {
        // Writes to primary
        vectorStore.add(documents);
    }
}
```

### Partitioning Strategy

```sql
-- Partition by source/category for better performance
CREATE TABLE vector_store_partitioned (
    id UUID DEFAULT gen_random_uuid(),
    content TEXT NOT NULL,
    metadata JSONB DEFAULT '{}'::jsonb,
    embedding vector(1536),
    category TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
) PARTITION BY LIST (category);

-- Create partitions
CREATE TABLE vector_store_tech PARTITION OF vector_store_partitioned
    FOR VALUES IN ('technology', 'programming');

CREATE TABLE vector_store_business PARTITION OF vector_store_partitioned
    FOR VALUES IN ('business', 'finance');

CREATE TABLE vector_store_general PARTITION OF vector_store_partitioned
    DEFAULT;

-- Create indexes on each partition
CREATE INDEX ON vector_store_tech 
USING hnsw (embedding vector_cosine_ops);

CREATE INDEX ON vector_store_business 
USING hnsw (embedding vector_cosine_ops);

CREATE INDEX ON vector_store_general 
USING hnsw (embedding vector_cosine_ops);
```

### Archival Strategy

```java
@Service
@Scheduled(cron = "0 0 2 * * *")  // Run at 2 AM daily
public class DocumentArchivalService {
    
    private final JdbcTemplate jdbcTemplate;
    
    /**
     * Move old documents to archive table
     */
    public void archiveOldDocuments(int daysOld) {
        String archiveSql = """
            INSERT INTO vector_store_archive
            SELECT * FROM vector_store
            WHERE created_at < NOW() - INTERVAL '? days'
            AND metadata->>'archive_eligible' = 'true'
            """;
        
        jdbcTemplate.update(archiveSql, daysOld);
        
        String deleteSql = """
            DELETE FROM vector_store
            WHERE created_at < NOW() - INTERVAL '? days'
            AND metadata->>'archive_eligible' = 'true'
            """;
        
        int deleted = jdbcTemplate.update(deleteSql, daysOld);
        log.info("Archived {} documents", deleted);
    }
}
```

---

## Production Deployment

### Docker Deployment

**Dockerfile**:

```dockerfile
FROM eclipse-temurin:17-jre-alpine
LABEL maintainer="your-team@example.com"

WORKDIR /app

# Copy application jar
COPY target/rag-pgvector-demo-1.0.0.jar app.jar

# Create non-root user
RUN addgroup -g 1000 appgroup && \
    adduser -D -u 1000 -G appgroup appuser

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]
```

**docker-compose-prod.yml**:

```yaml
version: '3.8'

services:
  postgres:
    image: pgvector/pgvector:pg16
    restart: unless-stopped
    environment:
      POSTGRES_DB: ragdb
      POSTGRES_USER: raguser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U raguser"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - rag-network
  
  app:
    build: .
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/ragdb
      SPRING_DATASOURCE_USERNAME: raguser
      SPRING_DATASOURCE_PASSWORD_FILE: /run/secrets/db_password
      OPENAI_API_KEY_FILE: /run/secrets/openai_key
    secrets:
      - db_password
      - openai_key
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - rag-network
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

secrets:
  db_password:
    file: ./secrets/db_password.txt
  openai_key:
    file: ./secrets/openai_key.txt

networks:
  rag-network:
    driver: bridge

volumes:
  postgres_data:
```

### Kubernetes Deployment

**k8s/deployment.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rag-application
  labels:
    app: rag-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rag-app
  template:
    metadata:
      labels:
        app: rag-app
    spec:
      containers:
      - name: rag-app
        image: your-registry/rag-pgvector:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: SPRING_DATASOURCE_URL
          value: "jdbc:postgresql://postgres-service:5432/ragdb"
        - name: SPRING_DATASOURCE_USERNAME
          valueFrom:
            secretKeyRef:
              name: postgres-credentials
              key: username
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-credentials
              key: password
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: openai-credentials
              key: api-key
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: rag-service
spec:
  selector:
    app: rag-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

### Health Checks

```java
@Component
public class VectorStoreHealthIndicator implements HealthIndicator {
    
    private final VectorStore vectorStore;
    private final JdbcTemplate jdbcTemplate;
    
    @Override
    public Health health() {
        try {
            // Check database connectivity
            jdbcTemplate.queryForObject("SELECT 1", Integer.class);
            
            // Check vector extension
            String version = jdbcTemplate.queryForObject(
                "SELECT extversion FROM pg_extension WHERE extname = 'vector'",
                String.class
            );
            
            // Check index exists
            Integer indexCount = jdbcTemplate.queryForObject(
                """
                SELECT COUNT(*) FROM pg_indexes 
                WHERE tablename = 'vector_store' 
                AND indexdef LIKE '%hnsw%'
                """,
                Integer.class
            );
            
            if (indexCount > 0) {
                return Health.up()
                    .withDetail("database", "connected")
                    .withDetail("pgvector_version", version)
                    .withDetail("hnsw_index", "present")
                    .build();
            } else {
                return Health.down()
                    .withDetail("reason", "HNSW index missing")
                    .build();
            }
            
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

---

## Cost Analysis

### Infrastructure Costs

**AWS RDS PostgreSQL**:

```
Small Deployment (100K documents):
- Instance: db.t3.medium (2 vCPU, 4GB RAM)
- Storage: 100GB GP3
- Backup: 100GB

Monthly Cost:
- Instance: $60
- Storage: $10
- Backup: $10
- Data Transfer: $5
Total: ~$85/month

Annual: ~$1,020
```

**Medium Deployment (1M documents)**:

```
Medium Deployment (1M documents):
- Instance: db.r6g.xlarge (4 vCPU, 32GB RAM)
- Storage: 500GB GP3
- Backup: 500GB
- Read Replica: 1

Monthly Cost:
- Primary Instance: $180
- Read Replica: $180
- Storage: $50
- Backup: $50
- Data Transfer: $20
Total: ~$480/month

Annual: ~$5,760
```

### API Costs (OpenAI)

```
Embedding Costs (text-embedding-3-small):
- Price: $0.02 per 1M tokens
- Average document: 500 tokens
- Cost per document: $0.00001

100K documents: $1
1M documents: $10

LLM Costs (GPT-4-turbo):
- Price: $10 per 1M input tokens, $30 per 1M output tokens
- Average query: 2000 input tokens, 500 output tokens
- Cost per query: $0.035

1000 queries/day: $35/day = $1,050/month
10000 queries/day: $350/day = $10,500/month

Cost Optimization: Use GPT-3.5-turbo
- Price: $0.50/$1.50 per 1M tokens
- Cost per query: $0.0018
- 10000 queries/day: $18/day = $540/month

Savings: 95% reduction
```

### Total Cost of Ownership

**Comparison: pgvector vs Pinecone**

```
Scenario: 500K documents, 5K queries/day

pgvector Setup:
- RDS: $240/month
- OpenAI Embeddings (one-time): $5
- OpenAI GPT-3.5: $270/month
Total: $510/month
Annual: $6,120

Pinecone Setup:
- Pinecone (p1.x1 × 3): $210/month
- OpenAI Embeddings (one-time): $5
- OpenAI GPT-3.5: $270/month
Total: $480/month
Annual: $5,760

Difference: pgvector $30/month more

BUT pgvector includes:
✅ Your application database
✅ ACID transactions
✅ No vendor lock-in
✅ Full PostgreSQL features

Real savings when you consider:
- No need for separate application database
- PostgreSQL RDS handles both workloads
- Actual additional cost: ~$100/month for larger instance

Net savings: ~$110/month or $1,320/year
```

---

## Real-World Use Cases

### Customer Support Knowledge Base

```java
@Service
public class CustomerSupportRagService {
    
    private final VectorStore vectorStore;
    private final ChatClient chatClient;
    
    public SupportResponse answerCustomerQuery(
            String customerQuery,
            String customerId,
            String productCategory) {
        
        // Search relevant support articles
        List<Document> articles = vectorStore.similaritySearch(
            SearchRequest.query(customerQuery)
                .withTopK(5)
                .withFilterExpression(
                    Filter.Expression.and(
                        Filter.Expression.eq("type", "support_article"),
                        Filter.Expression.eq("category", productCategory),
                        Filter.Expression.eq("status", "published")
                    )
                )
        );
        
        // Search past resolved tickets
        List<Document> pastTickets = vectorStore.similaritySearch(
            SearchRequest.query(customerQuery)
                .withTopK(3)
                .withFilterExpression(
                    Filter.Expression.and(
                        Filter.Expression.eq("type", "resolved_ticket"),
                        Filter.Expression.eq("category", productCategory)
                    )
                )
        );
        
        // Build comprehensive context
        String context = buildSupportContext(articles, pastTickets);
        
        // Generate response with empathy
        String prompt = String.format("""
            You are a helpful customer support agent.
            
            Customer Question: %s
            
            Knowledge Base Context:
            %s
            
            Previous Similar Cases:
            %s
            
            Provide a clear, empathetic response that:
            1. Directly answers the question
            2. References specific steps if applicable
            3. Offers additional help if needed
            
            Response:
            """, customerQuery, articles, pastTickets);
        
        String answer = chatClient.prompt()
            .user(prompt)
            .call()
            .content();
        
        return new SupportResponse(
            answer,
            articles.stream()
                .map(d -> (String) d.getMetadata().get("article_id"))
                .collect(Collectors.toList())
        );
    }
}
```

### Document Q&A System

```java
@Service
public class DocumentQAService {
    
    /**
     * Answer questions about uploaded documents
     */
    public DocumentAnswer queryDocument(
            String documentId,
            String question) {
        
        // Retrieve document chunks
        List<Document> chunks = vectorStore.similaritySearch(
            SearchRequest.query(question)
                .withTopK(10)
                .withFilterExpression(
                    Filter.Expression.eq("document_id", documentId)
                )
        );
        
        if (chunks.isEmpty()) {
            return DocumentAnswer.notFound(
                "No relevant information found in this document"
            );
        }
        
        // Extract answer with citations
        String prompt = String.format("""
            Answer the question based on the following document excerpts.
            Include page numbers or section references when citing information.
            
            Document Excerpts:
            %s
            
            Question: %s
            
            Answer (with citations):
            """,
            buildCitableContext(chunks),
            question
        );
        
        String answer = chatClient.prompt()
            .user(prompt)
            .call()
            .content();
        
        return DocumentAnswer.builder()
            .answer(answer)
            .documentId(documentId)
            .relevantPages(extractPages(chunks))
            .confidence(calculateConfidence(chunks))
            .build();
    }
    
    private String buildCitableContext(List<Document> chunks) {
        return chunks.stream()
            .map(chunk -> {
                int page = (int) chunk.getMetadata()
                    .getOrDefault("page", 0);
                return String.format("[Page %d] %s", page, chunk.getContent());
            })
            .collect(Collectors.joining("\n\n"));
    }
}
```

### Legal Document Search

```java
@Service
public class LegalDocumentSearchService {
    
    /**
     * Search legal documents with precision
     */
    public LegalSearchResult searchCaselaw(
            String legalQuery,
            String jurisdiction,
            Integer fromYear) {
        
        // Expand query with legal terms
        String expandedQuery = expandLegalQuery(legalQuery);
        
        // Search with strict filters
        List<Document> cases = vectorStore.similaritySearch(
            SearchRequest.query(expandedQuery)
                .withTopK(20)
                .withSimilarityThreshold(0.85)  // Higher threshold
                .withFilterExpression(
                    Filter.Expression.and(
                        Filter.Expression.eq("jurisdiction", jurisdiction),
                        Filter.Expression.gte("year", fromYear),
                        Filter.Expression.eq("document_type", "case")
                    )
                )
        );
        
        // Re-rank by legal relevance
        List<Document> reranked = rerankByLegalRelevance(
            cases, 
            legalQuery
        );
        
        return LegalSearchResult.builder()
            .cases(reranked.stream()
                .map(this::toLegalCitation)
                .collect(Collectors.toList()))
            .query(legalQuery)
            .jurisdiction(jurisdiction)
            .build();
    }
    
    private String expandLegalQuery(String query) {
        // Add legal synonyms and related terms
        Map<String, List<String>> legalSynonyms = Map.of(
            "contract", List.of("agreement", "covenant"),
            "negligence", List.of("tort", "breach of duty"),
            "damages", List.of("compensation", "remedy")
        );
        
        // Implementation details...
        return query;
    }
}
```

---

## Troubleshooting Guide

### Common Issues and Solutions

**Issue 1: Slow Query Performance**

```java
// Problem: Queries taking >1 second
// Diagnosis
@Service
public class PerformanceDiagnostics {
    
    public void diagnoseSlowQueries() {
        // Check index usage
        String sql = """
            SELECT 
                schemaname,
                tablename,
                indexname,
                idx_scan as index_scans,
                idx_tup_read as tuples_read,
                idx_tup_fetch as tuples_fetched
            FROM pg_stat_user_indexes
            WHERE tablename = 'vector_store'
            ORDER BY idx_scan DESC;
            """;
        
        // Check table statistics
        String statsSql = """
            SELECT 
                n_live_tup as live_tuples,
                n_dead_tup as dead_tuples,
                last_vacuum,
                last_autovacuum,
                last_analyze
            FROM pg_stat_user_tables
            WHERE tablename = 'vector_store';
            """;
        
        // Solution: Run VACUUM and ANALYZE
        jdbcTemplate.execute("VACUUM ANALYZE vector_store;");
        
        // Rebuild index if necessary
        jdbcTemplate.execute("""
            REINDEX INDEX CONCURRENTLY vector_store_embedding_idx;
            """);
    }
}
```

**Issue 2: Out of Memory**

```yaml
# Tune PostgreSQL memory settings
# postgresql.conf

shared_buffers = 4GB           # 25% of RAM
effective_cache_size = 12GB    # 75% of RAM
work_mem = 64MB               # Per query operation
maintenance_work_mem = 1GB    # For VACUUM, CREATE INDEX
max_connections = 100         # Limit connections

# Monitor memory usage
SELECT * FROM pg_stat_database WHERE datname = 'ragdb';
```

**Issue 3: Low Recall**

```java
// Problem: Not finding relevant documents
// Solutions:

// 1. Lower similarity threshold
SearchRequest.query(query)
    .withSimilarityThreshold(0.6);  // From 0.7

// 2. Increase topK
SearchRequest.query(query)
    .withTopK(20);  // From 10

// 3. Improve chunking strategy
// Use smaller chunks with more overlap
textSplitter.splitText(text, metadata, 800, 200);

// 4. Use query expansion
String expandedQuery = expandQueryWithSynonyms(query);
```

**Issue 4: High Costs**

```java
// Reduce OpenAI API costs

// 1. Cache embeddings
@Cacheable("embeddings")
public float[] embed(String text) {
    return embeddingClient.embed(text);
}

// 2. Use cheaper model
spring:
  ai:
    openai:
      chat:
        options:
          model: gpt-3.5-turbo  # Instead of GPT-4

// 3. Implement request throttling
@RateLimiter(name = "openai", fallbackMethod = "fallbackResponse")
public String query(String question) {
    return ragService.query(question);
}

// 4. Batch embed documents
List<float[]> embeddings = embeddingClient.embedBatch(texts);
```

---

## Best Practices

### Security Best Practices

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) 
            throws Exception {
        
        return http
            .csrf(csrf -> csrf
                .ignoringRequestMatchers("/api/v1/documents/**")
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/rag/query").authenticated()
                .requestMatchers("/api/v1/documents/**").hasRole("ADMIN")
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(withDefaults()))
            .build();
    }
    
    // Row-level security in PostgreSQL
    @PostConstruct
    public void enableRowLevelSecurity() {
        jdbcTemplate.execute("""
            ALTER TABLE vector_store ENABLE ROW LEVEL SECURITY;
            
            CREATE POLICY user_isolation ON vector_store
                USING (metadata->>'user_id' = current_user);
            """);
    }
}
```

### Data Quality Best Practices

```java
@Service
public class DataQualityService {
    
    /**
     * Validate documents before ingestion
     */
    public void validateDocument(Document doc) {
        // Check content length
        if (doc.getContent().length() < 50) {
            throw new ValidationException("Content too short");
        }
        
        // Check for required metadata
        if (!doc.getMetadata().containsKey("source")) {
            throw new ValidationException("Missing source metadata");
        }
        
        // Detect language
        String language = detectLanguage(doc.getContent());
        if (!language.equals("en")) {
            doc.getMetadata().put("language", language);
        }
        
        // Remove sensitive information
        String cleaned = removeSensitiveInfo(doc.getContent());
        doc.setContent(cleaned);
    }
    
    private String removeSensitiveInfo(String content) {
        // Remove email addresses
        content = content.replaceAll(
            "\\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Z|a-z]{2,}\\b",
            "[EMAIL]"
        );
        
        // Remove phone numbers
        content = content.replaceAll(
            "\\b\\d{3}[-.]?\\d{3}[-.]?\\d{4}\\b",
            "[PHONE]"
        );
        
        // Remove SSNs
        content = content.replaceAll(
            "\\b\\d{3}-\\d{2}-\\d{4}\\b",
            "[SSN]"
        );
        
        return content;
    }
}
```

### Monitoring Best Practices

```java
@Component
public class RagMetrics {
    
    private final MeterRegistry registry;
    
    public void recordQuery(QueryResponse response) {
        // Record query latency
        registry.timer("rag.query.duration")
            .record(response.getProcessingTimeMs(), TimeUnit.MILLISECONDS);
        
        // Record sources used
        registry.counter("rag.query.sources",
            "count", String.valueOf(response.getSourcesUsed())
        ).increment();
        
        // Record confidence
        registry.gauge("rag.query.confidence",
            response.getConfidenceScore()
        );
    }
    
    public void recordIngestion(int documentCount, long durationMs) {
        registry.counter("rag.documents.ingested").increment(documentCount);
        registry.timer("rag.ingestion.duration")
            .record(durationMs, TimeUnit.MILLISECONDS);
    }
}
```

---

## Conclusion

Building a RAG system with Spring AI and PostgreSQL pgvector offers an compelling combination of simplicity, cost-effectiveness, and reliability. Throughout this guide, we've covered:

✅ **Setup and Configuration**: From Docker to production Kubernetes  
✅ **Core Implementation**: Building a functional RAG pipeline  
✅ **Advanced Techniques**: Hybrid search, re-ranking, compression  
✅ **Performance Optimization**: Indexing, caching, batch processing  
✅ **Production Deployment**: Docker, Kubernetes, monitoring  
✅ **Cost Analysis**: Detailed TCO comparisons  
✅ **Real-World Use Cases**: Customer support, legal search, document Q&A  

### Key Takeaways

**When pgvector is Perfect**:
- Small to medium datasets (<1M documents)
- Already using PostgreSQL
- Need ACID transactions with vectors
- Budget-conscious projects
- Want to avoid vendor lock-in

**Performance Reality**:
- 30-80ms query latency (p95)
- 200-500 QPS throughput
- 95-98% recall
- More than sufficient for most applications

**Cost Benefits**:
- Leverage existing infrastructure
- No additional database licensing
- $100-200/month additional costs
- Savings of $1,000-$5,000/year vs specialized solutions

### Next Steps

1. **Start Small**: Deploy the basic RAG system locally
2. **Load Real Data**: Test with your actual documents
3. **Measure Performance**: Benchmark against your requirements
4. **Optimize**: Tune indexes and queries
5. **Deploy to Production**: Use the provided deployment guides
6. **Monitor and Iterate**: Track metrics and improve

### Additional Resources

- [Spring AI Documentation](https://docs.spring.io/spring-ai/reference/)
- [pgvector GitHub](https://github.com/pgvector/pgvector)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)
- [PostgreSQL Performance Tuning](https://www.postgresql.org/docs/current/performance-tips.html)

**Remember**: The best RAG system is one that ships. Start with pgvector, validate your concept, and scale when needed. You can always migrate to specialized vector databases later if requirements demand it.

Happy building! 🚀