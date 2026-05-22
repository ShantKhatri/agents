# Contributing to VideoSDK AI Agents

Thank you for your interest in contributing! We welcome developers of all skill levels. Whether you're fixing a typo, squashing a bug, or adding a massive new feature (like a new AI provider plugin!), your help is deeply appreciated. 

## Quick Links
- **[Official Documentation](https://docs.videosdk.live/ai_agents/introduction)**: Get familiar with the framework and its capabilities.
- **[Discord Community](https://discord.com/invite/f2WsNDN9S5)**: Join the `#agents` channel for help, to ask questions, or to discuss your ideas with maintainers.
- **[Code of Conduct](CODE_OF_CONDUCT.md)**: Please review and adhere to our community guidelines.

## Ways to Contribute

1. **Write a plugin**: If you use a TTS/STT/LLM/Realtime provider that isn't in our plugins list, consider writing a plugin for it! We have a detailed **[Plugin Development Guide](BUILD_YOUR_OWN_PLUGIN.md)** to help you get started.
2. **Fix bugs**: Help improve framework reliability by fixing bugs and enhancing stability.
3. **Add new features**: We welcome new feature contributions. Please open an issue first to discuss the proposed functionality and scope before starting development.
4. **Help the community**: Answer questions on our [Discord](https://discord.com/invite/f2WsNDN9S5) or review other people's pull requests.

## Local Development Setup

To contribute code, you'll need to set up the project locally. We support two methods for dependency management: **uv** (recommended for speed) and standard **pip** via our setup script.

### Prerequisites
- Python 3.12 or higher
- Git

### Method 1: Using `uv` (Recommended)

[uv](https://docs.astral.sh/uv/) is a blazingly fast Python package manager.

```bash
# 1. Clone the repository
git clone https://github.com/videosdk-live/agents.git
cd agents

# 2. Sync dependencies (this installs the core package and all plugins in editable mode)
uv sync

# 3. You're ready!
```

### Method 2: Using `setup.sh` (Pip)

If you prefer using standard `pip` and virtual environments, we've provided a handy setup script that handles the heavy lifting.

```bash
# 1. Clone the repository
git clone https://github.com/videosdk-live/agents.git
cd agents

# 2. Run the setup script to create a venv and install everything in editable mode
bash setup.sh

# 3. Activate the virtual environment
source venv/bin/activate
```

## Running an Agent Locally

Once your environment is set up, you can run an agent to test your changes. 

1. **Create an Environment File**: Copy the `.env.example` file from the `examples` directory and add your API keys (you will definitely need a `VIDEOSDK_AUTH_TOKEN`).
   ```bash
   cp examples/.env.example examples/.env
   ```
   
2. **Run a Basic Agent**: The examples are configured with `playground=True` by default when run from the terminal. This allows you to test the agent using your terminal's microphone and speakers without needing to connect a frontend client or an active VideoSDK room.
   
   If using `uv`:
   ```bash
   uv run python examples/cascade_basic.py
   ```
   
   If using the `venv` from `setup.sh`:
   ```bash
   python examples/cascade_basic.py
   ```

## Formatting and Linting

To ensure code quality, we use `ruff`. Please format and lint your code before submitting a pull request to ensure it passes CI checks.

```bash
# Format your code
uv run ruff format

# Check for linting errors and auto-fix them
uv run ruff check --fix
```

## Submitting a Pull Request

1. **Fork the repository** and create your feature branch from `main`.
2. **Test your changes** locally.
3. **Format your code** using `ruff`.
4. **Commit your changes**: Write clear, descriptive commit messages.
5. **Open a PR**: Describe your changes in detail, link any relevant issues, and wait for a maintainer to review!

We are excited to see what you build! Let's shape the future of real-time AI together.