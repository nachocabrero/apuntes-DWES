# UD09: El Paso a Producción: Docker y Despliegue 📦🚢

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar el problema del despliegue y la solución con Docker
- Crear Dockerfiles para aplicaciones Symfony y Flask
- Usar Docker Compose para orquestar multi-contenedor
- Desplegar aplicaciones en servidores de producción
- Introducir conceptos de CI/CD

---

## 📖 Contenidos

### 9.1 El Problema del Despliegue

**"En mi máquina funciona" 🤣**

Problemas comunes al desplegar:

```
Entorno de Desarrollo          Entorno de Producción
┌──────────────────┐          ┌──────────────────┐
│ PHP 8.3          │          │ PHP 8.1          │
│ Symfony 7        │    ❌     │ Symfony 6        │
│ MySQL 8.0        │  ─────▶  │ MySQL 5.7        │
│ Python 3.12      │          │ Python 3.9       │
│ Node.js 22       │          │ Node.js 18       │
└──────────────────┘          └──────────────────┘
         DIFERENTE!                    FALLA!
```

**Solución: Docker**

```
Entorno de Desarrollo          Entorno de Producción
┌──────────────────┐          ┌──────────────────┐
│ Docker Container │          │ Docker Container │
│ PHP 8.3          │    ✅     │ PHP 8.3          │
│ Symfony 7        │  ─────▶  │ Symfony 7        │
│ MySQL 8.0        │          │ MySQL 8.0        │
│ Python 3.12      │          │ Python 3.12      │
└──────────────────┘          └──────────────────┘
         IGUAL!                    FUNCIONA!
```

---

### 9.2 ¿Qué es Docker?

**Docker** es una plataforma para crear, distribuir y ejecutar aplicaciones en contenedores.

**Contenedor:** Paquete autónomo que incluye todo lo necesario para ejecutar una aplicación.

```
┌─────────────────────────────────────────────────────────┐
│                      Host (Tu PC)                       │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  Contenedor  │  │  Contenedor  │  │  Contenedor  │    │
│  │  Symfony    │  │  Python/Flask│  │  MySQL      │    │
│  │  PHP 8.3    │  │  Python 3.12 │  │  MySQL 8.0  │    │
│  │  Nginx      │  │  Flask       │  │  Extensions │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
│                                                         │
│  Todos aislados entre sí, pero pueden comunicarse       │
└─────────────────────────────────────────────────────────┘
```

**Conceptos clave:**

- **Imagen:** Plantilla para crear contenedores (como una clase)
- **Contenedor:** Instancia de una imagen (como un objeto)
- **Dockerfile:** Instrucciones para crear una imagen
- **Docker Compose:** Herramienta para definir y ejecutar multi-contenedor

---

### 9.3 Instalación de Docker

**Instalar Docker (Ubuntu/Debian):**

```bash
# Instalar Docker
sudo apt update
sudo apt install docker.io docker-compose-plugin

# Iniciar Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verificar instalación
docker --version
docker compose version
```

**Usar Docker sin sudo (opcional):**

```bash
# Añadir usuario al grupo docker
sudo usermod -aG docker $USER

# Reiniciar sesión
# O ejecutar: newgrp docker
```

---

### 9.4 Primeros Pasos con Docker

**Imágenes básicas:**

```bash
# Buscar imagen
docker search php

# Descargar imagen
docker pull php:8.3

# Ejecutar contenedor
docker run -d --nombre mi-php php:8.3

# Ver contenedores activos
docker ps

# Ver todos los contenedores
docker ps -a

# Parar contenedor
docker stop mi-php

# Eliminar contenedor
docker rm mi-php

# Eliminar imagen
docker rmi php:8.3
```

**Ejecutar PHP interactivo:**

```bash
# Ejecutar PHP en un contenedor
docker run -it --rm php:8.3 php -r "echo 'Hola Mundo';"

# Ejecutar shell interactivo
docker run -it --rm php:8.3 bash
```

