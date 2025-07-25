# 🌙 Bedtime Stories Generator

A Streamlit application that generates personalized bedtime stories with AI-powered narration and illustrations.

## Features

- **AI Story Generation**: Custom bedtime stories using Claude AI
- **Voice Narration**: Text-to-speech with ElevenLabs voices
- **Story Illustrations**: DALL-E generated images for visual storytelling
- **Interactive Elements**: Read-along demos and customizable themes
- **Audio Recording**: Voice input for story prompts

## 🐳 Docker Deployment

This project uses **uv** for fast Python package management and **pyproject.toml** for modern dependency management.

### Prerequisites

- Docker and Docker Compose installed
- API keys for:
  - Anthropic Claude API
  - OpenAI API (for images)
  - ElevenLabs API (for voice)

### Quick Start

1. **Clone the repository**:
   ```bash
   git clone https://github.com/dan-shah/bedtime-stories.git
   cd bedtime-stories
   ```

2. **Set up environment variables**:
   Copy `.env.example` to `.env` and add your API keys:
   ```bash
   cp .env.example .env
   # Edit .env with your actual API keys
   ```

3. **Run with Docker Compose**:
   ```bash
   docker-compose up -d
   ```

4. **Access the application**:
   Open your browser to `http://localhost:8501`

### Alternative Docker Commands

**Build and run manually**:
```bash
# Build the image
docker build -t bedtime-stories .

# Run the container
docker run -p 8501:8501 --env-file .env bedtime-stories
```

**Stop the application**:
```bash
docker-compose down
```

## 🛠️ Local Development

### Prerequisites

1. **Install uv** (fast Python package manager):
   ```bash
   # macOS/Linux
   curl -LsSf https://astral.sh/uv/install.sh | sh
   
   # Windows
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   ```

### Setup

1. **Create virtual environment and install dependencies**:
   ```bash
   uv venv
   uv sync
   ```
   *Note: This will create a `uv.lock` file for reproducible installations*

2. **Set up environment variables**:
   Copy `.env.example` to `.env` and add your API keys:
   ```bash
   cp .env.example .env
   # Edit .env with your actual API keys
   ```

3. **Run the application**:
   ```bash
   uv run streamlit run app.py
   ```

## 📝 Usage

1. Enter a story prompt or use voice recording
2. Customize with child's name and theme
3. Generate story with optional illustrations
4. Listen to AI narration with selected voice
5. Enjoy interactive read-along features

## 🔧 Configuration

- **Image Generation**: Choose between DALL-E 2 or DALL-E 3
- **Voice Selection**: Multiple ElevenLabs voice options
- **Story Themes**: Adventure, friendship, magic, and more

## 🚀 Production Deployment

For production use, consider:
- Using a reverse proxy (nginx)
- Adding SSL/TLS termination
- Implementing proper logging
- Setting up monitoring and health checks
- Using managed services for APIs

## 📄 License

This project is open source and available under the MIT License.