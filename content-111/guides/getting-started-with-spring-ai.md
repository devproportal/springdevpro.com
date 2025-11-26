---
title: "Getting Started with Spring AI: A Complete Step-by-Step Guide"
date: 2025-01-15T10:00:00Z
draft: false
author: "SpringDevPro Team"
featuredImage: "images/spring-ai-start-example.jpg"
tags: ["Spring AI", "RAG", "LLM", "Spring Boot"]
hasCodeExample: true  # 启用代码在线运行
codeSandboxId: "github/devproportal/spring-ai-start-example"  # CodeSandbox 仓库ID


title: "Spring Boot Complete Tutorial 2025"
date: 2025-11-25T10:00:00-05:00
draft: false
description: "Learn Spring Boot from scratch with practical examples, best practices, and real-world projects."
slug: "spring-boot-tutorial-2025"
tags: ["Spring Boot", "Java", "Tutorial"]
categories: ["Backend Development"]
series: ["Spring Boot Essentials"]
showAuthor: true
authors:
  - "springdevpro"
showTableOfContents: true
showReadingTime: true
showWordCount: true

---

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



```xml
 <!-- Dependency Management: Import Spring AI BOM (unifies versioning) -->

 <dependencyManagement>

     <dependencies>

         <dependency>

             <groupId>org.springframework.ai </groupId>

             <artifactId>spring-ai-bom </artifactId>

             <version>1.0.0 </version>

             <type>pom </type>

             <scope>import </scope>

         </dependency>

     </dependencies>

 </dependencyManagement>

 <!-- Core Dependencies -->

 <dependencies>

     <!-- Spring Web -->

     <dependency>

         <groupId>org.springframework.boot </groupId>

         <artifactId>spring-boot-starter-web </artifactId>

     </dependency>

     <!-- Spring AI OpenAI Starter (auto-configures ChatClient) -->

     <dependency>

         <groupId>org.springframework.ai </groupId>

         <artifactId>spring-ai-openai-spring-boot-starter </artifactId>

     </dependency>

     <!-- Lombok (Optional) -->

     <dependency>

         <groupId>org.projectlombok </groupId>

         <artifactId>lombok </artifactId>

         <optional>true </optional>

     </dependency>

 </dependencies>
```

#### 2.1.2 Snapshot/Milestone Versions (Before 1.0.0)

For development versions (e.g., `1.1.0-SNAPSHOT`), add Spring snapshot repositories:



```xml
 <repositories>

     <!-- Spring Snapshot Repository -->

     <repository>

         <id>spring-snapshots </id>

         <name>Spring Snapshots </name>

         <url>https://repo.spring.io/snapshot </url>

         <releases>

             <enabled>false </enabled>

         </releases>

     </repository>

     <!-- Central Portal Snapshot Repository -->

     <repository>

         <id>central-portal-snapshots </id>

         <name>Central Portal Snapshots </name>

         <url>https://central.sonatype.com/repository/maven-snapshots/ </url>

         <releases>

             <enabled>false </enabled>

         </releases>

         <snapshots>

             <enabled>true </enabled>

         </snapshots>

     </repository>

 </repositories>

 <!-- Update BOM version to snapshot -->

 <dependencyManagement>

     <dependencies>

         <dependency>

             <groupId>org.springframework.ai </groupId>

             <artifactId>spring-ai-bom </artifactId>

             <version>1.1.0-SNAPSHOT </version>

             <type>pom </type>

             <scope>import </scope>

         </dependency>

     </dependencies>

 </dependencyManagement>
```

### 2.2 Gradle (`build.gradle`)



```gradle
// Dependency Management: Import Spring AI BOM

dependencies {

    implementation platform("org.springframework.ai:spring-ai-bom:1.0.0")

    

    // Spring Web

    implementation 'org.springframework.boot:spring-boot-starter-web'

    

    // Spring AI OpenAI

    implementation 'org.springframework.ai:spring-ai-openai-spring-boot-starter'

    

    // Lombok (Optional)

    compileOnly 'org.projectlombok:lombok'

    annotationProcessor 'org.projectlombok:lombok'

}

// For Snapshot Versions (add this block)

repositories {

    mavenCentral()

    maven { url 'https://repo.spring.io/snapshot' }

    maven {

        name = 'Central Portal Snapshots'

        url = 'https://central.sonatype.com/repository/maven-snapshots/'

    }

}
```

