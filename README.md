# Posts Processing Workflow

> An automated n8n workflow for fetching, processing, and storing blog posts with advanced error handling and data quality management.

[![n8n](https://img.shields.io/badge/n8n-workflow-FF6D5A?style=flat-square)](https://n8n.io)
[![API](https://img.shields.io/badge/API-JSONPlaceholder-blue?style=flat-square)](https://jsonplaceholder.typicode.com)
[![Storage](https://img.shields.io/badge/storage-Google%20Sheets-34A853?style=flat-square)](https://sheets.google.com)

## 🚀 Overview

This n8n workflow automatically processes blog posts from the JSONPlaceholder API, performing data transformation, quality validation, and storing results in Google Sheets. Built with robust error handling and comprehensive data processing capabilities.

## 📋 Features

- **Automated Data Pipeline**: ETL process for blog posts
- **Advanced Error Handling**: Multi-level retry logic and graceful failures
- **Data Quality Management**: Comprehensive validation and default values
- **Text Processing**: Title case conversion, keyword extraction, content analysis
- **Batch Processing**: Memory-efficient sequential processing
- **Audit Trail**: Complete processing history and metadata tracking

## 🏗️ Architecture

```mermaid
graph LR
    A[Manual Trigger] --> B[Initialize Posts]
    B --> C[Split Posts]
    C --> D[Fetch Post]
    D --> E[Process Data]
    E --> F[Save to Sheets]
    F --> C
    style A fill:#ff6d5a
    style F fill:#34a853
```

## 📊 Workflow Components

### 1. **Manual Trigger**
- **Type**: `n8n-nodes-base.manualTrigger`
- **Purpose**: Workflow initiation point

### 2. **Initialize Posts**
- **Type**: `n8n-nodes-base.code`
- **Function**: Generate post IDs (1-10)
- **Output**: Array of `{post_id: number}` objects

### 3. **Split Posts**
- **Type**: `n8n-nodes-base.splitInBatches`
- **Function**: Process posts individually
- **Pattern**: Producer-Consumer with loop control

### 4. **Fetch Post**
- **Type**: `n8n-nodes-base.httpRequest`
- **API**: `https://jsonplaceholder.typicode.com/posts/{id}`
- **Error Handling**: 2 retry attempts, continue on failure
- **Response Format**:
```json
{
  "userId": 1,
  "id": 1,
  "title": "Post title",
  "body": "Post content..."
}
```

### 5. **Process Data**
- **Type**: `n8n-nodes-base.code`
- **Function**: Advanced data transformation and validation

#### Data Processing Features:
- **Default Value Management**: Handles missing/null fields
- **Text Normalization**: Title case conversion and cleanup
- **Content Analysis**: Word count, keyword extraction, common words
- **Categorization**: Content-based classification (short/medium/long)
- **Metadata Generation**: Processing steps, timestamps, quality flags

#### Output Schema:
```javascript
{
  record_id: "POST_001",           // Unique identifier
  title: "Processed Title",        // Cleaned and formatted
  body: "Truncated content...",    // Limited to 100 chars
  word_count: 45,                  // Content analysis
  category: "medium",              // Auto-categorization
  keywords: "keyword1, keyword2",  // Extracted keywords
  original_id: 1,                  // Source post ID
  user_id: 1,                      // Author ID
  processed_date: "2025-06-08",    // Processing timestamp
  status: "processed",             // Processing status
  tags: "processed, latin",        // Auto-generated tags
  processing_steps: "title_cleaned, body_truncated...",
  title_length: 25,                // Metadata
  body_length: 89,                 // Metadata
  has_long_title: false,           // Quality flag
  common_words: "word1, word2",    // Frequency analysis
  has_missing_data: false,         // Quality indicator
  missing_fields: "none"           // Data quality report
}
```

### 6. **Save to Google Sheets**
- **Type**: `n8n-nodes-base.googleSheets`
- **Operation**: `appendOrUpdate`
- **Matching Column**: `record_id`
- **Error Strategy**: Continue on failure

## ⚙️ Error Handling Strategy

### HTTP Request Level
```javascript
{
  "retryOnFail": true,
  "maxTries": 2,
  "onError": "continueRegularOutput"
}
```

### Data Processing Level
- **Null Safety**: All fields protected with defaults
- **Type Validation**: String and number type checking
- **Unique ID Generation**: Timestamp + random for missing IDs

### Database Level
- **Conflict Resolution**: Update existing records by `record_id`
- **Schema Flexibility**: Dynamic column mapping
- **Transaction Safety**: Individual row processing

## 🔧 Setup Instructions

### Prerequisites
- n8n instance (cloud or self-hosted)
- Google Sheets API access
- Internet connection for JSONPlaceholder API

### Installation Steps

1. **Import Workflow**
   ```bash
   # Copy the workflow.json file to your n8n instance
   # Or import via n8n UI: Settings > Import from file
   ```

2. **Configure Google Sheets Credentials**
   - Go to n8n Credentials
   - Add "Google Sheets OAuth2 API" credential
   - Authorize your Google account

3. **Update Google Sheets Configuration**
   ```javascript
   // Replace with your spreadsheet ID
   "documentId": "YOUR_SPREADSHEET_ID_HERE"
   ```

4. **Test the Workflow**
   - Click "Test workflow" in n8n
   - Monitor execution in the workflow editor

### Environment Variables
```env
# Optional: Customize post range
POST_START_ID=1
POST_END_ID=10

# Optional: API timeout settings
HTTP_TIMEOUT=30000
```

## 📈 Data Flow

### Successful Execution Path
```
Manual Trigger → Initialize Posts → Split Batch → 
HTTP Request (200) → Process Data → Google Sheets (Success) → 
Loop Continue → Complete
```

### Error Recovery Paths
```
HTTP Failure → Retry (2x) → Default Values → Continue
Google Sheets Failure → Error Output → Loop Continue
```

## 🛠️ Customization Options

### Modify Post Range
```javascript
// In "Initialize Posts" node
const postIds = [1, 2, 3, 4, 5]; // Customize range
```

### Adjust Processing Logic
```javascript
// In "Process Data" node - modify categorization
if (wordCount < 10) category = 'very_short';
else if (wordCount < 30) category = 'short';
// ... customize as needed
```

### Change Output Format
```javascript
// Modify the output object structure
const output = {
  // Add/remove fields as needed
  custom_field: "custom_value"
};
```

## 📊 Monitoring & Debugging

### Execution Logs
- Check n8n execution history
- Monitor individual node outputs
- Review error logs for failures

### Data Quality Checks
```sql
-- Example Google Sheets queries
=COUNTIF(status, "processed")        -- Successful records
=COUNTIF(has_missing_data, TRUE)     -- Records with missing data
=AVERAGE(word_count)                 -- Average content length
```

### Performance Metrics
- **Execution Time**: ~2-5 seconds per post
- **Memory Usage**: Minimal (batch processing)
- **API Rate Limits**: Respects JSONPlaceholder limits

## 🚨 Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|--------|----------|
| HTTP timeout | Network/API issues | Check retry settings |
| Google Sheets auth | Expired credentials | Refresh OAuth token |
| Missing data | API response changes | Review default values |
| Loop stuck | Split batch error | Check batch configuration |

### Debug Mode
```javascript
// Add to Process Data node for debugging
console.log("Processing post:", post);
console.log("Generated output:", output);
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Test your changes thoroughly
4. Submit a pull request with detailed description

### Development Guidelines
- Follow n8n best practices
- Add comprehensive error handling
- Document new features
- Test with various data scenarios

## 🔗 Related Resources

- [n8n Documentation](https://docs.n8n.io/)
- [JSONPlaceholder API](https://jsonplaceholder.typicode.com/)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [n8n Community](https://community.n8n.io/)

---

**⭐ If this workflow helped you, please give it a star!**

---

*Built with ❤️ using n8n*