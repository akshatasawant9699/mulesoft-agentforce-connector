# Developer Notes: Agentforce-MuleSoft Integration

## Quick Reference Guide

### Environment Setup Checklist
- [ ] Salesforce Org with Agentforce enabled
- [ ] Connected App created with OAuth credentials
- [ ] MuleSoft Anypoint Studio installed
- [ ] Java 8+ and Maven configured
- [ ] Git repository access

### Configuration Files Overview

#### 1. application.properties
```properties
# Core configuration - keep these externalized
agentforce.oauth.clientId=${AGENTFORCE_CLIENT_ID}
agentforce.oauth.clientSecret=${AGENTFORCE_CLIENT_SECRET}
agentforce.oauth.tokenUrl=${AGENTFORCE_TOKEN_URL}
agentforce.agent.id=${AGENTFORCE_AGENT_ID}

# Local development settings
http.listener.host=localhost
http.listener.port=8081

# Optional: Logging levels
root.logger.level=INFO
agentforce.logger.level=DEBUG
```

#### 2. Key Environment Variables
```bash
# Required for all environments
export AGENTFORCE_CLIENT_ID="your_client_id_here"
export AGENTFORCE_CLIENT_SECRET="your_client_secret_here"
export AGENTFORCE_TOKEN_URL="https://your-org.my.salesforce.com/services/oauth2/token"
export AGENTFORCE_AGENT_ID="your_agent_id_here"

# Optional: Override defaults
export HTTP_LISTENER_HOST="0.0.0.0"  # For container deployment
export HTTP_LISTENER_PORT="8080"     # For cloud deployment
```

## Technical Implementation Notes

### 1. OAuth Flow Details
- **Grant Type**: Client Credentials (service-to-service)
- **Token Scope**: Agentforce API access
- **Token Lifecycle**: Automatic refresh handled by connector
- **Security**: Store credentials in secure vault (never in code)

### 2. Error Handling Strategy
```xml
<!-- Global error handler (add to main flow) -->
<error-handler>
    <on-error-continue type="MS-AGENTFORCE:CONNECTIVITY">
        <logger level="ERROR" message="Agentforce connectivity error: #[error.description]"/>
        <set-payload value='{"error": "Service temporarily unavailable", "code": "CONN_ERROR"}'/>
        <set-variable variableName="httpStatus" value="503"/>
    </on-error-continue>
    
    <on-error-continue type="MS-AGENTFORCE:INVALID_CREDENTIALS">
        <logger level="ERROR" message="Invalid Agentforce credentials"/>
        <set-payload value='{"error": "Authentication failed", "code": "AUTH_ERROR"}'/>
        <set-variable variableName="httpStatus" value="401"/>
    </on-error-continue>
    
    <on-error-continue type="ANY">
        <logger level="ERROR" message="Unexpected error: #[error.description]"/>
        <set-payload value='{"error": "Internal server error", "code": "UNKNOWN"}'/>
        <set-variable variableName="httpStatus" value="500"/>
    </on-error-continue>
</error-handler>
```

### 3. Performance Optimization Tips

#### Connection Pooling
```xml
<ms-agentforce:config name="Config">
    <ms-agentforce:oauth-client-credentials-connection>
        <ms-agentforce:oauth-client-credentials 
            clientId="${agentforce.oauth.clientId}" 
            clientSecret="${agentforce.oauth.clientSecret}" 
            tokenUrl="${agentforce.oauth.tokenUrl}">
        </ms-agentforce:oauth-client-credentials>
        <!-- Connection pooling settings -->
        <pooling-profile maxActive="10" maxIdle="5" maxWait="5000"/>
    </ms-agentforce:oauth-client-credentials-connection>
</ms-agentforce:config>
```

#### Timeout Configuration
```xml
<ms-agentforce:continue-agent-conversation 
    config-ref="Config" 
    timeout="30000"  <!-- 30 seconds -->
    target="response">
    <reconnect count="3" frequency="2000"/>
</ms-agentforce:continue-agent-conversation>
```

## Deployment Strategies

### 1. Local Development
```bash
# Run with hot deployment
mvn clean install
mvn mule:run

# Access endpoints
curl http://localhost:8081/test
```

### 2. CloudHub Deployment
```xml
<!-- cloudhub-deployment.xml -->
<cloudHubDeployment>
    <uri>https://anypoint.mulesoft.com</uri>
    <muleVersion>4.4.0</muleVersion>
    <applicationName>agentforce-integration</applicationName>
    <environment>Production</environment>
    <workers>1</workers>
    <workerType>MICRO</workerType>
    <properties>
        <property>
            <key>agentforce.oauth.clientId</key>
            <value>${AGENTFORCE_CLIENT_ID}</value>
        </property>
    </properties>
</cloudHubDeployment>
```

### 3. On-Premises Deployment
```bash
# Package for standalone runtime
mvn clean package

# Deploy to Mule Runtime
cp target/a2a-test-1.0.0-SNAPSHOT-mule-application.jar $MULE_HOME/apps/

# Monitor deployment
tail -f $MULE_HOME/logs/mule.log
```

## Testing Strategies

