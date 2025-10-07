# Terraform Infrastructure Standards & Documentation

This repository contains comprehensive documentation and standards for Terraform infrastructure projects, designed to guide AI assistants (like Cursor) in maintaining consistent, well-structured infrastructure code.

## 🎯 **For AI Assistants (Cursor)**

This repository provides standards and examples for Terraform infrastructure projects. When working with Terraform code, always reference the documentation in the `docs/` folder to ensure compliance with established patterns.

### **Quick Start for AI:**
1. **Read**: `docs/README.md` - Overview and quick reference
2. **Follow**: `docs/coding-standards.md` - Core requirements and patterns
3. **Reference**: `docs/examples/good-examples.md` - Proper implementation examples
4. **Avoid**: `docs/examples/anti-patterns.md` - Common mistakes to avoid

## 📁 **Repository Structure**

```
├── docs/                    # 📚 Complete documentation
│   ├── README.md           # Documentation overview
│   ├── coding-standards.md # Core standards and requirements
│   ├── terraform-standards.md # Terraform-specific standards
│   └── examples/
│       ├── good-examples.md # Best practices and examples
│       └── anti-patterns.md # What NOT to do
├── infra-example/          # 🏗️ Real infrastructure example
│   ├── modules/            # Custom Terraform modules
│   ├── dev/                # Development environment
│   └── ...
└── README.md               # This file
```

## 🚨 **Critical Requirements for AI Assistants**

**MANDATORY PATTERNS TO ENFORCE:**
- **Repository Structure**: `modules/`, `dev/`, `staging/`, `prod/`, `bin/`
- **Module Files**: `variables.tf`, `main.tf`, `outputs.tf` (ALL REQUIRED)
- **Environment Files**: `main.tf`, `locals.tf`, `provider.tf`, `versions.tf`, `state.tf` (ALL REQUIRED)
- **Comment Format**: `#############################################` for module separation
- **Resource Naming**: `"${var.name}-${var.env}-${resource_type}"`
- **Variable Naming**: Module prefixes (e.g., `alb_`, `ecs_`, `dynamodb_`)
- **Locals Structure**: `common` + module-specific sections
- **Minimum Tags**: `Name`, `Environment`, `Project`

## 📖 **Documentation Files**

| File | Purpose | For AI Assistants |
|------|---------|-------------------|
| [`docs/README.md`](docs/README.md) | Complete documentation overview | Start here for full context |
| [`docs/coding-standards.md`](docs/coding-standards.md) | Core coding standards | **CRITICAL** - Enforce these patterns |
| [`docs/terraform-standards.md`](docs/terraform-standards.md) | Terraform-specific standards | Reference for file organization |
| [`docs/examples/good-examples.md`](docs/examples/good-examples.md) | Best practices examples | Use as templates for suggestions |
| [`docs/examples/anti-patterns.md`](docs/examples/anti-patterns.md) | Common mistakes | Identify and fix these issues |

## 🏗️ **Infrastructure Example**

The `infra-example/` folder contains a real-world Terraform infrastructure example that demonstrates all the standards and patterns documented in the `docs/` folder. Use this as a reference for:

- Proper module organization
- Environment structure
- File naming conventions
- Comment formatting
- Resource tagging
- Variable organization

## 🤖 **AI Assistant Usage Guidelines**

When working with Terraform code, AI assistants should:

1. **Always Check Structure**: Verify proper folder structure and required files
2. **Enforce Standards**: Apply naming conventions and comment formats
3. **Reference Examples**: Use good examples as templates
4. **Identify Anti-Patterns**: Flag and suggest fixes for common mistakes
5. **Validate Compliance**: Use checklists before suggesting changes

## 🔍 **Quick Validation Checklist**

Before suggesting any Terraform changes, verify:
- [ ] Repository has required folder structure
- [ ] All required files are present
- [ ] Comment format follows standards
- [ ] Resource naming follows conventions
- [ ] Proper tagging is applied
- [ ] Code passes `terraform fmt` and `terraform validate`

## 📚 **Getting Started**

1. **For AI Assistants**: Start with `docs/README.md` for complete context
2. **For Developers**: Review `docs/coding-standards.md` for requirements
3. **For Examples**: Check `infra-example/` for real-world implementation
4. **For Validation**: Use checklists in documentation files

---

**This repository is designed to help AI assistants maintain consistent, high-quality Terraform infrastructure code. Always reference the documentation when working with Terraform projects.**
