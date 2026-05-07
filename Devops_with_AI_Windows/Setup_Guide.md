# Windows VM Setup Guide: AI Driven DevOps with Docker

This guide walks through installing every tool required for the lab on the provisioned Windows VM. Complete all sections in order before opening the notebook.

---

## Prerequisites

The following are already provisioned for you:

- Windows VM with internet access
- Azure OpenAI endpoint, API key, and deployment name
- GitHub Copilot license linked to your GitHub account

---

## 1. Python 3.12 (already installed)

Install via winget:

```powershell
winget install --id Python.Python.3.12 -e
```
OR 

1. Open a browser and go to `https://www.python.org/downloads/windows/`
2. Download the **Python 3.12.x Windows installer (64-bit)**
3. Run the installer. On the first screen, check **Add python.exe to PATH** before clicking Install Now
4. When installation completes, open a new PowerShell window and verify:

```powershell
python --version
# Expected: Python 3.12.x

pip --version
# Expected: pip 24.x from ...
```

---

## 2. Git

Install via winget:

```powershell
winget install --id Git.Git -e
```
OR

1. Go to `https://git-scm.com/download/win`
2. Download and run the 64-bit installer
3. Accept all defaults. On the "Adjusting your PATH environment" screen, keep **Git from the command line and also from 3rd-party software** selected
4. Verify in a new PowerShell window:

```powershell
git --version
# Expected: git version 2.x.x.windows.x
```

---

## 3. Visual Studio Code (already installed)

Install via winget:

```powershell
winget install --id Microsoft.VisualStudioCode -e
```
OR

1. Go to `https://code.visualstudio.com/`
2. Download and run the **User Installer (64-bit)**
3. During installation, check both:
   - **Add "Open with Code" action to Windows Explorer file context menu**
   - **Add to PATH**
4. Launch VS Code after installation

### 3.1 Install Required Extensions

Open VS Code. Press `Ctrl+Shift+X` to open the Extensions panel. Search for and install each of the following:

| Extension | Publisher | Purpose |
|---|---|---|
| Python | Microsoft | Jupyter kernel + IntelliSense |
| Jupyter | Microsoft | Run .ipynb notebooks |
| GitHub | GitHub | AI inline suggestions |
| Docker | Microsoft | Dockerfile syntax + container management |

---

## 4. Docker Desktop

Install via winget:

```powershell
winget install --id Docker.DockerDesktop -e
```
OR

1. Go to `https://www.docker.com/products/docker-desktop/`
2. Download **Docker Desktop for Windows (AMD64)**
3. Run the installer. When prompted, keep **Use WSL 2 instead of Hyper-V** selected (recommended)
4. Restart the VM when asked
5. After restart, Docker Desktop launches automatically. Wait for the status bar to show **Engine running**
6. Verify in PowerShell:

```powershell
docker --version
# Expected: Docker version 26.x.x, build ...

docker run --rm hello-world
# Expected: Hello from Docker! ... message
```

### 4.1 WSL 2 Backend (if prompted)

If Docker Desktop asks you to install the WSL 2 Linux kernel update:

1. Go to `https://aka.ms/wsl2kernel`
2. Download and run the MSI
3. Restart Docker Desktop

---

## 5. Jenkins (Local Installation)

Jenkins runs locally on this Windows VM. Follow all steps below to install it, complete initial setup, install required plugins, add credentials, and verify the pipeline works before the lab.

### 5.1 Install Jenkins via winget

Open PowerShell as Administrator and run:

```powershell
winget install --id Jenkins.Jenkins -e --source winget
```

winget installs Jenkins as a Windows service that starts automatically. When the command completes, verify the service is running:

```powershell
Get-Service -Name Jenkins
# Expected: Status = Running
```

Jenkins listens on port 8080 by default. Open a browser and go to:

```
http://localhost:8080
```

### 5.2 Unlock Jenkins and Create an Admin Account

On first launch, Jenkins displays an unlock screen.

1. Retrieve the initial admin password from PowerShell:

```powershell
Get-Content "C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword"
```

2. Paste that password into the unlock screen and click **Continue**
3. On the next screen, click **Install suggested plugins**. Wait for all plugins to finish installing
4. When prompted, fill in the **Create First Admin User** form:
   - Username: choose a username (e.g., `admin`)
   - Password: choose a strong password
   - Full name and email: fill in as required
5. Click **Save and Continue**
6. On the Instance Configuration screen, leave the URL as `http://localhost:8080/` and click **Save and Finish**
7. Click **Start using Jenkins**. You are now on the Jenkins dashboard

### 5.3 Add Credentials to Jenkins

