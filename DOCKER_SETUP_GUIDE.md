# Docker Configuration Reuse Guide

This guide explains how to reuse these Docker configurations in another Laravel application.

## Quick Setup Checklist

When setting up a new Laravel application with Docker, follow these steps:

### 1. Copy Docker Files
Copy these files/folders to your new project:
- `Dockerfile`
- `docker-compose.yml`
- `docker-compose/` folder (nginx and php configs)

### 2. Customize docker-compose.yml

Replace all instances of `laravel_apis` with your application name:

**Service Names:**
- `laravel_apis_app` → `your_app_name_app`
- `laravel_apis_webserver` → `your_app_name_webserver`
- `laravel_apis_app_mysql` → `your_app_name_mysql`
- `laravel_apis_app_phpmyadmin` → `your_app_name_phpmyadmin`
- `laravel_apis_app_redis` → `your_app_name_redis`

**Container Names:**
- `laravel_apis_app` → `your_app_name_app`
- `laravel_apis_webserver_container` → `your_app_name_webserver_container`
- `laravel_apis_mysql_container` → `your_app_name_mysql_container`
- etc.

**Ports (to avoid conflicts):**
- Web server: `6162:80` → `YOUR_PORT:80` (e.g., `8080:80`)
- MySQL: `3337:3306` → `YOUR_PORT:3306` (e.g., `3307:3306`)
- phpMyAdmin: `8383:80` → `YOUR_PORT:80` (e.g., `8081:80`)
- Redis: `7379:6379` → `YOUR_PORT:6379` (e.g., `6380:6379`)

**Database Credentials:**
```yaml
MYSQL_ROOT_PASSWORD: your_secure_password
MYSQL_DATABASE: your_database_name
MYSQL_USER: your_database_user
MYSQL_PASSWORD: your_secure_password
```

**Network & Volume Names:**
- `laravel_apis_app_network` → `your_app_name_network`
- `laravel_apis_app_mysql_data` → `your_app_name_mysql_data`

**Build Args:**
- `user: laravel_apis_user` → `user: your_app_user`
- `uid: 1000` → Keep as `1000` (or match your system user ID)

### 3. Customize Nginx Configuration

**Rename the config file:**
- `docker-compose/nginx/laravel_apis.conf` → `docker-compose/nginx/your_app_name.conf`

**Update fastcgi_pass:**
- Change `fastcgi_pass laravel_apis_app:9000;` to `fastcgi_pass your_app_name_app:9000;`

**Update docker-compose.yml volume mount:**
- Ensure the nginx config path matches your renamed file

### 4. Update .env File

In your Laravel application's `.env` file, set:

```env
DB_CONNECTION=mysql
DB_HOST=your_app_name_mysql  # Must match MySQL service name in docker-compose.yml
DB_PORT=3306
DB_DATABASE=your_database_name  # Must match MYSQL_DATABASE in docker-compose.yml
DB_USERNAME=your_database_user  # Must match MYSQL_USER in docker-compose.yml
DB_PASSWORD=your_secure_password  # Must match MYSQL_PASSWORD in docker-compose.yml

REDIS_HOST=your_app_name_redis  # Must match Redis service name in docker-compose.yml
REDIS_PORT=6379
```

### 5. Dockerfile (Usually No Changes Needed)

The Dockerfile is reusable as-is for most Laravel applications. Only modify if you need:
- Different PHP version (change `FROM php:8.2-fpm`)
- Additional PHP extensions
- Different system dependencies

## Example: Setting Up "MyBlog" Application

1. **Service names:** `myblog_app`, `myblog_webserver`, `myblog_mysql`, etc.
2. **Ports:** `8080:80` (web), `3307:3306` (mysql), `8081:80` (phpmyadmin), `6380:6379` (redis)
3. **Database:** `myblog_db`, user: `myblog_user`, password: `secure_password_123`
4. **Nginx config:** `docker-compose/nginx/myblog.conf`
5. **fastcgi_pass:** `myblog_app:9000`

## Usage

After customization:

```bash
# Build and start containers
docker-compose up --build -d

# Run migrations
docker-compose exec your_app_name_app php artisan migrate

# Access your application
# http://localhost:YOUR_WEB_PORT
```

## Troubleshooting

- **Port conflicts:** Change ports in docker-compose.yml if you get "port already in use" errors
- **Connection refused:** Ensure service names in .env match docker-compose.yml service names
- **504 Gateway Timeout:** Check that PHP-FPM is running: `docker-compose exec your_app_name_app ps aux | grep php-fpm`

