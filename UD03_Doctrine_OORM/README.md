# UD03: Persistencia de Datos con Doctrine ORM 💾

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar qué es un ORM y por qué es útil
- Crear entidades Doctrine
- Implementar operaciones CRUD
- Usar Repositorios para consultas
- Gestionar migraciones de base de datos

---

## 📖 Contenidos

### 3.1 ¿Qué es un ORM?

**ORM** (Object-Relational Mapping) es una técnica que permite trabajar con bases de datos relacionales usando objetos de programación.

**Sin ORM:**

```php
// Usando SQL directo
$sql = "SELECT * FROM usuario WHERE id = ?";
$stmt = $pdo->prepare($sql);
$stmt->execute([1]);
$usuario = $stmt->fetch();
```

**Con ORM (Doctrine):**

```php
// Usando objetos
$usuario = $entityManager->find(Usuario::class, 1);
echo $usuario->getNombre();
```

**Ventajas de Doctrine ORM:**

- Código más legible y mantenible
- Abstracción del motor de base de datos
- Prevención de inyección SQL
- Migraciones automáticas
- Relación entre objetos simplificada

---

### 3.2 Instalación y Configuración

**Instalar Doctrine en Symfony:**

```bash
# Instalar el bundle Doctrine
symfony composer require doctrine

# Instalar la extensión Maker (opcional, para generar código)
symfony composer require make
```

**Configuración de la base de datos:**

```env
# .env
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=8.0"
```

**Verificar la conexión:**

```bash
# Probar la conexión
symfony console doctrine:database:create

# Listar tablas
symfony console doctrine:database:tables-info
```

---

### 3.3 Entidades: Clases PHP que Representan Tablas

Una **entidad** es una clase PHP que representa una tabla de la base de datos.

**Crear una entidad con el Maker:**

```bash
symfony console make:entity
```

**Ejemplo: Entidad Usuario:**

```php
// src/Entity/Usuario.php
<?php

namespace App\Entity;

use App\Repository\UsuarioRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: UsuarioRepository::class)]
class Usuario
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 100)]
    private string $nombre;

    #[ORM\Column(type: 'string', length: 200, unique: true)]
    private string $email;

    #[ORM\Column(type: 'string', length: 50)]
    private string $rol = 'usuario';

    #[ORM\Column(type: 'datetime')]
    private ?\DateTimeInterface $fechaRegistro = null;

    public function __construct()
    {
        $this->fechaRegistro = new \DateTimeImmutable();
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getNombre(): string
    {
        return $this->nombre;
    }

    public function setNombre(string $nombre): self
    {
        $this->nombre = $nombre;
        return $this;
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    public function setEmail(string $email): self
    {
        $this->email = $email;
        return $this;
    }

    public function getRol(): string
    {
        return $this->rol;
    }

    public function setRol(string $rol): self
    {
        $this->rol = $rol;
        return $this;
    }

    public function getFechaRegistro(): ?\DateTimeInterface
    {
        return $this->fechaRegistro;
    }

    public function setFechaRegistro(\DateTimeInterface $fechaRegistro): self
    {
        $this->fechaRegistro = $fechaRegistro;
        return $this;
    }
}
```

**Ejemplo: Entidad Tema:**

```php
// src/Entity/Tema.php
<?php

namespace App\Entity;

use App\Repository\TemaRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: TemaRepository::class)]
class Tema
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 200)]
    private string $titulo;

    #[ORM\Column(type: 'text')]
    private string $contenido;

    #[ORM\ManyToOne(targetEntity: Usuario::class)]
    #[ORM\JoinColumn(nullable: false)]
    private ?Usuario $autor = null;

    #[ORM\Column(type: 'datetime')]
    private ?\DateTimeInterface $fechaCreacion = null;

    public function __construct()
    {
        $this->fechaCreacion = new \DateTimeImmutable();
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getTitulo(): string
    {
        return $this->titulo;
    }

    public function setTitulo(string $titulo): self
    {
        $this->titulo = $titulo;
        return $this;
    }

    public function getContenido(): string
    {
        return $this->contenido;
    }

    public function setContenido(string $contenido): self
    {
        $this->contenido = $contenido;
        return $this;
    }

    public function getAutor(): ?Usuario
    {
        return $this->autor;
    }

    public function setAutor(?Usuario $autor): self
    {
        $this->autor = $autor;
        return $this;
    }

    public function getFechaCreacion(): ?\DateTimeInterface
    {
        return $this->fechaCreacion;
    }

    public function setFechaCreacion(\DateTimeInterface $fechaCreacion): self
    {
        $this->fechaCreacion = $fechaCreacion;
        return $this;
    }
}
```

