# DevOps TP-2 Report

## ✨Overview✨
This report documents the steps taken and concepts learned while configuring CI/CD pipelines with GitHub Actions, Docker, and SonarCloud.

## ✨Goals and Good Practices✨
- **Goal**: The main objective was to set up a CI/CD pipeline that builds, tests, and deploys a Docker image for the application, ensuring code quality and best practices.
- **Good Practices**: Documenting every step, using secure environment variables, and separating different jobs in the CI/CD pipeline.

## ✨CI/CD Pipeline Details✨

### CI Pipeline - `build-and-test`
- **Checkout Code**: Clones the repository.
- **Set up JDK**: Uses Amazon Corretto JDK 17.
- **Build and Test**: Runs `mvn clean verify` with integration tests using Testcontainers.

### CD Pipeline - `build-and-push-docker-image`
- **Login to DockerHub**: Uses GitHub Secrets for DockerHub credentials.
- **Build and Push Image**: Builds Docker images for the backend, database, and HTTP server, then pushes them to DockerHub.

### Quality Gate - SonarCloud Analysis
- **Run SonarCloud Analysis**: Configured in `main.yml` to analyze code quality on each commit. Uses a specific project and organization key.


## Questions and Answers

### ❓What are testcontainers?

⭐️ Testcontainers are Java libraries that allow integration tests to run in lightweight, throwaway Docker containers. For this TP, Testcontainers were used to spin up a PostgreSQL container to verify database integration.
here's an example of testcontainers in the pom.xml:

<img width="625" alt="image" src="https://github.com/user-attachments/assets/ea7aad98-4ca7-4867-af4b-f82a80f38fca">


### ❓ documenting the First CI with backend test :

⭐️ here's my snippet of code for the main.yaml :

```bash
name: CI Workflow

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'corretto'  
      - name: Build and Test with Maven
        run: mvn clean verify
```

⭐️ PS: it's  validated

<img width="625" alt="image" src="https://github.com/user-attachments/assets/337f1f7c-ef47-433b-be42-741238e54fe4">
<img width="625" alt="image" src="https://github.com/user-attachments/assets/2a69beef-4690-4c4a-942f-0abe0eba0a0a">



### ❓ Why did we use secured variables?

⭐️ Secured variables (GitHub Secrets) are used to protect sensitive information, such as DockerHub credentials, from being exposed in the public repository. They ensure that credentials are only accessible within the GitHub Actions environment.

<img width="625" alt="image" src="https://github.com/user-attachments/assets/7f753a0e-19b3-4939-b8d5-fb43b58db4f4">




### ❓ Why did we use the `needs: build-and-test` on the job?

⭐️ The `needs: build-and-test` directive ensures that the Docker image is only built and pushed if the build-and-test job completes successfully. This dependency prevents deploying an untested or failed build.



### ❓ For what purpose do we need to push Docker images?

⭐️ Pushing Docker images to a repository (like DockerHub) allows for easy deployment and version control of the application. Once an image is built, it can be reused across environments, ensuring consistency.

<img width="625" alt="image" src="https://github.com/user-attachments/assets/75b4281c-ec53-416b-a08a-4e4042201979">

⭐️ here's my snippet of code for the main.yaml :

```bash
name: CI Workflow

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Set up Amazon Corretto JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'corretto'

      - name: Build and Test with Maven
        run: mvn -f simple-api/pom.xml clean verify

  build-and-push-docker-image:
    needs: build-and-test
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin <<< ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push backend image
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-simple-api:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build and push database image
        uses: docker/build-push-action@v3
        with:
          context: ./database
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-database:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build and push http-server image
        uses: docker/build-push-action@v3
        with:
          context: ./http-server
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-http-server:latest
          push: ${{ github.ref == 'refs/heads/main' }}

```

⭐️ PS: It's validated

<img width="1180" alt="image" src="https://github.com/user-attachments/assets/426149ce-4f3b-40fb-b17d-f0a0d8e90546">







### ❓ What is a Quality Gate?

⭐️ A Quality Gate is a set of conditions that code must meet to be considered production-ready. Using SonarCloud, the Quality Gate checks code metrics such as coverage, bugs, and code smells to ensure maintainability and security.

### ❓ Explain the Quality Gate configuration used

⭐️ The Quality Gate in SonarCloud was configured to check metrics such as code coverage, maintainability, reliability, and security. If any thresholds are not met, the build fails, enforcing a standard for code quality.

### ❓Document your quality gate configuration.


⭐️ here's my snippet of code for the main.yaml :

```bash
 name: CI Workflow

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Set up Amazon Corretto JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'corretto'

      - name: Build and Test with Maven
        run: mvn -f simple-api/pom.xml clean verify

      - name: Build and Test with Sonar
        run: mvn -B verify sonar:sonar 
          -Dsonar.projectKey=maghwa_devops-livecoding
          -Dsonar.organization=maghwa
          -Dsonar.host.url=https://sonarcloud.io 
          -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./simple-api/pom.xml


  build-and-push-docker-image:
    needs: build-and-test
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code

        uses: actions/checkout@v2

      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin <<< ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push backend image
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-simple-api:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build and push database image
        uses: docker/build-push-action@v3
        with:
          context: ./database
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-database:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build and push http-server image
        uses: docker/build-push-action@v3
        with:
          context: ./http-server
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-http-server:latest
          push: ${{ github.ref == 'refs/heads/main' }}

```


⭐️ PS: It's Validated

<img width="625" alt="image" src="https://github.com/user-attachments/assets/668042df-efb2-4ec2-893c-6097f9cd221f">






## Split Pipelines (I did it and you already checked it ☺️)
The pipeline is split into two jobs:
- `test-backend`: Runs on `develop` and `main` branches for tests only.
- `build-and-push-docker-image`: Runs on `main` branch only after `test-backend` passes, ensuring the Docker image is pushed only for tested code.
  <img width="625" alt="image" src="https://github.com/user-attachments/assets/afa7b756-f992-48c8-94b8-fdc3e7cafad0">

- here's my snippet code for test-backend:

```bash

name: Test Backend
on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0

      - name: Set up JDK 17
        uses : actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'corretto'

      - name: Build and test with Maven
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=maghwa_devops-livecoding -Dsonar.organization=maghwa -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./simple-api/pom.xml
```

- here's my snippet code for build-and-push-docker-image:

```bash
name: Build and Push Docker Image
on:
  workflow_run:
    branches: [main]
    workflows: ["Test Backend"]
    types:
      - completed

jobs:
  build-and-push-docker-image:
    if: ${{github.event.workflow_run.conclusion == 'success'}}
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}


      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api
          tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-simple-api:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          context: ./database
          tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-database:latest
          push: ${{ github.ref == 'refs/heads/main' }}


      - name: Build image and push front
        uses: docker/build-push-action@v3
        with:
          context: ./http-server
          tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-http-server:latest
          push: ${{ github.ref == 'refs/heads/main' }}
```






## ✨Conclusion✨
This TP exercise provided practical experience in configuring a CI/CD pipeline, handling secure credentials, integrating Docker and SonarCloud, and enforcing quality standards. The setup is modular and adheres to best practices for DevOps workflows.
