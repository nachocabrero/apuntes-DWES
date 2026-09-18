# UD04: Seguridad y Sesiones en Aplicaciones Profesionales 🔐

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar la diferencia entre autenticación y autorización
- Implementar un sistema de login y registro con Symfony
- Gestionar sesiones y cookies
- Usar roles y permisos para proteger rutas
- Hashing seguro de contraseñas

---

## 📖 Contenidos

### 4.1 Autenticación vs. Autorización

**Autenticación:** Verificar quién es el usuario (¿eres tú?)

```
Usuario → Credenciales → ¿Eres válido? → SÍ/NO
```

**Autorización:** Verificar qué puede hacer el usuario (¿tienes permiso?)

```
Usuario autenticado → ¿Tiene rol/admin? → SÍ/NO → Acceso concedido/negado
```

**Analogía:**

- **Autenticación:** Mostrar el DNI en la puerta del club
- **Autorización:** Tener la pulsera VIP para entrar a la zona VIP

---

### 4.2 Sesiones HTTP

HTTP es un protocolo **sin estado**: cada petición es independiente.

**Problema:** ¿Cómo recordar que un usuario ha iniciado sesión?

**Solución: Sesiones**

```
┌──────────┐         ┌──────────────┐
│  Cliente  │         │   Servidor   │
│           │         │              │
│ 1. Login  │────────▶│ Verificar    │
│           │◀────────│ Sesión creada│
│ 2. Cookie │◀────────│ SessionID    │
│           │         │              │
│ 3. Siguiente│──────▶│ Leer Session │
│   petición│         │ Datos usuario│
└──────────┘         └──────────────┘
```

**Cookies de sesión:**

```
Set-Cookie: PHPSESSID=abc123; path=/; HttpOnly; Secure
```

- `HttpOnly`: No accesible desde JavaScript (seguridad)
- `Secure`: Solo se envía por HTTPS

---

### 4.3 Configuración de Seguridad en Symfony

**Instalar el componente de seguridad:**

```bash
symfony composer require security
```

**Configuración básica:**

```yaml
# config/packages/security.yaml
security:
    # Hashers: cómo se almacenan las contraseñas
    password_hashers:
        App\Entity\Usuario:
            algorithm: auto

    # Providers: de dónde se obtienen los usuarios
    providers:
        app_user_provider:
            entity:
                class: App\Entity\Usuario
                property: email

    # Firewalls: qué rutas están protegidas
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            lazy: true
            provider: app_user_provider
            form_login:
                login_path: login
                check_path: login
                enable_csrf: true
            logout:
                path: logout
                target: inicio

    # Access Control: qué rutas requieren qué rol
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
        - { path: ^/moderar, roles: ROLE_MODERADOR }
        - { path: ^/login, roles: PUBLIC_ACCESS }
```

---

### 4.4 Formulario de Login

**Controlador de Login:**

```php
// src/Controller/SecurityController.php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Security\Http\Authentication\AuthenticationUtils;

class SecurityController extends AbstractController
{
    #[Route('/login', name: 'login')]
    public function login(AuthenticationUtils $authenticationUtils): Response
    {
        // Obtener error de autenticación si lo hay
        $error = $authenticationUtils->getLastAuthenticationError();

        // Último nombre de usuario introducido
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('security/login.html.twig', [
            'last_username' => $lastUsername,
            'error' => $error,
        ]);
    }

    #[Route('/logout', name: 'logout')]
    public function logout(): void
    {
        // Esta ruta puede ser cualquier cosa, Symfony la intercepta
        throw new \Exception('Debes eliminar este controlador');
    }
}
```

**Plantilla de Login:**

```twig
{# templates/security/login.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Iniciar Sesión{% endblock %}

{% block body %}
    <div class="login-container">
        <h1>Iniciar Sesión</h1>

        {% if error %}
            <div class="alert alert-danger">
                {{ error.messageKey|trans(error.messageData, 'security') }}
            </div>
        {% endif %}

        {% if app.user %}
            <div class="alert alert-info">
                Bienvenido, {{ app.user.nombre }}.
                <a href="{{ path('logout') }}">Cerrar sesión</a>
            </div>
        {% endif %}

        <form action="{{ path('login') }}" method="POST">
            <label for="username">Email:</label>
            <input type="email" id="username" name="_username"
                   value="{{ last_username }}" required>

            <label for="password">Contraseña:</label>
            <input type="password" id="password" name="_password" required>

            <input type="hidden" name="_csrf_token"
                   value="{{ csrf_token('authenticate') }}">

            <button type="submit" class="btn">Entrar</button>
        </form>
    </div>
{% endblock %}
```

