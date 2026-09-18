# UD05: Creación de Servicios: APIs REST con Symfony 📡

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar los principios de la arquitectura REST
- Construir endpoints que devuelven JSON
- Usar API Platform para crear APIs rápidamente
- Implementar autenticación JWT
- Documentar APIs con OpenAPI (Swagger)

---

## 📖 Contenidos

### 5.1 Principios de REST

**REST** (Representational State Transfer) es un estilo de arquitectura para APIs.

**Principios REST:**

1. **Recursos identificados por URLs:** `/api/usuarios`, `/api/temas/1`
2. **Métodos HTTP:** GET, POST, PUT, DELETE
3. **Respuestas en JSON:** `{"id": 1, "nombre": "Nacho"}`
4. **Sin estado:** Cada petición es independiente
5. **Códigos de estado HTTP:** 200, 201, 400, 401, 404, 500

**Ejemplo de API REST:**

```
GET    /api/usuarios          → 200 OK → [ {...}, {...} ]
GET    /api/usuarios/1        → 200 OK → { "id": 1, ... }
POST   /api/usuarios          → 201 Created → { "id": 2, ... }
PUT    /api/usuarios/1        → 200 OK → { "id": 1, ... }
DELETE /api/usuarios/1        → 204 No Content
```

**Códigos de estado HTTP:**

| Código | Significado | Uso |
| --- | --- | --- |
| 200 | OK | Petición exitosa |
| 201 | Created | Recurso creado |
| 204 | No Content | Eliminación exitosa |
| 400 | Bad Request | Petición inválida |
| 401 | Unauthorized | No autenticado |
| 403 | Forbidden | Sin permiso |
| 404 | Not Found | Recurso no encontrado |
| 500 | Internal Server Error | Error del servidor |

---

### 5.2 Construir Endpoints JSON

**Controlador que devuelve JSON:**

```php
// src/Controller/Api/UsuarioController.php
<?php

namespace App\Controller\Api;

use App\Entity\Usuario;
use App\Repository\UsuarioRepository;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

#[Route('/api/usuarios', name: 'api_usuario_')]
class UsuarioController extends AbstractController
{
    #[Route('', name: 'listar', methods: ['GET'])]
    public function listar(UsuarioRepository $usuarioRepository): JsonResponse
    {
        $usuarios = $usuarioRepository->findAll();

        $data = [];
        foreach ($usuarios as $usuario) {
            $data[] = [
                'id' => $usuario->getId(),
                'nombre' => $usuario->getNombre(),
                'email' => $usuario->getEmail(),
                'rol' => $usuario->getRol(),
            ];
        }

        return new JsonResponse($data);
    }

    #[Route('/{id}', name: 'mostrar', methods: ['GET'])]
    public function mostrar(int $id, UsuarioRepository $usuarioRepository): JsonResponse
    {
        $usuario = $usuarioRepository->find($id);

        if (!$usuario) {
            return new JsonResponse(['error' => 'Usuario no encontrado'], Response::HTTP_NOT_FOUND);
        }

        return new JsonResponse([
            'id' => $usuario->getId(),
            'nombre' => $usuario->getNombre(),
            'email' => $usuario->getEmail(),
            'rol' => $usuario->getRol(),
        ]);
    }

    #[Route('', name: 'crear', methods: ['POST'])]
    public function crear(Request $request, EntityManagerInterface $entityManager): JsonResponse
    {
        $data = json_decode($request->getContent(), true);

        $usuario = new Usuario();
        $usuario->setNombre($data['nombre'] ?? '');
        $usuario->setEmail($data['email'] ?? '');
        $usuario->setRol($data['rol'] ?? 'usuario');

        $entityManager->persist($usuario);
        $entityManager->flush();

        return new JsonResponse([
            'id' => $usuario->getId(),
            'nombre' => $usuario->getNombre(),
            'email' => $usuario->getEmail(),
            'rol' => $usuario->getRol(),
        ], Response::HTTP_CREATED);
    }

    #[Route('/{id}', name: 'actualizar', methods: ['PUT'])]
    public function actualizar(int $id, Request $request, EntityManagerInterface $entityManager): JsonResponse
    {
        $usuario = $usuarioRepository->find($id);

        if (!$usuario) {
            return new JsonResponse(['error' => 'Usuario no encontrado'], Response::HTTP_NOT_FOUND);
        }

        $data = json_decode($request->getContent(), true);

        if (isset($data['nombre'])) {
            $usuario->setNombre($data['nombre']);
        }
        if (isset($data['email'])) {
            $usuario->setEmail($data['email']);
        }
        if (isset($data['rol'])) {
            $usuario->setRol($data['rol']);
        }

        $entityManager->flush();

        return new JsonResponse([
            'id' => $usuario->getId(),
            'nombre' => $usuario->getNombre(),
            'email' => $usuario->getEmail(),
            'rol' => $usuario->getRol(),
        ]);
    }

    #[Route('/{id}', name: 'eliminar', methods: ['DELETE'])]
    public function eliminar(int $id, EntityManagerInterface $entityManager): JsonResponse
    {
        $usuario = $usuarioRepository->find($id);

        if (!$usuario) {
            return new JsonResponse(['error' => 'Usuario no encontrado'], Response::HTTP_NOT_FOUND);
        }

        $entityManager->remove($usuario);
        $entityManager->flush();

        return new JsonResponse(null, Response::HTTP_NO_CONTENT);
    }
}
```

