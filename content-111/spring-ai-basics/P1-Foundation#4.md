# OpenAI Integration with Spring AI: GPT-4, Embeddings & Streaming

**A Complete Guide to Building Production-Ready OpenAI Applications with Spring Boot**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Setup and Configuration](#setup-and-configuration)
3. [OpenAI Chat Models](#openai-chat-models)
4. [Working with GPT-4 and GPT-4o](#working-with-gpt-4-and-gpt-4o)
5. [Embeddings and Vector Search](#embeddings-and-vector-search)
6. [Streaming Responses](#streaming-responses)
7. [Function Calling](#function-calling)
8. [Vision Capabilities](#vision-capabilities)
9. [Advanced Features](#advanced-features)
10. [Production Best Practices](#production-best-practices)
11. [Cost Optimization](#cost-optimization)
12. [Conclusion](#conclusion)

---

## Introduction

OpenAI's GPT models have revolutionized how we build AI-powered applications. Spring AI provides first-class integration with OpenAI, making it seamless to add GPT-4, embeddings, and streaming capabilities to your Spring Boot applications.

### What You'll Learn

In this comprehensive guide, you'll master:

- **Complete OpenAI setup** in Spring Boot applications
- **GPT-4 and GPT-4o** usage patterns and best practices
- **Embeddings generation** for semantic search and RAG
- **Real-time streaming** for better user experience
- **Function calling** to extend GPT capabilities
- **Vision API** for image understanding
- **Production deployment** strategies and cost optimization

### Why Spring AI for OpenAI?

Spring AI offers significant advantages over using OpenAI's SDK directly:

```java
// Direct OpenAI SDK (verbose, manual setup)
OpenAiService service = new OpenAiService("api-key");
CompletionRequest request = CompletionRequest.builder()
        .model("gpt-4")
        .prompt("Hello")
        .build();
CompletionResult result = service.createCompletion(request);
String response = result.getChoices().get(0).getText();

// Spring AI (concise, auto-configured)
@Autowired ChatClient chatClient;
String response = chatClient.prompt("Hello").call().content();
```

**Benefits**:
- ✅ **Auto-configuration**: Zero boilerplate setup
- ✅ **Abstraction**: Switch providers without code changes
- ✅ **Observability**: Built-in metrics and tracing
- ✅ **Resilience**: Automatic retries and circuit breakers
- ✅ **Spring Integration**: Works seamlessly with Spring ecosystem

Let's dive in!

---

## Setup and Configuration

### Dependencies

Add Spring AI OpenAI starter to your `pom.xml`:

```xml
<project>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <properties>
        <spring-ai.version>1.0.0-M3</spring-ai.version>
    </properties>

    <dependencies>
        <!-- Spring AI OpenAI Starter -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
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

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>
</project>
```

**For Gradle**:

```gradle
plugins {
    id 'org.springframework.boot' version '3.2.0'
}

ext {
    springAiVersion = '1.0.0-M3'
}

dependencies {
    implementation 'org.springframework.ai:spring-ai-openai-spring-boot-starter'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.ai:spring-ai-bom:${springAiVersion}"
    }
}

repositories {
    mavenCentral()
    maven { url 'https://repo.spring.io/milestone' }
}
```

### Configuration

**application.yml**:

```yaml
spring:
  application:
    name: openai-demo
  
  ai:
    openai:
      # API Key - NEVER commit this to version control
      api-key: ${OPENAI_API_KEY}
      
      # Base URL (default: https://api.openai.com)
      # Override for Azure OpenAI or custom proxies
      base-url: https://api.openai.com
      
      # Chat Model Configuration
      chat:
        enabled: true
        options:
          # Model selection
          model: gpt-4o
          
          # Temperature (0.0 - 2.0)
          # Lower = more deterministic, Higher = more creative
          temperature: 0.7
          
          # Maximum tokens in response
          max-tokens: 2000
          
          # Top-p sampling (alternative to temperature)
          top-p: 1.0
          
          # Frequency penalty (-2.0 to 2.0)
          # Positive values penalize repeated tokens
          frequency-penalty: 0.0
          
          # Presence penalty (-2.0 to 2.0)
          # Positive values encourage topic diversity
          presence-penalty: 0.0
          
          # Stop sequences
          stop:
            - "\n\n\n"
        
        # Retry configuration
        retry:
          max-attempts: 10
          backoff:
            initial-interval: 2s
            multiplier: 5
            max-interval: 3m
      
      # Embedding Model Configuration
      embedding:
        enabled: true
        options:
          model: text-embedding-3-small
          # Dimensions (for text-embedding-3-* models)
          dimensions: 1536
      
      # Image Generation Configuration
      image:
        enabled: true
        options:
          model: dall-e-3
          quality: standard  # standard or hd
          size: 1024x1024
          style: vivid       # vivid or natural

# Logging
logging:
  level:
    org.springframework.ai.openai: DEBUG
```

### Environment Variables

**Production-safe API key management**:

```bash
# .env file (add to .gitignore!)
OPENAI_API_KEY=sk-proj-...your-key-here...

# Or use Spring Cloud Config, Vault, AWS Secrets Manager, etc.
```

**Loading environment variables**:

```java
@Configuration
public class OpenAiConfig {
    
    @Value("${OPENAI_API_KEY:}")
    private String apiKey;
    
    @PostConstruct
    public void validateConfig() {
        if (apiKey == null || apiKey.isEmpty()) {
            throw new IllegalStateException(
                "OPENAI_API_KEY environment variable is required"
            );
        }
    }
}
```

### Verify Setup

Create a simple test:

```java
@SpringBootTest
class OpenAiIntegrationTest {
    
    @Autowired
    private ChatClient chatClient;
    
    @Test
    void testOpenAiConnection() {
        String response = chatClient.prompt()
                .user("Say 'Hello Spring AI!'")
                .call()
                .content();
        
        assertThat(response).containsIgnoringCase("hello");
        assertThat(response).containsIgnoringCase("spring");
    }
}
```

Run the test to verify your OpenAI integration is working.

---

## OpenAI Chat Models

### Available Models

OpenAI offers several models with different capabilities and pricing:

| Model | Context Window | Best For | Cost (Per 1M tokens) |
|-------|---------------|----------|---------------------|
| **gpt-4o** | 128K | Latest, fastest GPT-4 level | $5 / $15 (in/out) |
| **gpt-4o-mini** | 128K | Fast, cost-effective | $0.15 / $0.60 |
| **gpt-4-turbo** | 128K | Previous flagship | $10 / $30 |
| **gpt-3.5-turbo** | 16K | Legacy, cheapest | $0.50 / $1.50 |
| **o1-preview** | 128K | Advanced reasoning | $15 / $60 |
| **o1-mini** | 128K | Fast reasoning | $3 / $12 |

**Model Selection Strategy**:

```java
@Service
public class ModelSelectionService {
    
    public String selectModel(TaskComplexity complexity, int budgetCents) {
        return switch (complexity) {
            case SIMPLE -> {
                // Quick Q&A, summaries
                yield budgetCents < 1 ? "gpt-4o-mini" : "gpt-3.5-turbo";
            }
            case MODERATE -> {
                // Code generation, analysis
                yield "gpt-4o-mini";
            }
            case COMPLEX -> {
                // Complex reasoning, long-form content
                yield budgetCents > 10 ? "gpt-4o" : "gpt-4o-mini";
            }
            case REASONING -> {
                // Math, logic, planning
                yield "o1-mini";
            }
        };
    }
    
    enum TaskComplexity { SIMPLE, MODERATE, COMPLEX, REASONING }
}
```

### Basic Chat Interaction

**Simple Request**:

```java
@RestController
@RequestMapping("/api/chat")
public class ChatController {
    
    private final ChatClient chatClient;
    
    @PostMapping("/simple")
    public String simpleChat(@RequestBody String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .content();
    }
}
```

**With System Message**:

```java
@Service
public class CustomerSupportService {
    
    private final ChatClient chatClient;
    
    public CustomerSupportService(ChatClient.Builder builder) {
        this.chatClient = builder
                .defaultSystem("""
                        You are a helpful customer support agent for Acme Corp.
                        Be professional, friendly, and concise.
                        If you don't know something, say so honestly.
                        """)
                .build();
    }
    
    public String handleQuery(String query) {
        return chatClient.prompt()
                .user(query)
                .call()
                .content();
    }
}
```

### Conversation Context

Maintain multi-turn conversations:

```java
@Service
public class ConversationService {
    
    private final ChatClient chatClient;
    
    public String chat(String userId, String message) {
        // Get conversation history
        List<Message> history = getConversationHistory(userId);
        
        // Add new user message
        history.add(new UserMessage(message));
        
        // Create prompt with full context
        Prompt prompt = new Prompt(history);
        
        // Get response
        ChatResponse response = chatClient.prompt(prompt)
                .call()
                .chatResponse();
        
        AssistantMessage assistantMessage = response.getResult()
                .getOutput();
        
        // Save to history
        history.add(assistantMessage);
        saveConversationHistory(userId, history);
        
        return assistantMessage.getContent();
    }
    
    private List<Message> getConversationHistory(String userId) {
        // Load from database, Redis, etc.
        return conversationRepository.findByUserId(userId);
    }
    
    private void saveConversationHistory(String userId, List<Message> history) {
        conversationRepository.save(userId, history);
    }
}
```

**Using ChatMemory Advisor** (simpler approach):

```java
@Configuration
public class ChatConfig {
    
    @Bean
    public ChatClient chatClientWithMemory(
            ChatClient.Builder builder,
            ChatMemory chatMemory) {
        
        return builder
                .defaultAdvisors(
                        new MessageChatMemoryAdvisor(chatMemory)
                )
                .build();
    }
    
    @Bean
    public ChatMemory chatMemory() {
        // In-memory storage (use Redis/DB for production)
        return new InMemoryChatMemory();
    }
}
```

Now conversations automatically maintain context:

```java
// First message
String response1 = chatClient.prompt()
        .user("My name is Alice")
        .call()
        .content();
// "Nice to meet you, Alice!"

// Second message - context is remembered
String response2 = chatClient.prompt()
        .user("What's my name?")
        .call()
        .content();
// "Your name is Alice."
```

---

## Working with GPT-4 and GPT-4o

### GPT-4o: The Latest and Greatest

GPT-4o (released May 2024) is OpenAI's most advanced model:

**Key Features**:
- ⚡ **2x faster** than GPT-4 Turbo
- 💰 **50% cheaper** than GPT-4 Turbo
- 🧠 **128K context window** (500 pages of text)
- 👁️ **Vision capabilities** built-in
- 📊 **Superior performance** on benchmarks

**Configuration**:

```yaml
spring:
  ai:
    openai:
      chat:
        options:
          model: gpt-4o
          max-tokens: 4096
          temperature: 0.7
```

### GPT-4o Use Cases

**1. Code Generation**:

```java
@Service
public class CodeGenerationService {
    
    private final ChatClient chatClient;
    
    public CodeGenerationService(ChatClient.Builder builder) {
        this.chatClient = builder
                .defaultOptions(ChatOptions.builder()
                        .withModel("gpt-4o")
                        .withTemperature(0.3)  // More deterministic for code
                        .build())
                .build();
    }
    
    public String generateCode(String description) {
        String prompt = """
                Generate production-ready Java Spring Boot code for:
                
                %s
                
                Requirements:
                - Follow best practices
                - Include JavaDoc comments
                - Add error handling
                - Use modern Java features
                
                Return only the code, no explanations.
                """.formatted(description);
        
        return chatClient.prompt(prompt).call().content();
    }
}
```

**Example**:

```java
String code = service.generateCode(
    "A REST controller for user registration with email validation"
);

/*
Output:
@RestController
@RequestMapping("/api/users")
@Validated
public class UserRegistrationController {
    
    private final UserService userService;
    
    /**
     * Register a new user.
     * @param request User registration request
     * @return Created user details
     */
    @PostMapping("/register")
    public ResponseEntity<UserResponse> registerUser(
            @Valid @RequestBody RegistrationRequest request) {
        
        User user = userService.register(request);
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(new UserResponse(user));
    }
    
    // ... (full implementation)
}
*/
```

**2. Complex Data Analysis**:

```java
@Service
public class DataAnalysisService {
    
    private final ChatClient chatClient;
    
    public record AnalysisResult(
            String summary,
            List<String> keyFindings,
            List<String> recommendations,
            Map<String, Object> metrics
    ) {}
    
    public AnalysisResult analyzeData(String csvData) {
        String prompt = """
                Analyze the following sales data and provide insights:
                
                %s
                
                Provide:
                1. Executive summary
                2. Top 3 key findings
                3. Actionable recommendations
                4. Key metrics (revenue, growth rate, top products)
                
                Return as JSON matching the AnalysisResult structure.
                """.formatted(csvData);
        
        return chatClient.prompt(prompt)
                .call()
                .entity(AnalysisResult.class);
    }
}
```

**3. Long Document Processing**:

GPT-4o's 128K context window handles massive documents:

```java
@Service
public class DocumentSummarizationService {
    
    private final ChatClient chatClient;
    
    public String summarizeLongDocument(String document) {
        // GPT-4o can handle up to ~96,000 words in one request
        
        String prompt = """
                Summarize this %d-word document in 500 words or less.
                
                Focus on:
                - Main arguments
                - Key findings
                - Conclusions
                - Recommendations
                
                Document:
                %s
                """.formatted(
                        document.split("\\s+").length,
                        document
                );
        
        return chatClient.prompt()
                .user(prompt)
                .options(ChatOptions.builder()
                        .withModel("gpt-4o")
                        .withMaxTokens(1000)
                        .build())
                .call()
                .content();
    }
}
```

### Temperature and Creativity Control

Temperature affects response randomness:

```java
@Service
public class TemperatureDemo {
    
    private final ChatClient chatClient;
    
    /**
     * Temperature 0.0 - Deterministic, factual
     * Use for: Math, code, factual Q&A
     */
    public String deterministicResponse(String question) {
        return chatClient.prompt()
                .user(question)
                .options(ChatOptions.builder()
                        .withTemperature(0.0)
                        .build())
                .call()
                .content();
    }
    
    /**
     * Temperature 0.7 - Balanced (default)
     * Use for: General conversations, explanations
     */
    public String balancedResponse(String question) {
        return chatClient.prompt()
                .user(question)
                .options(ChatOptions.builder()
                        .withTemperature(0.7)
                        .build())
                .call()
                .content();
    }
    
    /**
     * Temperature 1.5 - Creative, diverse
     * Use for: Creative writing, brainstorming
     */
    public String creativeResponse(String prompt) {
        return chatClient.prompt()
                .user(prompt)
                .options(ChatOptions.builder()
                        .withTemperature(1.5)
                        .build())
                .call()
                .content();
    }
}
```

**Example - Math Problem**:

```java
String question = "What is 127 * 83?";

// Temperature 0.0 - Always correct
deterministicResponse(question);  // "10,541"

// Temperature 1.5 - May vary or be wrong
creativeResponse(question);       // "10,541" or "10,542" or explanations
```

### Token Management

Control response length:

```java
@Service
public class TokenControlService {
    
    /**
     * Generate concise responses (tweets, notifications)
     */
    public String generateTweet(String topic) {
        return chatClient.prompt()
                .user("Write a tweet about: " + topic)
                .options(ChatOptions.builder()
                        .withMaxTokens(60)  // ~45 words, under 280 chars
                        .build())
                .call()
                .content();
    }
    
    /**
     * Generate detailed responses (blog posts, documentation)
     */
    public String generateBlogPost(String topic) {
        return chatClient.prompt()
                .user("Write a detailed blog post about: " + topic)
                .options(ChatOptions.builder()
                        .withMaxTokens(2000)  // ~1500 words
                        .build())
                .call()
                .content();
    }
    
    /**
     * Estimate tokens before calling API
     */
    public int estimateTokens(String text) {
        // Rough estimate: 1 token ≈ 4 characters (English)
        return text.length() / 4;
    }
}
```

### Stop Sequences

Control when generation stops:

```java
String prompt = """
        Generate a Python function to calculate fibonacci numbers.
        
        ```python
        """;

String code = chatClient.prompt()
        .user(prompt)
        .options(ChatOptions.builder()
                .withStop(List.of("```"))  // Stop at code block end
                .build())
        .call()
        .content();

// Output: Just the Python code, stops at ```
```

---

## Embeddings and Vector Search

OpenAI's embedding models convert text to vectors for semantic search.

### Embedding Models

| Model | Dimensions | Cost (per 1M tokens) | Best For |
|-------|-----------|---------------------|----------|
| **text-embedding-3-small** | 512-1536 | $0.02 | Most use cases |
| **text-embedding-3-large** | 256-3072 | $0.13 | Highest quality |
| **text-embedding-ada-002** | 1536 | $0.10 | Legacy (deprecated) |

**Configuration**:

```yaml
spring:
  ai:
    openai:
      embedding:
        options:
          model: text-embedding-3-small
          dimensions: 1536  # 512, 1024, or 1536
```

### Generating Embeddings

```java
@Service
public class EmbeddingService {
    
    private final EmbeddingModel embeddingModel;
    
    /**
     * Generate embedding for single text.
     */
    public float[] embedText(String text) {
        return embeddingModel.embed(text);
    }
    
    /**
     * Generate embeddings for multiple texts (batched).
     */
    public List<float[]> embedTexts(List<String> texts) {
        return embeddingModel.embed(texts);
    }
    
    /**
     * Get embedding dimensions.
     */
    public int getDimensions() {
        return embeddingModel.dimensions();
    }
}
```

### Vector Store Integration

**Using PostgreSQL with pgvector**:

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
</dependency>
```

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/vectordb
    username: postgres
    password: postgres
  
  ai:
    vectorstore:
      pgvector:
        index-type: HNSW
        distance-type: COSINE_DISTANCE
        dimensions: 1536
```

**Complete RAG Implementation**:

```java
@Service
public class SemanticSearchService {
    
    private final VectorStore vectorStore;
    private final EmbeddingModel embeddingModel;
    private final ChatClient chatClient;
    
    /**
     * Index documents for semantic search.
     */
    public void indexDocuments(List<String> documents) {
        List<Document> docs = documents.stream()
                .map(content -> new Document(content, Map.of(
                        "indexed_at", Instant.now().toString(),
                        "source", "user_upload"
                )))
                .toList();
        
        vectorStore.add(docs);
        
        log.info("Indexed {} documents", docs.size());
    }
    
    /**
     * Search similar documents.
     */
    public List<Document> search(String query, int topK) {
        return vectorStore.similaritySearch(
                SearchRequest.query(query)
                        .withTopK(topK)
                        .withSimilarityThreshold(0.7)
        );
    }
    
    /**
     * RAG: Retrieval-Augmented Generation
     */
    public String answerQuestion(String question) {
        // 1. Retrieve relevant documents
        List<Document> relevantDocs = search(question, 3);
        
        if (relevantDocs.isEmpty()) {
            return "I don't have enough information to answer this question.";
        }
        
        // 2. Build context
        String context = relevantDocs.stream()
                .map(Document::getContent)
                .collect(Collectors.joining("\n\n---\n\n"));
        
        // 3. Generate answer with context
        String prompt = """
                Answer the question based on the following context.
                If the answer is not in the context, say "I don't know."
                
                Context:
                %s
                
                Question: %s
                
                Answer:
                """.formatted(context, question);
        
        return chatClient.prompt(prompt).call().content();
    }
}
```

### Cosine Similarity Calculation

Calculate similarity between vectors manually:

```java
public class VectorUtils {
    
    /**
     * Calculate cosine similarity between two vectors.
     * Returns value between -1 (opposite) and 1 (identical).
     */
    public static double cosineSimilarity(float[] a, float[] b) {
        if (a.length != b.length) {
            throw new IllegalArgumentException("Vectors must have same length");
        }
        
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
    
    /**
     * Find most similar text from a list.
     */
    public static String findMostSimilar(
            String query,
            List<String> candidates,
            EmbeddingModel embeddingModel) {
        
        float[] queryEmbedding = embeddingModel.embed(query);
        List<float[]> candidateEmbeddings = embeddingModel.embed(candidates);
        
        double maxSimilarity = -1.0;
        String mostSimilar = null;
        
        for (int i = 0; i < candidates.size(); i++) {
            double similarity = cosineSimilarity(
                    queryEmbedding,
                    candidateEmbeddings.get(i)
            );
            
            if (similarity > maxSimilarity) {
                maxSimilarity = similarity;
                mostSimilar = candidates.get(i);
            }
        }
        
        return mostSimilar;
    }
}
```

### Semantic Caching

Cache responses based on semantic similarity:

```java
@Service
public class SemanticCacheService {
    
    private final VectorStore cacheStore;
    private final EmbeddingModel embeddingModel;
    private final ChatClient chatClient;
    
    /**
     * Get response with semantic caching.
     */
    public String getCachedResponse(String query, double similarityThreshold) {
        // 1. Search for similar queries in cache
        List<Document> similar = cacheStore.similaritySearch(
                SearchRequest.query(query)
                        .withTopK(1)
                        .withSimilarityThreshold(similarityThreshold)
        );
        
        // 2. If found, return cached response
        if (!similar.isEmpty()) {
            Document cached = similar.get(0);
            log.info("Cache hit for query: {}", query);
            return cached.getMetadata("response");
        }
        
        // 3. Generate new response
        log.info("Cache miss for query: {}", query);
        String response = chatClient.prompt(query).call().content();
        
        // 4. Cache the response
        Document cacheEntry = new Document(query, Map.of(
                "response", response,
                "timestamp", Instant.now().toString()
        ));
        cacheStore.add(List.of(cacheEntry));
        
        return response;
    }
}
```

Usage:

```java
// First call - cache miss, calls OpenAI
String response1 = service.getCachedResponse(
    "What is machine learning?",
    0.9  // 90% similarity threshold
);

// Similar query - cache hit, no API call!
String response2 = service.getCachedResponse(
    "Can you explain machine learning?",
    0.9
);
// Returns same response as response1
```

---

## Streaming Responses

Streaming provides a better user experience for long responses.

### Basic Streaming

```java
@Service
public class StreamingService {
    
    private final ChatClient chatClient;
    
    /**
     * Stream response as Flux<String>.
     */
    public Flux<String> streamResponse(String prompt) {
        return chatClient.prompt()
                .user(prompt)
                .stream()
                .content();
    }
}
```

### Server-Sent Events (SSE) Controller

Stream to web clients:

```java
@RestController
@RequestMapping("/api/chat")
public class StreamingChatController {
    
    private final ChatClient chatClient;
    
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<String>> streamChat(
            @RequestParam String message) {
        
        return chatClient.prompt()
                .user(message)
                .stream()
                .content()
                .map(chunk -> ServerSentEvent.<String>builder()
                        .data(chunk)
                        .build());
    }
    
    @GetMapping(value = "/stream-complete", 
                produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<ChatMessage>> streamWithCompletion(
            @RequestParam String message) {
        
        return chatClient.prompt()
                .user(message)
                .stream()
                .content()
                .map(chunk -> ServerSentEvent.<ChatMessage>builder()
                        .data(new ChatMessage(chunk, false))
                        .build())
                .concatWith(Mono.just(
                        ServerSentEvent.<ChatMessage>builder()
                                .data(new ChatMessage("", true))
                                .event("done")
                                .build()
                ));
    }
    
    record ChatMessage(String content, boolean done) {}
}
```

### Frontend Integration

**JavaScript (vanilla)**:

```javascript
const eventSource = new EventSource(
    '/api/chat/stream?message=' + encodeURIComponent(userMessage)
);

const responseDiv = document.getElementById('response');
responseDiv.innerHTML = '';

eventSource.onmessage = (event) => {
    const chunk = event.data;
    responseDiv.innerHTML += chunk;
};

eventSource.addEventListener('done', () => {
    eventSource.close();
    console.log('Stream complete');
});

eventSource.onerror = (error) => {
    console.error('Stream error:', error);
    eventSource.close();
};
```

**React**:

```jsx
import { useEffect, useState } from 'react';

function StreamingChat() {
    const [response, setResponse] = useState('');
    const [loading, setLoading] = useState(false);
    
    const sendMessage = async (message) => {
        setResponse('');
        setLoading(true);
        
        const eventSource = new EventSource(
            `/api/chat/stream?message=${encodeURIComponent(message)}`
        );
        
        eventSource.onmessage = (event) => {
            setResponse(prev => prev + event.data);
        };
        
        eventSource.addEventListener('done', () => {
            eventSource.close();
            setLoading(false);
        });
        
        eventSource.onerror = () => {
            eventSource.close();
            setLoading(false);
        };
    };
    
    return (
        <div>
            <div className="response">
                {response}
                {loading && <span className="cursor">▊</span>}
            </div>
        </div>
    );
}
```

### Streaming with Backpressure

Handle slow consumers:

```java
@Service
public class BackpressureStreamingService {
    
    private final ChatClient chatClient;
    
    /**
     * Stream with backpressure control.
     */
    public Flux<String> streamWithBackpressure(String prompt) {
        return chatClient.prompt()
                .user(prompt)
                .stream()
                .content()
                // Buffer chunks to smooth delivery
                .bufferTimeout(5, Duration.ofMillis(100))
                .map(chunks -> String.join("", chunks))
                // Limit rate to prevent overwhelming client
                .limitRate(10)  // Max 10 chunks buffered
                // Handle errors
                .onErrorResume(error -> {
                    log.error("Streaming error", error);
                    return Flux.just("[Error: " + error.getMessage() + "]");
                });
    }
}
```

### Streaming to File

Save long responses to file:

```java
@Service
public class FileStreamingService {
    
    private final ChatClient chatClient;
    
    /**
     * Stream response directly to file.
     */
    public Path streamToFile(String prompt, Path outputPath) {
        try (BufferedWriter writer = Files.newBufferedWriter(
                outputPath, 
                StandardCharsets.UTF_8)) {
            
            chatClient.prompt()
                    .user(prompt)
                    .stream()
                    .content()
                    .doOnNext(chunk -> {
                        try {
                            writer.write(chunk);
                            writer.flush();
                        } catch (IOException e) {
                            throw new UncheckedIOException(e);
                        }
                    })
                    .blockLast();  // Wait for completion
            
            return outputPath;
            
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }
}
```

---

## Function Calling

Let GPT-4o call your Java methods to access real-time data or perform actions.

### Defining Functions

```java
@Component
public class WeatherFunctions {
    
    @Bean
    @Description("Get current weather for a location")
    public Function<WeatherRequest, WeatherResponse> getCurrentWeather() {
        return request -> {
            // Call actual weather API
            WeatherData data = weatherApi.getCurrentWeather(
                    request.location(),
                    request.unit()
            );
            
            return new WeatherResponse(
                    request.location(),
                    data.temperature(),
                    data.condition(),
                    data.humidity(),
                    data.windSpeed()
            );
        };
    }
    
    @Bean
    @Description("Get weather forecast for next N days")
    public Function<ForecastRequest, ForecastResponse> getWeatherForecast() {
        return request -> {
            List<DailyForecast> forecast = weatherApi.getForecast(
                    request.location(),
                    request.days()
            );
            
            return new ForecastResponse(
                    request.location(),
                    forecast
            );
        };
    }
    
    record WeatherRequest(String location, String unit) {}
    
    record WeatherResponse(
            String location,
            double temperature,
            String condition,
            int humidity,
            double windSpeed
    ) {}
    
    record ForecastRequest(String location, int days) {}
    
    record ForecastResponse(
            String location,
            List<DailyForecast> forecast
    ) {}
    
    record DailyForecast(
            String date,
            double highTemp,
            double lowTemp,
            String condition
    ) {}
}
```

### Using Functions

```java
@Service
public class WeatherAssistant {
    
    private final ChatClient chatClient;
    
    public WeatherAssistant(ChatClient.Builder builder) {
        this.chatClient = builder
                .defaultFunctions("getCurrentWeather", "getWeatherForecast")
                .build();
    }
    
    public String chat(String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .content();
    }
}
```

**Example conversation**:

```java
// User: "What's the weather in Paris?"

// [Behind the scenes]
// 1. GPT-4o decides to call getCurrentWeather
// 2. Spring AI executes: getCurrentWeather({location: "Paris", unit: "celsius"})
// 3. Function returns: {temperature: 18, condition: "partly cloudy", ...}
// 4. GPT-4o generates natural response

assistant.chat("What's the weather in Paris?");
// Output: "The current weather in Paris is partly cloudy with a 
//          temperature of 18°C. Humidity is at 65% with light winds."

// User: "How about the next 3 days?"

// [Remembers context + calls getWeatherForecast]
assistant.chat("How about the next 3 days?");
// Output: "The weather forecast for Paris over the next 3 days shows:
//          - Tomorrow: Sunny, high of 20°C, low of 14°C
//          - Day after: Cloudy, high of 19°C, low of 13°C  
//          - Third day: Rain, high of 17°C, low of 12°C"
```

### Complex Function Example

Database query function:

```java
@Component
public class DatabaseFunctions {
    
    private final JdbcTemplate jdbcTemplate;
    
    @Bean
    @Description("Execute SQL SELECT query and return results")
    public Function<SqlQueryRequest, QueryResult> executeSqlQuery() {
        return request -> {
            // Validate query (security!)
            if (!isSafeQuery(request.sql())) {
                throw new SecurityException("Unsafe SQL query");
            }
            
            List<Map<String, Object>> rows = jdbcTemplate.queryForList(
                    request.sql()
            );
            
            return new QueryResult(
                    rows.size(),
                    rows
            );
        };
    }
    
    private boolean isSafeQuery(String sql) {
        String normalized = sql.toLowerCase().trim();
        
        // Only allow SELECT queries
        if (!normalized.startsWith("select")) {
            return false;
        }
        
        // Block dangerous keywords
        String[] dangerousKeywords = {
                "drop", "delete", "update", "insert",
                "alter", "create", "truncate", "exec"
        };
        
        for (String keyword : dangerousKeywords) {
            if (normalized.contains(keyword)) {
                return false;
            }
        }
        
        return true;
    }
    
    record SqlQueryRequest(String sql) {}
    record QueryResult(int rowCount, List<Map<String, Object>> rows) {}
}
```

**Natural language to SQL**:

```java
String question = "Show me the top 5 customers by total purchases";

String response = chatClient.prompt()
        .user(question)
        .functions("executeSqlQuery")
        .call()
        .content();

// GPT-4o:
// 1. Generates SQL: SELECT customer_name, SUM(amount) as total 
//                   FROM purchases GROUP BY customer_name 
//                   ORDER BY total DESC LIMIT 5
// 2. Calls executeSqlQuery
// 3. Formats results into natural language

// Output: "The top 5 customers by total purchases are:
//          1. Alice Johnson - $12,450
//          2. Bob Smith - $9,820
//          ..."
```

### Function Calling Best Practices

**1. Input Validation**:

```java
@Bean
public Function<TransferRequest, TransferResult> transferMoney() {
    return request -> {
        // Validate inputs
        if (request.amount() <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        
        if (request.amount() > 10000) {
            throw new IllegalArgumentException("Amount exceeds limit");
        }
        
        // Execute transfer
        return bankingService.transfer(request);
    };
}
```

**2. Error Handling**:

```java
@Bean
public Function<EmailRequest, EmailResult> sendEmail() {
    return request -> {
        try {
            emailService.send(request);
            return new EmailResult(true, "Email sent successfully");
            
        } catch (Exception e) {
            log.error("Failed to send email", e);
            return new EmailResult(false, "Failed: " + e.getMessage());
        }
    };
}
```

**3. Async Functions**:

```java
@Bean
public Function<ReportRequest, ReportResult> generateReport() {
    return request -> {
        // Start async report generation
        String jobId = reportService.generateAsync(request);
        
        return new ReportResult(
                jobId,
                "Report generation started. Check status with job ID."
        );
    };
}
```

---

继续完成剩余章节...

## Vision Capabilities

GPT-4o has built-in vision capabilities for analyzing images.

### Image Analysis

```java
@Service
public class VisionService {
    
    private final ChatClient chatClient;
    
    /**
     * Analyze image from URL.
     */
    public String analyzeImageFromUrl(String imageUrl, String question) {
        UserMessage message = new UserMessage(
                question,
                List.of(new Media(MimeTypeUtils.IMAGE_PNG, new URL(imageUrl)))
        );
        
        return chatClient.prompt()
                .messages(message)
                .call()
                .content();
    }
    
    /**
     * Analyze image from file.
     */
    public String analyzeImageFromFile(Path imagePath, String question) 
            throws IOException {
        
        Resource imageResource = new FileSystemResource(imagePath);
        
        UserMessage message = new UserMessage(
                question,
                List.of(new Media(MimeTypeUtils.IMAGE_JPEG, imageResource))
        );
        
        return chatClient.prompt()
                .messages(message)
                .call()
                .content();
    }
    
    /**
     * Analyze image from byte array.
     */
    public String analyzeImageFromBytes(byte[] imageBytes, String question) {
        String base64Image = Base64.getEncoder().encodeToString(imageBytes);
        
        UserMessage message = new UserMessage(
                question,
                List.of(new Media(
                        MimeTypeUtils.IMAGE_PNG,
                        new ByteArrayResource(imageBytes)
                ))
        );
        
        return chatClient.prompt()
                .messages(message)
                .call()
                .content();
    }
}
```

### Structured Image Analysis

Extract structured data from images:

```java
public record ProductImage(
        String productName,
        String brand,
        double estimatedPrice,
        String condition,
        List<String> features,
        List<String> defects
) {}

@Service
public class ProductImageAnalyzer {
    
    private final ChatClient chatClient;
    
    public ProductImage analyzeProduct(Resource imageResource) {
        UserMessage message = new UserMessage(
                """
                Analyze this product image and extract:
                - Product name
                - Brand
                - Estimated price
                - Condition (new/used/damaged)
                - Notable features
                - Any defects or issues
                
                Return as JSON.
                """,
                List.of(new Media(MimeTypeUtils.IMAGE_JPEG, imageResource))
        );
        
        return chatClient.prompt()
                .messages(message)
                .call()
                .entity(ProductImage.class);
    }
}
```

### OCR and Document Processing

Extract text from images:

```java
@Service
public class OcrService {
    
    private final ChatClient chatClient;
    
    /**
     * Extract text from document image.
     */
    public String extractText(Resource documentImage) {
        UserMessage message = new UserMessage(
                "Extract all text from this document. Preserve formatting.",
                List.of(new Media(MimeTypeUtils.IMAGE_PNG, documentImage))
        );
        
        return chatClient.prompt()
                .messages(message)
                .call()
                .content();
    }
    
    /**
     * Extract structured data from invoice.
     */
    public record Invoice(
            String invoiceNumber,
            String date,
            String vendor,
            List<LineItem> items,
            double subtotal,
            double tax,
            double total
    ) {}
    
    public record LineItem(
            String description,
            int quantity,
            double unitPrice,
            double amount
    ) {}
    
    public Invoice extractInvoiceData(Resource invoiceImage) {
        UserMessage message = new UserMessage(
                "Extract invoice data from this image and return as JSON.",
                List.of(new Media(MimeTypeUtils.IMAGE_JPEG, invoiceImage))
        );
        
        return chatClient.prompt()
                .messages(message)
                .call()
                .entity(Invoice.class);
    }
}
```

### Multi-Image Analysis

Compare multiple images:

```java
@Service
public class MultiImageService {
    
    private final ChatClient chatClient;
    
    public String compareImages(Resource image1, Resource image2) {
        UserMessage message = new UserMessage(
                "Compare these two images. What are the differences?",
                List.of(
                        new Media(MimeTypeUtils.IMAGE_JPEG, image1),
                        new Media(MimeTypeUtils.IMAGE_JPEG, image2)
                )
        );
        
        return chatClient.prompt()
                .messages(message)
                .call()
                .content();
    }
    
    public String analyzeImageSequence(List<Resource> images) {
        List<Media> mediaList = images.stream()
                .map(img -> new Media(MimeTypeUtils.IMAGE_JPEG, img))
                .toList();
        
        UserMessage message = new UserMessage(
                "Analyze this sequence of images. Describe what's happening.",
                mediaList
        );
        
        return chatClient.prompt()
                .messages(message)
                .call()
                .content();
    }
}
```

---

## Advanced Features

### JSON Mode

Force responses to be valid JSON:

```java
public record Person(
        String name,
        int age,
        String occupation,
        List<String> hobbies
) {}

@Service
public class JsonModeService {
    
    private final ChatClient chatClient;
    
    public Person extractPersonInfo(String text) {
        return chatClient.prompt()
                .user("Extract person information from: " + text)
                .options(ChatOptions.builder()
                        .withResponseFormat(
                                new ResponseFormat(ResponseFormat.Type.JSON_OBJECT)
                        )
                        .build())
                .call()
                .entity(Person.class);
    }
}
```

### Seed for Reproducibility

Make responses deterministic:

```java
@Service
public class DeterministicService {
    
    private final ChatClient chatClient;
    
    /**
     * Generate reproducible responses.
     * Same seed + same prompt = same response.
     */
    public String deterministicGeneration(String prompt, long seed) {
        return chatClient.prompt()
                .user(prompt)
                .options(ChatOptions.builder()
                        .withTemperature(0.0)
                        .withSeed(seed)
                        .build())
                .call()
                .content();
    }
}

// Usage
String response1 = service.deterministicGeneration("Tell me a joke", 12345L);
String response2 = service.deterministicGeneration("Tell me a joke", 12345L);
// response1 == response2 (deterministic!)

String response3 = service.deterministicGeneration("Tell me a joke", 67890L);
// response3 != response1 (different seed)
```

### Logit Bias

Control token probability:

```java
@Service
public class LogitBiasService {
    
    private final ChatClient chatClient;
    
    /**
     * Encourage/discourage specific tokens.
     */
    public String biasedGeneration(String prompt, Map<String, Integer> bias) {
        return chatClient.prompt()
                .user(prompt)
                .options(ChatOptions.builder()
                        .withLogitBias(bias)
                        .build())
                .call()
                .content();
    }
    
    /**
     * Example: Avoid using word "AI"
     */
    public String avoidAI(String prompt) {
        // Token ID for "AI" is approximately 15836
        return biasedGeneration(
                prompt,
                Map.of("15836", -100)  // Strongly discourage
        );
    }
    
    /**
     * Example: Encourage positive sentiment
     */
    public String encouragePositive(String prompt) {
        Map<String, Integer> positiveBias = Map.of(
                "0000", 10,   // "great"
                "0001", 10,   // "excellent"
                "0002", 10    // "amazing"
                // Note: Actual token IDs vary
        );
        
        return biasedGeneration(prompt, positiveBias);
    }
}
```

### System Fingerprint

Track model versions:

```java
@Service
public class ModelVersionTracking {
    
    public void trackModelVersion(ChatResponse response) {
        String fingerprint = response.getMetadata().getSystemFingerprint();
        
        log.info("Response generated by model version: {}", fingerprint);
        
        // Store for audit trail
        auditRepository.save(new ModelAudit(
                fingerprint,
                response.getMetadata().getModel(),
                Instant.now()
        ));
    }
}
```

---

## Production Best Practices

### Error Handling

Comprehensive error handling:

```java
@Service
public class RobustChatService {
    
    private final ChatClient chatClient;
    
    public String chat(String message) {
        try {
            return chatClient.prompt(message).call().content();
            
        } catch (RateLimitException e) {
            log.warn("Rate limit hit, retry after: {}", 
                    e.getRetryAfter());
            
            // Wait and retry
            Thread.sleep(e.getRetryAfter().toMillis());
            return chatClient.prompt(message).call().content();
            
        } catch (ApiException e) {
            if (e.getStatusCode() == 400) {
                log.error("Invalid request: {}", e.getMessage());
                return "Invalid request. Please rephrase your question.";
            }
            
            if (e.getStatusCode() == 500) {
                log.error("OpenAI server error", e);
                return "Service temporarily unavailable. Please try again.";
            }
            
            throw e;
            
        } catch (Exception e) {
            log.error("Unexpected error", e);
            return "An error occurred. Please try again later.";
        }
    }
}
```

### Rate Limiting

Client-side rate limiting:

```java
@Configuration
public class RateLimiterConfig {
    
    @Bean
    public RateLimiter openAiRateLimiter() {
        // OpenAI Tier 1: 500 requests per minute
        return RateLimiter.create(8.0);  // ~480 per minute with buffer
    }
}

@Component
public class RateLimitingAdvisor implements RequestResponseAdvisor {
    
    private final RateLimiter rateLimiter;
    
    @Override
    public AdvisedRequest adviseRequest(
            AdvisedRequest request,
            Map<String, Object> context) {
        
        // Acquire permit (blocks if rate limit exceeded)
        rateLimiter.acquire();
        
        return request;
    }
}
```

### Monitoring and Metrics

Track usage and performance:

```java
@Component
public class MetricsAdvisor implements RequestResponseAdvisor {
    
    private final MeterRegistry meterRegistry;
    
    @Override
    public ChatResponse adviseResponse(
            AdvisedRequest request,
            ChatResponse response,
            Map<String, Object> context) {
        
        Usage usage = response.getMetadata().getUsage();
        String model = response.getMetadata().getModel();
        
        // Record token usage
        meterRegistry.counter("openai.tokens.prompt",
                "model", model).increment(usage.getPromptTokens());
        
        meterRegistry.counter("openai.tokens.completion",
                "model", model).increment(usage.getGenerationTokens());
        
        // Record cost
        double cost = calculateCost(model, usage);
        meterRegistry.counter("openai.cost.dollars",
                "model", model).increment(cost);
        
        // Record latency
        Instant requestTime = (Instant) context.get("request_time");
        Duration latency = Duration.between(requestTime, Instant.now());
        
        meterRegistry.timer("openai.request.duration",
                "model", model).record(latency);
        
        return response;
    }
    
    private double calculateCost(String model, Usage usage) {
        return switch (model) {
            case "gpt-4o" ->
                    (usage.getPromptTokens() / 1_000_000.0 * 5.0) +
                    (usage.getGenerationTokens() / 1_000_000.0 * 15.0);
            case "gpt-4o-mini" ->
                    (usage.getPromptTokens() / 1_000_000.0 * 0.15) +
                    (usage.getGenerationTokens() / 1_000_000.0 * 0.60);
            default -> 0.0;
        };
    }
}
```

### Caching Strategy

Multi-level caching:

```java
@Service
public class CachedChatService {
    
    private final ChatClient chatClient;
    private final Cache<String, String> l1Cache;  // In-memory
    private final RedisTemplate<String, String> l2Cache;  // Redis
    
    public CachedChatService(
            ChatClient chatClient,
            RedisTemplate<String, String> redisTemplate) {
        
        this.chatClient = chatClient;
        this.redisTemplate = redisTemplate;
        
        // L1: Caffeine cache (10,000 entries, 1 hour TTL)
        this.l1Cache = CacheBuilder.newBuilder()
                .maximumSize(10_000)
                .expireAfterWrite(Duration.ofHours(1))
                .recordStats()
                .build();
    }
    
    public String chat(String message) {
        String cacheKey = generateCacheKey(message);
        
        // L1 cache lookup
        String l1Result = l1Cache.getIfPresent(cacheKey);
        if (l1Result != null) {
            metrics.counter("cache.hit", "level", "L1").increment();
            return l1Result;
        }
        
        // L2 cache lookup
        String l2Result = redisTemplate.opsForValue().get(cacheKey);
        if (l2Result != null) {
            metrics.counter("cache.hit", "level", "L2").increment();
            l1Cache.put(cacheKey, l2Result);  // Populate L1
            return l2Result;
        }
        
        // Cache miss - call API
        metrics.counter("cache.miss").increment();
        String response = chatClient.prompt(message).call().content();
        
        // Populate both cache levels
        l1Cache.put(cacheKey, response);
        redisTemplate.opsForValue().set(
                cacheKey,
                response,
                Duration.ofDays(7)
        );
        
        return response;
    }
    
    private String generateCacheKey(String message) {
        return DigestUtils.md5DigestAsHex(message.getBytes());
    }
}
```

### Health Checks

Monitor OpenAI service health:

```java
@Component
public class OpenAiHealthIndicator implements HealthIndicator {
    
    private final ChatClient chatClient;
    
    @Override
    public Health health() {
        try {
            long startTime = System.currentTimeMillis();
            
            String response = chatClient.prompt()
                    .user("test")
                    .options(ChatOptions.builder()
                            .withMaxTokens(5)
                            .build())
                    .call()
                    .content();
            
            long latency = System.currentTimeMillis() - startTime;
            
            return Health.up()
                    .withDetail("latency_ms", latency)
                    .withDetail("response_length", response.length())
                    .withDetail("timestamp", Instant.now())
                    .build();
                    
        } catch (Exception e) {
            return Health.down()
                    .withDetail("error", e.getMessage())
                    .withException(e)
                    .build();
        }
    }
}
```

---

## Cost Optimization

### Token Counting

Estimate costs before API calls:

```java
@Service
public class TokenCounterService {
    
    /**
     * Rough token estimation (actual count may vary).
     * 1 token ≈ 4 characters for English text.
     */
    public int estimateTokens(String text) {
        return text.length() / 4;
    }
    
    /**
     * More accurate counting using tiktoken (requires native library).
     */
    public int countTokensAccurate(String text, String model) {
        Encoding encoding = switch (model) {
            case "gpt-4", "gpt-4o" -> Encoding.CL100K_BASE;
            case "gpt-3.5-turbo" -> Encoding.CL100K_BASE;
            default -> Encoding.CL100K_BASE;
        };
        
        return encoding.countTokens(text);
    }
    
    /**
     * Estimate cost before making request.
     */
    public record CostEstimate(
            int promptTokens,
            int maxCompletionTokens,
            double minCost,
            double maxCost
    ) {}
    
    public CostEstimate estimateCost(String prompt, int maxTokens, String model) {
        int promptTokens = estimateTokens(prompt);
        
        double inputCostPer1M = getInputCost(model);
        double outputCostPer1M = getOutputCost(model);
        
        double minCost = (promptTokens / 1_000_000.0) * inputCostPer1M;
        
        double maxCost = minCost +
                (maxTokens / 1_000_000.0) * outputCostPer1M;
        
        return new CostEstimate(
                promptTokens,
                maxTokens,
                minCost,
                maxCost
        );
    }
    
    private double getInputCost(String model) {
        return switch (model) {
            case "gpt-4o" -> 5.0;
            case "gpt-4o-mini" -> 0.15;
            case "gpt-4-turbo" -> 10.0;
            default -> 0.0;
        };
    }
    
    private double getOutputCost(String model) {
        return switch (model) {
            case "gpt-4o" -> 15.0;
            case "gpt-4o-mini" -> 0.60;
            case "gpt-4-turbo" -> 30.0;
            default -> 0.0;
        };
    }
}
```

### Budget Controls

Enforce spending limits:

```java
@Service
public class BudgetControlService {
    
    private final AtomicDouble dailySpend = new AtomicDouble(0.0);
    private final double DAILY_BUDGET = 100.0;  // $100 per day
    
    @Scheduled(cron = "0 0 0 * * *")  // Reset at midnight
    public void resetDailyBudget() {
        double spent = dailySpend.getAndSet(0.0);
        log.info("Daily spend reset. Yesterday: ${}", 
                String.format("%.2f", spent));
    }
    
    public boolean canAfford(double estimatedCost) {
        double currentSpend = dailySpend.get();
        return (currentSpend + estimatedCost) <= DAILY_BUDGET;
    }
    
    public void recordSpend(double cost) {
        double newTotal = dailySpend.addAndGet(cost);
        
        if (newTotal > DAILY_BUDGET * 0.8) {
            log.warn("Approaching daily budget: ${} / ${}",
                    String.format("%.2f", newTotal),
                    DAILY_BUDGET);
        }
        
        if (newTotal > DAILY_BUDGET) {
            log.error("Daily budget exceeded: ${}", 
                    String.format("%.2f", newTotal));
            
            // Alert operations team
            alertService.sendAlert(
                    "OpenAI daily budget exceeded",
                    "Current spend: $" + String.format("%.2f", newTotal)
            );
        }
    }
}

@Component
public class BudgetAdvisor implements RequestResponseAdvisor {
    
    private final BudgetControlService budgetControl;
    private final TokenCounterService tokenCounter;
    
    @Override
    public AdvisedRequest adviseRequest(
            AdvisedRequest request,
            Map<String, Object> context) {
        
        // Estimate cost
        String prompt = request.prompt().getInstructions().toString();
        int maxTokens = getMaxTokens(request);
        String model = getModel(request);
        
        CostEstimate estimate = tokenCounter.estimateCost(
                prompt,
                maxTokens,
                model
        );
        
        // Check budget
        if (!budgetControl.canAfford(estimate.maxCost())) {
            throw new BudgetExceededException(
                    "Request would exceed daily budget"
            );
        }
        
        context.put("cost_estimate", estimate);
        return request;
    }
    
    @Override
    public ChatResponse adviseResponse(
            AdvisedRequest request,
            ChatResponse response,
            Map<String, Object> context) {
        
        // Record actual cost
        Usage usage = response.getMetadata().getUsage();
        String model = response.getMetadata().getModel();
        
        double actualCost = calculateActualCost(model, usage);
        budgetControl.recordSpend(actualCost);
        
        return response;
    }
}
```

### Model Selection for Cost

Choose cheaper models when appropriate:

```java
@Service
public class CostOptimizedChatService {
    
    private final ChatClient fastClient;    // gpt-4o-mini
    private final ChatClient smartClient;   // gpt-4o
    
    public String chat(String message, TaskComplexity complexity) {
        return switch (complexity) {
            case SIMPLE -> fastClient.prompt(message).call().content();
            case COMPLEX -> smartClient.prompt(message).call().content();
        };
    }
    
    /**
     * Auto-detect complexity and route to appropriate model.
     */
    public String chatAuto(String message) {
        TaskComplexity complexity = detectComplexity(message);
        return chat(message, complexity);
    }
    
    private TaskComplexity detectComplexity(String message) {
        // Simple heuristics
        int wordCount = message.split("\\s+").length;
        boolean hasCodeKeywords = message.matches(".*\\b(code|function|class|algorithm)\\b.*");
        boolean isQuestion = message.trim().endsWith("?");
        
        if (wordCount < 20 && isQuestion && !hasCodeKeywords) {
            return TaskComplexity.SIMPLE;
        }
        
        return TaskComplexity.COMPLEX;
    }
    
    enum TaskComplexity { SIMPLE, COMPLEX }
}
```

---

## Conclusion

You've now mastered OpenAI integration with Spring AI! Let's recap the key points:

### What We Covered

**Setup & Configuration**:
- Zero-boilerplate setup with Spring Boot auto-configuration
- Production-safe API key management
- Model selection strategies (GPT-4o, GPT-4o-mini, etc.)

**Chat Models**:
- GPT-4o for best performance and cost
- Temperature and creativity control
- Multi-turn conversations with context
- Token management and cost estimation

**Embeddings & Vector Search**:
- text-embedding-3-small for most use cases
- RAG (Retrieval-Augmented Generation) implementation
- Semantic search and caching
- Vector store integration (PostgreSQL, Redis, ChromaDB)

**Streaming**:
- Real-time response streaming with Flux
- Server-Sent Events for web clients
- Backpressure handling
- Progressive rendering for better UX

**Function Calling**:
- Extend GPT-4o with Java methods
- Real-time data access
- Action execution (emails, database queries)
- Multi-step workflows

**Vision**:
- Image analysis with GPT-4o
- OCR and document processing
- Structured data extraction from images
- Multi-image comparison

**Production Ready**:
- Comprehensive error handling
- Rate limiting and circuit breakers
- Multi-level caching
- Monitoring and metrics
- Health checks
- Budget controls

### Key Takeaways

**Best Practices**:
- ✅ Use **gpt-4o** for production (best performance/cost ratio)
- ✅ Enable **streaming** for long responses
- ✅ Implement **semantic caching** to reduce costs
- ✅ Add **monitoring and alerting** from day one
- ✅ Use **function calling** instead of parsing responses
- ✅ Set **budget controls** to prevent overspending

**Cost Optimization**:
- 💰 Route simple tasks to **gpt-4o-mini** (90% cheaper)
- 💰 Cache responses aggressively
- 💰 Use **smaller embedding models** (text-embedding-3-small)
- 💰 Limit **max_tokens** appropriately
- 💰 Monitor usage with metrics

**Common Pitfalls to Avoid**:
- ❌ Committing API keys to version control
- ❌ Not handling rate limits
- ❌ Ignoring token counting (cost surprises)
- ❌ Missing error handling
- ❌ Over-using expensive models

### Next Steps

**Immediate Actions**:
1. Set up monitoring and alerts
2. Implement caching strategy
3. Add budget controls
4. Create health checks
5. Document your prompts and model choices

**Advanced Topics to Explore**:
- Fine-tuning models for your domain
- Building multi-agent systems
- Hybrid search (keyword + semantic)
- A/B testing different prompts
- Prompt engineering optimization

**Production Checklist**:
- [ ] API keys secured (Vault/Secrets Manager)
- [ ] Rate limiting implemented
- [ ] Error handling comprehensive
- [ ] Monitoring and metrics active
- [ ] Caching strategy deployed
- [ ] Budget controls enforced
- [ ] Health checks configured
- [ ] Fallback strategies in place
- [ ] Load testing completed
- [ ] Documentation updated

### Resources

**Official Documentation**:
- [Spring AI Reference](https://docs.spring.io/spring-ai/reference/)
- [OpenAI API Documentation](https://platform.openai.com/docs/)
- [OpenAI Models](https://platform.openai.com/docs/models)
- [OpenAI Pricing](https://openai.com/pricing)

**Community**:
- [Spring AI on GitHub](https://github.com/spring-projects/spring-ai)
- [Spring AI Discord](https://discord.gg/spring)
- [OpenAI Community Forum](https://community.openai.com/)

**Tools**:
- [OpenAI Playground](https://platform.openai.com/playground) - Test prompts
- [Tokenizer](https://platform.openai.com/tokenizer) - Count tokens
- [OpenAI Status](https://status.openai.com/) - Service status

---

**Thank you for reading this comprehensive guide!**

You're now equipped to build production-ready OpenAI applications with Spring AI. Whether you're building chatbots, RAG systems, data analysis tools, or autonomous agents, you have the knowledge to do it efficiently and cost-effectively.

**Questions? Feedback?** Join the Spring AI community on Discord or open an issue on GitHub.

**Happy coding with Spring AI and OpenAI!** 🚀

---

*Last updated: November 20, 2025*  
*Spring AI version: 1.0.0-M3*  
*OpenAI models: GPT-4o, GPT-4o-mini, text-embedding-3-small/large*  
*Author: SpringDevPro Team*