---

### 4.5 Formulario de Registro

**Entidad Usuario con contraseña:**

```php
// src/Entity/Usuario.php
#[ORM\Column(type: 'string')]
private string $password;

public function setPassword(string $password): self
{
    $this->password = $password;
    return $this;
}
```

**Formulario de Registro:**

```php
// src/Controller/RegistroController.php
<?php

namespace App\Controller;

use App\Entity\Usuario;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Security\Http\Authentication\UserAuthenticatorInterface;
use Symfony\Component\Security\Http\Authenticator\FormLoginAuthenticator;

class RegistroController extends AbstractController
{
    #[Route('/registro', name: 'registro')]
    public function registro(
        Request $request,
        UserPasswordHasherInterface $passwordHasher,
        EntityManagerInterface $entityManager,
        UserAuthenticatorInterface $authenticator,
        FormLoginAuthenticator $authenticator
    ): Response {
        // Si ya está logueado, redirigir
        if ($this->getUser()) {
            return $this->redirectToRoute('inicio');
        }

        $usuario = new Usuario();
        $usuario->setEmail($request->request->get('email'));
        $usuario->setNombre($request->request->get('nombre'));
        $usuario->setRol('usuario');

        // Hash de la contraseña
        $password = $request->request->get('password');
        $hashedPassword = $passwordHasher->hashPassword($usuario, $password);
        $usuario->setPassword($hashedPassword);

        $entityManager->persist($usuario);
        $entityManager->flush();

        // Auto-login después del registro
        return $authenticator->authenticateUser($usuario, $authenticator, $request);
    }
}
```

**Plantilla de Registro:**

```twig
{# templates/registro/index.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}Registrarse{% endblock %}

{% block body %}
    <div class="registro-container">
        <h1>Crear Cuenta</h1>

        <form action="{{ path('registro') }}" method="POST">
            <label for="nombre">Nombre:</label>
            <input type="text" id="nombre" name="nombre" required>

            <label for="email">Email:</label>
            <input type="email" id="email" name="email" required>

            <label for="password">Contraseña:</label>
            <input type="password" id="password" name="password" required>

            <label for="password_confirm">Confirmar Contraseña:</label>
            <input type="password" id="password_confirm" name="password_confirm" required>

            <input type="hidden" name="_csrf_token" value="{{ csrf_token('registro') }}">

            <button type="submit" class="btn">Registrarse</button>
        </form>

        <p>¿Ya tienes cuenta? <a href="{{ path('login') }}">Iniciar sesión</a></p>
    </div>
{% endblock %}
```

---

### 4.6 Roles y Permisos

**Roles en Symfony:**

```
ROLE_USER → Usuario básico
ROLE_MODERADOR → Puede moderar contenido
ROLE_ADMIN → Administrador total
ROLE_SUPER_ADMIN → Super administrador
```

**Proteger rutas con roles:**

```php
// src/Controller/AdminController.php
#[Route('/admin', name: 'admin')]
#[IsGranted('ROLE_ADMIN')]
public function index(): Response
{
    return $this->render('admin/index.html.twig');
}

#[Route('/moderar', name: 'moderar')]
#[IsGranted('ROLE_MODERADOR')]
public function moderar(): Response
{
    return $this->render('moderar/index.html.twig');
}
```

**Proteger con access_control en security.yaml:**

```yaml
access_control:
    - { path: ^/admin, roles: ROLE_ADMIN }
    - { path: ^/moderar, roles: ROLE_MODERADOR }
    - { path: ^/login, roles: PUBLIC_ACCESS }
    - { path: ^/registro, roles: PUBLIC_ACCESS }
```

**Comprobar roles en Twig:**

```twig
{# templates/base.html.twig #}
<nav>
    <a href="{{ path('inicio') }}">Inicio</a>

    {% if is_granted('ROLE_MODERADOR') %}
        <a href="{{ path('moderar') }}">Moderar</a>
    {% endif %}

    {% if is_granted('ROLE_ADMIN') %}
        <a href="{{ path('admin') }}">Admin</a>
    {% endif %}

    {% if app.user %}
        <a href="{{ path('logout') }}">Salir</a>
    {% else %}
        <a href="{{ path('login') }}">Entrar</a>
    {% endif %}
</nav>
```

