# Indian News Scraper Automation with n8n

A comprehensive news aggregation system built with n8n that automatically collects, processes, and stores news articles from major Indian news sources into a PostgreSQL database. Covers 85+ RSS feeds from Hindustan Times, NDTV, and Times of India with detailed categorization.

## 🚀 Features

- **Multi-source RSS aggregation**: 85+ RSS feeds from 3 major Indian news sources
- **Comprehensive categorization**: City-wise, topic-wise, and regional news coverage
- **Automated data processing**: XML to JSON conversion with robust error handling
- **Smart data extraction**: Extracts images, metadata, and content hashing
- **Database storage**: PostgreSQL with duplicate prevention and optimized schema
- **Scheduled automation**: Runs every 10 minutes for fresh content
- **Batched processing**: HTTP requests in batches of 5 with 2-second intervals

## 🏗️ Architecture

```
Schedule Trigger → RSS Feed Config → HTTP Request (Batched) → XML Parser → 
Item Extraction → Data Normalization → Database Mapping → PostgreSQL Storage
```

### Workflow Components

1. **Schedule Trigger**: Runs every 10 minutes automatically
2. **RSS Feed Configuration**: 85+ pre-configured feeds with metadata
3. **HTTP Request Node**: Fetches RSS content with proper headers and batching
4. **XML Parser**: Converts XML response to JSON format
5. **Item Extractor**: Processes all feeds and extracts individual articles
6. **Data Normalizer**: Standardizes article data structure
7. **Database Mapper**: Generates IDs, hashes, and formats for PostgreSQL
8. **PostgreSQL Node**: Stores articles with conflict resolution

## 📋 Prerequisites

- **n8n**: Self-hosted or cloud instance
- **PostgreSQL**: Database server (local or remote)
- **Node.js**: Required for n8n (if self-hosting)

## ⚙️ Setup

### 2. Database Setup

Create your PostgreSQL database and articles table:

```sql
CREATE DATABASE news_scraper;

CREATE TABLE news_articles (
    id BIGINT PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    link VARCHAR(1000) UNIQUE NOT NULL,
    image VARCHAR(1000),
    description TEXT,
    source VARCHAR(100) NOT NULL,
    source_domain VARCHAR(200) NOT NULL,
    published_time VARCHAR(100),
    author VARCHAR(200),
    category VARCHAR(100),
    scraped_at TIMESTAMP,
    content_hash VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for better performance
CREATE INDEX idx_news_articles_published_time ON news_articles(published_time);
CREATE INDEX idx_news_articles_source ON news_articles(source);
CREATE INDEX idx_news_articles_category ON news_articles(category);
CREATE INDEX idx_news_articles_content_hash ON news_articles(content_hash);
CREATE INDEX idx_news_articles_scraped_at ON news_articles(scraped_at);
```

### 3. n8n Workflow Import

1. **Import the workflow**: Copy the provided `workflow.json` into your n8n instance
2. **Configure PostgreSQL credentials**: Set up your database connection
3. **Activate the workflow**: Enable the schedule trigger

### 4. RSS Feed Sources

The workflow monitors 85+ RSS feeds from major Indian news sources:

#### Hindustan Times (31 feeds)
- **Categories**: World News, Education, Entertainment, Sports, Cricket, Lifestyle
- **City Coverage**: Kolkata, Lucknow, Delhi, Mumbai, Bangalore, Chennai, etc.
- **Special Sections**: Astrology, Infographics, HT Insight, Trending

#### NDTV (13 feeds)
- **Categories**: Top Stories, Latest News, India/World News, Business, Sports
- **Specialized**: Technology (Gadgets360), Food (NDTV Cooks), Movies
- **Regional**: South India coverage

#### Times of India (41 feeds)
- **Categories**: Business, Science, Environment, Technology, Education
- **City Coverage**: 25+ major Indian cities including Mumbai, Delhi, Bangalore
- **Comprehensive**: Entertainment, Lifestyle, Astrology sections

## 🔧 Configuration

### Schedule Settings

- **Frequency**: Every 10 minutes (configurable via Schedule Trigger node)
- **Batching**: HTTP requests processed in batches of 5 feeds
- **Batch Interval**: 2-second delay between batches to avoid rate limiting

### HTTP Request Headers

The workflow uses browser-like headers for better compatibility:

```json
{
  "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
  "Accept": "application/rss+xml, application/xml, text/xml, */*",
  "Accept-Language": "en-US,en;q=0.9",
  "Referer": "https://www.hindustantimes.com/",
  "Cache-Control": "no-cache"
}
```

### Feed Configuration Structure

