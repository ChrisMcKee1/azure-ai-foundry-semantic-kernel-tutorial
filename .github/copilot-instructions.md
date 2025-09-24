# Copilot Instructions for SKNotebooks

## Project Overview
This is an **Azure AI Foundry + Semantic Kernel integration project** focused on educational Jupyter notebooks demonstrating advanced AI agent patterns. The project showcases dual-SDK architecture with proper async/sync credential separation and comprehensive resource management.

## Architecture & Key Patterns

### Dual-SDK Integration Pattern
- **Azure AI Foundry SDK**: Uses `sync DefaultAzureCredential` for direct Azure AI service calls
- **Semantic Kernel**: Uses `AsyncDefaultAzureCredential` for agent wrapper operations
- **Critical**: Never mix sync/async credentials - this separation prevents coroutine errors

```python
# Correct pattern from azure_ai_foundry_semantic_kernel_comprehensive.ipynb
credential = DefaultAzureCredential()  # Sync for AgentsClient
async_credential = AsyncDefaultAzureCredential()  # Async for Semantic Kernel
```

### Environment & Configuration
- Uses `.env` files with `python-dotenv` for configuration (copy from `template.env`)
- Fallback defaults for Azure endpoints if env vars not set
- Environment loading consolidated in imports cell (not duplicated)
- **Never commit .env** - use `template.env` for version control

### Resource Management Pattern
- **Consolidated cleanup**: All resource cleanup happens in final notebook cell
- **Proper lifecycle**: Azure agents → threads → files → credentials (in order)
- **Local preservation**: Downloads preserved locally while Azure resources cleaned up

```python
# From cleanup cell - proper order matters
await thread.delete()
foundry_client.delete_agent(agent_definition.id)
await sk_client.__aexit__(None, None, None)
await async_credential.close()
```

## Development Workflows

### Virtual Environment Setup
```bash
# Always use virtual environment for dependency isolation
.\.venv\Scripts\Activate.ps1  # Windows PowerShell
pip install -r requirements.txt
```

### Code Quality Tools
- **Vulture**: Dead code detection (dev dependency only, not in requirements.txt)
- **Requirements pinning**: Use `>=` for stable versions, exact versions for betas

### Notebook Development
- **Cell execution order**: Follow sequential pattern (imports → config → agents → operations → cleanup)
- **Async handling**: Use `nest_asyncio.apply()` for Jupyter compatibility
- **File operations**: Create `azure_ai_files/downloads/` structure for outputs

## Key Dependencies & Versions
- `semantic-kernel>=1.37.0` - Latest stable SK with Azure AI agent support
- `azure-ai-agents>=1.2.0b4` - Beta version for latest features
- `azure-identity>=1.25.0` - Latest auth library with async support

## File Organization
```
├── azure_ai_foundry_semantic_kernel_comprehensive.ipynb  # Main educational notebook
├── requirements.txt                                       # Pinned dependencies
├── template.env                                          # Environment template (committed)
├── .env                                                  # Local configuration (gitignored)
├── .gitignore                                           # Python/Jupyter optimized
└── azure_ai_files/downloads/                            # Generated file outputs (gitignored)
```

## Common Patterns to Follow

### Agent Creation Separation
Keep Azure AI Foundry agent creation and Semantic Kernel wrapper creation in separate cells for educational clarity.

### Type Checking for File Detection
```python
# Robust file detection pattern
if hasattr(message, 'image_contents') and message.image_contents:
    # Handle image files
if hasattr(message, 'file_path_annotations') and message.file_path_annotations:
    # Handle generated files
```

### Error Handling for File Operations
Always include fallback filenames and path extraction for robust file downloads.

## Integration Points
- **Azure AI Foundry**: Direct service calls for agent creation, file operations
- **Semantic Kernel**: Agent wrapper for conversation management
- **Environment Variables**: Azure endpoint and model deployment configuration
- **File System**: Local downloads directory for preserving generated content

## Testing & Validation
- Run notebook cells sequentially to validate end-to-end flow
- Verify resource cleanup with no orphaned Azure resources
- Check file downloads in `azure_ai_files/downloads/` directory
- Use vulture in virtual environment to detect unused imports/variables