---

### 5.3 API Platform

**API Platform** es un framework para crear APIs REST y GraphQL de forma rápida.

**Instalar API Platform:**

```bash
symfony composer require api
```

**Crear un recurso con API Platform:**

```php
// src/Entity/Tema.php
<?php

namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\Get;
use ApiPlatform\Metadata\GetCollection;
use ApiPlatform\Metadata\Post;
use ApiPlatform\Metadata\Put;
use ApiPlatform\Metadata\Delete;
use Doctrine\ORM\Mapping as ORM;

#[ApiResource(
    operations: [
        new GetCollection(),
        new Get(),
        new Post(
            validation: ['groups' => ['crear']],
            denormalization_context: ['groups' => ['crear']]
        ),
        new Put(),
        new Delete(),
    ]
)]
#[ORM\Entity]
class Tema
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    public ?int $id = null;

    #[ORM\Column(length: 200)]
    public ?string $titulo = null;

    #[ORM\Column(type: 'text')]
    public ?string $contenido = null;

    // Getters y setters...
}
```

**Endpoints automáticos:**

```
GET    /api/temas              → Listar todos los temas
GET    /api/temas/1            → Obtener tema 1
POST   /api/temas              → Crear tema
PUT    /api/temas/1            → Actualizar tema 1
DELETE /api/temas/1            → Eliminar tema 1
```

**Documentación automática:**

- Swagger UI: `http://localhost:8000/api/docs`
- OpenAPI: `http://localhost:8000/api/docs.json`

---

### 5.4 Autenticación JWT

**JWT** (JSON Web Token) es un estándar para autenticación sin estado.

**Instalar LexikJWTAuthenticationBundle:**

```bash
symfony composer require lexik/jwt-authentication-bundle
```

**Generar clave JWT:**

```bash
# Generar clave privada y pública
symfony console lexik:jwt:generate-keypair
```

**Configuración:**

```env
# .env
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=tu_contraseña_secreta
```

```yaml
# config/packages/lexik_jwt_authentication.yaml
lexik_jwt_authentication:
    secret_key: '%env(resolve:JWT_SECRET_KEY)%'
    public_key: '%env(resolve:JWT_PUBLIC_KEY)%'
    pass_phrase: '%env(JWT_PASSPHRASE)%'
    token_ttl: 3600
```

**Endpoint de login para obtener token:**

```php
// src/Controller/Api/AuthController.php
<?php

namespace App\Controller\Api;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;

class AuthController extends AbstractController
{
    #[Route('/api/login', name: 'api_login', methods: ['POST'])]
    public function login(Request $request): JsonResponse
    {
        $username = $request->request->get('email');
        $password = $request->request->get('password');

        // Autenticar usuario
        $user = $this->getUser();

        if (!$user) {
            return new JsonResponse(['error' => 'Credenciales incorrectas'], Response::HTTP_UNAUTHORIZED);
        }

        // Generar token JWT
        $token = $this->container->get('lexik_jwt_authentication.jwt_manager')
            ->generate($user);

        return new JsonResponse([
            'token' => $token,
            'user' => [
                'id' => $user->getId(),
                'nombre' => $user->getNombre(),
                'email' => $user->getEmail(),
                'rol' => $user->getRol(),
            ],
        ]);
    }
}
```

**Proteger endpoints con JWT:**

```yaml
# config/packages/security.yaml
security:
    firewalls:
        api:
            pattern: ^/api
            jwt: ~

    access_control:
        - { path: ^/api/login, roles: PUBLIC_ACCESS }
        - { path: ^/api, roles: IS_AUTHENTICATED_FULLY }
```

**Usar el token en peticiones:**

```bash
# Obtener token
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email": "nacho@ieshlanz.es", "password": "mi_password"}'

# Usar el token
curl http://localhost:8000/api/temas \
  -H "Authorization: Bearer TU_TOKEN_AQUI"
```

---

### 5.5 Documentación con OpenAPI (Swagger)

**API Platform genera documentación automáticamente.**

**Acceder a la documentación:**

```
http://localhost:8000/api/docs
```

**Personalizar la documentación:**

```php
#[ApiResource(
    operations: [
        new GetCollection(
            normalization_context: ['groups' => ['tema:read']],
            denormalization_context: ['groups' => ['tema:write']]
        ),
    ],
    attributes: [
        'openapi' => [
            'info' => [
                'title' => 'API del Foro',
                'description' => 'API para gestionar el foro de discusión',
                'version' => '1.0.0',
            ],
        ],
    ]
)]
```

