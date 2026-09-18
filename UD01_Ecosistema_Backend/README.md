# UD01: El Ecosistema del Desarrollo Backend Moderno 🌍

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar la arquitectura cliente-servidor y el protocolo HTTP
- Comprender el rol de las APIs en la web moderna
- Configurar un entorno de desarrollo profesional con PHP y Composer
- Utilizar Git para control de versiones
- Crear scripts PHP modernos con sintaxis actual

---

## 📖 Contenidos

### 1.1 Arquitectura Cliente-Servidor

La web moderna se basa en la arquitectura **cliente-servidor**:

```
┌─────────────┐         ┌──────────────┐
│   Cliente    │────────▶│   Servidor    │
│ (Navegador)  │◀────────│   (Backend)   │
└─────────────┘         └──────────────┘
      Petición              Respuesta
      (HTTP)                (HTTP)
```

**Flujo de una petición HTTP:**

1. El cliente (navegador) envía una petición HTTP al servidor
2. El servidor procesa la petición (PHP, Python, Node.js, etc.)
3. El servidor devuelve una respuesta HTTP al cliente
4. El cliente renderiza la respuesta (HTML, JSON, etc.)

**Métodos HTTP principales:**

| Método | Acción | Ejemplo |
| --- | --- | --- |
| GET | Obtener recursos | `GET /api/usuarios` |
| POST | Crear recursos | `POST /api/usuarios` |
| PUT | Actualizar recursos | `PUT /api/usuarios/1` |
| DELETE | Eliminar recursos | `DELETE /api/usuarios/1` |

---

### 1.2 APIs: El Lenguaje de la Web Moderna

Una **API** (Application Programming Interface) es un conjunto de reglas que permite que dos aplicaciones se comuniquen entre sí.

**Tipos de APIs:**

- **REST**: Arquitectura basada en HTTP (la más común)
- **GraphQL**: Lenguaje de consulta flexible
- **gRPC**: Comunicación de alto rendimiento
- **SOAP**: Protocolo antiguo pero aún usado

**Ejemplo de API REST:**

```
GET    /api/usuarios          → Obtener todos los usuarios
GET    /api/usuarios/1        → Obtener el usuario 1
POST   /api/usuarios          → Crear un nuevo usuario
PUT    /api/usuarios/1        → Actualizar el usuario 1
DELETE /api/usuarios/1        → Eliminar el usuario 1
```

**Formato de respuesta (JSON):**

```json
{
  "id": 1,
  "nombre": "Nacho",
  "email": "nacho@ieshlanz.es",
  "rol": "profesor"
}
```

---

### 1.3 PHP en el Siglo XXI

PHP ha evolucionado enormemente. Las versiones modernas (8.x) incluyen:

- **Tipado fuerte**: Variables y parámetros tipados
- **Arrow functions**: Sintaxis compacta para funciones
- **Match expression**: Alternativa moderna a switch
- **Named arguments**: Argumentos con nombre
- **Nullsafe operator**: Manejo elegante de nulos

**Ejemplo de PHP moderno:**

```php
<?php

// Tipado fuerte
function saludar(string $nombre): string {
    return "¡Hola, {$nombre}!";
}

// Arrow function (PHP 7.4+)
$sumar = fn($a, $b) => $a + $b;
echo $sumar(2, 3); // 5

// Match expression (PHP 8.0+)
$dia = "lunes";
$tipo = match($dia) {
    "lunes", "martes", "miércoles", "jueves", "viernes" => "laboral",
    "sábado", "domingo" => "fin de semana",
    default => "desconocido"
};
echo $tipo; // laboral

// Named arguments (PHP 8.0+)
function crearUsuario(string $nombre, string $email, int $edad = 18): void {
    echo "Usuario: {$nombre}, Email: {$email}, Edad: {$edad}\n";
}

crearUsuario(email: "nacho@ieshlanz.es", nombre: "Nacho", edad: 46);
```

---

### 1.4 Composer: Gestión de Dependencias

**Composer** es el gestor de paquetes de PHP. Permite:

- Instalar librerías de terceros
- Gestionar dependencias del proyecto
- Autoloading de clases

**Instalación:**

```bash
# Verificar si Composer está instalado
composer --version

# Si no está instalado (Linux/macOS)
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

**Crear un proyecto con Composer:**

```bash
# Crear un nuevo proyecto
composer init

# Responder a las preguntas:
# Package name: michatoproyecto/miprograma
# Description: Mi primer proyecto con Composer
# Author: Nacho Cabrero
# License: MIT

# Instalar una librería
composer require monolog/monolog

