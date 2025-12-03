# S2S Integration Module - Data Model

## Overview
This document defines the complete data model for the S2S Integration Module, including all custom objects, fields, relationships, custom metadata types, and data architecture decisions.

## Table of Contents
1. [Data Model Overview](#data-model-overview)
2. [Custom Objects](#custom-objects)
3. [Object Relationships](#object-relationships)
4. [Custom Metadata Types](#custom-metadata-types)
5. [Field Definitions](#field-definitions)
6. [Data Volume Estimates](#data-volume-estimates)
7. [Indexing Strategy](#indexing-strategy)
8. [Sharing Model](#sharing-model)
9. [Field History Tracking](#field-history-tracking)
10. [Entity Relationship Diagram](#entity-relationship-diagram)

---

## Data Model Overview

### Purpose
The data model supports:
- Managing connections to multiple Salesforce orgs using Salesforce Named Credentials and External Credentials
- Configuring complex object and field mappings with advanced transformation logic
- Executing and tracking sync operations with sophisticated sync patterns
- Monitoring sync status and errors
- Maintaining audit trails
- Handling conflicts and retries
- Supporting complex relationships and custom objects (key differentiator from Salesforce Connect)

### Design Principles
- **Security First:** Use Salesforce Named Credentials and External Credentials - never store credentials in custom objects
- **Complex Sync Logic:** Support advanced sync patterns, transformations, and relationship handling
- **Custom Object Support:** Full support for custom objects and complex relationships (unlike Salesforce Connect)
- **Scalability:** Support for high-volume sync operations
- **Auditability:** Complete audit trail of all operations
- **Flexibility:** Support for various sync patterns and configurations
- **Performance:** Optimized for query performance and governor limits
- **Security:** Field-level security and sharing model support

---

## Custom Objects

### 1. Integration_Connection__c
**Purpose:** Stores connection information and references to Salesforce Named Credentials/External Credentials for integrated Salesforce organizations.

**Key Characteristics:**
- One record per connected Salesforce org
- **References Named Credential or External Credential** (credentials stored securely in Salesforce, not in this object)
- Tracks connection health and availability
- Supports both production and sandbox environments
- No sensitive data stored in custom fields

**Security Note:** This object does NOT store OAuth tokens, refresh tokens, consumer keys, or any other sensitive credentials. All authentication credentials are managed through Salesforce Named Credentials and External Credentials features, which provide:
- Encrypted storage managed by Salesforce
- Automatic token refresh
- Secure credential management
- Compliance with security best practices

**Record Type:** Standard (no record types initially)

**Sharing Model:** Private (controlled by parent)

---

### 2. Object_Mapping__c
**Purpose:** Defines complex mappings between source and target objects across orgs, supporting custom objects and advanced sync patterns.

**Key Characteristics:**
- One record per object mapping (e.g., Account in Org A → Account in Org B)
- Links source and target objects (supports both standard and custom objects)
- Defines sync direction (unidirectional or bidirectional)
- Contains filter criteria and sync schedule
- **Supports complex sync logic:** Relationship traversal, conditional sync, data transformation
- **Custom Object Support:** Unlike Salesforce Connect, fully supports custom objects and complex relationships

**Record Type:** Standard (no record types initially)

**Sharing Model:** Private (controlled by parent)

---

### 3. Field_Mapping__c
**Purpose:** Defines complex field-level mappings with advanced transformation logic between source and target objects.

**Key Characteristics:**
- One record per field mapping
- Links source field to target field
- Contains complex transformation rules (formulas, lookups, concatenation, data type conversion)
- Supports conditional mappings based on field values
- **Advanced Logic:** Supports multi-step transformations, cross-object lookups, and complex calculations
- **Custom Field Support:** Full support for custom fields and field types

**Record Type:** Standard (no record types initially)

**Sharing Model:** Private (controlled by parent)

---

### 4. Sync_Job__c
**Purpose:** Tracks sync job execution and status.

**Key Characteristics:**
- One record per sync job execution
- Tracks job status, progress, and results
- Links to object mapping and connection
- Stores performance metrics

**Record Type:** Standard (no record types initially)

**Sharing Model:** Private (controlled by parent)

---

### 5. Sync_Log__c
**Purpose:** Detailed logging of sync operations at record and field level.

**Key Characteristics:**
- One record per synced record (or field, depending on granularity)
- Tracks success/failure status
- Stores error details
- Links to sync job

**Record Type:** Standard (no record types initially)

**Sharing Model:** Private (controlled by parent)

---

### 6. Sync_Record__c (Optional - for detailed tracking)
**Purpose:** Tracks individual record sync operations with detailed status.

**Key Characteristics:**
- One record per synced record
- Stores source and target record IDs
- Tracks sync status and timestamps
- Links to sync job

**Record Type:** Standard (no record types initially)

**Sharing Model:** Private (controlled by parent)

---

### 7. Conflict_Record__c (Future - for conflict resolution)
**Purpose:** Tracks conflicts when same record is modified in multiple orgs.

**Key Characteristics:**
- One record per conflict
- Stores conflict details and resolution status
- Links to source records in both orgs
- Tracks resolution history

**Record Type:** Standard (no record types initially)

**Sharing Model:** Private (controlled by parent)

---

## Field Definitions

### Integration_Connection__c

| Field Name | API Name | Type | Length | Required | Unique | Description |
|------------|----------|------|--------|----------|--------|-------------|
| Name | Name | Text(80) | 80 | Yes | No | Connection name (e.g., "Production Org A") |
| Connection Type | Connection_Type__c | Picklist | - | Yes | No | Production, Sandbox, Developer |
| Target Org URL | Target_Org_URL__c | URL | 255 | Yes | No | Salesforce org instance URL (for reference only) |
| Target Org ID | Target_Org_ID__c | Text(18) | 18 | No | Yes | Salesforce org ID |
| **Named Credential** | **Named_Credential_API_Name__c** | **Text(80)** | **80** | **Yes** | **No** | **Named Credential API name (stores credentials securely)** |
| **External Credential** | **External_Credential_API_Name__c** | **Text(80)** | **80** | **No** | **No** | **External Credential API name (alternative to Named Credential)** |
| Authentication Type | Authentication_Type__c | Picklist | - | Yes | No | Named Credential, External Credential, OAuth 2.0 |
| Status | Status__c | Picklist | - | Yes | No | Active, Inactive, Error, Testing |
| Last Health Check | Last_Health_Check__c | DateTime | - | No | No | Last successful health check |
| Health Check Status | Health_Check_Status__c | Picklist | - | No | No | Healthy, Unhealthy, Unknown |
| Availability Percentage | Availability_Percentage__c | Percent | - | No | No | Uptime percentage (last 30 days) |
| Total Health Checks | Total_Health_Checks__c | Number | - | No | No | Total health checks performed |
| Failed Health Checks | Failed_Health_Checks__c | Number | - | No | No | Failed health checks count |
| Last Error | Last_Error__c | Long Text Area | 32000 | No | No | Last error message |
| Last Error Time | Last_Error_Time__c | DateTime | - | No | No | Timestamp of last error |
| Description | Description__c | Long Text Area | 32000 | No | No | Connection description |
| Created By | CreatedById | Lookup(User) | - | System | No | Record creator |
| Last Modified By | LastModifiedById | Lookup(User) | - | System | No | Last modifier |
| Created Date | CreatedDate | DateTime | - | System | No | Creation timestamp |
| Last Modified Date | LastModifiedDate | DateTime | - | System | No | Last modification timestamp |

**⚠️ SECURITY NOTE:** This object does NOT store:
- OAuth access tokens
- OAuth refresh tokens
- Consumer keys or secrets
- Passwords or API keys
- Any other sensitive credentials

All authentication credentials are managed through Salesforce **Named Credentials** or **External Credentials**, which provide:
- Encrypted storage managed by Salesforce
- Automatic token refresh
- Secure credential management
- Compliance with security best practices
- No credential exposure in custom objects

**Picklist Values:**

- **Connection_Type__c:** Production, Sandbox, Developer Edition
- **Authentication_Type__c:** Named Credential, External Credential, OAuth 2.0
- **Status__c:** Active, Inactive, Error, Testing
- **Health_Check_Status__c:** Healthy, Unhealthy, Unknown, Checking

**Validation Rules:**
- If Status = Active, then Named_Credential_API_Name__c OR External_Credential_API_Name__c must be populated
- If Authentication_Type__c = "Named Credential", then Named_Credential_API_Name__c must be populated
- If Authentication_Type__c = "External Credential", then External_Credential_API_Name__c must be populated
- Target_Org_URL__c is optional (can be derived from Named Credential)

**Formula Fields:**
- **Days_Since_Last_Health_Check__c:** `TODAY() - DATEVALUE(Last_Health_Check__c)`
- **Is_Healthy__c:** `Health_Check_Status__c = "Healthy" && Status__c = "Active"`
- **Credential_Reference__c:** Returns the active credential reference (Named Credential or External Credential API name)

**Note:** Token expiration is handled automatically by Named Credentials/External Credentials - no need to track in custom fields.

---

### Object_Mapping__c

| Field Name | API Name | Type | Length | Required | Unique | Description |
|------------|----------|------|--------|----------|--------|-------------|
| Name | Name | Text(80) | 80 | Yes | No | Mapping name (e.g., "Account Sync") |
| Integration Connection | Integration_Connection__c | Master-Detail | - | Yes | No | Parent connection |
| Source Object API Name | Source_Object_API_Name__c | Text(100) | 100 | Yes | No | Source object (e.g., "Account") |
| Target Object API Name | Target_Object_API_Name__c | Text(100) | 100 | Yes | No | Target object (e.g., "Account") |
| Sync Direction | Sync_Direction__c | Picklist | - | Yes | No | Unidirectional, Bidirectional |
| Sync Mode | Sync_Mode__c | Picklist | - | Yes | No | Real-time, Scheduled, On-demand |
| Sync Schedule | Sync_Schedule__c | Text(100) | 100 | No | No | Cron expression for scheduled syncs |
| Filter Criteria | Filter_Criteria__c | Long Text Area | 32000 | No | No | SOQL WHERE clause for filtering |
| **Complex Sync Logic** | **Complex_Sync_Logic__c** | **Long Text Area** | **32000** | **No** | **No** | **JSON configuration for complex sync patterns (relationship traversal, conditional logic)** |
| **Sync Script** | **Sync_Script__c** | **Long Text Area** | **32000** | **No** | **No** | **Apex class name or script for custom sync logic** |
| **Relationship Depth** | **Relationship_Depth__c** | **Number** | **-** | **No** | **No** | **Depth for relationship traversal (e.g., Account → Contacts → Opportunities)** |
| **Custom Object Support** | **Supports_Custom_Objects__c** | **Checkbox** | **-** | **No** | **No** | **Indicates custom object mapping (key differentiator from Salesforce Connect)** |
| Is Active | Is_Active__c | Checkbox | - | Yes | No | Enable/disable mapping |
| Last Sync Time | Last_Sync_Time__c | DateTime | - | No | No | Last successful sync timestamp |
| Next Sync Time | Next_Sync_Time__c | DateTime | - | No | No | Next scheduled sync time |
| Total Records Synced | Total_Records_Synced__c | Number | - | No | No | Total records synced (lifetime) |
| Successful Syncs | Successful_Syncs__c | Number | - | No | No | Successful sync count |
| Failed Syncs | Failed_Syncs__c | Number | - | No | No | Failed sync count |
| Success Rate | Success_Rate__c | Percent | - | No | No | Success percentage |
| Include Related Records | Include_Related_Records__c | Checkbox | - | No | No | Sync related records |
| Related Records Depth | Related_Records_Depth__c | Number | - | No | No | Levels of related records to sync |
| Conflict Resolution Strategy | Conflict_Resolution_Strategy__c | Picklist | - | No | No | Last-write-wins, Manual, Priority |
| Description | Description__c | Long Text Area | 32000 | No | No | Mapping description |
| Created By | CreatedById | Lookup(User) | - | System | No | Record creator |
| Last Modified By | LastModifiedById | Lookup(User) | - | System | No | Last modifier |
| Created Date | CreatedDate | DateTime | - | System | No | Creation timestamp |
| Last Modified Date | LastModifiedDate | DateTime | - | System | No | Last modification timestamp |

**Picklist Values:**

- **Sync_Direction__c:** Unidirectional, Bidirectional
- **Sync_Mode__c:** Real-time, Scheduled, On-demand, Manual
- **Conflict_Resolution_Strategy__c:** Last-Write-Wins, Manual Review, Priority-Based, Source-Wins, Target-Wins

**Validation Rules:**
- If Sync_Mode__c = Scheduled, then Sync_Schedule__c must be populated
- Source_Object_API_Name__c and Target_Object_API_Name__c cannot be the same if Sync_Direction__c = Unidirectional
- If Include_Related_Records__c = true, then Related_Records_Depth__c must be > 0

**Formula Fields:**
- **Is_Sync_Due__c:** `Is_Active__c && Next_Sync_Time__c <= NOW()`
- **Sync_Frequency__c:** Calculated from Sync_Schedule__c

**Unique Constraint:**
- Combination of Integration_Connection__c, Source_Object_API_Name__c, and Target_Object_API_Name__c must be unique

---

### Field_Mapping__c

| Field Name | API Name | Type | Length | Required | Unique | Description |
|------------|----------|------|--------|----------|--------|-------------|
| Name | Name | Text(80) | 80 | Yes | No | Field mapping name |
| Object Mapping | Object_Mapping__c | Master-Detail | - | Yes | No | Parent object mapping |
| Source Field API Name | Source_Field_API_Name__c | Text(100) | 100 | Yes | No | Source field API name |
| Target Field API Name | Target_Field_API_Name__c | Text(100) | 100 | Yes | No | Target field API name |
| Is Required | Is_Required__c | Checkbox | - | No | No | Field is required in target |
| Transformation Type | Transformation_Type__c | Picklist | - | No | No | None, Formula, Lookup, Concatenate, Complex |
| Transformation Formula | Transformation_Formula__c | Long Text Area | 32000 | No | No | Formula for transformation |
| **Complex Transformation** | **Complex_Transformation__c** | **Long Text Area** | **32000** | **No** | **No** | **JSON configuration for multi-step transformations** |
| Default Value | Default_Value__c | Text(255) | 255 | No | No | Default value if source is null |
| Conditional Criteria | Conditional_Criteria__c | Long Text Area | 32000 | No | No | Condition for applying mapping |
| Lookup Mapping | Lookup_Mapping__c | Text(255) | 255 | No | No | Lookup value mapping JSON |
| **Cross-Object Lookup** | **Cross_Object_Lookup__c** | **Text(255)** | **255** | **No** | **No** | **Cross-object lookup configuration (e.g., Account → Custom_Object__c)** |
| **Data Type Conversion** | **Data_Type_Conversion__c** | **Picklist** | **-** | **No** | **No** | **Automatic data type conversion rules** |
| Data Type | Data_Type__c | Picklist | - | No | No | String, Number, Date, DateTime, Boolean |
| Is Active | Is_Active__c | Checkbox | - | Yes | No | Enable/disable field mapping |
| Sync Success Count | Sync_Success_Count__c | Number | - | No | No | Successful syncs for this field |
| Sync Failure Count | Sync_Failure_Count__c | Number | - | No | No | Failed syncs for this field |
| Last Sync Time | Last_Sync_Time__c | DateTime | - | No | No | Last sync timestamp |
| Description | Description__c | Long Text Area | 32000 | No | No | Field mapping description |
| Created By | CreatedById | Lookup(User) | - | System | No | Record creator |
| Last Modified By | LastModifiedById | Lookup(User) | - | System | No | Last modifier |
| Created Date | CreatedDate | DateTime | - | System | No | Creation timestamp |
| Last Modified Date | LastModifiedDate | DateTime | - | System | No | Last modification timestamp |

**Picklist Values:**

- **Transformation_Type__c:** None, Formula, Lookup, Concatenate, Split, Date Format, Currency Convert
- **Data_Type__c:** String, Number, Date, DateTime, Boolean, Email, Phone, URL, Picklist

**Validation Rules:**
- If Transformation_Type__c = Formula, then Transformation_Formula__c must be populated
- Source_Field_API_Name__c and Target_Field_API_Name__c cannot be the same
- If Is_Required__c = true, then Default_Value__c must be populated OR source field must exist

**Unique Constraint:**
- Combination of Object_Mapping__c and Target_Field_API_Name__c must be unique

---

### Sync_Job__c

| Field Name | API Name | Type | Length | Required | Unique | Description |
|------------|----------|------|--------|----------|--------|-------------|
| Name | Name | Text(80) | 80 | Yes | No | Job name (auto-generated) |
| Integration Connection | Integration_Connection__c | Lookup | - | Yes | No | Related connection |
| Object Mapping | Object_Mapping__c | Lookup | - | Yes | No | Related object mapping |
| Job Type | Job_Type__c | Picklist | - | Yes | No | Real-time, Scheduled, Manual, Bulk |
| Status | Status__c | Picklist | - | Yes | No | Queued, Running, Completed, Failed, Cancelled |
| Trigger Type | Trigger_Type__c | Picklist | - | No | No | User, System, Scheduled, API |
| Triggered By | Triggered_By__c | Lookup(User) | - | No | No | User who triggered (if manual) |
| Start Time | Start_Time__c | DateTime | - | No | No | Job start timestamp |
| End Time | End_Time__c | DateTime | - | No | No | Job end timestamp |
| Duration (seconds) | Duration_Seconds__c | Number | - | No | No | Job duration in seconds |
| Total Records | Total_Records__c | Number | - | No | No | Total records to sync |
| Processed Records | Processed_Records__c | Number | - | No | No | Records processed |
| Successful Records | Successful_Records__c | Number | - | No | No | Successfully synced records |
| Failed Records | Failed_Records__c | Number | - | No | No | Failed records |
| Success Rate | Success_Rate__c | Percent | - | No | No | Success percentage |
| API Calls Used | API_Calls_Used__c | Number | - | No | No | API calls consumed |
| Error Message | Error_Message__c | Long Text Area | 32000 | No | No | Error details if failed |
| Retry Count | Retry_Count__c | Number | - | No | No | Number of retry attempts |
| Max Retries | Max_Retries__c | Number | - | No | No | Maximum retry attempts |
| Next Retry Time | Next_Retry_Time__c | DateTime | - | No | No | Next retry timestamp |
| Source Record IDs | Source_Record_IDs__c | Long Text Area | 32000 | No | No | Comma-separated source record IDs |
| Filter Applied | Filter_Applied__c | Long Text Area | 32000 | No | No | Filter criteria applied |
| Created By | CreatedById | Lookup(User) | - | System | No | Record creator |
| Last Modified By | LastModifiedById | Lookup(User) | - | System | No | Last modifier |
| Created Date | CreatedDate | DateTime | - | System | No | Creation timestamp |
| Last Modified Date | LastModifiedDate | DateTime | - | System | No | Last modification timestamp |

**Picklist Values:**

- **Job_Type__c:** Real-time, Scheduled, Manual, Bulk, Backfill
- **Status__c:** Queued, Running, Completed, Failed, Cancelled, Paused
- **Trigger_Type__c:** User, System, Scheduled, API, Platform Event, CDC

**Validation Rules:**
- End_Time__c must be >= Start_Time__c if both are populated
- Retry_Count__c cannot exceed Max_Retries__c
- If Status__c = Running, then Start_Time__c must be populated

**Formula Fields:**
- **Is_Completed__c:** `Status__c = "Completed" || Status__c = "Failed" || Status__c = "Cancelled"`
- **Progress_Percentage__c:** `(Processed_Records__c / Total_Records__c) * 100`
- **Estimated_Time_Remaining__c:** Calculated based on processing rate

**Indexes:**
- Integration_Connection__c, Status__c (composite)
- Object_Mapping__c, Status__c (composite)
- CreatedDate (for date range queries)

---

### Sync_Log__c

| Field Name | API Name | Type | Length | Required | Unique | Description |
|------------|----------|------|--------|----------|--------|-------------|
| Name | Name | Text(80) | 80 | Yes | No | Log entry name (auto-generated) |
| Sync Job | Sync_Job__c | Lookup | - | Yes | No | Parent sync job |
| Log Level | Log_Level__c | Picklist | - | Yes | No | Info, Warning, Error, Debug |
| Source Record ID | Source_Record_ID__c | Text(18) | 18 | No | No | Source Salesforce record ID |
| Target Record ID | Target_Record_ID__c | Text(18) | 18 | No | No | Target Salesforce record ID |
| Object API Name | Object_API_Name__c | Text(100) | 100 | No | No | Object being synced |
| Field API Name | Field_API_Name__c | Text(100) | 100 | No | No | Field being synced (if field-level) |
| Status | Status__c | Picklist | - | Yes | No | Success, Failed, Skipped, Pending |
| Error Code | Error_Code__c | Text(50) | 50 | No | No | Error code if failed |
| Error Message | Error_Message__c | Long Text Area | 32000 | No | No | Detailed error message |
| Error Stack Trace | Error_Stack_Trace__c | Long Text Area | 32000 | No | No | Stack trace if error |
| Sync Time | Sync_Time__c | DateTime | - | No | No | Sync timestamp |
| Processing Time (ms) | Processing_Time_MS__c | Number | - | No | No | Processing time in milliseconds |
| Data Before | Data_Before__c | Long Text Area | 32000 | No | No | Record data before sync (JSON) |
| Data After | Data_After__c | Long Text Area | 32000 | No | No | Record data after sync (JSON) |
| Retry Attempt | Retry_Attempt__c | Number | - | No | No | Retry attempt number |
| Created By | CreatedById | Lookup(User) | - | System | No | Record creator |
| Last Modified By | LastModifiedById | Lookup(User) | - | System | No | Last modifier |
| Created Date | CreatedDate | DateTime | - | System | No | Creation timestamp |
| Last Modified Date | LastModifiedDate | DateTime | - | System | No | Last modification timestamp |

**Picklist Values:**

- **Log_Level__c:** Info, Warning, Error, Debug, Fatal
- **Status__c:** Success, Failed, Skipped, Pending, Retrying

**Validation Rules:**
- If Status__c = Failed, then Error_Message__c must be populated
- Sync_Time__c must be <= NOW()

**Indexes:**
- Sync_Job__c, Status__c (composite)
- Source_Record_ID__c (for record lookup)
- Target_Record_ID__c (for record lookup)
- CreatedDate (for date range queries)
- Log_Level__c, CreatedDate (composite for error queries)

**Data Retention:**
- Consider archiving logs older than 90 days
- Implement data retention policies

---

## Object Relationships

### Relationship Diagram

```
Integration_Connection__c (1)
    │
    ├── Object_Mapping__c (M) [Master-Detail]
    │       │
    │       ├── Field_Mapping__c (M) [Master-Detail]
    │       │
    │       └── Sync_Job__c (M) [Lookup]
    │               │
    │               └── Sync_Log__c (M) [Lookup]
    │
    └── Sync_Job__c (M) [Lookup]
```

### Relationship Details

#### Integration_Connection__c → Object_Mapping__c
- **Type:** Master-Detail
- **Cardinality:** 1:M
- **Cascade Delete:** Yes
- **Required:** Yes
- **Purpose:** Object mappings belong to a specific connection

#### Object_Mapping__c → Field_Mapping__c
- **Type:** Master-Detail
- **Cardinality:** 1:M
- **Cascade Delete:** Yes
- **Required:** Yes
- **Purpose:** Field mappings belong to a specific object mapping

#### Integration_Connection__c → Sync_Job__c
- **Type:** Lookup
- **Cardinality:** 1:M
- **Cascade Delete:** No (set to null)
- **Required:** Yes
- **Purpose:** Sync jobs are associated with a connection

#### Object_Mapping__c → Sync_Job__c
- **Type:** Lookup
- **Cardinality:** 1:M
- **Cascade Delete:** No (set to null)
- **Required:** Yes
- **Purpose:** Sync jobs execute a specific object mapping

#### Sync_Job__c → Sync_Log__c
- **Type:** Lookup
- **Cardinality:** 1:M
- **Cascade Delete:** No (set to null)
- **Required:** Yes
- **Purpose:** Logs belong to a specific sync job

---

## Named Credentials and External Credentials

### Overview
The S2S Integration Module uses Salesforce **Named Credentials** and **External Credentials** for secure authentication management. This approach provides:

- **Secure Storage:** Credentials encrypted and managed by Salesforce
- **Automatic Token Refresh:** OAuth tokens refreshed automatically
- **No Custom Object Storage:** No sensitive data stored in custom objects
- **Compliance:** Meets security and compliance requirements
- **Best Practice:** Follows Salesforce security best practices

### Named Credentials Configuration

Each `Integration_Connection__c` record references a Named Credential that contains:
- **Endpoint URL:** Target Salesforce org instance URL
- **Identity Type:** OAuth 2.0
- **Authentication Protocol:** OAuth 2.0
- **Scope:** API, Refresh Token, Full Access (as needed)
- **Consumer Key:** Connected App Consumer Key
- **Consumer Secret:** Connected App Consumer Secret (stored securely)
- **Token Endpoint:** OAuth token endpoint URL
- **Authorization Endpoint:** OAuth authorization endpoint URL

### External Credentials (Alternative)

For more complex scenarios, External Credentials can be used:
- **Multiple Credentials:** Support for different credential types
- **Credential Principals:** Multiple principals per credential
- **Advanced Authentication:** Support for certificate-based auth, JWT, etc.

### Implementation Pattern

```apex
// Example: Using Named Credential in Apex
HttpRequest req = new HttpRequest();
req.setEndpoint('callout:NamedCredentialAPI/services/data/v65.0/sobjects/Account');
req.setMethod('GET');
Http http = new Http();
HttpResponse res = http.send(req);
// Named Credential automatically handles authentication
```

### Migration from Custom Object Storage

If migrating from a system that stored credentials in custom objects:
1. Create Named Credential for each connection
2. Update Integration_Connection__c to reference Named Credential API name
3. Remove credential fields from custom object
4. Update Apex code to use Named Credentials

---

## Custom Metadata Types

### 1. Sync_Configuration__mdt
**Purpose:** System-wide configuration settings stored as metadata.

**Fields:**
- **Label:** Configuration Name
- **API Name:** DeveloperName (auto)
- **Default_Retry_Count__c:** Number - Default retry attempts
- **Default_Max_Retries__c:** Number - Default max retries
- **Default_Batch_Size__c:** Number - Default batch size for bulk operations
- **Health_Check_Interval_Minutes__c:** Number - Health check frequency
- **Log_Retention_Days__c:** Number - Log retention period
- **Enable_Platform_Events__c:** Checkbox - Enable platform events
- **Enable_CDC__c:** Checkbox - Enable Change Data Capture
- **API_Rate_Limit_Threshold__c:** Percent - API limit warning threshold

---

### 2. Mapping_Template__mdt
**Purpose:** Reusable mapping templates for common use cases.

**Fields:**
- **Label:** Template Name
- **API Name:** DeveloperName (auto)
- **Source_Object__c:** Text - Source object API name
- **Target_Object__c:** Text - Target object API name
- **Field_Mappings_JSON__c:** Long Text Area - JSON array of field mappings
- **Filter_Criteria__c:** Text - Default filter criteria
- **Description__c:** Long Text Area - Template description

---

## Data Volume Estimates

### Integration_Connection__c
- **Estimated Records:** 1-50 per org
- **Growth Rate:** Low (new connections infrequent)
- **Retention:** Permanent (no deletion)

### Object_Mapping__c
- **Estimated Records:** 10-200 per connection
- **Growth Rate:** Medium (new mappings as needed)
- **Retention:** Permanent (no deletion)

### Field_Mapping__c
- **Estimated Records:** 5-50 per object mapping
- **Growth Rate:** Medium (field mappings may change)
- **Retention:** Permanent (no deletion)

### Sync_Job__c
- **Estimated Records:** 1,000-100,000 per month (depends on sync frequency)
- **Growth Rate:** High (continuous creation)
- **Retention:** 90 days (archive older records)

### Sync_Log__c
- **Estimated Records:** 10,000-1,000,000 per month (depends on sync volume)
- **Growth Rate:** Very High (one log per record/field sync)
- **Retention:** 30-90 days (implement data archival)

**Data Archival Strategy:**
- Archive Sync_Job__c records older than 90 days
- Archive Sync_Log__c records older than 30 days
- Use BigObjects or external storage for archived data

---

## Indexing Strategy

### Custom Indexes

#### Integration_Connection__c
- **Status__c, Health_Check_Status__c** (composite) - For filtering active/healthy connections
- **Target_Org_ID__c** (unique) - For org lookup

#### Object_Mapping__c
- **Integration_Connection__c, Is_Active__c** (composite) - For active mappings per connection
- **Integration_Connection__c, Source_Object_API_Name__c, Target_Object_API_Name__c** (composite, unique) - For mapping lookup

#### Field_Mapping__c
- **Object_Mapping__c, Is_Active__c** (composite) - For active field mappings
- **Object_Mapping__c, Target_Field_API_Name__c** (composite, unique) - For field lookup

#### Sync_Job__c
- **Integration_Connection__c, Status__c, CreatedDate** (composite) - For job queries by connection and status
- **Object_Mapping__c, Status__c, CreatedDate** (composite) - For job queries by mapping
- **Status__c, CreatedDate** (composite) - For pending/queued jobs
- **Triggered_By__c, CreatedDate** (composite) - For user-initiated jobs

#### Sync_Log__c
- **Sync_Job__c, Status__c** (composite) - For job logs by status
- **Source_Record_ID__c** - For record lookup
- **Target_Record_ID__c** - For record lookup
- **Log_Level__c, CreatedDate** (composite) - For error queries
- **Object_API_Name__c, CreatedDate** (composite) - For object-level queries

### Standard Indexes (Auto-created)
- CreatedDate (all objects)
- LastModifiedDate (all objects)
- OwnerId (if sharing model uses owner)

---

## Sharing Model

### Sharing Strategy: Private with Sharing Rules

**Default Access:** Private (users can only see records they own or are shared with)

**Sharing Rules:**
1. **Integration Administrators:** Full access to all records via permission set
2. **Integration Users:** Read access to connections and mappings, create/read sync jobs
3. **Integration Viewers:** Read-only access to connections, mappings, and sync jobs

**Manual Sharing:**
- Support manual sharing for specific use cases
- Sharing with public groups for team access

**Apex Sharing:**
- Implement Apex sharing for programmatic access control
- Respect field-level security in all queries

---

## Field History Tracking

### Integration_Connection__c
**Track History For:**
- Status__c
- Health_Check_Status__c
- Last_Error__c
- Named_Credential_API_Name__c
- External_Credential_API_Name__c

**Retention:** 18 months

### Object_Mapping__c
**Track History For:**
- Is_Active__c
- Sync_Mode__c
- Sync_Schedule__c
- Filter_Criteria__c
- Conflict_Resolution_Strategy__c

**Retention:** 18 months

### Field_Mapping__c
**Track History For:**
- Is_Active__c
- Transformation_Type__c
- Transformation_Formula__c
- Default_Value__c

**Retention:** 18 months

### Sync_Job__c
**Track History For:**
- Status__c
- Error_Message__c
- Retry_Count__c

**Retention:** 12 months

**Note:** Sync_Log__c does not need field history tracking as it's already a log/audit record.

---

## Entity Relationship Diagram

### Text-Based ERD

```
┌─────────────────────────────────────┐
│   Integration_Connection__c          │
│   ───────────────────────────        │
│   • Name                             │
│   • Connection_Type__c                │
│   • Target_Org_URL__c                │
│   • Status__c                        │
│   • Health_Check_Status__c            │
│   • Last_Health_Check__c             │
│   ───────────────────────────        │
│   PK: Id                              │
└──────────────┬───────────────────────┘
               │
               │ 1:M (Master-Detail)
               │
               ▼
┌─────────────────────────────────────┐
│   Object_Mapping__c                 │
│   ───────────────────────────       │
│   • Name                             │
│   • Source_Object_API_Name__c        │
│   • Target_Object_API_Name__c        │
│   • Sync_Direction__c                │
│   • Sync_Mode__c                     │
│   • Is_Active__c                     │
│   ───────────────────────────       │
│   PK: Id                             │
│   FK: Integration_Connection__c      │
└──────────────┬───────────────────────┘
               │
               │ 1:M (Master-Detail)
               │
               ▼
┌─────────────────────────────────────┐
│   Field_Mapping__c                 │
│   ───────────────────────────       │
│   • Name                             │
│   • Source_Field_API_Name__c         │
│   • Target_Field_API_Name__c         │
│   • Transformation_Type__c           │
│   • Is_Active__c                     │
│   ───────────────────────────       │
│   PK: Id                             │
│   FK: Object_Mapping__c              │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│   Integration_Connection__c          │
│   (from above)                       │
└──────────────┬───────────────────────┘
               │
               │ 1:M (Lookup)
               │
               ▼
┌─────────────────────────────────────┐
│   Object_Mapping__c                 │
│   (from above)                       │
└──────────────┬───────────────────────┘
               │
               │ 1:M (Lookup)
               │
               ▼
┌─────────────────────────────────────┐
│   Sync_Job__c                       │
│   ───────────────────────────       │
│   • Name                             │
│   • Job_Type__c                      │
│   • Status__c                        │
│   • Start_Time__c                    │
│   • End_Time__c                      │
│   • Total_Records__c                 │
│   • Successful_Records__c            │
│   • Failed_Records__c                │
│   ───────────────────────────       │
│   PK: Id                             │
│   FK: Integration_Connection__c      │
│   FK: Object_Mapping__c              │
└──────────────┬───────────────────────┘
               │
               │ 1:M (Lookup)
               │
               ▼
┌─────────────────────────────────────┐
│   Sync_Log__c                       │
│   ───────────────────────────       │
│   • Name                             │
│   • Log_Level__c                     │
│   • Source_Record_ID__c              │
│   • Target_Record_ID__c               │
│   • Status__c                        │
│   • Error_Message__c                 │
│   ───────────────────────────       │
│   PK: Id                             │
│   FK: Sync_Job__c                    │
└─────────────────────────────────────┘
```

---

## Data Model Considerations

### Security
- **Named Credentials:** All authentication credentials stored in Salesforce Named Credentials/External Credentials (NOT in custom objects)
- **Field-Level Security:** Implement FLS on all custom fields
- **No Credential Storage:** Never store OAuth tokens, refresh tokens, consumer keys, or secrets in custom objects
- **Sharing:** Private sharing model with permission set-based access
- **Compliance:** Using Named Credentials ensures compliance with security best practices

### Performance
- **Bulkification:** All DML operations must be bulkified
- **Query Optimization:** Use indexed fields in WHERE clauses
- **Governor Limits:** Design to minimize SOQL queries and DML statements
- **Pagination:** Implement pagination for large data sets

### Scalability
- **Data Archival:** Implement archival strategy for high-volume objects
- **BigObjects:** Consider BigObjects for Sync_Log__c if volume exceeds limits
- **Async Processing:** Use async processing for large sync jobs
- **Batch Processing:** Implement batch processing for bulk operations

### Maintainability
- **Naming Conventions:** Consistent API naming (__c suffix, descriptive names)
- **Documentation:** Document all custom fields and objects
- **Validation:** Comprehensive validation rules to ensure data quality
- **Formulas:** Use formula fields for calculated values to reduce code complexity

---

## Complex Sync Logic Support

### Overview
The S2S Integration Module supports sophisticated sync logic that goes beyond standard Salesforce Connect capabilities. This includes complex relationship traversal, multi-step transformations, and custom sync patterns.

### Complex Relationship Support

#### Relationship Traversal
- **Multi-Level Relationships:** Traverse relationships across multiple levels (e.g., Account → Contacts → Opportunities → Products)
- **Relationship Depth Configuration:** Configurable depth for relationship traversal
- **Custom Object Relationships:** Full support for custom object relationships (lookup, master-detail)
- **Self-Referential Relationships:** Support for self-referencing objects (e.g., Account hierarchies)

#### Relationship Mapping
- **Cross-Object Lookups:** Map fields from related objects (e.g., Account.Name → Contact.Account_Name__c)
- **Relationship Preservation:** Maintain relationships when syncing related records
- **Circular Reference Detection:** Detect and handle circular relationship references

### Complex Transformation Logic

#### Multi-Step Transformations
- **Transformation Pipeline:** Chain multiple transformations together
- **Conditional Transformations:** Apply different transformations based on field values
- **Cross-Object Data:** Pull data from related objects during transformation
- **Custom Apex Hooks:** Execute custom Apex code for complex transformations

#### Data Type Handling
- **Automatic Conversion:** Automatic data type conversion (e.g., String → Number, Date → DateTime)
- **Custom Conversion Rules:** Define custom conversion rules for specific scenarios
- **Validation:** Validate data before and after transformation
- **Error Handling:** Handle transformation errors gracefully

### Custom Sync Patterns

#### Conditional Sync
- **Field-Based Conditions:** Sync records only when specific field conditions are met
- **Complex Criteria:** Support for complex SOQL-based filter criteria
- **Dynamic Filtering:** Apply filters dynamically based on runtime conditions

#### Sync Orchestration
- **Dependency Management:** Sync objects in correct order based on dependencies
- **Parallel Processing:** Sync independent objects in parallel
- **Batch Processing:** Process large datasets in batches
- **Incremental Sync:** Sync only changed records (delta sync)

### Custom Object Support (vs. Salesforce Connect)

**Salesforce Connect Limitations:**
- Primarily supports standard objects
- Limited custom object support
- Basic relationship handling
- Limited transformation capabilities

**S2S Integration Module Advantages:**
- **Full Custom Object Support:** Complete support for all custom objects and custom fields
- **Complex Relationships:** Advanced relationship traversal and mapping
- **Custom Sync Logic:** Support for custom Apex scripts and complex sync patterns
- **Flexible Mapping:** Configurable mappings with advanced transformation rules
- **Relationship Depth:** Configurable depth for relationship traversal

### Implementation Examples

#### Example 1: Multi-Level Relationship Sync
```
Account (Org A) → Account (Org B)
  ├── Contacts → Contacts (with Account lookup)
  │   └── Opportunities → Opportunities (with Contact lookup)
  └── Custom_Objects__c → Custom_Objects__c (with Account lookup)
```

#### Example 2: Complex Transformation
```
Source: Account.Industry = "Technology"
Target: Account.Category__c = "Tech" (via lookup mapping)
        Account.Priority__c = "High" (conditional)
        Account.Full_Name__c = Account.Name + " - " + Account.BillingCity (concatenation)
```

#### Example 3: Conditional Sync
```
Sync Account only if:
- Account.Type = "Customer"
- AND Account.AnnualRevenue > 1000000
- AND Account.CreatedDate > LAST_N_DAYS:30
```

---

## Next Steps

1. **Review and Approve:** Review this data model with stakeholders
2. **Create Named Credentials:** Set up Named Credentials/External Credentials for each connection
3. **Create Objects:** Create custom objects in Salesforce
4. **Create Fields:** Implement all fields as defined (excluding credential fields)
5. **Set Up Relationships:** Configure master-detail and lookup relationships
6. **Configure Sharing:** Set up sharing model and permission sets
7. **Enable Field History:** Enable field history tracking as specified
8. **Create Indexes:** Request custom indexes (if needed beyond standard)
9. **Test Data Model:** Create test data and validate relationships
10. **Document in Salesforce:** Add object and field descriptions in Salesforce UI

---

*Last Updated: [Date]*
*Version: 1.1*
*Next Review: After initial implementation*

