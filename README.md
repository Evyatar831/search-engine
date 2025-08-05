# Web Crawler Search Engine

A distributed web crawler built with Spring Boot that crawls websites, indexes content in Elasticsearch, and uses Kafka for distributed processing.

## Architecture Overview

This application implements a scalable web crawler that:
- Accepts crawling requests via REST API
- Distributes crawling tasks using Apache Kafka
- Stores crawl status and visited URLs in Redis
- Indexes page content in Elasticsearch
- Provides real-time crawl status monitoring

## Tech Stack

- **Backend**: Spring Boot 2.5.2, Java 11
- **Message Queue**: Apache Kafka
- **Cache/Storage**: Redis
- **Search Engine**: Elasticsearch
- **Web Scraping**: JSoup
- **Containerization**: Docker Compose

## Features

- **Distributed Crawling**: Uses Kafka to distribute crawling tasks across multiple consumers
- **Configurable Limits**: Set maximum crawl distance, number of URLs, and time limits
- **Real-time Status**: Monitor crawl progress and statistics
- **Content Indexing**: Automatically indexes crawled content in Elasticsearch
- **Duplicate Prevention**: Uses Redis to track visited URLs and prevent re-crawling

## Quick Start

### Prerequisites

- Java 11+
- Maven
- Docker and Docker Compose

### 1. Start Infrastructure Services

```bash
docker-compose up -d
```

This starts:
- Kafka (port 9092)
- Zookeeper (port 2181)  
- Redis (port 6379)

### 2. Configure Elasticsearch (Optional)

Update `src/main/resources/application.properties`:

```properties
elasticsearch.base.url=your-elasticsearch-url
elasticsearch.key=your-elasticsearch-api-key
elasticsearch.index=your-index-name
```

### 3. Run the Application

```bash
mvn spring-boot:run
```

The application will start on port 8080.

### 4. Access Swagger UI

Visit: `http://localhost:8080/swagger-ui.html`

## API Endpoints

### Start a Crawl

```http
POST /api/crawl
Content-Type: application/json

{
  "url": "https://example.com",
  "maxDistance": 2,
  "maxSeconds": 300,
  "maxUrls": 100
}
```

**Response**: Returns a unique crawl ID (e.g., "AbC123")

### Check Crawl Status

```http
GET /api/crawl/{crawlId}
```

**Response**:
```json
{
  "distance": 1,
  "startTime": "2024-01-15 10:30:00",
  "lastModified": "2024-01-15 10:35:00",
  "stopReason": null,
  "numPages": 25
}
```

### Send Direct Kafka Message

```http
POST /api/sendKafka
Content-Type: application/json

{
  "url": "https://example.com",
  "maxDistance": 1,
  "maxSeconds": 60,
  "maxUrls": 10
}
```

## Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `maxDistance` | Maximum link depth to crawl | - |
| `maxSeconds` | Maximum crawl duration | - |
| `maxUrls` | Maximum number of URLs to crawl | - |
| `url` | Starting URL (auto-prepends https://) | - |

## Stop Reasons

The crawler stops when it hits any of these limits:

- `maxUrls`: Reached maximum number of URLs
- `maxDistance`: Reached maximum crawl depth  
- `timeout`: Exceeded maximum time limit

## Data Models

### CrawlerRequest
```java
{
  "url": "string",
  "maxDistance": "integer", 
  "maxSeconds": "integer",
  "maxUrls": "integer"
}
```

### CrawlStatus
```java
{
  "distance": "integer",
  "startTime": "datetime",
  "lastModified": "datetime", 
  "stopReason": "enum",
  "numPages": "long"
}
```

## How It Works

1. **Request Received**: POST to `/api/crawl` creates a crawl job
2. **Kafka Message**: Initial crawl record sent to Kafka topic
3. **Consumer Processing**: Kafka consumer picks up the message
4. **Page Crawling**: JSoup fetches and parses the webpage
5. **Link Extraction**: Extracts all links from the same domain
6. **Content Indexing**: Stores page content in Elasticsearch
7. **Queue New URLs**: Sends discovered URLs back to Kafka
8. **Status Updates**: Updates crawl progress in Redis
9. **Termination**: Stops when limits are reached

## Development

### Project Structure

```
src/main/java/com/handson/searchengine/
├── config/          # Configuration classes
├── controller/      # REST API controllers  
├── crawler/         # Core crawling logic
├── kafka/          # Kafka producer/consumer
├── model/          # Data models
└── util/           # Utility classes
```

### Key Components

- **AppController**: REST API endpoints
- **Crawler**: Core crawling and coordination logic
- **Producer/Consumer**: Kafka message handling
- **ElasticSearch**: Content indexing utility
- **RedisConfig**: Redis connection setup

### Running Tests

```bash
mvn test
```

## Docker Services

The `docker-compose.yml` includes:

- **Zookeeper**: Kafka coordination
- **Kafka**: Message queue for distributed processing
- **Redis**: Fast storage for crawl state and visited URLs

## Monitoring

Monitor your crawl progress by:
- Checking crawl status via GET `/api/crawl/{crawlId}`
- Viewing application logs for detailed crawling information
- Monitoring Kafka topics and Redis keys

## Troubleshooting

### Common Issues

1. **Kafka Connection**: Ensure Kafka is running on port 9092
2. **Redis Connection**: Verify Redis is accessible on port 6379
3. **Elasticsearch**: Check if Elasticsearch credentials are configured
4. **Memory**: Large crawls may require increased JVM heap size

### Logs

The application provides detailed logging for:
- URL crawling progress
- Kafka message processing
- Elasticsearch indexing
- Error handling

## Contributing

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality  
4. Submit a pull request

## License

This project is provided as-is for educational and development purposes.
