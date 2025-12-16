# 🦊 Kumiho ComfyUI Nodes - Implementation Plan

## Status: ✅ Phase 1-6 Complete

**Last Updated**: January 2025

### Completed
- ✅ Core node implementations (KumihoRegister, KumihoLoadAsset, KumihoSaveImage, KumihoSaveVideo)
- ✅ Workflow parser with dependency extraction
- ✅ kref URI parsing and building utilities
- ✅ Frontend JavaScript extension with sidebar panel
- ✅ API client with resolve_kref and impact analysis
- ✅ Unit tests for workflow parser
- ✅ **KumihoAssetCatalog** - Cached catalog for dropdown population
- ✅ **Dropdown mode** for KumihoLoadAsset (browse projects/spaces/items)
- ✅ **API endpoints** for dynamic catalog refresh (/kumiho/catalog/*)
- ✅ **Frontend asset browser** with search and catalog integration
- ✅ **KumihoSearchItems** - Search items using `kumiho.item_search()` with iterative output
- ✅ **KumihoSaveImage/Video** - Optional `file_path` input for custom save locations
- ✅ **Inline video preview** - Videos display in ComfyUI output panel
- ✅ **Lineage tracking** - Automatic DERIVED_FROM edges between assets
- ✅ **Python SDK v0.4.2** - Item.project, Item.space properties; Kref.get_project() method

### Remaining
- ⏳ ComfyUI model loader wrappers (KumihoCheckpointLoader, etc.)
- ⏳ Batch registration optimization
- ⏳ Advanced caching strategies

---

## Executive Summary

This document outlines the implementation plan for two core Kumiho ComfyUI custom nodes:
1. **KumihoRegister** - Auto-register outputs and dependencies to Kumiho Cloud
2. **KumihoLoadAsset** - Load assets from Kumiho Cloud using kref URIs or dropdown selection

---

## 📋 Node Specifications

### 1. KumihoRegister (Output Node)

**Purpose**: Automatically capture ComfyUI workflow outputs and register them with full lineage tracking to Kumiho Cloud.

**Key Features**:
- Parse workflow JSON to extract all dependencies (models, LoRAs, input images, etc.)
- Auto-detect and register unregistered assets using `get_artifacts_by_location`
- Create output artifact with workflow JSON as metadata for reproducibility
- Create lineage edges: `CREATED_FROM`, `USED_MODEL`, `USED_LORA`, `USED_INPUT`
- Support manual category selection (Project/Space hierarchy)
- Default project: `ComfyUI@{TenantName}`

**Inputs**:
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| images | IMAGE | Yes | Output images to register |
| artifact_name | STRING | Yes | Name for the output artifact |
| category_path | STRING | Yes | Space path (e.g., "outputs/portraits") |
| auto_register_deps | BOOLEAN | Optional | Auto-detect & register dependencies (default: True) |
| workflow_json | STRING | Hidden | Auto-captured workflow JSON |

**Outputs**:
| Output | Type | Description |
|--------|------|-------------|
| revision_kref | STRING | kref URI of created revision |
| lineage_summary | STRING | JSON summary of created lineage |

---

### 2. KumihoLoadAsset

**Purpose**: Load registered Kumiho assets (LoRA, Checkpoints, Images) using kref URIs or dropdown-based browsing.

**Key Features**:
- **Two input modes**: Direct kref URI entry OR dropdown browsing
- Browse projects, spaces (as categories), and items via dropdown menus
- Search items across all projects with type filtering
- Support multiple asset types: LoRA, Checkpoint, Image, ControlNet, etc.
- Resolve kref URIs to local file paths (BYO Storage model)
- Support revision tags (latest, published, specific version)
- Cache resolved paths for performance

**Input Modes**:
1. **dropdown** - Browse and select from Project → Space → Item dropdowns
2. **kref_uri** - Enter kref URI directly

**kref URI Format**:
```
kref://ComfyUI@{TenantName}/{category}/{subcategory}/{name}.{kind}?r={revision}&a={artifact_type}

Examples:
- kref://ComfyUI@KumihoClouds/lora/flux/Eye-Lora.lora?r=1&a=lora
- kref://ComfyUI@KumihoClouds/checkpoint/flux/flux1-schnell.safetensors?r=latest&a=fp8
- kref://ComfyUI@KumihoClouds/checkpoint/flux/flux1-schnell.safetensors?r=latest&a=fp16
```

**Inputs**:
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| input_mode | COMBO | Yes | Selection mode: "dropdown" or "kref_uri" |
| asset_type | COMBO | Yes | Type: checkpoint, lora, image, controlnet, etc. |
| project | COMBO | Optional | Project dropdown (dropdown mode) |
| space | COMBO | Optional | Space dropdown (dropdown mode) |
| item_name | STRING | Optional | Item name (dropdown mode) |
| kref_uri | STRING | Optional | Kumiho kref URI (kref_uri mode) |
| revision | STRING | Optional | Revision (default: "latest") |
| artifact_variant | STRING | Optional | Specific artifact variant (fp8, fp16, etc.) |
| fallback_path | STRING | Optional | Local fallback path if kref resolution fails |

**Outputs**:
| Output | Type | Description |
|--------|------|-------------|
| file_path | STRING | Resolved local file path |
| metadata | STRING | Asset metadata JSON |

---

## 🗂️ Project Structure

```
kumiho-comfyUI/
├── __init__.py                      # Main entry point
├── pyproject.toml                   # Package config
├── requirements.txt                 # Dependencies
├── README.md                        # Documentation
│
├── kumiho_nodes/
│   ├── __init__.py                  # Node exports
│   ├── nodes.py                     # All node definitions
│   │
│   ├── core/
│   │   ├── __init__.py
│   │   ├── api_client.py            # Kumiho API client wrapper
│   │   ├── auth.py                  # Authentication handling
│   │   └── config.py                # Configuration management
│   │
│   ├── parsers/
│   │   ├── __init__.py
│   │   ├── workflow_parser.py       # ComfyUI workflow JSON parser
│   │   ├── metadata_parser.py       # PNG metadata extractor
│   │   └── path_resolver.py         # File path to kref resolver
│   │
│   ├── registry/
│   │   ├── __init__.py
│   │   ├── asset_detector.py        # Auto-detect asset types
│   │   ├── dependency_scanner.py    # Scan workflow for dependencies
│   │   └── smart_register.py        # Smart registration logic
│   │
│   ├── lineage/
│   │   ├── __init__.py
│   │   ├── edge_builder.py          # Build lineage edges
│   │   └── graph_builder.py         # Build complete lineage graph
│   │
│   └── utils/
│       ├── __init__.py
│       ├── kref.py                  # kref URI parsing/building
│       ├── image_utils.py           # Image tensor utilities
│       └── cache.py                 # Caching utilities
│
├── web/
│   └── js/
│       ├── kumiho.js                # Main extension
│       ├── kumiho_browser.js        # Asset browser UI
│       └── kumiho_settings.js       # Settings panel
│
└── docs/
    ├── PRD.md                       # Product requirements
    └── implementation_plan.md       # This file
```

---

## 🚀 Implementation Phases

### Phase 1: Foundation (Week 1)
**Goal**: Establish core infrastructure and basic node structure

#### Checklist:
- [ ] **1.1 Project Setup**
  - [ ] Initialize proper Python package structure
  - [ ] Set up Kumiho Python SDK integration
  - [ ] Configure authentication flow (API key / OAuth)
  - [ ] Create configuration management system

- [ ] **1.2 Core API Client**
  - [ ] Wrap Kumiho Python SDK for ComfyUI context
  - [ ] Implement connection pooling and error handling
  - [ ] Add retry logic with exponential backoff
  - [ ] Create async/sync API method wrappers

- [ ] **1.3 kref URI System**
  - [ ] Implement kref parser with full query parameter support
  - [ ] Implement kref builder for various asset types
  - [ ] Support artifact type variants (fp8, fp16, etc.)
  - [ ] Add validation and normalization

- [ ] **1.4 Basic Node Skeleton**
  - [ ] Create KumihoRegister node class with INPUT_TYPES
  - [ ] Create KumihoLoadAsset node class with INPUT_TYPES
  - [ ] Register nodes in NODE_CLASS_MAPPINGS
  - [ ] Test basic node loading in ComfyUI

---

### Phase 2: Workflow Parser & Dependency Scanner (Week 2)
**Goal**: Parse ComfyUI workflows and detect all dependencies

#### Checklist:
- [ ] **2.1 Workflow JSON Parser**
  - [ ] Parse ComfyUI workflow JSON structure
  - [ ] Extract node configurations
  - [ ] Identify file path inputs across all node types
  - [ ] Handle nested and complex workflow structures

- [ ] **2.2 Dependency Scanner**
  - [ ] Detect checkpoint/model paths from loader nodes
  - [ ] Detect LoRA paths from LoRA loader nodes
  - [ ] Detect input image paths
  - [ ] Detect ControlNet model paths
  - [ ] Detect VAE paths
  - [ ] Detect upscale model paths
  - [ ] Detect any other file-based inputs

- [ ] **2.3 Asset Type Classifier**
  - [ ] Map file extensions to asset kinds
  - [ ] Map node types to expected asset categories
  - [ ] Create asset type hierarchy mapping:
    ```
    checkpoints/     → kind: checkpoint
    loras/           → kind: lora
    controlnet/      → kind: controlnet
    vae/             → kind: vae
    upscale_models/  → kind: upscaler
    embeddings/      → kind: embedding
    clip_vision/     → kind: clip_vision
    ...
    ```

- [ ] **2.4 PNG Metadata Extractor**
  - [ ] Extract embedded workflow from PNG
  - [ ] Extract prompt/negative prompt
  - [ ] Extract generation parameters (seed, cfg, steps, sampler)
  - [ ] Handle various metadata formats

---

### Phase 3: Smart Registration System (Week 3)
**Goal**: Implement intelligent asset registration with deduplication

#### Checklist:
- [ ] **3.1 Location-based Lookup**
  - [ ] Implement `get_artifacts_by_location` integration
  - [ ] Cache lookup results for session
  - [ ] Handle partial path matching
  - [ ] Support both absolute and relative paths

- [ ] **3.2 Smart Registration Logic**
  - [ ] Check if asset already registered before creating
  - [ ] Generate appropriate item names from file paths
  - [ ] Determine correct project/space hierarchy
  - [ ] Handle hash-based deduplication (optional)

- [ ] **3.3 Batch Registration**
  - [ ] Collect all unregistered dependencies
  - [ ] Create items in optimal order (dependencies first)
  - [ ] Create revisions with proper metadata
  - [ ] Upload artifacts (file references)
  - [ ] Return mapping of paths to krefs

- [ ] **3.4 Default Project Setup**
  - [ ] Auto-create `ComfyUI@{TenantName}` project if not exists
  - [ ] Create default space structure:
    ```
    ComfyUI@TenantName/
    ├── checkpoints/
    │   ├── sd15/
    │   ├── sdxl/
    │   └── flux/
    ├── loras/
    │   ├── sd15/
    │   ├── sdxl/
    │   └── flux/
    ├── controlnet/
    ├── vae/
    ├── outputs/
    │   ├── images/
    │   └── videos/
    └── inputs/
    ```

---

### Phase 4: Lineage Graph Builder (Week 4)
**Goal**: Build complete lineage graphs with proper edge relationships

#### Checklist:
- [ ] **4.1 Edge Type Definitions**
  - [ ] Define edge types for ComfyUI context:
    ```
    CREATED_FROM    - Output created from workflow
    USED_MODEL      - Used checkpoint model
    USED_LORA       - Used LoRA model
    USED_CONTROLNET - Used ControlNet model
    USED_VAE        - Used VAE model
    USED_INPUT      - Used input image/asset
    USED_EMBEDDING  - Used text embedding
    DERIVED_FROM    - img2img derivation
    ```

- [ ] **4.2 Graph Builder**
  - [ ] Build output artifact node
  - [ ] Connect all dependency edges
  - [ ] Include workflow revision reference
  - [ ] Handle multiple outputs per workflow

- [ ] **4.3 Lineage API Integration**
  - [ ] Create edges via Kumiho API
  - [ ] Handle circular dependency prevention
  - [ ] Support batch edge creation
  - [ ] Generate lineage summary JSON

- [ ] **4.4 Workflow as Artifact**
  - [ ] Store workflow JSON as artifact metadata
  - [ ] Enable workflow reproduction from lineage
  - [ ] Version workflow changes

---

### Phase 5: KumihoLoadAsset Node (Week 5)
**Goal**: Implement asset loading from Kumiho Cloud

#### Checklist:
- [ ] **5.1 Asset Browser Integration**
  - [ ] Fetch project structure from Kumiho API
  - [ ] Build space/item hierarchy
  - [ ] Cache structure for performance
  - [ ] Support search functionality

- [ ] **5.2 kref Resolution**
  - [ ] Resolve kref URI to artifact location
  - [ ] Handle revision tags (latest, published, numbered)
  - [ ] Handle artifact type variants
  - [ ] Return local file path

- [ ] **5.3 Dynamic COMBO Population**
  - [ ] Populate asset type dropdown
  - [ ] Populate category dropdown based on type
  - [ ] Populate item dropdown based on category
  - [ ] Support lazy loading for large catalogs

- [ ] **5.4 Path Output Integration**
  - [ ] Output resolved path for downstream nodes
  - [ ] Support checkpoint loader compatibility
  - [ ] Support LoRA loader compatibility
  - [ ] Support image loader compatibility

---

### Phase 6: Frontend JavaScript Extension (Week 6)
**Goal**: Build rich UI for asset browsing and lineage preview

#### Checklist:
- [ ] **6.1 Settings Panel**
  - [ ] API key configuration
  - [ ] Default project selection
  - [ ] Auto-register toggle
  - [ ] Cache settings

- [ ] **6.2 Asset Browser Sidebar**
  - [ ] Tree view of project/spaces/items
  - [ ] Search functionality
  - [ ] Filter by asset type
  - [ ] Drag-and-drop to canvas

- [ ] **6.3 Lineage Preview Panel**
  - [ ] Show lineage for selected output
  - [ ] Interactive graph visualization
  - [ ] Link to Kumiho dashboard

- [ ] **6.4 Node Enhancements**
  - [ ] Custom styling for Kumiho nodes
  - [ ] Progress indicators during upload
  - [ ] Toast notifications for status
  - [ ] Context menu actions

---

### Phase 7: Testing & Polish (Week 7)
**Goal**: Comprehensive testing and documentation

#### Checklist:
- [ ] **7.1 Unit Tests**
  - [ ] Test kref parsing/building
  - [ ] Test workflow parsing
  - [ ] Test dependency detection
  - [ ] Test API client methods

- [ ] **7.2 Integration Tests**
  - [ ] Test end-to-end registration flow
  - [ ] Test asset loading flow
  - [ ] Test lineage creation
  - [ ] Test with various workflow types

- [ ] **7.3 Compatibility Testing**
  - [ ] Test with SD 1.5 workflows
  - [ ] Test with SDXL workflows
  - [ ] Test with Flux workflows
  - [ ] Test with AnimateDiff workflows
  - [ ] Test with custom node packs

- [ ] **7.4 Documentation**
  - [ ] Complete README with examples
  - [ ] API documentation
  - [ ] Tutorial videos/GIFs
  - [ ] Troubleshooting guide

- [ ] **7.5 Registry Publishing**
  - [ ] Prepare pyproject.toml for registry
  - [ ] Set up GitHub Actions for auto-publish
  - [ ] Create release notes
  - [ ] Submit to ComfyUI Manager

---

## 🔧 Technical Details

### ComfyUI Model Categories Mapping

Based on the standard ComfyUI folder structure:

| Category | Space Path | File Extensions | Kind |
|----------|------------|-----------------|------|
| Animatediff_models | animatediff/models | .ckpt, .safetensors | animatediff_model |
| Animatediff_motion_lora | animatediff/motion_lora | .ckpt, .safetensors | motion_lora |
| Audio_encoders | audio/encoders | .ckpt, .safetensors | audio_encoder |
| Checkpoints | checkpoint/{base_model} | .ckpt, .safetensors | checkpoint |
| Clip_vision | clip/vision | .safetensors | clip_vision |
| Controlnet | controlnet/{base_model} | .safetensors, .pth | controlnet |
| Depthanything | depth/depthanything | .pth | depth_model |
| Detection | detection | .pt, .pth | detection_model |
| Diffusion_models | diffusion | .safetensors | diffusion_model |
| Gligen | gligen | .safetensors | gligen |
| Ipadapter | ipadapter | .safetensors | ipadapter |
| Latent_upscale_models | upscale/latent | .pt | latent_upscaler |
| Loras | lora/{base_model} | .safetensors | lora |
| Model_patches | patches | .safetensors | model_patch |
| Sam2 | sam/sam2 | .pt | sam2 |
| Sams | sam | .pth | sam |
| Style_models | style | .safetensors | style_model |
| Text_encoders | text/encoders | .safetensors | text_encoder |
| Upscale_models | upscale/image | .pth | upscaler |
| Vae | vae | .safetensors, .pt | vae |

### Node Detection for Dependency Scanning

| Node Class | Asset Type | Input Field |
|------------|------------|-------------|
| CheckpointLoaderSimple | checkpoint | ckpt_name |
| CheckpointLoader | checkpoint | ckpt_name |
| LoraLoader | lora | lora_name |
| LoraLoaderModelOnly | lora | lora_name |
| ControlNetLoader | controlnet | control_net_name |
| VAELoader | vae | vae_name |
| UpscaleModelLoader | upscaler | model_name |
| LoadImage | image | image |
| CLIPVisionLoader | clip_vision | clip_name |
| unCLIPCheckpointLoader | checkpoint | ckpt_name |
| GLIGENLoader | gligen | gligen_name |
| StyleModelLoader | style_model | style_model_name |

### Lineage Edge Schema

```json
{
  "source_kref": "kref://ComfyUI@Tenant/outputs/portraits/hero_portrait.image?r=1",
  "target_kref": "kref://ComfyUI@Tenant/checkpoint/sdxl/juggernaut.checkpoint?r=1",
  "edge_type": "USED_MODEL",
  "metadata": {
    "node_id": "4",
    "node_type": "CheckpointLoaderSimple",
    "workflow_position": [100, 200]
  }
}
```

---

## 📊 Success Metrics

1. **Registration Success Rate**: >99% of workflows should register without errors
2. **Dependency Detection**: Detect 100% of standard ComfyUI asset types
3. **Performance**: Registration should complete within 5 seconds for typical workflows
4. **User Adoption**: Track unique users registering workflows per week

---

## 🚨 Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Large file uploads slow | Store file references only, not actual files |
| API rate limiting | Implement batching and caching |
| Workflow JSON changes | Version workflow schema, handle migrations |
| Custom node compatibility | Use generic path detection, allow manual override |
| Authentication failures | Clear error messages, token refresh flow |

---

## 📅 Timeline Summary

| Phase | Duration | Deliverable |
|-------|----------|-------------|
| Phase 1: Foundation | Week 1 | Basic nodes, API client, kref system |
| Phase 2: Parser | Week 2 | Workflow parsing, dependency detection |
| Phase 3: Registration | Week 3 | Smart registration, deduplication |
| Phase 4: Lineage | Week 4 | Graph building, edge creation |
| Phase 5: Load Asset | Week 5 | Asset browser, kref resolution |
| Phase 6: Frontend | Week 6 | UI components, settings, browser |
| Phase 7: Testing | Week 7 | Tests, docs, registry publish |

**Total Estimated Time**: 7 weeks

---

## 🎯 MVP Definition (Phase 1-4)

Minimum Viable Product includes:
- ✅ KumihoRegister node with auto-detection
- ✅ Workflow JSON capture and storage
- ✅ Dependency scanning for checkpoints and LoRAs
- ✅ Lineage edge creation
- ✅ Basic settings (API key)

Post-MVP additions (Now Complete):
- ✅ KumihoLoadAsset node with dropdown and kref modes
- ✅ KumihoSearchItems node with SDK integration
- ✅ KumihoSaveImage/Video with optional file_path input
- ✅ Full asset browser UI
- ✅ Video preview support
- ✅ Lineage visualization

Still Planned:
- ⏳ Model loader wrappers (KumihoCheckpointLoader, KumihoLoraLoader)
- ⏳ Advanced caching and performance optimization

---

## 📦 Recent Updates (v0.4.x)

### v0.4.2 (January 2025)
- **KumihoSearchItems improvements**:
  - Now uses `kumiho.item_search()` SDK function
  - Supports `item_name_filter`, `kind_filter`, and `context_filter`
  - Returns iterative lists (krefs, file_paths) via OUTPUT_IS_LIST
  - Skips items without valid artifacts to prevent downstream errors
- **KumihoSaveImage/Video enhancements**:
  - Added optional `file_path` input for custom save locations
  - Description field changed from multiline to single-line input
- **Python SDK v0.4.2**:
  - `Item.project` and `Item.space` properties for easy access
  - `Kref.get_project()` method to extract project from kref

### v0.4.1
- Fixed KumihoLoadAsset space duplication bug
- Removed redundant project input field
- Added `get_input_kind()` helper function
- Improved kref URI handling

### v0.4.0
- Initial inline video preview support
- Video playback in ComfyUI output panel
- Lineage tracking with DERIVED_FROM edges

---

*Last Updated: January 2025*
*Version: 1.2*