**Ejemplo: Entidad Respuesta:**

```php
// src/Entity/Respuesta.php
<?php

namespace App\Entity;

use App\Repository\RespuestaRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: RespuestaRepository::class)]
class Respuesta
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'text')]
    private string $contenido;

    #[ORM\ManyToOne(targetEntity: Tema::class)]
    #[ORM\JoinColumn(nullable: false)]
    private ?Tema $tema = null;

    #[ORM\ManyToOne(targetEntity: Usuario::class)]
    #[ORM\JoinColumn(nullable: false)]
    private ?Usuario $autor = null;

    #[ORM\Column(type: 'datetime')]
    private ?\DateTimeInterface $fechaCreacion = null;

    public function __construct()
    {
        $this->fechaCreacion = new \DateTimeImmutable();
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getContenido(): string
    {
        return $this->contenido;
    }

    public function setContenido(string $contenido): self
    {
        $this->contenido = $contenido;
        return $this;
    }

    public function getTema(): ?Tema
    {
        return $this->tema;
    }

    public function setTema(?Tema $tema): self
    {
        $this->tema = $tema;
        return $this;
    }

    public function getAutor(): ?Usuario
    {
        return $this->autor;
    }

    public function setAutor(?Usuario $autor): self
    {
        $this->autor = $autor;
        return $this;
    }

    public function getFechaCreacion(): ?\DateTimeInterface
    {
        return $this->fechaCreacion;
    }
}
```

---

### 3.4 Relaciones entre Entidades

**Tipos de relaciones en Doctrine:**

| Relación | Descripción | Ejemplo |
| --- | --- | --- |
| `@ManyToOne` | Muchos a uno | Muchos temas, un autor |
| `@OneToMany` | Uno a muchos | Un autor, muchos temas |
| `@OneToOne` | Uno a uno | Usuario y perfil |
| `@ManyToMany` | Muchos a muchos | Estudiantes y cursos |

**Ejemplo: Relación Uno a Muchos:**

```php
// src/Entity/Usuario.php
#[ORM\OneToMany(mappedBy: 'autor', targetEntity: Tema::class)]
private Collection $temas;

public function __construct()
{
    $this->temas = new ArrayCollection();
}

public function getTemas(): Collection
{
    return $this->temas;
}

public function addTema(Tema $tema): self
{
    if (!$this->temas->contains($tema)) {
        $this->temas[] = $tema;
        $tema->setAutor($this);
    }
    return $this;
}
```

```php
// src/Entity/Tema.php
#[ORM\ManyToOne(targetEntity: Usuario::class, inversedBy: 'temas')]
#[ORM\JoinColumn(nullable: false)]
private ?Usuario $autor = null;
```

---

### 3.5 Operaciones CRUD

**CRUD** = Create, Read, Update, Delete

**Crear (Create):**

```php
// src/Controller/TemaController.php
#[Route('/tema/nuevo', name: 'tema_nuevo')]
public function nuevo(Request $request, EntityManagerInterface $entityManager): Response
{
    $tema = new Tema();
    $tema->setTitulo($request->request->get('titulo'));
    $tema->setContenido($request->request->get('contenido'));

    // Buscar el autor (en producción, usar sesión)
    $autor = $entityManager->getRepository(Usuario::class)->find(1);
    $tema->setAutor($autor);

    $entityManager->persist($tema);
    $entityManager->flush();

    return $this->redirectToRoute('temas');
}
```

**Leer (Read):**

