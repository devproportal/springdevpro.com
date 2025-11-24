---
title: "Getting Started with Spring AI: A Complete Step-by-Step Guide"
date: 2025-01-15T10:00:00Z
author: "SpringDevPro Team"
featuredImage: "images/spring-ai-start-example.jpg"
tags: ["Spring AI", "RAG", "LLM", "Spring Boot"]
hasCodeExample: true  # 启用代码在线运行
codeSandboxId: "github/devproportal/spring-ai-start-example"  # CodeSandbox 仓库ID
---

# Getting Started with Spring AI: A Complete Step-by-Step Guide

## Overview

Spring AI is a powerful library that integrates large language models (LLMs) like OpenAI, Hugging Face, and Azure OpenAI into Spring Boot applications. It simplifies LLM calls to be as intuitive as operating a database (similar to `JdbcTemplate`), enabling developers to quickly build intelligent features like chatbots, question-answering systems, and content generators.

## Prerequisites

Before starting, ensure your development environment meets the following requirements:



* **Java Version**: 17 or higher (Spring AI relies on Java 17+ features)

* **Spring Boot Version**: 3.2.x or higher (officially supports 3.4.x; 3.5.x will be supported after release)

* **Build Tool**: Maven 3.6.3+ or Gradle 5.0+

* **API Key**: An OpenAI API Key (obtain from [OpenAI Platform](https://platform.openai.com/api-keys))

## 1. Create a Spring Boot Project

Use **Spring Initializr** (the fastest way) to generate a project with the required dependencies:

### Step 1.1: Access Spring Initializr

Visit [start.spring.io](https://start.spring.io/) and configure the project as follows:



| Configuration Item  | Value                                     |
| ------------------- | ----------------------------------------- |
| Project Type        | Maven/Gradle (choose based on preference) |
| Language            | Java                                      |
| Spring Boot Version | 3.2.x or higher                           |
| Group               | com.springdevpro (customize as needed)    |
| Artifact            | spring-ai-demo                            |
| Name                | SpringAIDemo                              |
| Packaging           | Jar                                       |
| Java Version        | 17                                        |

### Step 1.2: Add Dependencies

Select the following dependencies (Spring Initializr will auto-include them):



* **Spring Web**: For building REST APIs

* **Spring AI OpenAI**: Core dependency for integrating OpenAI models

* **Lombok** (Optional): Reduces boilerplate code (e.g., getters/setters)

* **Spring Boot Actuator** (Optional): For monitoring application health

Click **Generate** to download the project zip, then extract it to your local directory.

## 2. Configure Dependencies

If you didn’t use Spring Initializr, manually add the Spring AI dependencies to your build file.

### 2.1 Maven (`pom.xml`)

#### 2.1.1 Stable Versions (1.0.0+) (Maven Central)

Stable releases are available in Maven Central (no extra repository configuration needed):



```
\<!-- Dependency Management: Import Spring AI BOM (unifies versioning) -->

\<dependencyManagement>

&#x20;   \<dependencies>

&#x20;       \<dependency>

&#x20;           \<groupId>org.springframework.ai\</groupId>

&#x20;           \<artifactId>spring-ai-bom\</artifactId>

&#x20;           \<version>1.0.0\</version>

&#x20;           \<type>pom\</type>

&#x20;           \<scope>import\</scope>

&#x20;       \</dependency>

&#x20;   \</dependencies>

\</dependencyManagement>

\<!-- Core Dependencies -->

\<dependencies>

&#x20;   \<!-- Spring Web -->

&#x20;   \<dependency>

&#x20;       \<groupId>org.springframework.boot\</groupId>

&#x20;       \<artifactId>spring-boot-starter-web\</artifactId>

&#x20;   \</dependency>

&#x20;   \<!-- Spring AI OpenAI Starter (auto-configures ChatClient) -->

&#x20;   \<dependency>

&#x20;       \<groupId>org.springframework.ai\</groupId>

&#x20;       \<artifactId>spring-ai-openai-spring-boot-starter\</artifactId>

&#x20;   \</dependency>

&#x20;   \<!-- Lombok (Optional) -->

&#x20;   \<dependency>

&#x20;       \<groupId>org.projectlombok\</groupId>

&#x20;       \<artifactId>lombok\</artifactId>

&#x20;       \<optional>true\</optional>

&#x20;   \</dependency>

\</dependencies>
```

#### 2.1.2 Snapshot/Milestone Versions (Before 1.0.0)

For development versions (e.g., `1.1.0-SNAPSHOT`), add Spring snapshot repositories:



```
\<repositories>

&#x20;   \<!-- Spring Snapshot Repository -->

&#x20;   \<repository>

&#x20;       \<id>spring-snapshots\</id>

&#x20;       \<name>Spring Snapshots\</name>

&#x20;       \<url>https://repo.spring.io/snapshot\</url>

&#x20;       \<releases>

&#x20;           \<enabled>false\</enabled>

&#x20;       \</releases>

&#x20;   \</repository>

&#x20;   \<!-- Central Portal Snapshot Repository -->

&#x20;   \<repository>

&#x20;       \<id>central-portal-snapshots\</id>

&#x20;       \<name>Central Portal Snapshots\</name>

&#x20;       \<url>https://central.sonatype.com/repository/maven-snapshots/\</url>

&#x20;       \<releases>

&#x20;           \<enabled>false\</enabled>

&#x20;       \</releases>

&#x20;       \<snapshots>

&#x20;           \<enabled>true\</enabled>

&#x20;       \</snapshots>

&#x20;   \</repository>

\</repositories>

\<!-- Update BOM version to snapshot -->

\<dependencyManagement>

&#x20;   \<dependencies>

&#x20;       \<dependency>

&#x20;           \<groupId>org.springframework.ai\</groupId>

&#x20;           \<artifactId>spring-ai-bom\</artifactId>

&#x20;           \<version>1.1.0-SNAPSHOT\</version>

&#x20;           \<type>pom\</type>

&#x20;           \<scope>import\</scope>

&#x20;       \</dependency>

&#x20;   \</dependencies>

\</dependencyManagement>
```

### 2.2 Gradle (`build.gradle`)



```
// Dependency Management: Import Spring AI BOM

dependencies {

&#x20;   implementation platform("org.springframework.ai:spring-ai-bom:1.0.0")

&#x20;  &#x20;

&#x20;   // Spring Web

&#x20;   implementation 'org.springframework.boot:spring-boot-starter-web'

&#x20;  &#x20;

&#x20;   // Spring AI OpenAI

&#x20;   implementation 'org.springframework.ai:spring-ai-openai-spring-boot-starter'

&#x20;  &#x20;

&#x20;   // Lombok (Optional)

&#x20;   compileOnly 'org.projectlombok:lombok'

&#x20;   annotationProcessor 'org.projectlombok:lombok'

}

// For Snapshot Versions (add this block)

repositories {

&#x20;   mavenCentral()

&#x20;   maven { url 'https://repo.spring.io/snapshot' }

&#x20;   maven {

&#x20;       name = 'Central Portal Snapshots'

&#x20;       url = 'https://central.sonatype.com/repository/maven-snapshots/'

&#x20;   }

}
```

## 3. Configure OpenAI API Key

Spring AI needs your OpenAI API Key to communicate with OpenAI’s services. Configure it in the `application.yml` (or `application.properties`) file.

### 3.1 `src/main/resources/application.yml`



```
spring:

&#x20; ai:

&#x20;   openai:

&#x20;     api-key: "sk-your-openai-api-key-here"  # Replace with your actual key

&#x20;     base-url: "https://api.openai.com/v1"   # Default OpenAI API endpoint

&#x20;   chat:

&#x20;     options:

&#x20;       model: "gpt-4"  # Use "gpt-3.5-turbo" for lower cost

&#x20;       temperature: 0.7  # Controls randomness (0 = precise, 1 = creative)
```

### 3.2 Alternative: Use a `.openai` File (For Spring CLI)

If you use Spring CLI (e.g., `spring ai add` commands), save the API Key in a `.openai` file in your home directory (`~/.openai` on macOS/Linux, `C:\Users\YourName\.openai` on Windows):



```
open\_ai\_api\_key=sk-your-openai-api-key-here
```

## 4. Implement Core Features

We’ll build three common features:



1. Basic LLM call (single-turn question-answering)

2. Multi-turn conversation (with memory)

3. Structured prompts (using `PromptTemplate`)

### 4.1 Feature 1: Basic LLM Call (Single-Turn)

Create a **service** to wrap `ChatClient` (Spring AI’s core interface for LLM calls) and a **controller** to expose a REST API.

#### 4.1.1 Service Layer (`AIService.java`)



```
package com.springdevpro.springai.service;

import org.springframework.ai.chat.ChatClient;

import org.springframework.stereotype.Service;

@Service

public class AIService {

&#x20;   // ChatClient is auto-configured by Spring AI OpenAI Starter

&#x20;   private final ChatClient chatClient;

&#x20;   // Constructor injection (no @Autowired needed in Spring 4.3+)

&#x20;   public AIService(ChatClient chatClient) {

&#x20;       this.chatClient = chatClient;

&#x20;   }

&#x20;   /\*\*

&#x20;    \* Send a single question to OpenAI and return the response.

&#x20;    \* @param question User's question

&#x20;    \* @return LLM-generated response

&#x20;    \*/

&#x20;   public String askSingleQuestion(String question) {

&#x20;       // Use chatClient.call() for simple one-way calls

&#x20;       return chatClient.call(question);

&#x20;   }

}
```

#### 4.1.2 Controller Layer (`AIController.java`)



```
package com.springdevpro.springai.controller;

import com.springdevpro.springai.service.AIService;

import org.springframework.web.bind.annotation.GetMapping;

import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.bind.annotation.RequestParam;

import org.springframework.web.bind.annotation.RestController;

@RestController

@RequestMapping("/api/ai")  // Base path for AI APIs

public class AIController {

&#x20;   private final AIService aiService;

&#x20;   public AIController(AIService aiService) {

&#x20;       this.aiService = aiService;

&#x20;   }

&#x20;   /\*\*

&#x20;    \* REST API for single-turn question-answering.

&#x20;    \* Example: http://localhost:8080/api/ai/ask?question=What is Spring AI?

&#x20;    \*/

&#x20;   @GetMapping("/ask")

&#x20;   public String askQuestion(@RequestParam String question) {

&#x20;       return aiService.askSingleQuestion(question);

&#x20;   }

}
```

### 4.2 Feature 2: Multi-Turn Conversation (With Memory)

To enable the LLM to "remember" previous messages, use Spring AI’s `ChatMemory` (we’ll use `InMemoryChatMemory` for simplicity).

#### 4.2.1 Add Memory Dependency

First, add the `spring-ai-memory-in-memory` dependency (already included in the starter for 1.0.0+; if not, add manually):



```
\<!-- Maven (if missing) -->

\<dependency>

&#x20;   \<groupId>org.springframework.ai\</groupId>

&#x20;   \<artifactId>spring-ai-memory-in-memory\</artifactId>

\</dependency>
```



```
// Gradle (if missing)

implementation 'org.springframework.ai:spring-ai-memory-in-memory'
```

#### 4.2.2 Update `AIService.java` for Multi-Turn



```
package com.springdevpro.springai.service;

import org.springframework.ai.chat.ChatClient;

import org.springframework.ai.chat.memory.ChatMemory;

import org.springframework.ai.chat.memory.InMemoryChatMemory;

import org.springframework.ai.chat.prompt.Prompt;

import org.springframework.stereotype.Service;

@Service

public class AIService {

&#x20;   private final ChatClient chatClient;

&#x20;   private final ChatMemory chatMemory;  // Stores conversation history

&#x20;   public AIService(ChatClient chatClient) {

&#x20;       this.chatClient = chatClient;

&#x20;       // Initialize in-memory chat memory (use Redis for distributed apps)

&#x20;       this.chatMemory = new InMemoryChatMemory();

&#x20;   }

&#x20;   // ... (keep askSingleQuestion method)

&#x20;   /\*\*

&#x20;    \* Multi-turn conversation: remembers previous messages.

&#x20;    \* @param userInput User's new message

&#x20;    \* @return LLM response (considers history)

&#x20;    \*/

&#x20;   public String chatWithMemory(String userInput) {

&#x20;       // 1. Add user's new input to memory

&#x20;       Prompt userPrompt = new Prompt(userInput);

&#x20;       chatMemory.add(userPrompt);

&#x20;       // 2. Send full conversation history to LLM

&#x20;       String llmResponse = chatClient.call(chatMemory.get());

&#x20;       // 3. Add LLM's response to memory for next turns

&#x20;       chatMemory.add(new Prompt(llmResponse));

&#x20;       return llmResponse;

&#x20;   }

}
```

#### 4.2.3 Add Controller Endpoint for Multi-Turn

Update `AIController.java`:



```
@RestController

@RequestMapping("/api/ai")

public class AIController {

&#x20;   // ... (keep existing code)

&#x20;   /\*\*

&#x20;    \* REST API for multi-turn conversation.

&#x20;    \* Example: http://localhost:8080/api/ai/chat?input=What is its core feature?

&#x20;    \*/

&#x20;   @GetMapping("/chat")

&#x20;   public String chatWithMemory(@RequestParam String input) {

&#x20;       return aiService.chatWithMemory(input);

&#x20;   }

}
```

### 4.3 Feature 3: Structured Prompts (PromptTemplate)

Use `PromptTemplate` to create reusable, parameterized prompts (e.g., define a "role" for the LLM).

#### 4.3.1 Update `AIService.java` with `PromptTemplate`



```
package com.springdevpro.springai.service;

import org.springframework.ai.chat.ChatClient;

import org.springframework.ai.chat.memory.ChatMemory;

import org.springframework.ai.chat.memory.InMemoryChatMemory;

import org.springframework.ai.chat.prompt.Prompt;

import org.springframework.ai.chat.prompt.PromptTemplate;

import org.springframework.stereotype.Service;

@Service

public class AIService {

&#x20;   // ... (keep chatClient and chatMemory fields)

&#x20;   // ... (keep existing methods)

&#x20;   /\*\*

&#x20;    \* Structured prompt: use a template to define roles/format.

&#x20;    \* @param role LLM's role (e.g., "senior Java architect")

&#x20;    \* @param question User's question

&#x20;    \* @return Formatted LLM response

&#x20;    \*/

&#x20;   public String askWithRole(String role, String question) {

&#x20;       // Define a reusable prompt template

&#x20;       String template = "You are a {role}. Answer the following question clearly and concisely: {question}";

&#x20;       PromptTemplate promptTemplate = new PromptTemplate(template);

&#x20;       // Fill in template parameters

&#x20;       promptTemplate.add("role", role);

&#x20;       promptTemplate.add("question", question);

&#x20;       // Generate the final prompt and call LLM

&#x20;       Prompt finalPrompt = promptTemplate.create();

&#x20;       return chatClient.call(finalPrompt);

&#x20;   }

}
```

#### 4.3.2 Add Controller Endpoint for Structured Prompts

Update `AIController.java`:



```
@RestController

@RequestMapping("/api/ai")

public class AIController {

&#x20;   // ... (keep existing code)

&#x20;   /\*\*

&#x20;    \* REST API for structured prompts (with role).

&#x20;    \* Example: http://localhost:8080/api/ai/ask-with-role?role=senior%20Java%20architect\&question=Explain%20Spring%20AI%20injection

&#x20;    \*/

&#x20;   @GetMapping("/ask-with-role")

&#x20;   public String askWithRole(

&#x20;           @RequestParam String role,

&#x20;           @RequestParam String question

&#x20;   ) {

&#x20;       return aiService.askWithRole(role, question);

&#x20;   }

}
```

## 5. Run and Test the Application

### 5.1 Start the Spring Boot App

Run the main class (e.g., `SpringAiDemoApplication.java`):



```
package com.springdevpro.springai;

import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication

public class SpringAiDemoApplication {

&#x20;   public static void main(String\[] args) {

&#x20;       SpringApplication.run(SpringAiDemoApplication.class, args);

&#x20;   }

}
```

### 5.2 Test the APIs

Use a browser, Postman, or `curl` to test the endpoints:

#### 1. Single-Turn Question



```
curl "http://localhost:8080/api/ai/ask?question=What is Spring AI?"
```

**Expected Response**:

`Spring AI is a library that integrates large language models (LLMs) into Spring Boot applications, simplifying LLM calls with intuitive APIs similar to Spring's data access patterns (e.g., JdbcTemplate). It supports models like OpenAI GPT-3.5/4, Hugging Face, and Azure OpenAI, and includes features for conversation memory, prompt templates, and vector database integration.`

#### 2. Multi-Turn Conversation

First call:



```
curl "http://localhost:8080/api/ai/chat?input=What is Spring AI?"
```

Second call (LLM remembers the first question):



```
curl "http://localhost:8080/api/ai/chat?input=How do I integrate it with OpenAI?"
```

#### 3. Structured Prompt with Role



```
curl "http://localhost:8080/api/ai/ask-with-role?role=senior%20Java%20architect\&question=Explain%20ChatClient%20in%20Spring%20AI"
```

## 6. Troubleshooting Common Issues

### 6.1 "API Key Not Found" Error



* Check if `application.yml` has the correct `spring.ai``.openai.api-key`.

* Ensure the `.openai` file (for Spring CLI) is in your home directory and has no typos.

### 6.2 "Maven Cannot Find Spring AI Dependencies"



* For stable versions: Verify Maven Central is enabled (it’s default in Maven/Gradle).

* For snapshots: Add the Spring snapshot repositories (see Section 2.1.2).

* If using a corporate Maven mirror: Modify `settings.xml` to exclude Spring repositories:



```
\<mirror>

&#x20;   \<id>my-mirror\</id>

&#x20;   \<mirrorOf>\*,!spring-snapshots,!central-portal-snapshots\</mirrorOf>

&#x20;   \<url>https://your-corporate-mirror.com/maven\</url>

\</mirror>
```

### 6.3 Slow LLM Responses



* Larger models (e.g., GPT-4) take 3-4 minutes for complex requests. Use GPT-3.5-turbo for faster results.

* Check OpenAI’s API status at [status.openai.com](https://status.openai.com/).

## 7. Next Steps



* **Vector Databases**: Integrate with Pinecone/Weaviate for retrieval-augmented generation (RAG).

* **Other Models**: Try Hugging Face (e.g., `spring-ai-huggingface`) or Azure OpenAI.

* **Spring CLI AI Commands**: Use `spring ai add "describe functionality"` to auto-generate code (see [Spring CLI AI Guide](https://docs.spring.io/spring-cli/reference/0.8/ai-guide.html)).

## References



1. [Official Spring AI Documentation](https://docs.spring.io/spring-ai/reference/getting-started.html)

2. [Spring AI OpenAI Starter](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-openai-spring-boot-starter)

3. [OpenAI API Key Setup](https://platform.openai.com/api-keys)

4. [Spring CLI AI Commands](https://docs.spring.io/spring-cli/reference/0.8/ai-guide.html)