**Comprobar roles en controlador:**

```php
use Symfony\Component\Security\Http\Attribute\IsGranted;

#[Route('/borrar/{id}', name: 'tema_borrar')]
#[IsGranted('ROLE_MODERADOR')]
public function borrar(int $id, EntityManagerInterface $entityManager): Response
{
    $tema = $entityManager->find(Tema::class, $id);

    if ($tema) {
        $entityManager->remove($tema);
        $entityManager->flush();
    }

    return $this->redirectToRoute('temas');
}
```

---

### 4.7 Hashing de Contraseñas

**¿Por qué hash?**

```
❌ Almacenar contraseñas en texto plano:
   "mi_contraseña" → Se ve en la base de datos

✅ Almacenar contraseñas con hash:
   "mi_contraseña" → "$2y$10$abc123..." → No se puede revertir
```

**Symfony usa bcrypt por defecto:**

```php
// src/Controller/RegistroController.php
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

public function registro(UserPasswordHasherInterface $passwordHasher): Response
{
    $usuario = new Usuario();
    $password = $request->request->get('password');

    // Hash de la contraseña
    $hashedPassword = $passwordHasher->hashPassword($usuario, $password);
    $usuario->setPassword($hashedPassword);

    $entityManager->persist($usuario);
    $entityManager->flush();
}
```

**Verificar contraseña en login:**

```php
// Symfony lo hace automáticamente con FormLoginAuthenticator
// No necesitas verificar manualmente
```

---

### 4.8 Autenticador Personalizado (Opcional)

**Crear un autenticador personalizado:**

```php
// src/Security/FormLoginAuthenticator.php
<?php

namespace App\Security;

use Symfony\Component\HttpFoundation\RedirectResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Http\Authenticator\AbstractLoginFormAuthenticator;
use Symfony\Component\Security\Http\Authenticator\Passport\Badge\CsrfTokenBadge;
use Symfony\Component\Security\Http\Authenticator\Passport\Badge\RememberMeBadge;
use Symfony\Component\Security\Http\Authenticator\Passport\Badge\UserBadge;
use Symfony\Component\Security\Http\Authenticator\Passport\Credentials\PasswordCredentials;
use Symfony\Component\Security\Http\Authenticator\Passport\Passport;
use Symfony\Component\Security\Http\Util\TargetPathTrait;

class FormLoginAuthenticator extends AbstractLoginFormAuthenticator
{
    use TargetPathTrait;

    public const LOGIN_ROUTE = 'login';

    public function __construct(private UrlGeneratorInterface $urlGenerator) {}

    public function authenticate(Request $request): Passport
    {
        $username = $request->request->get('_username', '');

        $request->getSession()->set(Security::LAST_USERNAME, $username);

        return new Passport(
            new UserBadge($username),
            new PasswordCredentials($request->request->get('_password', '')),
            [
                new CsrfTokenBadge('authenticate', $request->request->get('_csrf_token')),
                new RememberMeBadge(),
            ]
        );
    }

    public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
    {
        if ($targetPath = $this->getTargetPath($request->getSession())) {
            return new RedirectResponse($targetPath);
        }

        return new RedirectResponse($this->urlGenerator->generate('inicio'));
    }

    protected function getLoginUrl(Request $request): string
    {
        return $this->urlGenerator->generate(self::LOGIN_ROUTE);
    }
}
```

---

### 4.9 Protección CSRF

**CSRF** (Cross-Site Request Forgery) es un ataque que fuerza al usuario a realizar acciones no deseadas.

**Protección en formularios:**

```twig
<input type="hidden" name="_csrf_token" value="{{ csrf_token('action_name') }}">
```

**Protección en controladores:**

```php
#[Route('/borrar/{id}', name: 'tema_borrar')]
#[IsGranted('ROLE_MODERADOR')]
#[IsGranted('csrf_token')]
public function borrar(int $id, EntityManagerInterface $entityManager): Response
{
    // ...
}
```

---

## 💻 Ejercicios

### Ejercicio 4.1: Formulario de Login

Crea un formulario de login con:

1. Campo de email
2. Campo de contraseña
3. Token CSRF
4. Mensaje de error si las credenciales son incorrectas