## 3. Configure OpenAI API Key

Spring AI needs your OpenAI API Key to communicate with OpenAI’s services. Configure it in the `application.yml` (or `application.properties`) file.

### 3.1 `src/main/resources/application.yml`



```yaml
spring:

  ai:

    openai:

      api-key: "sk-your-openai-api-key-here"  # Replace with your actual key

      base-url: "https://api.openai.com/v1"   # Default OpenAI API endpoint

    chat:

      options:

        model: "gpt-4"  # Use "gpt-3.5-turbo" for lower cost

        temperature: 0.7  # Controls randomness (0 = precise, 1 = creative)
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



```java
package com.springdevpro.springai.service;

import org.springframework.ai.chat.ChatClient;

import org.springframework.stereotype.Service;

@Service

public class AIService {

    // ChatClient is auto-configured by Spring AI OpenAI Starter

    private final ChatClient chatClient;

    // Constructor injection (no @Autowired needed in Spring 4.3+)

    public AIService(ChatClient chatClient) {

        this.chatClient = chatClient;

    }

    /**

     * Send a single question to OpenAI and return the response.

     * @param question User's question

     * @return LLM-generated response

     */

    public String askSingleQuestion(String question) {

        // Use chatClient.call() for simple one-way calls

        return chatClient.call(question);

    }

}
```

#### 4.1.2 Controller Layer (`AIController.java`)



```java
package com.springdevpro.springai.controller;

import com.springdevpro.springai.service.AIService;

import org.springframework.web.bind.annotation.GetMapping;

import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.bind.annotation.RequestParam;

import org.springframework.web.bind.annotation.RestController;

@RestController

@RequestMapping("/api/ai")  // Base path for AI APIs

public class AIController {

    private final AIService aiService;

    public AIController(AIService aiService) {

        this.aiService = aiService;

    }

    /**

     * REST API for single-turn question-answering.

     * Example: http://localhost:8080/api/ai/ask?question=What is Spring AI?

     */

    @GetMapping("/ask")

    public String askQuestion(@RequestParam String question) {

        return aiService.askSingleQuestion(question);

    }

}
```

### 4.2 Feature 2: Multi-Turn Conversation (With Memory)

To enable the LLM to "remember" previous messages, use Spring AI’s `ChatMemory` (we’ll use `InMemoryChatMemory` for simplicity).

#### 4.2.1 Add Memory Dependency

First, add the `spring-ai-memory-in-memory` dependency (already included in the starter for 1.0.0+; if not, add manually):



```xml
 <!-- Maven (if missing) -->

 <dependency>

     <groupId>org.springframework.ai </groupId>

     <artifactId>spring-ai-memory-in-memory </artifactId>

 </dependency>
```



```java
// Gradle (if missing)

implementation 'org.springframework.ai:spring-ai-memory-in-memory'
```

#### 4.2.2 Update `AIService.java` for Multi-Turn



```java
package com.springdevpro.springai.service;

import org.springframework.ai.chat.ChatClient;

import org.springframework.ai.chat.memory.ChatMemory;

import org.springframework.ai.chat.memory.InMemoryChatMemory;

import org.springframework.ai.chat.prompt.Prompt;

import org.springframework.stereotype.Service;

@Service

public class AIService {

    private final ChatClient chatClient;

    private final ChatMemory chatMemory;  // Stores conversation history

    public AIService(ChatClient chatClient) {

        this.chatClient = chatClient;

        // Initialize in-memory chat memory (use Redis for distributed apps)

        this.chatMemory = new InMemoryChatMemory();

    }

    // ... (keep askSingleQuestion method)

    /**

     * Multi-turn conversation: remembers previous messages.

     * @param userInput User's new message

     * @return LLM response (considers history)

     */

    public String chatWithMemory(String userInput) {

        // 1. Add user's new input to memory

        Prompt userPrompt = new Prompt(userInput);

        chatMemory.add(userPrompt);

        // 2. Send full conversation history to LLM

        String llmResponse = chatClient.call(chatMemory.get());

        // 3. Add LLM's response to memory for next turns

        chatMemory.add(new Prompt(llmResponse));

        return llmResponse;

    }

}
```

#### 4.2.3 Add Controller Endpoint for Multi-Turn

Update `AIController.java`:



```java
@RestController

