# S2S Integration Module

A Salesforce-to-Salesforce (S2S) Integration Module that enables seamless data synchronization and integration between multiple Salesforce organizations.

## Overview

The S2S Integration Module provides a comprehensive solution for:
- Connecting and managing multiple Salesforce org integrations
- Monitoring org availability and health
- Configuring flexible object and field mappings
- Tracking and reporting sync status across objects and fields

## Documentation

Comprehensive documentation is available in the [`docs/`](./docs/) directory:

### Core Documentation
- **[Features](./docs/FEATURES.md)** - All features and capabilities (core and additional)
- **[Use Case Analysis](./docs/USE_CASE_ANALYSIS.md)** - Business use cases and scenarios
- **[Security & Compliance](./docs/SECURITY_COMPLIANCE.md)** - Security, compliance, and threat detection
- **[Architecture Recommendations](./docs/ARCHITECTURE_RECOMMENDATIONS.md)** - Recommended architecture and documentation structure
- **[Packaging Architecture](./docs/PACKAGING_ARCHITECTURE.md)** - Scratch org requirements and packaging strategy
- **[Executive Summary](./docs/EXECUTIVE_SUMMARY.md)** - High-level overview for stakeholders

See the [Documentation README](./docs/README.md) for a complete index.

## Core Features

1. **Integrate with Salesforce Orgs** - Establish and manage connections with multiple Salesforce organizations
2. **Monitor and Track Availability** - Continuous monitoring of connected orgs for availability and health
3. **Configurable Object and Field Mappings** - Flexible configuration for mapping objects and fields between orgs
4. **Monitor and Report Sync Status** - Comprehensive monitoring and reporting of synchronization status
5. **Manual Sync Operations** - On-demand sync for individual records, multiple records, or entire objects with related records

See [Features](./docs/FEATURES.md) for complete feature documentation including additional enhancements.

## Project Structure

```
├── docs/                    # Documentation
├── force-app/              # Salesforce source code (organized by package)
│   ├── core/               # Core Package - Integration logic and data model
│   │   ├── objects/        # Custom objects
│   │   ├── classes/        # Apex classes
│   │   ├── triggers/       # Apex triggers
│   │   ├── metadataTypes/   # Custom metadata types
│   │   ├── platformEvents/ # Platform events
│   │   └── flows/          # Flows
│   ├── ui/                 # UI Package - Lightning components
│   │   ├── lwc/            # Lightning Web Components
│   │   ├── aura/           # Aura components
│   │   ├── flexipages/     # Lightning pages
│   │   └── staticresources/# Static resources
│   └── admin/              # Admin Package - Permissions and setup
│       ├── permissionsets/ # Permission sets
│       ├── settings/       # Custom settings
│       └── postInstallScripts/ # Post-install scripts
├── config/                 # Configuration files
│   └── project-scratch-def.json  # Scratch org definition
├── manifest/               # Package manifest
└── sfdx-project.json       # Project configuration (package definitions)
```

See [force-app/README.md](./force-app/README.md) for detailed package structure information.

## Getting Started

1. Review the [Use Case Analysis](./docs/USE_CASE_ANALYSIS.md) to understand the business context
2. Read the [Features](./docs/FEATURES.md) document for core capabilities
3. Set up your development environment using [Packaging Architecture](./docs/PACKAGING_ARCHITECTURE.md) guide
4. Check [Architecture Recommendations](./docs/ARCHITECTURE_RECOMMENDATIONS.md) for implementation guidance

## Salesforce DX Resources

- [Salesforce Extensions Documentation](https://developer.salesforce.com/tools/vscode/)
- [Salesforce CLI Setup Guide](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_intro.htm)
- [Salesforce DX Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_intro.htm)
- [Salesforce CLI Command Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference.htm)