Each RSS feed includes metadata for better categorization:

```json
{
  "url": "https://www.hindustantimes.com/feeds/rss/world-news/rssfeed.xml",
  "category": "World News",
  "subcategory": "General",
  "city": null,
  "country": "India"
}
```

### Data Processing Pipeline

#### Stage 1: Item Extraction
- Processes ALL 85 feeds simultaneously
- Extracts individual articles from each RSS feed
- Preserves feed metadata for categorization

#### Stage 2: Data Normalization
- Standardizes article structure
- Extracts images from `media:content` and other sources
- Handles various RSS format variations

#### Stage 3: Database Mapping
- Generates unique IDs using timestamp + random suffix
- Creates content hashes for duplicate detection
- Extracts source and domain information from URLs
- Adds creation and update timestamps

## 📊 Database Schema

| Column | Type | Description |
|--------|------|-------------|
| `id` | BIGINT | Primary key (generated using timestamp + random) |
| `title` | VARCHAR(500) | Article headline |
| `link` | VARCHAR(1000) | Article URL (unique constraint) |
| `image` | VARCHAR(1000) | Article image URL |
| `description` | TEXT | Article summary/excerpt |
| `source` | VARCHAR(100) | News source (extracted from URL) |
| `source_domain` | VARCHAR(200) | Full domain name |
| `published_time` | VARCHAR(100) | Publication timestamp from RSS |
| `author` | VARCHAR(200) | Article author |
| `category` | VARCHAR(100) | Content category |
| `scraped_at` | TIMESTAMP | When article was scraped |
| `content_hash` | VARCHAR(50) | Hash for duplicate detection |
| `created_at` | TIMESTAMP | Record creation time |
| `updated_at` | TIMESTAMP | Last update time |

### Key Features
- **Unique constraint** on `link` prevents duplicates
- **Content hashing** for additional duplicate detection
- **Skip on conflict** enabled in PostgreSQL node
- **Optimized indexes** for common query patterns

## 🚀 Usage

### Running the Automation

1. **Automatic**: Workflow runs on schedule
2. **Manual**: Execute from n8n interface
3. **Webhook**: Trigger via HTTP endpoint (optional)

### Monitoring

Monitor your scraper through:
- n8n execution history
- PostgreSQL query logs
- Database record counts
- Error notifications (configure in n8n)

## 🛠️ Maintenance

### Regular Tasks

- **Database cleanup**: Remove old articles periodically
- **Feed validation**: Check RSS feed availability
- **Performance monitoring**: Track execution times
- **Storage management**: Monitor database size

## 🛠️ Maintenance & Monitoring

### Regular Tasks

- **Database cleanup**: Remove articles older than 30 days
- **Feed health monitoring**: Check for failed RSS requests
- **Performance tracking**: Monitor execution times and batch processing
- **Storage management**: Archive old data and manage disk space

### Troubleshooting

Common issues and solutions:

1. **Duplicate articles**: 
   - Check unique constraints on `link` field
   - Verify content hash generation is working
   - Review "skip on conflict" setting in PostgreSQL node

2. **XML parsing errors**: 
   - Monitor HTTP Request node for failed feeds
   - Check RSS feed availability and format changes
   - Review error logs in n8n execution history

3. **Database connection issues**: 
   - Verify PostgreSQL credentials in n8n
   - Check database server connectivity
   - Monitor connection pool usage

4. **Memory/performance issues**: 
   - Adjust batch size in HTTP Request node (currently 5)
   - Increase batch interval if rate limiting occurs
   - Monitor execution times for individual feeds

## 📈 Performance & Scaling

### Current Performance Metrics

- **Processing capacity**: 85 RSS feeds every 10 minutes
- **Batch processing**: 5 feeds per batch with 2-second intervals
- **Average articles per run**: 500-1000 articles (varies by feed activity)
- **Execution time**: ~2-3 minutes per complete cycle

### Optimization Strategies

1. **Database Optimization**
   - Regular VACUUM and ANALYZE operations
   - Proper indexing on frequently queried columns
   - Partitioning by date for large datasets

2. **Feed Processing**
   - Monitor individual feed response times
   - Adjust batch sizes based on server capacity
   - Implement retry logic for failed requests

3. **Memory Management**
   - Process feeds in smaller batches if memory issues occur
   - Clear intermediate data between processing stages

### Adding New RSS Sources

To add new feeds, update the "Edit Fields" node in n8n:

```json
{
  "url": "https://new-source.com/rss/feed.xml",
  "category": "New Category",
  "subcategory": "Subcategory Name",
  "city": "City Name (if applicable)",
  "country": "Country"
}
```
