# UD06: Python y Flask: Creando Microservicios Inteligentes 🐍

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar los fundamentos de Python
- Comparar Python con PHP
- Instalar y configurar Flask
- Crear rutas y manejar peticiones JSON
- Construir micro-servicios independientes

---

## 📖 Contenidos

### 6.1 Fundamentos de Python

**Python** es un lenguaje de programación versátil, ideal para IA, ciencia de datos y desarrollo web.

**Características:**

- Sintaxis clara y legible
- Tipado dinámico
- Multiplataforma
- Gran ecosistema de librerías
- Ideal para IA y machine learning

**Comparación PHP vs Python:**

| Aspecto | PHP | Python |
| --- | --- | --- |
| Tipado | Estático (8.x) | Dinámico |
| Framework web | Symfony, Laravel | Flask, Django |
| Uso principal | Web backend | IA, Data, Web, Scripts |
| Sintaxis | `{ }` y `;` | Indentación |
| Tipado de variables | `string $nombre` | `nombre = "texto"` |

---

### 6.2 Sintaxis Básica de Python

**Variables y tipos:**

```python
# Variables (no se declaran con tipo)
nombre = "Nacho"
edad = 46
altura = 1.84
es_profesor = True

# Tipos de datos
entero = 42
decimal = 3.14
texto = "Hola Mundo"
booleano = True
lista = [1, 2, 3, 4, 5]
diccionario = {"nombre": "Nacho", "edad": 46}
tupla = (1, 2, 3)
conjunto = {1, 2, 3, 4, 5}
nulo = None
```

**Estructuras de control:**

```python
# Condicional
edad = 18

if edad < 18:
    print("Menor de edad")
elif edad < 65:
    print("Adulto")
else:
    print("Jubilado")

# Bucle for
for i in range(5):
    print(f"Número: {i}")

# Bucle for con lista
frutas = ["manzana", "pera", "plátano"]
for fruta in frutas:
    print(f"Me gusta la {fruta}")

# Bucle while
contador = 0
while contador < 5:
    print(f"Contador: {contador}")
    contador += 1

# Diccionario
usuario = {
    "nombre": "Nacho",
    "email": "nacho@ieshlanz.es",
    "rol": "profesor"
}

for clave, valor in usuario.items():
    print(f"{clave}: {valor}")
```

**Funciones:**

```python
# Función básica
def saludar(nombre):
    return f"¡Hola, {nombre}!"

# Función con tipo (opcional)
def sumar(a: int, b: int) -> int:
    return a + b

# Función con valor por defecto
def crear_usuario(nombre, email, rol="usuario"):
    return {
        "nombre": nombre,
        "email": email,
        "rol": rol
    }

# Llamadas
print(saludar("Nacho"))
print(sumar(2, 3))
print(crear_usuario("Celia", "celia@ieshlanz.es"))
print(crear_usuario("Pedro", "pedro@ieshlanz.es", "moderador"))
```

**Clases (POO):**

```python
class Usuario:
    def __init__(self, nombre, email, rol="usuario"):
        self.nombre = nombre
        self.email = email
        self.rol = rol

    def get_info(self):
        return f"Usuario: {self.nombre}, Email: {self.email}, Rol: {self.rol}"

    def es_admin(self):
        return self.rol == "admin"

# Crear instancia
usuario = Usuario("Nacho", "nacho@ieshlanz.es", "profesor")
print(usuario.get_info())
print(usuario.es_admin())  # False
```

---

### 6.3 Instalación de Python y Flask

**Instalar Python:**

```bash
# Verificar instalación
python3 --version

# Instalar Python (Ubuntu/Debian)
sudo apt update
sudo apt install python3 python3-pip python3-venv

# Crear entorno virtual
mkdir mi-proyecto-python
cd mi-proyecto-python
python3 -m venv venv
source venv/bin/activate

# Instalar Flask
pip install flask

# Instalar dependencias adicionales
pip install requests python-dotenv
```

