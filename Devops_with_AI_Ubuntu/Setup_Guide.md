# Setup Guide

Quick start for getting the lab environment running on Linux.

## 1. System packages

Open a terminal as administrator and install the required system tools:

```bash
sudo apt-get install git
sudo apt-get install docker.io
sudo apt-get install python3-pip
```

## 2. Open the project in VS Code

Open the project folder in Visual Studio Code.

## 3. Create the environment file

Create a `.env` file in the project root for your environment variables (API keys, endpoints, etc.).

## 4. Set up the Python virtual environment

Open a new terminal inside VS Code (Terminal → New Terminal) and run:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Your prompt should now show `(.venv)` at the start.

## 5. Install Python packages

```bash
pip install openai python-dotenv requests ipykernel --quiet
```

No output means the install succeeded.

## 6. Select the kernel

In VS Code, open the notebook and select the kernel:

- Click the kernel selector (top right of the notebook)
- Choose the `.venv` environment
- Wait for the kernel to start

## 7. Reload if needed

If the kernel doesn't appear or behaves oddly, reload the VS Code window:

- Press `Ctrl+Shift+P`
- Run **Developer: Reload Window**

## 8. Install any missing packages

If still facing issues in running the notebook, please install Python and Jupyter Notebook extension inside VS Code. 

## 9. Run the notebook

Install any additional packages prompted by the notebook, then click **Run All**.