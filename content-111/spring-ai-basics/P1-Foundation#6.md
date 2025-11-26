
基于下面的信息，给出英文技术博客文章（面向欧美用户，基于 Google Adsense赚钱）：
文章为主，代码为辅。
要有图表和表格。

Reference Title: Spring AI Function Calling Tutorial: Build Your First AI Agent
Reference Keywords: spring ai function calling
Target Word Count: 5000-6000

# Spring AI Function Calling Tutorial: Build Your First AI Agent

**Transform Your AI from Chat to Action: A Complete Guide to Function Calling in Spring AI**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Understanding Function Calling](#understanding-function-calling)
3. [Setup and Prerequisites](#setup-and-prerequisites)
4. [Your First Function](#your-first-function)
5. [Function Registration and Discovery](#function-registration-and-discovery)
6. [Building Real-World Functions](#building-real-world-functions)
7. [Advanced Function Patterns](#advanced-function-patterns)
8. [Building a Complete AI Agent](#building-a-complete-ai-agent)
9. [Multi-Step Workflows](#multi-step-workflows)
10. [Error Handling and Validation](#error-handling-and-validation)
11. [Testing Function Calling](#testing-function-calling)
12. [Security and Best Practices](#security-and-best-practices)
13. [Production Deployment](#production-deployment)
14. [Conclusion](#conclusion)

---

## Introduction

Function calling is the game-changer that transforms AI from a passive chatbot into an active agent capable of taking real actions. With Spring AI's function calling, your AI can check weather, send emails, query databases, make API calls, and interact with any part of your system—all through natural language.

### What is Function Calling?

**Traditional Chat**:
```
User: "What's the weather in Paris?"
AI: "I don't have access to real-time weather data..."
```

**With Function Calling**:
```
User: "What's the weather in Paris?"
AI: [calls getCurrentWeather("Paris")]
    → {temp: 18°C, condition: "partly cloudy"}
AI: "The current weather in Paris is 18°C and partly cloudy."
```

The AI automatically decides when to call functions, what parameters to use, and how to present the results in natural language.

### Why Function Calling Matters

**Real-World Applications**:

🤖 **Personal Assistants** - Schedule meetings, send emails, set reminders  
📊 **Data Analysis** - Query databases, generate reports, create visualizations  
🛒 **E-commerce** - Check inventory, place orders, track shipments  
🏦 **Banking** - Check balances, transfer money, pay bills  
🏥 **Healthcare** - Schedule appointments, check test results, refill prescriptions  
🎫 **Customer Service** - Look up orders, process returns, update accounts  

### What You'll Build

In this tutorial, you'll create a complete AI agent with:

- **Weather functions** - Real-time weather data
- **Database functions** - Query and update operations
- **Email functions** - Send notifications
- **Calendar functions** - Schedule and manage events
- **Multi-step workflows** - Complex task automation
- **Error handling** - Robust failure management
- **Security** - Input validation and authorization

By the end, you'll have a production-ready AI agent that can autonomously handle complex user requests.

### Prerequisites

**Required Knowledge**:
- Java 17+ and Spring Boot basics
- REST API concepts
- Basic understanding of AI/LLMs

**What We'll Use**:
- Spring AI 1.0.0-M3
- Spring Boot 3.2+
- OpenAI GPT-4o (or compatible model)
- PostgreSQL for data storage
- Maven or Gradle

Let's dive in!

---

## Understanding Function Calling

### How Function Calling Works

The function calling flow has four steps:

```
1. User Request → "Book a flight to Paris for next Monday"

2. AI Analysis → Determines it needs to call functions:
   - searchFlights(destination: "Paris", date: "2025-11-27")
   - getUserPreferences(userId: "user123")

3. Function Execution → Spring AI executes your Java methods:
   searchFlights() → returns flight options
   getUserPreferences() → returns seat preference, airline

4. AI Response → Synthesizes results into natural language:
   "I found 3 flights to Paris on Monday. Based on your 
    preference for window seats, I recommend Air France 
    flight AF1234 departing at 10:30 AM..."
```

### The Function Calling Protocol

**Step-by-step breakdown**:

```java
// 1. User sends message
String userMessage = "What's the temperature in Tokyo?";

// 2. AI determines function needed
FunctionCall functionCall = {
    name: "getCurrentWeather",
    arguments: {
        location: "Tokyo",
        unit: "celsius"
    }
}

// 3. Spring AI executes the function
WeatherData result = getCurrentWeather("Tokyo", "celsius");
// → {temperature: 22, condition: "sunny", humidity: 60}

// 4. AI formats the response
String response = "The current temperature in Tokyo is 22°C. 
                   It's sunny with 60% humidity."
```

### Function Definitions

AI models need to understand what your functions do. Spring AI automatically generates function schemas from your Java code:

```java
@Bean
@Description("Get current weather for a location")
public Function<WeatherRequest, WeatherResponse> getCurrentWeather() {
    return request -> {
        // Implementation
    };
}

record WeatherRequest(
    @Description("City name or location") String location,
    @Description("Temperature unit: celsius or fahrenheit") String unit
) {}
```

This generates a schema like:

```json
{
  "name": "getCurrentWeather",
  "description": "Get current weather for a location",
  "parameters": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City name or location"
      },
      "unit": {
        "type": "string",
        "description": "Temperature unit: celsius or fahrenheit"
      }
    },
    "required": ["location", "unit"]
  }
}
```

### Supported Models

Not all AI models support function calling. Here's what works:

| Provider | Models | Notes |
|----------|--------|-------|
| **OpenAI** | GPT-4o, GPT-4o-mini, GPT-4-turbo | ✅ Best support |
| **OpenAI** | GPT-3.5-turbo | ✅ Good support |
| **Anthropic** | Claude 3.5 Sonnet, Claude 3 Opus | ✅ Excellent |
| **Azure OpenAI** | GPT-4, GPT-3.5 | ✅ Full support |
| **Mistral** | Mistral Large, Mixtral | ✅ Supported |
| **Ollama** | Varies by model | ⚠️ Limited |

---

## Setup and Prerequisites

### Project Setup

**1. Create Spring Boot Project**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>ai-agent-demo</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <java.version>17</java.version>
        <spring-ai.version>1.0.0-M3</spring-ai.version>
    </properties>
    
    <dependencies>
        <!-- Spring AI OpenAI -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
        </dependency>
        
        <!-- Spring Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Spring Data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- PostgreSQL -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Lombok (optional) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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

**2. Application Configuration**:

```yaml
# application.yml
spring:
  application:
    name: ai-agent-demo
  
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4o
          temperature: 0.0  # Deterministic for function calling
  
  datasource:
    url: jdbc:postgresql://localhost:5432/aiagent
    username: postgres
    password: postgres
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

# Logging
logging:
  level:
    org.springframework.ai: DEBUG
    com.example: DEBUG
```

**3. Main Application Class**:

```java
package com.example.aiagent;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AiAgentApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(AiAgentApplication.class, args);
    }
}
```

---

## Your First Function

### Simple Function Example

Let's create a basic function that calculates the sum of two numbers:

```java
package com.example.aiagent.functions;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Description;

import java.util.function.Function;

@Configuration
public class MathFunctions {
    
    /**
     * Simple addition function.
     */
    @Bean
    @Description("Add two numbers together")
    public Function<AddRequest, AddResponse> addNumbers() {
        return request -> {
            int result = request.a() + request.b();
            return new AddResponse(result);
        };
    }
    
    public record AddRequest(
        @Description("First number") int a,
        @Description("Second number") int b
    ) {}
    
    public record AddResponse(int sum) {}
}
```

### Using the Function

**Create a controller to test**:

```java
package com.example.aiagent.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/chat")
public class ChatController {
    
    private final ChatClient chatClient;
    
    public ChatController(ChatClient.Builder builder) {
        // Register the function
        this.chatClient = builder
            .defaultFunctions("addNumbers")
            .build();
    }
    
    @PostMapping
    public String chat(@RequestBody String message) {
        return chatClient.prompt()
            .user(message)
            .call()
            .content();
    }
}
```

**Test it**:

```bash
curl -X POST http://localhost:8080/api/chat \
  -H "Content-Type: text/plain" \
  -d "What is 25 + 17?"
```

**Response**:
```
The sum of 25 and 17 is 42.
```

**What happened**:
1. AI received: "What is 25 + 17?"
2. AI decided to call: `addNumbers(a: 25, b: 17)`
3. Function returned: `{sum: 42}`
4. AI formatted response: "The sum of 25 and 17 is 42."

### Adding More Math Functions

```java
@Configuration
public class MathFunctions {
    
    @Bean
    @Description("Add two numbers")
    public Function<BinaryOp, IntResult> addNumbers() {
        return op -> new IntResult(op.a() + op.b());
    }
    
    @Bean
    @Description("Subtract second number from first")
    public Function<BinaryOp, IntResult> subtractNumbers() {
        return op -> new IntResult(op.a() - op.b());
    }
    
    @Bean
    @Description("Multiply two numbers")
    public Function<BinaryOp, IntResult> multiplyNumbers() {
        return op -> new IntResult(op.a() * op.b());
    }
    
    @Bean
    @Description("Divide first number by second")
    public Function<BinaryOp, DoubleResult> divideNumbers() {
        return op -> {
            if (op.b() == 0) {
                throw new IllegalArgumentException("Cannot divide by zero");
            }
            return new DoubleResult((double) op.a() / op.b());
        };
    }
    
    record BinaryOp(
        @Description("First number") int a,
        @Description("Second number") int b
    ) {}
    
    record IntResult(int result) {}
    record DoubleResult(double result) {}
}
```

**Test with complex expression**:

```
User: "What's (15 + 25) * 2 divided by 4?"

AI: [calls addNumbers(15, 25)] → 40
    [calls multiplyNumbers(40, 2)] → 80
    [calls divideNumbers(80, 4)] → 20.0

Response: "The result is 20."
```

---

## Function Registration and Discovery

### Manual Registration

Register specific functions:

```java
@Service
public class AgentService {
    
    private final ChatClient chatClient;
    
    public AgentService(ChatClient.Builder builder) {
        this.chatClient = builder
            .defaultFunctions(
                "getCurrentWeather",
                "searchFlights",
                "sendEmail"
            )
            .build();
    }
}
```

### Dynamic Registration

Register functions at runtime:

```java
@Service
public class DynamicFunctionService {
    
    private final ChatClient.Builder chatClientBuilder;
    private final Map<String, ChatClient> clientCache = new ConcurrentHashMap<>();
    
    /**
     * Get chat client with specific functions.
     */
    public ChatClient getClientWithFunctions(List<String> functionNames) {
        String cacheKey = String.join(",", functionNames);
        
        return clientCache.computeIfAbsent(cacheKey, k ->
            chatClientBuilder
                .defaultFunctions(functionNames.toArray(new String[0]))
                .build()
        );
    }
    
    /**
     * Chat with dynamic function selection.
     */
    public String chat(String message, List<String> enabledFunctions) {
        ChatClient client = getClientWithFunctions(enabledFunctions);
        
        return client.prompt()
            .user(message)
            .call()
            .content();
    }
}
```

### Auto-Discovery

Automatically discover all function beans:

```java
@Configuration
public class FunctionDiscoveryConfig {
    
    @Bean
    public ChatClient.Builder chatClientBuilder(
            ChatClient.Builder builder,
            ApplicationContext context) {
        
        // Find all function beans
        Map<String, Function> functionBeans = context.getBeansOfType(Function.class);
        
        List<String> functionNames = functionBeans.keySet().stream()
            .filter(name -> !name.startsWith("scopedTarget."))
            .toList();
        
        log.info("Auto-discovered {} functions: {}", 
                functionNames.size(), functionNames);
        
        return builder.defaultFunctions(
            functionNames.toArray(new String[0])
        );
    }
}
```

### Conditional Function Registration

Enable functions based on configuration:

```java
@Configuration
public class ConditionalFunctionConfig {
    
    @Value("${features.weather.enabled:false}")
    private boolean weatherEnabled;
    
    @Value("${features.email.enabled:false}")
    private boolean emailEnabled;
    
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder) {
        List<String> functions = new ArrayList<>();
        
        if (weatherEnabled) {
            functions.add("getCurrentWeather");
            functions.add("getWeatherForecast");
        }
        
        if (emailEnabled) {
            functions.add("sendEmail");
            functions.add("searchEmails");
        }
        
        // Always include core functions
        functions.add("getCurrentTime");
        functions.add("calculateExpression");
        
        return builder
            .defaultFunctions(functions.toArray(new String[0]))
            .build();
    }
}
```

**Configuration**:

```yaml
features:
  weather:
    enabled: true
  email:
    enabled: true
    smtp:
      host: smtp.gmail.com
      port: 587
```

---

## Building Real-World Functions

### Weather Functions

**Complete weather service with external API**:

```java
package com.example.aiagent.functions;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Description;
import org.springframework.web.client.RestTemplate;

import java.util.function.Function;

@Configuration
public class WeatherFunctions {
    
    private final RestTemplate restTemplate;
    private final String apiKey;
    
    public WeatherFunctions(
            RestTemplate restTemplate,
            @Value("${weather.api.key}") String apiKey) {
        this.restTemplate = restTemplate;
        this.apiKey = apiKey;
    }
    
    @Bean
    @Description("Get current weather for a specific location")
    public Function<WeatherRequest, WeatherResponse> getCurrentWeather() {
        return request -> {
            String url = String.format(
                "https://api.openweathermap.org/data/2.5/weather?q=%s&units=%s&appid=%s",
                request.location(),
                request.unit(),
                apiKey
            );
            
            WeatherApiResponse apiResponse = restTemplate.getForObject(
                url, 
                WeatherApiResponse.class
            );
            
            if (apiResponse == null) {
                throw new RuntimeException("Failed to fetch weather data");
            }
            
            return new WeatherResponse(
                request.location(),
                apiResponse.main().temp(),
                apiResponse.weather().get(0).description(),
                apiResponse.main().humidity(),
                apiResponse.wind().speed()
            );
        };
    }
    
    @Bean
    @Description("Get weather forecast for the next 5 days")
    public Function<ForecastRequest, ForecastResponse> getWeatherForecast() {
        return request -> {
            String url = String.format(
                "https://api.openweathermap.org/data/2.5/forecast?q=%s&units=%s&appid=%s",
                request.location(),
                request.unit(),
                apiKey
            );
            
            ForecastApiResponse apiResponse = restTemplate.getForObject(
                url,
                ForecastApiResponse.class
            );
            
            if (apiResponse == null) {
                throw new RuntimeException("Failed to fetch forecast data");
            }
            
            // Convert API response to our format
            List<DailyForecast> forecasts = apiResponse.list().stream()
                .map(item -> new DailyForecast(
                    item.dt_txt(),
                    item.main().temp(),
                    item.weather().get(0).description()
                ))
                .toList();
            
            return new ForecastResponse(request.location(), forecasts);
        };
    }
    
    // Request/Response records
    public record WeatherRequest(
        @Description("City name (e.g., 'Paris', 'New York')") 
        String location,
        
        @Description("Temperature unit: 'metric' (Celsius) or 'imperial' (Fahrenheit)")
        String unit
    ) {}
    
    public record WeatherResponse(
        String location,
        double temperature,
        String condition,
        int humidity,
        double windSpeed
    ) {}
    
    public record ForecastRequest(
        @Description("City name") String location,
        @Description("Temperature unit") String unit
    ) {}
    
    public record ForecastResponse(
        String location,
        List<DailyForecast> forecasts
    ) {}
    
    public record DailyForecast(
        String date,
        double temperature,
        String condition
    ) {}
    
    // API response models (private, not exposed to AI)
    private record WeatherApiResponse(
        Main main,
        List<Weather> weather,
        Wind wind
    ) {}
    
    private record Main(double temp, int humidity) {}
    private record Weather(String description) {}
    private record Wind(double speed) {}
    private record ForecastApiResponse(List<ForecastItem> list) {}
    private record ForecastItem(Main main, List<Weather> weather, String dt_txt) {}
}
```

**Configuration**:

```yaml
weather:
  api:
    key: ${OPENWEATHER_API_KEY}
```

**Usage**:

```
User: "What's the weather in Tokyo?"

AI: [calls getCurrentWeather("Tokyo", "metric")]
Response: "The current weather in Tokyo is 22°C with clear skies. 
           Humidity is at 60% with wind speed of 3.5 m/s."

User: "How about the forecast for the next few days?"

AI: [calls getWeatherForecast("Tokyo", "metric")]
Response: "Here's the 5-day forecast for Tokyo:
           - Tomorrow: 23°C, partly cloudy
           - Day 2: 21°C, light rain
           - Day 3: 24°C, sunny
           ..."
```

### Database Functions

**Query and manipulate database data**:

```java
package com.example.aiagent.functions;

import com.example.aiagent.entity.User;
import com.example.aiagent.entity.Order;
import com.example.aiagent.repository.UserRepository;
import com.example.aiagent.repository.OrderRepository;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Description;

import java.util.function.Function;
import java.util.List;

@Configuration
public class DatabaseFunctions {
    
    private final UserRepository userRepository;
    private final OrderRepository orderRepository;
    
    public DatabaseFunctions(
            UserRepository userRepository,
            OrderRepository orderRepository) {
        this.userRepository = userRepository;
        this.orderRepository = orderRepository;
    }
    
    @Bean
    @Description("Search for users by email or name")
    public Function<UserSearchRequest, UserSearchResponse> searchUsers() {
        return request -> {
            List<User> users;
            
            if (request.email() != null) {
                users = userRepository.findByEmailContainingIgnoreCase(
                    request.email()
                );
            } else if (request.name() != null) {
                users = userRepository.findByNameContainingIgnoreCase(
                    request.name()
                );
            } else {
                throw new IllegalArgumentException(
                    "Either email or name must be provided"
                );
            }
            
            List<UserInfo> userInfos = users.stream()
                .map(u -> new UserInfo(
                    u.getId(),
                    u.getName(),
                    u.getEmail(),
                    u.getCreatedAt()
                ))
                .toList();
            
            return new UserSearchResponse(userInfos);
        };
    }
    
    @Bean
    @Description("Get order history for a user")
    public Function<OrderHistoryRequest, OrderHistoryResponse> getOrderHistory() {
        return request -> {
            List<Order> orders = orderRepository.findByUserIdOrderByCreatedAtDesc(
                request.userId()
            );
            
            List<OrderInfo> orderInfos = orders.stream()
                .limit(request.limit())
                .map(o -> new OrderInfo(
                    o.getId(),
                    o.getTotal(),
                    o.getStatus(),
                    o.getCreatedAt()
                ))
                .toList();
            
            return new OrderHistoryResponse(orderInfos);
        };
    }
    
    @Bean
    @Description("Get total sales for a date range")
    public Function<SalesRequest, SalesResponse> getSalesReport() {
        return request -> {
            Double totalSales = orderRepository.sumTotalByDateRange(
                request.startDate(),
                request.endDate()
            );
            
            Long orderCount = orderRepository.countByDateRange(
                request.startDate(),
                request.endDate()
            );
            
            return new SalesResponse(
                totalSales != null ? totalSales : 0.0,
                orderCount != null ? orderCount : 0L,
                request.startDate(),
                request.endDate()
            );
        };
    }
    
    // Request/Response records
    public record UserSearchRequest(
        @Description("Email to search for (partial match)") String email,
        @Description("Name to search for (partial match)") String name
    ) {}
    
    public record UserSearchResponse(List<UserInfo> users) {}
    
    public record UserInfo(
        Long id,
        String name,
        String email,
        LocalDateTime createdAt
    ) {}
    
    public record OrderHistoryRequest(
        @Description("User ID") Long userId,
        @Description("Maximum number of orders to return") int limit
    ) {}
    
    public record OrderHistoryResponse(List<OrderInfo> orders) {}
    
    public record OrderInfo(
        Long id,
        double total,
        String status,
        LocalDateTime createdAt
    ) {}
    
    public record SalesRequest(
        @Description("Start date (YYYY-MM-DD)") String startDate,
        @Description("End date (YYYY-MM-DD)") String endDate
    ) {}
    
    public record SalesResponse(
        double totalSales,
        long orderCount,
        String startDate,
        String endDate
    ) {}
}
```

**Repository interfaces**:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByEmailContainingIgnoreCase(String email);
    List<User> findByNameContainingIgnoreCase(String name);
}

public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByUserIdOrderByCreatedAtDesc(Long userId);
    
    @Query("SELECT SUM(o.total) FROM Order o WHERE o.createdAt BETWEEN :start AND :end")
    Double sumTotalByDateRange(
        @Param("start") LocalDateTime start,
        @Param("end") LocalDateTime end
    );
    
    @Query("SELECT COUNT(o) FROM Order o WHERE o.createdAt BETWEEN :start AND :end")
    Long countByDateRange(
        @Param("start") LocalDateTime start,
        @Param("end") LocalDateTime end
    );
}
```

**Usage**:

```
User: "Find the customer with email john@example.com"

AI: [calls searchUsers(email: "john@example.com")]
Response: "I found John Doe (ID: 123, email: john@example.com, 
           created: 2024-01-15)."

User: "What are his recent orders?"

AI: [calls getOrderHistory(userId: 123, limit: 5)]
Response: "John has 3 recent orders:
           1. Order #456 - $129.99 (Completed, Nov 15)
           2. Order #445 - $89.50 (Shipped, Nov 10)
           3. Order #432 - $54.25 (Delivered, Nov 5)"

User: "What were our total sales last month?"

AI: [calls getSalesReport(startDate: "2024-10-01", endDate: "2024-10-31")]
Response: "Last month (October 2024), total sales were $45,280.50 
           across 342 orders."
```

### Email Functions

**Send and manage emails**:

```java
package com.example.aiagent.functions;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Description;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;

import java.util.function.Function;

@Configuration
public class EmailFunctions {
    
    private final JavaMailSender mailSender;
    private final String fromEmail;
    
    public EmailFunctions(
            JavaMailSender mailSender,
            @Value("${spring.mail.from}") String fromEmail) {
        this.mailSender = mailSender;
        this.fromEmail = fromEmail;
    }
    
    @Bean
    @Description("Send an email to a recipient")
    public Function<SendEmailRequest, SendEmailResponse> sendEmail() {
        return request -> {
            try {
                SimpleMailMessage message = new SimpleMailMessage();
                message.setFrom(fromEmail);
                message.setTo(request.to());
                message.setSubject(request.subject());
                message.setText(request.body());
                
                mailSender.send(message);
                
                return new SendEmailResponse(
                    true,
                    "Email sent successfully to " + request.to()
                );
                
            } catch (Exception e) {
                log.error("Failed to send email", e);
                return new SendEmailResponse(
                    false,
                    "Failed to send email: " + e.getMessage()
                );
            }
        };
    }
    
    public record SendEmailRequest(
        @Description("Recipient email address") String to,
        @Description("Email subject") String subject,
        @Description("Email body content") String body
    ) {}
    
    public record SendEmailResponse(
        boolean success,
        String message
    ) {}
}
```

**Configuration**:

```yaml
spring:
  mail:
    host: smtp.gmail.com
    port: 587
    username: ${EMAIL_USERNAME}
    password: ${EMAIL_PASSWORD}
    from: noreply@example.com
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
```

**Usage**:

```
User: "Send an email to customer@example.com confirming their order #12345"

AI: [calls sendEmail(
      to: "customer@example.com",
      subject: "Order Confirmation #12345",
      body: "Thank you for your order! Your order #12345 has been confirmed..."
    )]
Response: "I've sent a confirmation email to customer@example.com 
           for order #12345."
```

---

## Advanced Function Patterns

### Composite Functions

Combine multiple functions into one:

```java
@Configuration
public class CompositeFunctions {
    
    private final Function<WeatherRequest, WeatherResponse> weatherFunc;
    private final Function<SendEmailRequest, SendEmailResponse> emailFunc;
    
    @Bean
    @Description("Get weather and send it via email")
    public Function<WeatherEmailRequest, WeatherEmailResponse> sendWeatherEmail() {
        return request -> {
            // 1. Get weather
            WeatherResponse weather = weatherFunc.apply(
                new WeatherRequest(request.location(), "metric")
            );
            
            // 2. Format email
            String emailBody = String.format("""
                Weather Update for %s:
                
                Temperature: %.1f°C
                Condition: %s
                Humidity: %d%%
                Wind Speed: %.1f m/s
                """,
                request.location(),
                weather.temperature(),
                weather.condition(),
                weather.humidity(),
                weather.windSpeed()
            );
            
            // 3. Send email
            SendEmailResponse emailResult = emailFunc.apply(
                new SendEmailRequest(
                    request.email(),
                    "Weather Update: " + request.location(),
                    emailBody
                )
            );
            
            return new WeatherEmailResponse(
                emailResult.success(),
                weather,
                emailResult.message()
            );
        };
    }
    
    public record WeatherEmailRequest(
        @Description("Location to get weather for") String location,
        @Description("Email address to send to") String email
    ) {}
    
    public record WeatherEmailResponse(
        boolean emailSent,
        WeatherResponse weather,
        String message
    ) {}
}
```

### Async Functions

Handle long-running operations:

```java
@Configuration
public class AsyncFunctions {
    
    private final ReportService reportService;
    
    @Bean
    @Description("Generate a sales report (async operation)")
    public Function<ReportRequest, ReportResponse> generateReport() {
        return request -> {
            // Start async report generation
            String jobId = UUID.randomUUID().toString();
            
            CompletableFuture.runAsync(() -> {
                reportService.generateReport(
                    jobId,
                    request.startDate(),
                    request.endDate(),
                    request.format()
                );
            });
            
            return new ReportResponse(
                jobId,
                "STARTED",
                "Report generation started. Job ID: " + jobId
            );
        };
    }
    
    @Bean
    @Description("Check status of a report generation job")
    public Function<JobStatusRequest, JobStatusResponse> checkReportStatus() {
        return request -> {
            ReportJob job = reportService.getJob(request.jobId());
            
            return new JobStatusResponse(
                job.getId(),
                job.getStatus(),
                job.getProgress(),
                job.getDownloadUrl()
            );
        };
    }
    
    public record ReportRequest(
        @Description("Start date") String startDate,
        @Description("End date") String endDate,
        @Description("Format: PDF or EXCEL") String format
    ) {}
    
    public record ReportResponse(
        String jobId,
        String status,
        String message
    ) {}
    
    public record JobStatusRequest(
        @Description("Job ID from report generation") String jobId
    ) {}
    
    public record JobStatusResponse(
        String jobId,
        String status,
        int progress,
        String downloadUrl
    ) {}
}
```

**Usage**:

```
User: "Generate a sales report for October 2024 as PDF"

AI: [calls generateReport(...)]
Response: "I've started generating your report. Job ID: abc-123. 
           I'll check the status for you..."

AI: [calls checkReportStatus(jobId: "abc-123")]
Response: "Your report is 45% complete. I'll notify you when it's ready."
```

### Paginated Functions

Handle large result sets:

```java
@Bean
@Description("Search products with pagination")
public Function<ProductSearchRequest, ProductSearchResponse> searchProducts() {
    return request -> {
        Pageable pageable = PageRequest.of(
            request.page(),
            request.pageSize()
        );
        
        Page<Product> page = productRepository.findByNameContainingIgnoreCase(
            request.query(),
            pageable
        );
        
        List<ProductInfo> products = page.getContent().stream()
            .map(p -> new ProductInfo(
                p.getId(),
                p.getName(),
                p.getPrice(),
                p.getStock()
            ))
            .toList();
        
        return new ProductSearchResponse(
            products,
            page.getNumber(),
            page.getTotalPages(),
            page.getTotalElements(),
            page.hasNext()
        );
    };
}

public record ProductSearchRequest(
    @Description("Search query") String query,
    @Description("Page number (0-based)") int page,
    @Description("Items per page") int pageSize
) {}

public record ProductSearchResponse(
    List<ProductInfo> products,
    int currentPage,
    int totalPages,
    long totalResults,
    boolean hasMore
) {}
```

---

## Building a Complete AI Agent

Let's build a complete customer service agent:

### Agent Architecture

```java
package com.example.aiagent.agent;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.stereotype.Service;

@Service
public class CustomerServiceAgent {
    
    private final ChatClient chatClient;
    
    public CustomerServiceAgent(ChatClient.Builder builder) {
        this.chatClient = builder
            .defaultSystem("""
                You are a helpful customer service agent for an e-commerce store.
                
                Your capabilities:
                - Look up customer information
                - Check order status
                - Process returns and refunds
                - Answer product questions
                - Resolve issues
                
                Always:
                - Be polite and professional
                - Verify customer identity before sharing information
                - Explain actions you take
                - Escalate complex issues to human agents
                """)
            .defaultFunctions(
                // Customer functions
                "searchCustomer",
                "getCustomerOrders",
                "updateCustomerInfo",
                
                // Order functions
                "getOrderDetails",
                "trackShipment",
                "cancelOrder",
                "initiateReturn",
                
                // Product functions
                "searchProducts",
                "checkInventory",
                "getProductDetails",
                
                // Communication functions
                "sendEmail",
                "createTicket"
            )
            .build();
    }
    
    public String chat(String customerId, String message) {
        return chatClient.prompt()
            .user(u -> u.text("""
                    Customer ID: {customerId}
                    Message: {message}
                    """)
                    .param("customerId", customerId)
                    .param("message", message))
            .call()
            .content();
    }
}
```

### Customer Functions

```java
@Configuration
public class CustomerFunctions {
    
    private final CustomerRepository customerRepository;
    private final OrderRepository orderRepository;
    
    @Bean
    @Description("Search for a customer by email, phone, or name")
    public Function<CustomerSearchRequest, CustomerSearchResponse> searchCustomer() {
        return request -> {
            List<Customer> customers = new ArrayList<>();
            
            if (request.email() != null) {
                customerRepository.findByEmail(request.email())
                    .ifPresent(customers::add);
            } else if (request.phone() != null) {
                customerRepository.findByPhone(request.phone())
                    .ifPresent(customers::add);
            } else if (request.name() != null) {
                customers.addAll(
                    customerRepository.findByNameContainingIgnoreCase(request.name())
                );
            }
            
            List<CustomerInfo> customerInfos = customers.stream()
                .map(c -> new CustomerInfo(
                    c.getId(),
                    c.getName(),
                    c.getEmail(),
                    c.getPhone(),
                    c.getTier()
                ))
                .toList();
            
            return new CustomerSearchResponse(customerInfos);
        };
    }
    
    @Bean
    @Description("Get all orders for a customer")
    public Function<CustomerOrdersRequest, CustomerOrdersResponse> getCustomerOrders() {
        return request -> {
            List<Order> orders = orderRepository
                .findByCustomerIdOrderByCreatedAtDesc(request.customerId());
            
            List<OrderSummary> orderSummaries = orders.stream()
                .map(o -> new OrderSummary(
                    o.getId(),
                    o.getStatus(),
                    o.getTotal(),
                    o.getCreatedAt(),
                    o.getTrackingNumber()
                ))
                .toList();
            
            return new CustomerOrdersResponse(orderSummaries);
        };
    }
    
    public record CustomerSearchRequest(
        @Description("Customer email") String email,
        @Description("Customer phone") String phone,
        @Description("Customer name") String name
    ) {}
    
    public record CustomerSearchResponse(List<CustomerInfo> customers) {}
    
    public record CustomerInfo(
        Long id,
        String name,
        String email,
        String phone,
        String tier
    ) {}
    
    public record CustomerOrdersRequest(
        @Description("Customer ID") Long customerId
    ) {}
    
    public record CustomerOrdersResponse(List<OrderSummary> orders) {}
    
    public record OrderSummary(
        Long id,
        String status,
        double total,
        LocalDateTime createdAt,
        String trackingNumber
    ) {}
}
```

### Order Management Functions

```java
@Configuration
public class OrderFunctions {
    
    private final OrderRepository orderRepository;
    private final OrderService orderService;
    
    @Bean
    @Description("Get detailed information about an order")
    public Function<OrderDetailsRequest, OrderDetailsResponse> getOrderDetails() {
        return request -> {
            Order order = orderRepository.findById(request.orderId())
                .orElseThrow(() -> new NotFoundException("Order not found"));
            
            List<OrderItemInfo> items = order.getItems().stream()
                .map(item -> new OrderItemInfo(
                    item.getProductName(),
                    item.getQuantity(),
                    item.getPrice(),
                    item.getSubtotal()
                ))
                .toList();
            
            return new OrderDetailsResponse(
                order.getId(),
                order.getStatus(),
                items,
                order.getSubtotal(),
                order.getTax(),
                order.getShipping(),
                order.getTotal(),
                order.getShippingAddress(),
                order.getTrackingNumber(),
                order.getEstimatedDelivery()
            );
        };
    }
    
    @Bean
    @Description("Track shipment status for an order")
    public Function<TrackShipmentRequest, TrackShipmentResponse> trackShipment() {
        return request -> {
            Order order = orderRepository.findById(request.orderId())
                .orElseThrow(() -> new NotFoundException("Order not found"));
            
            if (order.getTrackingNumber() == null) {
                return new TrackShipmentResponse(
                    "NOT_SHIPPED",
                    "Order has not been shipped yet",
                    null,
                    null
                );
            }
            
            // Call shipping carrier API
            ShipmentTracking tracking = orderService.trackShipment(
                order.getTrackingNumber()
            );
            
            return new TrackShipmentResponse(
                tracking.getStatus(),
                tracking.getCurrentLocation(),
                tracking.getEstimatedDelivery(),
                tracking.getEvents()
            );
        };
    }
    
    @Bean
    @Description("Cancel an order if it hasn't shipped yet")
    public Function<CancelOrderRequest, CancelOrderResponse> cancelOrder() {
        return request -> {
            try {
                orderService.cancelOrder(request.orderId(), request.reason());
                
                return new CancelOrderResponse(
                    true,
                    "Order cancelled successfully",
                    "CANCELLED"
                );
                
            } catch (OrderAlreadyShippedException e) {
                return new CancelOrderResponse(
                    false,
                    "Order has already shipped and cannot be cancelled. " +
                    "Please initiate a return instead.",
                    null
                );
                
            } catch (Exception e) {
                return new CancelOrderResponse(
                    false,
                    "Failed to cancel order: " + e.getMessage(),
                    null
                );
            }
        };
    }
    
    @Bean
    @Description("Initiate a return for an order")
    public Function<InitiateReturnRequest, InitiateReturnResponse> initiateReturn() {
        return request -> {
            ReturnRequest returnRequest = orderService.createReturn(
                request.orderId(),
                request.items(),
                request.reason()
            );
            
            return new InitiateReturnResponse(
                true,
                returnRequest.getId(),
                "Return initiated successfully. Return ID: " + returnRequest.getId(),
                returnRequest.getReturnLabel()
            );
        };
    }
    
    // Request/Response records
    public record OrderDetailsRequest(
        @Description("Order ID") Long orderId
    ) {}
    
    public record OrderDetailsResponse(
        Long orderId,
        String status,
        List<OrderItemInfo> items,
        double subtotal,
        double tax,
        double shipping,
        double total,
        String shippingAddress,
        String trackingNumber,
        LocalDateTime estimatedDelivery
    ) {}
    
    public record OrderItemInfo(
        String productName,
        int quantity,
        double price,
        double subtotal
    ) {}
    
    public record TrackShipmentRequest(
        @Description("Order ID") Long orderId
    ) {}
    
    public record TrackShipmentResponse(
        String status,
        String currentLocation,
        LocalDateTime estimatedDelivery,
        List<TrackingEvent> events
    ) {}
    
    public record TrackingEvent(
        LocalDateTime timestamp,
        String location,
        String description
    ) {}
    
    public record CancelOrderRequest(
        @Description("Order ID") Long orderId,
        @Description("Reason for cancellation") String reason
    ) {}
    
    public record CancelOrderResponse(
        boolean success,
        String message,
        String newStatus
    ) {}
    
    public record InitiateReturnRequest(
        @Description("Order ID") Long orderId,
        @Description("Items to return") List<Long> items,
        @Description("Reason for return") String reason
    ) {}
    
    public record InitiateReturnResponse(
        boolean success,
        String returnId,
        String message,
        String returnLabel
    ) {}
}
```

### Complete Agent Usage

```
User: "I need to check the status of my recent order"

AI: [searches for customer context]
    [calls getCustomerOrders(customerId)]
Response: "I found your most recent order #12345 placed on Nov 15. 
           The status is 'Shipped'. Would you like tracking details?"

User: "Yes, where is it now?"

AI: [calls trackShipment(orderId: 12345)]
Response: "Your package is currently in Chicago, IL and is on schedule 
           for delivery tomorrow, Nov 21 by 5:00 PM. The last update 
           was at 2:30 PM today: 'Out for delivery'."

User: "Actually, I need to return one of the items"

AI: [calls getOrderDetails(orderId: 12345)]
Response: "I see your order contains 3 items:
           1. Blue T-Shirt ($29.99)
           2. Black Jeans ($59.99)
           3. Running Shoes ($89.99)
           
           Which item would you like to return?"

User: "The jeans don't fit"

AI: [calls initiateReturn(
      orderId: 12345,
      items: [item_id_for_jeans],
      reason: "Item doesn't fit"
    )]
Response: "I've initiated a return for the Black Jeans ($59.99). 
           Your return ID is RET-789. I've emailed you a prepaid 
           return label. Once we receive the item, you'll get a 
           refund of $59.99 within 5-7 business days."
```

---

## Multi-Step Workflows

### Sequential Function Execution

Chain multiple functions together:

```java
@Service
public class WorkflowService {
    
    private final ChatClient chatClient;
    
    /**
     * Handle complex workflow: New order placement.
     */
    public String processNewOrder(String customerId, String productQuery) {
        String workflow = """
                Help the customer place an order:
                
                1. Search for products matching: {productQuery}
                2. Check inventory for selected product
                3. Get customer's shipping address
                4. Calculate total with tax and shipping
                5. Create the order
                6. Send confirmation email
                
                Customer ID: {customerId}
                Product Query: {productQuery}
                """;
        
        return chatClient.prompt()
                .user(u -> u.text(workflow)
                        .param("customerId", customerId)
                        .param("productQuery", productQuery))
                .call()
                .content();
    }
}
```

**AI automatically executes**:

```
1. searchProducts("wireless headphones") 
   → Returns 5 products

2. User selects: "Sony WH-1000XM5"

3. checkInventory(productId: 123)
   → {inStock: true, quantity: 15}

4. getCustomerAddress(customerId)
   → "123 Main St, New York, NY 10001"

5. calculateOrderTotal(productId: 123, shippingAddress: "NY")
   → {subtotal: 399.99, tax: 35.99, shipping: 0, total: 435.98}

6. createOrder(customerId, productId: 123, ...)
   → {orderId: 789, status: "confirmed"}

7. sendEmail(to: customer.email, subject: "Order Confirmation #789", ...)
   → {success: true}
```

### Conditional Workflows

```java
@Bean
@Description("Process a refund with approval workflow")
public Function<RefundRequest, RefundResponse> processRefund() {
    return request -> {
        Order order = orderRepository.findById(request.orderId())
            .orElseThrow();
        
        // Automatic approval for small amounts
        if (order.getTotal() < 50.00) {
            refundService.processRefund(order.getId(), request.reason());
            
            return new RefundResponse(
                true,
                "AUTO_APPROVED",
                "Refund processed automatically: $" + order.getTotal(),
                null
            );
        }
        
        // Requires manager approval for large amounts
        String ticketId = ticketService.createApprovalTicket(
            order.getId(),
            request.reason(),
            order.getTotal()
        );
        
        return new RefundResponse(
            false,
            "PENDING_APPROVAL",
            "Refund requires manager approval. Ticket created: " + ticketId,
            ticketId
        );
    };
}
```

### Iterative Workflows

```java
@Bean
@Description("Search for flights with preference matching")
public Function<FlightSearchRequest, FlightSearchResponse> searchFlights() {
    return request -> {
        // 1. Get user preferences
        UserPreferences prefs = userService.getPreferences(request.userId());
        
        // 2. Search flights
        List<Flight> flights = flightService.search(
            request.origin(),
            request.destination(),
            request.date()
        );
        
        // 3. Filter by preferences
        List<Flight> filtered = flights.stream()
            .filter(f -> matchesPreferences(f, prefs))
            .sorted(Comparator.comparing(Flight::getPrice))
            .limit(5)
            .toList();
        
        // 4. If no matches, relax constraints
        if (filtered.isEmpty()) {
            filtered = flights.stream()
                .sorted(Comparator.comparing(Flight::getPrice))
                .limit(5)
                .toList();
        }
        
        return new FlightSearchResponse(
            filtered,
            filtered.isEmpty() ? 
                "No flights match your preferences, showing all options" :
                "Showing flights matching your preferences"
        );
    };
}
```

---

继续完成剩余章节...

## Error Handling and Validation

### Input Validation

```java
@Configuration
public class ValidatedFunctions {
    
    @Bean
    @Description("Transfer money between accounts")
    public Function<TransferRequest, TransferResponse> transferMoney() {
        return request -> {
            // Validate inputs
            List<String> errors = validateTransferRequest(request);
            
            if (!errors.isEmpty()) {
                return new TransferResponse(
                    false,
                    null,
                    "Validation failed: " + String.join(", ", errors)
                );
            }
            
            try {
                Transaction transaction = bankingService.transfer(
                    request.fromAccount(),
                    request.toAccount(),
                    request.amount()
                );
                
                return new TransferResponse(
                    true,
                    transaction.getId(),
                    "Transfer completed successfully"
                );
                
            } catch (InsufficientFundsException e) {
                return new TransferResponse(
                    false,
                    null,
                    "Insufficient funds in source account"
                );
                
            } catch (Exception e) {
                log.error("Transfer failed", e);
                return new TransferResponse(
                    false,
                    null,
                    "Transfer failed: " + e.getMessage()
                );
            }
        };
    }
    
    private List<String> validateTransferRequest(TransferRequest request) {
        List<String> errors = new ArrayList<>();
        
        if (request.amount() <= 0) {
            errors.add("Amount must be positive");
        }
        
        if (request.amount() > 10000) {
            errors.add("Amount exceeds single transfer limit of $10,000");
        }
        
        if (request.fromAccount().equals(request.toAccount())) {
            errors.add("Source and destination accounts must be different");
        }
        
        if (!accountService.exists(request.fromAccount())) {
            errors.add("Source account not found");
        }
        
        if (!accountService.exists(request.toAccount())) {
            errors.add("Destination account not found");
        }
        
        return errors;
    }
    
    public record TransferRequest(
        @Description("Source account number") String fromAccount,
        @Description("Destination account number") String toAccount,
        @Description("Amount to transfer") double amount
    ) {}
    
    public record TransferResponse(
        boolean success,
        String transactionId,
        String message
    ) {}
}
```

### Graceful Degradation

```java
@Bean
@Description("Get product recommendations")
public Function<RecommendationRequest, RecommendationResponse> getRecommendations() {
    return request -> {
        try {
            // Try ML-based recommendations
            List<Product> recommendations = mlService.getRecommendations(
                request.userId(),
                request.limit()
            );
            
            return new RecommendationResponse(
                recommendations,
                "personalized"
            );
            
        } catch (Exception e) {
            log.warn("ML service failed, falling back to popular products", e);
            
            // Fallback to popular products
            List<Product> popular = productService.getPopularProducts(
                request.limit()
            );
            
            return new RecommendationResponse(
                popular,
                "popular"
            );
        }
    };
}
```

### Rate Limiting

```java
@Component
public class RateLimitedFunctions {
    
    private final LoadingCache<String, AtomicInteger> requestCounts;
    
    public RateLimitedFunctions() {
        this.requestCounts = CacheBuilder.newBuilder()
            .expireAfterWrite(Duration.ofMinutes(1))
            .build(CacheLoader.from(() -> new AtomicInteger(0)));
    }
    
    @Bean
    @Description("Send SMS message (rate limited)")
    public Function<SmsRequest, SmsResponse> sendSms() {
        return request -> {
            // Check rate limit (5 SMS per minute per user)
            String key = "sms:" + request.userId();
            AtomicInteger count = requestCounts.getUnchecked(key);
            
            if (count.incrementAndGet() > 5) {
                return new SmsResponse(
                    false,
                    "Rate limit exceeded. Maximum 5 SMS per minute."
                );
            }
            
            try {
                smsService.send(request.to(), request.message());
                
                return new SmsResponse(
                    true,
                    "SMS sent successfully"
                );
                
            } catch (Exception e) {
                count.decrementAndGet(); // Don't count failed attempts
                return new SmsResponse(
                    false,
                    "Failed to send SMS: " + e.getMessage()
                );
            }
        };
    }
}
```

---

## Testing Function Calling

### Unit Testing Functions

```java
@ExtendWith(MockitoExtension.class)
class WeatherFunctionsTest {
    
    @Mock
    private RestTemplate restTemplate;
    
    @InjectMocks
    private WeatherFunctions weatherFunctions;
    
    @Test
    void getCurrentWeather_success() {
        // Arrange
        WeatherApiResponse mockResponse = new WeatherApiResponse(
            new Main(22.5, 60),
            List.of(new Weather("clear sky")),
            new Wind(3.5)
        );
        
        when(restTemplate.getForObject(anyString(), eq(WeatherApiResponse.class)))
            .thenReturn(mockResponse);
        
        Function<WeatherRequest, WeatherResponse> function = 
            weatherFunctions.getCurrentWeather();
        
        // Act
        WeatherResponse response = function.apply(
            new WeatherRequest("Tokyo", "metric")
        );
        
        // Assert
        assertThat(response.location()).isEqualTo("Tokyo");
        assertThat(response.temperature()).isEqualTo(22.5);
        assertThat(response.condition()).isEqualTo("clear sky");
        assertThat(response.humidity()).isEqualTo(60);
        assertThat(response.windSpeed()).isEqualTo(3.5);
    }
    
    @Test
    void getCurrentWeather_apiFailure() {
        // Arrange
        when(restTemplate.getForObject(anyString(), eq(WeatherApiResponse.class)))
            .thenThrow(new RestClientException("API unavailable"));
        
        Function<WeatherRequest, WeatherResponse> function = 
            weatherFunctions.getCurrentWeather();
        
        // Act & Assert
        assertThatThrownBy(() -> 
            function.apply(new WeatherRequest("Tokyo", "metric"))
        ).isInstanceOf(RuntimeException.class)
         .hasMessageContaining("Failed to fetch weather data");
    }
}
```

### Integration Testing

```java
@SpringBootTest
@AutoConfigureMockMvc
class FunctionCallingIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private WeatherService weatherService;
    
    @Test
    void chatWithWeatherFunction() throws Exception {
        // Arrange
        when(weatherService.getCurrentWeather("Paris", "metric"))
            .thenReturn(new WeatherData(18.0, "cloudy", 70, 2.5));
        
        // Act & Assert
        mockMvc.perform(post("/api/chat")
                .contentType(MediaType.TEXT_PLAIN)
                .content("What's the weather in Paris?"))
            .andExpect(status().isOk())
            .andExpect(content().string(
                containsString("18")
            ))
            .andExpect(content().string(
                containsString("Paris")
            ))
            .andExpect(content().string(
                containsString("cloudy")
            ));
        
        verify(weatherService).getCurrentWeather("Paris", "metric");
    }
}
```

### End-to-End Testing

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class AgentE2ETest {
    
    @LocalServerPort
    private int port;
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void completeOrderWorkflow() {
        // 1. Search products
        String response1 = chat("I'm looking for wireless headphones");
        assertThat(response1).contains("headphones");
        
        // 2. Check specific product
        String response2 = chat("Tell me about the Sony WH-1000XM5");
        assertThat(response2).contains("Sony");
        assertThat(response2).contains("price");
        
        // 3. Check availability
        String response3 = chat("Is it in stock?");
        assertThat(response3).containsAnyOf("in stock", "available");
        
        // 4. Place order
        String response4 = chat("I'd like to order one");
        assertThat(response4).containsAnyOf("order", "confirmation");
    }
    
    private String chat(String message) {
        return restTemplate.postForObject(
            "http://localhost:" + port + "/api/chat",
            message,
            String.class
        );
    }
}
```

### Mock Function for Testing

```java
@TestConfiguration
public class TestFunctionConfig {
    
    @Bean
    @Primary
    public Function<WeatherRequest, WeatherResponse> mockGetCurrentWeather() {
        return request -> new WeatherResponse(
            request.location(),
            20.0,  // Fixed temperature for testing
            "sunny",
            50,
            2.0
        );
    }
}
```

---

## Security and Best Practices

### Input Sanitization

```java
@Component
public class SecureFunctions {
    
    @Bean
    @Description("Execute database query (admin only)")
    public Function<QueryRequest, QueryResponse> executeQuery() {
        return request -> {
            // 1. Validate user authorization
            if (!securityService.hasRole(request.userId(), "ADMIN")) {
                throw new UnauthorizedException(
                    "Only administrators can execute queries"
                );
            }
            
            // 2. Sanitize SQL
            String sanitizedSql = sanitizeSql(request.sql());
            
            // 3. Validate query type
            if (!isSafeQuery(sanitizedSql)) {
                throw new SecurityException(
                    "Only SELECT queries are allowed"
                );
            }
            
            // 4. Execute with timeout
            try {
                List<Map<String, Object>> results = jdbcTemplate.query(
                    sanitizedSql,
                    new ColumnMapRowMapper()
                );
                
                return new QueryResponse(true, results, null);
                
            } catch (Exception e) {
                log.error("Query execution failed", e);
                return new QueryResponse(
                    false,
                    null,
                    "Query failed: " + e.getMessage()
                );
            }
        };
    }
    
    private String sanitizeSql(String sql) {
        // Remove comments
        sql = sql.replaceAll("--.*", "");
        sql = sql.replaceAll("/\\*.*?\\*/", "");
        
        // Remove excess whitespace
        sql = sql.trim().replaceAll("\\s+", " ");
        
        return sql;
    }
    
    private boolean isSafeQuery(String sql) {
        String normalized = sql.toLowerCase();
        
        // Must start with SELECT
        if (!normalized.startsWith("select")) {
            return false;
        }
        
        // Block dangerous keywords
        String[] forbidden = {
            "drop", "delete", "update", "insert",
            "alter", "create", "truncate", "exec",
            "execute", "xp_", "sp_"
        };
        
        for (String keyword : forbidden) {
            if (normalized.contains(keyword)) {
                return false;
            }
        }
        
        return true;
    }
}
```

### Authorization Checks

```java
@Aspect
@Component
public class FunctionSecurityAspect {
    
    @Around("@annotation(requiresAuth)")
    public Object checkAuthorization(
            ProceedingJoinPoint joinPoint,
            RequiresAuth requiresAuth) throws Throwable {
        
        // Extract user context from function arguments
        Object[] args = joinPoint.getArgs();
        String userId = extractUserId(args[0]);
        
        // Check authorization
        if (!securityService.isAuthorized(userId, requiresAuth.permission())) {
            throw new UnauthorizedException(
                "User not authorized for this operation"
            );
        }
        
        // Proceed with function execution
        return joinPoint.proceed();
    }
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RequiresAuth {
    String permission();
}

// Usage
@Bean
@RequiresAuth(permission = "CANCEL_ORDER")
public Function<CancelOrderRequest, CancelOrderResponse> cancelOrder() {
    // Implementation
}
```

### Audit Logging

```java
@Component
public class FunctionAuditAdvisor implements RequestResponseAdvisor {
    
    private final AuditRepository auditRepository;
    
    @Override
    public ChatResponse adviseResponse(
            AdvisedRequest request,
            ChatResponse response,
            Map<String, Object> context) {
        
        // Log function calls
        List<FunctionCall> functionCalls = extractFunctionCalls(response);
        
        for (FunctionCall call : functionCalls) {
            AuditLog log = new AuditLog();
            log.setFunctionName(call.getName());
            log.setArguments(call.getArguments());
            log.setUserId((String) context.get("user_id"));
            log.setTimestamp(Instant.now());
            log.setResult(call.getResult());
            
            auditRepository.save(log);
        }
        
        return response;
    }
}
```

### Best Practices Checklist

**Function Design**:
- ✅ Clear, descriptive function names
- ✅ Comprehensive `@Description` annotations
- ✅ Well-defined input/output types
- ✅ Validation for all inputs
- ✅ Meaningful error messages

**Security**:
- ✅ Authorization checks for sensitive operations
- ✅ Input sanitization (SQL injection, XSS, etc.)
- ✅ Rate limiting for expensive operations
- ✅ Audit logging for compliance
- ✅ Principle of least privilege

**Reliability**:
- ✅ Timeouts for external calls
- ✅ Retry logic with exponential backoff
- ✅ Circuit breakers for failing services
- ✅ Graceful degradation
- ✅ Comprehensive error handling

**Performance**:
- ✅ Caching for expensive operations
- ✅ Async execution for long-running tasks
- ✅ Pagination for large result sets
- ✅ Connection pooling
- ✅ Resource cleanup

**Testing**:
- ✅ Unit tests for each function
- ✅ Integration tests for workflows
- ✅ Mock external dependencies
- ✅ Test error scenarios
- ✅ Load testing for production readiness

---

## Production Deployment

### Configuration Management

```yaml
# application-prod.yml
spring:
  ai:
    openai:
      chat:
        options:
          model: gpt-4o-mini  # Cost-effective for production
          temperature: 0.0    # Deterministic
          max-tokens: 1000    # Limit costs
  
functions:
  weather:
    enabled: true
    cache-ttl: 300  # 5 minutes
  
  email:
    enabled: true
    rate-limit: 10  # per minute
    
  database:
    enabled: true
    read-only: true  # Safety first
    timeout: 30s
  
  external-apis:
    timeout: 10s
    retry:
      max-attempts: 3
      backoff: 2s
```

### Monitoring

```java
@Component
public class FunctionMetricsAdvisor implements RequestResponseAdvisor {
    
    private final MeterRegistry metrics;
    
    @Override
    public ChatResponse adviseResponse(
            AdvisedRequest request,
            ChatResponse response,
            Map<String, Object> context) {
        
        List<FunctionCall> calls = extractFunctionCalls(response);
        
        for (FunctionCall call : calls) {
            // Count function invocations
            metrics.counter("function.calls",
                    "name", call.getName(),
                    "status", call.getStatus()
            ).increment();
            
            // Record execution time
            if (context.containsKey("function_duration_" + call.getName())) {
                Duration duration = (Duration) context.get(
                    "function_duration_" + call.getName()
                );
                
                metrics.timer("function.duration",
                        "name", call.getName()
                ).record(duration);
            }
            
            // Track errors
            if (call.hasError()) {
                metrics.counter("function.errors",
                        "name", call.getName(),
                        "error", call.getError()
                ).increment();
            }
        }
        
        return response;
    }
}
```

### Caching Strategy

```java
@Configuration
@EnableCaching
public class FunctionCacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        return new CaffeineCacheManager("weather", "products", "users");
    }
    
    @Bean
    public Caffeine<Object, Object> caffeineConfig() {
        return Caffeine.newBuilder()
            .expireAfterWrite(Duration.ofMinutes(5))
            .maximumSize(1000)
            .recordStats();
    }
}

// Cached function
@Bean
@Cacheable(value = "weather", key = "#request.location + '-' + #request.unit")
public Function<WeatherRequest, WeatherResponse> getCurrentWeather() {
    return request -> {
        // Expensive API call
    };
}
```

### Health Checks

```java
@Component
public class FunctionHealthIndicator implements HealthIndicator {
    
    private final List<Function<?, ?>> functions;
    
    @Override
    public Health health() {
        Map<String, Object> details = new HashMap<>();
        
        for (Function<?, ?> function : functions) {
            String name = function.getClass().getSimpleName();
            
            try {
                // Test function with sample input
                testFunction(function);
                details.put(name, "UP");
                
            } catch (Exception e) {
                details.put(name, "DOWN: " + e.getMessage());
                return Health.down()
                    .withDetails(details)
                    .build();
            }
        }
        
        return Health.up()
            .withDetails(details)
            .build();
    }
}
```

---

## Conclusion

Congratulations! You've built a complete AI agent with function calling capabilities. Let's recap what you've learned:

### Key Takeaways

**Function Calling Fundamentals**:
- ✅ AI automatically decides when to call functions
- ✅ Functions extend AI with real-world capabilities
- ✅ Spring AI handles the complex orchestration
- ✅ Type-safe with Java records and generics

**Building Functions**:
- ✅ Simple `Function<Request, Response>` beans
- ✅ `@Description` annotations guide the AI
- ✅ Automatic schema generation
- ✅ Support for complex workflows

**Real-World Integration**:
- ✅ External APIs (weather, email, etc.)
- ✅ Database operations (queries, updates)
- ✅ Multi-step workflows
- ✅ Error handling and validation

**Production Ready**:
- ✅ Security and authorization
- ✅ Rate limiting and caching
- ✅ Monitoring and metrics
- ✅ Comprehensive testing
- ✅ Audit logging

### Your AI Agent Can Now:

🤖 **Understand** natural language requests  
🔍 **Search** databases and external APIs  
📧 **Send** emails and notifications  
💳 **Process** transactions and orders  
📊 **Generate** reports and analytics  
🔄 **Execute** multi-step workflows  
⚡ **Handle** errors gracefully  
🔒 **Secure** sensitive operations  

### Next Steps

**Expand Your Agent**:
1. Add more domain-specific functions
2. Implement complex workflows
3. Integrate with your existing systems
4. Add voice/chat interfaces
5. Deploy to production

**Advanced Topics**:
- Multi-agent systems
- Function chaining optimization
- Custom function discovery
- Advanced caching strategies
- A/B testing different functions

### Resources

- [Spring AI Function Calling Docs](https://docs.spring.io/spring-ai/reference/api/functions.html)
- [OpenAI Function Calling Guide](https://platform.openai.com/docs/guides/function-calling)
- [Example Projects on GitHub](https://github.com/spring-projects/spring-ai)

---

**You're now ready to build production AI agents with Spring AI!** 🚀

Start small, test thoroughly, and gradually expand your agent's capabilities. The possibilities are endless!

---

*Last updated: November 20, 2025*  
*Spring AI version: 1.0.0-M3*  
*Author: Spring AI Agent Team*