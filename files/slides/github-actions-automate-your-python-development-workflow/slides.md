---
title: GitHub Actions - Automate Your Python Development Workflow
theme: white
highlight-theme: atom-one-dark
revealOptions:
    transition: none
    slideNumber: c
    controls: false
    progress: false
---
## GitHub Actions: Automate Your Python Development Workflow
 
![BeeWare Logo](images/github-actions-logo.png)
<!-- .element style="border: 0; box-shadow: None" -->

---
## Automate your Python workflow

- GitHub Actions is automation built in to GitHub.
- CI/CD. Build, test, and deploy your code right from GitHub.
- Make code reviews, branch management, and issue triaging work the way you want.
---
# Short History
- At GitHub Universe 2018, GitHub launched GitHub Actions in beta

- August 2019, GitHub announced the expansion of GitHub Actions to include
Continuous Integration / Continuous Delivery (CI/CD).

- At Universe 2019, GitHub announced that Actions are out of beta and generally available.

---
## Overview of CI

![Continuous Integration](images/continuous-integration.svg)
<!-- .element style="border: 0; box-shadow: None" -->

---
## Overview of CD 

![Continuous Delivery / Deployment](images/continuous-delivery-deployment.svg)
<!-- .element style="border: 0; box-shadow: None" -->

---
# Before GitHub Actions

![Travis CI](images/travis-ci.svg)
<!-- .element style="border: 0; box-shadow: None" -->
![Circle CI](images/circleci.svg)
<!-- .element style="border: 0; box-shadow: None" -->
![Appveyor](images/appveyor.svg)
<!-- .element style="border: 0; box-shadow: None" -->

---
# Then There was Azure Pipelines

![Azure Pipelines](images/azure.svg)
<!-- .element style="border: 0; box-shadow: None" -->

---

## GitHub Actions CI/CD for a Python Library

![GitHub Azure](images/github-actions-tab.png)
<!-- .element style="border: 0; box-shadow: None" -->

1. Run lint using pre-commit to run Black over the code base
2. Use a matrix build to test the library using Python 2.7, 3.6, 3.7, and 3.8
3. Upload coverage information

---

## GitHub Actions CI/CD Templates

1. Python application - test on a single Python version
2. Python package - test on multiple Python versions
3. Publish Python Package - publish a package to PyPI using Twine

---

## Library Workflow 

![Library Workflow](images/library-workflow.svg)
<!-- .element style="border: 0; box-shadow: None" -->

---

# Lint

---

## Execute on Events

```yaml
name: Build
on:
  pull_request:
  push:
    branches: master
```

---

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: Setup Python
        uses: actions/setup-python@v1
        with:
          python-version: '3.x'
```

---

```yaml
      - name: Install Dependencies
        run: |
          pip install pre-commit
          pre-commit install-hooks
      - name: Lint with pre-commit
        run: pre-commit run --all-files
```

---

# Test

---

```yaml
  test:
    needs: lint
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [2.7, 3.6, 3.7, 3.8]

```

---

```yaml
    steps:
      - uses: actions/checkout@v1
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v1
        with:
          python-version: ${{ matrix.python-version }}
```

---

```yaml
- name: Install Ubuntu Dependencies
  run: >
    sudo apt-get update -q && sudo apt-get install
    --no-install-recommends -y xvfb python3-dev python3-gi
    python3-gi-cairo gir1.2-gtk-3.0 libgirepository1.0-dev
    libcairo2-dev
```

---

```yaml
      - name: Install Poetry
        uses: dschep/install-poetry-action@v1.2
        with:
          version: 1.0.0b3
      - name: Turn off Virtualenvs
        run: poetry config virtualenvs.create false
```

---

```yaml
      - name: Code Climate Coverage Action
        uses: paambaati/codeclimate-action@v2.3.0
        env:
          CC_TEST_REPORTER_ID: 195e9f83022747c8eefa3ec9510dd730081ef111acd99c98ea0efed7f632ff8a
        with:
          coverageCommand: coverage xml
```

---

# CD Workflow
# Upload to PyPI

---

```yaml
on:
  release:
    types: published
```

---

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: Set up Python
      uses: actions/setup-python@v1
      with:
        python-version: '3.x'
```

---

```yaml
- name: Install Poetry
  uses: dschep/install-poetry-action@v1.2
  with:
    version: 1.0.0b3
- name: Install Dependencies
  run: poetry install
- name: Build and publish
  run: |
    poetry build
    poetry publish -u ${{ secrets.PYPI_USERNAME }} -p ${{ secrets.PYPI_PASSWORD }}
```
---

![GitHub Actions Output](images/github-actions-output.png)
<!-- .element style="border: 0; box-shadow: None" -->

---
## Caching Dependencies

```yaml
- name: Use Python Dependency Cache
  uses: actions/cache@v1.0.3
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/poetry.lock') }}
    restore-keys: ${{ runner.os }}-pip-
```
```bash
~\AppData\Local\pip\Cache
~/Library/Caches/pip
```
---

## Test and Deploy a Python Application

---

![App Workflow](images/app-workflow.svg)
<!-- .element style="border: 0; box-shadow: None" -->

---

# Test

---

```yaml
runs-on: ${{ matrix.os }}
strategy:
    matrix:
        os: [ubuntu-latest, windows-latest, macOS-latest]
```

---

# Release

---

```yaml
name: Release

on:
  release:
    types: [created, edited]
```

---

```yaml
      - name: Upload Assets
        uses: AButler/upload-release-assets@v2.0
        with:
          files: 'macos-dmg/*dmg;dist/*;win-installer/*.exe'
          repo-token: ${{ secrets.GITHUB_TOKEN }}
```

---

@danyeaw  
  
github.com/danyeaw  
  
dan.yeaw.me  
  
linkedin.com/in/danyeaw  
  
dan@yeaw.me  
