# 🏗️ BlackRoad Terraform Module Registry

[![CI](https://github.com/BlackRoad-OS/blackroad-terraform-modules/actions/workflows/ci.yml/badge.svg)](https://github.com/BlackRoad-OS/blackroad-terraform-modules/actions/workflows/ci.yml)
[![Python](https://img.shields.io/badge/python-3.10%2B-blue)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-Proprietary-red)](./LICENSE)
[![Modules](https://img.shields.io/badge/built--in%20modules-8-green)](#built-in-modules)
[![Providers](https://img.shields.io/badge/providers-aws%20%7C%20gcp%20%7C%20kubernetes-orange)](#built-in-modules)

> Production-grade Python registry for managing, generating, and validating Terraform modules.  
> Store modules in SQLite, render HCL templates with variable substitution, validate syntax,
> export plans, and generate markdown documentation — all from the CLI or Python API.

---

## Table of Contents

- [Features](#-features)
- [Built-in Modules](#-built-in-modules)
- [Quick Start](#-quick-start)
- [Installation](#-installation)
- [CLI Reference](#-cli-reference)
  - [list](#list)
  - [register](#register)
  - [generate](#generate)
  - [validate](#validate)
  - [plan](#plan)
  - [search](#search)
  - [docs](#docs)
  - [stats](#stats)
- [Python API](#-python-api)
  - [TerraformRegistry](#terraformregistry)
  - [TerraformVariable](#terraformvariable)
  - [TerraformOutput](#terraformoutput)
  - [TerraformExample](#terraformexample)
  - [TerraformModule](#terraformmodule)
  - [ValidationResult](#validationresult)
- [HCL Template Format](#-hcl-template-format)
- [Validation Rules](#-validation-rules)
- [Database](#-database)
- [Project Structure](#-project-structure)
- [Running Tests](#-running-tests)
- [Contributing](#-contributing)
- [Security](#-security)
- [License](#-license)

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 📦 **Module Registry** | Register and version Terraform modules in a local SQLite database |
| 🔧 **HCL Generation** | Render HCL templates with `${var.name}` substitution and default merging |
| ✅ **Validation** | Static analysis of HCL: brace balance, resource syntax, interpolation checks |
| 📋 **Plan Export** | Human-readable Terraform plan showing resources to be created |
| 📚 **Docs Generator** | Auto-generate Markdown docs from module metadata |
| 🔍 **Search** | Full-text search across name, description, provider, and tags |
| 📊 **Statistics** | Module counts by provider, most-downloaded leaderboard |
| 🎨 **Rich CLI** | Beautiful terminal output with tables, syntax highlighting, and panels |

---

## 📦 Built-in Modules

| Name | Provider | Resource Type | Version | Tags |
|------|----------|---------------|---------|------|
| `aws_ec2_instance` | `aws` | `aws_instance` | 2.1.0 | compute, vm |
| `aws_s3_bucket` | `aws` | `aws_s3_bucket` | 3.0.1 | storage |
| `aws_rds_instance` | `aws` | `aws_db_instance` | 1.4.2 | database |
| `aws_vpc` | `aws` | `aws_vpc` | 2.0.0 | networking |
| `gcp_gce_instance` | `gcp` | `google_compute_instance` | 1.2.0 | compute, vm |
| `gcp_gcs_bucket` | `gcp` | `google_storage_bucket` | 1.1.0 | storage |
| `kubernetes_deployment` | `kubernetes` | `kubernetes_deployment` | 1.3.0 | k8s, container |
| `kubernetes_service` | `kubernetes` | `kubernetes_service` | 1.1.0 | k8s, networking |

All 8 modules are seeded automatically on first run.

---

## 🚀 Quick Start

```bash
# Install dependencies
pip install typer rich

# List all built-in modules
python terraform_modules.py list

# Generate HCL for an EC2 instance
python terraform_modules.py generate aws_ec2_instance \
  --var name=my-server \
  --var ami_id=ami-0abcdef1234567890 \
  --var instance_type=t3.small

# Validate an HCL file
python terraform_modules.py validate path/to/main.tf

# Show module documentation
python terraform_modules.py docs aws_s3_bucket

# Search for storage-related modules
python terraform_modules.py search storage

# Registry statistics
python terraform_modules.py stats
```

---

## 📥 Installation

### Requirements

| Dependency | Version | Purpose |
|------------|---------|---------|
| Python | ≥ 3.10 | Runtime |
| `typer[all]` | ≥ 0.9.0 | CLI framework |
| `rich` | ≥ 13.0.0 | Terminal output |

### Install

```bash
# Clone the repository
git clone https://github.com/BlackRoad-OS/blackroad-terraform-modules.git
cd blackroad-terraform-modules

# Install runtime dependencies
pip install typer[all] rich

# (Optional) Install all dev dependencies
pip install -r requirements.txt
```

The registry database is created automatically at `~/.blackroad/terraform-modules.db` on first run.

---

## 📖 CLI Reference

### `list`
```
python terraform_modules.py list [--provider PROVIDER] [--resource RESOURCE]
```
List all registered modules, optionally filtered by provider or resource type.

**Options:**
| Flag | Short | Description |
|------|-------|-------------|
| `--provider` | `-p` | Filter by cloud provider (`aws`, `gcp`, `kubernetes`, …) |
| `--resource` | `-r` | Filter by Terraform resource type |

---

### `register`
```
python terraform_modules.py register NAME PROVIDER RESOURCE_TYPE TEMPLATE_FILE \
  [--description DESC] [--version VERSION]
```
Register a new module from an HCL template file.

---

### `generate`
```
python terraform_modules.py generate MODULE_NAME_OR_ID \
  --var key=value [--var key2=value2 ...] [--out output.tf]
```
Render a module's HCL template with variable substitution.

---

### `validate`
```
python terraform_modules.py validate TEMPLATE_FILE
```
Validate HCL syntax with static analysis.

---

### `plan`
```
python terraform_modules.py plan MODULE_NAME_OR_ID --var key=value [...]
```
Export a human-readable Terraform plan.

---

### `search`
```
python terraform_modules.py search QUERY
```
Search modules by name, description, provider, or tags.

---

### `docs`
```
python terraform_modules.py docs MODULE_NAME_OR_ID
```
Generate and display Markdown documentation for a module.

---

### `stats`
```
python terraform_modules.py stats
```
Display registry statistics: module counts and download leaderboard.

---

## 🐍 Python API

### `TerraformRegistry`

The main entry point. Manages the SQLite database and all module operations.

```python
from terraform_modules import TerraformRegistry

registry = TerraformRegistry()                            # default DB path
registry = TerraformRegistry(db_path=Path("custom.db"))  # custom path
```

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `register_module` | `(name, provider, resource_type, hcl_template, variables, outputs, ...)` | `TerraformModule` | Register a new module |
| `get_module` | `(module_id_or_name)` | `TerraformModule` | Fetch a module by ID or name |
| `list_modules` | `(provider_filter?, resource_type_filter?)` | `list[TerraformModule]` | List/filter modules |
| `generate_tf` | `(module_id_or_name, vars)` | `str` | Render HCL with variable substitution |
| `validate_hcl` | `(hcl_string)` | `ValidationResult` | Static HCL analysis |
| `export_plan` | `(module_id_or_name, vars)` | `str` | Human-readable plan export |
| `generate_docs` | `(module_id_or_name)` | `str` | Auto-generate Markdown docs |
| `search` | `(query)` | `list[TerraformModule]` | Full-text search |
| `get_stats` | `()` | `dict` | Registry statistics |
| `delete_module` | `(module_id_or_name)` | `bool` | Delete a module |

---

### `TerraformVariable`

```python
from terraform_modules import TerraformVariable

TerraformVariable(
    name="instance_type",
    type="string",           # string | number | bool | list | map | object | any
    description="EC2 type",
    default="t3.micro",
    required=False,
    sensitive=False,
)
```

---

### `TerraformOutput`

```python
from terraform_modules import TerraformOutput

TerraformOutput(
    name="instance_id",
    description="EC2 Instance ID",
    value_expression="aws_instance.main.id",
    sensitive=False,
)
```

---

### `TerraformExample`

```python
from terraform_modules import TerraformExample

TerraformExample(
    title="Basic web server",
    description="A minimal t3.small web server.",
    hcl_code='module "web" { ... }',
)
```

---

### `TerraformModule`

Dataclass holding all module metadata. Key fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | `str` | UUID primary key |
| `name` | `str` | Unique module name |
| `provider` | `str` | Cloud provider |
| `resource_type` | `str` | Terraform resource type |
| `version` | `str` | Semver version string |
| `hcl_template` | `str` | Template with `${var.*}` placeholders |
| `variables` | `list[TerraformVariable]` | Input variable definitions |
| `outputs` | `list[TerraformOutput]` | Output value definitions |
| `examples` | `list[TerraformExample]` | Usage examples |
| `tags` | `list[str]` | Searchable tags |
| `download_count` | `int` | Number of times generated |

Helper method: `module.bump_version("patch" | "minor" | "major") -> str`

---

### `ValidationResult`

```python
result = registry.validate_hcl(hcl)
result.valid     # bool
result.errors    # list[str]  – fatal issues
result.warnings  # list[str]  – non-fatal advisories
```

---

### Full Python API Example

```python
from terraform_modules import (
    TerraformRegistry,
    TerraformVariable,
    TerraformOutput,
    TerraformExample,
)

registry = TerraformRegistry()

# List all modules
modules = registry.list_modules()

# Filter by provider
aws_modules = registry.list_modules(provider_filter="aws")

# Generate HCL
hcl = registry.generate_tf("aws_s3_bucket", {
    "bucket_name": "my-unique-bucket-name-2024",
    "environment": "production",
})
print(hcl)

# Validate HCL
result = registry.validate_hcl(hcl)
print(result)  # Valid: True

# Export plan
plan = registry.export_plan("aws_ec2_instance", {
    "name":          "web-prod",
    "ami_id":        "ami-0abcdef1234567890",
    "instance_type": "t3.medium",
    "subnet_id":     "subnet-12345678",
})
print(plan)

# Generate docs
docs = registry.generate_docs("aws_rds_instance")
print(docs)

# Register a custom module
mod = registry.register_module(
    name="my_custom_bucket",
    provider="aws",
    resource_type="aws_s3_bucket",
    hcl_template='''resource "aws_s3_bucket" "${var.name}" {
  bucket = "${var.name}"
}''',
    variables=[
        TerraformVariable("name", "string", "Bucket name"),
    ],
    outputs=[
        TerraformOutput("arn", "Bucket ARN", "aws_s3_bucket.${var.name}.arn"),
    ],
    description="My custom S3 bucket module",
    version="1.0.0",
)
print(f"Registered: {mod.id}")

# Search
results = registry.search("kubernetes")

# Statistics
stats = registry.get_stats()
print(stats["total_modules"])       # e.g. 9
print(stats["by_provider"])         # {"aws": 4, "gcp": 2, ...}

# Delete
registry.delete_module("my_custom_bucket")
```

---

## 📐 HCL Template Format

Templates use `${var.name}` placeholders that are replaced during generation:

```hcl
resource "aws_instance" "${var.name}" {
  ami           = "${var.ami_id}"
  instance_type = "${var.instance_type}"

  tags = {
    Name        = "${var.name}"
    Environment = "${var.environment}"
  }

  root_block_device {
    volume_size = ${var.root_volume_size}
    encrypted   = true
  }
}
```

**Rules:**
- Use `${var.<name>}` for string/interpolated values
- Use `${var.<name>}` (no quotes) for number/bool attributes
- Variables with `required=True` and no `default` must be supplied at generation time
- All other variables fall back to their `default` value

---

## ✅ Validation Rules

The `validate_hcl()` method performs these checks:

| Check | Severity | Description |
|-------|----------|-------------|
| Balanced `{ }` | **Error** | Every opening brace must have a closing brace |
| Balanced `[ ]` | **Error** | Every opening bracket must have a closing bracket |
| Balanced `( )` | **Error** | Every opening parenthesis must have a closing one |
| Resource block labels | **Error** | `resource` blocks must have two string labels |
| Empty template | **Error** | Template must not be empty |
| No block found | **Warning** | Template has no `resource`/`data`/`module` block |
| Unknown interpolation | **Warning** | `${something.*}` — not `var.`, `local.`, `module.`, `data.` |
| Escaped dollar | **Warning** | `$${` is only needed for literal `$` in strings |

---

## 🗄️ Database

Modules are stored in a SQLite database at `~/.blackroad/terraform-modules.db`.
The schema is auto-created on first run.

```
~/.blackroad/
└── terraform-modules.db   # SQLite module registry
```

**Indexes:**

| Index | Column | Type |
|-------|--------|------|
| `idx_provider` | `provider` | Standard |
| `idx_resource_type` | `resource_type` | Standard |
| `idx_name` | `name` | Unique |

To reset the registry: `rm ~/.blackroad/terraform-modules.db`

---

## 📁 Project Structure

```
blackroad-terraform-modules/
├── terraform_modules.py          # Core implementation (dataclasses, registry, CLI)
├── tests/
│   ├── __init__.py
│   └── test_terraform_modules.py # 30 pytest tests
├── .github/
│   └── workflows/
│       └── ci.yml                # CI: lint (ruff) + test matrix (3.10/3.11/3.12) + security
├── requirements.txt              # Runtime + dev dependencies
├── LICENSE
└── README.md
```

---

## 🧪 Running Tests

```bash
pip install pytest pytest-cov typer rich
pytest tests/ -v --cov=terraform_modules --cov-report=term-missing
```

Expected: **30 tests passing** across registration, generation, validation, listing, plan export, search, docs, stats, and delete.

---

## 🤝 Contributing

1. Fork the repository and create a feature branch.
2. Make your changes and add/update tests in `tests/test_terraform_modules.py`.
3. Ensure all 30 tests pass: `pytest tests/ -v`
4. Run the linter: `ruff check terraform_modules.py tests/`
5. Open a pull request against `main`.

Please follow existing code style and keep changes focused and minimal.

---

## 🔒 Security

See [SECURITY.md](./SECURITY.md) for the vulnerability disclosure policy.

To report a security issue, please **do not** open a public GitHub issue.  
Contact the security team directly per the instructions in `SECURITY.md`.

---

## 📄 License

Proprietary — © BlackRoad OS, Inc. All rights reserved.  
See [LICENSE](./LICENSE) for full terms.