The Jenkinsfile generated in Section 6 of the lab expects two credentials stored in the Jenkins credentials store.

1. Go to **Manage Jenkins > Credentials > System > Global credentials (unrestricted) > Add Credentials**

2. Add the Docker registry credential:
   - Kind: **Username with password**
   - Scope: Global
   - Username: your Azure Container Registry username
   - Password: your registry password or access token
   - ID: `GITHUB_CREDS`
   - Click **Create**

## 6. Terraform

Terraform is used in Section 7 of the notebook to generate and review Azure infrastructure as code. Install it via winget or manually.

### Option A — Install via winget (recommended)

Open PowerShell as Administrator and run:

```powershell
winget install --id Hashicorp.Terraform -e --source winget
```

Close and reopen PowerShell, then verify:

```powershell
terraform --version
# Expected: Terraform v1.x.x
```

### Option B — Manual install

If winget is unavailable or blocked:

```powershell
# 1. Download the latest Terraform zip for Windows AMD64
$version = "1.8.5"
$url = "https://releases.hashicorp.com/terraform/$version/terraform_${version}_windows_amd64.zip"
$dest = "$env:TEMP\terraform.zip"

Invoke-WebRequest -Uri $url -OutFile $dest

# 2. Extract to C:\terraform
Expand-Archive -Path $dest -DestinationPath "C:\terraform" -Force

# 3. Add C:\terraform to the system PATH permanently
[System.Environment]::SetEnvironmentVariable(
    "Path",
    $env:Path + ";C:\terraform",
    [System.EnvironmentVariableTarget]::Machine
)

# 4. Reload PATH in the current session
$env:Path += ";C:\terraform"

# 5. Verify
terraform --version
```

> **Note:** Terraform commands in the notebook (`terraform init`, `terraform plan`, `terraform apply`) are shown for reference only and are not executed in the notebook cells. Run them manually in a terminal with Azure credentials configured.

---

## 7. Create a GitHub Repository and Push lab_output

The Jenkinsfile generated in the notebook must be committed to a Git repository so Jenkins can check it out during the pipeline run. This section creates a new GitHub repository and initialises it from the `lab_output` folder.

### 6.1 Create a New Repository on GitHub

1. Open a browser and go to `https://github.com/new`
2. Log in with the GitHub account that has Copilot access
3. Fill in the repository details:
   - **Repository name:** `ai-devops-lab`
   - **Visibility:** Private (recommended for a lab containing generated configs)
   - Leave **Initialize this repository with a README** unchecked
4. Click **Create repository**
5. Copy the repository URL shown on the next screen. It will look like:
   ```
   https://github.com/<your-username>/ai-devops-lab.git
   ```

### 6.2 Configure Git Identity (first-time setup)

If you have not used Git on this VM before, set your identity. Open PowerShell and run:

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### 6.3 Add a .gitignore to lab_output

Before initialising the repository, create a `.gitignore` file inside `lab_output` so the `.env` file and any local Terraform state are never committed.

Open PowerShell, navigate to your notebook folder, and run:

```powershell
cd lab_output

@"
.env
*.tfstate
*.tfstate.backup
.terraform/
tfplan
"@ | Out-File -FilePath .gitignore -Encoding utf8

type .gitignore
# Verify all five entries are printed
```

### 6.4 Initialise the Repository and Push

Still inside the `lab_output` directory, run the following commands one at a time:

```powershell
# Initialise a new local Git repository
git init

# Stage all generated files (Dockerfile, Jenkinsfile, docker-compose.yml, Terraform, app.py)
git add .

# Verify what will be committed - .env must NOT appear in this list
git status

# Create the first commit
git commit -m "Initial lab output: AI-generated DevOps artifacts"

# Rename the default branch to main
git branch -M main

# Link the local repository to the GitHub remote
git remote add origin https://github.com/<your-username>/ai-devops-lab.git

# Push to GitHub
git push -u origin main
```

After the push completes, refresh the repository page on GitHub. You should see these files:

```
.gitignore
Dockerfile
Dockerfile.broken
Jenkinsfile
app.py
docker-compose.yml
main.tf
requirements.txt
variables.tf
```

> **Important:** If `.env` appears in the file list on GitHub, the push must be undone immediately. Run `git rm --cached .env`, add `.env` to `.gitignore`, commit, and force-push: `git push --force origin main`.

### 6.5 Authenticate Git with GitHub (if prompted)

When you run `git push`, Windows may open a browser window asking you to sign in to GitHub. Complete the login flow. Git Credential Manager, which is bundled with the Git for Windows installer, will store the token securely so you are not prompted again.

