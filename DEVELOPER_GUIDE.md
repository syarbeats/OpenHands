# OpenHands Developer Guide

This guide provides comprehensive instructions for setting up the development environment and running the OpenHands platform. OpenHands is a software development agent platform powered by AI that can modify code, run commands, browse the web, call APIs, and more.

## Table of Contents

- [System Requirements](#system-requirements)
- [Installation Methods](#installation-methods)
  - [Method 1: Running with Docker (Easiest)](#method-1-running-with-docker-easiest)
  - [Method 2: Local Development Setup](#method-2-local-development-setup)
  - [Method 3: Development Inside Docker Container](#method-3-development-inside-docker-container)
- [Configuration](#configuration)
- [Running OpenHands](#running-openhands)
- [Advanced Usage](#advanced-usage)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## System Requirements

### Operating Systems
- **Linux**: Ubuntu >= 22.04 (recommended)
- **macOS**: All recent versions supported
- **Windows**: Via WSL (Windows Subsystem for Linux)

### For Docker Deployment (Method 1)
- Docker

### For Local Development (Method 2)
- Python 3.12
- Node.js >= 22.x
- Poetry >= 1.8
- Docker
- OS-specific dependencies:
  - Ubuntu: `build-essential python3.12-dev`
  - WSL: `netcat`

## Installation Methods

### Method 1: Running with Docker (Easiest)

This is the simplest way to run OpenHands without setting up the development environment:

```bash
# Pull the Docker image
docker pull docker.all-hands.dev/all-hands-ai/runtime:0.39-nikolaik

# Run OpenHands in Docker
docker run -it --rm --pull=always \
    -e SANDBOX_RUNTIME_CONTAINER_IMAGE=docker.all-hands.dev/all-hands-ai/runtime:0.39-nikolaik \
    -e LOG_ALL_EVENTS=true \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v ~/.openhands-state:/.openhands-state \
    -p 3000:3000 \
    --add-host host.docker.internal:host-gateway \
    --name openhands-app \
    docker.all-hands.dev/all-hands-ai/openhands:0.39
```

After running the command, OpenHands will be available at http://localhost:3000.

Alternatively, you can use the Makefile to run the Docker container:

```bash
make docker-run
```

### Method 2: Local Development Setup

#### Step 1: Install Dependencies

If you don't have sudo access, you can use conda/mamba:

```bash
# Download and install Mamba (a faster version of conda)
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh

# Install Python 3.12, nodejs, and poetry
mamba install python=3.12
mamba install conda-forge::nodejs
mamba install conda-forge::poetry
```

For Ubuntu/Debian:

```bash
# Install Python 3.12 and dev tools
sudo apt update
sudo apt install build-essential python3.12 python3.12-dev netcat

# Install Node.js 22.x
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Install Poetry
curl -sSL https://install.python-poetry.org | python3.12 -
```

#### Step 2: Clone the Repository

```bash
git clone https://github.com/All-Hands-AI/OpenHands.git
cd OpenHands
```

#### Step 3: Build the Project

This step sets up the environment and installs all dependencies:

```bash
make build
```

If you encounter an error related to missing `_sqlite3` module during the build process, it means your Python was compiled without SQLite3 support. You can use the alternative build command that skips pre-commit hooks installation:

```bash
make build-no-hooks
```

> **Note**: Pre-commit hooks require SQLite3 support in Python. If you need to use pre-commit hooks, you'll need to reinstall Python with SQLite3 support by installing the development libraries first:
> ```bash
> # For Ubuntu/Debian
> sudo apt-get install -y libsqlite3-dev
> # Then reinstall Python with SQLite support
> ```

#### Step 4: Configure OpenHands

Setup the configuration for OpenHands, including LLM API key and model:

```bash
make setup-config
```

You'll be prompted to enter:
- Workspace directory (default: ./workspace)
- LLM model name (default: gpt-4o)
- LLM API key
- LLM base URL (if needed, usually for local models)

### Method 3: Development Inside Docker Container

This approach allows you to develop inside a Docker container without installing all dependencies locally:

```bash
make docker-dev
```

or if you don't have `make`:

```bash
cd ./containers/dev
./dev.sh
```

## Configuration

OpenHands uses a `config.toml` file for configuration. The configuration priorities from highest to lowest are:

1. Environment variables
2. config.toml variables
3. Default variables

Key configuration sections:

```toml
[core]
workspace_base = "./workspace"  # Directory where workspaces are stored

[llm]
model = "gpt-4o"                # Default LLM model
api_key = "your-api-key-here"   # Your LLM API key
base_url = ""                   # Optional base URL for local LLMs
```

## Running OpenHands

### Running the Full Application

```bash
make run
```

This starts both the backend and frontend servers.

### Running Individual Components

**Start the Backend Server Only:**

```bash
make start-backend
```

**Start the Frontend Server Only:**

```bash
make start-frontend
```

OpenHands will be accessible at:
- Backend API: http://localhost:3000
- Frontend (when started separately): http://localhost:3001

## Advanced Usage

### LLM Debugging

To enable detailed LLM logs:

```bash
export DEBUG=1
make start-backend
```

The logs will be stored in `logs/llm/CURRENT_DATE/` directory.

### Custom Docker Image

To use a custom Docker image for the sandbox runtime:

```bash
export SANDBOX_RUNTIME_CONTAINER_IMAGE=your-custom-image:tag
make run
```

### Running Headless Mode

For information on headless mode, CLI mode, or GitHub action integrations, refer to the documentation at:
https://docs.all-hands.dev/modules/usage/how-to/headless-mode

## Testing

### Running Unit Tests

```bash
poetry run pytest ./tests/unit/test_*.py
```

### Running Frontend Tests

```bash
cd frontend
npm run test
```

## Troubleshooting

If you encounter issues:

1. Check that all dependencies are properly installed
2. Verify that Docker is running and accessible
3. If using a local LLM, ensure it's running and the base URL is correct
4. For detailed LLM debugging, set `export DEBUG=1`
5. Check logs in the `logs/` directory

### Common Issues

#### ModuleNotFoundError: No module named '_sqlite3'

This error occurs during the build process when your Python installation is compiled without SQLite3 support. The issue specifically affects the pre-commit hooks installation step.

**Symptoms:**
```
ModuleNotFoundError: No module named '_sqlite3'
make[1]: *** [Makefile:187: install-pre-commit-hooks] Error 1
make: *** [Makefile:29: build] Error 2
```

**Quick Solution:**
Use the alternative build target that skips pre-commit hooks:
```bash
make build-no-hooks
```

**Permanent Solution:**
1. Install SQLite3 development libraries:
   ```bash
   # Ubuntu/Debian
   sudo apt-get install -y libsqlite3-dev
   
   # CentOS/RHEL
   sudo yum install -y sqlite-devel
   
   # macOS
   brew install sqlite3
   ```

2. Reinstall Python with SQLite support:
   - If using system Python, you may need to recompile Python from source
   - If using conda/mamba, you can reinstall Python: `mamba install python=3.12`

For more troubleshooting tips, see the [Troubleshooting Guide](https://docs.all-hands.dev/modules/usage/troubleshooting).

## Additional Resources

- [OpenHands Documentation](https://docs.all-hands.dev/modules/usage/getting-started)
- [OpenHands Cloud](https://app.all-hands.dev)
- [GitHub Repository](https://github.com/All-Hands-AI/OpenHands)
- [Contributing Guide](https://github.com/All-Hands-AI/OpenHands/blob/main/CONTRIBUTING.md)
- [Development Guide](https://github.com/All-Hands-AI/OpenHands/blob/main/Development.md)

Join the community:
- [Slack](https://join.slack.com/t/openhands-ai/shared_invite/zt-34zm4j0gj-Qz5kRHoca8DFCbqXPS~f_A)
- [Discord](https://discord.gg/ESHStjSjD4)
