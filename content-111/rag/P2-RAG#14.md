
基于下面的信息，给出英文技术博客文章（面向欧美用户，基于 Google Adsense赚钱）：
文章为主，代码为辅。
要有图表和表格。

Reference Title: Document Processing Pipeline: PDF, Word, Excel to Embeddings
Reference Keywords: spring ai document loader
Target Word Count: 5000-6000


# Document Processing Pipeline: PDF, Word, Excel to Embeddings

**Complete Guide to Building Production-Ready Document ETL with Spring AI**

---

## Table of Contents

- [Document Processing Pipeline: PDF, Word, Excel to Embeddings](#document-processing-pipeline-pdf-word-excel-to-embeddings)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [What You'll Build](#what-youll-build)
    - [The Document Processing Challenge](#the-document-processing-challenge)
    - [Processing Pipeline Overview](#processing-pipeline-overview)
    - [Prerequisites](#prerequisites)
  - [Architecture Overview](#architecture-overview)
    - [System Architecture](#system-architecture)
    - [Data Flow](#data-flow)
  - [Document Loaders in Spring AI](#document-loaders-in-spring-ai)
    - [Core Dependencies](#core-dependencies)
    - [Base Document Loader Interface](#base-document-loader-interface)
    - [Document Processing Configuration](#document-processing-configuration)
  - [PDF Processing](#pdf-processing)
    - [Advanced PDF Loader](#advanced-pdf-loader)
    - [PDF with OCR Support](#pdf-with-ocr-support)
  - [Word Document Processing](#word-document-processing)
    - [Word Document Loader](#word-document-loader)
  - [Excel Processing](#excel-processing)
    - [Excel Loader](#excel-loader)
  - [Text Extraction and Cleaning](#text-extraction-and-cleaning)
    - [Text Cleaning Service](#text-cleaning-service)
  - [Advanced Chunking Strategies](#advanced-chunking-strategies)
    - [Intelligent Chunking Service](#intelligent-chunking-service)
  - [Metadata Extraction](#metadata-extraction)
    - [Metadata Extraction Service](#metadata-extraction-service)
  - [Embedding Generation](#embedding-generation)
    - [Embedding Service](#embedding-service)
  - [Batch Processing Pipeline](#batch-processing-pipeline)
    - [Document Processing Pipeline](#document-processing-pipeline)
    - [Document Loader Factory](#document-loader-factory)
  - [Error Handling and Retry Logic](#error-handling-and-retry-logic)
    - [Robust Error Handling](#robust-error-handling)
  - [Performance Optimization](#performance-optimization)
    - [Performance Monitoring](#performance-monitoring)
  - [Production Deployment](#production-deployment)
    - [REST Controller](#rest-controller)
  - [Conclusion](#conclusion)
    - [Key Takeaways](#key-takeaways)
    - [Performance Metrics](#performance-metrics)
    - [Next Steps](#next-steps)
    - [Resources](#resources)

---

## Introduction

Document processing is the foundation of any RAG (Retrieval Augmented Generation) system. Converting unstructured documents into searchable vector embeddings requires a robust, scalable pipeline that handles multiple formats, preserves context, and maintains quality.

### What You'll Build

This comprehensive guide walks through building an enterprise-grade document processing pipeline:

- **Multi-Format Support**: PDF, Word, Excel, PowerPoint, images
- **Intelligent Chunking**: Context-aware text splitting
- **Rich Metadata**: Automatic extraction and preservation
- **High Performance**: Parallel processing, caching, optimization
- **Production Ready**: Error handling, monitoring, scalability

### The Document Processing Challenge

```
Input: Diverse document formats
├─ PDFs: Reports, papers, manuals
├─ Word: Contracts, proposals, documentation
├─ Excel: Data tables, financial reports
├─ PowerPoint: Presentations, slide decks
└─ Images: Scanned documents, diagrams

Challenges:
❌ Complex layouts and formatting
❌ Embedded images and tables
❌ Multi-column text
❌ Headers, footers, page numbers
❌ Different encodings and languages
❌ Large file sizes
❌ Scanned/image-based content

Output: Clean, structured, searchable vectors
✓ Semantic chunks with context
✓ Preserved metadata
✓ High-quality embeddings
✓ Efficient storage
```

### Processing Pipeline Overview

```
Document Upload
     ↓
Format Detection
     ↓
Text Extraction
     ↓
Content Cleaning
     ↓
Metadata Extraction
     ↓
Intelligent Chunking
     ↓
Embedding Generation
     ↓
Vector Storage
     ↓
Indexing
```

### Prerequisites

**Technical Requirements**:

```yaml
Core Dependencies:
  Java: 17+
  Spring Boot: 3.2+
  Spring AI: 1.0.0-M3+

Document Processing:
  Apache PDFBox: 3.0+
  Apache POI: 5.2+
  Apache Tika: 2.9+
  
OCR (Optional):
  Tesseract: 5.0+
  
Build Tools:
  Maven: 3.9+ or Gradle 8+
```

---

## Architecture Overview

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Document Upload Layer                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  REST API    │  │  File Upload │  │  S3 Trigger  │      │
│  │  Endpoints   │  │  Controller  │  │  Handler     │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
└─────────┼──────────────────┼──────────────────┼──────────────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│                 Document Processing Service                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Format Detection & Routing                  │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────┐  │   │
│  │  │   PDF   │ │  Word   │ │  Excel  │ │  Other   │  │   │
│  │  │ Detector│ │Detector │ │Detector │ │ Formats  │  │   │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬─────┘  │   │
│  └───────┼───────────┼───────────┼───────────┼─────────┘   │
│          │           │           │           │              │
│  ┌───────▼───────────▼───────────▼───────────▼─────────┐   │
│  │              Document Loaders                        │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │   │
│  │  │  PDF     │ │  Word    │ │  Excel   │            │   │
│  │  │  Loader  │ │  Loader  │ │  Loader  │            │   │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘            │   │
│  └───────┼────────────┼────────────┼──────────────────┘   │
│          │            │            │                       │
│  ┌───────▼────────────▼────────────▼──────────────────┐   │
│  │         Text Extraction & Processing               │   │
│  │  ┌──────────────┐  ┌──────────────┐               │   │
│  │  │  Content     │  │  Metadata    │               │   │
│  │  │  Cleaner     │  │  Extractor   │               │   │
│  │  └──────┬───────┘  └──────┬───────┘               │   │
│  └─────────┼──────────────────┼───────────────────────┘   │
│            │                  │                            │
│  ┌─────────▼──────────────────▼───────────────────────┐   │
│  │           Chunking Service                          │   │
│  │  ┌──────────────┐  ┌──────────────┐               │   │
│  │  │  Recursive   │  │  Semantic    │               │   │
│  │  │  Splitter    │  │  Chunker     │               │   │
│  │  └──────┬───────┘  └──────┬───────┘               │   │
│  └─────────┼──────────────────┼───────────────────────┘   │
│            │                  │                            │
└────────────┼──────────────────┼────────────────────────────┘
             │                  │
             ↓                  ↓
┌─────────────────────────────────────────────────────────────┐
│                  Embedding Generation                        │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Embedding Client                         │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │
│  │  │ OpenAI   │  │ Cohere   │  │ Local    │           │   │
│  │  │ Embedder │  │ Embedder │  │ Model    │           │   │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘           │   │
│  └───────┼─────────────┼─────────────┼──────────────────┘   │
└──────────┼─────────────┼─────────────┼──────────────────────┘
           │             │             │
           └─────────────┼─────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Vector Store Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Pinecone    │  │  Milvus      │  │  Weaviate    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

**Complete Processing Flow**:

```
1. Document Upload
   ├─> File received (multipart/form-data)
   ├─> Validate size and type
   ├─> Store in temp location
   └─> Generate processing ID

2. Format Detection
   ├─> Check file extension
   ├─> Verify MIME type
   ├─> Detect encoding
   └─> Route to appropriate loader

3. Content Extraction
   ├─> Parse document structure
   ├─> Extract text content
   ├─> Extract images/tables
   ├─> Extract metadata
   └─> Handle errors gracefully

4. Text Preprocessing
   ├─> Remove noise (headers, footers, page numbers)
   ├─> Normalize whitespace
   ├─> Fix encoding issues
   ├─> Clean special characters
   └─> Preserve semantic structure

5. Metadata Enrichment
   ├─> Extract document properties
   ├─> Add custom metadata
   ├─> Generate checksums
   └─> Timestamp processing

6. Intelligent Chunking
   ├─> Determine optimal chunk size
   ├─> Split preserving context
   ├─> Add overlap between chunks
   ├─> Maintain paragraph boundaries
   └─> Track chunk relationships

7. Embedding Generation
   ├─> Batch chunks for efficiency
   ├─> Generate embeddings
   ├─> Cache results
   └─> Handle rate limits

8. Vector Storage
   ├─> Prepare vector data
   ├─> Batch insert to vector DB
   ├─> Update indexes
   └─> Confirm success

Total Time (typical):
├─> Small doc (1-10 pages): 5-15 seconds
├─> Medium doc (10-100 pages): 30-90 seconds
└─> Large doc (100+ pages): 2-10 minutes
```

---

## Document Loaders in Spring AI

### Core Dependencies

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
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>document-processing-pipeline</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <java.version>17</java.version>
        <spring-ai.version>1.0.0-M3</spring-ai.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Spring AI -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-pdf-document-reader</artifactId>
            <version>${spring-ai.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-tika-document-reader</artifactId>
            <version>${spring-ai.version}</version>
        </dependency>
        
        <!-- Apache PDFBox -->
        <dependency>
            <groupId>org.apache.pdfbox</groupId>
            <artifactId>pdfbox</artifactId>
            <version>3.0.0</version>
        </dependency>
        
        <!-- Apache POI (Word, Excel, PowerPoint) -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>5.2.5</version>
        </dependency>
        
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>5.2.5</version>
        </dependency>
        
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-scratchpad</artifactId>
            <version>5.2.5</version>
        </dependency>
        
        <!-- Apache Tika (Universal document parser) -->
        <dependency>
            <groupId>org.apache.tika</groupId>
            <artifactId>tika-core</artifactId>
            <version>2.9.1</version>
        </dependency>
        
        <dependency>
            <groupId>org.apache.tika</groupId>
            <artifactId>tika-parsers-standard-package</artifactId>
            <version>2.9.1</version>
        </dependency>
        
        <!-- OCR Support (Optional) -->
        <dependency>
            <groupId>net.sourceforge.tess4j</groupId>
            <artifactId>tess4j</artifactId>
            <version>5.9.0</version>
        </dependency>
        
        <!-- Metadata extraction -->
        <dependency>
            <groupId>com.drewnoakes</groupId>
            <artifactId>metadata-extractor</artifactId>
            <version>2.18.0</version>
        </dependency>
        
        <!-- Async processing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        
        <!-- Caching -->
        <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
        </dependency>
        
        <!-- Utilities -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.15.1</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>
</project>
```

### Base Document Loader Interface

**DocumentLoaderService.java**:

```java
package com.example.pipeline.service;

import org.springframework.ai.document.Document;
import org.springframework.core.io.Resource;

import java.io.IOException;
import java.util.List;

public interface DocumentLoaderService {
    
    /**
     * Load and parse document
     */
    List<Document> load(Resource resource) throws IOException;
    
    /**
     * Check if loader supports format
     */
    boolean supports(String mimeType);
    
    /**
     * Get supported MIME types
     */
    List<String> getSupportedMimeTypes();
    
    /**
     * Extract metadata only
     */
    DocumentMetadata extractMetadata(Resource resource) throws IOException;
}
```

### Document Processing Configuration

**DocumentProcessingConfig.java**:

```java
package com.example.pipeline.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@Configuration
@EnableAsync
public class DocumentProcessingConfig {
    
    @Value("${document.processing.thread-pool.core-size:5}")
    private int corePoolSize;
    
    @Value("${document.processing.thread-pool.max-size:20}")
    private int maxPoolSize;
    
    @Value("${document.processing.thread-pool.queue-capacity:100}")
    private int queueCapacity;
    
    @Value("${document.processing.chunk-size:1000}")
    private int defaultChunkSize;
    
    @Value("${document.processing.chunk-overlap:200}")
    private int defaultChunkOverlap;
    
    @Bean(name = "documentProcessingExecutor")
    public Executor documentProcessingExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(queueCapacity);
        executor.setThreadNamePrefix("doc-processor-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        executor.initialize();
        return executor;
    }
    
    @Bean
    public DocumentProcessingProperties processingProperties() {
        return DocumentProcessingProperties.builder()
            .defaultChunkSize(defaultChunkSize)
            .defaultChunkOverlap(defaultChunkOverlap)
            .maxFileSize(100 * 1024 * 1024) // 100MB
            .allowedMimeTypes(getAllowedMimeTypes())
            .build();
    }
    
    private List<String> getAllowedMimeTypes() {
        return List.of(
            "application/pdf",
            "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
            "application/msword",
            "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
            "application/vnd.ms-excel",
            "application/vnd.openxmlformats-officedocument.presentationml.presentation",
            "application/vnd.ms-powerpoint",
            "text/plain",
            "text/csv",
            "text/html",
            "image/jpeg",
            "image/png",
            "image/tiff"
        );
    }
    
    @lombok.Data
    @lombok.Builder
    public static class DocumentProcessingProperties {
        private Integer defaultChunkSize;
        private Integer defaultChunkOverlap;
        private Long maxFileSize;
        private List<String> allowedMimeTypes;
    }
}
```

---

## PDF Processing

### Advanced PDF Loader

**PdfDocumentLoader.java**:

```java
package com.example.pipeline.service.loader;

import com.example.pipeline.service.DocumentLoaderService;
import com.example.pipeline.service.DocumentMetadata;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDDocumentInformation;
import org.apache.pdfbox.text.PDFTextStripper;
import org.apache.pdfbox.text.PDFTextStripperByArea;
import org.springframework.ai.document.Document;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.InputStream;
import java.util.*;

@Slf4j
@Service
@RequiredArgsConstructor
public class PdfDocumentLoader implements DocumentLoaderService {
    
    private static final List<String> SUPPORTED_TYPES = List.of(
        "application/pdf"
    );
    
    @Override
    public List<Document> load(Resource resource) throws IOException {
        log.info("Loading PDF: {}", resource.getFilename());
        
        List<Document> documents = new ArrayList<>();
        
        try (InputStream inputStream = resource.getInputStream();
             PDDocument pdDocument = PDDocument.load(inputStream)) {
            
            // Extract metadata
            DocumentMetadata metadata = extractMetadata(pdDocument, resource);
            
            // Configure text stripper
            PDFTextStripper stripper = new PDFTextStripper();
            stripper.setSortByPosition(true);
            stripper.setLineSeparator("\n");
            stripper.setParagraphStart("\n\n");
            
            int totalPages = pdDocument.getNumberOfPages();
            
            // Process by page or in chunks
            if (totalPages <= 10) {
                // Small PDF: Extract all at once
                String fullText = stripper.getText(pdDocument);
                documents.add(createDocument(fullText, metadata, 0, totalPages));
                
            } else {
                // Large PDF: Extract page by page
                for (int pageNum = 1; pageNum <= totalPages; pageNum++) {
                    stripper.setStartPage(pageNum);
                    stripper.setEndPage(pageNum);
                    
                    String pageText = stripper.getText(pdDocument);
                    
                    if (pageText != null && !pageText.trim().isEmpty()) {
                        Map<String, Object> pageMetadata = new HashMap<>(metadata.toMap());
                        pageMetadata.put("page_number", pageNum);
                        pageMetadata.put("total_pages", totalPages);
                        
                        documents.add(new Document(
                            cleanText(pageText),
                            pageMetadata
                        ));
                    }
                }
            }
            
            log.info("Extracted {} documents from PDF with {} pages",
                documents.size(), totalPages);
            
        } catch (Exception e) {
            log.error("Failed to load PDF: {}", resource.getFilename(), e);
            throw new IOException("PDF processing failed", e);
        }
        
        return documents;
    }
    
    @Override
    public DocumentMetadata extractMetadata(Resource resource) throws IOException {
        try (InputStream inputStream = resource.getInputStream();
             PDDocument pdDocument = PDDocument.load(inputStream)) {
            
            return extractMetadata(pdDocument, resource);
        }
    }
    
    private DocumentMetadata extractMetadata(
            PDDocument pdDocument,
            Resource resource) {
        
        PDDocumentInformation info = pdDocument.getDocumentInformation();
        
        return DocumentMetadata.builder()
            .filename(resource.getFilename())
            .mimeType("application/pdf")
            .title(info.getTitle())
            .author(info.getAuthor())
            .subject(info.getSubject())
            .keywords(info.getKeywords())
            .creator(info.getCreator())
            .producer(info.getProducer())
            .creationDate(info.getCreationDate() != null ?
                info.getCreationDate().getTime() : null)
            .modificationDate(info.getModificationDate() != null ?
                info.getModificationDate().getTime() : null)
            .pageCount(pdDocument.getNumberOfPages())
            .encrypted(pdDocument.isEncrypted())
            .build();
    }
    
    private Document createDocument(
            String text,
            DocumentMetadata metadata,
            int startPage,
            int endPage) {
        
        Map<String, Object> docMetadata = new HashMap<>(metadata.toMap());
        docMetadata.put("page_range", String.format("%d-%d", startPage, endPage));
        
        return new Document(cleanText(text), docMetadata);
    }
    
    private String cleanText(String text) {
        if (text == null) return "";
        
        return text
            // Remove excessive whitespace
            .replaceAll("[ \\t]+", " ")
            // Normalize line breaks
            .replaceAll("\\r\\n", "\n")
            .replaceAll("\\r", "\n")
            // Remove multiple consecutive newlines
            .replaceAll("\\n{3,}", "\n\n")
            // Trim each line
            .lines()
            .map(String::trim)
            .filter(line -> !line.isEmpty())
            .collect(java.util.stream.Collectors.joining("\n"));
    }
    
    @Override
    public boolean supports(String mimeType) {
        return SUPPORTED_TYPES.contains(mimeType);
    }
    
    @Override
    public List<String> getSupportedMimeTypes() {
        return SUPPORTED_TYPES;
    }
    
    /**
     * Extract text from specific page region
     */
    public String extractRegion(
            PDDocument document,
            int pageNum,
            int x,
            int y,
            int width,
            int height) throws IOException {
        
        PDFTextStripperByArea stripper = new PDFTextStripperByArea();
        stripper.setSortByPosition(true);
        
        java.awt.Rectangle rect = new java.awt.Rectangle(x, y, width, height);
        stripper.addRegion("region", rect);
        
        stripper.extractRegions(document.getPage(pageNum - 1));
        
        return stripper.getTextForRegion("region");
    }
    
    /**
     * Check if PDF contains images
     */
    public boolean hasImages(PDDocument document) {
        try {
            return document.getPages().stream()
                .anyMatch(page -> {
                    try {
                        return page.getResources().getXObjectNames()
                            .iterator().hasNext();
                    } catch (Exception e) {
                        return false;
                    }
                });
        } catch (Exception e) {
            return false;
        }
    }
}
```

### PDF with OCR Support

**PdfOcrLoader.java**:

```java
package com.example.pipeline.service.loader;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import net.sourceforge.tess4j.Tesseract;
import net.sourceforge.tess4j.TesseractException;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.rendering.PDFRenderer;
import org.springframework.ai.document.Document;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;

import java.awt.image.BufferedImage;
import java.io.IOException;
import java.io.InputStream;
import java.util.*;

@Slf4j
@Service
@RequiredArgsConstructor
public class PdfOcrLoader {
    
    private final Tesseract tesseract;
    
    /**
     * Extract text from scanned PDF using OCR
     */
    public List<Document> loadWithOcr(Resource resource) throws IOException {
        
        log.info("Loading PDF with OCR: {}", resource.getFilename());
        
        List<Document> documents = new ArrayList<>();
        
        try (InputStream inputStream = resource.getInputStream();
             PDDocument pdDocument = PDDocument.load(inputStream)) {
            
            PDFRenderer renderer = new PDFRenderer(pdDocument);
            int totalPages = pdDocument.getNumberOfPages();
            
            for (int pageNum = 0; pageNum < totalPages; pageNum++) {
                log.debug("OCR processing page {}/{}", pageNum + 1, totalPages);
                
                // Render page as image
                BufferedImage image = renderer.renderImageWithDPI(
                    pageNum,
                    300 // DPI for quality OCR
                );
                
                // Perform OCR
                String text = performOcr(image);
                
                if (text != null && !text.trim().isEmpty()) {
                    Map<String, Object> metadata = new HashMap<>();
                    metadata.put("filename", resource.getFilename());
                    metadata.put("page_number", pageNum + 1);
                    metadata.put("total_pages", totalPages);
                    metadata.put("processing_method", "ocr");
                    
                    documents.add(new Document(text, metadata));
                }
            }
            
            log.info("OCR extracted {} pages from PDF", documents.size());
            
        } catch (Exception e) {
            log.error("OCR processing failed", e);
            throw new IOException("OCR failed", e);
        }
        
        return documents;
    }
    
    private String performOcr(BufferedImage image) {
        try {
            return tesseract.doOCR(image);
        } catch (TesseractException e) {
            log.error("OCR failed for image", e);
            return "";
        }
    }
    
    /**
     * Configure Tesseract
     */
    @org.springframework.context.annotation.Bean
    public Tesseract tesseract() {
        Tesseract instance = new Tesseract();
        
        // Set path to tessdata (language data)
        instance.setDatapath("/usr/share/tesseract-ocr/4.00/tessdata");
        
        // Set language (default English)
        instance.setLanguage("eng");
        
        // Set OCR Engine Mode (3 = Default, based on what is available)
        instance.setOcrEngineMode(3);
        
        // Set Page Segmentation Mode (1 = Automatic page segmentation with OSD)
        instance.setPageSegMode(1);
        
        return instance;
    }
}
```

---

## Word Document Processing

### Word Document Loader

**WordDocumentLoader.java**:

```java
package com.example.pipeline.service.loader;

import com.example.pipeline.service.DocumentLoaderService;
import com.example.pipeline.service.DocumentMetadata;
import lombok.extern.slf4j.Slf4j;
import org.apache.poi.xwpf.usermodel.*;
import org.apache.poi.hwpf.HWPFDocument;
import org.apache.poi.hwpf.extractor.WordExtractor;
import org.springframework.ai.document.Document;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.InputStream;
import java.util.*;

@Slf4j
@Service
public class WordDocumentLoader implements DocumentLoaderService {
    
    private static final List<String> SUPPORTED_TYPES = List.of(
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document", // .docx
        "application/msword" // .doc
    );
    
    @Override
    public List<Document> load(Resource resource) throws IOException {
        
        String filename = resource.getFilename();
        log.info("Loading Word document: {}", filename);
        
        if (filename.endsWith(".docx")) {
            return loadDocx(resource);
        } else if (filename.endsWith(".doc")) {
            return loadDoc(resource);
        } else {
            throw new IOException("Unsupported Word format: " + filename);
        }
    }
    
    /**
     * Load modern Word document (.docx)
     */
    private List<Document> loadDocx(Resource resource) throws IOException {
        
        List<Document> documents = new ArrayList<>();
        
        try (InputStream inputStream = resource.getInputStream();
             XWPFDocument document = new XWPFDocument(inputStream)) {
            
            // Extract metadata
            DocumentMetadata metadata = extractDocxMetadata(document, resource);
            
            // Extract text content
            StringBuilder contentBuilder = new StringBuilder();
            
            // Process paragraphs
            for (XWPFParagraph paragraph : document.getParagraphs()) {
                String text = paragraph.getText();
                if (text != null && !text.trim().isEmpty()) {
                    contentBuilder.append(text).append("\n");
                }
            }
            
            // Process tables
            for (XWPFTable table : document.getTables()) {
                String tableText = extractTableText(table);
                if (!tableText.isEmpty()) {
                    contentBuilder.append("\n[Table]\n");
                    contentBuilder.append(tableText);
                    contentBuilder.append("\n");
                }
            }
            
            // Process headers/footers
            for (XWPFHeader header : document.getHeaderList()) {
                String headerText = header.getText();
                if (headerText != null && !headerText.trim().isEmpty()) {
                    contentBuilder.append("\n[Header]\n");
                    contentBuilder.append(headerText).append("\n");
                }
            }
            
            String content = contentBuilder.toString();
            documents.add(new Document(content, metadata.toMap()));
            
            log.info("Extracted {} characters from Word document", content.length());
            
        } catch (Exception e) {
            log.error("Failed to load .docx: {}", resource.getFilename(), e);
            throw new IOException("Word document processing failed", e);
        }
        
        return documents;
    }
    
    /**
     * Load legacy Word document (.doc)
     */
    private List<Document> loadDoc(Resource resource) throws IOException {
        
        List<Document> documents = new ArrayList<>();
        
        try (InputStream inputStream = resource.getInputStream();
             HWPFDocument document = new HWPFDocument(inputStream)) {
            
            WordExtractor extractor = new WordExtractor(document);
            
            String text = extractor.getText();
            
            Map<String, Object> metadata = new HashMap<>();
            metadata.put("filename", resource.getFilename());
            metadata.put("mime_type", "application/msword");
            metadata.put("format", "doc");
            
            documents.add(new Document(text, metadata));
            
            log.info("Extracted {} characters from .doc", text.length());
            
        } catch (Exception e) {
            log.error("Failed to load .doc: {}", resource.getFilename(), e);
            throw new IOException("Legacy Word processing failed", e);
        }
        
        return documents;
    }
    
    private DocumentMetadata extractDocxMetadata(
            XWPFDocument document,
            Resource resource) {
        
        var coreProps = document.getProperties().getCoreProperties();
        
        return DocumentMetadata.builder()
            .filename(resource.getFilename())
            .mimeType("application/vnd.openxmlformats-officedocument.wordprocessingml.document")
            .title(coreProps.getTitle())
            .author(coreProps.getCreator())
            .subject(coreProps.getSubject())
            .keywords(coreProps.getKeywords())
            .creationDate(coreProps.getCreated() != null ?
                coreProps.getCreated().getTime() : null)
            .modificationDate(coreProps.getModified() != null ?
                coreProps.getModified().getTime() : null)
            .build();
    }
    
    private String extractTableText(XWPFTable table) {
        StringBuilder tableText = new StringBuilder();
        
        for (XWPFTableRow row : table.getRows()) {
            List<String> cellTexts = new ArrayList<>();
            
            for (XWPFTableCell cell : row.getTableCells()) {
                String cellText = cell.getText().trim();
                cellTexts.add(cellText);
            }
            
            tableText.append(String.join(" | ", cellTexts)).append("\n");
        }
        
        return tableText.toString();
    }
    
    @Override
    public DocumentMetadata extractMetadata(Resource resource) throws IOException {
        // Implementation would open document and extract only metadata
        return null;
    }
    
    @Override
    public boolean supports(String mimeType) {
        return SUPPORTED_TYPES.contains(mimeType);
    }
    
    @Override
    public List<String> getSupportedMimeTypes() {
        return SUPPORTED_TYPES;
    }
}
```

---

## Excel Processing

### Excel Loader

**ExcelDocumentLoader.java**:

```java
package com.example.pipeline.service.loader;

import com.example.pipeline.service.DocumentLoaderService;
import com.example.pipeline.service.DocumentMetadata;
import lombok.extern.slf4j.Slf4j;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.springframework.ai.document.Document;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.InputStream;
import java.util.*;

@Slf4j
@Service
public class ExcelDocumentLoader implements DocumentLoaderService {
    
    private static final List<String> SUPPORTED_TYPES = List.of(
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", // .xlsx
        "application/vnd.ms-excel" // .xls
    );
    
    @Override
    public List<Document> load(Resource resource) throws IOException {
        
        String filename = resource.getFilename();
        log.info("Loading Excel document: {}", filename);
        
        List<Document> documents = new ArrayList<>();
        
        try (InputStream inputStream = resource.getInputStream();
             Workbook workbook = createWorkbook(inputStream, filename)) {
            
            // Extract metadata
            DocumentMetadata metadata = extractExcelMetadata(workbook, resource);
            
            // Process each sheet
            for (int sheetIndex = 0; sheetIndex < workbook.getNumberOfSheets(); sheetIndex++) {
                Sheet sheet = workbook.getSheetAt(sheetIndex);
                
                if (sheet.getPhysicalNumberOfRows() == 0) {
                    continue; // Skip empty sheets
                }
                
                String sheetContent = extractSheetContent(sheet);
                
                if (!sheetContent.isEmpty()) {
                    Map<String, Object> sheetMetadata = new HashMap<>(metadata.toMap());
                    sheetMetadata.put("sheet_name", sheet.getSheetName());
                    sheetMetadata.put("sheet_index", sheetIndex);
                    sheetMetadata.put("row_count", sheet.getPhysicalNumberOfRows());
                    
                    documents.add(new Document(sheetContent, sheetMetadata));
                }
            }
            
            log.info("Extracted {} sheets from Excel", documents.size());
            
        } catch (Exception e) {
            log.error("Failed to load Excel: {}", resource.getFilename(), e);
            throw new IOException("Excel processing failed", e);
        }
        
        return documents;
    }
    
    private Workbook createWorkbook(InputStream inputStream, String filename) 
            throws IOException {
        
        if (filename.endsWith(".xlsx")) {
            return new XSSFWorkbook(inputStream);
        } else if (filename.endsWith(".xls")) {
            return new HSSFWorkbook(inputStream);
        } else {
            throw new IOException("Unsupported Excel format: " + filename);
        }
    }
    
    private String extractSheetContent(Sheet sheet) {
        StringBuilder content = new StringBuilder();
        
        DataFormatter formatter = new DataFormatter();
        
        // Detect header row
        Row headerRow = sheet.getRow(sheet.getFirstRowNum());
        List<String> headers = new ArrayList<>();
        
        if (headerRow != null) {
            for (Cell cell : headerRow) {
                String headerValue = formatter.formatCellValue(cell);
                headers.add(headerValue);
            }
            content.append("Headers: ").append(String.join(", ", headers)).append("\n\n");
        }
        
        // Process data rows
        for (int rowNum = sheet.getFirstRowNum() + 1; rowNum <= sheet.getLastRowNum(); rowNum++) {
            Row row = sheet.getRow(rowNum);
            
            if (row == null) continue;
            
            List<String> cellValues = new ArrayList<>();
            
            for (int cellNum = 0; cellNum < headers.size(); cellNum++) {
                Cell cell = row.getCell(cellNum);
                String cellValue = cell != null ? 
                    formatter.formatCellValue(cell) : "";
                cellValues.add(cellValue);
            }
            
            // Only add non-empty rows
            if (cellValues.stream().anyMatch(v -> !v.trim().isEmpty())) {
                for (int i = 0; i < headers.size() && i < cellValues.size(); i++) {
                    if (!cellValues.get(i).trim().isEmpty()) {
                        content.append(headers.get(i))
                            .append(": ")
                            .append(cellValues.get(i))
                            .append("; ");
                    }
                }
                content.append("\n");
            }
        }
        
        return content.toString();
    }
    
    /**
     * Extract structured data as JSON-like format
     */
    public String extractAsStructured(Sheet sheet) {
        StringBuilder json = new StringBuilder();
        json.append("[\n");
        
        DataFormatter formatter = new DataFormatter();
        
        Row headerRow = sheet.getRow(sheet.getFirstRowNum());
        List<String> headers = new ArrayList<>();
        
        if (headerRow != null) {
            for (Cell cell : headerRow) {
                headers.add(formatter.formatCellValue(cell));
            }
        }
        
        for (int rowNum = sheet.getFirstRowNum() + 1; rowNum <= sheet.getLastRowNum(); rowNum++) {
            Row row = sheet.getRow(rowNum);
            if (row == null) continue;
            
            json.append("  {\n");
            
            for (int cellNum = 0; cellNum < headers.size(); cellNum++) {
                Cell cell = row.getCell(cellNum);
                String value = cell != null ? formatter.formatCellValue(cell) : "";
                
                json.append("    \"").append(headers.get(cellNum)).append("\": ")
                    .append("\"").append(value).append("\"");
                
                if (cellNum < headers.size() - 1) {
                    json.append(",");
                }
                json.append("\n");
            }
            
            json.append("  }");
            if (rowNum < sheet.getLastRowNum()) {
                json.append(",");
            }
            json.append("\n");
        }
        
        json.append("]");
        return json.toString();
    }
    
    private DocumentMetadata extractExcelMetadata(
            Workbook workbook,
            Resource resource) {
        
        return DocumentMetadata.builder()
            .filename(resource.getFilename())
            .mimeType(resource.getFilename().endsWith(".xlsx") ?
                "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" :
                "application/vnd.ms-excel")
            .sheetCount(workbook.getNumberOfSheets())
            .build();
    }
    
    @Override
    public DocumentMetadata extractMetadata(Resource resource) throws IOException {
        // Implementation would open workbook and extract only metadata
        return null;
    }
    
    @Override
    public boolean supports(String mimeType) {
        return SUPPORTED_TYPES.contains(mimeType);
    }
    
    @Override
    public List<String> getSupportedMimeTypes() {
        return SUPPORTED_TYPES;
    }
}
```

## Text Extraction and Cleaning

### Text Cleaning Service

**TextCleaningService.java**:

```java
package com.example.pipeline.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.regex.Pattern;

@Slf4j
@Service
public class TextCleaningService {
    
    // Common patterns to remove
    private static final Pattern HEADER_FOOTER_PATTERN = 
        Pattern.compile("^(Page \\d+|\\d+)$", Pattern.MULTILINE);
    
    private static final Pattern EXCESSIVE_WHITESPACE = 
        Pattern.compile("\\s{3,}");
    
    private static final Pattern MULTIPLE_NEWLINES = 
        Pattern.compile("\\n{4,}");
    
    private static final Pattern SPECIAL_CHARS = 
        Pattern.compile("[\\x00-\\x08\\x0B\\x0C\\x0E-\\x1F\\x7F]");
    
    private static final Pattern URL_PATTERN = 
        Pattern.compile("https?://[^\\s]+");
    
    private static final Pattern EMAIL_PATTERN = 
        Pattern.compile("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}");
    
    /**
     * Comprehensive text cleaning
     */
    public String clean(String text, CleaningOptions options) {
        
        if (text == null || text.isEmpty()) {
            return "";
        }
        
        String cleaned = text;
        
        // Remove control characters
        cleaned = SPECIAL_CHARS.matcher(cleaned).replaceAll("");
        
        // Normalize line endings
        cleaned = cleaned.replace("\r\n", "\n").replace("\r", "\n");
        
        // Remove headers and footers
        if (options.isRemoveHeadersFooters()) {
            cleaned = HEADER_FOOTER_PATTERN.matcher(cleaned).replaceAll("");
        }
        
        // Handle URLs
        if (options.isRemoveUrls()) {
            cleaned = URL_PATTERN.matcher(cleaned).replaceAll("");
        } else if (options.isNormalizeUrls()) {
            cleaned = URL_PATTERN.matcher(cleaned).replaceAll("[URL]");
        }
        
        // Handle emails
        if (options.isRemoveEmails()) {
            cleaned = EMAIL_PATTERN.matcher(cleaned).replaceAll("");
        }
        
        // Normalize whitespace
        cleaned = EXCESSIVE_WHITESPACE.matcher(cleaned).replaceAll("  ");
        
        // Normalize newlines (max 3 consecutive)
        cleaned = MULTIPLE_NEWLINES.matcher(cleaned).replaceAll("\n\n\n");
        
        // Remove bullet points if requested
        if (options.isNormalizeBullets()) {
            cleaned = normalizeBulletPoints(cleaned);
        }
        
        // Fix common encoding issues
        cleaned = fixEncodingIssues(cleaned);
        
        // Trim lines
        cleaned = cleaned.lines()
            .map(String::trim)
            .collect(java.util.stream.Collectors.joining("\n"));
        
        // Final trim
        cleaned = cleaned.trim();
        
        log.debug("Cleaned text: {} -> {} characters", 
            text.length(), cleaned.length());
        
        return cleaned;
    }
    
    /**
     * Remove boilerplate content
     */
    public String removeBoilerplate(String text) {
        
        // Common boilerplate patterns
        String[] boilerplatePatterns = {
            "(?i)^confidential.*$",
            "(?i)^proprietary.*$",
            "(?i)^copyright \\d{4}.*$",
            "(?i)^all rights reserved.*$",
            "(?i)^this document is.*$",
            "(?i)^printed on \\d{2}/\\d{2}/\\d{4}.*$"
        };
        
        String cleaned = text;
        
        for (String pattern : boilerplatePatterns) {
            cleaned = cleaned.replaceAll(pattern, "");
        }
        
        return cleaned;
    }
    
    /**
     * Extract main content (remove navigation, sidebars, etc.)
     */
    public String extractMainContent(String text) {
        
        // Split into paragraphs
        String[] paragraphs = text.split("\n\n+");
        
        // Keep paragraphs with substantial content
        StringBuilder mainContent = new StringBuilder();
        
        for (String paragraph : paragraphs) {
            // Skip short paragraphs (likely navigation/boilerplate)
            if (paragraph.length() < 50) {
                continue;
            }
            
            // Skip paragraphs with too many links
            long linkCount = URL_PATTERN.matcher(paragraph)
                .results().count();
            
            if (linkCount > 3) {
                continue;
            }
            
            mainContent.append(paragraph).append("\n\n");
        }
        
        return mainContent.toString().trim();
    }
    
    /**
     * Normalize bullet points and list markers
     */
    private String normalizeBulletPoints(String text) {
        
        return text.lines()
            .map(line -> {
                // Replace various bullet point characters
                if (line.trim().matches("^[•●○■□▪▫-]\\s+.*")) {
                    return "- " + line.trim().substring(1).trim();
                }
                // Normalize numbered lists
                if (line.trim().matches("^\\d+[.)]\\s+.*")) {
                    return line.trim().replaceFirst("^\\d+[.)]\\s+", "");
                }
                return line;
            })
            .collect(java.util.stream.Collectors.joining("\n"));
    }
    
    /**
     * Fix common encoding issues
     */
    private String fixEncodingIssues(String text) {
        
        return text
            // Fix smart quotes
            .replace("'", "'")
            .replace("'", "'")
            .replace(""", "\"")
            .replace(""", "\"")
            // Fix dashes
            .replace("–", "-")
            .replace("—", "-")
            // Fix ellipsis
            .replace("…", "...")
            // Fix non-breaking spaces
            .replace("\u00A0", " ")
            // Fix zero-width spaces
            .replace("\u200B", "");
    }
    
    /**
     * Detect and fix language-specific issues
     */
    public String normalizeLanguage(String text, String language) {
        
        if (language == null) {
            language = detectLanguage(text);
        }
        
        switch (language.toLowerCase()) {
            case "en":
                return normalizeEnglish(text);
            case "es":
                return normalizeSpanish(text);
            case "fr":
                return normalizeFrench(text);
            default:
                return text;
        }
    }
    
    private String normalizeEnglish(String text) {
        // English-specific normalizations
        return text
            .replaceAll("\\bcan't\\b", "cannot")
            .replaceAll("\\bwon't\\b", "will not")
            .replaceAll("\\bI'm\\b", "I am");
    }
    
    private String normalizeSpanish(String text) {
        // Spanish-specific normalizations
        return text;
    }
    
    private String normalizeFrench(String text) {
        // French-specific normalizations
        return text;
    }
    
    private String detectLanguage(String text) {
        // Simple language detection (would use library in production)
        if (text.matches(".*\\b(the|and|or|is|are)\\b.*")) {
            return "en";
        }
        if (text.matches(".*\\b(el|la|los|las|y|o)\\b.*")) {
            return "es";
        }
        if (text.matches(".*\\b(le|la|les|et|ou)\\b.*")) {
            return "fr";
        }
        return "unknown";
    }
    
    @lombok.Data
    @lombok.Builder
    public static class CleaningOptions {
        @lombok.Builder.Default
        private boolean removeHeadersFooters = true;
        
        @lombok.Builder.Default
        private boolean removeUrls = false;
        
        @lombok.Builder.Default
        private boolean normalizeUrls = true;
        
        @lombok.Builder.Default
        private boolean removeEmails = false;
        
        @lombok.Builder.Default
        private boolean normalizeBullets = true;
        
        @lombok.Builder.Default
        private boolean removeBoilerplate = true;
        
        public static CleaningOptions defaults() {
            return CleaningOptions.builder().build();
        }
    }
}
```

---

## Advanced Chunking Strategies

### Intelligent Chunking Service

**ChunkingService.java**:

```java
package com.example.pipeline.service;

import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.document.Document;
import org.springframework.stereotype.Service;

import java.util.*;
import java.util.stream.Collectors;

@Slf4j
@Service
public class ChunkingService {
    
    /**
     * Recursive character-based chunking with semantic awareness
     */
    public List<Document> chunkRecursive(
            String text,
            Map<String, Object> baseMetadata,
            ChunkingConfig config) {
        
        List<String> separators = List.of(
            "\n\n\n",  // Section breaks
            "\n\n",    // Paragraph breaks
            "\n",      // Line breaks
            ". ",      // Sentences
            "! ",
            "? ",
            "; ",
            ", ",
            " "        // Words
        );
        
        return splitRecursive(
            text,
            separators,
            baseMetadata,
            config,
            0
        );
    }
    
    private List<Document> splitRecursive(
            String text,
            List<String> separators,
            Map<String, Object> baseMetadata,
            ChunkingConfig config,
            int depth) {
        
        List<Document> chunks = new ArrayList<>();
        
        if (text.length() <= config.getChunkSize()) {
            chunks.add(createChunk(text, baseMetadata, 0, 1));
            return chunks;
        }
        
        if (separators.isEmpty()) {
            // No more separators, force split
            return forceSplit(text, baseMetadata, config);
        }
        
        String separator = separators.get(0);
        String[] splits = text.split(Pattern.quote(separator));
        List<String> remainingSeparators = separators.subList(1, separators.size());
        
        StringBuilder currentChunk = new StringBuilder();
        int chunkIndex = 0;
        
        for (int i = 0; i < splits.length; i++) {
            String split = splits[i];
            
            if (currentChunk.length() + split.length() + separator.length() 
                    <= config.getChunkSize()) {
                
                if (currentChunk.length() > 0) {
                    currentChunk.append(separator);
                }
                currentChunk.append(split);
                
            } else {
                // Current chunk is full
                if (currentChunk.length() > 0) {
                    String chunkText = currentChunk.toString();
                    
                    // Add overlap from previous chunk
                    if (chunkIndex > 0 && config.getChunkOverlap() > 0) {
                        String overlap = chunks.get(chunks.size() - 1)
                            .getContent();
                        overlap = getLastNCharacters(overlap, config.getChunkOverlap());
                        chunkText = overlap + separator + chunkText;
                    }
                    
                    chunks.add(createChunk(chunkText, baseMetadata, chunkIndex, -1));
                    chunkIndex++;
                }
                
                // Start new chunk with current split
                currentChunk = new StringBuilder(split);
            }
        }
        
        // Add final chunk
        if (currentChunk.length() > 0) {
            String chunkText = currentChunk.toString();
            
            if (chunkIndex > 0 && config.getChunkOverlap() > 0) {
                String overlap = chunks.get(chunks.size() - 1).getContent();
                overlap = getLastNCharacters(overlap, config.getChunkOverlap());
                chunkText = overlap + separator + chunkText;
            }
            
            chunks.add(createChunk(chunkText, baseMetadata, chunkIndex, -1));
        }
        
        // Update total chunks count
        for (int i = 0; i < chunks.size(); i++) {
            chunks.get(i).getMetadata().put("total_chunks", chunks.size());
        }
        
        return chunks;
    }
    
    /**
     * Semantic chunking based on topic boundaries
     */
    public List<Document> chunkSemantic(
            String text,
            Map<String, Object> baseMetadata,
            ChunkingConfig config) {
        
        // Split into sentences
        String[] sentences = text.split("(?<=[.!?])\\s+");
        
        List<Document> chunks = new ArrayList<>();
        StringBuilder currentChunk = new StringBuilder();
        String previousTopic = null;
        int chunkIndex = 0;
        
        for (String sentence : sentences) {
            
            // Detect topic shift (simplified - in production use ML model)
            String currentTopic = detectTopic(sentence);
            
            boolean topicShift = previousTopic != null && 
                !currentTopic.equals(previousTopic);
            
            if (currentChunk.length() + sentence.length() > config.getChunkSize() ||
                topicShift) {
                
                if (currentChunk.length() > 0) {
                    chunks.add(createChunk(
                        currentChunk.toString(),
                        baseMetadata,
                        chunkIndex++,
                        -1
                    ));
                    currentChunk = new StringBuilder();
                }
            }
            
            currentChunk.append(sentence).append(" ");
            previousTopic = currentTopic;
        }
        
        // Add final chunk
        if (currentChunk.length() > 0) {
            chunks.add(createChunk(
                currentChunk.toString(),
                baseMetadata,
                chunkIndex,
                chunks.size() + 1
            ));
        }
        
        return chunks;
    }
    
    /**
     * Fixed-size chunking with sentence boundary awareness
     */
    public List<Document> chunkFixed(
            String text,
            Map<String, Object> baseMetadata,
            ChunkingConfig config) {
        
        List<Document> chunks = new ArrayList<>();
        String[] sentences = text.split("(?<=[.!?])\\s+");
        
        StringBuilder currentChunk = new StringBuilder();
        int chunkIndex = 0;
        
        for (String sentence : sentences) {
            
            if (currentChunk.length() + sentence.length() > config.getChunkSize()) {
                
                if (currentChunk.length() > 0) {
                    chunks.add(createChunk(
                        currentChunk.toString(),
                        baseMetadata,
                        chunkIndex++,
                        -1
                    ));
                    
                    // Add overlap
                    if (config.getChunkOverlap() > 0) {
                        String overlap = getLastNCharacters(
                            currentChunk.toString(),
                            config.getChunkOverlap()
                        );
                        currentChunk = new StringBuilder(overlap);
                    } else {
                        currentChunk = new StringBuilder();
                    }
                }
            }
            
            currentChunk.append(sentence).append(" ");
        }
        
        if (currentChunk.length() > 0) {
            chunks.add(createChunk(
                currentChunk.toString(),
                baseMetadata,
                chunkIndex,
                chunks.size() + 1
            ));
        }
        
        return chunks;
    }
    
    /**
     * Markdown-aware chunking
     */
    public List<Document> chunkMarkdown(
            String text,
            Map<String, Object> baseMetadata,
            ChunkingConfig config) {
        
        // Split by headers
        String[] sections = text.split("(?=^#{1,6}\\s)", 
            java.util.regex.Pattern.MULTILINE);
        
        List<Document> chunks = new ArrayList<>();
        int chunkIndex = 0;
        
        for (String section : sections) {
            
            if (section.length() <= config.getChunkSize()) {
                chunks.add(createChunk(section, baseMetadata, chunkIndex++, -1));
            } else {
                // Section too large, recursively chunk
                List<Document> subChunks = chunkRecursive(
                    section,
                    baseMetadata,
                    config
                );
                chunks.addAll(subChunks);
                chunkIndex += subChunks.size();
            }
        }
        
        return chunks;
    }
    
    private Document createChunk(
            String text,
            Map<String, Object> baseMetadata,
            int chunkIndex,
            int totalChunks) {
        
        Map<String, Object> metadata = new HashMap<>(baseMetadata);
        metadata.put("chunk_index", chunkIndex);
        
        if (totalChunks > 0) {
            metadata.put("total_chunks", totalChunks);
        }
        
        metadata.put("chunk_size", text.length());
        
        return new Document(text.trim(), metadata);
    }
    
    private List<Document> forceSplit(
            String text,
            Map<String, Object> baseMetadata,
            ChunkingConfig config) {
        
        List<Document> chunks = new ArrayList<>();
        int chunkSize = config.getChunkSize();
        int overlap = config.getChunkOverlap();
        
        for (int i = 0; i < text.length(); i += (chunkSize - overlap)) {
            int end = Math.min(i + chunkSize, text.length());
            String chunk = text.substring(i, end);
            chunks.add(createChunk(chunk, baseMetadata, chunks.size(), -1));
        }
        
        return chunks;
    }
    
    private String getLastNCharacters(String text, int n) {
        if (text.length() <= n) {
            return text;
        }
        
        String last = text.substring(text.length() - n);
        
        // Try to start at sentence boundary
        int periodIndex = last.indexOf(". ");
        if (periodIndex > 0 && periodIndex < n / 2) {
            return last.substring(periodIndex + 2);
        }
        
        return last;
    }
    
    private String detectTopic(String sentence) {
        // Simplified topic detection
        // In production, use embeddings or topic modeling
        
        if (sentence.toLowerCase().contains("introduction") ||
            sentence.toLowerCase().contains("overview")) {
            return "introduction";
        }
        
        if (sentence.toLowerCase().contains("conclusion") ||
            sentence.toLowerCase().contains("summary")) {
            return "conclusion";
        }
        
        if (sentence.toLowerCase().contains("method") ||
            sentence.toLowerCase().contains("approach")) {
            return "methodology";
        }
        
        return "content";
    }
    
    @Data
    @lombok.Builder
    public static class ChunkingConfig {
        
        @lombok.Builder.Default
        private Integer chunkSize = 1000;
        
        @lombok.Builder.Default
        private Integer chunkOverlap = 200;
        
        @lombok.Builder.Default
        private ChunkingStrategy strategy = ChunkingStrategy.RECURSIVE;
        
        public static ChunkingConfig defaults() {
            return ChunkingConfig.builder().build();
        }
    }
    
    public enum ChunkingStrategy {
        RECURSIVE,
        SEMANTIC,
        FIXED,
        MARKDOWN
    }
}
```

---

## Metadata Extraction

### Metadata Extraction Service

**MetadataExtractionService.java**:

```java
package com.example.pipeline.service;

import com.drew.imaging.ImageMetadataReader;
import com.drew.metadata.Metadata;
import com.drew.metadata.exif.ExifSubIFDDirectory;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.apache.tika.Tika;
import org.apache.tika.metadata.TikaCoreProperties;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.InputStream;
import java.security.MessageDigest;
import java.util.*;

@Slf4j
@Service
public class MetadataExtractionService {
    
    private final Tika tika = new Tika();
    
    /**
     * Extract comprehensive metadata from document
     */
    public DocumentMetadata extractMetadata(Resource resource) throws IOException {
        
        log.debug("Extracting metadata from: {}", resource.getFilename());
        
        DocumentMetadata.DocumentMetadataBuilder builder = 
            DocumentMetadata.builder();
        
        // Basic file information
        builder.filename(resource.getFilename());
        builder.fileSize(resource.contentLength());
        
        try (InputStream inputStream = resource.getInputStream()) {
            
            // Detect MIME type
            String mimeType = tika.detect(inputStream);
            builder.mimeType(mimeType);
            
            // Calculate checksum
            String checksum = calculateChecksum(resource);
            builder.checksum(checksum);
            
            // Extract Tika metadata
            org.apache.tika.metadata.Metadata tikaMetadata = 
                extractTikaMetadata(resource);
            
            enrichFromTikaMetadata(builder, tikaMetadata);
            
            // Extract format-specific metadata
            if (mimeType.startsWith("image/")) {
                extractImageMetadata(builder, resource);
            }
            
        } catch (Exception e) {
            log.error("Error extracting metadata", e);
        }
        
        return builder.build();
    }
    
    /**
     * Extract metadata using Apache Tika
     */
    private org.apache.tika.metadata.Metadata extractTikaMetadata(Resource resource) 
            throws IOException {
        
        org.apache.tika.metadata.Metadata metadata = 
            new org.apache.tika.metadata.Metadata();
        
        try (InputStream inputStream = resource.getInputStream()) {
            org.apache.tika.parser.AutoDetectParser parser = 
                new org.apache.tika.parser.AutoDetectParser();
            
            org.apache.tika.sax.BodyContentHandler handler = 
                new org.apache.tika.sax.BodyContentHandler(-1);
            
            parser.parse(
                inputStream,
                handler,
                metadata,
                new org.apache.tika.parser.ParseContext()
            );
        } catch (Exception e) {
            log.warn("Tika parsing failed", e);
        }
        
        return metadata;
    }
    
    /**
     * Enrich metadata from Tika extraction
     */
    private void enrichFromTikaMetadata(
            DocumentMetadata.DocumentMetadataBuilder builder,
            org.apache.tika.metadata.Metadata tikaMetadata) {
        
        // Title
        String title = tikaMetadata.get(TikaCoreProperties.TITLE);
        if (title != null) {
            builder.title(title);
        }
        
        // Author
        String author = tikaMetadata.get(TikaCoreProperties.CREATOR);
        if (author != null) {
            builder.author(author);
        }
        
        // Subject
        String subject = tikaMetadata.get(TikaCoreProperties.SUBJECT);
        if (subject != null) {
            builder.subject(subject);
        }
        
        // Keywords
        String keywords = tikaMetadata.get(TikaCoreProperties.KEYWORDS);
        if (keywords != null) {
            builder.keywords(keywords);
        }
        
        // Creation date
        String created = tikaMetadata.get(TikaCoreProperties.CREATED);
        if (created != null) {
            builder.creationDate(parseDate(created));
        }
        
        // Modified date
        String modified = tikaMetadata.get(TikaCoreProperties.MODIFIED);
        if (modified != null) {
            builder.modificationDate(parseDate(modified));
        }
        
        // Language
        String language = tikaMetadata.get(TikaCoreProperties.LANGUAGE);
        if (language != null) {
            builder.language(language);
        }
        
        // Page count
        String pageCount = tikaMetadata.get("xmpTPg:NPages");
        if (pageCount != null) {
            try {
                builder.pageCount(Integer.parseInt(pageCount));
            } catch (NumberFormatException e) {
                // Ignore
            }
        }
    }
    
    /**
     * Extract image-specific metadata
     */
    private void extractImageMetadata(
            DocumentMetadata.DocumentMetadataBuilder builder,
            Resource resource) {
        
        try (InputStream inputStream = resource.getInputStream()) {
            
            Metadata metadata = ImageMetadataReader.readMetadata(inputStream);
            
            // Extract EXIF data
            ExifSubIFDDirectory directory = 
                metadata.getFirstDirectoryOfType(ExifSubIFDDirectory.class);
            
            if (directory != null) {
                Date dateTime = directory.getDate(ExifSubIFDDirectory.TAG_DATETIME_ORIGINAL);
                if (dateTime != null) {
                    builder.creationDate(dateTime);
                }
                
                // Camera info
                Map<String, Object> custom = new HashMap<>();
                custom.put("camera_make", 
                    directory.getString(ExifSubIFDDirectory.TAG_MAKE));
                custom.put("camera_model", 
                    directory.getString(ExifSubIFDDirectory.TAG_MODEL));
                builder.customMetadata(custom);
            }
            
        } catch (Exception e) {
            log.warn("Failed to extract image metadata", e);
        }
    }
    
    /**
     * Calculate file checksum
     */
    private String calculateChecksum(Resource resource) {
        try (InputStream inputStream = resource.getInputStream()) {
            MessageDigest digest = MessageDigest.getInstance("SHA-256");
            
            byte[] buffer = new byte[8192];
            int bytesRead;
            
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                digest.update(buffer, 0, bytesRead);
            }
            
            byte[] hashBytes = digest.digest();
            
            StringBuilder hexString = new StringBuilder();
            for (byte b : hashBytes) {
                String hex = Integer.toHexString(0xff & b);
                if (hex.length() == 1) hexString.append('0');
                hexString.append(hex);
            }
            
            return hexString.toString();
            
        } catch (Exception e) {
            log.error("Failed to calculate checksum", e);
            return null;
        }
    }
    
    /**
     * Auto-generate tags from content
     */
    public List<String> generateTags(String content) {
        
        // Extract keywords using simple frequency analysis
        Map<String, Integer> wordFrequency = new HashMap<>();
        
        String[] words = content.toLowerCase()
            .replaceAll("[^a-z\\s]", "")
            .split("\\s+");
        
        Set<String> stopWords = getStopWords();
        
        for (String word : words) {
            if (word.length() > 3 && !stopWords.contains(word)) {
                wordFrequency.merge(word, 1, Integer::sum);
            }
        }
        
        return wordFrequency.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .limit(10)
            .map(Map.Entry::getKey)
            .collect(java.util.stream.Collectors.toList());
    }
    
    /**
     * Detect document language
     */
    public String detectLanguage(String content) {
        
        // Simple language detection based on common words
        Map<String, List<String>> languageMarkers = Map.of(
            "en", List.of("the", "and", "is", "are", "was", "were"),
            "es", List.of("el", "la", "los", "las", "y", "es", "son"),
            "fr", List.of("le", "la", "les", "et", "est", "sont"),
            "de", List.of("der", "die", "das", "und", "ist", "sind")
        );
        
        String lower = content.toLowerCase();
        
        Map<String, Integer> scores = new HashMap<>();
        
        for (Map.Entry<String, List<String>> entry : languageMarkers.entrySet()) {
            int score = 0;
            for (String marker : entry.getValue()) {
                if (lower.contains(" " + marker + " ")) {
                    score++;
                }
            }
            scores.put(entry.getKey(), score);
        }
        
        return scores.entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElse("unknown");
    }
    
    private Set<String> getStopWords() {
        return Set.of(
            "the", "and", "is", "are", "was", "were", "been", "be",
            "have", "has", "had", "do", "does", "did", "will", "would",
            "could", "should", "may", "might", "must", "can", "this",
            "that", "these", "those", "with", "from", "into", "for"
        );
    }
    
    private Date parseDate(String dateString) {
        try {
            return new java.text.SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'")
                .parse(dateString);
        } catch (Exception e) {
            return null;
        }
    }
}

@Data
@lombok.Builder
class DocumentMetadata {
    private String filename;
    private String mimeType;
    private Long fileSize;
    private String checksum;
    private String title;
    private String author;
    private String subject;
    private String keywords;
    private String creator;
    private String producer;
    private Date creationDate;
    private Date modificationDate;
    private String language;
    private Integer pageCount;
    private Integer sheetCount;
    private Boolean encrypted;
    private Map<String, Object> customMetadata;
    
    public Map<String, Object> toMap() {
        Map<String, Object> map = new HashMap<>();
        
        if (filename != null) map.put("filename", filename);
        if (mimeType != null) map.put("mime_type", mimeType);
        if (fileSize != null) map.put("file_size", fileSize);
        if (checksum != null) map.put("checksum", checksum);
        if (title != null) map.put("title", title);
        if (author != null) map.put("author", author);
        if (subject != null) map.put("subject", subject);
        if (keywords != null) map.put("keywords", keywords);
        if (creator != null) map.put("creator", creator);
        if (producer != null) map.put("producer", producer);
        if (creationDate != null) map.put("creation_date", creationDate.getTime());
        if (modificationDate != null) map.put("modification_date", modificationDate.getTime());
        if (language != null) map.put("language", language);
        if (pageCount != null) map.put("page_count", pageCount);
        if (sheetCount != null) map.put("sheet_count", sheetCount);
        if (encrypted != null) map.put("encrypted", encrypted);
        if (customMetadata != null) map.putAll(customMetadata);
        
        return map;
    }
}
```

---

## Embedding Generation

### Embedding Service

**EmbeddingGenerationService.java**:

```java
package com.example.pipeline.service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.document.Document;
import org.springframework.ai.embedding.EmbeddingClient;
import org.springframework.ai.embedding.EmbeddingResponse;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import java.util.*;
import java.util.concurrent.*;
import java.util.stream.Collectors;

@Slf4j
@Service
@RequiredArgsConstructor
public class EmbeddingGenerationService {
    
    private final EmbeddingClient embeddingClient;
    private final ExecutorService executorService = 
        Executors.newFixedThreadPool(5);
    
    private static final int BATCH_SIZE = 100;
    private static final int MAX_RETRIES = 3;
    
    /**
     * Generate embeddings for documents with batching
     */
    public List<Document> generateEmbeddings(List<Document> documents) {
        
        log.info("Generating embeddings for {} documents", documents.size());
        
        List<Document> enrichedDocs = new ArrayList<>();
        
        // Process in batches
        for (int i = 0; i < documents.size(); i += BATCH_SIZE) {
            int end = Math.min(i + BATCH_SIZE, documents.size());
            List<Document> batch = documents.subList(i, end);
            
            log.debug("Processing batch {}-{}", i, end);
            
            List<Document> batchResult = generateEmbeddingsBatch(batch);
            enrichedDocs.addAll(batchResult);
        }
        
        log.info("Successfully generated {} embeddings", enrichedDocs.size());
        
        return enrichedDocs;
    }
    
    /**
     * Generate embeddings for a batch of documents
     */
    private List<Document> generateEmbeddingsBatch(List<Document> documents) {
        
        List<String> texts = documents.stream()
            .map(Document::getContent)
            .collect(Collectors.toList());
        
        try {
            EmbeddingResponse response = embeddingClient.embedForResponse(texts);
            
            List<Document> enriched = new ArrayList<>();
            
            for (int i = 0; i < documents.size(); i++) {
                Document doc = documents.get(i);
                
                float[] embedding = response.getResults().get(i)
                    .getOutput()
                    .stream()
                    .map(Double::floatValue)
                    .toArray(Float[]::new);
                
                // Create new document with embedding
                Map<String, Object> metadata = new HashMap<>(doc.getMetadata());
                metadata.put("embedding_model", "text-embedding-3-small");
                metadata.put("embedding_dimension", embedding.length);
                
                Document enrichedDoc = new Document(
                    doc.getContent(),
                    metadata
                );
                enrichedDoc.setEmbedding(Arrays.asList(embedding));
                
                enriched.add(enrichedDoc);
            }
            
            return enriched;
            
        } catch (Exception e) {
            log.error("Batch embedding failed", e);
            
            // Retry with smaller batches or individual docs
            return retryIndividually(documents);
        }
    }
    
    /**
     * Retry failed embeddings individually
     */
    private List<Document> retryIndividually(List<Document> documents) {
        
        log.warn("Retrying {} documents individually", documents.size());
        
        return documents.stream()
            .map(this::generateEmbeddingWithRetry)
            .filter(Objects::nonNull)
            .collect(Collectors.toList());
    }
    
    /**
     * Generate embedding with retry logic
     */
    @Cacheable(value = "embeddings", key = "#document.content.hashCode()")
    private Document generateEmbeddingWithRetry(Document document) {
        
        for (int attempt = 1; attempt <= MAX_RETRIES; attempt++) {
            try {
                EmbeddingResponse response = embeddingClient.embedForResponse(
                    List.of(document.getContent())
                );
                
                float[] embedding = response.getResults().get(0)
                    .getOutput()
                    .stream()
                    .map(Double::floatValue)
                    .toArray(Float[]::new);
                
                Map<String, Object> metadata = new HashMap<>(document.getMetadata());
                metadata.put("embedding_model", "text-embedding-3-small");
                
                Document enriched = new Document(document.getContent(), metadata);
                enriched.setEmbedding(Arrays.asList(embedding));
                
                return enriched;
                
            } catch (Exception e) {
                log.warn("Embedding attempt {} failed", attempt, e);
                
                if (attempt < MAX_RETRIES) {
                    try {
                        Thread.sleep(1000 * attempt); // Exponential backoff
                    } catch (InterruptedException ie) {
                        Thread.currentThread().interrupt();
                    }
                }
            }
        }
        
        log.error("Failed to generate embedding after {} attempts", MAX_RETRIES);
        return null;
    }
    
    /**
     * Parallel embedding generation for large batches
     */
    public CompletableFuture<List<Document>> generateEmbeddingsAsync(
            List<Document> documents) {
        
        return CompletableFuture.supplyAsync(() -> 
            generateEmbeddings(documents),
            executorService
        );
    }
    
    /**
     * Calculate embedding quality metrics
     */
    public EmbeddingQuality assessQuality(List<Document> documents) {
        
        if (documents.isEmpty()) {
            return EmbeddingQuality.builder().build();
        }
        
        int totalDocs = documents.size();
        int successfulEmbeddings = (int) documents.stream()
            .filter(doc -> doc.getEmbedding() != null && !doc.getEmbedding().isEmpty())
            .count();
        
        double successRate = (successfulEmbeddings * 100.0) / totalDocs;
        
        return EmbeddingQuality.builder()
            .totalDocuments(totalDocs)
            .successfulEmbeddings(successfulEmbeddings)
            .successRate(successRate)
            .build();
    }
    
    @lombok.Data
    @lombok.Builder
    public static class EmbeddingQuality {
        private Integer totalDocuments;
        private Integer successfulEmbeddings;
        private Double successRate;
    }
}
```

---

## Batch Processing Pipeline

### Document Processing Pipeline

**DocumentProcessingPipeline.java**:

```java
package com.example.pipeline.service;

import lombok.Data;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.document.Document;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.core.io.Resource;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.util.*;
import java.util.concurrent.CompletableFuture;

@Slf4j
@Service
@RequiredArgsConstructor
public class DocumentProcessingPipeline {
    
    private final DocumentLoaderFactory loaderFactory;
    private final TextCleaningService textCleaning;
    private final MetadataExtractionService metadataExtraction;
    private final ChunkingService chunking;
    private final EmbeddingGenerationService embeddingGeneration;
    private final VectorStore vectorStore;
    
    /**
     * Complete document processing pipeline
     */
    public ProcessingResult processDocument(
            Resource resource,
            ProcessingOptions options) {
        
        long startTime = System.currentTimeMillis();
        String filename = resource.getFilename();
        
        log.info("Starting processing pipeline for: {}", filename);
        
        ProcessingResult.ProcessingResultBuilder resultBuilder = 
            ProcessingResult.builder()
                .filename(filename)
                .startTime(startTime);
        
        try {
            // Step 1: Load document
            log.debug("Step 1: Loading document");
            DocumentLoaderService loader = loaderFactory.getLoader(resource);
            List<Document> documents = loader.load(resource);
            resultBuilder.documentsExtracted(documents.size());
            
            // Step 2: Extract metadata
            log.debug("Step 2: Extracting metadata");
            DocumentMetadata metadata = metadataExtraction.extractMetadata(resource);
            resultBuilder.metadata(metadata);
            
            // Step 3: Clean text
            log.debug("Step 3: Cleaning text");
            documents = cleanDocuments(documents, options);
            
            // Step 4: Chunk documents
            log.debug("Step 4: Chunking documents");
            List<Document> chunks = chunkDocuments(documents, metadata, options);
            resultBuilder.chunksCreated(chunks.size());
            
            // Step 5: Generate embeddings
            log.debug("Step 5: Generating embeddings");
            List<Document> embedded = embeddingGeneration.generateEmbeddings(chunks);
            resultBuilder.embeddingsGenerated(embedded.size());
            
            // Step 6: Store in vector database
            if (options.isStoreVectors()) {
                log.debug("Step 6: Storing vectors");
                vectorStore.add(embedded);
                resultBuilder.vectorsStored(embedded.size());
            }
            
            long duration = System.currentTimeMillis() - startTime;
            resultBuilder.durationMs(duration);
            resultBuilder.success(true);
            
            log.info("Processing completed for {} in {}ms", 
                filename, duration);
            
        } catch (Exception e) {
            log.error("Processing failed for: {}", filename, e);
            resultBuilder.success(false);
            resultBuilder.error(e.getMessage());
        }
        
        return resultBuilder.build();
    }
    
    /**
     * Async batch processing
     */
    @Async("documentProcessingExecutor")
    public CompletableFuture<List<ProcessingResult>> processBatch(
            List<Resource> resources,
            ProcessingOptions options) {
        
        log.info("Starting batch processing for {} documents", resources.size());
        
        List<ProcessingResult> results = new ArrayList<>();
        
        for (Resource resource : resources) {
            ProcessingResult result = processDocument(resource, options);
            results.add(result);
        }
        
        int successCount = (int) results.stream()
            .filter(ProcessingResult::isSuccess)
            .count();
        
        log.info("Batch processing completed: {}/{} successful",
            successCount, resources.size());
        
        return CompletableFuture.completedFuture(results);
    }
    
    private List<Document> cleanDocuments(
            List<Document> documents,
            ProcessingOptions options) {
        
        return documents.stream()
            .map(doc -> {
                String cleaned = textCleaning.clean(
                    doc.getContent(),
                    options.getCleaningOptions()
                );
                return new Document(cleaned, doc.getMetadata());
            })
            .collect(java.util.stream.Collectors.toList());
    }
    
    private List<Document> chunkDocuments(
            List<Document> documents,
            DocumentMetadata metadata,
            ProcessingOptions options) {
        
        List<Document> allChunks = new ArrayList<>();
        
        for (Document doc : documents) {
            List<Document> chunks = chunking.chunkRecursive(
                doc.getContent(),
                mergeMetadata(doc.getMetadata(), metadata),
                options.getChunkingConfig()
            );
            allChunks.addAll(chunks);
        }
        
        return allChunks;
    }
    
    private Map<String, Object> mergeMetadata(
            Map<String, Object> docMetadata,
            DocumentMetadata extractedMetadata) {
        
        Map<String, Object> merged = new HashMap<>(extractedMetadata.toMap());
        merged.putAll(docMetadata);
        return merged;
    }
    
    @Data
    @lombok.Builder
    public static class ProcessingOptions {
        
        @lombok.Builder.Default
        private TextCleaningService.CleaningOptions cleaningOptions = 
            TextCleaningService.CleaningOptions.defaults();
        
        @lombok.Builder.Default
        private ChunkingService.ChunkingConfig chunkingConfig = 
            ChunkingService.ChunkingConfig.defaults();
        
        @lombok.Builder.Default
        private boolean storeVectors = true;
        
        @lombok.Builder.Default
        private boolean extractMetadata = true;
        
        public static ProcessingOptions defaults() {
            return ProcessingOptions.builder().build();
        }
    }
    
    @Data
    @lombok.Builder
    public static class ProcessingResult {
        private String filename;
        private Long startTime;
        private Long durationMs;
        private Boolean success;
        private String error;
        private Integer documentsExtracted;
        private Integer chunksCreated;
        private Integer embeddingsGenerated;
        private Integer vectorsStored;
        private DocumentMetadata metadata;
    }
}
```

### Document Loader Factory

**DocumentLoaderFactory.java**:

```java
package com.example.pipeline.service;

import com.example.pipeline.service.loader.*;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.apache.tika.Tika;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.List;

@Slf4j
@Component
@RequiredArgsConstructor
public class DocumentLoaderFactory {
    
    private final PdfDocumentLoader pdfLoader;
    private final WordDocumentLoader wordLoader;
    private final ExcelDocumentLoader excelLoader;
    private final TikaDocumentLoader tikaLoader; // Fallback
    
    private final Tika tika = new Tika();
    
    /**
     * Get appropriate loader for document
     */
    public DocumentLoaderService getLoader(Resource resource) throws IOException {
        
        String mimeType = detectMimeType(resource);
        
        log.debug("Detected MIME type: {} for file: {}", 
            mimeType, resource.getFilename());
        
        // Try specific loaders first
        for (DocumentLoaderService loader : getLoaders()) {
            if (loader.supports(mimeType)) {
                log.debug("Using loader: {}", loader.getClass().getSimpleName());
                return loader;
            }
        }
        
        // Fallback to Tika
        log.debug("Using fallback Tika loader");
        return tikaLoader;
    }
    
    private String detectMimeType(Resource resource) throws IOException {
        try (var inputStream = resource.getInputStream()) {
            return tika.detect(inputStream, resource.getFilename());
        }
    }
    
    private List<DocumentLoaderService> getLoaders() {
        return List.of(pdfLoader, wordLoader, excelLoader);
    }
}
```

---

## Error Handling and Retry Logic

### Robust Error Handling

**DocumentProcessingErrorHandler.java**:

```java
package com.example.pipeline.error;

import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.*;

@Slf4j
@Component
public class DocumentProcessingErrorHandler {
    
    /**
     * Retry with exponential backoff
     */
    @Retryable(
        value = {IOException.class, RuntimeException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public <T> T executeWithRetry(RetryableOperation<T> operation) 
            throws Exception {
        
        return operation.execute();
    }
    
    /**
     * Handle and categorize errors
     */
    public ProcessingError categorizeError(Exception e, String filename) {
        
        ProcessingError.ErrorCategory category;
        boolean recoverable;
        String solution;
        
        if (e instanceof IOException) {
            category = ProcessingError.ErrorCategory.IO_ERROR;
            recoverable = true;
            solution = "Check file permissions and disk space";
            
        } else if (e instanceof OutOfMemoryError) {
            category = ProcessingError.ErrorCategory.MEMORY_ERROR;
            recoverable = false;
            solution = "Increase heap size or process smaller batches";
            
        } else if (e.getMessage().contains("timeout")) {
            category = ProcessingError.ErrorCategory.TIMEOUT;
            recoverable = true;
            solution = "Increase timeout or check network connectivity";
            
        } else if (e.getMessage().contains("format")) {
            category = ProcessingError.ErrorCategory.FORMAT_ERROR;
            recoverable = false;
            solution = "Check document format and corruption";
            
        } else {
            category = ProcessingError.ErrorCategory.UNKNOWN;
            recoverable = false;
            solution = "Review error logs and contact support";
        }
        
        return ProcessingError.builder()
            .filename(filename)
            .exception(e.getClass().getSimpleName())
            .message(e.getMessage())
            .category(category)
            .recoverable(recoverable)
            .solution(solution)
            .timestamp(new Date())
            .build();
    }
    
    /**
     * Log structured error
     */
    public void logError(ProcessingError error) {
        
        log.error("Document processing error - File: {}, Category: {}, Message: {}",
            error.getFilename(),
            error.getCategory(),
            error.getMessage());
        
        if (error.isRecoverable()) {
            log.info("Error is recoverable. Suggested solution: {}", 
                error.getSolution());
        } else {
            log.error("Error is not recoverable. Action needed: {}", 
                error.getSolution());
        }
    }
    
    @FunctionalInterface
    public interface RetryableOperation<T> {
        T execute() throws Exception;
    }
    
    @Data
    @lombok.Builder
    public static class ProcessingError {
        private String filename;
        private String exception;
        private String message;
        private ErrorCategory category;
        private boolean recoverable;
        private String solution;
        private Date timestamp;
        
        public enum ErrorCategory {
            IO_ERROR,
            MEMORY_ERROR,
            TIMEOUT,
            FORMAT_ERROR,
            NETWORK_ERROR,
            PERMISSION_ERROR,
            UNKNOWN
        }
    }
}
```

---

## Performance Optimization

### Performance Monitoring

**PerformanceMonitor.java**:

```java
package com.example.pipeline.monitor;

import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.util.concurrent.TimeUnit;

@Slf4j
@Component
@RequiredArgsConstructor
public class PerformanceMonitor {
    
    private final MeterRegistry meterRegistry;
    
    /**
     * Record processing metrics
     */
    public void recordProcessing(
            String operation,
            long durationMs,
            int documentsProcessed,
            boolean success) {
        
        // Record duration
        Timer.builder("document.processing.duration")
            .tag("operation", operation)
            .tag("success", String.valueOf(success))
            .register(meterRegistry)
            .record(durationMs, TimeUnit.MILLISECONDS);
        
        // Record throughput
        meterRegistry.counter("document.processing.count",
            "operation", operation,
            "success", String.valueOf(success)
        ).increment(documentsProcessed);
        
        // Calculate and record throughput
        if (durationMs > 0) {
            double throughput = (documentsProcessed * 1000.0) / durationMs;
            meterRegistry.gauge("document.processing.throughput",
                List.of(Tag.of("operation", operation)),
                throughput
            );
        }
    }
    
    /**
     * Record file size metrics
     */
    public void recordFileSize(String mimeType, long sizeBytes) {
        
        meterRegistry.summary("document.size.bytes",
            "mime_type", mimeType
        ).record(sizeBytes);
    }
}
```

---

## Production Deployment

### REST Controller

**DocumentProcessingController.java**:

```java
package com.example.pipeline.controller;

import com.example.pipeline.service.*;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.*;
import java.util.concurrent.CompletableFuture;

@Slf4j
@RestController
@RequestMapping("/api/v1/documents")
@RequiredArgsConstructor
public class DocumentProcessingController {
    
    private final DocumentProcessingPipeline pipeline;
    private final DocumentProcessingConfig.DocumentProcessingProperties properties;
    
    private final Path tempDir = Paths.get(System.getProperty("java.io.tmpdir"));
    
    /**
     * Upload and process single document
     */
    @PostMapping("/upload")
    public ResponseEntity<?> uploadDocument(
            @RequestParam("file") MultipartFile file,
            @RequestParam(required = false) Map<String, String> options) {
        
        log.info("Received document upload: {}", file.getOriginalFilename());
        
        try {
            // Validate file
            validateFile(file);
            
            // Save temporarily
            Path tempFile = saveTemporarily(file);
            Resource resource = new UrlResource(tempFile.toUri());
            
            // Process
            ProcessingOptions processingOptions = buildOptions(options);
            DocumentProcessingPipeline.ProcessingResult result = 
                pipeline.processDocument(resource, processingOptions);
            
            // Cleanup
            Files.deleteIfExists(tempFile);
            
            return ResponseEntity.ok(result);
            
        } catch (Exception e) {
            log.error("Upload processing failed", e);
            return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(Map.of("error", e.getMessage()));
        }
    }
    
    /**
     * Batch upload and process
     */
    @PostMapping("/batch")
    public CompletableFuture<ResponseEntity<?>> batchUpload(
            @RequestParam("files") MultipartFile[] files) {
        
        log.info("Received batch upload: {} files", files.length);
        
        try {
            List<Resource> resources = new ArrayList<>();
            
            for (MultipartFile file : files) {
                validateFile(file);
                Path tempFile = saveTemporarily(file);
                resources.add(new UrlResource(tempFile.toUri()));
            }
            
            return pipeline.processBatch(
                resources,
                ProcessingOptions.defaults()
            ).thenApply(ResponseEntity::ok);
            
        } catch (Exception e) {
            log.error("Batch upload failed", e);
            return CompletableFuture.completedFuture(
                ResponseEntity
                    .status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body(Map.of("error", e.getMessage()))
            );
        }
    }
    
    private void validateFile(MultipartFile file) throws IOException {
        
        if (file.isEmpty()) {
            throw new IOException("File is empty");
        }
        
        if (file.getSize() > properties.getMaxFileSize()) {
            throw new IOException("File too large: " + file.getSize());
        }
        
        String contentType = file.getContentType();
        if (!properties.getAllowedMimeTypes().contains(contentType)) {
            throw new IOException("Unsupported file type: " + contentType);
        }
    }
    
    private Path saveTemporarily(MultipartFile file) throws IOException {
        
        String filename = UUID.randomUUID().toString() + "_" + 
            file.getOriginalFilename();
        
        Path path = tempDir.resolve(filename);
        file.transferTo(path);
        
        return path;
    }
    
    private ProcessingOptions buildOptions(Map<String, String> options) {
        // Build processing options from request parameters
        return ProcessingOptions.defaults();
    }
}
```

---

## Conclusion

### Key Takeaways

**Document Processing Best Practices**:

```
1. Multi-Format Support
   ✓ Use Apache Tika for universal parsing
   ✓ Implement format-specific loaders for better quality
   ✓ Handle edge cases and corrupted files

2. Intelligent Chunking
   ✓ Preserve semantic boundaries
   ✓ Add overlap for context continuity
   ✓ Adjust chunk size based on content type

3. Robust Error Handling
   ✓ Retry with exponential backoff
   ✓ Categorize and log errors
   ✓ Provide actionable error messages

4. Performance Optimization
   ✓ Batch processing where possible
   ✓ Cache embeddings
   ✓ Parallel processing for large batches

5. Production Readiness
   ✓ Comprehensive monitoring
   ✓ Rate limiting and validation
   ✓ Async processing for large files
```

### Performance Metrics

**Typical Processing Times**:

```
Document Type    | Size    | Processing Time
─────────────────┼─────────┼─────────────────
PDF (10 pages)   | 500KB   | 5-10 seconds
Word (20 pages)  | 200KB   | 3-8 seconds
Excel (5 sheets) | 100KB   | 2-5 seconds
PowerPoint       | 2MB     | 8-15 seconds
Image (OCR)      | 5MB     | 20-30 seconds

Factors affecting speed:
- File size and complexity
- OCR requirements
- Embedding generation
- Network latency
- Server resources
```

### Next Steps

1. **Week 1**: Implement basic loaders
2. **Week 2**: Add chunking and cleaning
3. **Week 3**: Integrate embeddings
4. **Week 4**: Production deployment
5. **Ongoing**: Monitor and optimize

### Resources

- [Apache Tika Documentation](https://tika.apache.org/)
- [Apache PDFBox](https://pdfbox.apache.org/)
- [Apache POI](https://poi.apache.org/)
- [Spring AI Docs](https://docs.spring.io/spring-ai/)
- [Tesseract OCR](https://github.com/tesseract-ocr/tesseract)

This comprehensive guide provides everything needed to build a production-ready document processing pipeline with Spring AI!
