# UD02: Construcción de Aplicaciones con Symfony (I) - El Núcleo del Framework 🚀

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar el patrón MVC y su importancia en el desarrollo web
- Instalar y configurar Symfony
- Crear rutas y controladores
- Usar Twig para generar vistas dinámicas
- Construir una aplicación web básica con Symfony

---

## 📖 Contenidos

### 2.1 El Patrón MVC

**MVC** (Modelo-Vista-Controlador) es un patrón de arquitectura de software que separa la aplicación en tres componentes:

```
┌─────────────────────────────────────────────────────────┐
│                      Cliente (Navegador)                  │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼ Petición HTTP
┌─────────────────────────────────────────────────────────┐
│                    Servidor (Symfony)                     │
│ ┌─────────────┐  ┌──────────────┐  ┌────────────────┐  │
│ │ Controlador  │──▶│   Modelo     │──▶│     Vista      │  │
│ │ (Lógica)    │  │ (Datos)      │  │ (Presentación) │  │
│ └─────────────┘  └──────────────┘  └────────────────┘  │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼ Respuesta HTTP
┌─────────────────────────────────────────────────────────┐
│                      Cliente (Navegador)                  │
└─────────────────────────────────────────────────────────┘
```

**Componentes:**

- **Modelo**: Gestiona los datos y la lógica de negocio
- **Vista**: Se encarga de la presentación (HTML, CSS, JS)
- **Controlador**: Recibe la petición, interactúa con el Modelo y selecciona la Vista

**Ventajas de MVC:**

- Separación de responsabilidades
- Código más organizado y mantenible
- Facilita el trabajo en equipo
- Reutilización de código

---

### 2.2 Instalación de Symfony

**Requisitos:**

- PHP 8.2+
- Composer
- Symfony CLI (recomendado)

**Instalación:**

```bash
# Instalar Symfony CLI
composer global require symfony-cli

# Crear un nuevo proyecto Symfony
symfony new mi-app-web --webapp

# Entrar al directorio
cd mi-app-web

# Iniciar el servidor de desarrollo
symfony server:start
```

**Estructura de un proyecto Symfony:**

```
mi-app-web/
├── config/
│   ├── packages/          # Configuración de bundles
│   ├── routes/            # Rutas de la aplicación
│   └── bundles.php        # Bundles activos
├── public/                # Raíz del servidor web
│   └── index.php          # Punto de entrada
├── src/
│   ├── Controller/        # Controladores
│   ├── Entity/            # Entidades (Doctrine)
│   ├── Repository/        # Repositorios (Doctrine)
│   ├── Service/           # Servicios
│   └── Kernel.php         # Kernel de la aplicación
├── templates/             # Plantillas Twig
├── tests/                 # Tests
├── migrations/            # Migraciones de base de datos
├── composer.json
└── symfony.lock
```

---

### 2.3 Enrutamiento

El enrutamiento mapea URLs a controladores.

**Método 1: Usando atributos (PHP 8+):**

```php
// src/Controller/InicioController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class InicioController
{
    #[Route('/', name: 'inicio')]
    public function index(): Response
    {
        return new Response('¡Bienvenido a mi aplicación!');
    }
}
```

**Método 2: Usando atributos con parámetros:**

```php
// src/Controller/UsuarioController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class UsuarioController
{
    #[Route('/usuario/{id}', name: 'usuario_show')]
    public function show(int $id): Response
    {
        return new Response("Mostrando usuario con ID: {$id}");
    }

    #[Route('/usuario/{nombre}/{email}', name: 'usuario_info')]
    public function info(string $nombre, string $email): Response
    {
        return new Response("Nombre: {$nombre}, Email: {$email}");
    }
}
```

**Método 3: Usando el componente Request:**

```php
// src/Controller/UsuarioController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class UsuarioController
{
    #[Route('/buscar', name: 'buscar')]
    public function buscar(Request $request): Response
    {
        $termino = $request->query->get('termino', '');
        return new Response("Buscando: {$termino}");
    }
}
```

---

### 2.4 Controladores

Los controladores contienen la lógica de negocio de cada petición.

**Controlador básico:**

```php
// src/Controller/ProductoController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ProductoController
{
    #[Route('/productos', name: 'productos')]
    public function listar(): Response
    {
        $productos = [
            ["id" => 1, "nombre" => "Portátil", "precio" => 899.99],
            ["id" => 2, "nombre" => "Ratón", "precio" => 29.99],
            ["id" => 3, "nombre" => "Teclado", "precio" => 59.99]
        ];

        return new Response(json_encode($productos));
    }
}
```

**Controlador con inyección de dependencias:**

