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

<img width="625" alt="image" src="https://github.com/user-attachments/assets/426149ce-4f3b-40fb-b17d-f0a0d8e90546">







### ❓ What is a Quality Gate?

⭐️ A Quality Gate is a set of conditions that code must meet to be considered production-ready. Using SonarCloud, the Quality Gate checks code metrics such as coverage, bugs, and code smells to ensure maintainability and security.

### ❓ Explain the Quality Gate configuration used

⭐️ The Quality Gate in SonarCloud was configured to check metrics such as code coverage, maintainability, reliability, and security. If any thresholds are not met, the build fails, enforcing a standard for code quality.

### ❓Documenting the quality gate configuration.


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


in SonarCloud:


<img width="625" alt="image" src="https://github.com/user-attachments/assets/e57239cc-a414-467b-9923-a45bbad11d89">





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


# DevOps TP-3 Report


## ✨Goals and Introduction✨
The goal of this TP is to install and deploy an application automatically using Ansible. By utilizing Ansible's automation capabilities, we streamline the setup and configuration of infrastructure. In this TP, the focus is on using Ansible to:
- Install Docker on a remote server.
- Create and configure various Docker containers for database, network, application, and front-end components.
- Ensure continuous deployment and verify the successful setup of our application environment.

## ✨Inventory and Setup✨
### ⭐️ Inventory File
We created an inventory file at `ansible/inventories/setup.yml` to define the servers that Ansible will manage. This file groups hosts together and specifies SSH access information.
```yaml
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: /path/to/your/private/key
  children:
    prod:
      hosts:
        my-server: 192.168.x.x  # Replace with actual IP or hostname
```

### ⭐️ Hosts Configuration
In this setup, `ansible_user` is set to "admin", and the `ansible_ssh_private_key_file` points to the private SSH key needed for connecting to the remote server.

### ⭐️ Command for Testing
To confirm connectivity to the server, we used the following command:
```bash
ansible all -i inventories/setup.yml -m ping
```
Upon successful configuration, this command should return "pong" as output, verifying that the server is accessible.

## ✨Facts and Variables✨
### ⭐️Facts
Ansible facts are automatically gathered system information variables that help with making decisions in playbooks. For example, facts can tell us the operating system, IP address, and available memory of a host.

### ⭐️Usage
In our setup, we used facts to determine the OS distribution with the command:
```bash
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution"
```
This was essential to ensure compatibility, particularly for the Docker installation process.

## ✨Playbooks✨
### ⭐️Initial Playbook

Our first playbook, `playbook.yml`, was a simple connectivity test:
```yaml
- hosts: all
  gather_facts: false
  become: true
  tasks:
    - name: Test connection
      ping:
```
This playbook verifies that Ansible can connect to all hosts defined in the inventory.

### Advanced Playbook
We created an advanced playbook to handle the Docker installation. This included the following tasks:
1. **Install Required Packages**: Install prerequisites like `apt-transport-https`, `curl`, `gnupg`, and more.
2. **Add Docker's GPG Key**: Add Docker's official GPG key.
3. **Set Up Docker Repository**: Set up the Docker stable repository for downloading Docker packages.
4. **Install Docker**: Install the Docker engine.
5. **Ensure Docker is Running**: Verify that Docker is active.

Example task structure:
```yaml
- name: Install Docker
  hosts: all
  become: true
  tasks:
    - name: Install prerequisites for Docker
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - apt-transport-https
        - ca-certificates
        - curl
        - gnupg
        - lsb-release
        - python3-venv

    - name: Add Docker's official GPG key
      apt_key:
        url: https://download.docker.com/linux/debian/gpg
        state: present

    - name: Set up Docker stable repository
      apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
        state: present
        update_cache: yes

    - name: Install Docker
      apt:
        name: docker-ce
        state: present

    - name: Ensure Docker is running
      service:
        name: docker
        state: started
```

## ✨Roles and Task Organization✨
### ⭐️ Role Structure
Using the command `ansible-galaxy init roles/docker`, we created a role for Docker installation. This structure allows us to separate concerns and modularize our Ansible code. The primary directories used within the role are:
- `tasks`: Contains the main tasks for Docker installation.
- `handlers`: Contains handlers to restart Docker if needed.

### ⭐️Role Execution
In the main playbook which is playbook.yml, we referenced the Docker role as follows:
```yaml
- hosts: all
  gather_facts: true
  become: true
  roles:
    - docker
    - network
    - database
    - app
    - proxy
```

## ✨Application Deployment✨
### ⭐️ Docker Containers
For the application, we set up several Docker containers:
1. **Database Container**:
    ```yaml
    - name: Run database container
      docker_container:
        name: my-db
        image: maghwa88/tp-devops-database
        env:
          POSTGRES_USER: user
          POSTGRES_PASSWORD: pwd
          POSTGRES_DB: db
        networks:
          - my-network
        volumes:
          - db-volume:/var/lib/postgresql/data
        state: started
    ```

2. **Network Configuration**:
   We configured a Docker network to allow containers to communicate:
   ```yaml
   - name: Create Docker network
     docker_network:
       name: my-network
       state:present
   ```

3. **App and Proxy Containers**:
   Similar configuration was used to set up application and proxy containers, ensuring that each container joined `my-network` and could communicate internally.
   ```yaml
   - name: Run proxy container
     docker_container:
       name: httpd
       image: maghwa88/tp-devops-http-server
       ports:
         - "80:80"
    networks:
      - name: my-network

    state: started
      ```
   ```yaml
    ---
    - name: Run application container
      docker_container:
         name: my-api
         image: maghwa88/tp-devops-simple-api
         env:
            DATABASE_HOST: my-db
            DATABASE_PORT: "5342"
            POSTGRES_DB: db
            POSTGRES_USER: user
            POSTGRES_PASSWORD: pwd

         networks:
          - name: my-network

         state: started
   ```

   




## ✨Conclusion✨
Through this TP-3, we gained a deep understanding of:
- How to use Ansible for infrastructure automation.
- Managing roles and modularizing tasks for reusability.
- Deploying applications using Docker and connecting services via networks.