```php
// Obtener un tema por ID
$tema = $entityManager->find(Tema::class, 1);

// Obtener todos los temas
$temas = $entityManager->getRepository(Tema::class)->findAll();

// Obtener temas con condiciones
$temas = $entityManager->getRepository(Tema::class)->findBy(
    ['autor' => 1],  // WHERE autor = 1
    ['fechaCreacion' => 'DESC']  // ORDER BY fechaCreacion DESC
);

// Consulta personalizada
$temas = $entityManager->createQuery(
    'SELECT t FROM App\Entity\Tema t WHERE t.titulo LIKE :busqueda'
)->setParameter('busqueda', '%PHP%')
->getResult();
```

**Actualizar (Update):**

```php
#[Route('/tema/{id}/editar', name: 'tema_editar')]
public function editar(int $id, Request $request, EntityManagerInterface $entityManager): Response
{
    $tema = $entityManager->find(Tema::class, $id);

    if ($tema) {
        $tema->setTitulo($request->request->get('titulo'));
        $tema->setContenido($request->request->get('contenido'));

        $entityManager->flush();
    }

    return $this->redirectToRoute('temas');
}
```

**Eliminar (Delete):**

```php
#[Route('/tema/{id}/borrar', name: 'tema_borrar')]
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

### 3.6 Repositorios: Consultas Avanzadas

El **Repositorio** es una clase que facilita las consultas a la base de datos.

**Crear un repositorio con el Maker:**

```bash
symfony console make:repository
```

**Repositorio personalizado:**

```php
// src/Repository/TemaRepository.php
<?php

namespace App\Repository;

use App\Entity\Tema;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

class TemaRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Tema::class);
    }

    // Obtener temas ordenados por fecha
    public function findByFechaDesc(): array
    {
        return $this->createQueryBuilder('t')
            ->orderBy('t.fechaCreacion', 'DESC')
            ->getQuery()
            ->getResult();
    }

    // Buscar temas por título
    public function buscarPorTitulo(string $termino): array
    {
        return $this->createQueryBuilder('t')
            ->where('t.titulo LIKE :termino')
            ->setParameter('termino', '%' . $termino . '%')
            ->getQuery()
            ->getResult();
    }

    // Obtener temas de un autor
    public function findByAutor(int $autorId): array
    {
        return $this->createQueryBuilder('t')
            ->join('t.autor', 'u')
            ->where('u.id = :autorId')
            ->setParameter('autorId', $autorId)
            ->getQuery()
            ->getResult();
    }

    // Contar temas
    public function countTemas(): int
    {
        return $this->createQueryBuilder('t')
            ->select('COUNT(t.id)')
            ->getQuery()
            ->getSingleScalarResult();
    }
}
```

**Usar el repositorio en un controlador:**

```php
// src/Controller/TemaController.php
#[Route('/temas', name: 'temas')]
public function listar(TemaRepository $temaRepository): Response
{
    $temas = $temaRepository->findByFechaDesc();

    return $this->render('tema/lista.html.twig', [
        'temas' => $temas,
    ]);
}

#[Route('/buscar', name: 'buscar')]
public function buscar(Request $request, TemaRepository $temaRepository): Response
{
    $termino = $request->query->get('q', '');
    $temas = $temaRepository->buscarPorTitulo($termino);

    return $this->render('tema/buscar.html.twig', [
        'temas' => $temas,
        'termino' => $termino,
    ]);
}
```

---

### 3.7 Migraciones: Versionado de la Base de Datos

Las **migraciones** permiten actualizar la base de datos de forma controlada.

**Generar migración:**

```bash
# Generar la migración basada en las entidades
symfony console make:migration

# Ejecutar la migración
symfony console doctrine:migrations:migrate
```

**Comandos útiles:**

```bash
# Ver el estado de las migraciones
symfony console doctrine:migrations:status

# Generar migración
symfony console make:migration

# Ejecutar migraciones pendientes
symfony console doctrine:migrations:migrate

# Ejecutar una migración específica
symfony console doctrine:migrations:migrate 20240115120000

