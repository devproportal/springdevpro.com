基于下面的信息，给出英文技术博客文章（面向欧美用户，基于 Google Adsense赚钱）：
文章为主，代码为辅。
要有图表和表格。

Reference Title: What is RAG? A Complete Guide for Spring Developers
Reference Keywords: spring ai rag
Target Word Count: 7470


# What is RAG? A Complete Guide for Spring Developers

**Master Retrieval-Augmented Generation to Build Intelligent, Context-Aware AI Applications with Spring AI**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Understanding RAG Fundamentals](#understanding-rag-fundamentals)
3. [Why RAG Matters](#why-rag-matters)
4. [RAG Architecture Overview](#rag-architecture-overview)
5. [Setting Up RAG in Spring AI](#setting-up-rag-in-spring-ai)
6. [Document Processing Pipeline](#document-processing-pipeline)
7. [Vector Embeddings Explained](#vector-embeddings-explained)
8. [Vector Stores and Databases](#vector-stores-and-databases)
9. [Implementing Similarity Search](#implementing-similarity-search)
10. [Building Your First RAG Application](#building-your-first-rag-application)
11. [Advanced RAG Patterns](#advanced-rag-patterns)
12. [Optimizing RAG Performance](#optimizing-rag-performance)
13. [RAG Use Cases and Examples](#rag-use-cases-and-examples)
14. [Challenges and Solutions](#challenges-and-solutions)
15. [Production Considerations](#production-considerations)
16. [Best Practices](#best-practices)
17. [Conclusion](#conclusion)

---

## Introduction

Imagine asking an AI assistant about your company's latest product updates, internal policies, or customer data. Standard language models can't answer these questions because they weren't trained on your specific data. This is where Retrieval-Augmented Generation (RAG) transforms the game.

RAG bridges the gap between general AI capabilities and your specific knowledge base. Instead of retraining expensive models, RAG dynamically retrieves relevant information and augments AI responses with your data in real-time.

### What You'll Learn

This comprehensive guide covers everything Spring developers need to know about RAG:

✅ **Core Concepts**: Understanding how RAG works under the hood  
✅ **Spring AI Integration**: Implementing RAG with Spring Boot  
✅ **Vector Databases**: Choosing and configuring vector stores  
✅ **Document Processing**: Loading, chunking, and embedding documents  
✅ **Production Patterns**: Scaling RAG for real-world applications  
✅ **Best Practices**: Optimizing accuracy, performance, and cost  

### Prerequisites

To get the most from this guide, you should have:

- Basic Spring Boot knowledge
- Understanding of REST APIs
- Familiarity with Spring AI basics
- Java 17 or higher installed
- An OpenAI API key (or alternative LLM provider)

Let's dive into the fascinating world of RAG!

---

## Understanding RAG Fundamentals

### What is RAG?

**Retrieval-Augmented Generation (RAG)** is a technique that enhances language model responses by retrieving relevant information from external knowledge sources before generating an answer.

**Think of it like this**:

```
Traditional LLM:
User: "What's our Q4 revenue?"
LLM: "I don't have access to your company's financial data."

With RAG:
User: "What's our Q4 revenue?"
System: [Retrieves: "Q4 2024 Revenue: $2.3M, up 15% from Q3"]
LLM: "Your Q4 2024 revenue was $2.3 million, representing a 15% 
      increase from Q3."
```

### The RAG Process

RAG works in two main phases:

**1. Indexing Phase (One-time setup)**:
```
Documents → Split into chunks → Generate embeddings → Store in vector DB
```

**2. Query Phase (Every user request)**:
```
User question → Generate embedding → Search vector DB → 
Retrieve relevant chunks → Augment prompt → Generate answer
```

### Why Not Just Fine-tune?

You might wonder: "Why not just fine-tune the model on my data?" Here's the comparison:

**Fine-tuning**:
- ❌ Expensive ($thousands per training run)
- ❌ Time-consuming (hours/days)
- ❌ Static knowledge (outdated immediately)
- ❌ Requires ML expertise
- ❌ Hard to update or correct

**RAG**:
- ✅ Cost-effective (pennies per query)
- ✅ Instant deployment
- ✅ Dynamic knowledge (update anytime)
- ✅ No ML expertise needed
- ✅ Easy to update and audit

### Key Components

Every RAG system has four essential components:

**1. Document Loader**:
- Reads documents from various sources
- Supports PDFs, Word docs, web pages, databases
- Example: `PagePdfDocumentReader`, `TextReader`

**2. Text Splitter**:
- Breaks documents into manageable chunks
- Maintains context across splits
- Example: `TokenTextSplitter`, `CharacterTextSplitter`

**3. Embedding Model**:
- Converts text into numerical vectors
- Captures semantic meaning
- Example: `OpenAiEmbeddingClient`, `OllamaEmbeddingClient`

**4. Vector Store**:
- Stores and indexes embeddings
- Enables fast similarity search
- Example: `PgVectorStore`, `ChromaVectorStore`

### Simple RAG Example

Here's a minimal RAG implementation to illustrate the concept:

```java
// 1. Load and process documents
List<Document> documents = new TextReader("knowledge-base.txt").get();

// 2. Generate embeddings and store
vectorStore.add(documents);

// 3. Query with RAG
String userQuestion = "What is our return policy?";

// 4. Retrieve relevant context
List<Document> relevantDocs = vectorStore.similaritySearch(
    SearchRequest.query(userQuestion).withTopK(3)
);

// 5. Augment prompt with context
String context = relevantDocs.stream()
    .map(Document::getContent)
    .collect(Collectors.joining("\n\n"));

String prompt = String.format("""
    Answer the question based on the following context:
    
    Context:
    %s
    
    Question: %s
    
    Answer:
    """, context, userQuestion);

// 6. Generate answer
String answer = chatClient.prompt()
    .user(prompt)
    .call()
    .content();
```

This simple flow demonstrates RAG's power: the model answers questions about your specific data without any training!

---

## Why RAG Matters

### The Knowledge Problem

Modern language models are incredibly capable, but they face a fundamental limitation: their knowledge is frozen at training time. If GPT-4 was trained on data up to April 2023, it knows nothing about events after that date.

**Real-world consequences**:

```java
// Without RAG
User: "What's the status of project Phoenix?"
AI: "I don't have information about project Phoenix."

// With RAG
User: "What's the status of project Phoenix?"
AI: "Project Phoenix is 85% complete. The team finished the backend 
     migration last week and is now working on the frontend redesign. 
     Expected completion: December 15th."
```

### Business Value

RAG delivers tangible business benefits:

**1. Instant Knowledge Integration**:
- Deploy new information in minutes, not months
- No retraining required
- Update knowledge base in real-time

**2. Cost Efficiency**:
```
Fine-tuning GPT-4:
- Training: $5,000 - $50,000
- Updates: Retrain from scratch
- Total: $50,000+ annually

RAG with GPT-4:
- Setup: < $100
- Updates: Free (just add documents)
- Queries: $0.01 - $0.10 each
- Total: $1,000 - $5,000 annually
```

**3. Accuracy and Trust**:
- Cite sources for every answer
- Verify information provenance
- Update incorrect information immediately

**4. Privacy and Control**:
- Keep sensitive data in your infrastructure
- Control what information the AI can access
- Comply with data regulations (GDPR, HIPAA)

### Use Case Categories

RAG excels in several domains:

**Internal Knowledge Management**:
- Employee onboarding assistants
- Policy and procedure lookup
- Technical documentation Q&A
- Meeting notes and decision tracking

**Customer Support**:
- Product documentation assistance
- Troubleshooting guides
- FAQ automation
- Ticket routing and categorization

**Data Analysis**:
- Report summarization
- Trend identification
- Insight extraction from logs
- Compliance monitoring

**Content Generation**:
- Personalized email responses
- Document drafting with company context
- Social media content aligned with brand voice
- Marketing copy from product specs

### Real-World Impact

Consider this customer support scenario:

**Without RAG**:
```
Customer: "My device shows error code E447. What does this mean?"
Support: [Searches through 50 pages of documentation]
Support: [After 15 minutes] "Error E447 indicates a sensor malfunction.
         Try resetting the device."
Resolution time: 20 minutes
```

**With RAG**:
```
Customer: "My device shows error code E447. What does this mean?"
System: [Instantly retrieves relevant troubleshooting steps]
AI: "Error E447 indicates a temperature sensor malfunction. Here's 
     how to resolve it:
     1. Power off the device
     2. Wait 30 seconds
     3. Hold the reset button for 5 seconds while powering on
     4. If the error persists, contact support for replacement
     
     This solution resolves E447 in 94% of cases."
Resolution time: 30 seconds
```

The impact is dramatic: **40x faster resolution** with more accurate, consistent answers.

---

## RAG Architecture Overview

### High-Level Architecture

Here's how RAG components work together:

```
┌─────────────────────────────────────────────────────────────┐
│                     RAG SYSTEM ARCHITECTURE                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐         ┌──────────────┐                 │
│  │   Documents  │────────▶│Document Loader│                │
│  │ (PDF, TXT,   │         │   & Reader    │                │
│  │  JSON, etc.) │         └───────┬───────┘                │
│  └──────────────┘                 │                         │
│                                    ▼                         │
│                          ┌─────────────────┐                │
│                          │  Text Splitter  │                │
│                          │   (Chunking)    │                │
│                          └────────┬────────┘                │
│                                   │                          │
│                                   ▼                          │
│                          ┌─────────────────┐                │
│                          │Embedding Client │                │
│                          │ (Text→Vectors)  │                │
│                          └────────┬────────┘                │
│                                   │                          │
│                                   ▼                          │
│  ┌─────────────────────────────────────────────────┐       │
│  │           VECTOR DATABASE                        │       │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │       │
│  │  │Document 1│  │Document 2│  │Document N│      │       │
│  │  │[Vector]  │  │[Vector]  │  │[Vector]  │      │       │
│  │  └──────────┘  └──────────┘  └──────────┘      │       │
│  └─────────────────────────────────────────────────┘       │
│                                                              │
│  ═══════════════ QUERY TIME ════════════════════           │
│                                                              │
│  ┌──────────────┐                                           │
│  │ User Query   │                                           │
│  └──────┬───────┘                                           │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐         ┌──────────────┐                │
│  │  Embedding   │────────▶│Vector Search │                │
│  │    Client    │         │ (Similarity) │                │
│  └──────────────┘         └──────┬───────┘                │
│                                   │                          │
│                                   ▼                          │
│                          ┌────────────────┐                 │
│                          │   Retrieved    │                 │
│                          │   Documents    │                 │
│                          └────────┬───────┘                 │
│                                   │                          │
│                                   ▼                          │
│  ┌────────────────────────────────────────────┐            │
│  │         Prompt Construction                 │            │
│  │  Context + Question → Complete Prompt      │            │
│  └─────────────────────┬──────────────────────┘            │
│                        │                                     │
│                        ▼                                     │
│               ┌─────────────────┐                           │
│               │   LLM (GPT-4)   │                           │
│               │   Generation    │                           │
│               └────────┬────────┘                           │
│                        │                                     │
│                        ▼                                     │
│               ┌─────────────────┐                           │
│               │     Answer      │                           │
│               └─────────────────┘                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

**Indexing Phase**:

1. **Document Ingestion**: Load documents from various sources
2. **Text Splitting**: Break into overlapping chunks (500-1000 tokens)
3. **Embedding Generation**: Convert chunks to vector representations
4. **Storage**: Index vectors in database for fast retrieval

**Query Phase**:

1. **Query Embedding**: Convert user question to vector
2. **Similarity Search**: Find most relevant document chunks
3. **Context Assembly**: Combine retrieved chunks
4. **Prompt Augmentation**: Add context to user question
5. **Generation**: LLM produces answer based on context
6. **Response**: Return answer to user

### Component Interaction

Let's trace a real query through the system:

```java
// User asks a question
String question = "What are the main features of Spring AI?";

// 1. EMBEDDING: Convert question to vector
float[] questionEmbedding = embeddingClient.embed(question);
// Result: [0.123, -0.456, 0.789, ..., 0.234] (1536 dimensions)

// 2. SEARCH: Find similar documents in vector store
SearchRequest searchRequest = SearchRequest.query(question)
    .withTopK(5)  // Get 5 most relevant chunks
    .withSimilarityThreshold(0.7);  // Minimum 70% similarity

List<Document> relevantDocs = vectorStore.similaritySearch(searchRequest);

// Retrieved documents might be:
// Doc 1: "Spring AI provides ChatClient for LLM integration..." (0.92 similarity)
// Doc 2: "Key features include prompt templates, streaming..." (0.88 similarity)
// Doc 3: "Vector stores enable RAG implementations..." (0.85 similarity)
// Doc 4: "Embedding clients support multiple providers..." (0.78 similarity)
// Doc 5: "Function calling allows tool integration..." (0.75 similarity)

// 3. CONTEXT ASSEMBLY: Combine relevant content
String context = relevantDocs.stream()
    .map(doc -> doc.getContent())
    .collect(Collectors.joining("\n\n---\n\n"));

// 4. PROMPT CONSTRUCTION: Create augmented prompt
String augmentedPrompt = String.format("""
    You are a helpful assistant. Answer the question based on the 
    following context. If you cannot answer from the context, say so.
    
    CONTEXT:
    %s
    
    QUESTION: %s
    
    ANSWER:
    """, context, question);

// 5. GENERATION: Get LLM response
String answer = chatClient.prompt()
    .user(augmentedPrompt)
    .call()
    .content();

// 6. RESPONSE: Return to user
System.out.println(answer);
```

### Architectural Patterns

**Pattern 1: Simple RAG** (What we've seen so far):
```
Query → Retrieve → Augment → Generate
```

**Pattern 2: Conversational RAG**:
```
Query + Chat History → Retrieve → Augment → Generate → Update History
```

**Pattern 3: Multi-Index RAG**:
```
Query → Route to appropriate index → Retrieve → Augment → Generate
```

**Pattern 4: Hierarchical RAG**:
```
Query → Broad search → Narrow search → Retrieve → Generate
```

**Pattern 5: Hybrid Search RAG**:
```
Query → Vector Search + Keyword Search → Merge results → Generate
```

### Spring AI Components Mapping

Here's how Spring AI components map to RAG architecture:

```java
// Document Loading
DocumentReader → Loads documents from sources
TextReader, PagePdfDocumentReader, JsonReader

// Text Processing
TextSplitter → Breaks documents into chunks
TokenTextSplitter, CharacterTextSplitter

// Embeddings
EmbeddingClient → Converts text to vectors
OpenAiEmbeddingClient, OllamaEmbeddingClient, TransformersEmbeddingClient

// Vector Storage
VectorStore → Stores and searches embeddings
PgVectorStore, ChromaVectorStore, SimpleVectorStore

// Generation
ChatClient → Generates responses
OpenAiChatClient, OllamaChatClient, AnthropicChatClient

// Orchestration
RAG Advisors → Coordinates retrieval and generation
QuestionAnswerAdvisor, VectorStoreRetriever
```

---

## Setting Up RAG in Spring AI

### Project Dependencies

Create a new Spring Boot project and add these dependencies:

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
        <version>3.3.5</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>spring-ai-rag</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <java.version>17</java.version>
        <spring-ai.version>1.0.0-M4</spring-ai.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Spring AI OpenAI -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
        </dependency>
        
        <!-- Spring AI PDF Reader -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-pdf-document-reader</artifactId>
        </dependency>
        
        <!-- Spring AI PGVector Store -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
        </dependency>
        
        <!-- PostgreSQL Driver -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
        
        <!-- Spring Data JPA (for PGVector) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- Lombok (optional, for cleaner code) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.ai</groupId>
                <artifactId>spring-ai-bom</artifactId>
                <version>${spring-ai.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
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
    name: spring-ai-rag-demo
    
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4o-mini
          temperature: 0.7
      embedding:
        options:
          model: text-embedding-3-small
          
  datasource:
    url: jdbc:postgresql://localhost:5432/vectordb
    username: postgres
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver
    
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        
  ai:
    vectorstore:
      pgvector:
        index-type: HNSW
        distance-type: COSINE_DISTANCE
        dimensions: 1536  # For OpenAI text-embedding-3-small

logging:
  level:
    org.springframework.ai: DEBUG
    com.example.rag: DEBUG
```

### Database Setup

Start PostgreSQL with pgvector extension using Docker:

```bash
docker run -d \
  --name pgvector \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=vectordb \
  -p 5432:5432 \
  pgvector/pgvector:pg16
```

Connect and enable the extension:

```sql
-- Connect to your database
\c vectordb

-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Verify installation
SELECT * FROM pg_extension WHERE extname = 'vector';
```

### Basic Configuration Class

```java
package com.example.rag.config;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.embedding.EmbeddingClient;
import org.springframework.ai.vectorstore.PgVectorStore;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;

@Configuration
public class RagConfiguration {
    
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder) {
        return builder.build();
    }
    
    @Bean
    public VectorStore vectorStore(
            EmbeddingClient embeddingClient,
            JdbcTemplate jdbcTemplate) {
        
        return new PgVectorStore(jdbcTemplate, embeddingClient);
    }
}
```

### Environment Variables

Create a `.env` file (don't commit to Git!):

```bash
# OpenAI API Key
OPENAI_API_KEY=sk-your-api-key-here

# Database Password
DB_PASSWORD=postgres

# Optional: Adjust embedding model
EMBEDDING_MODEL=text-embedding-3-small

# Optional: Adjust chat model
CHAT_MODEL=gpt-4o-mini
```

### Verification Test

Create a simple test to verify setup:

```java
package com.example.rag;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.embedding.EmbeddingClient;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.List;

@SpringBootApplication
public class RagApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(RagApplication.class, args);
    }
    
    @Bean
    CommandLineRunner testSetup(
            ChatClient chatClient,
            EmbeddingClient embeddingClient,
            VectorStore vectorStore) {
        
        return args -> {
            System.out.println("Testing RAG setup...");
            
            // Test 1: Chat client
            String response = chatClient.prompt()
                .user("Say 'Setup successful' if you can read this")
                .call()
                .content();
            System.out.println("✓ Chat Client: " + response);
            
            // Test 2: Embedding client
            List<Double> embedding = embeddingClient.embed("test");
            System.out.println("✓ Embedding Client: Generated " + 
                embedding.size() + " dimensions");
            
            // Test 3: Vector store
            System.out.println("✓ Vector Store: Ready");
            
            System.out.println("\n✅ All components initialized successfully!");
        };
    }
}
```

Run the application:

```bash
mvn spring-boot:run
```

Expected output:

```
Testing RAG setup...
✓ Chat Client: Setup successful
✓ Embedding Client: Generated 1536 dimensions
✓ Vector Store: Ready

✅ All components initialized successfully!
```

---

## Document Processing Pipeline

### Document Loaders

Spring AI supports various document formats. Let's explore each:

**Text Files**:

```java
@Service
public class DocumentLoaderService {
    
    public List<Document> loadTextFile(String filePath) {
        TextReader textReader = new TextReader(filePath);
        return textReader.get();
    }
    
    public List<Document> loadTextWithMetadata(String filePath) {
        TextReader textReader = new TextReader(filePath);
        textReader.getCustomMetadata().put("source", "internal-docs");
        textReader.getCustomMetadata().put("department", "engineering");
        textReader.getCustomMetadata().put("version", "1.0");
        
        return textReader.get();
    }
}
```

**PDF Documents**:

```java
import org.springframework.ai.reader.pdf.PagePdfDocumentReader;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;

@Service
public class PdfLoaderService {
    
    public List<Document> loadPdf(String resourcePath) {
        Resource resource = new ClassPathResource(resourcePath);
        
        PagePdfDocumentReader pdfReader = new PagePdfDocumentReader(
            resource,
            PdfDocumentReaderConfig.builder()
                .withPageExtractedTextFormatter(
                    new ExtractedTextFormatter.Builder()
                        .withNumberOfBottomTextLinesToDelete(3)
                        .withNumberOfTopPagesToSkipBeforeDelete(1)
                        .build())
                .withPagesPerDocument(1)
                .build()
        );
        
        return pdfReader.get();
    }
    
    public List<Document> loadPdfFromUrl(String url) {
        try {
            Resource resource = new UrlResource(url);
            PagePdfDocumentReader pdfReader = 
                new PagePdfDocumentReader(resource);
            return pdfReader.get();
        } catch (Exception e) {
            throw new RuntimeException("Failed to load PDF from URL", e);
        }
    }
}
```

**JSON Documents**:

```java
import org.springframework.ai.reader.JsonReader;

@Service
public class JsonLoaderService {
    
    public List<Document> loadJson(String resourcePath) {
        Resource resource = new ClassPathResource(resourcePath);
        
        JsonReader jsonReader = new JsonReader(
            resource,
            "data",  // JSON key containing array of items
            "title", "content", "category"  // Fields to extract
        );
        
        return jsonReader.get();
    }
    
    public List<Document> loadJsonArray(String filePath) {
        // For JSON like: [{"text": "...", "metadata": {...}}, ...]
        Resource resource = new FileSystemResource(filePath);
        
        JsonReader jsonReader = new JsonReader(resource);
        return jsonReader.get();
    }
}
```

**Example JSON structure**:

```json
{
  "data": [
    {
      "title": "Getting Started with Spring AI",
      "content": "Spring AI provides abstractions for AI integration...",
      "category": "tutorial",
      "tags": ["spring", "ai", "tutorial"]
    },
    {
      "title": "Understanding RAG",
      "content": "RAG combines retrieval and generation...",
      "category": "guide",
      "tags": ["rag", "vector-store"]
    }
  ]
}
```

**Web Scraping** (Custom implementation):

```java
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document as JsoupDocument;

@Service
public class WebLoaderService {
    
    public List<Document> loadWebPage(String url) {
        try {
            JsoupDocument webDoc = Jsoup.connect(url).get();
            
            // Extract main content
            String title = webDoc.title();
            String content = webDoc.select("article, .content, main")
                .text();
            
            // Create Spring AI Document
            Document document = new Document(content);
            document.getMetadata().put("source", url);
            document.getMetadata().put("title", title);
            document.getMetadata().put("type", "web");
            document.getMetadata().put("scraped_at", 
                Instant.now().toString());
            
            return List.of(document);
            
        } catch (IOException e) {
            throw new RuntimeException("Failed to scrape " + url, e);
        }
    }
    
    public List<Document> loadMultiplePages(List<String> urls) {
        return urls.stream()
            .flatMap(url -> loadWebPage(url).stream())
            .collect(Collectors.toList());
    }
}
```

### Text Chunking Strategies

Proper chunking is crucial for RAG performance. Here are the main strategies:

**Token-based Splitting**:

```java
import org.springframework.ai.transformer.splitter.TokenTextSplitter;

@Service
public class ChunkingService {
    
    public List<Document> chunkByTokens(List<Document> documents) {
        TokenTextSplitter splitter = new TokenTextSplitter(
            500,   // Chunk size in tokens
            100,   // Overlap between chunks
            5,     // Minimum chunk size
            10000  // Maximum chunk size
        );
        
        return splitter.apply(documents);
    }
    
    public List<Document> chunkWithCustomSettings(
            List<Document> documents,
            int chunkSize,
            int overlap) {
        
        TokenTextSplitter splitter = new TokenTextSplitter(
            chunkSize,
            overlap
        );
        
        return splitter.apply(documents);
    }
}
```

**Why overlap matters**:

```
Without overlap:
Chunk 1: "...introduced Spring AI in 2024."
Chunk 2: "It provides ChatClient for..."
Problem: Lost context between chunks!

With overlap (100 tokens):
Chunk 1: "...introduced Spring AI in 2024. It provides ChatClient..."
Chunk 2: "Spring AI in 2024. It provides ChatClient for..."
Benefit: Context preserved across boundaries!
```

**Character-based Splitting**:

```java
import org.springframework.ai.transformer.splitter.TextSplitter;

@Service
public class CharacterChunkingService {
    
    public List<Document> chunkByCharacters(List<Document> documents) {
        TextSplitter splitter = new TextSplitter(
            2000,  // Characters per chunk
            400,   // Overlap
            "\n\n" // Split on paragraph breaks
        );
        
        return splitter.split(documents);
    }
    
    public List<Document> chunkBySentences(List<Document> documents) {
        // Split on sentence boundaries
        TextSplitter splitter = new TextSplitter(
            1000,
            200,
            ". "  // Split on periods
        );
        
        return splitter.split(documents);
    }
}
```

**Semantic Chunking** (Advanced):

```java
@Service
public class SemanticChunkingService {
    
    private final EmbeddingClient embeddingClient;
    
    public List<Document> chunkSemantically(List<Document> documents) {
        List<Document> chunks = new ArrayList<>();
        
        for (Document doc : documents) {
            // Split into sentences
            String[] sentences = doc.getContent().split("\\. ");
            
            List<String> currentChunk = new ArrayList<>();
            List<Double> previousEmbedding = null;
            
            for (String sentence : sentences) {
                if (sentence.isBlank()) continue;
                
                // Get embedding for sentence
                List<Double> embedding = embeddingClient.embed(sentence);
                
                // Check semantic similarity with previous
                if (previousEmbedding != null) {
                    double similarity = cosineSimilarity(
                        previousEmbedding, 
                        embedding
                    );
                    
                    // Start new chunk if similarity drops
                    if (similarity < 0.8) {
                        // Save current chunk
                        chunks.add(createDocument(currentChunk, doc));
                        currentChunk.clear();
                    }
                }
                
                currentChunk.add(sentence);
                previousEmbedding = embedding;
            }
            
            // Add final chunk
            if (!currentChunk.isEmpty()) {
                chunks.add(createDocument(currentChunk, doc));
            }
        }
        
        return chunks;
    }
    
    private double cosineSimilarity(List<Double> v1, List<Double> v2) {
        double dotProduct = 0.0;
        double normA = 0.0;
        double normB = 0.0;
        
        for (int i = 0; i < v1.size(); i++) {
            dotProduct += v1.get(i) * v2.get(i);
            normA += Math.pow(v1.get(i), 2);
            normB += Math.pow(v2.get(i), 2);
        }
        
        return dotProduct / (Math.sqrt(normA) * Math.sqrt(normB));
    }
    
    private Document createDocument(List<String> sentences, Document source) {
        String content = String.join(". ", sentences);
        Document doc = new Document(content);
        doc.getMetadata().putAll(source.getMetadata());
        return doc;
    }
}
```

### Metadata Enrichment

Add useful metadata to improve retrieval:

```java
@Service
public class MetadataEnricher {
    
    public void enrichMetadata(List<Document> documents) {
        for (Document doc : documents) {
            Map<String, Object> metadata = doc.getMetadata();
            
            // Add timestamps
            metadata.put("indexed_at", Instant.now().toString());
            
            // Add content statistics
            String content = doc.getContent();
            metadata.put("word_count", countWords(content));
            metadata.put("char_count", content.length());
            
            // Extract key information
            metadata.put("has_code", containsCode(content));
            metadata.put("language", detectLanguage(content));
            
            // Add content type hints
            if (content.contains("```")) {
                metadata.put("contains_code_blocks", true);
            }
            if (content.matches(".*\\d{4}-\\d{2}-\\d{2}.*")) {
                metadata.put("contains_dates", true);
            }
        }
    }
    
    public void addCustomMetadata(
            Document doc, 
            String category,
            String author,
            LocalDate publishDate) {
        
        doc.getMetadata().put("category", category);
        doc.getMetadata().put("author", author);
        doc.getMetadata().put("publish_date", publishDate.toString());
        doc.getMetadata().put("freshness_score", 
            calculateFreshness(publishDate));
    }
    
    private int countWords(String text) {
        return text.split("\\s+").length;
    }
    
    private boolean containsCode(String text) {
        return text.contains("public ") || 
               text.contains("class ") ||
               text.contains("function ") ||
               text.contains("```");
    }
    
    private String detectLanguage(String text) {
        // Simple heuristic - in production, use a proper library
        if (text.matches(".*\\b(the|and|is|are|was)\\b.*")) {
            return "en";
        }
        return "unknown";
    }
    
    private double calculateFreshness(LocalDate publishDate) {
        long daysSincePublish = ChronoUnit.DAYS.between(
            publishDate, 
            LocalDate.now()
        );
        
        // Exponential decay: newer = higher score
        return Math.exp(-daysSincePublish / 365.0);
    }
}
```

### Complete Document Processing Pipeline

```java
@Service
@Slf4j
public class DocumentProcessingPipeline {
    
    private final VectorStore vectorStore;
    private final EmbeddingClient embeddingClient;
    private final MetadataEnricher metadataEnricher;
    
    public void processAndIndex(String filePath, DocumentType type) {
        log.info("Starting document processing for: {}", filePath);
        
        try {
            // Step 1: Load documents
            List<Document> documents = loadDocuments(filePath, type);
            log.info("Loaded {} documents", documents.size());
            
            // Step 2: Chunk documents
            List<Document> chunks = chunkDocuments(documents);
            log.info("Created {} chunks", chunks.size());
            
            // Step 3: Enrich metadata
            metadataEnricher.enrichMetadata(chunks);
            log.info("Enriched metadata for {} chunks", chunks.size());
            
            // Step 4: Generate embeddings and store
            vectorStore.add(chunks);
            log.info("✅ Indexed {} chunks to vector store", chunks.size());
            
        } catch (Exception e) {
            log.error("Failed to process documents", e);
            throw new DocumentProcessingException(
                "Error processing " + filePath, e
            );
        }
    }
    
    private List<Document> loadDocuments(String path, DocumentType type) {
        return switch (type) {
            case PDF -> new PagePdfDocumentReader(
                new FileSystemResource(path)
            ).get();
            
            case TEXT -> new TextReader(path).get();
            
            case JSON -> new JsonReader(
                new FileSystemResource(path)
            ).get();
            
            default -> throw new IllegalArgumentException(
                "Unsupported type: " + type
            );
        };
    }
    
    private List<Document> chunkDocuments(List<Document> documents) {
        TokenTextSplitter splitter = new TokenTextSplitter(
            500,  // Optimal for most use cases
            100   // 20% overlap
        );
        
        return splitter.apply(documents);
    }
    
    public enum DocumentType {
        PDF, TEXT, JSON, WEB
    }
}
```

---

## Vector Embeddings Explained

### What Are Embeddings?

Embeddings are numerical representations of text that capture semantic meaning. Similar concepts have similar vectors.

**Example**:

```java
// Generate embeddings
List<Double> embedding1 = embeddingClient.embed("cat");
List<Double> embedding2 = embeddingClient.embed("kitten");
List<Double> embedding3 = embeddingClient.embed("democracy");

// Calculate similarities
double similarity12 = cosineSimilarity(embedding1, embedding2);
// Result: 0.85 (very similar)

double similarity13 = cosineSimilarity(embedding1, embedding3);
// Result: 0.12 (not similar)
```

### How Embeddings Work

**Traditional keyword matching**:
```
Query: "How do I reset my password?"
Doc: "Password recovery instructions"
Match: ❌ No common words (except "password")
```

**With embeddings**:
```
Query embedding: [0.2, -0.5, 0.8, ..., 0.3]
Doc embedding:   [0.3, -0.4, 0.7, ..., 0.4]
Similarity: 0.92 ✅ High semantic similarity!
```

### Embedding Models in Spring AI

**OpenAI Embeddings**:

```java
@Configuration
public class EmbeddingConfig {
    
    @Bean
    public OpenAiEmbeddingClient openAiEmbeddingClient(
            OpenAiApi openAiApi) {
        
        return new OpenAiEmbeddingClient(
            openAiApi,
            OpenAiEmbeddingOptions.builder()
                .withModel("text-embedding-3-small")  // 1536 dimensions
                .build()
        );
    }
    
    @Bean
    public OpenAiEmbeddingClient largeEmbeddingClient(
            OpenAiApi openAiApi) {
        
        return new OpenAiEmbeddingClient(
            openAiApi,
            OpenAiEmbeddingOptions.builder()
                .withModel("text-embedding-3-large")  // 3072 dimensions
                .build()
        );
    }
}
```

**Model comparison**:

```
text-embedding-3-small:
- Dimensions: 1536
- Cost: $0.02 / 1M tokens
- Speed: Fast
- Use: Most applications

text-embedding-3-large:
- Dimensions: 3072
- Cost: $0.13 / 1M tokens
- Speed: Slower
- Use: Maximum accuracy needed

text-embedding-ada-002 (legacy):
- Dimensions: 1536
- Cost: $0.10 / 1M tokens
- Use: Compatibility with old systems
```

**Local Embeddings with Ollama**:

```java
@Bean
public OllamaEmbeddingClient ollamaEmbeddingClient() {
    return new OllamaEmbeddingClient(
        OllamaApi.builder()
            .baseUrl("http://localhost:11434")
            .build(),
        OllamaOptions.create()
            .withModel("nomic-embed-text")  // Free, runs locally
    );
}
```

**Transformers (Hugging Face models)**:

```java
@Bean
public TransformersEmbeddingClient transformersEmbeddingClient() {
    return new TransformersEmbeddingClient(
        TransformersEmbeddingOptions.builder()
            .withModel("sentence-transformers/all-MiniLM-L6-v2")
            .build()
    );
}
```

### Embedding Service

```java
@Service
public class EmbeddingService {
    
    private final EmbeddingClient embeddingClient;
    
    public List<Double> generateEmbedding(String text) {
        return embeddingClient.embed(text);
    }
    
    public List<List<Double>> batchEmbed(List<String> texts) {
        // More efficient than individual calls
        return embeddingClient.embed(texts);
    }
    
    public Map<String, List<Double>> embedDocuments(
            List<Document> documents) {
        
        Map<String, List<Double>> embeddings = new HashMap<>();
        
        // Process in batches for efficiency
        int batchSize = 100;
        for (int i = 0; i < documents.size(); i += batchSize) {
            int end = Math.min(i + batchSize, documents.size());
            List<Document> batch = documents.subList(i, end);
            
            List<String> texts = batch.stream()
                .map(Document::getContent)
                .collect(Collectors.toList());
            
            List<List<Double>> batchEmbeddings = 
                embeddingClient.embed(texts);
            
            for (int j = 0; j < batch.size(); j++) {
                String docId = batch.get(j).getId();
                embeddings.put(docId, batchEmbeddings.get(j));
            }
        }
        
        return embeddings;
    }
    
    public double calculateSimilarity(String text1, String text2) {
        List<Double> emb1 = embeddingClient.embed(text1);
        List<Double> emb2 = embeddingClient.embed(text2);
        
        return cosineSimilarity(emb1, emb2);
    }
    
    private double cosineSimilarity(
            List<Double> vec1, 
            List<Double> vec2) {
        
        double dotProduct = 0.0;
        double norm1 = 0.0;
        double norm2 = 0.0;
        
        for (int i = 0; i < vec1.size(); i++) {
            dotProduct += vec1.get(i) * vec2.get(i);
            norm1 += vec1.get(i) * vec1.get(i);
            norm2 += vec2.get(i) * vec2.get(i);
        }
        
        return dotProduct / (Math.sqrt(norm1) * Math.sqrt(norm2));
    }
}
```

### Understanding Embedding Dimensions

**Visualization** (simplified to 2D):

```
High-dimensional space (1536D) projected to 2D:

     │
  cat•──•kitten
     │  
     │    •dog
     │     
     │         
  ───┼──────────•car────•vehicle
     │                    
     │  •democracy      
     │    •freedom
```

In reality, these vectors exist in 1536-dimensional space, allowing much more nuanced relationships!

---

## Vector Stores and Databases

### Available Vector Stores

Spring AI supports multiple vector databases. Let's explore each:

**1. PgVector (PostgreSQL)**:

Best for: Production applications needing ACID guarantees

```java
@Configuration
public class PgVectorConfig {
    
    @Bean
    public PgVectorStore pgVectorStore(
            JdbcTemplate jdbcTemplate,
            EmbeddingClient embeddingClient) {
        
        return new PgVectorStore(
            jdbcTemplate,
            embeddingClient,
            PgVectorStore.PgVectorStoreConfig.builder()
                .withIndexType(PgIndexType.HNSW)
                .withDistanceType(PgDistanceType.COSINE_DISTANCE)
                .withDimensions(1536)
                .withRemoveExistingVectorStoreTable(false)
                .build()
        );
    }
}
```

**Setup PostgreSQL schema**:

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- View created table structure
\d vector_store

-- Typical schema:
CREATE TABLE IF NOT EXISTS vector_store (
    id UUID PRIMARY KEY,
    content TEXT,
    metadata JSON,
    embedding vector(1536)
);

-- Create index for fast similarity search
CREATE INDEX ON vector_store 
USING hnsw (embedding vector_cosine_ops);
```

**2. Chroma**:

Best for: Development and prototyping

```java
@Bean
public ChromaVectorStore chromaVectorStore(
        EmbeddingClient embeddingClient) {
    
    ChromaApi chromaApi = new ChromaApi("http://localhost:8000");
    
    return new ChromaVectorStore(
        chromaApi,
        embeddingClient,
        "my-collection",  // Collection name
        true  // Initialize schema
    );
}
```

**Start Chroma**:

```bash
docker run -p 8000:8000 chromadb/chroma
```

**3. SimpleVectorStore**:

Best for: Testing and small datasets

```java
@Bean
public SimpleVectorStore simpleVectorStore(
        EmbeddingClient embeddingClient) {
    
    return new SimpleVectorStore(embeddingClient);
}

// Persist to file
@Component
public class VectorStorePersistence {
    
    @Autowired
    private SimpleVectorStore vectorStore;
    
    @PreDestroy
    public void saveToFile() {
        File file = new File("vector-store.json");
        vectorStore.save(file);
    }
    
    @PostConstruct
    public void loadFromFile() {
        File file = new File("vector-store.json");
        if (file.exists()) {
            vectorStore.load(file);
        }
    }
}
```

**4. Redis**:

Best for: High-performance caching scenarios

```java
@Bean
public RedisVectorStore redisVectorStore(
        EmbeddingClient embeddingClient,
        RedisVectorStoreConfig config) {
    
    return new RedisVectorStore(config, embeddingClient);
}
```

**5. Pinecone**:

Best for: Managed cloud solution

```java
@Bean
public PineconeVectorStore pineconeVectorStore(
        EmbeddingClient embeddingClient) {
    
    PineconeApi pineconeApi = new PineconeApi(
        "your-api-key",
        "your-environment"
    );
    
    return new PineconeVectorStore(
        pineconeApi,
        "your-index-name",
        embeddingClient
    );
}
```

### Vector Store Comparison

| Store | Pros | Cons | Best For |
|-------|------|------|----------|
| **PgVector** | ACID, mature, familiar SQL | Requires PostgreSQL | Production apps |
| **Chroma** | Easy setup, good for development | Less mature | Prototyping |
| **SimpleVector** | No external dependencies | Not scalable | Testing |
| **Redis** | Very fast, caching built-in | In-memory limits | High performance |
| **Pinecone** | Fully managed, scalable | Costs money, vendor lock-in | Cloud-first apps |

### Working with Vector Stores

**Adding documents**:

```java
@Service
public class VectorStoreService {
    
    private final VectorStore vectorStore;
    private final EmbeddingClient embeddingClient;
    
    public void addDocuments(List<Document> documents) {
        // Simple add
        vectorStore.add(documents);
    }
    
    public void addWithEmbeddings(List<Document> documents) {
        // If embeddings already generated
        for (Document doc : documents) {
            List<Double> embedding = embeddingClient.embed(
                doc.getContent()
            );
            doc.setEmbedding(embedding);
        }
        
        vectorStore.add(documents);
    }
    
    public void addBatch(List<Document> documents) {
        // Process in batches for better performance
        int batchSize = 100;
        
        for (int i = 0; i < documents.size(); i += batchSize) {
            int end = Math.min(i + batchSize, documents.size());
            List<Document> batch = documents.subList(i, end);
            
            vectorStore.add(batch);
            
            // Optional: Add delay to respect rate limits
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

**Deleting documents**:

```java
public void deleteDocuments(List<String> documentIds) {
    vectorStore.delete(documentIds);
}

public void deleteByMetadata(String key, String value) {
    // Search for documents
    SearchRequest searchRequest = SearchRequest.query("")
        .withFilterExpression(
            new Filter.Expression(
                Filter.ExpressionType.EQ,
                new Filter.Key(key),
                new Filter.Value(value)
            )
        );
    
    List<Document> docs = vectorStore.similaritySearch(searchRequest);
    
    // Delete found documents
    List<String> ids = docs.stream()
        .map(Document::getId)
        .collect(Collectors.toList());
    
    vectorStore.delete(ids);
}

public void clearAll() {
    // Warning: This deletes everything!
    // Implementation depends on vector store
    if (vectorStore instanceof PgVectorStore pgStore) {
        pgStore.dropVectorStoreTable();
        pgStore.initializeSchema();
    }
}
```

**Updating documents**:

```java
public void updateDocument(String documentId, String newContent) {
    // Delete old version
    vectorStore.delete(List.of(documentId));
    
    // Add new version
    Document updated = new Document(documentId, newContent, Map.of(
        "updated_at", Instant.now().toString()
    ));
    
    vectorStore.add(List.of(updated));
}

public void updateMetadata(String documentId, Map<String, Object> newMetadata) {
    // Retrieve existing document
    SearchRequest request = SearchRequest.query("")
        .withFilterExpression(
            new Filter.Expression(
                Filter.ExpressionType.EQ,
                new Filter.Key("id"),
                new Filter.Value(documentId)
            )
        );
    
    List<Document> docs = vectorStore.similaritySearch(request);
    
    if (!docs.isEmpty()) {
        Document doc = docs.get(0);
        doc.getMetadata().putAll(newMetadata);
        
        // Re-add with updated metadata
        vectorStore.delete(List.of(documentId));
        vectorStore.add(List.of(doc));
    }
}
```

---

## Implementing Similarity Search

### Basic Search

```java
@Service
public class SearchService {
    
    private final VectorStore vectorStore;
    
    public List<Document> search(String query) {
        SearchRequest request = SearchRequest.query(query)
            .withTopK(5);  // Return top 5 results
        
        return vectorStore.similaritySearch(request);
    }
    
    public List<Document> searchWithThreshold(
            String query, 
            double threshold) {
        
        SearchRequest request = SearchRequest.query(query)
            .withTopK(10)
            .withSimilarityThreshold(threshold);  // e.g., 0.7
        
        return vectorStore.similaritySearch(request);
    }
}
```

### Metadata Filtering

```java
public List<Document> searchWithFilter(
        String query, 
        String category) {
    
    Filter.Expression filterExpression = new Filter.Expression(
        Filter.ExpressionType.EQ,
        new Filter.Key("category"),
        new Filter.Value(category)
    );
    
    SearchRequest request = SearchRequest.query(query)
        .withTopK(5)
        .withFilterExpression(filterExpression);
    
    return vectorStore.similaritySearch(request);
}

public List<Document> complexFilter(String query) {
    // AND condition: category = "tutorial" AND language = "java"
    Filter.Expression filter = new Filter.Expression(
        Filter.ExpressionType.AND,
        List.of(
            new Filter.Expression(
                Filter.ExpressionType.EQ,
                new Filter.Key("category"),
                new Filter.Value("tutorial")
            ),
            new Filter.Expression(
                Filter.ExpressionType.EQ,
                new Filter.Key("language"),
                new Filter.Value("java")
            )
        )
    );
    
    SearchRequest request = SearchRequest.query(query)
        .withFilterExpression(filter);
    
    return vectorStore.similaritySearch(request);
}

public List<Document> dateRangeFilter(
        String query, 
        LocalDate startDate,
        LocalDate endDate) {
    
    Filter.Expression filter = new Filter.Expression(
        Filter.ExpressionType.AND,
        List.of(
            new Filter.Expression(
                Filter.ExpressionType.GTE,
                new Filter.Key("publish_date"),
                new Filter.Value(startDate.toString())
            ),
            new Filter.Expression(
                Filter.ExpressionType.LTE,
                new Filter.Key("publish_date"),
                new Filter.Value(endDate.toString())
            )
        )
    );
    
    SearchRequest request = SearchRequest.query(query)
        .withFilterExpression(filter);
    
    return vectorStore.similaritySearch(request);
}
```

### Hybrid Search (Vector + Keyword)

```java
@Service
public class HybridSearchService {
    
    private final VectorStore vectorStore;
    
    public List<Document> hybridSearch(String query) {
        // 1. Vector search
        List<Document> vectorResults = vectorStore.similaritySearch(
            SearchRequest.query(query).withTopK(10)
        );
        
        // 2. Keyword search (using metadata or content)
        List<Document> keywordResults = keywordSearch(query);
        
        // 3. Merge and re-rank
        return mergeResults(vectorResults, keywordResults, query);
    }
    
    private List<Document> keywordSearch(String query) {
        String[] keywords = query.toLowerCase().split("\\s+");
        
        Set<Document> results = new HashSet<>();
        
        for (String keyword : keywords) {
            Filter.Expression filter = new Filter.Expression(
                Filter.ExpressionType.CONTAINS,
                new Filter.Key("content"),
                new Filter.Value(keyword)
            );
            
            results.addAll(vectorStore.similaritySearch(
                SearchRequest.query("")
                    .withFilterExpression(filter)
                    .withTopK(5)
            ));
        }
        
        return new ArrayList<>(results);
    }
    
    private List<Document> mergeResults(
            List<Document> vectorResults,
            List<Document> keywordResults,
            String query) {
        
        Map<String, DocumentScore> scores = new HashMap<>();
        
        // Score vector results (higher weight)
        for (int i = 0; i < vectorResults.size(); i++) {
            Document doc = vectorResults.get(i);
            double score = (vectorResults.size() - i) * 0.7;  // 70% weight
            scores.put(doc.getId(), new DocumentScore(doc, score));
        }
        
        // Add keyword results (lower weight)
        for (int i = 0; i < keywordResults.size(); i++) {
            Document doc = keywordResults.get(i);
            double score = (keywordResults.size() - i) * 0.3;  // 30% weight
            
            scores.merge(
                doc.getId(),
                new DocumentScore(doc, score),
                (existing, newScore) -> new DocumentScore(
                    existing.document(),
                    existing.score() + newScore.score()
                )
            );
        }
        
        // Sort by combined score
        return scores.values().stream()
            .sorted((a, b) -> Double.compare(b.score(), a.score()))
            .map(DocumentScore::document)
            .limit(10)
            .collect(Collectors.toList());
    }
    
    private record DocumentScore(Document document, double score) {}
}
```

### Re-ranking Results

```java
@Service
public class ReRankingService {
    
    public List<Document> reRank(
            List<Document> documents, 
            String query) {
        
        // Score each document based on multiple factors
        return documents.stream()
            .map(doc -> new ScoredDocument(
                doc,
                calculateRelevanceScore(doc, query)
            ))
            .sorted((a, b) -> Double.compare(b.score(), a.score()))
            .map(ScoredDocument::document)
            .collect(Collectors.toList());
    }
    
    private double calculateRelevanceScore(Document doc, String query) {
        double score = 0.0;
        
        // Factor 1: Vector similarity (from metadata)
        if (doc.getMetadata().containsKey("similarity")) {
            score += (Double) doc.getMetadata().get("similarity") * 0.5;
        }
        
        // Factor 2: Recency boost
        if (doc.getMetadata().containsKey("publish_date")) {
            LocalDate publishDate = LocalDate.parse(
                doc.getMetadata().get("publish_date").toString()
            );
            long daysOld = ChronoUnit.DAYS.between(
                publishDate, 
                LocalDate.now()
            );
            double recencyScore = Math.exp(-daysOld / 365.0);
            score += recencyScore * 0.2;
        }
        
        // Factor 3: Keyword match
        String content = doc.getContent().toLowerCase();
        String queryLower = query.toLowerCase();
        String[] queryWords = queryLower.split("\\s+");
        
        int matches = 0;
        for (String word : queryWords) {
            if (content.contains(word)) {
                matches++;
            }
        }
        double keywordScore = (double) matches / queryWords.length;
        score += keywordScore * 0.2;
        
        // Factor 4: Document quality (if available)
        if (doc.getMetadata().containsKey("quality_score")) {
            score += (Double) doc.getMetadata().get("quality_score") * 0.1;
        }
        
        return score;
    }
    
    private record ScoredDocument(Document document, double score) {}
}
```

---

## Building Your First RAG Application

Let's build a complete RAG application from scratch!

### Project Structure

```
src/main/java/com/example/rag/
├── controller/
│   └── RagController.java
├── service/
│   ├── DocumentService.java
│   ├── RagService.java
│   └── SearchService.java
├── config/
│   └── RagConfiguration.java
├── model/
│   ├── QueryRequest.java
│   └── QueryResponse.java
└── RagApplication.java

src/main/resources/
├── application.yml
└── documents/
    ├── spring-ai-guide.pdf
    ├── faq.txt
    └── policies.json
```

### Core RAG Service

```java
package com.example.rag.service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.client.advisor.QuestionAnswerAdvisor;
import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.SearchRequest;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
@Slf4j
@RequiredArgsConstructor
public class RagService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    
    public String query(String userQuestion) {
        log.info("Processing query: {}", userQuestion);
        
        // Retrieve relevant documents
        List<Document> relevantDocs = retrieveDocuments(userQuestion);
        
        if (relevantDocs.isEmpty()) {
            return "I couldn't find relevant information to answer your question.";
        }
        
        // Build context from retrieved documents
        String context = buildContext(relevantDocs);
        
        // Generate answer with context
        String answer = generateAnswer(userQuestion, context);
        
        log.info("Generated answer for query: {}", userQuestion);
        return answer;
    }
    
    public String queryWithAdvisor(String userQuestion) {
        // Use Spring AI's built-in QuestionAnswerAdvisor
        return chatClient.prompt()
            .user(userQuestion)
            .advisors(new QuestionAnswerAdvisor(vectorStore))
            .call()
            .content();
    }
    
    private List<Document> retrieveDocuments(String query) {
        SearchRequest searchRequest = SearchRequest.query(query)
            .withTopK(5)
            .withSimilarityThreshold(0.7);
        
        return vectorStore.similaritySearch(searchRequest);
    }
    
    private String buildContext(List<Document> documents) {
        return documents.stream()
            .map(doc -> {
                String source = doc.getMetadata()
                    .getOrDefault("source", "Unknown")
                    .toString();
                return String.format(
                    "Source: %s\n%s",
                    source,
                    doc.getContent()
                );
            })
            .collect(Collectors.joining("\n\n---\n\n"));
    }
    
    private String generateAnswer(String question, String context) {
        String prompt = String.format("""
            You are a helpful assistant. Answer the question based on the 
            following context. If you cannot answer from the context, 
            say so honestly.
            
            CONTEXT:
            %s
            
            QUESTION: %s
            
            ANSWER:
            """, context, question);
        
        return chatClient.prompt()
            .user(prompt)
            .call()
            .content();
    }
    
    public String queryWithSources(String userQuestion) {
        List<Document> relevantDocs = retrieveDocuments(userQuestion);
        
        if (relevantDocs.isEmpty()) {
            return "No relevant information found.";
        }
        
        String context = buildContext(relevantDocs);
        String answer = generateAnswer(userQuestion, context);
        
        // Append sources
        String sources = relevantDocs.stream()
            .map(doc -> doc.getMetadata()
                .getOrDefault("source", "Unknown")
                .toString())
            .distinct()
            .collect(Collectors.joining(", "));
        
        return String.format(
            "%s\n\nSources: %s",
            answer,
            sources
        );
    }
}
```

### Document Management Service

```java
package com.example.rag.service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.document.Document;
import org.springframework.ai.reader.TextReader;
import org.springframework.ai.reader.pdf.PagePdfDocumentReader;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@Slf4j
@RequiredArgsConstructor
public class DocumentService {
    
    private final VectorStore vectorStore;
    private final ResourceLoader resourceLoader;
    
    public void indexDocument(String resourcePath) {
        log.info("Indexing document: {}", resourcePath);
        
        try {
            Resource resource = resourceLoader.getResource(resourcePath);
            
            // Load document
            List<Document> documents = loadDocument(resource);
            
            // Chunk document
            List<Document> chunks = chunkDocument(documents);
            
            // Add to vector store
            vectorStore.add(chunks);
            
            log.info("Successfully indexed {} chunks from {}",
                chunks.size(), resourcePath);
            
        } catch (Exception e) {
            log.error("Failed to index document: {}", resourcePath, e);
            throw new RuntimeException("Document indexing failed", e);
        }
    }
    
    public void indexMultipleDocuments(List<String> resourcePaths) {
        for (String path : resourcePaths) {
            indexDocument(path);
        }
    }
    
    private List<Document> loadDocument(Resource resource) {
        String filename = resource.getFilename();
        
        if (filename == null) {
            throw new IllegalArgumentException("Invalid resource");
        }
        
        if (filename.endsWith(".pdf")) {
            return new PagePdfDocumentReader(resource).get();
        } else if (filename.endsWith(".txt")) {
            return new TextReader(resource).get();
        } else {
            throw new UnsupportedOperationException(
                "Unsupported file type: " + filename
            );
        }
    }
    
    private List<Document> chunkDocument(List<Document> documents) {
        TokenTextSplitter splitter = new TokenTextSplitter(
            500,  // chunk size
            100   // overlap
        );
        
        return splitter.apply(documents);
    }
    
    public void clearVectorStore() {
        log.warn("Clearing all documents from vector store");
        // Implementation depends on vector store type
        // This is a simplified version
        vectorStore.delete(List.of());  // Remove all
    }
}
```

### REST Controller

```java
package com.example.rag.controller;

import com.example.rag.model.QueryRequest;
import com.example.rag.model.QueryResponse;
import com.example.rag.service.DocumentService;
import com.example.rag.service.RagService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

@RestController
@RequestMapping("/api/rag")
@RequiredArgsConstructor
public class RagController {
    
    private final RagService ragService;
    private final DocumentService documentService;
    
    @PostMapping("/query")
    public ResponseEntity<QueryResponse> query(
            @RequestBody QueryRequest request) {
        
        String answer = ragService.queryWithSources(request.question());
        
        return ResponseEntity.ok(new QueryResponse(
            answer,
            true,
            null
        ));
    }
    
    @PostMapping("/index")
    public ResponseEntity<String> indexDocument(
            @RequestParam("path") String resourcePath) {
        
        try {
            documentService.indexDocument(resourcePath);
            return ResponseEntity.ok(
                "Document indexed successfully: " + resourcePath
            );
        } catch (Exception e) {
            return ResponseEntity.badRequest()
                .body("Failed to index document: " + e.getMessage());
        }
    }
    
    @PostMapping("/index/batch")
    public ResponseEntity<String> indexMultipleDocuments(
            @RequestBody List<String> resourcePaths) {
        
        try {
            documentService.indexMultipleDocuments(resourcePaths);
            return ResponseEntity.ok(
                "Indexed " + resourcePaths.size() + " documents"
            );
        } catch (Exception e) {
            return ResponseEntity.badRequest()
                .body("Failed to index documents: " + e.getMessage());
        }
    }
    
    @DeleteMapping("/clear")
    public ResponseEntity<String> clearVectorStore() {
        documentService.clearVectorStore();
        return ResponseEntity.ok("Vector store cleared");
    }
}
```

### Request/Response Models

```java
package com.example.rag.model;

public record QueryRequest(
    String question,
    int maxResults,
    double similarityThreshold
) {
    public QueryRequest(String question) {
        this(question, 5, 0.7);
    }
}

public record QueryResponse(
    String answer,
    boolean success,
    String error
) {}
```

### Testing Your RAG Application

**1. Index documents**:

```bash
# Index a single document
curl -X POST "http://localhost:8080/api/rag/index?path=classpath:documents/spring-ai-guide.pdf"

# Index multiple documents
curl -X POST http://localhost:8080/api/rag/index/batch \
  -H "Content-Type: application/json" \
  -d '["classpath:documents/faq.txt", "classpath:documents/policies.json"]'
```

**2. Query the system**:

```bash
curl -X POST http://localhost:8080/api/rag/query \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What are the main features of Spring AI?",
    "maxResults": 5,
    "similarityThreshold": 0.7
  }'
```

**Response**:

```json
{
  "answer": "Spring AI provides several main features:\n\n1. ChatClient for easy integration with various LLM providers like OpenAI, Azure OpenAI, and Anthropic.\n\n2. Prompt templates for creating reusable, parameterized prompts.\n\n3. Vector stores for implementing RAG (Retrieval-Augmented Generation) patterns.\n\n4. Embedding support for converting text into vector representations.\n\n5. Function calling capabilities for extending LLM functionality with custom tools.\n\nSources: spring-ai-guide.pdf, faq.txt",
  "success": true,
  "error": null
}
```

---

## Advanced RAG Patterns

### Conversational RAG

Maintain chat history for context-aware conversations:

```java
@Service
public class ConversationalRagService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    private final Map<String, List<Message>> conversationHistory;
    
    public ConversationalRagService(
            ChatClient chatClient,
            VectorStore vectorStore) {
        this.chatClient = chatClient;
        this.vectorStore = vectorStore;
        this.conversationHistory = new ConcurrentHashMap<>();
    }
    
    public String chat(String sessionId, String userMessage) {
        // Get or create conversation history
        List<Message> history = conversationHistory.computeIfAbsent(
            sessionId,
            k -> new ArrayList<>()
        );
        
        // Retrieve relevant context
        List<Document> relevantDocs = vectorStore.similaritySearch(
            SearchRequest.query(userMessage).withTopK(3)
        );
        
        String context = relevantDocs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        // Build prompt with history and context
        String systemMessage = String.format("""
            You are a helpful assistant. Use the following context 
            to answer questions. Consider the conversation history 
            for context.
            
            CONTEXT:
            %s
            """, context);
        
        // Create conversation with history
        List<Message> messages = new ArrayList<>();
        messages.add(new Message(MessageType.SYSTEM, systemMessage));
        messages.addAll(history);
        messages.add(new Message(MessageType.USER, userMessage));
        
        // Generate response
        String response = chatClient.prompt()
            .messages(messages)
            .call()
            .content();
        
        // Update history
        history.add(new Message(MessageType.USER, userMessage));
        history.add(new Message(MessageType.ASSISTANT, response));
        
        // Keep only last 10 messages to prevent token overflow
        if (history.size() > 10) {
            history = history.subList(
                history.size() - 10,
                history.size()
            );
            conversationHistory.put(sessionId, history);
        }
        
        return response;
    }
    
    public void clearHistory(String sessionId) {
        conversationHistory.remove(sessionId);
    }
    
    private record Message(MessageType type, String content) {}
    
    private enum MessageType {
        SYSTEM, USER, ASSISTANT
    }
}
```

### Multi-Index RAG

Route queries to different vector stores based on content:

```java
@Service
public class MultiIndexRagService {
    
    private final Map<String, VectorStore> vectorStores;
    private final ChatClient chatClient;
    
    public MultiIndexRagService(
            @Qualifier("technicalDocs") VectorStore technicalDocs,
            @Qualifier("customerSupport") VectorStore customerSupport,
            @Qualifier("productInfo") VectorStore productInfo,
            ChatClient chatClient) {
        
        this.vectorStores = Map.of(
            "technical", technicalDocs,
            "support", customerSupport,
            "product", productInfo
        );
        this.chatClient = chatClient;
    }
    
    public String query(String userQuestion) {
        // Determine which index to query
        String indexType = classifyQuery(userQuestion);
        
        // Get appropriate vector store
        VectorStore vectorStore = vectorStores.get(indexType);
        
        // Retrieve documents
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.query(userQuestion).withTopK(5)
        );
        
        // Generate answer
        String context = docs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        return chatClient.prompt()
            .system(String.format("""
                Answer based on the following %s documentation:
                
                %s
                """, indexType, context))
            .user(userQuestion)
            .call()
            .content();
    }
    
    private String classifyQuery(String query) {
        // Simple classification - in production, use a classifier model
        String lower = query.toLowerCase();
        
        if (lower.matches(".*(error|bug|debug|stack trace).*")) {
            return "technical";
        } else if (lower.matches(".*(how to|setup|install|configure).*")) {
            return "support";
        } else if (lower.matches(".*(price|feature|spec|comparison).*")) {
            return "product";
        }
        
        return "support";  // default
    }
}
```

### Parent Document Retrieval

Retrieve small chunks but provide larger context:

```java
@Service
public class ParentDocumentRagService {
    
    private final VectorStore vectorStore;
    private final ChatClient chatClient;
    private final Map<String, Document> parentDocuments;
    
    public String queryWithParentContext(String question) {
        // 1. Search with small chunks for precision
        List<Document> chunks = vectorStore.similaritySearch(
            SearchRequest.query(question).withTopK(3)
        );
        
        // 2. Retrieve parent documents for full context
        List<Document> parents = chunks.stream()
            .map(chunk -> {
                String parentId = (String) chunk.getMetadata()
                    .get("parent_id");
                return parentDocuments.get(parentId);
            })
            .distinct()
            .collect(Collectors.toList());
        
        // 3. Build context from parent documents
        String context = parents.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n---\n\n"));
        
        // 4. Generate answer with full context
        return chatClient.prompt()
            .system("Answer based on this context:\n\n" + context)
            .user(question)
            .call()
            .content();
    }
    
    public void indexWithParents(List<Document> documents) {
        for (Document parent : documents) {
            // Store parent
            String parentId = UUID.randomUUID().toString();
            parentDocuments.put(parentId, parent);
            
            // Create small chunks
            TokenTextSplitter splitter = new TokenTextSplitter(200, 50);
            List<Document> chunks = splitter.apply(List.of(parent));
            
            // Link chunks to parent
            for (Document chunk : chunks) {
                chunk.getMetadata().put("parent_id", parentId);
            }
            
            // Index chunks
            vectorStore.add(chunks);
        }
    }
}
```

### Hypothetical Document Embeddings (HyDE)

Generate hypothetical answers and search with them:

```java
@Service
public class HydeRagService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    
    public String queryWithHyde(String question) {
        // 1. Generate hypothetical answer
        String hypotheticalAnswer = chatClient.prompt()
            .user(String.format("""
                Write a detailed answer to this question as if you 
                knew the answer. Be specific and technical.
                
                Question: %s
                """, question))
            .call()
            .content();
        
        // 2. Search using hypothetical answer
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.query(hypotheticalAnswer).withTopK(5)
        );
        
        // 3. Generate real answer from retrieved docs
        String context = docs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        return chatClient.prompt()
            .system("Answer based on this context:\n\n" + context)
            .user(question)
            .call()
            .content();
    }
}
```

---

## Optimizing RAG Performance

### Chunk Size Optimization

Finding the optimal chunk size:

```java
@Service
public class ChunkSizeOptimizer {
    
    public void analyzeOptimalChunkSize(List<Document> documents) {
        int[] chunkSizes = {250, 500, 750, 1000, 1500};
        
        for (int chunkSize : chunkSizes) {
            TokenTextSplitter splitter = new TokenTextSplitter(
                chunkSize,
                chunkSize / 5  // 20% overlap
            );
            
            List<Document> chunks = splitter.apply(documents);
            
            System.out.printf("""
                Chunk Size: %d tokens
                Total Chunks: %d
                Avg Content Length: %.1f chars
                """,
                chunkSize,
                chunks.size(),
                chunks.stream()
                    .mapToInt(d -> d.getContent().length())
                    .average()
                    .orElse(0)
            );
        }
    }
}
```

**Recommendations**:

```
Chunk Size Guidelines:

200-300 tokens:
✅ Precise retrieval
✅ Fast search
❌ May lose context
Use for: Q&A, FAQs

500-750 tokens:
✅ Good balance
✅ Maintains context
Use for: Documentation, articles

1000-1500 tokens:
✅ Maximum context
❌ Less precise
❌ Higher costs
Use for: Long-form content, books
```

### Caching Strategies

```java
@Service
public class CachedRagService {
    
    private final ChatClient chatClient;
    private final VectorStore vectorStore;
    
    @Cacheable(value = "ragQueries", key = "#question")
    public String queryCached(String question) {
        return query(question);
    }
    
    @Cacheable(value = "documentRetrievals", key = "#query")
    public List<Document> retrieveDocumentsCached(String query) {
        return vectorStore.similaritySearch(
            SearchRequest.query(query).withTopK(5)
        );
    }
    
    private String query(String question) {
        List<Document> docs = retrieveDocumentsCached(question);
        String context = docs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        return chatClient.prompt()
            .system("Context:\n" + context)
            .user(question)
            .call()
            .content();
    }
}
```

**Cache configuration**:

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        
        cacheManager.setCaches(Arrays.asList(
            // Cache query results for 1 hour
            new ConcurrentMapCache("ragQueries"),
            
            // Cache document retrievals for 30 minutes
            new ConcurrentMapCache("documentRetrievals")
        ));
        
        return cacheManager;
    }
}
```

### Batch Processing

```java
@Service
public class BatchIndexingService {
    
    private final VectorStore vectorStore;
    private final ExecutorService executorService;
    
    public BatchIndexingService(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
        this.executorService = Executors.newFixedThreadPool(4);
    }
    
    public CompletableFuture<Void> indexDocumentsAsync(
            List<Document> documents) {
        
        return CompletableFuture.runAsync(() -> {
            int batchSize = 100;
            
            for (int i = 0; i < documents.size(); i += batchSize) {
                int end = Math.min(i + batchSize, documents.size());
                List<Document> batch = documents.subList(i, end);
                
                try {
                    vectorStore.add(batch);
                    
                    // Log progress
                    int progress = (int) ((i + batch.size()) * 100.0 / 
                        documents.size());
                    System.out.printf("Indexing progress: %d%%\n", progress);
                    
                    // Rate limiting
                    Thread.sleep(100);
                    
                } catch (Exception e) {
                    System.err.println("Failed to index batch: " + e.getMessage());
                }
            }
        }, executorService);
    }
}
```

### Query Optimization

```java
@Service
public class QueryOptimizationService {
    
    private final ChatClient chatClient;
    
    public String optimizeQuery(String userQuery) {
        // Rewrite query for better retrieval
        String optimizedQuery = chatClient.prompt()
            .user(String.format("""
                Rewrite this question to be more specific and 
                searchable. Include key technical terms.
                
                Original: %s
                
                Optimized:
                """, userQuery))
            .call()
            .content();
        
        return optimizedQuery;
    }
    
    public List<String> generateQueryVariations(String query) {
        // Generate multiple query variations
        String prompt = String.format("""
            Generate 3 different ways to ask this question:
            
            "%s"
            
            Return as JSON array: ["variation1", "variation2", "variation3"]
            """, query);
        
        String response = chatClient.prompt()
            .user(prompt)
            .call()
            .content();
        
        // Parse JSON (simplified)
        return parseQueryVariations(response);
    }
    
    private List<String> parseQueryVariations(String json) {
        // Implement JSON parsing
        return List.of();  // Simplified
    }
}
```

---

## RAG Use Cases and Examples

### Customer Support Assistant

```java
@Service
public class CustomerSupportRagService {
    
    private final VectorStore knowledgeBase;
    private final ChatClient chatClient;
    
    public SupportResponse handleCustomerQuery(String customerQuery) {
        // 1. Classify query urgency
        String urgency = classifyUrgency(customerQuery);
        
        // 2. Search knowledge base
        List<Document> articles = knowledgeBase.similaritySearch(
            SearchRequest.query(customerQuery)
                .withTopK(3)
                .withFilterExpression(
                    new Filter.Expression(
                        Filter.ExpressionType.EQ,
                        new Filter.Key("category"),
                        new Filter.Value("support")
                    )
                )
        );
        
        // 3. Check if we can answer automatically
        if (canAnswerAutomatically(articles, urgency)) {
            String answer = generateAnswer(customerQuery, articles);
            return new SupportResponse(
                answer,
                findRelatedArticles(articles),
                false  // No human escalation needed
            );
        } else {
            return new SupportResponse(
                "I'll connect you with a support specialist.",
                findRelatedArticles(articles),
                true  // Escalate to human
            );
        }
    }
    
    private String classifyUrgency(String query) {
        String classification = chatClient.prompt()
            .user(String.format("""
                Classify this support query as: LOW, MEDIUM, or HIGH urgency.
                Respond with only the classification.
                
                Query: %s
                """, query))
            .call()
            .content();
        
        return classification.trim().toUpperCase();
    }
    
    private boolean canAnswerAutomatically(
            List<Document> articles, 
            String urgency) {
        
        // Don't auto-answer high urgency issues
        if ("HIGH".equals(urgency)) {
            return false;
        }
        
        // Need good matches to auto-answer
        return articles.size() >= 2;
    }
    
    private String generateAnswer(
            String query, 
            List<Document> articles) {
        
        String context = articles.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        return chatClient.prompt()
            .system("""
                You are a helpful customer support assistant.
                Provide clear, friendly answers based on the knowledge base.
                If the answer isn't in the context, politely say so.
                """)
            .user(String.format("""
                KNOWLEDGE BASE:
                %s
                
                CUSTOMER QUESTION:
                %s
                
                ANSWER:
                """, context, query))
            .call()
            .content();
    }
    
    private List<String> findRelatedArticles(List<Document> docs) {
        return docs.stream()
            .map(doc -> (String) doc.getMetadata().get("title"))
            .collect(Collectors.toList());
    }
    
    public record SupportResponse(
        String answer,
        List<String> relatedArticles,
        boolean escalateToHuman
    ) {}
}
```

### Code Documentation Assistant

```java
@Service
public class CodeDocumentationRagService {
    
    private final VectorStore codebaseIndex;
    private final ChatClient chatClient;
    
    public String explainCode(String codeSnippet) {
        // Find similar code patterns
        List<Document> similarExamples = codebaseIndex.similaritySearch(
            SearchRequest.query(codeSnippet)
                .withTopK(5)
                .withFilterExpression(
                    new Filter.Expression(
                        Filter.ExpressionType.EQ,
                        new Filter.Key("type"),
                        new Filter.Value("code")
                    )
                )
        );
        
        String examples = similarExamples.stream()
            .map(doc -> String.format("""
                Example from %s:
                %s
                
                Documentation:
                %s
                """,
                doc.getMetadata().get("file"),
                doc.getContent(),
                doc.getMetadata().get("documentation")
            ))
            .collect(Collectors.joining("\n\n---\n\n"));
        
        return chatClient.prompt()
            .system("""
                You are a code documentation expert.
                Explain code clearly with examples from the codebase.
                """)
            .user(String.format("""
                Explain this code:
                
                ```
                %s
                ```
                
                Similar examples from our codebase:
                %s
                """, codeSnippet, examples))
            .call()
            .content();
    }
    
    public String generateDocumentation(String code, String language) {
        return chatClient.prompt()
            .user(String.format("""
                Generate comprehensive documentation for this %s code:
                
                ```%s
                %s
                ```
                
                Include:
                - Brief description
                - Parameters and return values
                - Usage examples
                - Edge cases
                """, language, language, code))
            .call()
            .content();
    }
}
```

### Research Paper Assistant

```java
@Service
public class ResearchPaperRagService {
    
    private final VectorStore paperDatabase;
    private final ChatClient chatClient;
    
    public ResearchSummary analyzeTopic(String researchTopic) {
        // Find relevant papers
        List<Document> papers = paperDatabase.similaritySearch(
            SearchRequest.query(researchTopic)
                .withTopK(10)
                .withFilterExpression(
                    new Filter.Expression(
                        Filter.ExpressionType.GT,
                        new Filter.Key("citations"),
                        new Filter.Value(10)  // Only well-cited papers
                    )
                )
        );
        
        // Group by year for trend analysis
        Map<Integer, List<Document>> byYear = papers.stream()
            .collect(Collectors.groupingBy(
                doc -> (Integer) doc.getMetadata().get("year")
            ));
        
        // Generate summary
        String findings = synthesizeFindings(papers, researchTopic);
        String trends = analyzeTrends(byYear);
        List<String> keyPapers = extractKeyPapers(papers);
        
        return new ResearchSummary(
            researchTopic,
            findings,
            trends,
            keyPapers,
            papers.size()
        );
    }
    
    private String synthesizeFindings(
            List<Document> papers, 
            String topic) {
        
        String abstracts = papers.stream()
            .limit(5)  // Top 5 most relevant
            .map(doc -> String.format("""
                Paper: %s (%d)
                %s
                """,
                doc.getMetadata().get("title"),
                doc.getMetadata().get("year"),
                doc.getContent()
            ))
            .collect(Collectors.joining("\n\n"));
        
        return chatClient.prompt()
            .system("You are a research analyst.")
            .user(String.format("""
                Synthesize the key findings about "%s" from these papers:
                
                %s
                
                Provide:
                1. Main consensus findings
                2. Conflicting viewpoints
                3. Research gaps
                """, topic, abstracts))
            .call()
            .content();
    }
    
    private String analyzeTrends(Map<Integer, List<Document>> byYear) {
        StringBuilder trends = new StringBuilder();
        
        byYear.entrySet().stream()
            .sorted(Map.Entry.comparingByKey())
            .forEach(entry -> {
                trends.append(String.format(
                    "%d: %d papers\n",
                    entry.getKey(),
                    entry.getValue().size()
                ));
            });
        
        return trends.toString();
    }
    
    private List<String> extractKeyPapers(List<Document> papers) {
        return papers.stream()
            .sorted((a, b) -> {
                int citA = (Integer) a.getMetadata().get("citations");
                int citB = (Integer) b.getMetadata().get("citations");
                return Integer.compare(citB, citA);
            })
            .limit(5)
            .map(doc -> String.format(
                "%s (%d citations)",
                doc.getMetadata().get("title"),
                doc.getMetadata().get("citations")
            ))
            .collect(Collectors.toList());
    }
    
    public record ResearchSummary(
        String topic,
        String findings,
        String trends,
        List<String> keyPapers,
        int totalPapers
    ) {}
}
```

---

## Challenges and Solutions

### Challenge 1: Hallucinations

**Problem**: LLM generates plausible but incorrect information.

**Solution**:

```java
@Service
public class HallucinationPreventionService {
    
    private final ChatClient chatClient;
    
    public String generateFactualAnswer(
            String question, 
            List<Document> context) {
        
        String contextText = context.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        // Strict prompt to prevent hallucinations
        String answer = chatClient.prompt()
            .system("""
                IMPORTANT RULES:
                1. Answer ONLY based on the provided context
                2. If information is not in the context, say "I don't have information about that"
                3. Do NOT make up facts or speculate
                4. Quote directly from context when possible
                5. If context is ambiguous, acknowledge it
                """)
            .user(String.format("""
                CONTEXT:
                %s
                
                QUESTION: %s
                
                ANSWER (following all rules):
                """, contextText, question))
            .call()
            .content();
        
        // Verify answer against context
        if (isLikelyHallucination(answer, context)) {
            return "I cannot confidently answer this question " +
                   "based on the available information.";
        }
        
        return answer;
    }
    
    private boolean isLikelyHallucination(
            String answer, 
            List<Document> context) {
        
        // Check if answer contains facts not in context
        String combinedContext = context.stream()
            .map(Document::getContent)
            .collect(Collectors.joining(" "))
            .toLowerCase();
        
        // Extract potential facts from answer
        String[] answerSentences = answer.split("\\.");
        
        for (String sentence : answerSentences) {
            // Skip generic sentences
            if (sentence.length() < 20) continue;
            
            // Check if sentence content appears in context
            String[] words = sentence.toLowerCase()
                .split("\\s+");
            
            int matches = 0;
            for (String word : words) {
                if (word.length() > 3 && 
                    combinedContext.contains(word)) {
                    matches++;
                }
            }
            
            // If too few matches, might be hallucination
            if (matches < words.length * 0.3) {
                return true;
            }
        }
        
        return false;
    }
}
```

### Challenge 2: Context Window Limits

**Problem**: Too much context exceeds token limits.

**Solution**:

```java
@Service
public class ContextWindowManager {
    
    private static final int MAX_CONTEXT_TOKENS = 4000;
    
    public String truncateContext(
            List<Document> documents, 
            String question) {
        
        // Prioritize most relevant documents
        List<Document> sorted = documents.stream()
            .sorted((a, b) -> {
                double scoreA = (Double) a.getMetadata()
                    .getOrDefault("similarity", 0.0);
                double scoreB = (Double) b.getMetadata()
                    .getOrDefault("similarity", 0.0);
                return Double.compare(scoreB, scoreA);
            })
            .collect(Collectors.toList());
        
        StringBuilder context = new StringBuilder();
        int tokenCount = 0;
        
        for (Document doc : sorted) {
            String content = doc.getContent();
            int docTokens = estimateTokens(content);
            
            if (tokenCount + docTokens <= MAX_CONTEXT_TOKENS) {
                context.append(content).append("\n\n");
                tokenCount += docTokens;
            } else {
                // Add partial content if space remains
                int remainingTokens = MAX_CONTEXT_TOKENS - tokenCount;
                if (remainingTokens > 100) {
                    String partial = truncateToTokens(
                        content, 
                        remainingTokens
                    );
                    context.append(partial);
                }
                break;
            }
        }
        
        return context.toString();
    }
    
    private int estimateTokens(String text) {
        // Rough estimate: 1 token ≈ 4 characters
        return text.length() / 4;
    }
    
    private String truncateToTokens(String text, int maxTokens) {
        int maxChars = maxTokens * 4;
        if (text.length() <= maxChars) {
            return text;
        }
        
        // Truncate at sentence boundary
        String truncated = text.substring(0, maxChars);
        int lastPeriod = truncated.lastIndexOf('.');
        
        if (lastPeriod > 0) {
            return truncated.substring(0, lastPeriod + 1);
        }
        
        return truncated + "...";
    }
}
```

### Challenge 3: Slow Retrieval

**Problem**: Vector search takes too long.

**Solution**:

```java
@Service
public class FastRetrievalService {
    
    private final VectorStore vectorStore;
    
    @Async
    public CompletableFuture<List<Document>> retrieveAsync(
            String query) {
        
        return CompletableFuture.supplyAsync(() ->
            vectorStore.similaritySearch(
                SearchRequest.query(query).withTopK(5)
            )
        );
    }
    
    public List<Document> retrieveWithTimeout(
            String query, 
            Duration timeout) {
        
        try {
            return retrieveAsync(query)
                .get(timeout.toMillis(), TimeUnit.MILLISECONDS);
        } catch (TimeoutException e) {
            // Fallback to cached or default results
            return getCachedResults(query);
        } catch (Exception e) {
            throw new RuntimeException("Retrieval failed", e);
        }
    }
    
    @Cacheable("queryResults")
    private List<Document> getCachedResults(String query) {
        return vectorStore.similaritySearch(
            SearchRequest.query(query).withTopK(3)
        );
    }
}
```

### Challenge 4: Stale Data

**Problem**: Vector store contains outdated information.

**Solution**:

```java
@Service
public class DataFreshnessService {
    
    private final VectorStore vectorStore;
    
    @Scheduled(cron = "0 0 2 * * *")  // 2 AM daily
    public void refreshStaleDocuments() {
        // Find documents older than 30 days
        LocalDate cutoffDate = LocalDate.now().minusDays(30);
        
        SearchRequest request = SearchRequest.query("")
            .withFilterExpression(
                new Filter.Expression(
                    Filter.ExpressionType.LT,
                    new Filter.Key("indexed_at"),
                    new Filter.Value(cutoffDate.toString())
                )
            );
        
        List<Document> staleDocs = vectorStore.similaritySearch(request);
        
        // Re-index or remove
        for (Document doc : staleDocs) {
            if (shouldUpdate(doc)) {
                updateDocument(doc);
            } else {
                vectorStore.delete(List.of(doc.getId()));
            }
        }
    }
    
    private boolean shouldUpdate(Document doc) {
        // Check if source still exists
        String source = (String) doc.getMetadata().get("source");
        return sourceStillExists(source);
    }
    
    private void updateDocument(Document doc) {
        // Re-fetch and re-index
        // Implementation depends on source type
    }
    
    private boolean sourceStillExists(String source) {
        // Check if source file/URL still exists
        return true;  // Simplified
    }
}
```

---

## Production Considerations

### Monitoring and Logging

```java
@Service
@Slf4j
public class MonitoredRagService {
    
    private final VectorStore vectorStore;
    private final ChatClient chatClient;
    private final MeterRegistry meterRegistry;
    
    public String query(String question) {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try {
            log.info("Processing query: {}", question);
            
            // Retrieve
            List<Document> docs = retrieveWithMetrics(question);
            
            // Generate
            String answer = generateWithMetrics(question, docs);
            
            // Record success
            sample.stop(meterRegistry.timer(
                "rag.query.duration",
                "outcome", "success"
            ));
            
            meterRegistry.counter(
                "rag.queries",
                "status", "success"
            ).increment();
            
            return answer;
            
        } catch (Exception e) {
            log.error("Query failed", e);
            
            sample.stop(meterRegistry.timer(
                "rag.query.duration",
                "outcome", "failure"
            ));
            
            meterRegistry.counter(
                "rag.queries",
                "status", "failure"
            ).increment();
            
            throw e;
        }
    }
    
    private List<Document> retrieveWithMetrics(String query) {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.query(query).withTopK(5)
        );
        
        sample.stop(meterRegistry.timer("rag.retrieval.duration"));
        
        meterRegistry.gauge(
            "rag.retrieved.documents",
            docs.size()
        );
        
        return docs;
    }
    
    private String generateWithMetrics(
            String question, 
            List<Document> docs) {
        
        Timer.Sample sample = Timer.start(meterRegistry);
        
        String context = docs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        String answer = chatClient.prompt()
            .system("Context:\n" + context)
            .user(question)
            .call()
            .content();
        
        sample.stop(meterRegistry.timer("rag.generation.duration"));
        
        return answer;
    }
}
```

### Error Handling

```java
@Service
public class ResilientRagService {
    
    private final VectorStore primaryStore;
    private final VectorStore fallbackStore;
    private final ChatClient chatClient;
    
    @Retry(
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000)
    )
    public String queryWithRetry(String question) {
        try {
            return query(primaryStore, question);
        } catch (VectorStoreException e) {
            // Try fallback store
            return query(fallbackStore, question);
        }
    }
    
    @CircuitBreaker(
        name = "ragService",
        fallbackMethod = "queryFallback"
    )
    public String queryWithCircuitBreaker(String question) {
        return query(primaryStore, question);
    }
    
    public String queryFallback(
            String question, 
            Exception e) {
        
        return "I'm experiencing technical difficulties. " +
               "Please try again in a moment.";
    }
    
    private String query(VectorStore store, String question) {
        List<Document> docs = store.similaritySearch(
            SearchRequest.query(question).withTopK(5)
        );
        
        if (docs.isEmpty()) {
            throw new NoRelevantDocumentsException();
        }
        
        String context = docs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        return chatClient.prompt()
            .system("Context:\n" + context)
            .user(question)
            .call()
            .content();
    }
}
```

### Security

```java
@Service
public class SecureRagService {
    
    private final VectorStore vectorStore;
    private final ChatClient chatClient;
    
    public String querySecure(
            String question, 
            String userId,
            List<String> userRoles) {
        
        // Validate input
        validateQuery(question);
        
        // Apply access controls
        Filter.Expression accessFilter = buildAccessFilter(
            userId, 
            userRoles
        );
        
        // Retrieve with permissions
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.query(question)
                .withTopK(5)
                .withFilterExpression(accessFilter)
        );
        
        // Sanitize context
        String context = sanitizeContext(docs);
        
        // Generate answer
        return chatClient.prompt()
            .system(context)
            .user(question)
            .call()
            .content();
    }
    
    private void validateQuery(String question) {
        if (question == null || question.isBlank()) {
            throw new IllegalArgumentException("Empty query");
        }
        
        if (question.length() > 1000) {
            throw new IllegalArgumentException("Query too long");
        }
        
        // Check for injection attempts
        if (containsSuspiciousPatterns(question)) {
            throw new SecurityException("Suspicious query detected");
        }
    }
    
    private boolean containsSuspiciousPatterns(String text) {
        String[] suspiciousPatterns = {
            "(?i)ignore.*instructions",
            "(?i)disregard.*previous",
            "<script>",
            "javascript:"
        };
        
        for (String pattern : suspiciousPatterns) {
            if (text.matches(".*" + pattern + ".*")) {
                return true;
            }
        }
        
        return false;
    }
    
    private Filter.Expression buildAccessFilter(
            String userId, 
            List<String> roles) {
        
        // User can see public docs or their own docs
        return new Filter.Expression(
            Filter.ExpressionType.OR,
            List.of(
                new Filter.Expression(
                    Filter.ExpressionType.EQ,
                    new Filter.Key("visibility"),
                    new Filter.Value("public")
                ),
                new Filter.Expression(
                    Filter.ExpressionType.EQ,
                    new Filter.Key("owner"),
                    new Filter.Value(userId)
                )
            )
        );
    }
    
    private String sanitizeContext(List<Document> docs) {
        return docs.stream()
            .map(doc -> {
                String content = doc.getContent();
                // Remove sensitive information
                content = content.replaceAll(
                    "\\b\\d{3}-\\d{2}-\\d{4}\\b",  // SSN
                    "[REDACTED]"
                );
                content = content.replaceAll(
                    "\\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Z|a-z]{2,}\\b",  // Email
                    "[EMAIL]"
                );
                return content;
            })
            .collect(Collectors.joining("\n\n"));
    }
}
```

---

## Best Practices

### 1. Document Preparation

✅ **DO**:
- Clean and normalize text before indexing
- Add rich metadata for better filtering
- Maintain document versions
- Include source citations

❌ **DON'T**:
- Index raw, unprocessed documents
- Ignore metadata opportunities
- Mix unrelated content in one document
- Forget to track document lineage

**Example**:

```java
public Document prepareDocument(String rawContent, String source) {
    // Clean content
    String cleaned = rawContent
        .replaceAll("\\s+", " ")  // Normalize whitespace
        .replaceAll("[^\\p{Print}]", "")  // Remove non-printable
        .trim();
    
    // Create document with metadata
    Document doc = new Document(cleaned);
    
    // Rich metadata
    doc.getMetadata().put("source", source);
    doc.getMetadata().put("indexed_at", Instant.now().toString());
    doc.getMetadata().put("version", "1.0");
    doc.getMetadata().put("language", detectLanguage(cleaned));
    doc.getMetadata().put("word_count", cleaned.split("\\s+").length);
    
    return doc;
}
```

### 2. Chunking Strategy

✅ **DO**:
- Use semantic boundaries (paragraphs, sections)
- Maintain 10-20% overlap
- Keep chunks self-contained
- Test different chunk sizes

❌ **DON'T**:
- Split mid-sentence
- Use fixed character counts without overlap
- Make chunks too large (>1500 tokens)
- Ignore context preservation

### 3. Retrieval Optimization

✅ **DO**:
- Retrieve 3-5 documents for most queries
- Use similarity thresholds (0.7+)
- Implement re-ranking
- Cache frequent queries

❌ **DON'T**:
- Retrieve too many documents (>10)
- Accept low-similarity matches (<0.5)
- Return results without scoring
- Skip caching entirely

### 4. Prompt Engineering

✅ **DO**:
- Be explicit about using context
- Instruct model to admit ignorance
- Include examples in prompts
- Test prompts thoroughly

❌ **DON'T**:
- Assume model will use context correctly
- Allow hallucinations
- Use vague instructions
- Deploy untested prompts

**Good prompt template**:

```java
String prompt = """
    You are a helpful assistant. Answer the question based ONLY on the 
    context below. If you cannot answer from the context, say "I don't 
    have enough information to answer this question."
    
    CONTEXT:
    %s
    
    QUESTION: %s
    
    INSTRUCTIONS:
    - Answer concisely
    - Cite sources when possible
    - Admit if uncertain
    
    ANSWER:
    """;
```

### 5. Production Checklist

Before deploying RAG to production:

- [ ] Implement comprehensive error handling
- [ ] Add monitoring and metrics
- [ ] Set up caching for common queries
- [ ] Configure rate limiting
- [ ] Implement access controls
- [ ] Add input validation
- [ ] Test with realistic data volumes
- [ ] Document maintenance procedures
- [ ] Plan for data updates
- [ ] Set up alerting for failures

---

## Conclusion

You've now mastered RAG with Spring AI! Let's recap the key takeaways:

### Core Concepts

✅ RAG combines retrieval and generation for contextual answers  
✅ Vector embeddings enable semantic search  
✅ Proper chunking is critical for performance  
✅ Multiple vector stores support different use cases  

### Implementation

✅ Spring AI simplifies RAG with high-level abstractions  
✅ PgVector provides production-ready vector storage  
✅ Document processing pipelines enable automated indexing  
✅ QuestionAnswerAdvisor streamlines common patterns  

### Best Practices

✅ Test different chunk sizes for your use case  
✅ Implement caching for frequent queries  
✅ Monitor retrieval quality and latency  
✅ Validate answers to prevent hallucinations  
✅ Plan for data freshness and updates  

### Next Steps

To deepen your RAG expertise:

1. **Experiment** with different embedding models
2. **Optimize** chunk sizes for your domain
3. **Implement** advanced patterns like HyDE or multi-index RAG
4. **Monitor** production metrics and iterate
5. **Stay updated** on RAG research and techniques

### Resources

- Spring AI Documentation: [spring.io/projects/spring-ai](https://spring.io/projects/spring-ai)
- Vector Database Comparisons
- RAG Research Papers
- Community Examples and Patterns

RAG is revolutionizing how we build AI applications. With Spring AI, you have the tools to build production-ready RAG systems efficiently and reliably.

Happy building! 🚀