**Estructura del proyecto:**

```
mi-proyecto-python/
├── venv/
├── app.py
├── requirements.txt
└── README.md
```

**requirements.txt:**

```
flask==3.0.0
requests==2.31.0
python-dotenv==1.0.0
```

---

### 6.4 Primeros Pasos con Flask

**Aplicación básica:**

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>¡Hola, Mundo!</h1>'

if __name__ == '__main__':
    app.run(debug=True)
```

**Ejecutar:**

```bash
python app.py
# O:
flask run
```

**Acceder:** `http://localhost:5000`

---

### 6.5 Rutas y Parámetros

**Rutas básicas:**

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Inicio</h1>'

@app.route('/hola/<nombre>')
def hola(nombre):
    return f'<h1>¡Hola, {nombre}!</h1>'

@app.route('/suma/<int:a>/<int:b>')
def suma(a, b):
    return f'<h1>La suma es: {a + b}</h1>'

if __name__ == '__main__':
    app.run(debug=True)
```

**Métodos HTTP:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/datos', methods=['GET'])
def obtener_datos():
    return jsonify({"mensaje": "GET request"})

@app.route('/datos', methods=['POST'])
def crear_datos():
    data = request.get_json()
    return jsonify({"mensaje": "POST request", "datos": data}), 201

@app.route('/datos/<int:id>', methods=['PUT'])
def actualizar_datos(id):
    data = request.get_json()
    return jsonify({"mensaje": "PUT request", "id": id, "datos": data})

@app.route('/datos/<int:id>', methods=['DELETE'])
def eliminar_datos(id):
    return jsonify({"mensaje": "DELETE request", "id": id})

if __name__ == '__main__':
    app.run(debug=True)
```

---

### 6.6 Manejo de JSON

**Devolver JSON:**

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/usuarios')
def listar_usuarios():
    usuarios = [
        {"id": 1, "nombre": "Nacho", "email": "nacho@ieshlanz.es"},
        {"id": 2, "nombre": "Celia", "email": "celia@ieshlanz.es"},
        {"id": 3, "nombre": "Pedro", "email": "pedro@ieshlanz.es"}
    ]
    return jsonify(usuarios)

@app.route('/api/usuario/<int:id>')
def obtener_usuario(id):
    usuarios = {
        1: {"id": 1, "nombre": "Nacho", "email": "nacho@ieshlanz.es"},
        2: {"id": 2, "nombre": "Celia", "email": "celia@ieshlanz.es"},
        3: {"id": 3, "nombre": "Pedro", "email": "pedro@ieshlanz.es"}
    }

    if id not in usuarios:
        return jsonify({"error": "Usuario no encontrado"}), 404

    return jsonify(usuarios[id])

if __name__ == '__main__':
    app.run(debug=True)
```

**Recibir JSON:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/usuario', methods=['POST'])
def crear_usuario():
    data = request.get_json()

    if not data or 'nombre' not in data or 'email' not in data:
        return jsonify({"error": "Faltan campos obligatorios"}), 400

    usuario = {
        "id": 4,  # En producción, generar ID autoincremental
        "nombre": data['nombre'],
        "email": data['email'],
        "rol": data.get('rol', 'usuario')
    }

    return jsonify(usuario), 201

if __name__ == '__main__':
    app.run(debug=True)
```

---

### 6.7 El Concepto de Microservicio

**Microservicio:** Pequeño servicio independiente que hace una sola cosa bien.

```
┌─────────────────────────────────────────────────────────┐
│                    Arquitectura de Microservicios          │
├─────────────┬──────────────┬──────────────┬─────────────┤
│  App Symfony │  Microservicio│  Microservicio│  Base de   │
│  (Foro)      │  de IA (Flask)│  de Email    │  Datos     │
│             │              │              │             │
│  - Gestión  │  - Análisis  │  - Envío     │  - MySQL   │
│  - Usuarios │    de        │    de        │  - Redis   │
│  - Temas    │    sentimiento│   emails    │             │
│  - Respuestas│  - Moderación│  - Notif.   │             │
└─────────────┴──────────────┴──────────────┴─────────────┘
         │                │                │
         └────────────────┴────────────────┘
                    Comunicación HTTP/JSON
```

