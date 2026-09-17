# Jenkins CI/CD Demo

A simple CI/CD pipeline built with Jenkins that automatically checks out a Python project from GitHub, installs its dependencies, runs its test suite with `pytest`, and reaches a deployment stage — all triggered from a Jenkins job running in Docker.

## Overview

This project demonstrates a basic but complete CI/CD workflow:

1. **Checkout** — pulls the latest code from this GitHub repository
2. **Install Dependencies** — installs Python packages listed in `requirements.txt`
3. **Run Tests** — runs the project's test suite using `pytest`
4. **Deploy** — placeholder stage marking where a real deployment step would go

## Tech Stack

- **Jenkins** (`jenkins/jenkins:lts-jdk17`), running in Docker on Windows
- **Python 3.13**
- **pytest** for automated testing
- **Git** for version control integration

## Pipeline Configuration

The pipeline is defined declaratively in the [`Jenkinsfile`](./Jenkinsfile):

```groovy
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling source code from GitHub...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing requirements...'
                sh '''
                    python3 --version
                    python3 -m pip install -r requirements.txt --break-system-packages
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    python3 -m pytest -v
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deployment stage reached.'
            }
        }
    }

    post {
        always {
            echo 'Pipeline run finished.'
        }
        success {
            echo 'All tests passed successfully!'
        }
        failure {
            echo 'Build or tests failed.'
        }
    }
}
```

## Setup

1. Run Jenkins in Docker:
   ```powershell
   docker run -d `
     --name jenkins-server `
     --user root `
     -p 8080:8080 `
     -p 50000:50000 `
     -v jenkins_home:/var/jenkins_home `
     -v //var/run/docker.sock:/var/run/docker.sock `
     jenkins/jenkins:lts-jdk17
   ```
2. Open Jenkins at `http://localhost:8080` and complete the setup wizard.
3. Create a new **Pipeline** job pointing at this repository's `Jenkinsfile`.
4. Click **Build Now** to run the pipeline manually, or configure a GitHub webhook to trigger builds automatically on push.

## Troubleshooting Notes

Two real issues came up while building this pipeline — documenting them here since they're common gotchas:

**1. `permission denied ... docker.sock` / `docker: executable file not found`**
The Jenkins container needs the Docker socket mounted *and* the `docker` CLI installed inside it to run `docker` commands directly. Since this pipeline runs Python natively via `agent any` rather than spinning up Docker containers per stage, this was avoided entirely by not depending on Docker inside the pipeline at all.

**2. `error: externally-managed-environment`**
Newer Debian-based images (including recent Jenkins images) block `pip install` from touching the system Python directly, per [PEP 668](https://peps.python.org/pep-0668/). Fixed by adding the `--break-system-packages` flag to the pip install command — acceptable here since the Jenkins container is a disposable CI environment, not a system you need to protect long-term.

## Next Steps

- [ ] Add a GitHub webhook so builds trigger automatically on push, instead of manual `Build Now`
- [ ] Replace the placeholder Deploy stage with an actual deployment step (e.g. to AWS)
- [ ] Add code coverage reporting to the test stage

we are writing these line to check the jenkins webhook set up 