**Grupos de normalización:**

```php
#[ORM\Entity]
class Tema
{
    #[Groups(['tema:read', 'tema:write'])]
    public ?int $id = null;

    #[Groups(['tema:read', 'tema:write'])]
    public ?string $titulo = null;

    #[Groups(['tema:read'])]
    public ?string $contenido = null;

    #[Groups(['tema:read'])]
    public ?Usuario $autor = null;
}
```

---

### 5.6 Manejo de Errores

**Excepciones personalizadas:**

```php
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;
use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;

#[Route('/api/temas/{id}', name: 'api_tema_mostrar', methods: ['GET'])]
public function mostrar(int $id, TemaRepository $temaRepository): JsonResponse
{
    $tema = $temaRepository->find($id);

    if (!$tema) {
        throw new NotFoundHttpException('Tema no encontrado');
    }

    return new JsonResponse([
        'id' => $tema->getId(),
        'titulo' => $tema->getTitulo(),
        'contenido' => $tema->getContenido(),
    ]);
}
```

**Validación de datos:**

```php
use Symfony\Component\Validator\Constraints as Assert;

class Tema
{
    #[Assert\NotBlank]
    #[Assert\Length(min: 3, max: 200)]
    public ?string $titulo = null;

    #[Assert\NotBlank]
    public ?string $contenido = null;
}
```

---

## 💻 Ejercicios

### Ejercicio 5.1: API CRUD Básica

Crea una API REST para gestionar productos:

```
GET    /api/productos          → Listar productos
GET    /api/productos/1        → Obtener producto 1
POST   /api/productos          → Crear producto
PUT    /api/productos/1        → Actualizar producto 1
DELETE /api/productos/1        → Eliminar producto 1
```

---

### Ejercicio 5.2: API con API Platform

1. Instalar API Platform
2. Crear un recurso `Tema`
3. Acceder a `http://localhost:8000/api/docs`
4. Probar los endpoints con curl o Postman

---

### Ejercicio 5.3: Autenticación JWT

1. Instalar LexikJWTAuthenticationBundle
2. Generar claves JWT
3. Crear endpoint `/api/login`
4. Probar obtención de token
5. Proteger endpoints con JWT

```bash
# Obtener token
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email": "nacho@ieshlanz.es", "password": "mi_password"}'

# Usar token
curl http://localhost:8000/api/temas \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiJ9..."
```

---

### Ejercicio 5.4: Documentación OpenAPI

1. Personalizar la documentación de API Platform
2. Añadir título, descripción y versión
3. Usar grupos de normalización
4. Probar en Swagger UI

---

### Ejercicio 5.5: Proyecto - API del Foro

Crear una API REST completa para el foro:

1. Endpoints para temas: listar, mostrar, crear, editar, borrar
2. Endpoints para respuestas
3. Autenticación JWT
4. Documentación con OpenAPI
5. Validación de datos

---

## 🏗️ Proyecto Práctico: API REST del Foro

**Objetivo:** Crear una API REST completa para el foro.

**Endpoints a crear:**

```
GET    /api/temas              → Listar todos los temas
GET    /api/temas/{id}         → Obtener tema por ID
POST   /api/temas              → Crear tema (requiere JWT)
PUT    /api/temas/{id}         → Actualizar tema (requiere JWT)
DELETE /api/temas/{id}         → Borrar tema (requiere JWT + ROLE_MODERADOR)

GET    /api/respuestas         → Listar respuestas de un tema
POST   /api/respuestas         → Crear respuesta (requiere JWT)

POST   /api/login              → Obtener token JWT
```

**Pasos:**

1. **Instalar API Platform:**

```bash
symfony composer require api
```

2. **Añadir atributos `#[ApiResource]` a las entidades**

3. **Configurar JWT:**

```bash
symfony composer require lexik/jwt-authentication-bundle
symfony console lexik:jwt:generate-keypair
```

4. **Probar en `http://localhost:8000/api/docs`**

5. **Probar con curl o Postman**

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ Principios de la arquitectura REST
- ✅ Construir endpoints JSON con Symfony
- ✅ API Platform para crear APIs rápidamente
- ✅ Autenticación JWT con LexikJWTAuthenticationBundle
- ✅ Documentación con OpenAPI (Swagger)
- ✅ Manejo de errores y validación

**Próxima unidad:** UD06 - Python y Flask: Creando Microservicios Inteligentes

---

## 🔗 Recursos Adicionales

- [API Platform Documentation](https://api-platform.com/docs/)
- [REST API Best Practices](https://restfulapi.net/)
- [JWT.io](https://jwt.io/)
- [OpenAPI Specification](https://spec.openapis.org/)
- [HTTP Status Codes](https://httpstatuses.com/)