**Ventajas de microservicios:**

- Independencia: cada servicio puede escalarse por separado
- Tecnología: cada servicio puede usar el lenguaje/framework que quiera
- Mantenimiento: más fácil de entender y modificar
- Resiliencia: si un servicio falla, los otros siguen funcionando

**Desventajas:**

- Complejidad: más servicios = más infraestructura
- Comunicación: latencia entre servicios
- Testing: más difícil de testear en conjunto

---

### 6.8 Crear un Microservicio con Flask

**Microservicio de análisis de sentimiento:**

```python
# app.py
from flask import Flask, request, jsonify
import os

app = Flask(__name__)

# Simulación de análisis de sentimiento
def analizar_sentimiento(texto):
    """
    En producción, aquí se llamaría a una API de IA.
    Por ahora, simulamos con reglas simples.
    """
    palabras_positivas = ['bueno', 'excelente', 'genial', 'increíble', 'fantástico', 'me gusta']
    palabras_negativas = ['malo', 'terrible', 'horrible', 'no me gusta', 'feo', 'mal']

    texto_lower = texto.lower()
    positivo = sum(1 for palabra in palabras_positivas if palabra in texto_lower)
    negativo = sum(1 for palabra in palabras_negativas if palabra in texto_lower)

    if positivo > negativo:
        return "positivo"
    elif negativo > positivo:
        return "negativo"
    else:
        return "neutral"

@app.route('/api/sentimiento', methods=['POST'])
def obtener_sentimiento():
    data = request.get_json()

    if not data or 'texto' not in data:
        return jsonify({"error": "Falta el campo 'texto'"}), 400

    texto = data['texto']
    sentimiento = analizar_sentimiento(texto)

    return jsonify({
        "texto": texto,
        "sentimiento": sentimiento
    })

@app.route('/health', methods=['GET'])
def health_check():
    """Endpoint de verificación de salud"""
    return jsonify({"status": "ok", "servicio": "analisis-sentimiento"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001, debug=True)
```

**Ejecutar el microservicio:**

```bash
python app.py
# El microservicio se ejecuta en http://localhost:5001
```

**Probar con curl:**

```bash
# Analizar sentimiento
curl -X POST http://localhost:5001/api/sentimiento \
  -H "Content-Type: application/json" \
  -d '{"texto": "Este foro es excelente y muy útil"}'

# Verificar salud
curl http://localhost:5001/health
```

---

### 6.9 Llamar a un Microservicio desde Symfony

**Llamar al microservicio de Flask desde Symfony:**

```php
// src/Service/AiService.php
<?php

namespace App\Service;

use Symfony\Contracts\HttpClient\HttpClientInterface;

class AiService
{
    private HttpClientInterface $httpClient;
    private string $aiUrl;

    public function __construct(HttpClientInterface $httpClient)
    {
        $this->httpClient = $httpClient;
        $this->aiUrl = $_ENV['AI_SERVICE_URL'] ?? 'http://localhost:5001';
    }

    public function analizarSentimiento(string $texto): string
    {
        try {
            $response = $this->httpClient->request('POST', "{$this->aiUrl}/api/sentimiento", [
                'json' => [
                    'texto' => $texto,
                ],
            ]);

            $data = $response->toArray();
            return $data['sentimiento'] ?? 'neutral';
        } catch (\Exception $e) {
            // Si el microservicio no está disponible, devolver neutral
            return 'neutral';
        }
    }
}
```

**Configuración:**

```env
# .env
AI_SERVICE_URL=http://localhost:5001
```

**Usar en el controlador:**