@RequestMapping("/api/ai")

public class AIController {

    // ... (keep existing code)

    /**

     * REST API for multi-turn conversation.

     * Example: http://localhost:8080/api/ai/chat?input=What is its core feature?

     */

    @GetMapping("/chat")

    public String chatWithMemory(@RequestParam String input) {

        return aiService.chatWithMemory(input);

    }

}
```

### 4.3 Feature 3: Structured Prompts (PromptTemplate)

Use `PromptTemplate` to create reusable, parameterized prompts (e.g., define a "role" for the LLM).

#### 4.3.1 Update `AIService.java` with `PromptTemplate`



```java
package com.springdevpro.springai.service;

import org.springframework.ai.chat.ChatClient;

import org.springframework.ai.chat.memory.ChatMemory;

import org.springframework.ai.chat.memory.InMemoryChatMemory;

import org.springframework.ai.chat.prompt.Prompt;

import org.springframework.ai.chat.prompt.PromptTemplate;

import org.springframework.stereotype.Service;

@Service

public class AIService {

    // ... (keep chatClient and chatMemory fields)

    // ... (keep existing methods)

    /**

     * Structured prompt: use a template to define roles/format.

     * @param role LLM's role (e.g., "senior Java architect")

     * @param question User's question

     * @return Formatted LLM response

     */

    public String askWithRole(String role, String question) {

        // Define a reusable prompt template

        String template = "You are a {role}. Answer the following question clearly and concisely: {question}";

        PromptTemplate promptTemplate = new PromptTemplate(template);

        // Fill in template parameters

        promptTemplate.add("role", role);

        promptTemplate.add("question", question);

        // Generate the final prompt and call LLM

        Prompt finalPrompt = promptTemplate.create();

        return chatClient.call(finalPrompt);

    }

}
```

#### 4.3.2 Add Controller Endpoint for Structured Prompts

Update `AIController.java`:



```java
@RestController

@RequestMapping("/api/ai")

public class AIController {

    // ... (keep existing code)

    /**

     * REST API for structured prompts (with role).

     * Example: http://localhost:8080/api/ai/ask-with-role?role=senior%20Java%20architect\&question=Explain%20Spring%20AI%20injection

     */

    @GetMapping("/ask-with-role")

    public String askWithRole(

            @RequestParam String role,

            @RequestParam String question

    ) {

        return aiService.askWithRole(role, question);

    }

}
```

## 5. Run and Test the Application

### 5.1 Start the Spring Boot App

Run the main class (e.g., `SpringAiDemoApplication.java`):



```java
package com.springdevpro.springai;

import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication

public class SpringAiDemoApplication {

    public static void main(String\[] args) {

        SpringApplication.run(SpringAiDemoApplication.class, args);

    }

}
```

### 5.2 Test the APIs

Use a browser, Postman, or `curl` to test the endpoints:

#### 1. Single-Turn Question



```bash
curl "http://localhost:8080/api/ai/ask?question=What is Spring AI?"
```

**Expected Response**:

`Spring AI is a library that integrates large language models (LLMs) into Spring Boot applications, simplifying LLM calls with intuitive APIs similar to Spring's data access patterns (e.g., JdbcTemplate). It supports models like OpenAI GPT-3.5/4, Hugging Face, and Azure OpenAI, and includes features for conversation memory, prompt templates, and vector database integration.`

#### 2. Multi-Turn Conversation

First call:



```bash
curl "http://localhost:8080/api/ai/chat?input=What is Spring AI?"
```

Second call (LLM remembers the first question):



```bash
curl "http://localhost:8080/api/ai/chat?input=How do I integrate it with OpenAI?"
```

#### 3. Structured Prompt with Role



```bash
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



```xml
 <mirror>
     <id>my-mirror </id>
     <mirrorOf>*,!spring-snapshots,!central-portal-snapshots </mirrorOf>
     <url>https://your-corporate-mirror.com/maven </url>
 </mirror>
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