---

### 9.5 Crear un Dockerfile para Symfony

**Dockerfile para Symfony:**

```dockerfile
# Dockerfile
FROM php:8.3-fpm

# Instentar dependencias del sistema
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libonig-dev \
    libxml2-dev \
    libzip-dev \
    && docker-php-ext-install \
    pdo_mysql \
    zip \
    exif \
    mbstring \
    intl \
    && rm -rf /var/lib/apt/lists/*

# Instalar Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Crear directorio de trabajo
WORKDIR /var/www/html

# Copiar archivos del proyecto
COPY . .

# Instalar dependencias de PHP
RUN composer install --no-dev --optimize-autoloader

# Exponer puerto
EXPOSE 9000

# Comando por defecto
CMD ["php-fpm"]
```

**Dockerfile para Nginx:**

```dockerfile
# Dockerfile-nginx
FROM nginx:alpine

# Copiar configuración de Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copiar archivos públicos
COPY public/ /var/www/html/public/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Configuración de Nginx:**

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;
    root /var/www/html/public;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/index\.php$ {
        fastcgi_pass php:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ \.php$ {
        return 404;
    }

    location ~ /\.(env|git|svn) {
        deny all;
    }
}
```

---

### 9.6 Crear un Dockerfile para Flask

**Dockerfile para Flask:**

```dockerfile
# Dockerfile
FROM python:3.12-slim

# Instalar dependencias
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Crear directorio de trabajo
WORKDIR /app

# Copiar requirements e instalar dependencias
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar archivos del proyecto
COPY . .

# Exponer puerto
EXPOSE 5001

# Comando por defecto
CMD ["python", "app.py"]
```

**requirements.txt:**

```
flask==3.0.0
requests==2.31.0
python-dotenv==1.0.0
```

---

### 9.7 Docker Compose: Orquestación Multi-Contenedor

**Docker Compose** permite definir y ejecutar aplicaciones multi-contenedor.

**docker-compose.yml para el proyecto completo:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Base de datos MySQL
  mysql:
    image: mysql:8.0
    container_name: dwes-mysql
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: foro
      MYSQL_USER: foro_user
      MYSQL_PASSWORD: foro_password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - dwes-network

  # Aplicación Symfony
  symfony:
    build:
      context: ./symfony-app
      dockerfile: Dockerfile
    container_name: dwes-symfony
    ports:
      - "8000:80"
    environment:
      DATABASE_URL: "mysql://foro_user:foro_password@mysql:3306/foro?serverVersion=8.0"
      AI_SERVICE_URL: "http://flask:5001"
    depends_on:
      - mysql
      - flask
    networks:
      - dwes-network
    volumes:
      - ./symfony-app:/var/www/html
      - symfony_vendor:/var/www/html/vendor

  # Microservicio Flask (IA)
  flask:
    build:
      context: ./flask-app
      dockerfile: Dockerfile
    container_name: dwes-flask
    ports:
      - "5001:5001"
    environment:
      OPENAI_API_KEY: ${OPENAI_API_KEY}
    networks:
      - dwes-network

volumes:
  mysql_data:
  symfony_vendor:

networks:
  dwes-network:
    driver: bridge
```

**Ejecutar con Docker Compose:**

```bash
# Construir y levantar todos los contenedores
docker compose up --build

# Levantar en segundo plano
docker compose up -d

# Ver logs
docker compose logs -f

# Parar todos los contenedores
docker compose down