# Instalar una librería como dependencia de desarrollo
composer require --dev phpunit/phpunit
```

**Archivo `composer.json`:**

```json
{
    "name": "michatoproyecto/miprograma",
    "description": "Mi primer proyecto con Composer",
    "type": "project",
    "require": {
        "php": ">=8.1",
        "monolog/monolog": "^2.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^9.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

**Autoloading con PSR-4:**

```
proyecto/
├── composer.json
├── src/
│   └── Usuario.php
└── vendor/
    └── autoload.php
```

```php
// src/Usuario.php
<?php

namespace App;

class Usuario {
    public function __construct(
        public string $nombre,
        public string $email
    ) {}
}
```

```php
// index.php
<?php

require __DIR__ . '/vendor/autoload.php';

$usuario = new App\Usuario("Nacho", "nacho@ieshlanz.es");
echo $usuario->nombre; // Nacho
```

---

### 1.5 Control de Versiones con Git

**Git** es el sistema de control de versiones más utilizado.

**Comandos esenciales:**

```bash
# Inicializar un repositorio
git init

# Añadir archivos al staging
git add .

# Ver el estado
git status

# Crear un commit
git commit -m "Primer commit: Hola Mundo en PHP"

# Ver historial
git log

# Añadir remoto (GitHub)
git remote add origin https://github.com/nachocabrero/miprograma.git

# Subir cambios
git push -u origin main
```

**Flujo de trabajo:**

```
Trabajo → git add → git commit → git push
```

**Ejemplo de flujo:**

```bash
# Crear un archivo
echo '<?php echo "Hola Mundo"; ?>' > hola.php

# Ver cambios
git status

# Añadir al staging
git add hola.php

# Crear commit
git commit -m "Añadir hola.php"

# Subir a GitHub
git push origin main
```

---

### 1.6 Configuración del Entorno de Desarrollo

**Requisitos:**

- PHP 8.1+
- Composer
- Git
- Editor de código (VS Code recomendado)
- Servidor web local (XAMPP, MAMP, o Docker)

**Instalación de PHP (Ubuntu/Debian):**

```bash
# Instalar PHP y extensiones comunes
sudo apt update
sudo apt install php php-cli php-mbstring php-xml php-curl php-zip

# Verificar instalación
php -v

# Instalar Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

**Verificar entorno:**

```bash
php -v          # PHP 8.3.x
composer --version  # Composer 2.x
git --version   # git 2.x
```

---

## 💻 Ejercicios

### Ejercicio 1.1: Hola Mundo con PHP

Crea un archivo `hola.php` que muestre "¡Hola, Mundo!" y ejecútalo desde la terminal:

```bash
php hola.php
```

**Solución:**

```php
<?php

echo "¡Hola, Mundo!\n";
```

---

### Ejercicio 1.2: Variables y Tipos

Crea un script que declare variables de diferentes tipos y las muestre:

```php
<?php

$nombre = "Nacho";
$edad = 46;
$altura = 1.84;
$esProfesor = true;

echo "Nombre: {$nombre}\n";
echo "Edad: {$edad}\n";
echo "Altura: {$altura}m\n";
echo "¿Es profesor? " . ($esProfesor ? "Sí" : "No") . "\n";
```

---

### Ejercicio 1.3: Arrays y Estructuras de Control

Crea un array de usuarios y recórrelo mostrando su información:

```php
<?php

$usuarios = [
    ["nombre" => "Nacho", "email" => "nacho@ieshlanz.es", "rol" => "profesor"],
    ["nombre" => "Celia", "email" => "celia@ieshlanz.es", "rol" => "alumna"],
    ["nombre" => "Pedro", "email" => "pedro@ieshlanz.es", "rol" => "alumno"]
];

foreach ($usuarios as $usuario) {
    echo "Usuario: {$usuario['nombre']}\n";
    echo "Email: {$usuario['email']}\n";
    echo "Rol: {$usuario['rol']}\n";
    echo "---\n";
}
```

---

### Ejercicio 1.4: Funciones

Crea funciones para calcular el IMC y determinar si una persona está en su peso ideal:

```php
<?php

function calcularIMC(float $peso, float $altura): float {
    return $peso / ($altura * $altura);
}

function determinarEstado(float $imc): string {
    if ($imc < 18.5) {
        return "Bajo peso";
    } elseif ($imc < 25) {
        return "Peso normal";
    } elseif ($imc < 30) {
        return "Sobrepeso";
    } else {
        return "Obesidad";
    }
}

$peso = 120;
$altura = 1.84;

$imc = calcularIMC($peso, $altura);
$estado = determinarEstado($imc);

echo "IMC: {$imc}\n";
echo "Estado: {$estado}\n";
```

---

### Ejercicio 1.5: POO Básica

Crea una clase `Usuario` con propiedades y métodos:

```php
<?php

class Usuario {
    public string $nombre;
    public string $email;
    public string $rol;

    public function __construct(string $nombre, string $email, string $rol = "usuario") {
        $this->nombre = $nombre;
        $this->email = $email;
        $this->rol = $rol;
    }

    public function getInfo(): string {
        return "Usuario: {$this->nombre}, Email: {$this->email}, Rol: {$this->rol}";
    }
}

$usuario = new Usuario("Nacho", "nacho@ieshlanz.es", "profesor");
echo $usuario->getInfo();
```

---

### Ejercicio 1.6: Composer y Autoloading

1. Crea un directorio `src/` con una clase `Calculadora`
2. Configura `composer.json` con autoloading PSR-4
3. Usa la clase en `index.php`

**Solución:**

```php
// src/Calculadora.php
<?php

namespace App;

class Calculadora {
    public function sumar(float $a, float $b): float {
        return $a + $b;
    }

    public function restar(float $a, float $b): float {
        return $a - $b;
    }

    public function multiplicar(float $a, float $b): float {
        return $a * $b;
    }

    public function dividir(float $a, float $b): float {
        if ($b == 0) {
            throw new InvalidArgumentException("No se puede dividir por cero");
        }
        return $a / $b;
    }
}
```

```php
// index.php
<?php

require __DIR__ . '/vendor/autoload.php';

$calc = new App\Calculadora();

echo "2 + 3 = " . $calc->sumar(2, 3) . "\n";
echo "10 - 4 = " . $calc->restar(10, 4) . "\n";
echo "5 * 6 = " . $calc->multiplicar(5, 6) . "\n";
echo "15 / 3 = " . $calc->dividir(15, 3) . "\n";
```

---

### Ejercicio 1.7: Git - Primer Commit

1. Inicializa un repositorio Git
2. Crea un archivo `README.md` con la descripción del proyecto
3. Haz el primer commit
4. Crea un repositorio en GitHub
5. Sube los cambios

```bash
git init
echo "# Mi Proyecto PHP" > README.md
git add .
git commit -m "Primer commit: README.md"
git remote add origin https://github.com/nachocabrero/mi-proyecto-php.git
git push -u origin main
```

---

## 🏗️ Proyecto Práctico: Hola Mundo con Composer

**Objetivo:** Crear un proyecto PHP completo con Composer, autoloading y Git.

**Pasos:**

1. **Crear el directorio del proyecto:**

```bash
mkdir mi-proyecto-php
cd mi-proyecto-php
```

2. **Inicializar con Composer:**

```bash
composer init
# Responder a las preguntas
```

3. **Crear la estructura de carpetas:**

```
mi-proyecto-php/
├── composer.json
├── src/
│   └── App.php
├── tests/
└── README.md
```

4. **Crear la clase principal:**

```php
// src/App.php
<?php

namespace App;

class App {
    public function saludar(string $nombre): string {
        return "¡Hola, {$nombre}! Bienvenido a mi proyecto PHP.";
    }

    public function getVersion(): string {
        return "1.0.0";
    }
}
```

5. **Crear el archivo principal:**

```php
// index.php
<?php

require __DIR__ . '/vendor/autoload.php';

$app = new App\App();

echo $app->saludar("Mundo") . "\n";
echo "Versión: " . $app->getVersion() . "\n";
```

6. **Ejecutar el proyecto:**

```bash
php index.php
```

7. **Configurar Git:**

```bash
git init
git add .
git commit -m "Inicializar proyecto con Composer"
```

8. **Subir a GitHub:**

```bash
git remote add origin https://github.com/nachocabrero/mi-proyecto-php.git
git push -u origin main
```

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ La arquitectura cliente-servidor y el protocolo HTTP
- ✅ El rol de las APIs en la web moderna
- ✅ PHP moderno (8.x) con tipado fuerte, arrow functions y match
- ✅ Composer para gestión de dependencias y autoloading
- ✅ Git para control de versiones
- ✅ Configuración del entorno de desarrollo

**Próxima unidad:** UD02 - Construcción de Aplicaciones con Symfony (I)

---

## 🔗 Recursos Adicionales

- [Documentación oficial de PHP](https://www.php.net/manual/es/)
- [Documentación de Composer](https://getcomposer.org/doc/)
- [Documentación de Git](https://git-scm.com/doc)
- [PHP The Right Way](https://phptherightway.com/)
- [Modern PHP](https://phptherightway.com/)
