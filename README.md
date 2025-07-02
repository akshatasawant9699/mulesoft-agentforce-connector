# 🤖 MuleSoft Agentforce Connector Integration

[![MuleSoft](https://img.shields.io/badge/MuleSoft-4.4.0-blue)](https://www.mulesoft.com/)
[![Salesforce](https://img.shields.io/badge/Salesforce-Agentforce-orange)](https://www.salesforce.com/products/agentforce/)
[![Java](https://img.shields.io/badge/Java-8%2B-red)](https://www.oracle.com/java/)
[![Maven](https://img.shields.io/badge/Maven-3.6%2B-green)](https://maven.apache.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A production-ready integration between **Salesforce Agentforce** and **MuleSoft** that enables developers to build intelligent, AI-powered conversational interfaces for enterprise applications.

## 🚀 Features

- **🔐 Secure OAuth 2.0 Integration** - Enterprise-grade authentication with externalized credentials
- **🤖 AI-Powered Conversations** - Natural language processing with Salesforce Agentforce
- **🔄 Session Management** - Stateful conversation handling with automatic cleanup
- **⚡ High Performance** - Connection pooling and optimized resource utilization
- **🛡️ Enterprise Security** - Data encryption, audit logging, and compliance ready
- **📊 Comprehensive Monitoring** - Structured logging and health check endpoints
- **🎯 Error Resilience** - Automatic retry logic and graceful failure handling
- **🔧 Configuration Management** - Externalized properties for multiple environments

## 📋 Prerequisites

Before you begin, ensure you have the following installed and configured:

- **Java 8+** - [Download Java](https://www.oracle.com/java/technologies/javase-downloads.html)
- **Maven 3.6+** - [Install Maven](https://maven.apache.org/install.html)
- **MuleSoft Anypoint Studio** (optional) - [Download Studio](https://www.mulesoft.com/platform/studio)
- **Salesforce Org with Agentforce** - [Setup Agentforce](https://help.salesforce.com/agentforce)
- **Git** - [Install Git](https://git-scm.com/downloads)

### Salesforce Setup

1. **Enable Agentforce** in your Salesforce org
2. **Create a Connected App** with OAuth credentials
3. **Configure Agent** and note the Agent ID
4. **Set up OAuth scopes** for API access

## 🛠️ Installation

### 1. Clone the Repository

```bash
git clone https://github.com/akshatasawant9699/mulesoft-agentforce-connector.git
cd mulesoft-agentforce-connector
```

### 2. Configure Environment Variables

Create a `.env` file or set environment variables:

```bash
# Required OAuth Configuration
export AGENTFORCE_CLIENT_ID="your_salesforce_client_id"
export AGENTFORCE_CLIENT_SECRET="your_salesforce_client_secret"
export AGENTFORCE_TOKEN_URL="https://your-org.my.salesforce.com/services/oauth2/token"
export AGENTFORCE_AGENT_ID="your_agent_id"

# Optional: Override defaults
export HTTP_LISTENER_HOST="localhost"
export HTTP_LISTENER_PORT="8081"
```

### 3. Update Configuration

Edit `src/main/resources/application.properties`:

```properties
# Agentforce OAuth Configuration
agentforce.oauth.clientId=${AGENTFORCE_CLIENT_ID}
agentforce.oauth.clientSecret=${AGENTFORCE_CLIENT_SECRET}
agentforce.oauth.tokenUrl=${AGENTFORCE_TOKEN_URL}

# HTTP Listener Configuration
http.listener.host=${HTTP_LISTENER_HOST:localhost}
http.listener.port=${HTTP_LISTENER_PORT:8081}

# Agent Configuration
agentforce.agent.id=${AGENTFORCE_AGENT_ID}
```

### 4. Build the Application

```bash
mvn clean compile
```

## 🚀 Usage

### Starting the Application

#### Local Development
```bash
mvn clean package
java -jar target/a2a-test-1.0.0-SNAPSHOT-mule-application.jar
```

#### Using Maven Plugin
```bash
mvn mule:run
```

### API Endpoints

The application exposes the following endpoints:

#### Start/Continue Conversation
```http
POST http://localhost:8081/test
Content-Type: application/json

{
    "script": "Hello, I need help with my account"
}
```

#### End Conversation
```http
POST http://localhost:8081/test
Content-Type: application/json

{}
```

#### Health Check
```http
GET http://localhost:8081/health
```

### Example Usage

#### 1. Starting a Conversation

**Request:**
```bash
curl -X POST http://localhost:8081/test \
  -H "Content-Type: application/json" \
  -d '{"script": "I need to update my billing address"}'
```

**Response:**
```json
{
  "messages": [
    {
      "id": "msg_123456",
      "content": "I'd be happy to help you update your billing address. Can you please provide your account number?",
      "role": "assistant",
      "timestamp": "2025-01-09T10:30:00Z"
    }
  ],
  "sessionId": "session_abc123def456",
  "status": "active"
}
```

#### 2. Continuing a Conversation

**Request:**
```bash
curl -X POST http://localhost:8081/test \
  -H "Content-Type: application/json" \
  -d '{"script": "My account number is 123456789"}'
```

#### 3. Ending a Conversation

**Request:**
```bash
curl -X POST http://localhost:8081/test \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Response:**
```json
"Your connection has been terminated. Thank you for chatting with Agent"
```

## 🏗️ Architecture

### High-Level Architecture

```
┌─────────────────┐    HTTP/REST    ┌──────────────────┐    OAuth 2.0    ┌─────────────────────┐
│ Client          │────────────────▶│ MuleSoft         │─────────────────▶│ Salesforce          │
│ Application     │                 │ Integration      │                  │ Agentforce          │
│                 │◀────────────────│ Layer            │◀─────────────────│ Platform            │
└─────────────────┘    JSON         └──────────────────┘    AI Response   └─────────────────────┘
```

### Component Flow

```
HTTP Request → Message Extraction → Session Management → AI Processing → Response Handling
     │                │                      │                │               │
     ▼                ▼                      ▼                ▼               ▼
┌─────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌──────────┐  ┌─────────────┐
│ Payload │  │ DataWeave       │  │ Start/Continue  │  │ Agent    │  │ Format &    │
│ Parse   │  │ Transformation  │  │ Conversation    │  │ Response │  │ Return      │
└─────────┘  └─────────────────┘  └─────────────────┘  └──────────┘  └─────────────┘
```

### Key Components

1. **HTTP Listener** - Receives incoming requests
2. **Message Processor** - Extracts and validates user input
3. **Agentforce Connector** - Manages AI conversation flow
4. **Session Manager** - Handles conversation state
5. **Response Handler** - Formats and returns responses

## 🔧 Configuration

### Application Properties

| Property | Description | Default | Required |
|----------|-------------|---------|----------|
| `agentforce.oauth.clientId` | Salesforce Connected App Client ID | - | ✅ |
| `agentforce.oauth.clientSecret` | Salesforce Connected App Client Secret | - | ✅ |
| `agentforce.oauth.tokenUrl` | Salesforce OAuth Token Endpoint | - | ✅ |
| `agentforce.agent.id` | Agentforce Agent ID | - | ✅ |
| `http.listener.host` | HTTP Listener Host | `localhost` | ❌ |
| `http.listener.port` | HTTP Listener Port | `8081` | ❌ |

### Environment-Specific Configuration

#### Development
```properties
# development.properties
agentforce.oauth.tokenUrl=https://dev-org.my.salesforce.com/services/oauth2/token
http.listener.port=8081
log.level=DEBUG
```

#### Production
```properties
# production.properties
agentforce.oauth.tokenUrl=https://prod-org.my.salesforce.com/services/oauth2/token
http.listener.host=0.0.0.0
http.listener.port=8080
log.level=INFO
```

## 🧪 Testing

### Unit Tests

Run unit tests with MUnit:

```bash
mvn test
```

### Integration Tests

```bash
# Start the application in background
mvn mule:run &

# Wait for startup
sleep 30

# Run integration tests
curl -f http://localhost:8081/test \
  -H "Content-Type: application/json" \
  -d '{"script": "test message"}' || echo "Test failed"

# Stop the application
pkill -f mule
```

### Load Testing

Using k6 for load testing:

```bash
# Install k6
brew install k6  # macOS
# or visit https://k6.io/docs/getting-started/installation/

# Run load test
k6 run loadtest.js
```

## 📊 Monitoring & Observability

### Health Check

```bash
curl http://localhost:8081/health
```

**Response:**
```json
{
  "status": "UP",
  "version": "1.0.0",
  "timestamp": "2025-01-09T10:30:00Z"
}
```

### Logging

The application uses structured logging:

```json
{
  "event": "conversation_started",
  "sessionId": "session_123",
  "timestamp": "2025-01-09T10:30:00Z"
}
```

### Metrics

Key metrics tracked:
- Conversation count
- Response time
- Error rates
- Session duration

## 🚀 Deployment

### CloudHub Deployment

```bash
# Deploy to CloudHub
mvn clean package mule:deploy -DcloudHubDeployment
```

### On-Premises Deployment

```bash
# Package application
mvn clean package

# Copy to Mule Runtime
cp target/a2a-test-1.0.0-SNAPSHOT-mule-application.jar $MULE_HOME/apps/
```

### Docker Deployment

```dockerfile
FROM mulesoft/mule-runtime:4.4.0

COPY target/a2a-test-1.0.0-SNAPSHOT-mule-application.jar /opt/mule/apps/

ENV AGENTFORCE_CLIENT_ID=${AGENTFORCE_CLIENT_ID}
ENV AGENTFORCE_CLIENT_SECRET=${AGENTFORCE_CLIENT_SECRET}
ENV AGENTFORCE_TOKEN_URL=${AGENTFORCE_TOKEN_URL}
ENV AGENTFORCE_AGENT_ID=${AGENTFORCE_AGENT_ID}

EXPOSE 8081
```

## 🛡️ Security

### Best Practices Implemented

- ✅ **No Hardcoded Credentials** - All sensitive data externalized
- ✅ **OAuth 2.0 Client Credentials** - Secure service-to-service authentication
- ✅ **Environment Variable Injection** - Runtime configuration
- ✅ **Audit Logging** - Comprehensive request/response tracking
- ✅ **Input Validation** - Sanitization of user inputs
- ✅ **Error Masking** - Sensitive information not exposed in errors

### Security Checklist

- [ ] Rotate OAuth credentials regularly
- [ ] Enable IP allowlisting in production
- [ ] Implement API rate limiting
- [ ] Use HTTPS/TLS in production
- [ ] Monitor for suspicious activity
- [ ] Regular security audits

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

### Development Guidelines

- Follow existing code style and patterns
- Add unit tests for new features
- Update documentation as needed
- Ensure all tests pass before submitting PR

## 📝 Changelog

### [1.0.0] - 2025-01-09

#### Added
- Initial implementation of Agentforce-MuleSoft integration
- OAuth 2.0 authentication with externalized credentials
- Intelligent conversation flow with session management
- Comprehensive error handling and retry logic
- Production-ready configuration management
- Unit and integration testing framework
- Docker and CloudHub deployment support
- Security best practices implementation

## 🐛 Troubleshooting

### Common Issues

#### Authentication Errors
```
ERROR: Invalid credentials
```
**Solution:** Verify OAuth credentials and ensure Connected App is properly configured.

#### Connection Timeouts
```
ERROR: Connection timeout
```
**Solution:** Check network connectivity and increase timeout values if needed.

#### Agent Not Found
```
ERROR: Agent ID not found
```
**Solution:** Verify Agent ID in Salesforce org and ensure Agentforce is enabled.

### Debug Mode

Enable debug logging:
```properties
root.logger.level=DEBUG
agentforce.logger.level=TRACE
```

## 📚 Documentation

- **[Complete Blog Post](Agentforce_MuleSoft_Integration_Blog.md)** - Comprehensive implementation guide
- **[Developer Notes](Developer_Notes_and_Implementation_Guide.md)** - Technical implementation details
- **[API Documentation](docs/api.md)** - Detailed API reference
- **[Architecture Guide](docs/architecture.md)** - System design and patterns

## 🔗 Resources

### Official Documentation
- [Salesforce Agentforce Documentation](https://help.salesforce.com/agentforce)
- [MuleSoft Connector Documentation](https://docs.mulesoft.com/connectors/)
- [OAuth 2.0 Best Practices](https://oauth.net/2/)

### Community
- [MuleSoft Community](https://help.mulesoft.com/)
- [Salesforce Trailblazer Community](https://trailblazer.salesforce.com/)
- [Stack Overflow - MuleSoft](https://stackoverflow.com/questions/tagged/mulesoft)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Authors

- **Developer Enablement Team** - *Initial work* - [@akshatasawant9699](https://github.com/akshatasawant9699)

## 🙏 Acknowledgments

- Salesforce Agentforce team for the innovative AI platform
- MuleSoft community for the robust integration framework
- Contributors and testers who helped refine this implementation

## 📞 Support

For support and questions:

- **Issues**: [GitHub Issues](https://github.com/akshatasawant9699/mulesoft-agentforce-connector/issues)
- **Discussions**: [GitHub Discussions](https://github.com/akshatasawant9699/mulesoft-agentforce-connector/discussions)
- **Email**: [Contact Support](mailto:support@yourcompany.com)

---

⭐ **Star this repository** if you find it helpful!

**Built with ❤️ for the developer community** 