# Volcar la base de datos (DROP y CREATE)
symfony console doctrine:database:drop --force
symfony console doctrine:database:create
symfony console doctrine:migrations:migrate
```

**Ejemplo de archivo de migración:**

```php
// migrations/Version20240115120000.php
<?php

final class Version20240115120000 extends Migration
{
    public function up(Schema $schema): void
    {
        $this->addSql('CREATE TABLE usuario (id INT AUTO_INCREMENT NOT NULL, nombre VARCHAR(100) NOT NULL, email VARCHAR(200) NOT NULL, rol VARCHAR(50) NOT NULL, fecha_registro DATETIME NOT NULL, UNIQUE INDEX UNIQ_2265B01E E70 (email), PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8mb4');
        $this->addSql('CREATE TABLE tema (id INT AUTO_INCREMENT NOT NULL, autor_id INT NOT NULL, titulo VARCHAR(200) NOT NULL, contenido LONGTEXT NOT NULL, fecha_creacion DATETIME NOT NULL, INDEX IDX_4F0A6538 F69 (autor_id), PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8mb4');
        $this->addSql('CREATE TABLE respuesta (id INT AUTO_INCREMENT NOT NULL, tema_id INT NOT NULL, autor_id INT NOT NULL, contenido LONGTEXT NOT NULL, fecha_creacion DATETIME NOT NULL, INDEX IDX_4F0A6538 F69 (tema_id), INDEX IDX_4F0A6538 F69 (autor_id), PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8mb4');
    }

    public function down(Schema $schema): void
    {
        $this->addSql('DROP TABLE usuario');
        $this->addSql('DROP TABLE tema');
        $this->addSql('DROP TABLE respuesta');
    }
}
```

---

### 3.8 DQL: Doctrine Query Language

**DQL** es el lenguaje de consultas de Doctrine (similar a SQL pero orientado a objetos).

**Ejemplos de DQL:**

```php
// Consulta simple
$dql = 'SELECT u FROM App\Entity\Usuario u WHERE u.rol = :rol';
$query = $entityManager->createQuery($dql);
$query->setParameter('rol', 'admin');
$usuarios = $query->getResult();

// Consulta con JOIN
$dql = 'SELECT t, u FROM App\Entity\Tema t JOIN t.autor u WHERE u.rol = :rol';
$query = $entityManager->createQuery($dql);
$query->setParameter('rol', 'moderador');
$temas = $query->getResult();

// Consulta con agregación
$dql = 'SELECT u, COUNT(t) as numTemas FROM App\Entity\Usuario u LEFT JOIN App\Entity\Tema t WITH t.autor = u GROUP BY u';
$query = $entityManager->createQuery($dql);
$resultados = $query->getResult();
```

---

## 💻 Ejercicios

### Ejercicio 3.1: Crear Entidades

Crea las entidades `Producto` y `Categoria` con la siguiente estructura:

```
Producto:
- id: integer (auto)
- nombre: string (100)
- precio: float
- descripcion: text
- categoria: ManyToOne (Categoria)

Categoria:
- id: integer (auto)
- nombre: string (50)
- productos: OneToMany (Producto)
```

---

### Ejercicio 3.2: Operaciones CRUD

Crea un controlador que permita:

1. Listar todos los productos
2. Crear un nuevo producto
3. Editar un producto
4. Eliminar un producto

```php
// src/Controller/ProductoController.php
<?php

namespace App\Controller;

