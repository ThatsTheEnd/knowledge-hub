# 🚀 Publishing and Using Python Packages with Poetry on On-Prem Azure DevOps

This document details the step-by-step process of creating a Python package, uploading it to an **on-prem Azure DevOps instance**, and using it as a dependency in another project. The steps assume the use of **Poetry** for package management.

---

## 📌 Prerequisites
- Azure DevOps feed: **<YOUR_FEED_NAME>**
- Azure DevOps URL: **<YOUR_AZURE_DEVOPS_FEED_URL>**
- Python 3.13 installed
- Poetry installed
- Artifacts keyring installed (`pip install artifacts-keyring`)

---

## 🔑 Creating a New Python Package and Uploading to Azure DevOps

### 1. Create a New Personal Access Token (PAT)
- Go to your Azure DevOps instance.
- Navigate to **User Settings (top-right corner) > Personal Access Tokens**.
- Click **New Token**.
- Set the permissions to include **Packaging Read & Write** for the feed `<YOUR_FEED_NAME>`.
- Click **Create** and save the token somewhere safe.

### 2. Create a New Poetry Package
```bash
poetry new python-package-ados
```

### 3. Modify the `pyproject.toml`
Azure DevOps on-prem only supports **metadata versions 1.0 to 2.1**. Therefore, you must configure Poetry to use an older version of `poetry-core`.

Replace the content of `pyproject.toml` with the following:

```toml
[tool.poetry]
name = "python-package-ados"
version = "0.1.0"
description = "Your package description here"
authors = ["<YOUR_EMAIL>"]
readme = "README.md"
packages = [{include = "python_package_ados", from = "src"}]

[tool.poetry.dependencies]
python = ">=3.13, <4.0"
keyring = ">=25.6.0,<26.0.0"
artifacts-keyring = ">=0.4.0,<0.5.0"
twine = ">=6.1.0,<7.0.0"

[build-system]
requires = ["poetry-core>=1.0.0,<2.0.0"]
build-backend = "poetry.core.masonry.api"
```

### 4. Install Dependencies
```bash
poetry install
```

### 5. Build the Package
```bash
poetry build
```

### 6. Upload the Package to Azure DevOps
```bash
poetry run twine upload --repository-url <YOUR_AZURE_DEVOPS_FEED_URL>/pypi/upload/ dist/*
```

- When prompted, enter your username and your PAT from step 1.

---

## 📦 Using a Package as a Dependency

### 1. Create a New Poetry Project
```bash
poetry new consuming_package
```

### 2. Modify `pyproject.toml`
Replace the content of `pyproject.toml` with the following:

```toml
[tool.poetry]
name = "consuming-package"
version = "0.1.0"
description = "Your consuming project"
authors = ["<YOUR_EMAIL>"]
readme = "README.md"
packages = [{include = "consuming_package", from = "src"}]

[tool.poetry.dependencies]
python = ">=3.13,<4.0.0"
twine = "^6.1.0"
keyring = "^25.6.0"
artifacts-keyring = "^0.4.0"
python-package-ados = {version = "^0.1.0", source = "<YOUR_FEED_NAME>"}

[[tool.poetry.source]]
name = "<YOUR_FEED_NAME>"
url = "<YOUR_AZURE_DEVOPS_FEED_URL>/pypi/simple/"

[build-system]
requires = ["poetry-core>=1.0.0,<2.0.0"]
build-backend = "poetry.core.masonry.api"
```

### 3. Authenticate with Azure DevOps Feed

#### Mac/Linux
```bash
export POETRY_HTTP_BASIC_<YOUR_FEED_NAME>_USERNAME=<YOUR_USERNAME>
export POETRY_HTTP_BASIC_<YOUR_FEED_NAME>_PASSWORD=<YOUR_PERSONAL_ACCESS_TOKEN>
```

#### Windows (Temporary)
```cmd
set POETRY_HTTP_BASIC_<YOUR_FEED_NAME>_USERNAME=<YOUR_USERNAME>
set POETRY_HTTP_BASIC_<YOUR_FEED_NAME>_PASSWORD=<YOUR_PERSONAL_ACCESS_TOKEN>
```

#### Windows (Persistent)
```cmd
setx POETRY_HTTP_BASIC_<YOUR_FEED_NAME>_USERNAME <YOUR_USERNAME>
setx POETRY_HTTP_BASIC_<YOUR_FEED_NAME>_PASSWORD <YOUR_PERSONAL_ACCESS_TOKEN>
```
Restart your terminal after using `setx` for the changes to take effect.

### 4. Add the Dependency
```bash
poetry add python-package-ados --source <YOUR_FEED_NAME>
```

---

## ✅ Testing the Setup
- Make sure that your packages install correctly and are listed in your `pyproject.toml`.
- Verify that your package is accessible from your Azure DevOps feed by running:
  ```bash
  poetry install
  ```
- Ensure no authentication errors occur.

