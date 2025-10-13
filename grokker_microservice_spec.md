# Grokker Commit Message Microservice Specification

## Introduction

This document specifies the requirements and design for a microservice that generates intelligent commit messages using the Grokker tool. The service will analyze code changes and produce conventional commit messages using AI, serving as a bridge between web-based editors and the Grokker functionality.

## Approach Comparison

| Factor | Microservice Approach | Browser-Based (WASM) Approach |
|--------|------------------------|-------------------------------|
| **Development Complexity** | Medium - Standard server development | High - Requires WASM compilation, filesystem shims, browser compatibility work |
| **Performance** | High - Native Go execution on server | Medium - WASM has overhead, slower initialization |
| **User Experience** | Slight delay due to network calls | Potential UI freezing during processing, but no network latency |
| **Initial Load Time** | Fast - Server handles the heavy lifting | Slower - Large WASM bundle (3-10MB) needs to be downloaded |
| **Security** | Strong - API keys stored server-side | Weaker - API keys exposed to client unless proxied |
| **Maintenance** | Easier - Single deployment to update | Harder - Updates need to be recompiled and distributed to all clients |
| **Deployment** | Requires server infrastructure | No server required (fully decentralized) |
| **Operational Cost** | $5-20/month for hosting + API costs | Only API costs (no hosting needed) |
| **Privacy** | Code sent to server for processing | Everything stays on client (better privacy) |
| **Offline Capability** | None - Requires network connection | Possible with locally-stored API credentials |
| **Cross-platform Support** | Excellent - Works on any device with web access | Good - Some older browsers lack WASM support |
| **Resource Usage** | Low client requirements, server handles processing | Higher client CPU/memory requirements |
| **Consistency** | Guaranteed same behavior for all users | May vary based on user's browser/device |
| **Integration with GitHub** | Direct server-to-server communication possible | Requires user's browser to make API calls (CORS issues) |
| **Scalability** | Can scale with server resources | Limited by client browser capabilities |

## Requirements

### Functional Requirements

1. **Commit Message Generation**:
   - Accept file content and generate meaningful commit messages
   - Support both single-file and multi-file changes
   - Produce commit messages in conventional commit format
   - Allow optional context parameters to improve message quality

2. **API Interface**:
   - Provide a RESTful API endpoint for commit message generation
   - Accept JSON payloads with file content and metadata
   - Return structured commit messages
   - Support both synchronous and asynchronous request patterns

3. **Authentication and Authorization**:
   - Validate API keys for service access
   - Support rate limiting per API key
   - Provide usage statistics per user/key

4. **Error Handling**:
   - Return meaningful error messages
   - Handle graceful degradation
   - Provide appropriate HTTP status codes

### Non-Functional Requirements

1. **Performance**:
   - Process commit generation in under 3 seconds for most files
   - Support concurrent requests
   - Optimize for minimal memory usage

2. **Scalability**:
   - Handle multiple simultaneous requests
   - Support horizontal scaling for increased load

3. **Security**:
   - Securely store API keys
   - Implement TLS for all connections
   - Apply proper input sanitization
   - Ensure no persistent storage of user code

4. **Maintainability**:
   - Provide monitoring endpoints (health, metrics)
   - Include comprehensive logging
   - Support configuration via environment variables

5. **Compatibility**:
   - Support cross-origin requests (CORS)
   - Handle various file formats and encodings

## API Specification

### Base URL
`https://[service-host]/api`

### Endpoints

#### 1. Generate Commit Message

```
POST /grokker/commit
```

**Headers**:
- `X-Grokker-API-Key`: [Required] Authentication key
- `Content-Type`: application/json

**Request Body**:
```json
{
  "content": "file content after changes",
  "previousContent": "file content before changes (optional)",
  "filePath": "path/to/file.js (optional)",
  "fileType": "javascript (optional)",
  "repoContext": "brief description of repository (optional)",
  "commitStyle": "conventional (default) | detailed | simple",
  "async": false
}
```