# Parar y eliminar volúmenes
docker compose down -v
```

---

### 9.8 Estructura del Proyecto con Docker

```
apuntes-dwes-proyecto/
├── docker-compose.yml
├── symfony-app/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── composer.json
│   ├── src/
│   ├── templates/
│   └── public/
├── flask-app/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
└── .env
```

---

### 9.9 Despliegue en Producción

**Opciones de despliegue:**

| Opción | Descripción | Precio |
| --- | --- | --- |
| **VPS** | Servidor virtual privado | €5-20/mes |
| **AWS** | Amazon Web Services | Por uso |
| **DigitalOcean** | Droplets (VPS) | €5-20/mes |
| **Heroku** | Plataforma PaaS | Gratis-€25/mes |
| **Railway** | Plataforma de despliegue | Por uso |
| **Render** | Plataforma de despliegue | Gratis-$$$ |

**Desplegar en un VPS con Docker:**

```bash
# 1. Conectar al servidor
ssh usuario@tu-servidor.com

# 2. Instalar Docker (si no está)
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# 3. Clonar el repositorio
git clone https://github.com/nachocabrero/apuntes-DWES.git
cd apuntes-DWES

# 4. Copiar el archivo .env con las credenciales
cp .env.example .env
# Editar .env con las credenciales reales

# 5. Construir y levantar los contenedores
docker compose up --build -d