### 1. Unit Testing with MUnit
```xml
<!-- test/munit/agentforce-test-suite.xml -->
<munit:test name="test-successful-conversation">
    <munit:behavior>
        <munit-tools:mock-when processor="ms-agentforce:start-agent-conversation">
            <munit-tools:return-payload>
                <![CDATA[{"sessionId": "test-session-123"}]]>
            </munit-tools:return-payload>
        </munit-tools:mock-when>
    </munit:behavior>
    
    <munit:execution>
        <http:request method="POST" config-ref="http-request-config" path="/test">
            <http:body><![CDATA[{"script": "test message"}]]></http:body>
        </http:request>
    </munit:execution>
    
    <munit:validation>
        <munit-tools:assert-that expression="#[payload.sessionId]" 
                                 is="#[MunitTools::equalTo('test-session-123')]"/>
    </munit:validation>
</munit:test>
```

### 2. Integration Testing
```bash
# Test script for CI/CD pipeline
#!/bin/bash

# Start application
mvn mule:run &
MULE_PID=$!

# Wait for startup
sleep 30

# Run integration tests
curl -f http://localhost:8081/test \
  -H "Content-Type: application/json" \
  -d '{"script": "Hello"}' || exit 1

# Cleanup
kill $MULE_PID
```

### 3. Load Testing
```javascript
// k6 load test script
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 10 },
    { duration: '5m', target: 50 },
    { duration: '2m', target: 0 },
  ],
};

export default function() {
  let payload = JSON.stringify({
    script: 'Load test message'
  });
  
  let response = http.post('http://localhost:8081/test', payload, {
    headers: { 'Content-Type': 'application/json' },
  });
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response contains sessionId': (r) => r.json('sessionId') !== null,
  });
}
```

## Monitoring and Observability

### 1. Application Metrics
```xml
<!-- Add to flow for custom metrics -->
<custom-metrics:create-metric name="agentforce.conversations.started" value="1"/>
<custom-metrics:create-metric name="agentforce.response.time" value="#[vars.responseTime]"/>
```

### 2. Health Check Endpoint
```xml
<flow name="health-check">
    <http:listener config-ref="Listener-config" path="/health"/>
    <set-payload value='{"status": "UP", "version": "1.0.0", "timestamp": "#[now()]"}'/>
</flow>
```

### 3. Logging Best Practices
```xml
<!-- Structured logging for better monitoring -->
<logger level="INFO" message='{"event": "conversation_started", "sessionId": "#[payload.sessionId]", "timestamp": "#[now()]"}'/>
<logger level="DEBUG" message='{"event": "user_message", "message": "#[vars.message]", "sessionId": "#[payload.sessionId]"}'/>
<logger level="INFO" message='{"event": "agent_response", "responseLength": "#[sizeOf(vars.response)]", "sessionId": "#[payload.sessionId]"}'/>
```

## Security Hardening

### 1. Network Security
```yaml
# API Gateway policy
rateLimiting:
  maximumRequests: 100
  periodInMilliseconds: 60000
  
ipWhitelist:
  ips:
    - "10.0.0.0/8"
    - "192.168.1.0/24"
```

### 2. Data Sanitization
```xml
<!-- Sanitize user input -->
<set-variable variableName="sanitizedMessage" 
              value="#[vars.message replace /[<>\"']/ with '']"/>
```

### 3. Audit Logging
```xml
<logger level="AUDIT" 
        message='{"user": "#[authentication.principal]", "action": "agentforce_query", "resource": "#[vars.message]", "timestamp": "#[now()]"}'/>
```

## Troubleshooting Common Issues

### 1. Authentication Problems
```bash
# Test OAuth flow manually
curl -X POST https://your-org.my.salesforce.com/services/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET"
```

### 2. Connection Issues
```xml
<!-- Add detailed connection logging -->
<logger level="DEBUG" message="Attempting Agentforce connection with endpoint: ${agentforce.oauth.tokenUrl}"/>
<logger level="DEBUG" message="Using client ID: ${agentforce.oauth.clientId}"/>
```

### 3. Performance Issues
- Monitor heap usage: `-Xmx2G -Xms1G`
- Enable GC logging: `-XX:+PrintGC -XX:+PrintGCDetails`
- Profile with JProfiler or similar tools

## Best Practices Summary

### ✅ Do's
- Always externalize configuration
- Implement comprehensive error handling
- Use structured logging
- Write unit and integration tests
- Monitor application metrics
- Implement circuit breaker patterns
- Use connection pooling
- Validate and sanitize inputs

### ❌ Don'ts
- Never hardcode credentials
- Don't ignore error scenarios
- Avoid synchronous processing for long operations
- Don't skip input validation
- Avoid logging sensitive data
- Don't deploy without testing
- Avoid tight coupling between components

## Future Enhancements

### Planned Features
1. **Multi-language Support**: i18n for error messages and responses
2. **Caching Layer**: Redis integration for conversation state
3. **Message Queuing**: JMS for asynchronous processing
4. **Analytics Dashboard**: Real-time conversation analytics
5. **A/B Testing**: Framework for testing different AI models

### Architecture Evolution
```
Current: Client → MuleSoft → Agentforce
Future: Client → API Gateway → MuleSoft → [Cache/Queue] → Agentforce
                                      ↓
                               Analytics/Monitoring
```

## Support and Resources

### Internal Resources
- Confluence Wiki: [Internal Documentation]
- Slack Channel: #agentforce-integration
- Code Repository: https://github.com/akshatasawant9699/mulesoft-agentforce-connector

### External Resources
- MuleSoft Documentation: https://docs.mulesoft.com
- Salesforce Agentforce Docs: https://help.salesforce.com/agentforce
- OAuth 2.0 RFC: https://tools.ietf.org/html/rfc6749

---

**Document Version**: 1.0.0  
**Last Updated**: January 2025  
**Maintained by**: Development Team 