```php
// src/Controller/ProductoController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ProductoController
{
    #[Route('/productos', name: 'productos')]
    public function listar(): Response
    {
        // Lógica para obtener productos de la base de datos
        $productos = $this->obtenerProductos();

        return $this->render('producto/lista.html.twig', [
            'productos' => $productos,
        ]);
    }

    private function obtenerProductos(): array
    {
        // Simulación de consulta a base de datos
        return [
            ["id" => 1, "nombre" => "Portátil", "precio" => 899.99],
            ["id" => 2, "nombre" => "Ratón", "precio" => 29.99],
            ["id" => 3, "nombre" => "Teclado", "precio" => 59.99]
        ];
    }
}
```

---

### 2.5 Motor de Plantillas Twig

**Twig** es el motor de plantillas de Symfony. Permite separar la lógica de la presentación.

**Plantilla básica:**

```twig
{# templates/inicio/index.html.twig #}
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Bienvenido</title>
</head>
<body>
    <h1>¡Hola, {{ nombre }}!</h1>
    <p>Bienvenido a mi aplicación Symfony.</p>
</body>
</html>
```

**Controlador que usa Twig:**

```php
// src/Controller/InicioController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class InicioController
{
    #[Route('/hola/{nombre}', name: 'hola')]
    public function hola(string $nombre): Response
    {
        return $this->render('inicio/hola.html.twig', [
            'nombre' => $nombre,
        ]);
    }
}
```

**Twig: Variables y expresiones:**

```twig
{# templates/producto/lista.html.twig #}
<!DOCTYPE html>
<html>
<head>
    <title>Productos</title>
</head>
<body>
    <h1>Lista de Productos</h1>

    {# Mostrar variable #}
    <p>Total de productos: {{ productos|length }}</p>

    {# Bucle #}
    <ul>
        {% for producto in productos %}
            <li>
                {{ producto.nombre }} - {{ producto.precio }}€
                {% if producto.precio > 100 %}
                    <span class="caro">Caro</span>
                {% else %}
                    <span class="barato">Económico</span>
                {% endif %}
            </li>
        {% else %}
            <li>No hay productos disponibles</li>
        {% endfor %}
    </ul>
</body>
</html>
```

**Twig: Filtros comunes:**

```twig
{# Filtros útiles #}
<p>Nombre en mayúsculas: {{ nombre|upper }}</p>
<p>Nombre en minúsculas: {{ nombre|lower }}</p>
<p>Longitud: {{ nombre|length }}</p>
<p>Redondear: {{ 3.14159|round(2) }}</p>
<p>Fecha: {{ fecha|date('d/m/Y') }}</p>
<p>Truncar: {{ texto|truncate(50) }}</p>
<p>Valor por defecto: {{ dato|default('No disponible') }}</p>
```

**Twig: Layouts y herencia:**

```twig
{# templates/base.html.twig #}
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Mi Aplicación{% endblock %}</title>
    {% block stylesheets %}{% endblock %}
</head>
<body>
    <nav>
        <a href="{{ path('inicio') }}">Inicio</a>
        <a href="{{ path('productos') }}">Productos</a>
        <a href="{{ path('contacto') }}">Contacto</a>
    </nav>

    <main>
        {% block body %}{% endblock %}
    </main>

    {% block javascripts %}{% endblock %}
</body>
</html>
```

```twig
{# templates/producto/lista.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Productos - Mi Aplicación{% endblock %}

{% block body %}
    <h1>Lista de Productos</h1>
    <ul>
        {% for producto in productos %}
            <li>{{ producto.nombre }} - {{ producto.precio }}€</li>
        {% endfor %}
    </ul>
{% endblock %}
```

---

### 2.6 Proyecto: Foro de Discusión (Parte 1)

Vamos a crear la estructura inicial de un foro de discusión.

**Estructura de páginas:**

- `/` - Página de inicio
- `/temas` - Listado de temas
- `/tema/{id}` - Detalle de un tema
- `/contacto` - Página de contacto

**Controlador de Inicio:**

```php
// src/Controller/InicioController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class InicioController
{
    #[Route('/', name: 'inicio')]
    public function index(): Response
    {
        return $this->render('inicio/index.html.twig', [
            'titulo' => 'Foro de Discusión',
            'descripcion' => 'Un espacio para compartir ideas y conocimientos',
        ]);
    }
}
```

```twig
{# templates/inicio/index.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}{{ titulo }}{% endblock %}

{% block body %}
    <div class="hero">
        <h1>{{ titulo }}</h1>
        <p>{{ descripcion }}</p>
        <a href="{{ path('temas') }}" class="btn">Ver Temas</a>
    </div>
{% endblock %}
```

**Controlador de Temas:**

