
+++
title = "Spring AI 快速入门：3 分钟搭建第一个 AI 应用"  # 文章标题
date = 2025-11-24T10:00:00+08:00  # 发布时间
draft = false  # 草稿模式（本地调试用，发布时改为 false）
categories = ["Spring AI"]  # 分类
tags = ["Spring AI", "入门", "OpenAI"]  # 标签（多个用逗号分隔）
author = "你的名字"
description = "本文带你快速上手 Spring AI，集成 OpenAI API 实现文本生成功能"  # 文章摘要
featuredImage = "spring-ai-logo.png"  # 封面图（放在 static/images/ 目录）
summary = "Spring AI 是一个 Spring Boot 集成 OpenAI API 的工具库，用于实现文本生成功能"
+++

## 1. 什么是 Spring AI？
Spring AI 是 Spring 生态下的 AI 开发框架，简化了 AI 模型（如 OpenAI、Anthropic）的集成，让 Java 开发者无需关注底层 API 调用细节，专注业务逻辑。

## 2. 环境准备
- JDK 17+
- Maven/Gradle
- Spring Boot 3.2+
- OpenAI API Key（需在 OpenAI 官网申请）


```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring AI OpenAI 依赖 -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
        <version>1.0.0-M1</version>
    </dependency>
</dependencies>
```

```java
import org.springframework.ai.openai.OpenAiChatClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AIController {

    private final OpenAiChatClient chatClient;

    // 构造函数注入 OpenAiChatClient（Spring AI 自动配置）
    public AIController(OpenAiChatClient chatClient) {
        this.chatClient = chatClient;
    }

    @GetMapping("/generate")
    public String generateText(@RequestParam String prompt) {
        // 调用 OpenAI API 生成文本
        return chatClient.call(prompt);
    }
}
```