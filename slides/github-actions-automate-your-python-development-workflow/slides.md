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

# Summary

---

@danyeaw  
  
github.com/danyeaw  
  
dan.yeaw.me  
  
linkedin.com/in/danyeaw  
  
dan@yeaw.me  