```php
// src/Controller/TemaController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class TemaController
{
    #[Route('/temas', name: 'temas')]
    public function listar(): Response
    {
        $temas = [
            [
                'id' => 1,
                'titulo' => '¿Cómo aprender PHP?',
                'autor' => 'Nacho',
                'fecha' => '2024-01-15',
                'respuestas' => 5
            ],
            [
                'id' => 2,
                'titulo' => 'Symfony vs Laravel',
                'autor' => 'Celia',
                'fecha' => '2024-01-16',
                'respuestas' => 12
            ],
            [
                'id' => 3,
                'titulo' => 'Mejores prácticas de código',
                'autor' => 'Pedro',
                'fecha' => '2024-01-17',
                'respuestas' => 8
            ]
        ];

        return $this->render('tema/lista.html.twig', [
            'temas' => $temas,
        ]);
    }

    #[Route('/tema/{id}', name: 'tema_show')]
    public function show(int $id): Response
    {
        // Simulación de datos
        $tema = [
            'id' => $id,
            'titulo' => '¿Cómo aprender PHP?',
            'autor' => 'Nacho',
            'fecha' => '2024-01-15',
            'contenido' => 'Hola a todos, estoy empezando con PHP y me gustaría saber por dónde empezar...',
            'respuestas' => [
                ['autor' => 'Celia', 'contenido' => 'Te recomiendo empezar con la documentación oficial...', 'fecha' => '2024-01-15'],
                ['autor' => 'Pedro', 'contenido' => 'Yo empecé con tutoriales de YouTube...', 'fecha' => '2024-01-16']
            ]
        ];

        return $this->render('tema/show.html.twig', [
            'tema' => $tema,
        ]);
    }
}
```

```twig
{# templates/tema/lista.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Temas - Foro{% endblock %}

{% block body %}
    <h1>Temas del Foro</h1>
    <a href="{{ path('nuevo_tema') }}" class="btn">Nuevo Tema</a>

    <ul class="temas-lista">
        {% for tema in temas %}
            <li>
                <a href="{{ path('tema_show', {id: tema.id}) }}">{{ tema.titulo }}</a>
                <span class="autor">por {{ tema.autor }}</span>
                <span class="respuestas">{{ tema.respuestas }} respuestas</span>
            </li>
        {% else %}
            <li>No hay temas aún. ¡Sé el primero en crear uno!</li>
        {% endfor %}
    </ul>
{% endblock %}
```

```twig
{# templates/tema/show.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}{{ tema.titulo }} - Foro{% endblock %}

{% block body %}
    <article>
        <h1>{{ tema.titulo }}</h1>
        <p class="meta">Por {{ tema.autor }} el {{ tema.fecha|date('d/m/Y') }}</p>
        <div class="contenido">
            {{ tema.contenido }}
        </div>
    </article>

    <section class="respuestas">
        <h2>Respuestas ({{ tema.respuestas|length }})</h2>
        {% for respuesta in tema.respuestas %}
            <div class="respuesta">
                <p><strong>{{ respuesta.autor }}</strong> - {{ respuesta.fecha|date('d/m/Y') }}</p>
                <p>{{ respuesta.contenido }}</p>
            </div>
        {% endfor %}
    </section>

    <a href="{{ path('temas') }}" class="btn">Volver a Temas</a>
{% endblock %}
```

**Controlador de Contacto:**

```php
// src/Controller/ContactoController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ContactoController
{
    #[Route('/contacto', name: 'contacto')]
    public function index(): Response
    {
        return $this->render('contacto/index.html.twig', [
            'email' => 'contacto@foro.com',
            'telefono' => '+34 900 000 000',
        ]);
    }
}
```

```twig
{# templates/contacto/index.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Contacto - Foro{% endblock %}

{% block body %}
    <h1>Contacto</h1>
    <p>Email: <a href="mailto:{{ email }}">{{ email }}</a></p>
    <p>Teléfono: {{ telefono }}</p>
{% endblock %}
```

---

## 💻 Ejercicios

### Ejercicio 2.1: Instalar Symfony

1. Instala Symfony CLI
2. Crea un nuevo proyecto Symfony
3. Inicia el servidor de desarrollo
4. Accede a `http://localhost:8000`

---

### Ejercicio 2.2: Crear Rutas Básicas

Crea un controlador con las siguientes rutas:

- `/` - Página de inicio
- `/hola/{nombre}` - Saludar al usuario
- `/suma/{a}/{b}` - Sumar dos números

