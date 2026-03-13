# Setup Instructions: Looker Query to Google Slide Flow

This document outlines the setup steps required to build a workflow that runs a Looker query, generates a chart image, uploads it to Google Drive, and creates a Google Slides presentation.

## Prerequisites: Install Required Skills

To interact with Google Workspace APIs (specifically Drive and Slides), you need to install the following Gemini skills:

**Install the Google Slides Skill:** This skill allows the agent to read and write Google Slides presentations.

```bash
npx skills add --yes --global https://github.com/googleworkspace/cli/tree/main/skills/gws-slides
```

**Install the Google Drive Skill:** This skill allows the agent to manage files, folders, and shared drives in Google Drive.

```bash
npx skills add --yes --global https://github.com/googleworkspace/cli/tree/main/skills/gws-drive
```

> **Note:** The `--global` flag installs these to your universal `~/.agents/skills/` directory. If you prefer to limit them to Jetski specifically, you can ensure the directory exists and copy them over using the following commands:
> ```bash
> mkdir -p ~/.gemini/jetski/skills
> cp -r ~/.agents/skills/gws-* ~/.gemini/jetski/skills/
> ```

## Utilize Executive Slide Formatting

When prompting the agent to generate a slide deck from images or data, explicitly request the "Executive Slides Generation Flow" skill. This skill enforces:
- **Mandatory Image Handling:** Uploads local images to Google Drive first and feeds those generated URLs into Google Slides.
- **Premium Aesthetics:** Enforces High contrast, professional color palettes, and modern typography styles on all generated slides.

## Setup MCP Servers (`mcp_config.json`)

To enable the agent to query Looker and interact with Google Workspace APIs natively, you must configure the MCP servers in your `mcp_config.json` file.

Add the following configurations under your `"mcpServers"` block:

### 1. Google Workspace CLI (`googleworkspace-tool`)

This server provides the tools needed to interact directly with Drive and Slides.

#### Authentication Setup

Before running the server, you need to authenticate the Google Workspace CLI. You can find detailed instructions in the [googleworkspace/cli documentation](https://github.com/googleworkspace/cli/tree/main#authentication). Here is a high-level summary of the two most common methods:

**Method A: Interactive Setup (Fastest)**
First, ensure you are logged into the Google account (e.g., your personal or Argolis account) with access to Google Drive using the `gcloud` CLI:
```bash
gcloud auth login
```

Once logged in, you can automate project creation and credentials setup by running:
```bash
gws auth setup
```

During this interactive process, **make sure to confirm that the Google Drive API and the Google Slides API are enabled** for the project. 

> **Important Note for Jetski Users:** During the `gws auth setup` flow, you will be required to configure an OAuth Desktop client. Because Jetski is a remote development environment, you must **port forward the localhost port** used in the OAuth redirect URL to your local machine to successfully complete the browser authentication step.

The setup tool will log you in and securely store your credentials in your system keyring. If you use this method, you can **remove the `env` block** from the JSON configuration below.

**Method B: Service Account or Exported Credentials**
If you want explicit control, are running headlessly, or have an existing Google Cloud project, you can provide the path to your credentials file using the `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` environment variable.

#### Server Configuration

```json
"googleworkspace-tool": {
  "command": "npx",
  "args": [
    "@googleworkspace/cli@0.7.0",
    "mcp",
    "-s",
    "drive,slides"
  ],
}
```

*Note: If you used Method B, replace the path in `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` with the actual path to your service account or OAuth credentials file. If you used Method A (`gws auth setup`), you can delete the `env` block completely.*

### 2. Looker Toolbox (`looker`)

The Looker Toolbox is a built-in integration in Jetski. You do not need to manually add JSON configuration for it. Instead, simply search for the Looker integration in your Jetski IDE to enable and configure it with your Looker API credentials and instance URL.
