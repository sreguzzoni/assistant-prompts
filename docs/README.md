# Terraform Infrastructure Documentation

This documentation provides comprehensive standards and guidelines for Terraform infrastructure projects.

## 📁 Documentation Structure

```
docs/
├── README.md                    # This file - documentation overview
├── coding-standards.md          # Core coding standards and rules
├── terraform-standards.md       # Terraform-specific standards
└── examples/
    ├── good-examples.md         # Best practices and examples
    └── anti-patterns.md         # What NOT to do
```

## 🎯 Quick Reference

### Repository Structure Requirements
- ✅ `.modules/` folder for custom modules
- ✅ `dev/` folder for development environment
- ✅ `staging/` folder for staging environment  
- ✅ `prod/` folder for production environment
- ✅ `bin/` folder with `plan` and `apply` scripts

### Module Requirements
Each module must have:
- ✅ `variables.tf` - Input variables
- ✅ `main.tf` - Resource definitions
- ✅ `outputs.tf` - Output values

### Environment Requirements
Each environment must have:
- ✅ `main.tf` - Main configuration
- ✅ `locals.tf` - Local values
- ✅ `providers.tf` - Provider configuration
- ✅ `versions.tf` - Version constraints
- ✅ `state.tf` - Backend configuration

### File Formatting Requirements
- ✅ Use `##################################################################` comments to separate modules
- ✅ Follow consistent naming conventions
- ✅ Include proper tagging strategy
- ✅ Resource naming pattern: `"${var.name}-${var.env}-${resource_type}"`
- ✅ Variable naming: module prefixes + common variables
- ✅ Locals structure: common + module-specific sections

### Execution Requirements
- ✅ `bin/plan` script for formatting and planning
- ✅ `bin/apply` script for formatting and applying
- ✅ Usage: `bin/plan $ENV` and `bin/apply $ENV`

## 📖 Documentation Files

### [coding-standards.md](./coding-standards.md)
Core standards covering:
- Repository structure requirements
- Module file requirements
- Environment file requirements
- File formatting rules with module comments
- Validation checklist

### [terraform-standards.md](./terraform-standards.md)
Terraform-specific standards including:
- File organization standards
- Comment standards with module separation
- Validation commands
- File structure validation checklist

### [examples/good-examples.md](./examples/good-examples.md)
Comprehensive examples showing:
- Proper repository structure
- Well-formatted module examples
- Environment configuration examples
- Best practices demonstration

### [examples/anti-patterns.md](./examples/anti-patterns.md)
Common mistakes to avoid:
- Bad repository structures
- Incorrect file organization
- Poor coding practices
- Security and configuration issues

## 🚀 Getting Started

1. **Read the Standards**: Start with `coding-standards.md` for core requirements
2. **Review Examples**: Check `good-examples.md` for proper implementation
3. **Avoid Anti-Patterns**: Review `anti-patterns.md` to understand what NOT to do
4. **Follow Validation**: Use the provided checklists to validate your code

## 🔍 Validation Commands

### Using Bin Scripts (Recommended)
```bash
# Format and plan environment
bin/plan dev
bin/plan staging
bin/plan prod

# Format and apply environment
bin/apply dev
bin/apply staging
bin/apply prod
```

### Direct Commands
```bash
# Format your Terraform code
terraform fmt -recursive

# Validate configuration
terraform validate

# Plan changes
terraform plan

# Apply changes (after review)
terraform apply
```

## 📋 Quick Checklist

Before committing any Terraform code:

- [ ] Repository has required folder structure (`.modules/`, `dev/`, `staging/`, `prod/`, `bin/`)
- [ ] Bin scripts exist (`bin/plan`, `bin/apply`) and are executable
- [ ] All modules have required files (`variables.tf`, `main.tf`, `outputs.tf`)
- [ ] All environments have required files (`main.tf`, `locals.tf`, `providers.tf`, `versions.tf`, `state.tf`)
- [ ] Files use `##################################################################` module separation comments
- [ ] Resource naming follows pattern: `"${var.name}-${var.env}-${resource_type}"`
- [ ] Variables use proper naming (module prefixes + common variables)
- [ ] Locals structure follows common + module-specific format
- [ ] All resources have minimum tags: project, account, env
- [ ] Code passes `terraform fmt` and `terraform validate`

## 🤝 Contributing

When adding new standards or examples:

1. Follow the existing documentation structure
2. Include both good examples and anti-patterns
3. Update this README if adding new documentation files
4. Ensure all examples are tested and validated
