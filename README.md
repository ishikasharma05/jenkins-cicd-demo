# Jenkins CI/CD Demo

A complete CI/CD pipeline built with Jenkins that automatically triggers on every GitHub push, checks out the code, installs dependencies, runs the test suite with `pytest`, and deploys the build — with zero manual steps required after a `git push`.

## Overview

1. **Checkout** — pulls the latest code from this repository
2. **Install Dependencies** — installs Python packages from `requirements.txt`
3. **Run Tests** — runs the test suite with `pytest`
4. **Deploy** — copies the build to a deployment directory inside the Jenkins container

The whole pipeline runs automatically the moment code is pushed to `main`, via a GitHub webhook — no manual "Build Now" clicking required.

## Tech Stack

- **Jenkins** (`jenkins/jenkins:lts-jdk17`), running in Docker on Windows
- **Python 3.13**
- **pytest** for automated testing
- **Git** for version control integration
- **ngrok** to expose the local Jenkins instance for webhook delivery

## Pipeline Configuration

The pipeline is defined declaratively in [`Jenkinsfile`](./Jenkinsfile):

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
                echo 'Deploying application...'
                sh '''
                    mkdir -p /var/jenkins_home/deployed
                    cp -r * /var/jenkins_home/deployed/
                    echo "Deployed files:"
                    ls -la /var/jenkins_home/deployed/
                '''
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

### 1. Run Jenkins in Docker

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

Open `http://localhost:8080`, complete the setup wizard, and create a new **Pipeline** job pointing at this repo's `Jenkinsfile`.

### 2. Expose Jenkins with ngrok (for the webhook)

```powershell
ngrok http 8080
```

This prints a public URL (e.g. `https://your-subdomain.ngrok-free.dev`) that forwards to your local Jenkins. Keep this running — closing the terminal kills the tunnel.

### 3. Add a GitHub webhook

In the repo: **Settings → Webhooks → Add webhook**
- Payload URL: `https://your-subdomain.ngrok-free.dev/github-webhook/`
- Content type: `application/json`
- Event: Just the push event

### 4. Enable the trigger in Jenkins

Job → **Configure → Build Triggers** → check **"GitHub hook trigger for GITScm polling"** → Save.

From this point on, every `git push` to `main` triggers a build automatically.

## Viewing the Deployed Files

The Deploy stage copies the project into `/var/jenkins_home/deployed/` inside the Jenkins container. To check it:

```powershell
docker exec -it jenkins-server ls -la /var/jenkins_home/deployed/
```

Or check the **Console Output** of any completed build in the Jenkins UI — the Deploy stage prints the full file listing at the end of the log.

## Troubleshooting Notes

Real issues hit while building this, documented for reference:

**1. `permission denied ... docker.sock` / `docker: executable file not found`**
Mounting the Docker socket alone doesn't give the container the `docker` CLI binary. Avoided by running Python directly via `agent any` instead of a docker agent, so the pipeline never needs `docker` commands inside the container.

**2. `error: externally-managed-environment`**
Newer Debian-based images block `pip install` from touching system Python directly ([PEP 668](https://peps.python.org/pep-0668/)). Fixed with the `--break-system-packages` flag — acceptable for a disposable CI container.

**3. GitHub can't reach a local Jenkins server**
Jenkins running on `localhost` isn't reachable from GitHub's servers for webhook delivery. Solved with an `ngrok` tunnel to expose it on a public HTTPS URL.

## Possible Improvements

- [ ] Replace local file copy in Deploy with a real cloud deployment (e.g. AWS S3 or EC2)
- [ ] Add code coverage reporting to the test stage
- [ ] Run ngrok as a persistent service instead of a manual terminal session