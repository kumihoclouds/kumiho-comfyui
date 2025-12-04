# Kumiho ComfyUI Custom Nodes 🦊

> **The Revolution Begins** - Connect your ComfyUI workflows with Kumiho Cloud asset management system.

[![ComfyUI](https://img.shields.io/badge/ComfyUI-Custom%20Nodes-blue)](https://github.com/comfyanonymous/ComfyUI)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Kumiho](https://img.shields.io/badge/Kumiho-Cloud-orange)](https://kumiho.io)

## Overview

Kumiho ComfyUI Nodes seamlessly integrate ComfyUI with the Kumiho Cloud asset management platform. This enables:

- **Asset Versioning**: Automatically version your generated images and workflows
- **Dependency Tracking**: Track lineage and relationships between assets
- **Team Collaboration**: Share assets across your team with proper version control
- **Impact Analysis**: Understand downstream dependencies before making changes

## Installation

### Via ComfyUI Manager (Recommended)

1. Open ComfyUI
2. Go to Manager > Install Custom Nodes
3. Search for "Kumiho"
4. Click Install

### Manual Installation

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/kumihoclouds/kumiho-comfyui.git
pip install -r kumiho-comfyui/requirements.txt
```

### Via pip

```bash
pip install kumiho-comfyui
```

## Authentication Setup (Required)

Before using Kumiho nodes, you must authenticate with Kumiho Cloud:

```bash
# Install the Kumiho CLI
pip install kumiho-cli

# Login to Kumiho Cloud (opens browser for OAuth)
kumiho-auth login

# Verify authentication is working
kumiho-auth status
```

This creates an `authentication.json` file in your home directory (`~/.kumiho/`) that the SDK uses automatically.

> **Note**: You can also refresh your token anytime with `kumiho-auth refresh`

## Available Nodes

### Input/Output Nodes (`Kumiho/IO`)

#### 🔹 Kumiho Load Asset
Load assets from Kumiho Cloud using kref:// URIs.

**Inputs:**
- `kref_uri`: The kref:// URI of the asset (e.g., `kref://project/space/asset.texture`)
- `revision`: Revision to load ("latest", version number, or tag like "published")
- `api_endpoint`: Kumiho API endpoint (optional)

**Outputs:**
- `image`: The loaded image tensor
- `metadata`: JSON metadata string

#### 🔹 Kumiho Save Asset
Save generated images to Kumiho Cloud as new revisions.

**Inputs:**
- `images`: Image tensor to save
- `item_kref`: The kref:// URI of the item to create a revision for
- `metadata`: JSON metadata to attach
- `auto_tag`: Whether to automatically tag the revision
- `tag_name`: Tag to apply if auto_tag is enabled

**Outputs:**
- `revision_kref`: The kref:// URI of the created revision

### Graph Nodes (`Kumiho/Graph`)

#### 🔹 Kumiho Create Edge
Create dependency relationships between revisions.

**Inputs:**
- `source_kref`: Source revision kref
- `target_kref`: Target revision kref
- `edge_type`: Relationship type (DEPENDS_ON, DERIVED_FROM, REFERENCED, CONTAINS, CREATED_FROM)

**Outputs:**
- `edge_id`: The created edge identifier

#### 🔹 Kumiho Tag Revision
Apply tags to revisions for easy retrieval.

**Inputs:**
- `revision_kref`: The revision to tag
- `tag`: Tag name (e.g., "approved", "published", "wip")

**Outputs:**
- `success`: Boolean indicating success

#### 🔹 Kumiho Get Dependencies
Get all dependencies of a revision.

**Inputs:**
- `revision_kref`: The revision to query
- `max_depth`: Maximum traversal depth (1-20)
- `edge_types`: Comma-separated edge types to filter

**Outputs:**
- `dependencies_json`: JSON with dependency information

### Search Nodes (`Kumiho/Search`)

#### 🔹 Kumiho Search Items
Search for items across Kumiho projects and spaces.

**Inputs:**
- `query`: Search query (supports wildcards)
- `context_filter`: Filter by project/space path
- `kind_filter`: Filter by item kind (model, texture, workflow, etc.)

**Outputs:**
- `results_json`: JSON with matching items

## Configuration

### Authentication (Recommended)

The recommended way to authenticate is via the Kumiho CLI:

```bash
# Login (opens browser for OAuth)
kumiho-auth login

# Check authentication status
kumiho-auth status

# Refresh token if expired
kumiho-auth refresh
```

This creates `~/.kumiho/authentication.json` which the SDK discovers automatically.

### Environment Variables (Alternative)

You can also use environment variables:

```bash
# Alternative to CLI authentication
export KUMIHO_ACCESS_TOKEN="your_access_token_here"

# Optional: Override default API endpoint
export KUMIHO_API_ENDPOINT="https://api.kumiho.io"
```

### Node-Level Configuration

Each node accepts an `api_endpoint` input for per-node configuration if needed.

## Usage Examples

### Basic Workflow: Load, Process, Save

```
[Kumiho Load Asset] -> [Your Processing Nodes] -> [Kumiho Save Asset]
         |                                               |
         v                                               v
    kref://project/textures/input.png      kref://project/textures/output.png
```

### Dependency Tracking Workflow

```
[Kumiho Load Asset] -> [Processing] -> [Kumiho Save Asset] -> [Kumiho Create Edge]
         |                                    |                       |
         v                                    v                       v
    source_kref                         target_kref            DERIVED_FROM
```

## kref:// URI Format

Kumiho uses kref:// URIs to identify assets:

```
kref://project/space/item.kind?r=revision

Examples:
- kref://myproject/characters/hero.model
- kref://myproject/textures/skin.texture?r=latest
- kref://myproject/renders/final.image?r=5
- kref://myproject/workflows/processing.workflow?r=published
```

## API Documentation

For full API documentation, visit [docs.kumiho.io](https://docs.kumiho.io).

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

## Support

- 📚 [Documentation](https://docs.kumiho.io)
- 💬 [Discord Community](https://discord.gg/kumiho)
- 🐛 [Issue Tracker](https://github.com/kumihoclouds/kumiho-comfyui/issues)
- 📧 [Email Support](mailto:support@kumiho.io)

## License

MIT License - see [LICENSE](LICENSE) for details.

---

Made with 🦊 by [Kumiho Clouds](https://kumiho.io)
