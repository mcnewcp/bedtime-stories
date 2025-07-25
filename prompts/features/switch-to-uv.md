## FEATURE:
Switch from `requirements.txt` to uv.

### Overview
The repo is designed to be deployed as a container using a `DOCKERFILE` and `docker-compose.yaml`.  Because of this, it uses `requirements.txt` and `pip`.  I would instead like to switch to using the virtual environment manager uv.  This will speed up installs and imports, provide better dependecy resolution, provide a built-in lockfile, and simply becuase uv is a unified tool for dependency management. 

### Design
Required changes include:
1. Create a `pyproject.toml` which includes dependencies.  Create a dev group for any dev dependencies that are not needed to run the app in the container.
2. Update the Dockerfile to use uv and also pull depenendcies from `pyproject.toml` instead of `requirements.txt`.
3. Check `docker-compose.yml` to see if any changes are needed there.

## EXAMPLES:

`pyproject.toml` should look something like the following.  Please populate the dependency list on your own, this is just meant to serve as an example of the structure.

```
[project]
  name = "bedtime-stories"
  version = "0.1.0"
  dependencies = [
      "streamlit>=1.12.0",
      "anthropic>=0.25.0",
      "python-dotenv>=1.0.0",
      "openai>=1.0.0",
      "sounddevice>=0.4.0",
      "soundfile>=0.10.0",
      "numpy>=1.21.0",
      "elevenlabs>=0.2.0",
      "requests>=2.25.0"
  ]

  requires-python = ">=3.10"

[dependency-groups]
dev = [
    "pytest>=7.4.0",
    "black>=23.0.0",
    "ruff>=0.1.0",
    "pytest-cov>=6.2.1",
    "pytest-asyncio>=0.21.0",
]
```

`Dockerfile` should look something like this:

```
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies and uv
RUN apt-get update && apt-get install -y \
    portaudio19-dev \
    alsa-utils \
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && pip install uv

# Copy dependency files
COPY pyproject.toml .

# Install Python dependencies with uv
RUN uv pip install --system -r pyproject.toml

# Copy application code  
COPY app.py .

# Create non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8501

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8501/_stcore/health || exit 1

CMD ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```


## DOCUMENTATION:

## OTHER CONSIDERATIONS:

- Include a .env.example, which should include examples of all environment variables which the developer needs to provide in their `.env` file
- Include the project structure in the README.
- Update the README after you've made these changes.
- Use python_dotenv and load_env() for environment variables.
- A virtual environment has not yet been setup.  Please use `uv` to do so and install all required dependencies into the virtual environment.