---

### Ejercicio 4.2: Formulario de Registro

Crea un formulario de registro con:

1. Campo de nombre
2. Campo de email
3. Campo de contraseña (con confirmación)
4. Hash de la contraseña con bcrypt
5. Auto-login después del registro

---

### Ejercicio 4.3: Roles y Permisos

Implementa un sistema de roles:

1. Crear roles: `ROLE_USER`, `ROLE_MODERADOR`, `ROLE_ADMIN`
2. Proteger rutas con `#[IsGranted]`
3. Mostrar/ocultar enlaces según el rol del usuario
4. Crear una página de admin solo para administradores

```php
// src/Controller/AdminController.php
#[Route('/admin', name: 'admin')]
#[IsGranted('ROLE_ADMIN')]
public function index(): Response
{
    return $this->render('admin/index.html.twig', [
        'usuarios' => $this->entityManager->getRepository(Usuario::class)->findAll(),
    ]);
}
```

---

### Ejercicio 4.4: Proteger el Foro

Amplía el proyecto del foro:

1. Solo usuarios logueados pueden crear temas
2. Solo el autor y moderadores pueden editar/borrar temas
3. Página de perfil del usuario

```php
// src/Controller/TemaController.php
#[Route('/tema/nuevo', name: 'tema_nuevo')]
#[IsGranted('IS_AUTHENTICATED_FULLY')]
public function nuevo(Request $request, EntityManagerInterface $entityManager): Response
{
    $tema = new Tema();
    $tema->setTitulo($request->request->get('titulo'));
    $tema->setContenido($request->request->get('contenido'));
    $tema->setAutor($this->getUser());

    $entityManager->persist($tema);
    $entityManager->flush();

    return $this->redirectToRoute('temas');
}

#[Route('/tema/{id}/editar', name: 'tema_editar')]
#[IsGranted('ROLE_MODERADOR') or IsGranted('OWNER', subject='tema.autor')]
public function editar(int $id, Request $request, EntityManagerInterface $entityManager): Response
{
    // ...
}
```

---

### Ejercicio 4.5: Proyecto - Sistema de Usuarios Completo

Implementa un sistema de usuarios completo:

1. Registro con validación
2. Login con remember me
3. Logout
4. Perfil de usuario
5. Cambio de contraseña
6. Roles: usuario, moderador, administrador

---

## 🏗️ Proyecto Práctico: Foro con Autenticación

**Objetivo:** Añadir un sistema de autenticación y autorización al foro.

**Pasos:**

1. **Configurar security.yaml:**

```yaml
security:
    password_hashers:
        App\Entity\Usuario:
            algorithm: auto

    providers:
        app_user_provider:
            entity:
                class: App\Entity\Usuario
                property: email

    firewalls:
        main:
            lazy: true
            provider: app_user_provider
            form_login:
                login_path: login
                check_path: login
                enable_csrf: true
            logout:
                path: logout
                target: inicio
```

2. **Crear controladores de seguridad:**

```php
// src/Controller/SecurityController.php
#[Route('/login', name: 'login')]
public function login(AuthenticationUtils $utils): Response { ... }

#[Route('/registro', name: 'registro')]
public function registro(Request $request, UserPasswordHasherInterface $hasher): Response { ... }

#[Route('/logout', name: 'logout')]
public function logout(): void { ... }
```

3. **Crear plantillas de login y registro**

4. **Proteger rutas con `#[IsGranted]`**

5. **Mostrar enlaces según rol**

6. **Probar en `http://localhost:8000`**

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ Diferencia entre autenticación y autorización
- ✅ Sesiones HTTP y cookies
- ✅ Configuración de seguridad en Symfony
- ✅ Formulario de login con FormLoginAuthenticator
- ✅ Formulario de registro con hash de contraseñas
- ✅ Roles y permisos con `#[IsGranted]`
- ✅ Hashing seguro con bcrypt
- ✅ Protección CSRF

**Próxima unidad:** UD05 - Creación de Servicios: APIs REST con Symfony

---

## 🔗 Recursos Adicionales

- [Symfony Docs: Security](https://symfony.com/doc/current/security.html)
- [Symfony Docs: Form Login](https://symfony.com/doc/current/security/form_login.html)
- [Symfony Docs: Password Hashing](https://symfony.com/doc/current/security/password_hasher.html)
- [OWASP: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP: CSRF Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