If the browser flow does not appear, authenticate manually using a Personal Access Token:

1. Go to `https://github.com/settings/tokens/new`
2. Set a name (e.g., `ai-devops-lab-vm`), set expiry to 30 days, and check the **repo** scope
3. Click **Generate token** and copy the value
4. Re-run `git push` and when prompted for a password, paste the token instead

---

## 8. Environment File for Azure OpenAI

The notebook reads credentials from a `.env` file using `python-dotenv`. Create this file before running the notebook.

1. In VS Code, open the folder where you will place the notebook
2. Create a new file named `.env` (no other extension)
3. Add the following lines, replacing placeholder values with your actual credentials:

```
AZURE_OPENAI_ENDPOINT=https://<your-resource-name>.openai.azure.com/
AZURE_OPENAI_API_KEY=<your-api-key>
AZURE_OPENAI_API_VERSION=2024-02-01
DEPLOYMENT_NAME=<your-deployment-name>
```

4. Save the file
5. Verify the `.env` file is in the same directory as the notebook `.ipynb` file

> **Important:** Never commit the `.env` file to Git. Add `.env` to your `.gitignore` file immediately.

---

## 9. Open the Notebook in VS Code

1. Copy the `AI_Driven_DevOps_Lab.ipynb` file into your working folder (same folder as `.env`)
2. In VS Code, go to **File > Open Folder** and select that folder
3. Click on `AI_Driven_DevOps_Lab.ipynb` in the Explorer panel
4. When prompted to select a kernel, choose **Python 3.12.x**
5. Run Cell 1 (the environment check). Expected output:

```
Python: 3.12.x ...
Docker: Docker version 26.x.x, build ...
Git:    git version 2.x.x.windows.x
```

If Docker shows an error, ensure Docker Desktop is running (check the system tray icon).


---

## Troubleshooting

**Docker Desktop does not start after restart**
Open Task Manager, go to the Services tab, and check that the `com.docker.service` service is running. If not, right-click and start it. Then relaunch Docker Desktop.

**VS Code cannot find the Python kernel**
Press `Ctrl+Shift+P`, type **Python: Select Interpreter**, and choose the Python 3.12 interpreter from the list. If it does not appear, confirm Python was installed with "Add to PATH" checked, then reload VS Code.

**GitHub Copilot shows "not signed in"**
Press `Ctrl+Shift+P`, type **GitHub Copilot: Sign In**, and complete the browser flow again. If the problem persists, uninstall and reinstall the Copilot extension.

**pip install fails with SSL error**
Run PowerShell as Administrator and execute:
```powershell
pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org openai python-dotenv requests
```

**Jenkins service does not start**
Open PowerShell as Administrator and run `Get-Service Jenkins`. If the status is not Running, start it with `Start-Service Jenkins`. If it fails to start, check the Jenkins log at `C:\ProgramData\Jenkins\.jenkins\logs\jenkins.log` for the root cause.

**Jenkins pipeline fails with "sh: not found" or similar**
Jenkins on Windows does not have a Unix shell. All pipeline steps must use `bat` instead of `sh`. If the AI-generated Jenkinsfile contains `sh` steps, replace them with `bat` before running the pipeline.

**Jenkins cannot connect to Docker**
Docker Desktop must be running before Jenkins starts. Start Docker Desktop first, wait for "Engine running" in the system tray, then restart the Jenkins service: `Restart-Service Jenkins` in an Administrator PowerShell window.

The error is:

```
docker: not found
```

Jenkins is running inside a Docker container that does not have Docker installed in it. You need to mount the host's Docker socket into the Jenkins container so it can use the host's Docker daemon.

**Step 1: Stop your current Jenkins container**

```powershell
docker stop jenkins
docker rm jenkins
```

**Step 2: Restart Jenkins with the Docker socket mounted**

```powershell
docker run -d `
  --name jenkins `
  -p 8080:8080 `
  -p 50000:50000 `
  -v jenkins_home:/var/jenkins_home `
  -v //var/run/docker.sock:/var/run/docker.sock `
  jenkins/jenkins:lts
```

**Step 3: Install Docker CLI inside the Jenkins container**

```powershell
docker exec -u root jenkins bash -c "apt-get update && apt-get install -y docker.io"
```

**Step 4: Give Jenkins permission to use the socket**

```powershell
docker exec -u root jenkins bash -c "chmod 666 /var/run/docker.sock"
```

**Step 5: Verify Docker is accessible from inside Jenkins**

```powershell
docker exec jenkins docker --version
# Expected: Docker version 2x.x.x
```