# Tech Store - Spring Boot Web App

Tech Store is a web application that allows users to perform CRUD operations on tech products. Built using Spring Boot, the app uses Thymeleaf for templating and MySQL for data storage. The application is containerized with Docker and Docker Compose for easy deployment.

## Features

- CRUD operations on tech products
- User authentication using Spring Security
- Support for multiple profiles: `local` and `dev`
- CI/CD pipeline to deploy on VPS using GitHub Actions

---

## Getting Started

### Running Locally

Ensure Docker and Docker Compose are installed on your machine. Clone this repository and navigate to the project root.

Run the following command to start the app:

```bash
docker-compose up -d
```

Once the application is running, you can access it in your browser at [http://localhost:8080](http://localhost:8080). You will be prompted with a login page.

Use the following credentials to log in:

- **Username**: `admin`
- **Password**: `admin`

### Dependencies

The application relies on the following dependencies:

- **[Spring Web](https://spring.io/projects/spring-boot#overview)**: For building the web layer of the application.
- **[Spring Security](https://spring.io/projects/spring-security)**: For implementing authentication and authorization.
- **[Spring Validation](https://docs.spring.io/spring-framework/docs/4.1.x/spring-framework-reference/html/validation.html)**: For validating user input and data.
- **[Spring Data JPA](https://spring.io/projects/spring-data-jpa)**: For interacting with the database using JPA.
- **[MySQL Connector](https://mvnrepository.com/artifact/mysql/mysql-connector-java)**: For establishing a connection between the application and MySQL database.
- **[Thymeleaf](https://www.thymeleaf.org/)**: For server-side rendering of HTML templates.

---

## Project Structure

The application is organized into the following packages:

- **`config`**: Contains Spring Security configuration, defining a single user (`admin/admin`).
- **`controller`**: Houses the REST controller to handle incoming requests.
- **`model`**: Defines the `Product` entity with the following fields:
    - `id`: Unique identifier
    - `productType`: Enum representing the type of product (`SMARTPHONE`, `TABLET`, `LAPTOP`)
    - `brand`: Product brand
    - `model`: Product model
    - `price`: Price of the product
    - `year`: Year of manufacture
- **`repository`**: Contains JPA repositories for database interactions.

---

## Environment Variables

Environment variables for the MySQL configuration are defined in the `docker-compose.yml` file. These include:

- `MYSQL_ROOT_PASSWORD`
- `MYSQL_DATABASE`
- `MYSQL_USER`
- `MYSQL_PASSWORD`

> **Note:** For production, replace the default password with a secure one. You can use OpenSSL to generate a password:

```bash
openssl rand -base64 12
```

---

## Profiles

The application supports two profiles:

- **`local`**: Configured for local development (You can custom application.yml to add your local database credentials)
- **`dev`**: Used for CI/CD in GitHub Actions using the database information in `docker-compose.yml`..

---

## VPS Configuration

To prepare your VPS for deployment, follow these steps:

### 1. Access Your VPS

Log in to your VPS via SSH. If you are using Hostinger, you can access the browser console directly via your HPanel:

```bash
ssh root@your-vps-ip
```

---

### 2. VPS Setup

Run the following commands to update and configure your VPS:

```bash
# Update and upgrade the system
apt update && apt upgrade -y

# Add a new user
adduser youruser

# Grant the new user sudo privileges
usermod -aG sudo youruser

# Configure the firewall
ufw allow OpenSSH
ufw enable
ufw status
```

---

### 3. Local Machine SSH Key Setup

Generate and copy an SSH key for secure access to the VPS:

```bash
# Generate SSH key
ssh-keygen -t rsa -b 4096 -f github-deploy-key -N ""

# Copy the public key to the VPS
ssh-copy-id -i github-deploy-key.pub youruser@your-vps-ip

# Test SSH access
ssh -i github-deploy-key youruser@your-vps-ip

# Print the private key (you'll need this for GitHub Actions)
cat github-deploy-key
```

---

### 4. Install Docker and Docker Compose on the VPS

Log in to your VPS as the newly created user and install Docker and Docker Compose:

```bash
ssh -i github-deploy-key youruser@your-vps-ip

# Install prerequisites
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker and Docker Compose
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add your user to the Docker group
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker --version
docker-compose --version

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker
```

---

### 5. Create Application Directory

Create a directory on the VPS for your application files:

```bash
sudo mkdir -p /opt/techstore
sudo chown -R $USER:$USER /opt/techstore
```

---

### 6. Configure the Firewall

Allow traffic on port 8080 (used by the app):

```bash
sudo ufw allow 8080/tcp
sudo ufw status
```

---

With these steps completed, your VPS is ready for deployment. You can now proceed to the CI/CD setup using GitHub Actions.

---

## CI/CD Pipeline

The application includes a GitHub Actions pipeline for continuous integration and deployment. You can view the full configuration file [here](.github/workflows/ci-cd.yml).
### Key Steps in the Pipeline

1. **Code Checkout**: The repository is cloned using the `actions/checkout` action.
2. **Java Setup**: Configures JDK 17 using the Temurin distribution.
3. **Maven Dependency Cache**: Speeds up builds by caching Maven dependencies.
4. **Build**: Packages the application using the `dev` profile.
5. **File Transfer**: Uses SSH to copy the application JAR, Docker Compose file, and Dockerfile to the VPS.
6. **Deployment**: Deploys the application to the VPS by:
    - Bringing down any running containers.
    - Pulling the latest Docker images.
    - Rebuilding and restarting the containers.
    - Cleaning up unused Docker images.

> **Note**: Ensure the following secrets are set up in your GitHub repository:
> - `VPS_IP`: The IP address of your VPS.
> - `VPS_USER`: The username for the VPS.
> - `SSH_PRIVATE_KEY`: The private SSH key for accessing the VPS.
