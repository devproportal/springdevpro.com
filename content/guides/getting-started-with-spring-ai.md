---
title: "Getting Started with Spring AI: Build Your First RAG Application"
date: 2025-01-15T10:00:00Z
author: "SpringDevPro Team"
featuredImage: "images/spring-ai-rag-example.jpg"
tags: ["Spring AI", "RAG", "LLM", "Spring Boot"]
hasCodeExample: true  # 启用代码在线运行
codeSandboxId: "github/devproportal/spring-ai-rag-example"  # CodeSandbox 仓库ID
---

Spring AI simplifies integrating LLMs (Large Language Models) and AI tools into Spring Boot applications. In this tutorial, you'll build a basic RAG (Retrieval-Augmented Generation) app using Spring AI, OpenAI, and Pinecone.

## Prerequisites
- Java 17+
- Spring Boot 3.2+
- OpenAI API Key (get from [OpenAI Platform](https://platform.openai.com/))
- Pinecone API Key and Index (create from [Pinecone Console](https://app.pinecone.io/))

## Step 1: Add Spring AI Dependencies
Add the following dependencies to your `pom.xml` (Maven) or `build.gradle` (Gradle):

```xml
<!-- Spring AI Core -->
<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-core</artifactId>
  <version>1.0.0-M1</version>
</dependency>

<!-- OpenAI Integration -->
<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-openai</artifactId>
  <version>1.0.0-M1</version>
</dependency>

<!-- Pinecone Vector DB -->
<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-pinecone</artifactId>
  <version>1.0.0-M1</version>
</dependency>

Step 2: Configure Application Properties
Add the following to src/main/resources/application.properties:
properties
# OpenAI Configuration
spring.ai.openai.api-key=sk-your-openai-api-key
spring.ai.openai.chat.model=gpt-3.5-turbo

# Pinecone Configuration
spring.ai.pinecone.api-key=your-pinecone-api-key
spring.ai.pinecone.environment=gcp-starter
spring.ai.pinecone.index-name=spring-ai-rag-demo
Step 3: Build the RAG Service
Create a RagService class to handle retrieval and generation:
java
运行
import org.springframework.ai.openai.OpenAiChatClient;
import org.springframework.ai.pinecone.PineconeVectorStore;
import org.springframework.ai.search.service.SearchService;
import org.springframework.stereotype.Service;

@Service
public class RagService {

  private final OpenAiChatClient chatClient;
  private final SearchService searchService;

  public RagService(OpenAiChatClient chatClient, PineconeVectorStore vectorStore) {
    this.chatClient = chatClient;
    this.searchService = new SearchService(vectorStore);
  }

  public String generateResponse(String userQuery) {
    // 1. Retrieve relevant documents from Pinecone
    var retrievedDocs = searchService.search(userQuery, 3);
    
    // 2. Build prompt with retrieved documents
    var prompt = String.format(
      "Answer the user query using the following context:\n%s\n\nUser Query: %s",
      retrievedDocs.getContent(), userQuery
    );
    
    // 3. Generate response with OpenAI
    return chatClient.call(prompt);
  }
}
Step 4: Create a REST Controller
Expose the RAG service via a REST endpoint:
java
运行
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RagController {

  private final RagService ragService;

  public RagController(RagService ragService) {
    this.ragService = ragService;
  }

  @GetMapping("/api/rag")
  public String getRagResponse(@RequestParam String query) {
    return ragService.generateResponse(query);
  }
}
Step 5: Test the Application
Run the Spring Boot app and test the endpoint with curl:
bash
运行
curl "http://localhost:8080/api/rag?query=What is Spring AI?"
You'll get a response generated using the retrieved context from Pinecone and OpenAI's LLM!
Next Steps
Add document ingestion (load PDFs/Markdown into Pinecone)
Implement prompt engineering for better responses
Add error handling and rate limiting
Want to run this code yourself? Use the CodeSandbox embed below to test the full project: