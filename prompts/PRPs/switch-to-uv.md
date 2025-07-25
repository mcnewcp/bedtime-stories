name: "Switch from requirements.txt to uv - Complete Migration PRP"
description: |
  Comprehensive migration guide from pip/requirements.txt to uv package manager for the bedtime-stories Docker application, optimized for speed, dependency resolution, and modern Python tooling.

---

## Goal
Migrate the bedtime-stories project from traditional pip/requirements.txt package management to uv, a modern Rust-based Python package manager that provides 10-100x faster installations, better dependency resolution, built-in lockfiles, and unified tooling.

## Why
- **Performance**: uv is 10-100x faster than pip for large projects with parallel downloads and optimized dependency resolution
- **Reliability**: Better dependency resolution with built-in conflict detection and resolution
- **Modern Tooling**: Unified tool for project management, virtual environments, and dependency management
- **Lockfiles**: Built-in support for reproducible installations across environments
- **Docker Optimization**: Native Docker integration with multi-stage build optimizations
- **Future-proofing**: Industry trend toward faster, Rust-based Python tooling

## What
Convert the existing Docker-based Streamlit application from pip/requirements.txt to uv/pyproject.toml while maintaining all existing functionality and improving build performance.

### Success Criteria
- [ ] pyproject.toml created with all current dependencies plus dev dependencies
- [ ] Dockerfile updated to use uv with optimized multi-stage build
- [ ] Virtual environment setup with uv locally
- [ ] .env.example file created with all required environment variables
- [ ] README.md updated with new setup instructions
- [ ] Docker build time reduced by at least 20%
- [ ] All existing functionality preserved (app.py runs identically)
- [ ] Development workflow improved with faster dependency installs

## All Needed Context

### Documentation & References
```yaml
# MUST READ - Include these in your context window
- url: https://docs.astral.sh/uv/guides/integration/docker/
  why: Official Docker integration guide with best practices and examples
  
- url: https://docs.astral.sh/uv/concepts/projects/
  why: Understanding pyproject.toml structure and project management
  
- file: /Users/coymcnew/code/bedtime-stories/requirements.txt
  why: Current dependencies to migrate - 9 packages including streamlit, anthropic, openai
  
- file: /Users/coymcnew/code/bedtime-stories/Dockerfile
  why: Current Docker setup using pip - needs conversion to uv pattern
  
- file: /Users/coymcnew/code/bedtime-stories/docker-compose.yml
  why: Environment variable setup and volume mounts to preserve
  
- file: /Users/coymcnew/code/bedtime-stories/app.py
  why: Main application to verify all imports still work after migration

- doc: https://hynek.me/articles/docker-uv/
  section: Production-ready Python Docker Containers with uv
  critical: Multi-stage build patterns and cache optimization techniques
```

### Current Codebase tree
```bash
bedtime-stories/
├── CLAUDE.md
├── Dockerfile                 # Current pip-based Docker setup
├── README.md                  
├── app.py                     # Main Streamlit app
├── data/
├── docker-compose.yml         # Environment variables setup
├── requirements.txt           # 9 dependencies to migrate
├── test_app.py               # Test file to validate after migration
└── prompts/
    ├── PLANNING.md
    └── features/
        └── switch-to-uv.md   # Feature specification
```

### Desired Codebase tree with files to be added
```bash
bedtime-stories/
├── CLAUDE.md
├── Dockerfile                 # Updated for uv multi-stage build
├── README.md                  # Updated setup instructions
├── app.py                     # Unchanged
├── data/
├── docker-compose.yml         # Minimal/no changes needed
├── pyproject.toml             # NEW: Modern Python project configuration
├── uv.lock                    # NEW: Generated lockfile (auto-created)
├── .env.example               # NEW: Template for environment variables
├── test_app.py               # Unchanged
└── prompts/
    ├── PLANNING.md
    └── features/
        └── switch-to-uv.md   # Feature specification
```

### Known Gotchas & Library Quirks
```python
# CRITICAL: uv requires pyproject.toml in project root
# pyproject.toml must have [project] section with name, version, dependencies

# CRITICAL: Docker multi-stage builds with uv
# Use COPY --from=ghcr.io/astral-sh/uv:0.5.1 /uv /uvx /bin/ for specific version
# Pin to specific uv version to avoid build breaking changes

# GOTCHA: Audio dependencies in Docker
# Current Dockerfile has portaudio19-dev and alsa-utils for sounddevice/soundfile
# These system dependencies must be preserved in uv Dockerfile

# GOTCHA: Environment variables in Docker
# docker-compose.yml passes ANTHROPIC_API_KEY, OPENAI_API_KEY, ELEVENLABS_API_KEY
# These must be preserved exactly as they are

# GOTCHA: Virtual environment in Docker vs local
# Docker should use --system flag to avoid nested venv
# Local development should use uv venv for isolation

# GOTCHA: Streamlit port exposure
# Current setup exposes port 8501 - must be preserved
# Health check uses curl to /_stcore/health endpoint

# CRITICAL: uv sync vs uv pip install
# Use `uv sync --locked` for reproducible builds
# Use `uv sync --no-install-project` for dependencies-only layer in Docker
```

## Implementation Blueprint

### Data models and structure
```python
# pyproject.toml structure based on current requirements.txt:
[project]
name = "bedtime-stories"
version = "0.1.0"
dependencies = [
    "streamlit>=1.12.0",      # Web framework
    "anthropic>=0.25.0",      # Claude API
    "python-dotenv>=1.0.0",   # Environment variables
    "openai>=1.0.0",          # OpenAI API (Whisper + DALL-E)
    "sounddevice>=0.4.0",     # Audio recording
    "soundfile>=0.10.0",      # Audio file handling
    "numpy>=1.21.0",          # Required by sounddevice
    "elevenlabs>=0.2.0",      # Text-to-speech
    "requests>=2.25.0",       # HTTP requests
]
requires-python = ">=3.10"

[dependency-groups]
dev = [
    "pytest>=7.4.0",         # Testing framework
    "black>=23.0.0",          # Code formatting
    "ruff>=0.1.0",            # Linting
    "pytest-cov>=6.2.1",     # Coverage reporting
    "pytest-asyncio>=0.21.0", # Async testing
]

# .env.example structure:
ANTHROPIC_API_KEY=your_anthropic_api_key_here
OPENAI_API_KEY=your_openai_api_key_here  
ELEVENLABS_API_KEY=your_elevenlabs_api_key_here
```

### List of tasks to be completed in order

```yaml
Task 1: Create pyproject.toml
CREATE pyproject.toml:
  - COPY dependency list from requirements.txt
  - ADD [project] section with name="bedtime-stories", version="0.1.0"
  - ADD requires-python = ">=3.10" (current Dockerfile uses 3.11)
  - ADD [dependency-groups] with dev dependencies
  - PRESERVE exact version constraints from requirements.txt

Task 2: Create .env.example file
CREATE .env.example:
  - ADD ANTHROPIC_API_KEY=your_anthropic_api_key_here
  - ADD OPENAI_API_KEY=your_openai_api_key_here
  - ADD ELEVENLABS_API_KEY=your_elevenlabs_api_key_here
  - MATCH format from current README.md example

Task 3: Update Dockerfile for uv
MODIFY Dockerfile:
  - REPLACE pip installation with uv binary copy from official image
  - ADD cache mount for uv: --mount=type=cache,target=/root/.cache/uv
  - USE multi-stage pattern: dependencies first, then app code
  - PRESERVE system dependencies: portaudio19-dev, alsa-utils, curl
  - PRESERVE user creation, port exposure, health check
  - USE uv sync --locked --no-install-project for dependencies layer
  - USE uv sync --locked for final app layer

Task 4: Set up local uv environment  
LOCAL SETUP:
  - RUN uv venv to create virtual environment
  - RUN uv sync to install all dependencies from pyproject.toml
  - VERIFY all imports in app.py work correctly
  - TEST streamlit run app.py launches successfully

Task 5: Update README.md
MODIFY README.md:
  - REPLACE pip install -r requirements.txt with uv sync
  - ADD uv installation instructions
  - UPDATE Docker setup to reference pyproject.toml
  - ADD note about uv.lock file generation
  - PRESERVE all other content and structure

Task 6: Validate migration completeness
VERIFY:
  - Docker build succeeds with new Dockerfile
  - Docker container runs app successfully
  - All API integrations work (Anthropic, OpenAI, ElevenLabs)
  - Local development setup works with uv
  - No missing dependencies or import errors
```

### Per task pseudocode

```python
# Task 1: pyproject.toml creation
# Based on requirements.txt analysis:
[project]
name = "bedtime-stories"
version = "0.1.0"
dependencies = [
    # PATTERN: Copy each line from requirements.txt
    # PRESERVE: Version constraints exactly as specified
    "streamlit>=1.12.0",
    "anthropic>=0.25.0", 
    # ... continue for all 9 dependencies
]
requires-python = ">=3.10"  # Compatible with Docker python:3.11-slim

[dependency-groups]
dev = [
    # PATTERN: Add common Python dev tools
    # MATCH: Current project's testing approach from test_app.py
    "pytest>=7.4.0",
    "black>=23.0.0",
    "ruff>=0.1.0",
    "pytest-cov>=6.2.1",
    "pytest-asyncio>=0.21.0",
]

# Task 3: Dockerfile optimization
FROM python:3.11-slim AS base
# CRITICAL: Copy uv binary from official image with version pin
COPY --from=ghcr.io/astral-sh/uv:0.5.1 /uv /uvx /bin/

# PRESERVE: System dependencies for audio processing
RUN apt-get update && apt-get install -y \
    portaudio19-dev \
    alsa-utils \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# OPTIMIZATION: Dependencies layer (cached until pyproject.toml changes)
COPY pyproject.toml uv.lock ./
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-install-project

# APPLICATION: App code layer  
COPY app.py .
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked

# PRESERVE: Security and runtime setup
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8501
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8501/_stcore/health || exit 1

# MODIFY: Use uv to run streamlit
CMD ["uv", "run", "streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

### Integration Points
```yaml
DOCKER:
  - preserve: Environment variable passing from docker-compose.yml
  - preserve: Volume mounts for ./data:/app/data
  - preserve: Network configuration bedtime-stories-network
  
DEPENDENCIES:
  - migrate: All 9 packages from requirements.txt to pyproject.toml dependencies
  - add: Dev dependencies for local development and testing
  
ENVIRONMENT:
  - create: .env.example with all 3 API keys
  - preserve: Existing .env file structure and docker-compose env_file reference
  
LOCAL_DEV:
  - setup: uv venv for virtual environment
  - install: uv sync for dependency installation
  - verify: All imports and functionality work with new setup
```

## Validation Loop

### Level 1: Syntax & Style
```bash
# Validate pyproject.toml syntax
uv sync --dry-run  # Check if dependencies can be resolved

# Validate Docker build  
docker build -t bedtime-stories-uv .  # Should complete without errors

# Expected: Clean build with faster dependency installation
```

### Level 2: Functional Tests
```bash
# Test local uv setup
uv venv                               # Create virtual environment
uv sync                              # Install all dependencies  
uv run python -c "import streamlit, anthropic, openai, elevenlabs, sounddevice, soundfile, numpy, requests, dotenv; print('All imports successful')"

# Test Docker container
docker run -d --name test-bedtime -p 8501:8501 \
  -e ANTHROPIC_API_KEY=test \
  -e OPENAI_API_KEY=test \
  -e ELEVENLABS_API_KEY=test \
  bedtime-stories-uv

# Wait for container to start
sleep 10

# Test health endpoint
curl -f http://localhost:8501/_stcore/health

# Test main app loads
curl -s http://localhost:8501 | grep -q "Bedtime Story Generator"

# Cleanup
docker stop test-bedtime && docker rm test-bedtime

# Expected: All tests pass, app loads correctly
```

### Level 3: Performance Validation
```bash
# Measure build time improvement
time docker build -t bedtime-stories-pip -f Dockerfile.old .    # Current pip version
time docker build -t bedtime-stories-uv .                      # New uv version

# Compare layer caching
docker build -t bedtime-stories-uv . --no-cache                # First build
touch app.py && docker build -t bedtime-stories-uv .           # Incremental build

# Expected: 20%+ faster builds, better layer caching with uv
```

## Final Validation Checklist
- [ ] pyproject.toml created with all dependencies: `uv sync --dry-run`
- [ ] .env.example created with all API keys
- [ ] Dockerfile builds successfully: `docker build -t test .`
- [ ] Container runs and serves app: `docker run -p 8501:8501 test`
- [ ] Health check passes: `curl http://localhost:8501/_stcore/health`
- [ ] All imports work: `uv run python -c "import streamlit, anthropic, openai"`
- [ ] Local development setup: `uv sync && uv run streamlit run app.py`
- [ ] README.md updated with uv instructions
- [ ] Build time improved (measure with `time docker build`)
- [ ] No missing dependencies or runtime errors

---

## Anti-Patterns to Avoid
- ❌ Don't remove requirements.txt until after validating pyproject.toml works
- ❌ Don't change dependency versions during migration - keep exact same versions
- ❌ Don't skip system dependencies (portaudio19-dev, alsa-utils) needed for audio
- ❌ Don't forget to pin uv version in Dockerfile to avoid breaking changes
- ❌ Don't use nested virtual environments in Docker - use --system flag
- ❌ Don't modify environment variable names or docker-compose.yml setup
- ❌ Don't skip testing both local development and Docker builds
- ❌ Don't assume uv.lock will exist initially - it's generated on first sync

## Confidence Score: 9/10
This PRP provides comprehensive context for one-pass implementation with:
- Complete dependency mapping from current requirements.txt
- Detailed Docker integration with official uv best practices  
- Preserved system requirements and environment setup
- Clear validation steps and performance benchmarks
- Specific gotchas and anti-patterns to avoid
- Executable commands for verification at each step