
基于下面的信息，给出英文技术博客文章（面向欧美用户，基于 Google Adsense赚钱）：
文章为主，代码为辅。
要有图表和表格。

Reference Title: Error Handling and Retry Strategies in Spring AI Applications
Reference Keywords: spring ai error handling
Target Word Count: 4500-5000

# Error Handling and Retry Strategies in Spring AI Applications

**Build Resilient AI Applications: A Complete Guide to Handling Failures in Spring AI**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Understanding AI Application Failures](#understanding-ai-application-failures)
3. [Basic Error Handling](#basic-error-handling)
4. [Retry Strategies](#retry-strategies)
5. [Circuit Breaker Pattern](#circuit-breaker-pattern)
6. [Timeout Management](#timeout-management)
7. [Fallback Mechanisms](#fallback-mechanisms)
8. [Rate Limiting and Throttling](#rate-limiting-and-throttling)
9. [Monitoring and Observability](#monitoring-and-observability)
10. [Production Best Practices](#production-best-practices)
11. [Testing Error Scenarios](#testing-error-scenarios)
12. [Conclusion](#conclusion)

---

## Introduction

AI applications face unique challenges that traditional applications don't encounter. API rate limits, token exhaustion, model timeouts, and unpredictable response times make error handling critical for production Spring AI applications.

### Why Error Handling Matters

**Real-World Scenario**:
```
Your AI chatbot is handling customer support. An OpenAI API outage occurs.

❌ Without Error Handling:
- Application crashes
- Users see error messages
- Support queue backs up
- Revenue loss

✅ With Proper Error Handling:
- Graceful degradation to cached responses
- Automatic retry with exponential backoff
- Fallback to alternative AI provider
- User receives: "Experiencing high demand, response may be delayed"
- System continues operating
```

### Common Failure Scenarios

**API Provider Issues**:
- Rate limit exceeded (429)
- Service unavailable (503)
- Timeout errors
- Authentication failures
- Model overload

**Application Issues**:
- Invalid prompts or parameters
- Token limit exceeded
- Network failures
- Memory constraints
- Concurrent request limits

**Data Issues**:
- Malformed responses
- Missing or null data
- Encoding problems
- Content policy violations

### What You'll Learn

By the end of this guide, you'll be able to:

✅ Handle all types of AI API errors gracefully  
✅ Implement intelligent retry strategies  
✅ Use circuit breakers to prevent cascading failures  
✅ Manage timeouts and long-running operations  
✅ Build fallback mechanisms for reliability  
✅ Monitor and debug AI application issues  
✅ Deploy production-ready, resilient AI systems  

Let's build bulletproof AI applications!

---

## Understanding AI Application Failures

### Error Categories

**1. Transient Errors (Retry-able)**:
```java
// These errors are temporary and should be retried
- Rate limit exceeded (429)
- Service temporarily unavailable (503)
- Gateway timeout (504)
- Connection timeout
- Network interruption
```

**2. Permanent Errors (Non-retry-able)**:
```java
// These errors won't resolve with retries
- Invalid API key (401)
- Model not found (404)
- Invalid request format (400)
- Content policy violation
- Token limit exceeded
```

**3. Application Errors**:
```java
// Errors in your code or configuration
- Null pointer exceptions
- Invalid prompts
- Configuration errors
- Resource exhaustion
```

### Spring AI Exception Hierarchy

```java
// Base exception
org.springframework.ai.model.ModelException
    |
    ├── OpenAiException
    │   ├── OpenAiApiException (API errors)
    │   ├── OpenAiRateLimitException (429)
    │   └── OpenAiAuthenticationException (401)
    |
    ├── AzureOpenAiException
    |
    ├── AnthropicException
    |
    └── EmbeddingException
```

**Understanding Exception Details**:

```java
try {
    ChatResponse response = chatClient.call(prompt);
} catch (OpenAiApiException e) {
    // HTTP status code
    int statusCode = e.getStatusCode();
    
    // Error message from API
    String message = e.getMessage();
    
    // Original request (for retry)
    ChatRequest request = e.getRequest();
    
    // API error type
    String errorType = e.getErrorType();
    
    log.error("OpenAI API error: {} (status: {})", message, statusCode, e);
}
```

### Common Error Codes

| Status Code | Meaning | Action |
|-------------|---------|--------|
| **400** | Bad Request | Fix prompt/parameters, don't retry |
| **401** | Unauthorized | Check API key, don't retry |
| **403** | Forbidden | Check permissions, don't retry |
| **404** | Not Found | Check model name, don't retry |
| **429** | Rate Limit | Retry with backoff |
| **500** | Server Error | Retry with backoff |
| **503** | Service Unavailable | Retry with backoff |
| **504** | Gateway Timeout | Retry with backoff |

---

## Basic Error Handling

### Try-Catch Pattern

**Simple error handling**:

```java
package com.example.aiapp.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.openai.api.OpenAiApiException;
import org.springframework.stereotype.Service;

@Service
public class ChatService {
    
    private final ChatClient chatClient;
    
    public ChatService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }
    
    /**
     * Basic error handling with try-catch.
     */
    public String chat(String userMessage) {
        try {
            return chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        } catch (OpenAiApiException e) {
            log.error("OpenAI API error: {}", e.getMessage(), e);
            
            // Return user-friendly message
            return "I apologize, but I'm experiencing technical difficulties. " +
                   "Please try again in a moment.";
                   
        } catch (Exception e) {
            log.error("Unexpected error during chat", e);
            
            return "An unexpected error occurred. Our team has been notified.";
        }
    }
}
```

### Granular Error Handling

**Handle specific error types**:

```java
@Service
public class RobustChatService {
    
    private final ChatClient chatClient;
    
    public String chat(String userMessage) {
        try {
            return chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        } catch (OpenAiRateLimitException e) {
            // Rate limit exceeded
            log.warn("Rate limit exceeded, user should retry later", e);
            
            return "I'm currently handling a high volume of requests. " +
                   "Please try again in a few moments.";
                   
        } catch (OpenAiAuthenticationException e) {
            // API key issues
            log.error("Authentication failed - check API key", e);
            
            // Alert ops team
            alertingService.sendAlert("OpenAI authentication failed");
            
            return "Service temporarily unavailable. We're working to resolve this.";
            
        } catch (OpenAiApiException e) {
            // Other API errors
            if (e.getStatusCode() == 400) {
                log.error("Invalid request: {}", e.getMessage(), e);
                return "I couldn't process that request. Could you rephrase?";
            }
            
            if (e.getStatusCode() >= 500) {
                log.error("OpenAI server error: {}", e.getMessage(), e);
                return "The AI service is temporarily unavailable. Please try again shortly.";
            }
            
            log.error("OpenAI API error: {}", e.getMessage(), e);
            return "I encountered an issue processing your request.";
            
        } catch (Exception e) {
            log.error("Unexpected error", e);
            
            // Don't expose internal errors to users
            return "An unexpected error occurred.";
        }
    }
}
```

### Custom Exception Handling

**Create application-specific exceptions**:

```java
package com.example.aiapp.exception;

public class AiServiceException extends RuntimeException {
    
    private final ErrorCode errorCode;
    private final boolean retryable;
    
    public AiServiceException(ErrorCode errorCode, String message, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
        this.retryable = errorCode.isRetryable();
    }
    
    public enum ErrorCode {
        RATE_LIMIT_EXCEEDED(true, "Rate limit exceeded"),
        SERVICE_UNAVAILABLE(true, "Service temporarily unavailable"),
        AUTHENTICATION_FAILED(false, "Authentication failed"),
        INVALID_REQUEST(false, "Invalid request"),
        MODEL_NOT_FOUND(false, "Model not found"),
        CONTENT_VIOLATION(false, "Content policy violation"),
        TIMEOUT(true, "Request timeout"),
        UNKNOWN(false, "Unknown error");
        
        private final boolean retryable;
        private final String description;
        
        ErrorCode(boolean retryable, String description) {
            this.retryable = retryable;
            this.description = description;
        }
        
        public boolean isRetryable() {
            return retryable;
        }
        
        public String getDescription() {
            return description;
        }
    }
    
    public ErrorCode getErrorCode() {
        return errorCode;
    }
    
    public boolean isRetryable() {
        return retryable;
    }
}
```

**Using custom exceptions**:

```java
@Service
public class ChatServiceWithCustomExceptions {
    
    private final ChatClient chatClient;
    
    public String chat(String userMessage) {
        try {
            return chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        } catch (OpenAiRateLimitException e) {
            throw new AiServiceException(
                ErrorCode.RATE_LIMIT_EXCEEDED,
                "Rate limit exceeded",
                e
            );
            
        } catch (OpenAiAuthenticationException e) {
            throw new AiServiceException(
                ErrorCode.AUTHENTICATION_FAILED,
                "Authentication failed",
                e
            );
            
        } catch (OpenAiApiException e) {
            ErrorCode errorCode = mapStatusCodeToErrorCode(e.getStatusCode());
            throw new AiServiceException(errorCode, e.getMessage(), e);
            
        } catch (Exception e) {
            throw new AiServiceException(
                ErrorCode.UNKNOWN,
                "Unexpected error",
                e
            );
        }
    }
    
    private ErrorCode mapStatusCodeToErrorCode(int statusCode) {
        return switch (statusCode) {
            case 400 -> ErrorCode.INVALID_REQUEST;
            case 401 -> ErrorCode.AUTHENTICATION_FAILED;
            case 404 -> ErrorCode.MODEL_NOT_FOUND;
            case 429 -> ErrorCode.RATE_LIMIT_EXCEEDED;
            case 503 -> ErrorCode.SERVICE_UNAVAILABLE;
            default -> ErrorCode.UNKNOWN;
        };
    }
}
```

### Global Exception Handler

**Centralized error handling with @ControllerAdvice**:

```java
package com.example.aiapp.controller;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(AiServiceException.class)
    public ResponseEntity<ErrorResponse> handleAiServiceException(
            AiServiceException e) {
        
        HttpStatus status = e.isRetryable() 
            ? HttpStatus.SERVICE_UNAVAILABLE 
            : HttpStatus.BAD_REQUEST;
        
        ErrorResponse response = new ErrorResponse(
            e.getErrorCode().name(),
            e.getMessage(),
            e.isRetryable(),
            e.isRetryable() ? 60 : null  // Retry after 60 seconds
        );
        
        return ResponseEntity
            .status(status)
            .body(response);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception e) {
        log.error("Unhandled exception", e);
        
        ErrorResponse response = new ErrorResponse(
            "INTERNAL_ERROR",
            "An unexpected error occurred",
            false,
            null
        );
        
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(response);
    }
    
    public record ErrorResponse(
        String errorCode,
        String message,
        boolean retryable,
        Integer retryAfterSeconds
    ) {}
}
```

---

## Retry Strategies

### Spring Retry Integration

**Add dependency**:

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
</dependency>
```

**Enable retry**:

```java
@SpringBootApplication
@EnableRetry
public class AiApplication {
    public static void main(String[] args) {
        SpringApplication.run(AiApplication.class, args);
    }
}
```

### Basic Retry Pattern

**Simple retry with @Retryable**:

```java
@Service
public class RetryableChatService {
    
    private final ChatClient chatClient;
    
    /**
     * Retry up to 3 times with 2 second delay.
     */
    @Retryable(
        retryFor = {OpenAiApiException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 2000)
    )
    public String chat(String userMessage) {
        log.info("Attempting chat request...");
        
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
    
    /**
     * Fallback method called after all retries exhausted.
     */
    @Recover
    public String recoverFromChatFailure(OpenAiApiException e, String userMessage) {
        log.error("All retry attempts exhausted", e);
        
        return "I'm currently unable to process your request. " +
               "Please try again later.";
    }
}
```

### Exponential Backoff

**Increase delay between retries**:

```java
@Service
public class ExponentialBackoffService {
    
    /**
     * Retry with exponential backoff:
     * - Attempt 1: immediate
     * - Attempt 2: wait 1 second
     * - Attempt 3: wait 2 seconds
     * - Attempt 4: wait 4 seconds
     * - Attempt 5: wait 8 seconds
     */
    @Retryable(
        retryFor = {
            OpenAiRateLimitException.class,
            OpenAiApiException.class
        },
        maxAttempts = 5,
        backoff = @Backoff(
            delay = 1000,        // Initial delay: 1 second
            multiplier = 2.0,    // Double each time
            maxDelay = 10000     // Cap at 10 seconds
        )
    )
    public String chatWithBackoff(String userMessage) {
        log.info("Chat attempt with exponential backoff");
        
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
}
```

### Conditional Retry

**Retry only for specific errors**:

```java
@Service
public class ConditionalRetryService {
    
    /**
     * Custom retry logic based on exception type.
     */
    @Retryable(
        retryFor = {OpenAiApiException.class},
        maxAttempts = 5,
        backoff = @Backoff(delay = 2000, multiplier = 1.5),
        listeners = {"retryListener"}
    )
    public String chatWithConditionalRetry(String userMessage) {
        try {
            return chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        } catch (OpenAiApiException e) {
            // Don't retry for certain status codes
            if (e.getStatusCode() == 400 || 
                e.getStatusCode() == 401 || 
                e.getStatusCode() == 404) {
                throw new NonRetryableException(
                    "Permanent error, won't retry", e
                );
            }
            
            // Retry for other errors
            throw e;
        }
    }
}

/**
 * Mark exception as non-retryable.
 */
public class NonRetryableException extends RuntimeException {
    public NonRetryableException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Programmatic Retry

**Manual retry control with RetryTemplate**:

```java
@Configuration
public class RetryConfig {
    
    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();
        
        // Configure retry policy
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(3);
        retryTemplate.setRetryPolicy(retryPolicy);
        
        // Configure backoff policy
        ExponentialBackOffPolicy backOffPolicy = new ExponentialBackOffPolicy();
        backOffPolicy.setInitialInterval(1000);
        backOffPolicy.setMultiplier(2.0);
        backOffPolicy.setMaxInterval(10000);
        retryTemplate.setBackOffPolicy(backOffPolicy);
        
        // Add listeners
        retryTemplate.registerListener(new RetryListenerSupport() {
            @Override
            public <T, E extends Throwable> void onError(
                    RetryContext context,
                    RetryCallback<T, E> callback,
                    Throwable throwable) {
                log.warn("Retry attempt {} failed: {}",
                    context.getRetryCount(),
                    throwable.getMessage()
                );
            }
        });
        
        return retryTemplate;
    }
}

@Service
public class ProgrammaticRetryService {
    
    private final RetryTemplate retryTemplate;
    private final ChatClient chatClient;
    
    public String chat(String userMessage) {
        return retryTemplate.execute(context -> {
            log.info("Attempt {}", context.getRetryCount() + 1);
            
            return chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        }, context -> {
            // Recovery callback
            log.error("All retries failed after {} attempts",
                context.getRetryCount()
            );
            
            return "Service temporarily unavailable. Please try again later.";
        });
    }
}
```

### Advanced Retry Configuration

**Custom retry policy**:

```java
@Configuration
public class AdvancedRetryConfig {
    
    @Bean
    public RetryTemplate customRetryTemplate() {
        RetryTemplate template = new RetryTemplate();
        
        // Retry different exceptions differently
        Map<Class<? extends Throwable>, Boolean> retryableExceptions = 
            new HashMap<>();
        retryableExceptions.put(OpenAiRateLimitException.class, true);
        retryableExceptions.put(TimeoutException.class, true);
        retryableExceptions.put(OpenAiAuthenticationException.class, false);
        
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy(
            5,  // max attempts
            retryableExceptions,
            true  // traverse causes
        );
        
        template.setRetryPolicy(retryPolicy);
        
        // Random backoff with jitter
        ExponentialRandomBackOffPolicy backOffPolicy = 
            new ExponentialRandomBackOffPolicy();
        backOffPolicy.setInitialInterval(1000);
        backOffPolicy.setMultiplier(2.0);
        backOffPolicy.setMaxInterval(15000);
        
        template.setBackOffPolicy(backOffPolicy);
        
        return template;
    }
}
```

---

## Circuit Breaker Pattern

Circuit breakers prevent cascading failures by stopping requests to failing services.

### Resilience4j Integration

**Add dependency**:

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.1.0</version>
</dependency>
```

**Configuration**:

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      chatService:
        register-health-indicator: true
        sliding-window-size: 10
        minimum-number-of-calls: 5
        permitted-number-of-calls-in-half-open-state: 3
        automatic-transition-from-open-to-half-open-enabled: true
        wait-duration-in-open-state: 30s
        failure-rate-threshold: 50
        slow-call-rate-threshold: 100
        slow-call-duration-threshold: 10s
        record-exceptions:
          - org.springframework.ai.openai.api.OpenAiApiException
        ignore-exceptions:
          - com.example.aiapp.exception.NonRetryableException
```

### Basic Circuit Breaker

```java
@Service
public class CircuitBreakerChatService {
    
    private final ChatClient chatClient;
    private final CircuitBreakerRegistry circuitBreakerRegistry;
    
    public String chat(String userMessage) {
        CircuitBreaker circuitBreaker = circuitBreakerRegistry
            .circuitBreaker("chatService");
        
        return circuitBreaker.executeSupplier(() -> 
            chatClient.prompt()
                .user(userMessage)
                .call()
                .content()
        );
    }
}
```

### Circuit Breaker with Fallback

```java
@Service
public class ResilientChatService {
    
    private final ChatClient chatClient;
    private final CacheService cacheService;
    
    @CircuitBreaker(name = "chatService", fallbackMethod = "chatFallback")
    public String chat(String userMessage) {
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
    
    /**
     * Fallback when circuit is open.
     */
    private String chatFallback(String userMessage, Exception e) {
        log.warn("Circuit breaker activated, using fallback", e);
        
        // Try to serve from cache
        Optional<String> cached = cacheService.getCachedResponse(userMessage);
        if (cached.isPresent()) {
            log.info("Serving cached response");
            return cached.get();
        }
        
        // Return helpful message
        return "Our AI service is currently experiencing high demand. " +
               "Please try again in a moment.";
    }
}
```

### Circuit Breaker States

```java
@Service
public class CircuitBreakerMonitoringService {
    
    private final CircuitBreakerRegistry registry;
    
    @EventListener
    public void handleCircuitBreakerEvent(CircuitBreakerOnStateTransitionEvent event) {
        CircuitBreaker.State fromState = event.getStateTransition().getFromState();
        CircuitBreaker.State toState = event.getStateTransition().getToState();
        
        log.info("Circuit breaker {} transitioned from {} to {}",
            event.getCircuitBreakerName(),
            fromState,
            toState
        );
        
        if (toState == CircuitBreaker.State.OPEN) {
            // Alert ops team
            alertingService.sendAlert(
                "Circuit breaker OPEN for " + event.getCircuitBreakerName()
            );
        }
        
        if (toState == CircuitBreaker.State.CLOSED) {
            // Service recovered
            alertingService.sendAlert(
                "Circuit breaker CLOSED - service recovered"
            );
        }
    }
    
    /**
     * Check circuit breaker health.
     */
    public Map<String, CircuitBreakerStatus> getCircuitBreakerStatus() {
        return registry.getAllCircuitBreakers()
            .stream()
            .collect(Collectors.toMap(
                CircuitBreaker::getName,
                cb -> new CircuitBreakerStatus(
                    cb.getState(),
                    cb.getMetrics().getFailureRate(),
                    cb.getMetrics().getNumberOfFailedCalls(),
                    cb.getMetrics().getNumberOfSuccessfulCalls()
                )
            ));
    }
    
    public record CircuitBreakerStatus(
        CircuitBreaker.State state,
        float failureRate,
        long failedCalls,
        long successfulCalls
    ) {}
}
```

### Combining Retry and Circuit Breaker

```java
@Service
public class CombinedResilienceService {
    
    /**
     * First retry, then circuit breaker.
     * Order matters!
     */
    @CircuitBreaker(name = "chatService", fallbackMethod = "fallback")
    @Retry(name = "chatService")
    public String chat(String userMessage) {
        log.info("Executing chat request");
        
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
    
    private String fallback(String userMessage, Exception e) {
        log.error("All resilience mechanisms exhausted", e);
        
        return "Service temporarily unavailable.";
    }
}
```

**Configuration for combined pattern**:

```yaml
resilience4j:
  retry:
    instances:
      chatService:
        max-attempts: 3
        wait-duration: 1s
        enable-exponential-backoff: true
        exponential-backoff-multiplier: 2
        
  circuitbreaker:
    instances:
      chatService:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 30s
        sliding-window-size: 10
```

---

## Timeout Management

### Request Timeouts

**Configure timeouts in Spring AI**:

```java
@Configuration
public class AiClientConfig {
    
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder) {
        return builder
            .defaultOptions(OpenAiChatOptions.builder()
                .withModel("gpt-4o-mini")
                .withTemperature(0.7)
                // Add custom timeout via HTTP client
                .build())
            .build();
    }
    
    @Bean
    public RestClient.Builder restClientBuilder() {
        return RestClient.builder()
            .requestFactory(new HttpComponentsClientHttpRequestFactory(
                HttpClientBuilder.create()
                    .setConnectionTimeToLive(30, TimeUnit.SECONDS)
                    .setDefaultRequestConfig(
                        RequestConfig.custom()
                            .setConnectTimeout(5000)        // 5s connection timeout
                            .setResponseTimeout(30000)       // 30s response timeout
                            .build()
                    )
                    .build()
            ));
    }
}
```

### Programmatic Timeouts

```java
@Service
public class TimeoutChatService {
    
    private final ChatClient chatClient;
    private final ExecutorService executor;
    
    public TimeoutChatService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
        this.executor = Executors.newCachedThreadPool();
    }
    
    /**
     * Chat with timeout using CompletableFuture.
     */
    public String chatWithTimeout(String userMessage, Duration timeout) 
            throws TimeoutException {
        
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() ->
            chatClient.prompt()
                .user(userMessage)
                .call()
                .content(),
            executor
        );
        
        try {
            return future.get(timeout.toMillis(), TimeUnit.MILLISECONDS);
            
        } catch (java.util.concurrent.TimeoutException e) {
            future.cancel(true);
            throw new TimeoutException("Request timed out after " + timeout);
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("Request interrupted", e);
            
        } catch (ExecutionException e) {
            throw new RuntimeException("Request failed", e.getCause());
        }
    }
}
```

### Resilience4j TimeLimiter

```java
@Service
public class TimeLimitedChatService {
    
    private final ChatClient chatClient;
    
    @TimeLimiter(name = "chatService", fallbackMethod = "timeoutFallback")
    public CompletableFuture<String> chatAsync(String userMessage) {
        return CompletableFuture.supplyAsync(() ->
            chatClient.prompt()
                .user(userMessage)
                .call()
                .content()
        );
    }
    
    private CompletableFuture<String> timeoutFallback(
            String userMessage,
            TimeoutException e) {
        
        log.warn("Request timed out", e);
        
        return CompletableFuture.completedFuture(
            "Your request is taking longer than expected. " +
            "Please try a simpler query or try again later."
        );
    }
}
```

**Configuration**:

```yaml
resilience4j:
  timelimiter:
    instances:
      chatService:
        timeout-duration: 30s
        cancel-running-future: true
```

---

## Fallback Mechanisms

### Multi-Provider Fallback

**Switch to alternative AI provider**:

```java
@Service
public class MultiProviderChatService {
    
    private final ChatClient openAiClient;
    private final ChatClient anthropicClient;
    private final ChatClient azureClient;
    
    public String chat(String userMessage) {
        // Try OpenAI first
        try {
            return openAiClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        } catch (Exception e) {
            log.warn("OpenAI failed, trying Anthropic", e);
        }
        
        // Fallback to Anthropic
        try {
            return anthropicClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        } catch (Exception e) {
            log.warn("Anthropic failed, trying Azure", e);
        }
        
        // Fallback to Azure
        try {
            return azureClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        } catch (Exception e) {
            log.error("All AI providers failed", e);
            throw new AiServiceException(
                ErrorCode.SERVICE_UNAVAILABLE,
                "All AI services unavailable",
                e
            );
        }
    }
}
```

### Cache-Based Fallback

```java
@Service
public class CachedFallbackService {
    
    private final ChatClient chatClient;
    private final Cache<String, String> responseCache;
    
    public CachedFallbackService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
        this.responseCache = CacheBuilder.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(Duration.ofHours(24))
            .build();
    }
    
    public String chat(String userMessage) {
        try {
            // Try live API
            String response = chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
            
            // Cache successful response
            responseCache.put(userMessage, response);
            
            return response;
            
        } catch (Exception e) {
            log.warn("API failed, checking cache", e);
            
            // Try cache
            String cached = responseCache.getIfPresent(userMessage);
            if (cached != null) {
                log.info("Serving cached response");
                return "[Cached] " + cached;
            }
            
            // Try similar queries
            String similar = findSimilarCachedResponse(userMessage);
            if (similar != null) {
                log.info("Serving similar cached response");
                return "[From cache - similar query] " + similar;
            }
            
            throw new AiServiceException(
                ErrorCode.SERVICE_UNAVAILABLE,
                "No cached response available",
                e
            );
        }
    }
    
    private String findSimilarCachedResponse(String query) {
        // Simple similarity check (in production, use embeddings)
        return responseCache.asMap().entrySet().stream()
            .filter(entry -> calculateSimilarity(query, entry.getKey()) > 0.8)
            .findFirst()
            .map(Map.Entry::getValue)
            .orElse(null);
    }
}
```

### Template-Based Fallback

```java
@Service
public class TemplateFallbackService {
    
    private final ChatClient chatClient;
    private final Map<String, String> templates;
    
    public TemplateFallbackService() {
        // Predefined templates for common scenarios
        this.templates = Map.of(
            "greeting", "Hello! How can I assist you today?",
            "weather", "I can help you check the weather. Which city are you interested in?",
            "order_status", "I can help you track your order. Please provide your order number.",
            "support", "I'm here to help! Could you please describe your issue?",
            "default", "I apologize, but I'm experiencing technical difficulties. Please try again shortly."
        );
    }
    
    public String chat(String userMessage) {
        try {
            return chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
                
        } catch (Exception e) {
            log.warn("Using template fallback", e);
            
            // Determine intent and use appropriate template
            String intent = determineIntent(userMessage);
            return templates.getOrDefault(intent, templates.get("default"));
        }
    }
    
    private String determineIntent(String message) {
        message = message.toLowerCase();
        
        if (message.contains("hello") || message.contains("hi")) {
            return "greeting";
        }
        if (message.contains("weather")) {
            return "weather";
        }
        if (message.contains("order") || message.contains("track")) {
            return "order_status";
        }
        
        return "default";
    }
}
```

---

## Rate Limiting and Throttling

### Token Bucket Rate Limiter

```java
@Service
public class RateLimitedChatService {
    
    private final ChatClient chatClient;
    private final RateLimiter rateLimiter;
    
    public RateLimitedChatService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
        
        // Allow 10 requests per minute
        this.rateLimiter = RateLimiter.create(10.0 / 60.0);
    }
    
    public String chat(String userMessage) {
        // Try to acquire permit
        if (!rateLimiter.tryAcquire(Duration.ofSeconds(1))) {
            throw new AiServiceException(
                ErrorCode.RATE_LIMIT_EXCEEDED,
                "Rate limit exceeded. Please try again later.",
                null
            );
        }
        
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
}
```

### Per-User Rate Limiting

```java
@Service
public class PerUserRateLimiter {
    
    private final LoadingCache<String, RateLimiter> userRateLimiters;
    private final ChatClient chatClient;
    
    public PerUserRateLimiter(ChatClient.Builder builder) {
        this.chatClient = builder.build();
        this.userRateLimiters = CacheBuilder.newBuilder()
            .expireAfterAccess(Duration.ofHours(1))
            .build(CacheLoader.from(userId -> 
                RateLimiter.create(20.0 / 60.0)  // 20 per minute per user
            ));
    }
    
    public String chat(String userId, String userMessage) {
        RateLimiter limiter = userRateLimiters.getUnchecked(userId);
        
        if (!limiter.tryAcquire()) {
            throw new AiServiceException(
                ErrorCode.RATE_LIMIT_EXCEEDED,
                "You've exceeded your rate limit. Please wait before trying again.",
                null
            );
        }
        
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
}
```

### Resilience4j RateLimiter

```yaml
resilience4j:
  ratelimiter:
    instances:
      chatService:
        limit-for-period: 10
        limit-refresh-period: 1s
        timeout-duration: 0s  # Fail immediately if no permits
```

```java
@Service
public class Resilience4jRateLimitService {
    
    @RateLimiter(name = "chatService")
    public String chat(String userMessage) {
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
}
```

---

## Monitoring and Observability

### Logging Best Practices

```java
@Service
@Slf4j
public class ObservableChatService {
    
    private final ChatClient chatClient;
    
    public String chat(String userId, String userMessage) {
        String requestId = UUID.randomUUID().toString();
        
        try (MDC.MDCCloseable ignored1 = MDC.putCloseable("requestId", requestId);
             MDC.MDCCloseable ignored2 = MDC.putCloseable("userId", userId)) {
            
            log.info("Chat request received: {}", 
                truncate(userMessage, 100));
            
            long startTime = System.currentTimeMillis();
            
            String response = chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
            
            long duration = System.currentTimeMillis() - startTime;
            
            log.info("Chat request completed in {}ms, response length: {}", 
                duration, 
                response.length()
            );
            
            return response;
            
        } catch (Exception e) {
            log.error("Chat request failed", e);
            throw e;
        }
    }
    
    private String truncate(String text, int maxLength) {
        return text.length() > maxLength 
            ? text.substring(0, maxLength) + "..." 
            : text;
    }
}
```

### Metrics Collection

```java
@Service
public class MetricsCollectingChatService {
    
    private final ChatClient chatClient;
    private final MeterRegistry meterRegistry;
    
    public String chat(String userMessage) {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try {
            String response = chatClient.prompt()
                .user(userMessage)
                .call()
                .content();
            
            // Record success
            sample.stop(Timer.builder("ai.chat.requests")
                .tag("status", "success")
                .register(meterRegistry));
            
            // Record response length
            meterRegistry.counter("ai.chat.tokens.response",
                "status", "success"
            ).increment(estimateTokens(response));
            
            return response;
            
        } catch (OpenAiRateLimitException e) {
            sample.stop(Timer.builder("ai.chat.requests")
                .tag("status", "rate_limited")
                .register(meterRegistry));
            
            meterRegistry.counter("ai.chat.errors",
                "type", "rate_limit"
            ).increment();
            
            throw e;
            
        } catch (Exception e) {
            sample.stop(Timer.builder("ai.chat.requests")
                .tag("status", "error")
                .register(meterRegistry));
            
            meterRegistry.counter("ai.chat.errors",
                "type", e.getClass().getSimpleName()
            ).increment();
            
            throw e;
        }
    }
    
    private int estimateTokens(String text) {
        // Rough estimate: ~4 characters per token
        return text.length() / 4;
    }
}
```

### Health Checks

```java
@Component
public class AiServiceHealthIndicator implements HealthIndicator {
    
    private final ChatClient chatClient;
    private final AtomicLong lastSuccessTime = new AtomicLong(System.currentTimeMillis());
    private final AtomicInteger consecutiveFailures = new AtomicInteger(0);
    
    @Override
    public Health health() {
        try {
            // Simple health check with timeout
            String response = CompletableFuture.supplyAsync(() ->
                    chatClient.prompt()
                        .user("ping")
                        .call()
                        .content()
                )
                .get(5, TimeUnit.SECONDS);
            
            lastSuccessTime.set(System.currentTimeMillis());
            consecutiveFailures.set(0);
            
            return Health.up()
                .withDetail("status", "AI service is responsive")
                .withDetail("lastSuccess", Instant.ofEpochMilli(lastSuccessTime.get()))
                .build();
                
        } catch (Exception e) {
            int failures = consecutiveFailures.incrementAndGet();
            long timeSinceSuccess = System.currentTimeMillis() - lastSuccessTime.get();
            
            if (failures > 5 || timeSinceSuccess > 300000) {  // 5 minutes
                return Health.down()
                    .withDetail("error", e.getMessage())
                    .withDetail("consecutiveFailures", failures)
                    .withDetail("timeSinceSuccess", Duration.ofMillis(timeSinceSuccess))
                    .build();
            }
            
            return Health.up()
                .withDetail("warning", "Experiencing intermittent issues")
                .withDetail("consecutiveFailures", failures)
                .build();
        }
    }
}
```

---

## Production Best Practices

### Complete Production Service

```java
@Service
@Slf4j
public class ProductionChatService {
    
    private final ChatClient chatClient;
    private final MeterRegistry metrics;
    private final Cache<String, String> responseCache;
    private final RateLimiter rateLimiter;
    private final CircuitBreaker circuitBreaker;
    
    public ProductionChatService(
            ChatClient.Builder builder,
            MeterRegistry metrics,
            CircuitBreakerRegistry circuitBreakerRegistry) {
        
        this.chatClient = builder.build();
        this.metrics = metrics;
        this.circuitBreaker = circuitBreakerRegistry.circuitBreaker("chatService");
        this.rateLimiter = RateLimiter.create(100.0 / 60.0);  // 100 per minute
        this.responseCache = CacheBuilder.newBuilder()
            .maximumSize(10000)
            .expireAfterWrite(Duration.ofHours(1))
            .recordStats()
            .build();
    }
    
    @Retryable(
        retryFor = {OpenAiRateLimitException.class, TimeoutException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public String chat(String userId, String userMessage) {
        String requestId = UUID.randomUUID().toString();
        
        try (MDC.MDCCloseable reqId = MDC.putCloseable("requestId", requestId);
             MDC.MDCCloseable user = MDC.putCloseable("userId", userId)) {
            
            // Rate limiting
            if (!rateLimiter.tryAcquire(Duration.ofSeconds(2))) {
                metrics.counter("ai.chat.rate_limited").increment();
                throw new AiServiceException(
                    ErrorCode.RATE_LIMIT_EXCEEDED,
                    "Rate limit exceeded",
                    null
                );
            }
            
            // Check cache
            String cached = responseCache.getIfPresent(userMessage);
            if (cached != null) {
                metrics.counter("ai.chat.cache_hit").increment();
                log.info("Cache hit for request");
                return cached;
            }
            metrics.counter("ai.chat.cache_miss").increment();
            
            // Execute with circuit breaker
            Timer.Sample sample = Timer.start(metrics);
            
            try {
                String response = circuitBreaker.executeSupplier(() ->
                    chatClient.prompt()
                        .user(userMessage)
                        .call()
                        .content()
                );
                
                // Cache response
                responseCache.put(userMessage, response);
                
                // Record metrics
                sample.stop(Timer.builder("ai.chat.duration")
                    .tag("status", "success")
                    .register(metrics));
                
                metrics.counter("ai.chat.success").increment();
                
                log.info("Request completed successfully");
                return response;
                
            } catch (Exception e) {
                sample.stop(Timer.builder("ai.chat.duration")
                    .tag("status", "error")
                    .tag("error_type", e.getClass().getSimpleName())
                    .register(metrics));
                
                metrics.counter("ai.chat.errors",
                    "type", e.getClass().getSimpleName()
                ).increment();
                
                throw e;
            }
        }
    }
    
    @Recover
    public String recoverFromFailure(Exception e, String userId, String userMessage) {
        log.error("All retry attempts exhausted for user {}", userId, e);
        
        metrics.counter("ai.chat.exhausted").increment();
        
        // Try fallback mechanisms
        String cached = responseCache.getIfPresent(userMessage);
        if (cached != null) {
            return "[Cached] " + cached;
        }
        
        return "I apologize, but I'm currently experiencing technical difficulties. " +
               "Please try again in a few moments.";
    }
}
```

### Configuration for Production

```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4o-mini
          temperature: 0.7
          max-tokens: 2000

resilience4j:
  retry:
    instances:
      chatService:
        max-attempts: 3
        wait-duration: 1s
        enable-exponential-backoff: true
        exponential-backoff-multiplier: 2
        retry-exceptions:
          - org.springframework.ai.openai.api.OpenAiRateLimitException
          - java.util.concurrent.TimeoutException
        
  circuitbreaker:
    instances:
      chatService:
        failure-rate-threshold: 50
        slow-call-rate-threshold: 80
        slow-call-duration-threshold: 10s
        wait-duration-in-open-state: 60s
        permitted-number-of-calls-in-half-open-state: 3
        sliding-window-type: COUNT_BASED
        sliding-window-size: 10
        minimum-number-of-calls: 5
        automatic-transition-from-open-to-half-open-enabled: true
        
  timelimiter:
    instances:
      chatService:
        timeout-duration: 30s
        cancel-running-future: true

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

---

## Testing Error Scenarios

### Unit Tests

```java
@ExtendWith(MockitoExtension.class)
class ChatServiceErrorHandlingTest {
    
    @Mock
    private ChatClient chatClient;
    
    @InjectMocks
    private ChatService chatService;
    
    @Test
    void shouldHandleRateLimitError() {
        // Arrange
        when(chatClient.prompt().user(anyString()).call())
            .thenThrow(new OpenAiRateLimitException("Rate limit exceeded"));
        
        // Act & Assert
        assertThatThrownBy(() -> chatService.chat("test message"))
            .isInstanceOf(AiServiceException.class)
            .hasFieldOrPropertyWithValue("errorCode", ErrorCode.RATE_LIMIT_EXCEEDED);
    }
    
    @Test
    void shouldRetryOnTransientErrors() {
        // Arrange
        ChatClient.ChatClientRequest request = mock(ChatClient.ChatClientRequest.class);
        ChatClient.ChatClientRequest.CallPromptResponseSpec spec = 
            mock(ChatClient.ChatClientRequest.CallPromptResponseSpec.class);
        
        when(chatClient.prompt()).thenReturn(request);
        when(request.user(anyString())).thenReturn(request);
        when(request.call()).thenReturn(spec);
        when(spec.content())
            .thenThrow(new RuntimeException("Timeout"))
            .thenThrow(new RuntimeException("Timeout"))
            .thenReturn("Success!");
        
        // Act
        String result = chatService.chatWithRetry("test");
        
        // Assert
        assertThat(result).isEqualTo("Success!");
        verify(spec, times(3)).content();
    }
}
```

### Integration Tests

```java
@SpringBootTest
class ErrorHandlingIntegrationTest {
    
    @Autowired
    private ChatService chatService;
    
    @MockBean
    private ChatClient chatClient;
    
    @Test
    void shouldFallbackAfterMultipleFailures() {
        // Simulate persistent failures
        when(chatClient.prompt().user(anyString()).call())
            .thenThrow(new RuntimeException("Service unavailable"));
        
        String response = chatService.chat("test");
        
        assertThat(response).contains("technical difficulties");
    }
}
```

---

## Conclusion

Error handling is crucial for production AI applications. Key takeaways:

**Essential Patterns**:
✅ Retry with exponential backoff  
✅ Circuit breakers for cascading failure prevention  
✅ Graceful fallbacks (cache, templates, alternative providers)  
✅ Rate limiting to prevent abuse  
✅ Comprehensive monitoring and alerting  

**Production Checklist**:
✅ Handle all exception types appropriately  
✅ Implement retries for transient errors  
✅ Use circuit breakers to protect downstream services  
✅ Configure appropriate timeouts  
✅ Cache responses when possible  
✅ Monitor error rates and patterns  
✅ Test failure scenarios  
✅ Have fallback mechanisms ready  

Your Spring AI application is now resilient and production-ready! 🚀