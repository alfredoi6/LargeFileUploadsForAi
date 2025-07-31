# Large File Upload Requirements & Solutions

## Project Overview
Web application for uploading large PDF files (10-35+ MB) to be processed by machine learning models. PDFs contain text, tables, and images that need to be extracted and analyzed.

## Current Issues
- **File Size**: Large PDFs (10,721 KB - 35,569 KB)
- **Upload Failures**: Client-side chunking implementation sometimes stops
- **Platform**: Azure App Service hosting
- **Processing**: ML model processing of uploaded PDFs

## Recommended Solutions

### 1. Robust Client-Side Chunking Strategy

#### Implementation Approach:
- **Chunk Size**: 1-5 MB chunks (optimal for Azure App Service)
- **Retry Logic**: Exponential backoff with max 3 retries per chunk
- **Progress Tracking**: Real-time upload progress with resume capability
- **Error Handling**: Graceful failure recovery and user feedback

#### Key Features:
```javascript
// Recommended chunk configuration
const CHUNK_SIZE = 2 * 1024 * 1024; // 2MB chunks
const MAX_RETRIES = 3;
const RETRY_DELAY = 1000; // Start with 1 second
```

### 2. Server-Side Implementation (Azure App Service)

#### Configuration Requirements:
- **Request Timeout**: Increase to 300+ seconds
- **Max Request Size**: Configure for largest expected file
- **Temp Storage**: Use Azure Blob Storage for chunk assembly
- **Connection Limits**: Optimize for concurrent uploads

#### Recommended Architecture:
1. **Upload Controller**: `UploadController` with chunked upload actions
2. **Chunk Endpoint**: `POST /Upload/Chunk` for individual chunks
3. **Assembly Endpoint**: `POST /Upload/Complete` to finalize file
4. **Status Endpoint**: `GET /Upload/Status/{uploadId}` for progress tracking
5. **Background Service**: `IHostedService` for cleanup of incomplete uploads
6. **Services**: `IFileUploadService`, `IBlobStorageService` for business logic

### 3. Azure-Specific Optimizations

#### App Service Settings:
```json
{
  "WEBSITE_LOAD_CERTIFICATES": "*",
  "SCM_COMMAND_IDLE_TIMEOUT": "3600",
  "WEBSITES_ENABLE_APP_SERVICE_STORAGE": "true"
}
```

#### Blob Storage Integration:
- Use Azure Blob Storage for temporary chunk storage
- Implement SAS tokens for secure direct uploads
- Configure lifecycle policies for cleanup

### 4. Client-Side Best Practices

#### Upload Flow:
1. **File Validation**: Check file type, size, and integrity
2. **Chunk Generation**: Split file into optimal-sized chunks
3. **Parallel Upload**: Upload multiple chunks concurrently (limit: 3-5)
4. **Progress Monitoring**: Real-time progress updates
5. **Error Recovery**: Automatic retry with exponential backoff
6. **Completion**: Trigger server-side assembly

#### Recommended JavaScript Upload Libraries:

##### **Option 1: Uppy.js** (Most Feature-Rich)
- **Pros**: Built-in chunking, resumable uploads, progress tracking, drag-drop
- **Integration**: Works with vanilla JS, has MVC-friendly API
- **Size**: ~200KB (modular, can include only needed parts)
- **CDN**: Available via CDN or npm

##### **Option 2: Resumable.js** (Lightweight & Robust)
- **Pros**: Simple API, excellent chunking support, fault tolerance
- **Integration**: Pure JavaScript, easy MVC integration
- **Size**: ~15KB minified
- **GitHub**: https://github.com/23/resumable.js

##### **Option 3: Plupload** (Mature & Stable)
- **Pros**: Cross-browser support, chunking, multiple runtimes
- **Integration**: Works with any backend including ASP.NET MVC
- **Size**: ~100KB
- **Features**: Drag-drop, image resizing, filters

##### **Option 4: Fine Uploader** (Enterprise-Grade)
- **Pros**: Comprehensive features, excellent documentation
- **Integration**: Framework agnostic, works well with MVC
- **Size**: ~150KB
- **Features**: Chunking, retry logic, progress tracking

##### **Option 5: Custom Implementation with Axios**
- **Pros**: Maximum control, lightweight, tailored to your needs
- **Integration**: Perfect for ASP.NET MVC
- **Size**: ~13KB (Axios only)
- **Flexibility**: Complete customization of chunking logic

### 5. Monitoring & Debugging

#### Logging Strategy:
- Client-side: Upload progress, errors, retry attempts
- Server-side: Chunk receipt, assembly status, processing time
- Azure: Application Insights for performance monitoring

#### Key Metrics:
- Upload success rate
- Average upload time
- Chunk failure rate
- Server response times

### 6. Implementation Priority

#### Phase 1 (Critical):
1. Implement robust chunking with retry logic
2. Configure Azure App Service settings
3. Add comprehensive error handling

#### Phase 2 (Enhancement):
1. Add progress tracking and resume capability
2. Implement Azure Blob Storage integration
3. Add monitoring and analytics

#### Phase 3 (Optimization):
1. Performance tuning based on metrics
2. Advanced features (drag-drop, multiple files)
3. User experience improvements

## Technical Stack Recommendations

### Frontend:
- **Views**: ASP.NET MVC Razor Views
- **JavaScript**: Vanilla JavaScript or TypeScript
- **HTTP Client**: Axios library (via CDN or npm)
- **Upload Library**: Custom implementation with chunking logic
- **UI Framework**: Bootstrap or similar CSS framework
- **Progress Tracking**: HTML5 progress elements with JavaScript updates

### Backend:
- **Framework**: ASP.NET MVC Core
- **Runtime**: .NET Core/.NET 6+
- **Storage**: Azure Blob Storage
- **Database**: Azure SQL Database for metadata
- **Monitoring**: Application Insights
- **File Handling**: IFormFile with streaming support
- **Dependency Injection**: Built-in DI container

### Infrastructure:
- **Hosting**: Azure App Service (Premium tier recommended)
- **CDN**: Azure CDN for static assets
- **Load Balancing**: Azure Application Gateway if needed

## Expected Outcomes
- **Reliability**: 99%+ upload success rate
- **Performance**: Sub-30 second uploads for 35MB files
- **User Experience**: Real-time progress with error recovery
- **Scalability**: Support for concurrent users and larger files