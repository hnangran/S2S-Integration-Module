# S2S Integration Core Package

This package contains the core integration logic and data model for the S2S Integration Module.

## Package Contents

### Custom Objects
- `Integration_Connection__c` - Stores connection information for integrated Salesforce orgs
- `Object_Mapping__c` - Defines mappings between objects in source and target orgs
- `Field_Mapping__c` - Defines field-level mappings
- `Sync_Job__c` - Tracks sync job execution
- `Sync_Log__c` - Logs sync operations and results

### Apex Classes
Core integration logic including:
- Connection management
- Sync orchestration
- Mapping engine
- Error handling

### Apex Triggers
Automation triggers for:
- Integration connection management
- Sync job processing

### Custom Metadata Types
- Mapping configurations
- Sync rules

### Platform Events
- Sync status events
- Error events

### Flows
- Sync orchestration flows

## Installation Order
This is the base package with no dependencies. Install this package first.

## Package Name
S2S Integration Core

