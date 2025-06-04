# Deploying SonarQube with Docker Compose

## ✅ Why Use Docker Compose for SonarQube?
• Simplified Setup: Easily define and run SonarQube and its dependencies in one file.  
• Portability: Version-controlled configuration that works across environments.  
• Resource Management: Isolate and manage services with Docker containers.

---

## 📋 Prerequisites
• Docker: Ensure Docker is installed and running.  
• Docker Compose (CLI v2+): Confirm with:
```bash
docker compose version
```  
• Minimum Resources: At least 2GB RAM and 2 CPUs for smooth operation.  
• Linux-Specific Requirement: Set vm.max_map_count to 262144 (required for Elasticsearch).

---

## ⚠️ Set vm.max_map_count on Linux
Before starting SonarQube, run the following command to avoid memory-related errors from Elasticsearch:
```bash
sudo sysctl -w vm.max_map_count=262144
```
To make this setting persistent after reboot, add this line to `/etc/sysctl.conf`:
```bash
vm.max_map_count=262144
```
Then reload sysctl:
```bash
sudo sysctl -p
```

---

## 🛠️ Step-by-Step Deployment

### 1. Create a Project Directory
```bash
mkdir sonarqube-docker && cd sonarqube-docker
```

---

### 2. Create docker-compose.yml
Create a file named `docker-compose.yml` with the following content:
```yaml
services:
  sonarqube:
    image: sonarqube:community
    hostname: sonarqube
    container_name: sonarqube
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"

  db:
    image: postgres:13
    hostname: postgresql
    container_name: postgresql
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar
    volumes:
      - postgresql_data:/var/lib/postgresql/data

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql_data:
```

SonarQube Community Build release cycle model:  
`YY.M.0.BuildNumber`

---

### 3. Start the Services
In the same directory as your `docker-compose.yml`, run:
```bash
docker compose up -d
```
This will:  
• Pull the required images.  
• Start SonarQube and PostgreSQL in detached mode.

---

### 4. Access SonarQube
Open your browser and go to:  
http://localhost:9000

**Default credentials:**  
• Username: `admin`  
• Password: `admin`  

You’ll be prompted to change the password on first login.

---

## ⚙️ Customization Options
• **Change the Web Port:**
```yaml
ports:
  - "9080:9000"
```
• **Use an External PostgreSQL Database:** Replace the `postgres` service and adjust the SonarQube environment variables.  
• **Add Plugins:** Drop `.jar` files into the `sonarqube_extensions` volume path (`/opt/sonarqube/extensions`).  
• **Increase Resources:** If SonarQube fails to start, increase memory/CPU in Docker settings (if using Docker Desktop).

---

## 🧹 Stopping & Cleaning Up
• To stop services:
```bash
docker compose down
```

• To remove all containers and volumes:
```bash
docker compose down --volumes
```
