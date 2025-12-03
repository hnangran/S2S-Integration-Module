# S2S Integration Module - Credential Management Strategy

## Overview
This document outlines the credential management strategy for the S2S Integration Module, emphasizing the use of Salesforce Named Credentials and External Credentials instead of storing credentials in custom objects.

## Security Principle

**CRITICAL:** The S2S Integration Module **NEVER** stores OAuth tokens, refresh tokens, consumer keys, secrets, passwords, or any other sensitive credentials in custom objects. All authentication credentials are managed through Salesforce's built-in credential management features.

## Credential Storage Methods

### Primary Method: Named Credentials

**What are Named Credentials?**
- Salesforce feature for securely storing and managing authentication credentials
- Encrypted storage managed by Salesforce
- Automatic token refresh for OAuth 2.0
- No credential exposure in custom objects or code

**Configuration:**
1. Create Named Credential in Setup → Named Credentials
2. Configure OAuth 2.0 settings:
   - Endpoint URL (target Salesforce org)
   - Identity Type: OAuth 2.0
   - Authentication Protocol: OAuth 2.0
   - Scope: API, Refresh Token, Full Access
   - Consumer Key: Connected App Consumer Key
   - Consumer Secret: Connected App Consumer Secret
3. Reference Named Credential API name in `Integration_Connection__c.Named_Credential_API_Name__c`

**Usage in Apex:**
```apex
// Example: Using Named Credential
HttpRequest req = new HttpRequest();
req.setEndpoint('callout:MyNamedCredential/services/data/v65.0/sobjects/Account');
req.setMethod('GET');
Http http = new Http();
HttpResponse res = http.send(req);
// Authentication handled automatically by Named Credential
```

**Benefits:**
- Secure credential storage
- Automatic token refresh
- No credential exposure
- Compliance with security best practices
- Easy credential rotation

### Alternative Method: External Credentials

**What are External Credentials?**
- Advanced credential management for complex scenarios
- Support for multiple credential principals
- Certificate-based authentication
- JWT authentication
- Integration with external credential vaults

**When to Use:**
- Multiple authentication methods needed
- Certificate-based authentication required
- Integration with external credential vaults
- Complex authentication scenarios

**Configuration:**
1. Create External Credential in Setup → External Credentials
2. Configure credential principals
3. Reference External Credential API name in `Integration_Connection__c.External_Credential_API_Name__c`

## Integration_Connection__c Object

### Fields Related to Credentials

| Field | Type | Description |
|-------|------|-------------|
| Named_Credential_API_Name__c | Text(80) | API name of Named Credential (primary method) |
| External_Credential_API_Name__c | Text(80) | API name of External Credential (alternative) |
| Authentication_Type__c | Picklist | Named Credential, External Credential, OAuth 2.0 |

### Fields NOT Included (Security)

The following fields are **NOT** included in `Integration_Connection__c`:
- ❌ Access_Token__c
- ❌ Refresh_Token__c
- ❌ Consumer_Key__c
- ❌ Consumer_Secret__c
- ❌ Password__c
- ❌ API_Key__c
- ❌ Any other credential fields

**Rationale:** All credentials are stored in Named Credentials/External Credentials, not in custom objects.

## Migration from Custom Object Storage

If migrating from a system that stored credentials in custom objects:

### Step 1: Create Named Credentials
1. For each existing connection, create a Named Credential
2. Configure OAuth 2.0 settings
3. Test authentication

### Step 2: Update Integration_Connection__c
1. Add `Named_Credential_API_Name__c` field
2. Populate with Named Credential API names
3. Remove old credential fields (Access_Token__c, Refresh_Token__c, etc.)

### Step 3: Update Apex Code
1. Replace credential retrieval logic with Named Credential callouts
2. Remove any code that reads/writes credentials from custom objects
3. Update HTTP callouts to use `callout:NamedCredentialAPI` format

### Step 4: Testing
1. Test authentication with Named Credentials
2. Verify automatic token refresh
3. Test all sync operations

## Best Practices

### 1. Use Named Credentials by Default
- Named Credentials are the recommended approach for most scenarios
- Simpler configuration
- Automatic token refresh
- Better security

### 2. Use External Credentials for Complex Scenarios
- Only when Named Credentials don't meet requirements
- Certificate-based authentication
- Multiple credential principals
- External vault integration

### 3. Never Store Credentials in Custom Objects
- Security risk
- Compliance violation
- Maintenance burden
- Token expiration issues

### 4. Regular Credential Rotation
- Rotate credentials periodically
- Update Named Credentials/External Credentials
- No code changes needed (just update credential configuration)

### 5. Monitor Credential Health
- Track authentication failures
- Monitor token refresh success
- Alert on credential expiration

## Security Benefits

### Using Named Credentials/External Credentials:
✅ Encrypted storage managed by Salesforce  
✅ Automatic token refresh  
✅ No credential exposure in custom objects  
✅ Compliance with security best practices  
✅ Easy credential rotation  
✅ Audit trail of credential usage  
✅ Field-level security not needed for credentials  

### Storing Credentials in Custom Objects (NOT RECOMMENDED):
❌ Security risk  
❌ Manual token refresh required  
❌ Credential exposure in custom objects  
❌ Compliance concerns  
❌ Complex credential rotation  
❌ Requires field-level encryption  
❌ Maintenance burden  

## Implementation Checklist

- [ ] Create Named Credentials for each connection
- [ ] Configure OAuth 2.0 settings in Named Credentials
- [ ] Add `Named_Credential_API_Name__c` field to `Integration_Connection__c`
- [ ] Remove any credential storage fields from custom objects
- [ ] Update Apex code to use Named Credential callouts
- [ ] Test authentication with Named Credentials
- [ ] Verify automatic token refresh
- [ ] Update documentation
- [ ] Train administrators on Named Credential management

## Troubleshooting

### Issue: Authentication Fails
- Verify Named Credential configuration
- Check Connected App settings
- Verify OAuth scopes
- Check token expiration

### Issue: Token Not Refreshing
- Verify Named Credential refresh token settings
- Check Connected App refresh token policy
- Verify OAuth scopes include refresh token

### Issue: Credential Not Found
- Verify Named Credential API name spelling
- Check Named Credential is active
- Verify user has access to Named Credential

---

*Last Updated: [Date]*
*Version: 1.0*