# 6. Verificar que todo funciona
docker ps
docker compose logs symfony
```

**Configurar Nginx como proxy inverso:**

```nginx
# /etc/nginx/sites-available/dwes
server {
    listen 80;
    server_name dwes.ejemplo.com;

    location / {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api {
        proxy_pass http://localhost:5001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
    }
}
```

**Activar la configuración:**

```bash
sudo ln -s /etc/nginx/sites-available/dwes /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### 9.10 CI/CD: Integración y Despliegue Continuo

**CI/CD** (Continuous Integration/Continuous Deployment) automatiza el proceso de despliegue.

**Ejemplo con GitHub Actions:**

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/dwes
            git pull origin main
            docker compose down
            docker compose up --build -d
            docker image prune -f
```

**Configurar secrets en GitHub:**

1. Ir al repositorio → Settings → Secrets and variables → Actions
2. Añadir:
   - `SERVER_HOST`: IP del servidor
   - `SERVER_USER`: Usuario SSH
   - `SSH_PRIVATE_KEY`: Clave privada SSH

---

### 9.11 Comandos Útiles de Docker

```bash
# Ver contenedores activos
docker ps

# Ver todos los contenedores
docker ps -a

# Ver imágenes
docker images

# Ver logs de un contenedor
docker logs symfony
docker logs -f symfony  # Seguir logs en tiempo real

# Entrar a un contenedor
docker exec -it symfony bash

# Ejecutar comando en contenedor
docker exec symfony php bin/console doctrine:database:create
docker exec symfony php bin/console doctrine:migrations:migrate

# Parar contenedor
docker stop symfony

# Iniciar contenedor
docker start symfony

# Eliminar contenedor
docker rm symfony

# Eliminar imagen
docker rmi php:8.3

# Limpiar todo (contenedores, imágenes, volúmenes)
docker system prune -a --volumes

# Ver uso de disco
docker system df
```

---

## 💻 Ejercicios

### Ejercicio 9.1: Primer Contenedor

1. Instalar Docker
2. Ejecutar un contenedor PHP
3. Ejecutar un comando PHP dentro del contenedor

```bash
docker run --rm php:8.3 php -r "echo 'Hola Docker!';"
```

---

### Ejercicio 9.2: Dockerfile para Symfony

1. Crear un Dockerfile para una aplicación Symfony
2. Construir la imagen: `docker build -t mi-symfony .`
3. Ejecutar el contenedor: `docker run -p 8000:80 mi-symfony`

---

### Ejercicio 9.3: Docker Compose

1. Crear un `docker-compose.yml` con:
   - Aplicación Symfony
   - Base de datos MySQL
   - Microservicio Flask
2. Levantar todo: `docker compose up --build`
3. Verificar que todos los contenedores están activos

---

### Ejercicio 9.4: Desplegar en Producción

1. Configurar un VPS (DigitalOcean, AWS, etc.)
2. Instalar Docker
3. Clonar el repositorio
4. Levantar los contenedores con Docker Compose
5. Configurar Nginx como proxy inverso

---

### Ejercicio 9.5: Proyecto Final - Arquitectura Completa

**Objetivo:** Crear un `docker-compose.yml` que levante toda la arquitectura del foro.

**Requisitos:**

- Symfony en un contenedor
- MySQL en otro contenedor
- Flask (microservicio de IA) en otro contenedor
- Todos comunicándose entre sí
- Base de datos persistente con volúmenes
- Variables de entorno para configuración

```yaml
# docker-compose.yml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: foro
      MYSQL_USER: foro_user
      MYSQL_PASSWORD: foro_pass
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - dwes

  symfony:
    build: ./symfony-app
    ports:
      - "8000:80"
    environment:
      DATABASE_URL: "mysql://foro_user:foro_pass@mysql:3306/foro"
      AI_SERVICE_URL: "http://flask:5001"
    depends_on:
      - mysql
      - flask
    networks:
      - dwes

  flask:
    build: ./flask-app
    ports:
      - "5001:5001"
    environment:
      OPENAI_API_KEY: ${OPENAI_API_KEY}
    networks:
      - dwes

volumes:
  mysql_data:

networks:
  dwes:
    driver: bridge
```

---

## 🏗️ Proyecto Final: Despliegue Completo

**Objetivo:** Desplegar toda la arquitectura del foro en producción.

**Pasos:**

1. **Preparar el servidor:**

```bash
# Instalar Docker
curl -fsSL https://get.docker.com | sh

# Crear directorio
mkdir -p /var/www/dwes
cd /var/www/dwes
```

2. **Clonar el repositorio:**

```bash
git clone https://github.com/nachocabrero/apuntes-DWES.git .
```

3. **Configurar variables de entorno:**

```bash
# .env
OPENAI_API_KEY=tu_clave_aqui
DATABASE_URL=mysql://user:pass@mysql:3306/foro
```

4. **Levantar los contenedores:**

```bash
docker compose up --build -d
```

5. **Verificar:**

```bash
docker ps
docker compose logs -f
```

6. **Configurar Nginx:**

```bash
sudo nano /etc/nginx/sites-available/dwes
sudo ln -s /etc/nginx/sites-available/dwes /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

7. **Acceder:** `http://tu-dominio.com`

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ El problema del despliegue y la solución con Docker
- ✅ Conceptos básicos de Docker (imágenes, contenedores, Dockerfile)
- ✅ Crear Dockerfiles para Symfony y Flask
- ✅ Docker Compose para orquestar multi-contenedor
- ✅ Desplegar en un servidor de producción
- ✅ Configurar Nginx como proxy inverso
- ✅ Introducción a CI/CD con GitHub Actions

---

## 🎓 ¡Curso Completado!

Has completado las 9 unidades de DWES:

1. ✅ **UD01:** Ecosistema Backend (PHP, Composer, Git)
2. ✅ **UD02:** Symfony MVC (routing, controllers, Twig)
3. ✅ **UD03:** Doctrine ORM (entidades, CRUD, migraciones)
4. ✅ **UD04:** Seguridad (auth, sesiones, roles)
5. ✅ **UD05:** APIs REST (API Platform, JWT, OpenAPI)
6. ✅ **UD06:** Python y Flask (microservicios)
7. ✅ **UD07:** Integración de IA (OpenAI, Hugging Face)
8. ✅ **UD08:** BaaS (Supabase, Firebase)
9. ✅ **UD09:** Docker y Despliegue (producción, CI/CD)

**Ahora tienes las herramientas para:**

- Crear aplicaciones web completas con Symfony
- Construir microservicios con Python/Flask
- Integrar servicios de IA
- Usar BaaS para desarrollo rápido
- Contenerizar y desplegar en producción

---

## 🔗 Recursos Adicionales

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Hub](https://hub.docker.com/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Nginx Documentation](https://nginx.org/en/docs/)