```php
// src/Controller/TemaController.php
#[Route('/tema/nuevo', name: 'tema_nuevo')]
#[IsGranted('IS_AUTHENTICATED_FULLY')]
public function nuevo(
    Request $request,
    EntityManagerInterface $entityManager,
    AiService $aiService
): Response {
    $tema = new Tema();
    $tema->setTitulo($request->request->get('titulo'));
    $tema->setContenido($request->request->get('contenido'));
    $tema->setAutor($this->getUser());

    // Analizar sentimiento del contenido
    $sentimiento = $aiService->analizarSentimiento($tema->getContenido());

    $entityManager->persist($tema);
    $entityManager->flush();

    return $this->render('tema/show.html.twig', [
        'tema' => $tema,
        'sentimiento' => $sentimiento,
    ]);
}
```

---

### 6.10 Manejo de Errores

**Manejo de errores en Flask:**

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "Recurso no encontrado"}), 404

@app.errorhandler(500)
def internal_error(error):
    return jsonify({"error": "Error interno del servidor"}), 500

@app.errorhandler(400)
def bad_request(error):
    return jsonify({"error": "Petición inválida"}), 400
```

**Manejo de errores en Python:**

```python
@app.route('/api/sentimiento', methods=['POST'])
def obtener_sentimiento():
    try:
        data = request.get_json()

        if not data or 'texto' not in data:
            return jsonify({"error": "Falta el campo 'texto'"}), 400

        texto = data['texto']
        sentimiento = analizar_sentimiento(texto)

        return jsonify({
            "texto": texto,
            "sentimiento": sentimiento
        })

    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

---

## 💻 Ejercicios

### Ejercicio 6.1: Fundamentos de Python

1. Instalar Python y crear un entorno virtual
2. Crear variables de diferentes tipos
3. Escribir funciones y clases
4. Crear un script que gestione una lista de tareas

```python
# tareas.py
class Tarea:
    def __init__(self, titulo, completada=False):
        self.titulo = titulo
        self.completada = completada

    def marcar_completada(self):
        self.completada = True

    def __str__(self):
        estado = "✅" if self.completada else "⬜"
        return f"{estado} {self.titulo}"

# Crear tareas
tareas = [
    Tarea("Estudiar PHP"),
    Tarea("Estudiar Symfony"),
    Tarea("Hacer ejercicio"),
]

# Marcar una como completada
tareas[0].marcar_completada()

# Mostrar todas
for tarea in tareas:
    print(tarea)
```

---

### Ejercicio 6.2: Primer Microservicio con Flask

Crear un microservicio que:

1. Tenga un endpoint `/` que muestre "Microservicio de Saludos"
2. Tenga un endpoint `/saludo/<nombre>` que devuelva JSON con un saludo
3. Tenga un endpoint `/health` para verificación de salud

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/')
def index():
    return jsonify({"mensaje": "Microservicio de Saludos"})

@app.route('/saludo/<nombre>')
def saludo(nombre):
    return jsonify({"saludo": f"¡Hola, {nombre}!"})

