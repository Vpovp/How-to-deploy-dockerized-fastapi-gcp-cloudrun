# VS Code Dev Container Configuration Explanation

This document explains each key in the `devcontainer.json` file used for a Python FastAPI development environment.

---

## name
The `name` field defines a human-readable name for the dev container. This name appears in VS Code’s Dev Containers UI and helps identify the container configuration.

---

## build
The `build` section specifies how the Docker image for the dev container is created. The `dockerfile` field points to the Dockerfile (relative to the `.devcontainer` directory) that is used to build the image. This Dockerfile usually installs Python, FastAPI, and all required dependencies.

---

## customizations
The `customizations` section defines editor-specific settings that apply only inside the container. In this case, it customizes VS Code behavior.

### customizations.vscode.extensions
This list defines VS Code extensions that are automatically installed inside the container:
- **ms-python.python**: Provides core Python support such as running, linting, and testing.
- **ms-python.vscode-pylance**: Adds fast IntelliSense and type checking for Python.
- **ms-python.black-formatter**: Enables automatic Python code formatting using Black.
- **ms-python.debugpy**: Supports Python debugging, commonly used with FastAPI.
- **ms-azuretools.vscode-docker**: Adds Docker integration to VS Code.

These extensions are installed only inside the container and do not affect the local VS Code setup.

### customizations.vscode.settings
This field contains VS Code settings that apply only inside the container. It is empty in this configuration but can be used for settings such as Python interpreter paths or enabling format-on-save.

---

## forwardPorts
The `forwardPorts` field forwards network ports from the container to the host machine. The format is `hostPort:containerPort`. Port `5678` is commonly used by `debugpy` for Python debugging, allowing the host VS Code instance to attach to the debugger running inside the container.

---

## workspaceMount
The `workspaceMount` field defines how the local workspace directory is mounted into the container. The local project directory is bind-mounted into `/code` inside the container. The `consistency=delegated` option improves filesystem performance, especially on macOS.

---

## workspaceFolder
The `workspaceFolder` field defines the default working directory inside the container. VS Code opens this folder as the project root when the container starts. In this configuration, it is set to `/code`, matching the mount target.

---

## runArgs
The `runArgs` field allows passing additional arguments to the `docker run` command. It is empty here but can be used for advanced configuration such as environment variables, network settings, privileged mode, or GPU access.

---

## Summary
This Dev Container configuration builds a Docker image using a custom Dockerfile, mounts the local workspace into the container, installs Python and Docker-related VS Code extensions, forwards a debugging port, and provides a clean, reproducible development environment for FastAPI projects.