use App\Entity\Producto;
use App\Repository\ProductoRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ProductoController extends AbstractController
{
    #[Route('/productos', name: 'productos')]
    public function listar(ProductoRepository $productoRepository): Response
    {
        return $this->render('producto/lista.html.twig', [
            'productos' => $productoRepository->findAll(),
        ]);
    }

    #[Route('/producto/nuevo', name: 'producto_nuevo')]
    public function nuevo(Request $request, EntityManagerInterface $entityManager): Response
    {
        $producto = new Producto();
        $producto->setNombre($request->request->get('nombre'));
        $producto->setPrecio((float) $request->request->get('precio'));
        $producto->setDescripcion($request->request->get('descripcion'));

        $entityManager->persist($producto);
        $entityManager->flush();

        return $this->redirectToRoute('productos');
    }

    #[Route('/producto/{id}/editar', name: 'producto_editar')]
    public function editar(int $id, Request $request, EntityManagerInterface $entityManager): Response
    {
        $producto = $entityManager->find(Producto::class, $id);

        if ($producto) {
            $producto->setNombre($request->request->get('nombre'));
            $producto->setPrecio((float) $request->request->get('precio'));
            $producto->setDescripcion($request->request->get('descripcion'));

            $entityManager->flush();
        }

        return $this->redirectToRoute('productos');
    }

    #[Route('/producto/{id}/borrar', name: 'producto_borrar')]
    public function borrar(int $id, EntityManagerInterface $entityManager): Response
    {
        $producto = $entityManager->find(Producto::class, $id);

        if ($producto) {
            $entityManager->remove($producto);
            $entityManager->flush();
        }

        return $this->redirectToRoute('productos');
    }
}
```

---

### Ejercicio 3.3: Consultas con Repositorio

Añade métodos al repositorio `ProductoRepository`:

```php
// src/Repository/ProductoRepository.php
public function buscarPorNombre(string $termino): array
{
    return $this->createQueryBuilder('p')
        ->where('p.nombre LIKE :termino')
        ->setParameter('termino', '%' . $termino . '%')
        ->getQuery()
        ->getResult();
}

public function obtenerPorRangoPrecios(float $min, float $max): array
{
    return $this->createQueryBuilder('p')
        ->where('p.precio BETWEEN :min AND :max')
        ->setParameter('min', $min)
        ->setParameter('max', $max)
        ->getQuery()
        ->getResult();
}

public function contarProductos(): int
{
    return $this->createQueryBuilder('p')
        ->select('COUNT(p.id)')
        ->getQuery()
        ->getSingleScalarResult();
}
```

---

### Ejercicio 3.4: Migraciones

1. Crea las entidades `Usuario` y `Tema`
2. Genera la migración: `symfony console make:migration`
3. Ejecuta la migración: `symfony console doctrine:migrations:migrate`
4. Verifica la base de datos

---

### Ejercicio 3.5: Proyecto - Foro con Base de Datos

Amplía el proyecto del foro para usar Doctrine:

1. Crear entidades `Usuario`, `Tema` y `Respuesta`
2. Generar y ejecutar migraciones
3. Crear controladores para CRUD de temas
4. Mostrar datos reales desde la base de datos

---

## 🏗️ Proyecto Práctico: Foro con Doctrine ORM

**Objetivo:** Implementar la persistencia de datos del foro usando Doctrine ORM.

**Pasos:**

1. **Crear entidades:**

```bash
symfony console make:entity
# Crear Usuario, Tema, Respuesta
```

2. **Generar migraciones:**

```bash
symfony console make:migration
symfony console doctrine:migrations:migrate
```

3. **Crear controladores CRUD:**

```php
// src/Controller/TemaController.php
#[Route('/temas', name: 'temas')]
public function listar(TemaRepository $temaRepository): Response
{
    return $this->render('tema/lista.html.twig', [
        'temas' => $temaRepository->findBy([], ['fechaCreacion' => 'DESC']),
    ]);
}
```

4. **Actualizar plantillas para mostrar datos reales**

5. **Probar en `http://localhost:8000`**

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ Qué es un ORM y por qué es útil
- ✅ Crear entidades Doctrine con atributos
- ✅ Relaciones entre entidades (ManyToOne, OneToMany)
- ✅ Operaciones CRUD con EntityManager
- ✅ Consultas avanzadas con Repositorios
- ✅ Migraciones para versionar la base de datos
- ✅ DQL para consultas personalizadas

**Próxima unidad:** UD04 - Seguridad y Sesiones en Aplicaciones Profesionales

---

## 🔗 Recursos Adicionales

- [Documentación oficial de Doctrine ORM](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/)
- [Symfony Docs: Doctrine](https://symfony.com/doc/current/doctrine.html)
- [Doctrine DQL Reference](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html)
- [Doctrine Migrations](https://www.doctrine-project.org/projects/doctrine-migrations/en/latest/)
