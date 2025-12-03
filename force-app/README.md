# S2S Integration Module - Source Code

This directory contains the source code organized into separate packages for the S2S Integration Module.

## Package Structure

```
force-app/
├── core/          # Core Package - Base integration logic and data model
├── ui/            # UI Package - Lightning components and user interface
└── admin/         # Admin Package - Permission sets and administrative components
```

## Package Dependencies

```
core (no dependencies)
  ↓
ui (depends on core)
  ↓
admin (depends on core and ui)
```

## Development Workflow

1. **Core Package Development**
   - Work in `force-app/core/`
   - Contains custom objects, Apex classes, triggers, metadata types, platform events

2. **UI Package Development**
   - Work in `force-app/ui/`
   - Contains Lightning Web Components, Aura components, Lightning pages, static resources
   - Depends on core package

3. **Admin Package Development**
   - Work in `force-app/admin/`
   - Contains permission sets, custom settings, post-install scripts
   - Depends on both core and UI packages

## Package Creation

See [Packaging Architecture Documentation](../docs/PACKAGING_ARCHITECTURE.md) for detailed information on:
- Creating package versions
- Package dependencies
- Versioning strategy
- Distribution strategy

## Notes

- The `main/default/` directory is kept for backward compatibility but is not used for packaging
- Each package directory follows Salesforce DX metadata structure
- Package configuration is defined in `sfdx-project.json`