**Response (200 OK)**:
```json
{
  "commitMessage": "feat(component): add support for AI-generated commit messages",
  "details": "- Integrated OpenAI API for message generation\n- Added error handling for API failures\n- Implemented conventional commit format",
  "type": "feat",
  "scope": "component",
  "subject": "add support for AI-generated commit messages"
}
```

**Error Response (400 Bad Request)**:
```json
{
  "error": "Invalid request",
  "message": "Content field is required",
  "code": "INVALID_REQUEST"
}
```

**Error Response (401 Unauthorized)**:
```json
{
  "error": "Authentication failed",
  "message": "Invalid or missing API key",
  "code": "AUTH_ERROR"
}
```

**Error Response (429 Too Many Requests)**:
```json
{
  "error": "Rate limit exceeded",
  "message": "Try again in 60 seconds",
  "code": "RATE_LIMIT",
  "retryAfter": 60
}
```

#### 2. Service Health Check

```
GET /health
```

**Response (200 OK)**:
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime": 1234567
}
```

#### 3. API Key Management (Optional)

```
POST /keys
```

**Headers**:
- `Authorization`: Bearer [admin-token]
- `Content-Type`: application/json

**Request Body**:
```json
{
  "user": "user@example.com",
  "rateLimit": 100
}
```

**Response (201 Created)**:
```json
{
  "apiKey": "grk_123456789abcdef",
  "user": "user@example.com",
  "created": "2025-10-12T15:30:45Z",
  "rateLimit": 100
}
```

## Architecture

The microservice will be implemented in Go, leveraging the existing Grokker codebase while providing a web-friendly interface.

### Components

1. **Web Server**:
   - HTTP server with routing
   - Request validation and parsing
   - Response formatting
   - CORS handling

2. **Authentication Manager**:
   - API key validation
   - Rate limiting
   - Usage tracking

3. **Grokker Bridge**:
   - Integration with core Grokker functionality
   - Manages Grokker process execution
   - Handles result parsing

4. **Configuration Manager**:
   - Environment variable processing
   - Default configuration

5. **OpenAI Integration**:
   - Secure API key handling
   - Retry mechanism
   - Response processing

### Technology Stack

- **Language**: Go (to align with existing Grokker implementation)
- **Web Framework**: Gin or Echo (lightweight, performant)
- **Configuration**: Viper (flexible configuration)
- **Logging**: Zap (structured, performant logging)
- **Testing**: Go testing package with testify
- **Documentation**: Swagger/OpenAPI

## Implementation Details

### Directory Structure

```
/
├── cmd/
│   └── server/
│       └── main.go            # Application entry point
├── internal/
│   ├── api/                   # API handlers
│   │   ├── commit.go
│   │   ├── health.go
│   │   └── keys.go
│   ├── auth/                  # Authentication logic
│   │   ├── apikey.go
│   │   └── ratelimit.go
│   ├── config/                # Configuration
│   │   └── config.go
│   ├── grokker/               # Grokker integration
│   │   ├── bridge.go
│   │   └── processor.go
│   └── openai/                # OpenAI API integration
│       └── client.go
├── pkg/                       # Public packages
│   ├── models/                # Data models
│   └── utils/                 # Utility functions
├── scripts/                   # Build scripts
├── .env.example               # Example environment file
├── Dockerfile                 # Container definition
├── docker-compose.yml         # Local development setup
├── go.mod                     # Go modules
└── README.md                  # Project documentation
```

### Grokker Integration

The service will integrate with Grokker in one of two ways:

1. **Direct Library Integration**:
   - Import Grokker as a Go module
   - Call relevant functions directly
   - Advantages: Better performance, tighter integration
   - Challenges: Ensuring compatibility with Grokker's API

2. **Process Execution**:
   - Execute Grokker as a subprocess
   - Pass parameters and capture output
   - Advantages: Isolation, compatibility with CLI interface
   - Challenges: Process management, potential performance impact

The final approach will depend on the structure of the Grokker codebase and its modularity.

## Interface with githubCommitDialog.js

The microservice is designed to be a drop-in replacement for the placeholder functionality in the `githubCommitDialog.js` file. It will:

1. Respond to the existing API call pattern:
```javascript
fetch('/api/grokker/commit', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Grokker-API-Key': this.settings.grokkerApiKey
  },
  body: JSON.stringify({
    content: content
  })
})
```

2. Return the expected response format that the dialog can process:
```javascript
const data = await response.json();
return data.commitMessage;
```

3. Handle error cases in a way that the dialog can display appropriate messages to the user.

## Deployment Options

### 1. Docker Container

```dockerfile
FROM golang:1.21-alpine as builder
WORKDIR /app
COPY . .
RUN go build -o grokker-service ./cmd/server

FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/grokker-service .
EXPOSE 8080
CMD ["./grokker-service"]
```

### 2. Standalone Binary

```bash
go build -o grokker-service ./cmd/server
./grokker-service
```

### 3. Systemd Service

```ini
[Unit]
Description=Grokker Commit Message Service
After=network.target

[Service]
User=grokker
WorkingDirectory=/opt/grokker-service
ExecStart=/opt/grokker-service/grokker-service
Restart=on-failure
Environment=PORT=8080
Environment=OPENAI_API_KEY=your-key-here

[Install]
WantedBy=multi-user.target
```

## Configuration

The service will be configurable through environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | HTTP server port | 8080 |
| `HOST` | Host to bind to | 0.0.0.0 |
| `OPENAI_API_KEY` | OpenAI API key | (required) |
| `LOG_LEVEL` | Logging level | info |
| `ALLOWED_ORIGINS` | CORS allowed origins | * |
| `RATE_LIMIT` | Requests per hour per key | 100 |
| `ADMIN_TOKEN` | Token for admin operations | (required for admin APIs) |
| `TLS_CERT` | TLS certificate path | (optional) |
| `TLS_KEY` | TLS key path | (optional) |
| `GROKKER_PATH` | Path to Grokker binary | (required if using subprocess approach) |

## Security Considerations

1. **API Key Management**:
   - Store API keys securely (not in plain text)
   - Use proper key rotation practices
   - Implement key revocation

2. **Code Privacy**:
   - Do not persist submitted code
   - Process in memory only
   - Clear memory after processing

3. **Network Security**:
   - Implement TLS for all connections
   - Use secure headers
   - Apply appropriate CORS policies

4. **Dependency Management**:
   - Keep dependencies updated
   - Scan for vulnerabilities
   - Use vendoring for stability

5. **Rate Limiting and DoS Protection**:
   - Implement per-key and global rate limits
   - Add timeout for long-running operations
   - Monitor for abuse patterns

## Monitoring and Logging

1. **Health Metrics**:
   - CPU/Memory usage
   - Request count and latency
   - Error rates
   - API key usage

2. **Logging Strategy**:
   - Structured logs (JSON format)
   - Configurable verbosity
   - Request ID tracking
   - Sensitive data redaction

3. **Alerting**:
   - Error rate thresholds
   - Service availability
   - API key abuse detection

## Future Enhancements

1. **Multi-file Analysis**:
   - Support analyzing multiple files in a single request
   - Generate comprehensive commit messages

2. **Repository Context**:
   - Consider repository history for better messages
   - Support branch and PR specific messaging

3. **Custom Templates**:
   - Allow users to define commit message templates
   - Support organization-specific conventions

4. **Learning and Improvement**:
   - Track which messages are accepted vs. modified
   - Improve suggestion quality over time

5. **Webhook Integration**:
   - Direct integration with GitHub, GitLab, etc.
   - Automated commit message generation

## Conclusion

This specification outlines a robust, secure, and performant microservice for generating commit messages using Grokker. The service is designed to integrate seamlessly with the existing `githubCommitDialog.js` implementation while providing a foundation for broader adoption and enhancement.

By implementing this service in Go, we leverage the existing Grokker codebase while creating a web-friendly interface that can be deployed in various environments, from self-hosted servers to container platforms.
