# Cloudboosta WordPress — Docker Compose Deployment

This project containerises the **Cloudboosta** business-critical website using Docker Compose. It deploys a fully functional WordPress application backed by MySQL, ensuring consistent execution across all developer environments and eliminating "works on my machine" issues.

Deployed on **AWS EC2 (Ubuntu, eu-west-2)** 

## Architecture
Browser

│

▼

[WordPress container] ── port 80:80

│  (wp_net bridge network)

▼

[MySQL 8.0 container]

│

▼

[Named Volume: db_data]   ← persistent database storage

[Named Volume: wp_data]   ← persistent WordPress files


## Tech Stack

| Component | Technology 
|-----------|------------
| Application | WordPress (latest) 
| Database | MySQL 8.0 
| Containerisation | Docker & Docker Compose 
| Infrastructure | AWS EC2 — Ubuntu, t2.medium, eu-west-2 
| Version Control | GitHub 



## Prerequisites

- AWS EC2 instance (Ubuntu) with port 80 open in the security group
- Docker and Docker Compose installed
- Git installed

### Install Docker on EC2 (Ubuntu)

```bash
sudo apt update && sudo apt install docker.io docker-compose -y
sudo usermod -aG docker ubuntu
# Log out and back in for group change to take effect
```


## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/Emstev/cloudboosta-wordpress.git
cd cloudboosta-wordpress
```

### 2. Create your `.env` file

```bash
cp .env.example .env
```

Edit `.env` and replace the placeholder values with strong credentials:

```env
MYSQL_ROOT_PASSWORD=YourStrongRootPassword
MYSQL_DATABASE=wordpress
MYSQL_USER=wp_user
MYSQL_PASSWORD=YourStrongUserPassword
WORDPRESS_TABLE_PREFIX=cb_
```

> **Security note:** The `.env` file is listed in `.gitignore` and must never be committed to version control.


### 3. Launch the stack

```bash
docker-compose up -d
```


### 4. Verify containers are running

```bash
docker-compose ps
```

Both `cloudboosta_db` and `cloudboosta_wordpress` should show status **Up**.



### 5. Open the WordPress installer

Navigate to your EC2 public IP in a browser:http://<your-ec2-public-ip>


Complete the WordPress 5-minute setup wizard. Database credentials are pre-populated from your `.env` file — no manual entry needed.



## Project Structure

cloudboosta-wordpress/

├── docker-compose.yml    # Service definitions — WordPress + MySQL

├── .env                  # Secret credentials (not committed)

├── .env.example          # Template for onboarding new developers

├── .gitignore

└── README.md



## Useful Commands

| Action | Command |
|--------|---------|
| Start stack | `docker-compose up -d` 
| Stop stack | `docker-compose down` 
| Stop + delete volumes | `docker-compose down -v` 
| View logs | `docker-compose logs -f` 
| WordPress logs only | `docker-compose logs -f wordpress` 
| Rebuild after changes | `docker-compose up -d --build` 
| Shell into WordPress | `docker exec -it cloudboosta_wordpress bash` 
| Shell into MySQL | `docker exec -it cloudboosta_db mysql -u wp_user -p` |



## Key Design Decisions

**Health check on MySQL** — the WordPress container waits for MySQL to pass a health check before starting. This prevents connection errors on first boot when the database isn't ready yet.

**Named volumes** — `db_data` and `wp_data` persist across container restarts and `docker-compose down`. Only `docker-compose down -v` removes them.

**Bridge network** — both services share a private `wp_net` bridge network. WordPress connects to MySQL using the service name `db` as the hostname — no IP addresses needed.

**Custom table prefix** — `cb_` prefix replaces the default `wp_` prefix, reducing exposure to automated SQL injection attacks targeting default WordPress tables.

**Environment variables** — all credentials are injected at runtime from `.env`, keeping secrets out of the codebase and making the stack portable across environments.



## Security Considerations

- Change all default credentials in `.env` before deploying
- Restrict EC2 security group: only expose port 80 (and 443 for HTTPS)
- MySQL is not exposed on a host port — only accessible internally via `wp_net`
- Use HTTPS in production (consider adding an Nginx reverse proxy with Let's Encrypt)


## Author

**Ekwonu Emmanuel**
AWS | Cloud DevOps Engineer
[GitHub](https://github.com/Emstev) · [LinkedIn](https://www.linkedin.com/in/emmanuelekwonu/)


## License

MIT
