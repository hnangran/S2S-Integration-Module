# S2S Integration Module - Packaging Architecture and Strategy

## Overview
This document outlines the packaging architecture, scratch org requirements, and deployment strategy for the S2S Integration Module.

## Table of Contents
1. [Scratch Org Requirements](#scratch-org-requirements)
2. [Packaging Architecture](#packaging-architecture)
3. [Package Strategy](#package-strategy)
4. [Versioning Strategy](#versioning-strategy)
5. [Dependencies](#dependencies)
6. [Distribution Strategy](#distribution-strategy)
7. [Development Workflow](#development-workflow)

---

## Scratch Org Requirements

### Base Configuration

#### Edition Requirements
- **Minimum Edition:** Enterprise Edition
- **Recommended Edition:** Enterprise or Unlimited Edition
- **Rationale:** 
  - Enterprise Edition provides necessary API limits and features
  - Unlimited Edition recommended for production-like testing with higher limits

#### Required Features
The following features must be enabled in scratch org definitions:

```json
{
  "features": [
    "API",
    "Communities",
    "DevelopmentWave",
    "LightningServiceConsole",
    "PlatformEncryption",
    "ServiceCloud",
    "Sites"
  ]
}
```

**Feature Descriptions:**
- **API:** Required for REST/SOAP API access and integrations
- **Communities:** May be needed for partner portal integrations
- **DevelopmentWave:** For advanced development features
- **LightningServiceConsole:** For Lightning component development
- **PlatformEncryption:** For Shield Platform Encryption (security compliance)
- **ServiceCloud:** For service-related objects and features
- **Sites:** For potential web-based integration interfaces

#### Settings Configuration

##### Org Preferences
```json
{
  "settings": {
    "orgPreferenceSettings": {
      "networksEnabled": true,
      "s1DesktopEnabled": true,
      "s1EncryptedStoragePref2": false
    },
    "lightningExperienceSettings": {
      "enableS1DesktopEnabled": true
    },
    "securitySettings": {
      "sessionSettings": {
        "sessionTimeout": "TwelveHours",
        "sessionTimeoutWarning": true
      },
      "passwordPolicies": {
        "lockoutInterval": 15,
        "maxLoginAttempts": 5,
        "minimumPasswordLength": 8,
        "passwordComplexity": 3,
        "passwordExpiration": 90,
        "passwordHistory": 5,
        "questionRestriction": 2
      }
    },
    "mobileSettings": {
      "enableS1EncryptedStoragePref2": false
    }
  }
}
```

##### API Settings
```json
{
  "settings": {
    "apiSettings": {
      "enableApiAccess": true
    }
  }
}
```

##### Platform Encryption (Optional but Recommended)
```json
{
  "settings": {
    "encryptionSettings": {
      "enableDeterministicEncryption": false,
      "enableEncryptFieldHistory": false,
      "enableEventBusEncryption": false
    }
  }
}
```

#### Object Permissions
The following standard objects require access:
- Account
- Contact
- User
- Profile
- PermissionSet
- ConnectedApplication
- NamedCredential
- PlatformEvent

#### Permission Sets Required
- Custom integration permission sets (to be created)
- System Administrator access for testing

#### Connected App Requirements
- OAuth 2.0 enabled
- API access enabled
- Refresh token rotation (recommended)
- IP restrictions (optional, for security)

#### Named Credentials
- Support for Named Credentials configuration
- OAuth 2.0 authentication
- Certificate-based authentication (optional)

#### Platform Events
- Platform Event objects enabled
- Event publishing permissions
- Event subscription capabilities

#### Custom Metadata Types
- Custom Metadata Types enabled
- Metadata API access

#### API Limits
- Minimum API calls per 24 hours: 5,000 (Enterprise)
- Recommended: 10,000+ (Unlimited)
- Concurrent API requests: 25 (Enterprise), 50 (Unlimited)

### Complete Scratch Org Definition

**File:** `config/project-scratch-def.json`

```json
{
  "orgName": "S2S Integration Module",
  "edition": "Enterprise",
  "hasSampleData": false,
  "features": [
    "API",
    "Communities",
    "DevelopmentWave",
    "LightningServiceConsole",
    "PlatformEncryption",
    "ServiceCloud",
    "Sites"
  ],
  "settings": {
    "lightningExperienceSettings": {
      "enableS1DesktopEnabled": true
    },
    "mobileSettings": {
      "enableS1EncryptedStoragePref2": false
    },
    "securitySettings": {
      "sessionSettings": {
        "sessionTimeout": "TwelveHours",
        "sessionTimeoutWarning": true
      },
      "passwordPolicies": {
        "lockoutInterval": 15,
        "maxLoginAttempts": 5,
        "minimumPasswordLength": 8,
        "passwordComplexity": 3,
        "passwordExpiration": 90,
        "passwordHistory": 5
      }
    },
    "apiSettings": {
      "enableApiAccess": true
    },
    "encryptionSettings": {
      "enableDeterministicEncryption": false,
      "enableEncryptFieldHistory": false,
      "enableEventBusEncryption": false
    }
  }
}
```

### Scratch Org Variants

#### Development Scratch Org
- Edition: Enterprise
- Sample Data: No
- Duration: 7 days (default)
- Purpose: Feature development and unit testing

#### Integration Testing Scratch Org
- Edition: Enterprise
- Sample Data: Yes (for testing)
- Duration: 30 days
- Purpose: End-to-end integration testing

#### Performance Testing Scratch Org
- Edition: Unlimited
- Sample Data: Yes (large datasets)
- Duration: 30 days
- Purpose: Performance and load testing

---

## Packaging Architecture

### Package Type Decision

#### Recommended: Unlocked Package
**Rationale:**
- **Flexibility:** Components can be modified after installation
- **No Namespace:** Easier for customers to customize
- **Version Control:** Supports semantic versioning
- **Metadata Visibility:** All metadata is visible and editable
- **Upgrade Path:** Supports upgrades and patches
- **Best for:** ISV solutions, open-source projects, internal tools

**When to Use:**
- Customer customization expected
- Internal enterprise solutions
- Open-source or community projects
- Solutions requiring post-installation modifications

#### Alternative: Managed Package
**Consider if:**
- Need to protect intellectual property
- Require namespace isolation
- Need AppExchange distribution
- Want to prevent customer modifications

**Trade-offs:**
- Less flexible (harder to modify after install)
- Requires namespace
- More complex upgrade process
- Better IP protection

#### Alternative: Unmanaged Package
**Consider if:**
- One-time deployment
- No upgrade path needed
- Simple distribution
- Quick prototyping

**Trade-offs:**
- No upgrade support
- Manual dependency management
- No versioning

### Recommended Architecture: Multi-Package Strategy

#### Package Structure

```
S2S Integration Module
├── Core Package (s2s-core)
│   ├── Custom Objects
│   ├── Custom Metadata Types
│   ├── Apex Classes (Core Logic)
│   ├── Apex Triggers
│   └── Platform Events
│
├── UI Package (s2s-ui)
│   ├── Lightning Web Components
│   ├── Aura Components
│   ├── Lightning Pages
│   └── Static Resources
│
├── Admin Package (s2s-admin)
│   ├── Permission Sets
│   ├── Profiles (optional)
│   ├── Custom Settings
│   └── Setup Components
│
└── Sample Data Package (s2s-sample-data) [Optional]
    └── Sample records for testing
```

#### Package Organization Rationale

**Core Package (s2s-core)**
- **Purpose:** Core integration logic and data model
- **Dependencies:** None (base package)
- **Installation Order:** First
- **Contents:**
  - Custom objects (Integration Connection, Object Mapping, Field Mapping, Sync Job, etc.)
  - Custom metadata types (Mapping Configuration, Sync Rules)
  - Core Apex classes (Connection Manager, Sync Engine, Error Handler)
  - Platform Events (Sync Status Event, Error Event)
  - Apex Triggers (for automation)

**UI Package (s2s-ui)**
- **Purpose:** User interface components
- **Dependencies:** s2s-core
- **Installation Order:** Second
- **Contents:**
  - Lightning Web Components (Connection Manager, Mapping Configurator, Sync Monitor)
  - Aura Components (if needed for compatibility)
  - Lightning App Pages
  - Lightning Record Pages
  - Static Resources (images, CSS, JavaScript libraries)

**Admin Package (s2s-admin)**
- **Purpose:** Administrative setup and permissions
- **Dependencies:** s2s-core, s2s-ui
- **Installation Order:** Third
- **Contents:**
  - Permission Sets (Integration Administrator, Integration User, Integration Viewer)
  - Custom Settings (System Configuration)
  - Setup Wizard components
  - Post-installation scripts

**Sample Data Package (s2s-sample-data)** [Optional]
- **Purpose:** Sample data for testing and demos
- **Dependencies:** s2s-core
- **Installation Order:** Last (optional)
- **Contents:**
  - Sample integration connections
  - Sample mappings
  - Test records

### Package Directory Structure

The `sfdx-project.json` is configured as follows:

```json
{
  "packageDirectories": [
    {
      "path": "force-app/core",
      "default": true,
      "package": "S2S Integration Core",
      "versionName": "ver 1.0",
      "versionNumber": "1.0.0.NEXT",
      "dependencies": []
    },
    {
      "path": "force-app/ui",
      "package": "S2S Integration UI",
      "versionName": "ver 1.0",
      "versionNumber": "1.0.0.NEXT",
      "dependencies": [
        {
          "package": "S2S Integration Core",
          "versionNumber": "1.0.0.LATEST"
        }
      ]
    },
    {
      "path": "force-app/admin",
      "package": "S2S Integration Admin",
      "versionName": "ver 1.0",
      "versionNumber": "1.0.0.NEXT",
      "dependencies": [
        {
          "package": "S2S Integration Core",
          "versionNumber": "1.0.0.LATEST"
        },
        {
          "package": "S2S Integration UI",
          "versionNumber": "1.0.0.LATEST"
        }
      ]
    }
  ],
  "name": "S2S Integration Module",
  "namespace": "",
  "sfdcLoginUrl": "https://login.salesforce.com",
  "sourceApiVersion": "65.0"
}
```

**Note:** After creating packages, add package aliases to this file for easier reference.

### Alternative: Single Package Strategy

If multi-package complexity is not needed, you can consolidate everything into a single package:

```json
{
  "packageDirectories": [
    {
      "path": "force-app/core",
      "default": true,
      "package": "S2S Integration Module",
      "versionName": "ver 1.0",
      "versionNumber": "1.0.0.NEXT"
    }
  ],
  "name": "S2S Integration Module",
  "namespace": "",
  "sfdcLoginUrl": "https://login.salesforce.com",
  "sourceApiVersion": "65.0"
}
```

**Note:** With a single package strategy, you would combine core, UI, and admin components into the core directory.

---

## Package Strategy

### Package Naming Convention

**Format:** `s2s-{component}-{version}`

**Examples:**
- `s2s-core-1.0.0`
- `s2s-ui-1.0.0`
- `s2s-admin-1.0.0`

### Package Metadata Organization

#### Core Package Structure
```
force-app/core/
├── objects/
│   ├── Integration_Connection__c/
│   ├── Object_Mapping__c/
│   ├── Field_Mapping__c/
│   ├── Sync_Job__c/
│   └── Sync_Log__c/
├── classes/
│   ├── IntegrationConnectionManager.cls
│   ├── SyncOrchestrator.cls
│   ├── MappingEngine.cls
│   └── ErrorHandler.cls
├── triggers/
│   ├── IntegrationConnectionTrigger.trigger
│   └── SyncJobTrigger.trigger
├── metadataTypes/
│   ├── Mapping_Configuration.types
│   └── Sync_Rule.types
├── platformEvents/
│   ├── Sync_Status_Event.evt
│   └── Error_Event.evt
└── flows/
    └── SyncOrchestrationFlow.flow-meta.xml
```

#### UI Package Structure
```
force-app/ui/
├── lwc/
│   ├── connectionManager/
│   ├── mappingConfigurator/
│   ├── syncMonitor/
│   └── syncStatusDashboard/
├── aura/
│   └── (if needed for compatibility)
├── flexipages/
│   ├── Integration_Manager.app-meta.xml
│   └── Sync_Monitor.app-meta.xml
└── staticresources/
    ├── s2sStyles.css
    └── s2sImages.resource
```

#### Admin Package Structure
```
force-app/admin/
├── permissionsets/
│   ├── S2S_Integration_Administrator.permissionset-meta.xml
│   ├── S2S_Integration_User.permissionset-meta.xml
│   └── S2S_Integration_Viewer.permissionset-meta.xml
├── settings/
│   └── S2S_System_Configuration.settings-meta.xml
└── postInstallScripts/
    └── S2SPostInstallScript.cls
```

### Package Dependencies

#### External Dependencies
**None currently identified**

If external packages are needed in the future:
- Document in dependencies section
- Specify minimum version requirements
- Include in installation instructions

#### Internal Dependencies
```
s2s-core (no dependencies)
    ↓
s2s-ui (depends on s2s-core)
    ↓
s2s-admin (depends on s2s-core, s2s-ui)
```

### Package Installation Order

1. **s2s-core** - Base package, no dependencies
2. **s2s-ui** - Requires s2s-core
3. **s2s-admin** - Requires both s2s-core and s2s-ui

### Package Upgrade Strategy

#### Unlocked Package Upgrades
- **Patch Versions (1.0.1, 1.0.2):** Bug fixes, no breaking changes
- **Minor Versions (1.1.0, 1.2.0):** New features, backward compatible
- **Major Versions (2.0.0):** Breaking changes, may require migration

#### Upgrade Considerations
- **Backward Compatibility:** Maintain API compatibility within major versions
- **Data Migration:** Provide migration scripts for breaking changes
- **Deprecation Policy:** Deprecate features for one major version before removal
- **Testing:** Test upgrades in sandbox before production

---

## Versioning Strategy

### Semantic Versioning

**Format:** `MAJOR.MINOR.PATCH.BUILD`

- **MAJOR:** Breaking changes, incompatible API changes
- **MINOR:** New features, backward compatible
- **PATCH:** Bug fixes, backward compatible
- **BUILD:** Build number (auto-incremented)

### Version Examples

- `1.0.0.1` - Initial release
- `1.0.1.2` - Bug fix release
- `1.1.0.3` - New feature release (backward compatible)
- `2.0.0.4` - Major release (breaking changes)

### Version Management

#### Development
- Use `NEXT` for development: `1.0.0.NEXT`
- Auto-increment build numbers

#### Release Candidates
- Use `RC` suffix: `1.0.0.RC1`, `1.0.0.RC2`
- Test thoroughly before final release

#### Production Releases
- Use specific version: `1.0.0.1`
- Tag in version control
- Document release notes

### Version Compatibility Matrix

| Core Version | UI Version | Admin Version | Compatible |
|--------------|------------|---------------|------------|
| 1.0.0        | 1.0.0      | 1.0.0         | Yes        |
| 1.1.0        | 1.0.0      | 1.0.0         | Yes        |
| 1.1.0        | 1.1.0      | 1.0.0         | Yes        |
| 2.0.0        | 1.1.0      | 1.0.0         | No         |

**Rule:** Core package version must be >= UI and Admin package versions within same major version.

---

## Dependencies

### Salesforce Platform Dependencies

#### Required Features
- API access enabled
- Platform Events enabled
- Custom Metadata Types enabled
- Named Credentials enabled
- Connected Apps enabled

#### API Versions
- **Minimum:** API 60.0
- **Target:** API 65.0
- **Maximum:** Latest (tested with each release)

#### Governor Limits Considerations
- **API Calls:** Optimize to minimize API consumption
- **SOQL Queries:** Batch queries efficiently
- **DML Operations:** Use bulk DML patterns
- **CPU Time:** Optimize Apex code performance

### External Dependencies

**None currently**

### Internal Package Dependencies

See [Package Dependencies](#package-dependencies) section above.

---

## Distribution Strategy

### Distribution Channels

#### 1. AppExchange (If Managed Package)
- **Target Audience:** Public distribution
- **Requirements:**
  - Security review
  - Documentation
  - Support process
  - Marketing materials

#### 2. Private Distribution (Unlocked Package)
- **Target Audience:** Internal or specific customers
- **Methods:**
  - Package installation via CLI
  - Package installation via UI
  - DevOps pipeline integration

#### 3. Source Code Distribution
- **Target Audience:** Developers, open-source community
- **Methods:**
  - GitHub repository
  - Documentation
  - Installation scripts

### Installation Methods

#### CLI Installation
```bash
# Install core package
sf package install --package s2s-core@1.0.0-1 --target-org myorg

# Install UI package
sf package install --package s2s-ui@1.0.0-1 --target-org myorg --apex-compile package

# Install admin package
sf package install --package s2s-admin@1.0.0-1 --target-org myorg --apex-compile package
```

#### UI Installation
- Navigate to Setup → Installed Packages
- Upload package version
- Follow installation wizard

#### DevOps Integration
- CI/CD pipeline integration
- Automated testing
- Automated deployment

### Post-Installation Steps

1. **Assign Permission Sets**
   - S2S Integration Administrator
   - S2S Integration User (for end users)

2. **Configure Named Credentials**
   - Set up OAuth connections
   - Configure authentication

3. **Create Initial Mappings**
   - Configure object mappings
   - Configure field mappings

4. **Test Integration**
   - Verify org connections
   - Test sample syncs
   - Validate monitoring

---

## Development Workflow

### Package Development Workflow

#### 1. Create Scratch Org
```bash
sf org create scratch --definition-file config/project-scratch-def.json --alias s2s-dev
```

#### 2. Develop Features
- Create/update metadata in appropriate package directory
- Write unit tests (minimum 75% code coverage)
- Test in scratch org

#### 3. Create Package Version
```bash
# Create package version for core
sf package version create --package "S2S Integration Core" --wait 10

# Create package version for UI
sf package version create --package "S2S Integration UI" --wait 10

# Create package version for Admin
sf package version create --package "S2S Integration Admin" --wait 10
```

#### 4. Install in Test Org
```bash
# Install in test scratch org
sf package install --package s2s-core@1.0.0-1 --target-org test-org
sf package install --package s2s-ui@1.0.0-1 --target-org test-org
sf package install --package s2s-admin@1.0.0-1 --target-org test-org
```

#### 5. Integration Testing
- Run integration tests
- Validate functionality
- Performance testing

#### 6. Promote Package Version
```bash
# Promote to released
sf package version promote --package s2s-core@1.0.0-1
```

### Continuous Integration

#### CI Pipeline Steps
1. **Lint and Validate**
   ```bash
   sf project validate source
   ```

2. **Run Tests**
   ```bash
   sf apex run test --test-level RunLocalTests --result-format human --code-coverage
   ```

3. **Create Package Version** (on merge to main)
   ```bash
   sf package version create --package "S2S Integration Core" --wait 10
   ```

4. **Deploy to Test Org**
   ```bash
   sf package install --package s2s-core@1.0.0-1 --target-org test-org
   ```

5. **Run Integration Tests**

6. **Promote Package** (if tests pass)

### Package Version Promotion

#### Promotion Levels
1. **Beta:** For testing, not for production
2. **Released:** Production-ready, available for installation

#### Promotion Criteria
- All unit tests pass (minimum 75% coverage)
- Integration tests pass
- Security review completed
- Documentation updated
- Release notes prepared

---

## Best Practices

### Package Design
- **Single Responsibility:** Each package has a clear purpose
- **Minimal Dependencies:** Reduce coupling between packages
- **Backward Compatibility:** Maintain API compatibility
- **Documentation:** Document all public APIs

### Version Management
- **Semantic Versioning:** Follow versioning conventions
- **Release Notes:** Document all changes
- **Deprecation Policy:** Clear deprecation timeline
- **Migration Guides:** Provide migration instructions

### Testing
- **Unit Tests:** Minimum 75% code coverage
- **Integration Tests:** Test package interactions
- **Upgrade Tests:** Test upgrade scenarios
- **Performance Tests:** Validate performance

### Security
- **Security Review:** Complete security review before release
- **Permission Sets:** Least privilege principle
- **Data Access:** Respect sharing rules
- **API Security:** Secure all API endpoints

---

## Troubleshooting

### Common Issues

#### Package Installation Failures
- **Dependency Missing:** Ensure all dependencies installed first
- **API Version Mismatch:** Verify API version compatibility
- **Permission Issues:** Check user permissions
- **Governor Limits:** Review org limits

#### Package Upgrade Issues
- **Breaking Changes:** Review migration guide
- **Data Conflicts:** Resolve data conflicts manually
- **Customization Conflicts:** Review customizations

### Support Resources
- Package installation logs
- Salesforce CLI documentation
- Package version details
- Release notes

---

## Appendix

### Scratch Org Definition Files

#### Development
- `config/project-scratch-def.json` - Standard development org

#### Testing
- `config/integration-test-scratch-def.json` - Integration testing
- `config/performance-test-scratch-def.json` - Performance testing

### Package Version IDs
Maintain a registry of package version IDs for reference:
- Core Package: `04t...`
- UI Package: `04t...`
- Admin Package: `04t...`

### Related Documentation
- [Features](./FEATURES.md) - Feature requirements
- [Architecture Recommendations](./ARCHITECTURE_RECOMMENDATIONS.md) - System architecture
- [Security & Compliance](./SECURITY_COMPLIANCE.md) - Security requirements

---

*Last Updated: [Date]*
*Version: 1.0*

