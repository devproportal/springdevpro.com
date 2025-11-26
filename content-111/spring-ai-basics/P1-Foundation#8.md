
基于下面的信息，给出英文技术博客文章（面向欧美用户，基于 Google Adsense赚钱）：
文章为主，代码为辅。
要有图表和表格。

Reference Title: Spring AI Testing Guide: Unit Tests, Integration Tests & Mocking
Reference Keywords: spring ai testing
Target Word Count: 4500-5000

# Spring AI Testing Guide: Unit Tests, Integration Tests & Mocking

**Master Testing Strategies for Reliable Spring AI Applications**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Testing Fundamentals](#testing-fundamentals)
3. [Unit Testing AI Components](#unit-testing-ai-components)
4. [Mocking AI Clients](#mocking-ai-clients)
5. [Integration Testing](#integration-testing)
6. [Testing Prompts and Templates](#testing-prompts-and-templates)
7. [Testing RAG Systems](#testing-rag-systems)
8. [Testing Streaming Responses](#testing-streaming-responses)
9. [Testing Error Handling](#testing-error-handling)
10. [Performance Testing](#performance-testing)
11. [Contract Testing](#contract-testing)
12. [Best Practices](#best-practices)
13. [Conclusion](#conclusion)

---

## Introduction

Testing AI applications presents unique challenges compared to traditional software. AI responses are non-deterministic, external API calls are expensive, and model behavior can be unpredictable. This guide shows you how to build a comprehensive testing strategy for Spring AI applications.

### Why Testing AI Applications Is Different

**Traditional Application**:
```java
// Deterministic: same input = same output
int result = calculator.add(2, 3);
assertEquals(5, result);  // Always passes
```

**AI Application**:
```java
// Non-deterministic: same input = different outputs
String response = chatClient.call("Tell me a joke");
// Response varies each time
// "Why did the chicken cross the road?"
// "What do you call a bear with no teeth? A gummy bear!"
// "Knock knock..."
```

### Testing Challenges

**Non-Determinism**:
- Same prompt produces different responses
- Temperature and sampling affect outputs
- Can't use exact string matching

**External Dependencies**:
- API calls cost money and time
- Rate limits affect test execution
- Network issues cause flaky tests

**Complexity**:
- Multiple components (embeddings, vector stores, prompts)
- Context-dependent behavior
- Difficult to measure quality

**Cost Concerns**:
- Running full integration tests is expensive
- Need to balance coverage with budget
- Test data generation costs

### Testing Strategy Overview

```
┌─────────────────────────────────────┐
│         Testing Pyramid            │
├─────────────────────────────────────┤
│                                     │
│         E2E Tests (Few)             │  ← Expensive, slow
│      ▲                              │
│     ▲ ▲                             │
│    ▲   ▲  Integration Tests         │  ← Moderate cost
│   ▲     ▲  (Some)                   │
│  ▲       ▲                          │
│ ▲         ▲                         │
│▲▲▲▲▲▲▲▲▲▲▲▲ Unit Tests (Many)      │  ← Fast, cheap
│                                     │
└─────────────────────────────────────┘
```

**Our Approach**:
1. **Unit Tests**: Test logic without calling AI APIs (fast, free)
2. **Integration Tests**: Test with mocked or test AI clients (controlled)
3. **Contract Tests**: Verify API expectations (minimal API calls)
4. **E2E Tests**: Test critical paths with real APIs (selective)

### What You'll Learn

By the end of this guide, you'll master:

✅ Unit testing AI service logic  
✅ Mocking ChatClient and AI components  
✅ Integration testing with test containers  
✅ Testing prompts and templates  
✅ Verifying RAG system behavior  
✅ Testing streaming responses  
✅ Performance and load testing  
✅ Building reliable test suites  

Let's build confidence in your AI applications!

---

## Testing Fundamentals

### Test Dependencies

**Add to pom.xml**:

```xml
<dependencies>
    <!-- Spring Boot Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- Mockito for mocking -->
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- AssertJ for fluent assertions -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- Testcontainers for integration tests -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- WireMock for API mocking -->
    <dependency>
        <groupId>org.wiremock</groupId>
        <artifactId>wiremock-standalone</artifactId>
        <version>3.3.1</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Awaitility for async testing -->
    <dependency>
        <groupId>org.awaitility</groupId>
        <artifactId>awaitility</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Test Configuration

**src/test/resources/application-test.yml**:

```yaml
spring:
  ai:
    openai:
      api-key: test-key-12345
      base-url: http://localhost:8080  # WireMock server
      chat:
        options:
          model: gpt-4o-mini
          temperature: 0.7
          
logging:
  level:
    org.springframework.ai: DEBUG
    com.example.aiapp: DEBUG
```

### Base Test Classes

**Abstract test class with common setup**:

```java
package com.example.aiapp;

import org.junit.jupiter.api.BeforeEach;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;

@SpringBootTest
@ActiveProfiles("test")
public abstract class BaseIntegrationTest {
    
    @BeforeEach
    void setUp() {
        // Common setup for all integration tests
    }
}
```

**Unit test base**:

```java
package com.example.aiapp;

import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
public abstract class BaseUnitTest {
    // Common utilities for unit tests
}
```

---

## Unit Testing AI Components

### Testing Service Logic

**Service to test**:

```java
package com.example.aiapp.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.stereotype.Service;

@Service
public class ChatService {
    
    private final ChatClient chatClient;
    
    public ChatService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }
    
    public String chat(String userMessage) {
        if (userMessage == null || userMessage.isBlank()) {
            throw new IllegalArgumentException("Message cannot be empty");
        }
        
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
    
    public String generateSummary(String text, int maxWords) {
        if (text == null || text.isBlank()) {
            return "";
        }
        
        if (maxWords <= 0) {
            throw new IllegalArgumentException("Max words must be positive");
        }
        
        String prompt = String.format(
            "Summarize the following text in %d words or less:\n\n%s",
            maxWords,
            text
        );
        
        return chatClient.prompt()
            .user(prompt)
            .call()
            .content();
    }
}
```

**Unit tests**:

```java
package com.example.aiapp.service;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatResponse;
import org.springframework.ai.chat.prompt.Prompt;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class ChatServiceTest {
    
    @Mock
    private ChatClient.Builder chatClientBuilder;
    
    @Mock
    private ChatClient chatClient;
    
    @Mock
    private ChatClient.ChatClientRequest chatClientRequest;
    
    @Mock
    private ChatClient.ChatClientRequest.CallPromptResponseSpec responseSpec;
    
    @InjectMocks
    private ChatService chatService;
    
    @Test
    void shouldRejectNullMessage() {
        // Act & Assert
        assertThatThrownBy(() -> chatService.chat(null))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("Message cannot be empty");
    }
    
    @Test
    void shouldRejectBlankMessage() {
        // Act & Assert
        assertThatThrownBy(() -> chatService.chat("   "))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("Message cannot be empty");
    }
    
    @Test
    void shouldCallChatClientWithValidMessage() {
        // Arrange
        when(chatClientBuilder.build()).thenReturn(chatClient);
        when(chatClient.prompt()).thenReturn(chatClientRequest);
        when(chatClientRequest.user(anyString())).thenReturn(chatClientRequest);
        when(chatClientRequest.call()).thenReturn(responseSpec);
        when(responseSpec.content()).thenReturn("AI response");
        
        chatService = new ChatService(chatClientBuilder);
        
        // Act
        String response = chatService.chat("Hello");
        
        // Assert
        assertThat(response).isEqualTo("AI response");
        verify(chatClientRequest).user("Hello");
    }
    
    @Test
    void shouldGenerateSummaryWithCorrectPrompt() {
        // Arrange
        String text = "Long article text here...";
        int maxWords = 50;
        
        when(chatClientBuilder.build()).thenReturn(chatClient);
        when(chatClient.prompt()).thenReturn(chatClientRequest);
        when(chatClientRequest.user(anyString())).thenReturn(chatClientRequest);
        when(chatClientRequest.call()).thenReturn(responseSpec);
        when(responseSpec.content()).thenReturn("Summary");
        
        chatService = new ChatService(chatClientBuilder);
        
        // Act
        chatService.generateSummary(text, maxWords);
        
        // Assert
        verify(chatClientRequest).user(argThat(prompt ->
            prompt.contains("50 words") && prompt.contains(text)
        ));
    }
    
    @Test
    void shouldReturnEmptyStringForBlankText() {
        // Act
        String result = chatService.generateSummary("", 100);
        
        // Assert
        assertThat(result).isEmpty();
        verifyNoInteractions(chatClient);
    }
    
    @Test
    void shouldRejectNegativeMaxWords() {
        // Act & Assert
        assertThatThrownBy(() -> chatService.generateSummary("text", -1))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("Max words must be positive");
    }
}
```

### Testing Complex Business Logic

**Service with business rules**:

```java
@Service
public class ContentModerationService {
    
    private final ChatClient chatClient;
    private final Set<String> blockedWords;
    
    public ContentModerationService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
        this.blockedWords = Set.of("spam", "abuse", "scam");
    }
    
    public ModerationResult moderateContent(String content) {
        // Pre-check for obvious violations
        if (containsBlockedWords(content)) {
            return new ModerationResult(false, "Contains blocked words", null);
        }
        
        // Use AI for sophisticated analysis
        String prompt = String.format(
            "Analyze this content for inappropriate language, harassment, " +
            "or spam. Respond with JSON: {\"safe\": true/false, \"reason\": \"...\"}.\n\n%s",
            content
        );
        
        String aiResponse = chatClient.prompt()
            .user(prompt)
            .call()
            .content();
        
        return parseAiResponse(aiResponse);
    }
    
    private boolean containsBlockedWords(String content) {
        String lower = content.toLowerCase();
        return blockedWords.stream().anyMatch(lower::contains);
    }
    
    private ModerationResult parseAiResponse(String response) {
        // Parse JSON response
        // Implementation details...
        return new ModerationResult(true, "Content is safe", response);
    }
    
    public record ModerationResult(
        boolean approved,
        String reason,
        String aiAnalysis
    ) {}
}
```

**Unit tests**:

```java
@ExtendWith(MockitoExtension.class)
class ContentModerationServiceTest {
    
    @Mock
    private ChatClient.Builder chatClientBuilder;
    
    @Mock
    private ChatClient chatClient;
    
    @Mock
    private ChatClient.ChatClientRequest chatClientRequest;
    
    @Mock
    private ChatClient.ChatClientRequest.CallPromptResponseSpec responseSpec;
    
    private ContentModerationService service;
    
    @BeforeEach
    void setUp() {
        when(chatClientBuilder.build()).thenReturn(chatClient);
        service = new ContentModerationService(chatClientBuilder);
    }
    
    @Test
    void shouldRejectContentWithBlockedWords() {
        // Arrange
        String content = "This is spam content";
        
        // Act
        ModerationResult result = service.moderateContent(content);
        
        // Assert
        assertThat(result.approved()).isFalse();
        assertThat(result.reason()).contains("blocked words");
        verifyNoInteractions(chatClient);  // Should not call AI
    }
    
    @Test
    void shouldCallAiForContentWithoutBlockedWords() {
        // Arrange
        String content = "Normal content";
        String aiResponse = "{\"safe\": true, \"reason\": \"No issues found\"}";
        
        when(chatClient.prompt()).thenReturn(chatClientRequest);
        when(chatClientRequest.user(anyString())).thenReturn(chatClientRequest);
        when(chatClientRequest.call()).thenReturn(responseSpec);
        when(responseSpec.content()).thenReturn(aiResponse);
        
        // Act
        ModerationResult result = service.moderateContent(content);
        
        // Assert
        assertThat(result.approved()).isTrue();
        verify(chatClient).prompt();
    }
    
    @Test
    void shouldHandleMultipleBlockedWords() {
        // Arrange
        Set<String> testCases = Set.of(
            "spam message",
            "This is abuse",
            "scam alert"
        );
        
        // Act & Assert
        for (String content : testCases) {
            ModerationResult result = service.moderateContent(content);
            assertThat(result.approved())
                .as("Content: " + content)
                .isFalse();
        }
    }
}
```

### Testing Prompt Construction

```java
@Service
public class PromptBuilderService {
    
    public String buildTranslationPrompt(String text, String sourceLang, String targetLang) {
        return String.format(
            "Translate the following text from %s to %s:\n\n%s",
            sourceLang,
            targetLang,
            text
        );
    }
    
    public String buildSentimentPrompt(String text) {
        return String.format(
            "Analyze the sentiment of this text. " +
            "Respond with only: POSITIVE, NEGATIVE, or NEUTRAL.\n\n%s",
            text
        );
    }
}
```

**Tests for prompt construction**:

```java
class PromptBuilderServiceTest {
    
    private PromptBuilderService service;
    
    @BeforeEach
    void setUp() {
        service = new PromptBuilderService();
    }
    
    @Test
    void shouldBuildTranslationPrompt() {
        // Act
        String prompt = service.buildTranslationPrompt(
            "Hello world",
            "English",
            "Spanish"
        );
        
        // Assert
        assertThat(prompt)
            .contains("English")
            .contains("Spanish")
            .contains("Hello world")
            .contains("Translate");
    }
    
    @Test
    void shouldBuildSentimentPrompt() {
        // Act
        String prompt = service.buildSentimentPrompt("I love this!");
        
        // Assert
        assertThat(prompt)
            .contains("POSITIVE")
            .contains("NEGATIVE")
            .contains("NEUTRAL")
            .contains("I love this!");
    }
    
    @ParameterizedTest
    @CsvSource({
        "English, Spanish",
        "French, German",
        "Japanese, English"
    })
    void shouldHandleDifferentLanguagePairs(String source, String target) {
        // Act
        String prompt = service.buildTranslationPrompt("test", source, target);
        
        // Assert
        assertThat(prompt).contains(source, target);
    }
}
```

---

## Mocking AI Clients

### Mocking ChatClient

**Complete mock setup**:

```java
@ExtendWith(MockitoExtension.class)
class MockingExampleTest {
    
    @Mock
    private ChatClient chatClient;
    
    @Mock
    private ChatClient.ChatClientRequest chatClientRequest;
    
    @Mock
    private ChatClient.ChatClientRequest.CallPromptResponseSpec responseSpec;
    
    @Mock
    private ChatResponse chatResponse;
    
    @BeforeEach
    void setUp() {
        // Setup default mock behavior
        when(chatClient.prompt()).thenReturn(chatClientRequest);
        when(chatClientRequest.user(anyString())).thenReturn(chatClientRequest);
        when(chatClientRequest.system(anyString())).thenReturn(chatClientRequest);
        when(chatClientRequest.options(any())).thenReturn(chatClientRequest);
        when(chatClientRequest.call()).thenReturn(responseSpec);
    }
    
    @Test
    void shouldMockSimpleResponse() {
        // Arrange
        when(responseSpec.content()).thenReturn("Mocked response");
        
        // Act
        String response = chatClient.prompt()
            .user("Test")
            .call()
            .content();
        
        // Assert
        assertThat(response).isEqualTo("Mocked response");
    }
    
    @Test
    void shouldMockChatResponse() {
        // Arrange
        when(responseSpec.chatResponse()).thenReturn(chatResponse);
        when(chatResponse.getResult()).thenReturn(
            new Generation("Full response content")
        );
        
        // Act
        ChatResponse response = chatClient.prompt()
            .user("Test")
            .call()
            .chatResponse();
        
        // Assert
        assertThat(response.getResult().getOutput().getContent())
            .isEqualTo("Full response content");
    }
}
```

### Creating Test Doubles

**Custom test implementation**:

```java
public class TestChatClient implements ChatClient {
    
    private final Map<String, String> cannedResponses;
    
    public TestChatClient() {
        this.cannedResponses = new HashMap<>();
        setupDefaultResponses();
    }
    
    private void setupDefaultResponses() {
        cannedResponses.put("hello", "Hello! How can I help you?");
        cannedResponses.put("weather", "I can't check the weather right now.");
        cannedResponses.put("joke", "Why did the programmer quit? " +
            "Because they didn't get arrays!");
    }
    
    public void addResponse(String keyword, String response) {
        cannedResponses.put(keyword.toLowerCase(), response);
    }
    
    @Override
    public ChatClientRequest prompt() {
        return new TestChatClientRequest(cannedResponses);
    }
    
    private static class TestChatClientRequest 
            implements ChatClient.ChatClientRequest {
        
        private final Map<String, String> responses;
        private String userMessage;
        
        TestChatClientRequest(Map<String, String> responses) {
            this.responses = responses;
        }
        
        @Override
        public ChatClientRequest user(String message) {
            this.userMessage = message;
            return this;
        }
        
        @Override
        public CallPromptResponseSpec call() {
            return new TestResponseSpec(findResponse(userMessage));
        }
        
        private String findResponse(String message) {
            String lower = message.toLowerCase();
            return responses.entrySet().stream()
                .filter(e -> lower.contains(e.getKey()))
                .findFirst()
                .map(Map.Entry::getValue)
                .orElse("I don't understand.");
        }
        
        // Implement other required methods...
    }
    
    private static class TestResponseSpec 
            implements ChatClient.ChatClientRequest.CallPromptResponseSpec {
        
        private final String content;
        
        TestResponseSpec(String content) {
            this.content = content;
        }
        
        @Override
        public String content() {
            return content;
        }
        
        // Implement other required methods...
    }
}
```

**Using test double**:

```java
class TestChatClientTest {
    
    private TestChatClient testClient;
    private ChatService chatService;
    
    @BeforeEach
    void setUp() {
        testClient = new TestChatClient();
        chatService = new ChatService(builder -> testClient);
    }
    
    @Test
    void shouldReturnCannedResponse() {
        // Arrange
        testClient.addResponse("order", "Your order #12345 is being processed");
        
        // Act
        String response = chatService.chat("What's my order status?");
        
        // Assert
        assertThat(response).contains("order #12345");
    }
    
    @Test
    void shouldHandleMultipleKeywords() {
        // Arrange
        testClient.addResponse("refund", "Refunds take 5-7 business days");
        
        // Act
        String response = chatService.chat("How long for a refund?");
        
        // Assert
        assertThat(response).contains("5-7 business days");
    }
}
```

### Argument Captors

**Verify exact prompts sent**:

```java
@Test
void shouldSendCorrectPromptToAi() {
    // Arrange
    ArgumentCaptor<String> promptCaptor = ArgumentCaptor.forClass(String.class);
    
    when(chatClient.prompt()).thenReturn(chatClientRequest);
    when(chatClientRequest.user(anyString())).thenReturn(chatClientRequest);
    when(chatClientRequest.call()).thenReturn(responseSpec);
    when(responseSpec.content()).thenReturn("Response");
    
    ChatService service = new ChatService(builder -> chatClient);
    
    // Act
    service.generateSummary("Article about AI testing", 100);
    
    // Assert
    verify(chatClientRequest).user(promptCaptor.capture());
    
    String capturedPrompt = promptCaptor.getValue();
    assertThat(capturedPrompt)
        .contains("100 words")
        .contains("Article about AI testing")
        .contains("Summarize");
}
```

---

## Integration Testing

### WireMock for API Mocking

**Setup WireMock server**:

```java
@SpringBootTest
@AutoConfigureWireMock(port = 8080)
class ChatServiceIntegrationTest {
    
    @Autowired
    private ChatService chatService;
    
    @Test
    void shouldCallOpenAiApi() {
        // Arrange: Mock OpenAI API response
        stubFor(post(urlEqualTo("/v1/chat/completions"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("""
                    {
                        "id": "chatcmpl-test",
                        "object": "chat.completion",
                        "created": 1234567890,
                        "model": "gpt-4o-mini",
                        "choices": [{
                            "index": 0,
                            "message": {
                                "role": "assistant",
                                "content": "This is a test response"
                            },
                            "finish_reason": "stop"
                        }],
                        "usage": {
                            "prompt_tokens": 10,
                            "completion_tokens": 20,
                            "total_tokens": 30
                        }
                    }
                    """)));
        
        // Act
        String response = chatService.chat("Hello");
        
        // Assert
        assertThat(response).isEqualTo("This is a test response");
        
        // Verify request
        verify(postRequestedFor(urlEqualTo("/v1/chat/completions"))
            .withHeader("Content-Type", equalTo("application/json"))
            .withHeader("Authorization", matching("Bearer .*")));
    }
    
    @Test
    void shouldHandleApiError() {
        // Arrange: Mock error response
        stubFor(post(urlEqualTo("/v1/chat/completions"))
            .willReturn(aResponse()
                .withStatus(429)
                .withBody("""
                    {
                        "error": {
                            "message": "Rate limit exceeded",
                            "type": "rate_limit_error"
                        }
                    }
                    """)));
        
        // Act & Assert
        assertThatThrownBy(() -> chatService.chat("Hello"))
            .hasMessageContaining("Rate limit");
    }
}
```

### Test Containers Integration

**Embedding database testing**:

```java
@SpringBootTest
@Testcontainers
class VectorStoreIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>(
        "pgvector/pgvector:pg16"
    )
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private VectorStore vectorStore;
    
    @Autowired
    private EmbeddingClient embeddingClient;
    
    @Test
    void shouldStoreAndRetrieveDocuments() {
        // Arrange
        List<Document> documents = List.of(
            new Document("Spring AI makes testing easy"),
            new Document("Integration tests verify behavior"),
            new Document("Test containers provide isolated environments")
        );
        
        // Act
        vectorStore.add(documents);
        
        List<Document> results = vectorStore.similaritySearch(
            SearchRequest.query("testing").withTopK(2)
        );
        
        // Assert
        assertThat(results).hasSize(2);
        assertThat(results.get(0).getContent())
            .containsAnyOf("testing", "tests");
    }
}
```

### Testing with Real APIs (Sparingly)

**Smoke tests for critical paths**:

```java
@SpringBootTest
@Tag("smoke")
@ActiveProfiles("production")
class SmokeTest {
    
    @Autowired
    private ChatService chatService;
    
    @Test
    @EnabledIfEnvironmentVariable(
        named = "RUN_SMOKE_TESTS",
        matches = "true"
    )
    void shouldConnectToRealOpenAiApi() {
        // This test uses real API and costs money
        // Run only in CI/CD before deployment
        
        String response = chatService.chat("Say 'test successful' if you can read this");
        
        assertThat(response.toLowerCase())
            .containsAnyOf("test successful", "yes", "read");
    }
}
```

---

## Testing Prompts and Templates

### Template Rendering Tests

```java
@SpringBootTest
class PromptTemplateTest {
    
    @Autowired
    private PromptTemplate promptTemplate;
    
    @Test
    void shouldRenderTemplateCorrectly() {
        // Arrange
        String template = """
            You are a {role}.
            User question: {question}
            Context: {context}
            """;
        
        Map<String, Object> params = Map.of(
            "role", "helpful assistant",
            "question", "What is Spring AI?",
            "context", "Spring AI is a framework..."
        );
        
        // Act
        Prompt prompt = new PromptTemplate(template).create(params);
        
        // Assert
        String rendered = prompt.getContents();
        assertThat(rendered)
            .contains("helpful assistant")
            .contains("What is Spring AI?")
            .contains("Spring AI is a framework");
    }
    
    @Test
    void shouldHandleMissingParameters() {
        // Arrange
        String template = "Hello {name}";
        
        // Act & Assert
        assertThatThrownBy(() -> 
            new PromptTemplate(template).create(Map.of())
        ).isInstanceOf(IllegalArgumentException.class);
    }
}
```

### Testing System Messages

```java
class SystemMessageTest {
    
    @Mock
    private ChatClient chatClient;
    
    @Mock
    private ChatClient.ChatClientRequest request;
    
    @Mock
    private ChatClient.ChatClientRequest.CallPromptResponseSpec spec;
    
    @Test
    void shouldIncludeSystemMessage() {
        // Arrange
        when(chatClient.prompt()).thenReturn(request);
        when(request.system(anyString())).thenReturn(request);
        when(request.user(anyString())).thenReturn(request);
        when(request.call()).thenReturn(spec);
        when(spec.content()).thenReturn("Response");
        
        // Act
        chatClient.prompt()
            .system("You are a helpful assistant")
            .user("Hello")
            .call()
            .content();
        
        // Assert
        verify(request).system("You are a helpful assistant");
        verify(request).user("Hello");
    }
}
```

### Prompt Quality Testing

```java
@Service
public class PromptValidator {
    
    public ValidationResult validate(String prompt) {
        List<String> issues = new ArrayList<>();
        
        // Check length
        if (prompt.length() > 4000) {
            issues.add("Prompt too long (>4000 characters)");
        }
        
        // Check for clear instructions
        if (!containsActionVerb(prompt)) {
            issues.add("Missing clear action verb");
        }
        
        // Check for context
        if (!containsContext(prompt)) {
            issues.add("Missing context or background");
        }
        
        return new ValidationResult(issues.isEmpty(), issues);
    }
    
    private boolean containsActionVerb(String prompt) {
        String[] verbs = {"analyze", "summarize", "explain", "translate", 
                         "generate", "create", "write"};
        String lower = prompt.toLowerCase();
        return Arrays.stream(verbs).anyMatch(lower::contains);
    }
    
    private boolean containsContext(String prompt) {
        return prompt.contains("context") || 
               prompt.contains("background") ||
               prompt.length() > 100;  // Assume longer prompts have context
    }
    
    public record ValidationResult(boolean valid, List<String> issues) {}
}
```

**Testing prompt quality**:

```java
class PromptValidatorTest {
    
    private PromptValidator validator;
    
    @BeforeEach
    void setUp() {
        validator = new PromptValidator();
    }
    
    @Test
    void shouldAcceptWellFormedPrompt() {
        // Arrange
        String prompt = """
            Analyze the following customer feedback.
            Context: This is from our Q4 2024 survey.
            
            Feedback: The product is great but shipping was slow.
            """;
        
        // Act
        ValidationResult result = validator.validate(prompt);
        
        // Assert
        assertThat(result.valid()).isTrue();
        assertThat(result.issues()).isEmpty();
    }
    
    @Test
    void shouldRejectPromptWithoutActionVerb() {
        // Arrange
        String prompt = "Customer said: I like the product";
        
        // Act
        ValidationResult result = validator.validate(prompt);
        
        // Assert
        assertThat(result.valid()).isFalse();
        assertThat(result.issues())
            .anyMatch(issue -> issue.contains("action verb"));
    }
    
    @Test
    void shouldRejectTooLongPrompt() {
        // Arrange
        String prompt = "A".repeat(5000);
        
        // Act
        ValidationResult result = validator.validate(prompt);
        
        // Assert
        assertThat(result.valid()).isFalse();
        assertThat(result.issues())
            .anyMatch(issue -> issue.contains("too long"));
    }
}
```

---

## Testing RAG Systems

### Document Loading Tests

```java
class DocumentLoaderTest {
    
    @TempDir
    Path tempDir;
    
    @Test
    void shouldLoadTextDocuments() throws IOException {
        // Arrange
        Path testFile = tempDir.resolve("test.txt");
        Files.writeString(testFile, "This is test content");
        
        TextReader loader = new TextReader(testFile.toString());
        
        // Act
        List<Document> documents = loader.get();
        
        // Assert
        assertThat(documents).hasSize(1);
        assertThat(documents.get(0).getContent()).isEqualTo("This is test content");
    }
    
    @Test
    void shouldHandlePdfDocuments() {
        // Arrange
        Resource resource = new ClassPathResource("test-document.pdf");
        PagePdfDocumentReader loader = new PagePdfDocumentReader(resource);
        
        // Act
        List<Document> documents = loader.get();
        
        // Assert
        assertThat(documents).isNotEmpty();
        assertThat(documents.get(0).getContent()).isNotBlank();
    }
}
```

### Vector Store Tests

```java
@ExtendWith(MockitoExtension.class)
class RagServiceTest {
    
    @Mock
    private VectorStore vectorStore;
    
    @Mock
    private ChatClient chatClient;
    
    @Mock
    private ChatClient.ChatClientRequest request;
    
    @Mock
    private ChatClient.ChatClientRequest.CallPromptResponseSpec spec;
    
    private RagService ragService;
    
    @BeforeEach
    void setUp() {
        ragService = new RagService(vectorStore, chatClient);
    }
    
    @Test
    void shouldRetrieveRelevantDocuments() {
        // Arrange
        String query = "What is Spring AI?";
        
        List<Document> mockDocs = List.of(
            new Document("Spring AI is a framework for AI applications"),
            new Document("It provides abstractions for LLMs")
        );
        
        when(vectorStore.similaritySearch(any(SearchRequest.class)))
            .thenReturn(mockDocs);
        
        when(chatClient.prompt()).thenReturn(request);
        when(request.user(anyString())).thenReturn(request);
        when(request.call()).thenReturn(spec);
        when(spec.content()).thenReturn("Spring AI is a framework...");
        
        // Act
        String response = ragService.query(query);
        
        // Assert
        verify(vectorStore).similaritySearch(argThat(searchRequest ->
            searchRequest.getQuery().equals(query) &&
            searchRequest.getTopK() >= 3
        ));
        
        assertThat(response).isNotBlank();
    }
    
    @Test
    void shouldIncludeContextInPrompt() {
        // Arrange
        ArgumentCaptor<String> promptCaptor = ArgumentCaptor.forClass(String.class);
        
        List<Document> docs = List.of(
            new Document("Context document 1"),
            new Document("Context document 2")
        );
        
        when(vectorStore.similaritySearch(any())).thenReturn(docs);
        when(chatClient.prompt()).thenReturn(request);
        when(request.user(anyString())).thenReturn(request);
        when(request.call()).thenReturn(spec);
        when(spec.content()).thenReturn("Answer");
        
        // Act
        ragService.query("test question");
        
        // Assert
        verify(request).user(promptCaptor.capture());
        
        String prompt = promptCaptor.getValue();
        assertThat(prompt)
            .contains("Context document 1")
            .contains("Context document 2")
            .contains("test question");
    }
}
```

### Testing Similarity Search

```java
class SimilaritySearchTest {
    
    @Mock
    private VectorStore vectorStore;
    
    @Test
    void shouldReturnMostRelevantDocuments() {
        // Arrange
        Document doc1 = new Document("Spring Boot tutorial");
        doc1.getMetadata().put("score", 0.95);
        
        Document doc2 = new Document("Spring Framework guide");
        doc2.getMetadata().put("score", 0.85);
        
        Document doc3 = new Document("Unrelated content");
        doc3.getMetadata().put("score", 0.40);
        
        when(vectorStore.similaritySearch(any()))
            .thenReturn(List.of(doc1, doc2));  // Sorted by relevance
        
        // Act
        List<Document> results = vectorStore.similaritySearch(
            SearchRequest.query("Spring Boot").withTopK(2)
        );
        
        // Assert
        assertThat(results).hasSize(2);
        assertThat(results.get(0).getContent()).contains("Spring Boot");
        assertThat((Double) results.get(0).getMetadata().get("score"))
            .isGreaterThan(0.9);
    }
}
```

---

## Testing Streaming Responses

### Testing Flux Streams

```java
@ExtendWith(MockitoExtension.class)
class StreamingChatServiceTest {
    
    @Mock
    private ChatClient chatClient;
    
    @Mock
    private ChatClient.ChatClientRequest request;
    
    @Mock
    private ChatClient.ChatClientRequest.StreamResponseSpec streamSpec;
    
    @Test
    void shouldStreamResponses() {
        // Arrange
        Flux<String> mockStream = Flux.just(
            "Hello",
            " ",
            "world",
            "!"
        );
        
        when(chatClient.prompt()).thenReturn(request);
        when(request.user(anyString())).thenReturn(request);
        when(request.stream()).thenReturn(streamSpec);
        when(streamSpec.content()).thenReturn(mockStream);
        
        // Act
        Flux<String> stream = chatClient.prompt()
            .user("test")
            .stream()
            .content();
        
        // Assert
        StepVerifier.create(stream)
            .expectNext("Hello")
            .expectNext(" ")
            .expectNext("world")
            .expectNext("!")
            .verifyComplete();
    }
    
    @Test
    void shouldHandleStreamErrors() {
        // Arrange
        Flux<String> errorStream = Flux.concat(
            Flux.just("Start"),
            Flux.error(new RuntimeException("Stream error"))
        );
        
        when(chatClient.prompt()).thenReturn(request);
        when(request.user(anyString())).thenReturn(request);
        when(request.stream()).thenReturn(streamSpec);
        when(streamSpec.content()).thenReturn(errorStream);
        
        // Act
        Flux<String> stream = chatClient.prompt()
            .user("test")
            .stream()
            .content();
        
        // Assert
        StepVerifier.create(stream)
            .expectNext("Start")
            .expectError(RuntimeException.class)
            .verify();
    }
}
```

### Testing Async Operations

```java
class AsyncChatServiceTest {
    
    @Mock
    private ChatClient chatClient;
    
    @Test
    void shouldHandleAsyncCompletion() {
        // Arrange
        when(chatClient.prompt().user(anyString()).call().content())
            .thenReturn("Async response");
        
        AsyncChatService service = new AsyncChatService(chatClient);
        
        // Act
        CompletableFuture<String> future = service.chatAsync("Hello");
        
        // Assert
        await().atMost(Duration.ofSeconds(5))
            .until(() -> future.isDone());
        
        assertThat(future.join()).isEqualTo("Async response");
    }
    
    @Test
    void shouldTimeoutSlowResponses() {
        // Arrange
        when(chatClient.prompt().user(anyString()).call().content())
            .thenAnswer(invocation -> {
                Thread.sleep(10000);  // Simulate slow response
                return "Too late";
            });
        
        AsyncChatService service = new AsyncChatService(chatClient);
        
        // Act
        CompletableFuture<String> future = service.chatAsync("Hello");
        
        // Assert
        assertThatThrownBy(() -> future.get(1, TimeUnit.SECONDS))
            .isInstanceOf(TimeoutException.class);
    }
}
```

---

## Testing Error Handling

### Exception Testing

```java
class ErrorHandlingTest {
    
    @Mock
    private ChatClient chatClient;
    
    @Mock
    private ChatClient.ChatClientRequest request;
    
    @Mock
    private ChatClient.ChatClientRequest.CallPromptResponseSpec spec;
    
    @Test
    void shouldHandleRateLimitException() {
        // Arrange
        when(chatClient.prompt()).thenReturn(request);
        when(request.user(anyString())).thenReturn(request);
        when(request.call()).thenReturn(spec);
        when(spec.content())
            .thenThrow(new OpenAiRateLimitException("Rate limit exceeded"));
        
        ChatService service = new ChatService(builder -> chatClient);
        
        // Act & Assert
        assertThatThrownBy(() -> service.chat("Hello"))
            .isInstanceOf(OpenAiRateLimitException.class)
            .hasMessageContaining("Rate limit");
    }
    
    @Test
    void shouldRetryOnTransientErrors() {
        // Arrange
        when(chatClient.prompt()).thenReturn(request);
        when(request.user(anyString())).thenReturn(request);
        when(request.call()).thenReturn(spec);
        when(spec.content())
            .thenThrow(new RuntimeException("Timeout"))
            .thenThrow(new RuntimeException("Timeout"))
            .thenReturn("Success after retries");
        
        RetryableChatService service = new RetryableChatService(chatClient);
        
        // Act
        String result = service.chatWithRetry("Hello");
        
        // Assert
        assertThat(result).isEqualTo("Success after retries");
        verify(spec, times(3)).content();
    }
}
```

### Fallback Testing

```java
class FallbackTest {
    
    @Mock
    private ChatClient primaryClient;
    
    @Mock
    private ChatClient fallbackClient;
    
    @Test
    void shouldUseFallbackWhenPrimaryFails() {
        // Arrange
        when(primaryClient.prompt().user(anyString()).call().content())
            .thenThrow(new RuntimeException("Primary failed"));
        
        when(fallbackClient.prompt().user(anyString()).call().content())
            .thenReturn("Fallback response");
        
        MultiProviderChatService service = new MultiProviderChatService(
            primaryClient,
            fallbackClient
        );
        
        // Act
        String response = service.chat("Hello");
        
        // Assert
        assertThat(response).isEqualTo("Fallback response");
        verify(primaryClient).prompt();
        verify(fallbackClient).prompt();
    }
}
```

---

## Performance Testing

### Load Testing

```java
@SpringBootTest
class PerformanceTest {
    
    @Autowired
    private ChatService chatService;
    
    @Test
    @Timeout(value = 30, unit = TimeUnit.SECONDS)
    void shouldHandleConcurrentRequests() throws InterruptedException {
        // Arrange
        int concurrentRequests = 100;
        CountDownLatch latch = new CountDownLatch(concurrentRequests);
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        List<CompletableFuture<String>> futures = new ArrayList<>();
        
        // Act
        for (int i = 0; i < concurrentRequests; i++) {
            CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
                try {
                    return chatService.chat("Test message");
                } finally {
                    latch.countDown();
                }
            }, executor);
            
            futures.add(future);
        }
        
        latch.await(30, TimeUnit.SECONDS);
        
        // Assert
        long successCount = futures.stream()
            .filter(f -> !f.isCompletedExceptionally())
            .count();
        
        assertThat(successCount).isGreaterThan(90);  // 90% success rate
        
        executor.shutdown();
    }
}
```

### Response Time Testing

```java
class ResponseTimeTest {
    
    @Test
    void shouldRespondWithinTimeLimit() {
        // Arrange
        ChatService service = new ChatService(mockChatClient);
        
        // Act & Assert
        assertTimeout(Duration.ofSeconds(5), () -> {
            String response = service.chat("Quick question");
            assertThat(response).isNotBlank();
        });
    }
    
    @Test
    void shouldMeasureAverageResponseTime() {
        // Arrange
        List<Long> responseTimes = new ArrayList<>();
        int iterations = 100;
        
        // Act
        for (int i = 0; i < iterations; i++) {
            long start = System.currentTimeMillis();
            service.chat("Test");
            long duration = System.currentTimeMillis() - start;
            responseTimes.add(duration);
        }
        
        // Assert
        double average = responseTimes.stream()
            .mapToLong(Long::longValue)
            .average()
            .orElse(0);
        
        assertThat(average).isLessThan(1000);  // < 1 second average
    }
}
```

---

## Contract Testing

### API Contract Verification

```java
@SpringBootTest
@AutoConfigureWireMock(port = 8080)
class OpenAiContractTest {
    
    @Test
    void shouldMatchOpenAiApiContract() {
        // Arrange: Create request matching OpenAI's format
        String requestBody = """
            {
                "model": "gpt-4o-mini",
                "messages": [
                    {"role": "user", "content": "Hello"}
                ]
            }
            """;
        
        // Mock response matching OpenAI's format
        stubFor(post("/v1/chat/completions")
            .withRequestBody(matchingJsonPath("$.model"))
            .withRequestBody(matchingJsonPath("$.messages"))
            .willReturn(okJson("""
                {
                    "id": "chatcmpl-123",
                    "object": "chat.completion",
                    "created": 1234567890,
                    "model": "gpt-4o-mini",
                    "choices": [{
                        "index": 0,
                        "message": {
                            "role": "assistant",
                            "content": "Hello!"
                        },
                        "finish_reason": "stop"
                    }]
                }
                """)));
        
        // Act & Assert
        // Call service and verify it works with contract
    }
}
```

---

## Best Practices

### Test Organization

```
src/test/java/
├── unit/
│   ├── service/
│   │   ├── ChatServiceTest.java
│   │   └── PromptBuilderTest.java
│   └── util/
│       └── PromptValidatorTest.java
├── integration/
│   ├── ChatIntegrationTest.java
│   └── VectorStoreIntegrationTest.java
└── e2e/
    └── SmokeTest.java
```

### Test Naming Convention

```java
// Given-When-Then format
@Test
void givenValidMessage_whenChat_thenReturnsResponse() { }

// Should format
@Test
void shouldReturnResponseForValidMessage() { }

// Context-specific
@Test
void chat_withEmptyMessage_throwsException() { }
```

### Test Data Builders

```java
public class TestDataBuilder {
    
    public static Document createDocument(String content) {
        return new Document(content, Map.of(
            "source", "test",
            "timestamp", System.currentTimeMillis()
        ));
    }
    
    public static ChatResponse createChatResponse(String content) {
        return new ChatResponse(List.of(
            new Generation(content)
        ));
    }
    
    public static class ChatClientMockBuilder {
        private final ChatClient mockClient;
        
        public ChatClientMockBuilder() {
            this.mockClient = mock(ChatClient.class);
        }
        
        public ChatClientMockBuilder withResponse(String response) {
            // Setup mock behavior
            return this;
        }
        
        public ChatClient build() {
            return mockClient;
        }
    }
}
```

---

## Conclusion

Testing AI applications requires a balanced approach combining unit tests, integration tests, and selective end-to-end tests. Key takeaways:

**Testing Strategy**:
✅ Use mocks for fast, free unit tests  
✅ Use WireMock for controlled integration tests  
✅ Use test containers for database testing  
✅ Reserve real API calls for smoke tests  

**Best Practices**:
✅ Test business logic independently  
✅ Verify prompts are constructed correctly  
✅ Test error handling and fallbacks  
✅ Use parameterized tests for variations  
✅ Keep tests fast and reliable  

**What to Test**:
✅ Prompt construction and validation  
✅ Response parsing and transformation  
✅ Error handling and retries  
✅ RAG document retrieval  
✅ Streaming behavior  
✅ Performance under load  

Your Spring AI application now has comprehensive test coverage! 🎯