@app.route('/health')
def health():
    return jsonify({"status": "ok"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

---

### Ejercicio 6.3: API de Productos

Crear una API REST para gestionar productos:

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

productos = [
    {"id": 1, "nombre": "Portátil", "precio": 899.99},
    {"id": 2, "nombre": "Ratón", "precio": 29.99},
    {"id": 3, "nombre": "Teclado", "precio": 59.99},
]

@app.route('/api/productos', methods=['GET'])
def listar():
    return jsonify(productos)

@app.route('/api/productos/<int:id>', methods=['GET'])
def mostrar(id):
    producto = next((p for p in productos if p['id'] == id), None)
    if not producto:
        return jsonify({"error": "Producto no encontrado"}), 404
    return jsonify(producto)

@app.route('/api/productos', methods=['POST'])
def crear():
    data = request.get_json()
    if not data or 'nombre' not in data or 'precio' not in data:
        return jsonify({"error": "Faltan campos"}), 400

    nuevo = {
        "id": len(productos) + 1,
        "nombre": data['nombre'],
        "precio": data['precio']
    }
    productos.append(nuevo)
    return jsonify(nuevo), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

---

### Ejercicio 6.4: Integrar con Symfony

1. Crear el microservicio de Flask
2. Crear el servicio `AiService` en Symfony
3. Llamar al microservicio desde un controlador de Symfony
4. Probar la integración

---

### Ejercicio 6.5: Proyecto - Microservicio de Moderación

Crear un microservicio que:

1. Reciba un texto
2. Analice si contiene palabras ofensivas
3. Devuelva un resultado (aprobado/rechazado)
4. Tenga endpoint de health check

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

palabras_ofensivas = ['mala palabra 1', 'mala palabra 2', 'mala palabra 3']

@app.route('/api/moderar', methods=['POST'])
def moderar():
    data = request.get_json()

    if not data or 'texto' not in data:
        return jsonify({"error": "Falta el campo 'texto'"}), 400

    texto = data['texto'].lower()
    encontrado = [p for p in palabras_ofensivas if p in texto]

    if encontrado:
        return jsonify({
            "aprobado": False,
            "motivo": f"Palabras detectadas: {', '.join(encontrado)}"
        })

    return jsonify({
        "aprobado": True,
        "motivo": "Texto aprobado"
    })

@app.route('/health')
def health():
    return jsonify({"status": "ok"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5002)
```

---

## 🏗️ Proyecto Práctico: Microservicio de Análisis de Sentimiento

**Objetivo:** Crear un microservicio en Flask que analice el sentimiento de textos.

**Pasos:**

1. **Crear el entorno virtual:**

```bash
mkdir microservicio-ia
cd microservicio-ia
python3 -m venv venv
source venv/bin/activate
pip install flask requests python-dotenv
```

2. **Crear el microservicio:**

```python
# app.py
from flask import Flask, request, jsonify

app = Flask(__name__)

def analizar_sentimiento(texto):
    # Simulación con reglas simples
    positivas = ['bueno', 'excelente', 'genial', 'increíble', 'fantástico']
    negativas = ['malo', 'terrible', 'horrible', 'feo', 'mal']

    texto_lower = texto.lower()
    positivo = sum(1 for p in positivas if p in texto_lower)
    negativo = sum(1 for p in negativas if p in texto_lower)

    if positivo > negativo:
        return "positivo"
    elif negativo > positivo:
        return "negativo"
    else:
        return "neutral"

@app.route('/api/sentimiento', methods=['POST'])
def obtener_sentimiento():
    data = request.get_json()

    if not data or 'texto' not in data:
        return jsonify({"error": "Falta el campo 'texto'"}), 400

    texto = data['texto']
    sentimiento = analizar_sentimiento(texto)

    return jsonify({
        "texto": texto,
        "sentimiento": sentimiento
    })

@app.route('/health')
def health():
    return jsonify({"status": "ok"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

3. **Ejecutar:**

```bash
python app.py
```

4. **Probar:**

```bash
curl -X POST http://localhost:5001/api/sentimiento \
  -H "Content-Type: application/json" \
  -d '{"texto": "Este foro es excelente"}'
```

5. **Integrar con Symfony** (siguiente unidad)

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ Fundamentos de Python (variables, funciones, clases)
- ✅ Comparación PHP vs Python
- ✅ Instalación y configuración de Flask
- ✅ Rutas y manejo de peticiones JSON
- ✅ El concepto de microservicio
- ✅ Crear un microservicio con Flask
- ✅ Llamar a un microservicio desde Symfony

**Próxima unidad:** UD07 - Integración de Inteligencia Artificial (IA) en el Backend

---

## 🔗 Recursos Adicionales

- [Python Documentation](https://docs.python.org/3/)
- [Flask Documentation](https://flask.palletsprojects.com/)
- [Flask Quickstart](https://flask.palletsprojects.com/en/latest/quickstart/)
- [Python vs PHP](https://realpython.com/python-vs-php/)
- [Microservices.io](https://microservices.io/)