```php
// src/Controller/PruebaController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class PruebaController
{
    #[Route('/hola/{nombre}', name: 'hola')]
    public function hola(string $nombre): Response
    {
        return new Response("¡Hola, {$nombre}!");
    }

    #[Route('/suma/{a}/{b}', name: 'suma')]
    public function suma(int $a, int $b): Response
    {
        return new Response("La suma de {$a} y {$b} es: " . ($a + $b));
    }
}
```

---

### Ejercicio 2.3: Plantillas Twig

Crea una plantilla Twig que muestre una lista de tareas:

```twig
{# templates/tarea/lista.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Tareas{% endblock %}

{% block body %}
    <h1>Mis Tareas</h1>
    <ul>
        {% for tarea in tareas %}
            <li>
                {{ tarea.titulo }}
                {% if tarea.completada %}
                    ✅
                {% else %}
                    ⬜
                {% endif %}
            </li>
        {% endfor %}
    </ul>
{% endblock %}
```

```php
// src/Controller/TareaController.php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class TareaController
{
    #[Route('/tareas', name: 'tareas')]
    public function listar(): Response
    {
        $tareas = [
            ['titulo' => 'Estudiar PHP', 'completada' => true],
            ['titulo' => 'Estudiar Symfony', 'completada' => false],
            ['titulo' => 'Hacer ejercicio', 'completada' => false],
        ];

        return $this->render('tarea/lista.html.twig', [
            'tareas' => $tareas,
        ]);
    }
}
```

---

### Ejercicio 2.4: Layout con Herencia

Crea un layout base y extiéndelo en varias plantillas:

```twig
{# templates/base.html.twig #}
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Mi Foro{% endblock %}</title>
</head>
<body>
    <header>
        <nav>
            <a href="{{ path('inicio') }}">Inicio</a>
            <a href="{{ path('temas') }}">Temas</a>
            <a href="{{ path('contacto') }}">Contacto</a>
        </nav>
    </header>
    <main>
        {% block body %}{% endblock %}
    </main>
    <footer>
        <p>&copy; 2024 Foro de Discusión</p>
    </footer>
</body>
</html>
```

---

### Ejercicio 2.5: Proyecto - Ampliar el Foro

Añade al proyecto del foro:

1. Una página de "Acerca de"
2. Un buscador de temas (con parámetro de consulta)
3. Un contador de temas y respuestas en la página de inicio

```php
// src/Controller/InicioController.php
#[Route('/acerca', name: 'acerca')]
public function acerca(): Response
{
    return $this->render('inicio/acerca.html.twig', [
        'descripcion' => 'Foro de discusión para estudiantes de DWES',
        'version' => '1.0.0',
    ]);
}

#[Route('/buscar', name: 'buscar')]
public function buscar(Request $request): Response
{
    $termino = $request->query->get('q', '');
    // Lógica de búsqueda...
    return $this->render('inicio/buscar.html.twig', [
        'termino' => $termino,
    ]);
}
```

---

## 🏗️ Proyecto Práctico: Foro de Discusión (Parte 1)

**Objetivo:** Crear la estructura inicial de un foro de discusión con Symfony.

**Páginas a crear:**

1. **Inicio** (`/`) - Página principal con información del foro
2. **Listado de Temas** (`/temas`) - Lista de todos los temas
3. **Detalle de Tema** (`/tema/{id}`) - Vista de un tema con respuestas
4. **Contacto** (`/contacto`) - Página de contacto

**Estructura de archivos:**

```
templates/
├── base.html.twig
├── inicio/
│   └── index.html.twig
├── tema/
│   ├── lista.html.twig
│   └── show.html.twig
└── contacto/
    └── index.html.twig

src/Controller/
├── InicioController.php
├── TemaController.php
└── ContactoController.php
```

**Pasos:**

1. Crear el proyecto Symfony: `symfony new foro-discusion --webapp`
2. Crear los controladores
3. Crear las plantillas Twig
4. Configurar las rutas
5. Probar en `http://localhost:8000`

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ El patrón MVC y su importancia
- ✅ Instalación y estructura de Symfony
- ✅ Enrutamiento con atributos PHP
- ✅ Controladores y lógica de negocio
- ✅ Twig para plantillas dinámicas
- ✅ Herencia de plantillas con `{% extends %}`
- ✅ Creación de una aplicación web básica

**Próxima unidad:** UD03 - Persistencia de Datos con Doctrine ORM

---

## 🔗 Recursos Adicionales

- [Documentación oficial de Symfony](https://symfony.com/doc/current/index.html)
- [Documentación de Twig](https://twig.symfony.com/doc/)
- [Symfony Docs: Routing](https://symfony.com/doc/current/routing.html)
- [Symfony Docs: Controllers](https://symfony.com/doc/current/controllers.html)
- [Symfony Docs: Templates](https://symfony.com/doc/current/templates.html)
