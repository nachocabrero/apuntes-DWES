# UD07: Integración de Inteligencia Artificial (IA) en el Backend 🧠

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar qué es IA como Servicio (AIaaS)
- Consumir APIs de OpenAI, Hugging Face y Google AI desde Python
- Manejar claves API de forma segura
- Implementar casos de uso: análisis de sentimiento, generación de texto, clasificación
- Integrar servicios de IA en aplicaciones Symfony

---

## 📖 Contenidos

### 7.1 IA como Servicio (AIaaS)

**AIaaS** (Artificial Intelligence as a Service) permite usar modelos de IA sin necesidad de entrenarlos ni alojarlos.

**Modelos de IAaaS:**

```
┌─────────────────────────────────────────────────────────┐
│                  Tu Aplicación (Symfony)                  │
│                                                         │
│  1. Recibir texto del usuario                            │
│  2. Enviar a API de IA                                   │
│  3. Recibir respuesta de IA                              │
│  4. Usar resultado en la aplicación                      │
└─────────────────────────────────────────────────────────┘
              │                    │
              ▼                    ▼
┌─────────────────────┐  ┌─────────────────────┐
│  OpenAI API         │  │  Hugging Face API   │
│  - GPT-3.5/4        │  │  - Modelos abiertos │
│  - DALL-E           │  │  - Text generation  │
│  - Whisper          │  │  - Image generation │
└─────────────────────┘  └─────────────────────┘
```

**Proveedores principales:**

| Proveedor | Servicios | Precio |
| --- | --- | --- |
| OpenAI | GPT-4, GPT-3.5, DALL-E, Whisper | Por uso |
| Hugging Face | Modelos abiertos, inference API | Gratis/Pago |
| Google AI | Gemini, PaLM, Vision | Por uso |
| Anthropic | Claude | Por uso |
| Azure AI | Diversos servicios | Por uso |

---

### 7.2 APIs de OpenAI

**OpenAI** ofrece APIs para:

- **GPT-4/GPT-3.5**: Generación y comprensión de texto
- **DALL-E**: Generación de imágenes
- **Whisper**: Transcripción de audio a texto
- **Embeddings**: Representación vectorial de texto

**Obtener API Key:**

1. Crear cuenta en https://platform.openai.com/
2. Ir a API Keys
3. Crear nueva clave secreta

**Instalar librería:**

```bash
pip install openai python-dotenv
```

**Ejemplo básico:**

```python
# app.py
import os
from openai import OpenAI
from dotenv import load_dotenv
from flask import Flask, request, jsonify

load_dotenv()

app = Flask(__name__)

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

@app.route('/api/chat', methods=['POST'])
def chat():
    data = request.get_json()
    mensaje = data.get('mensaje', '')

    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "Eres un asistente útil."},
            {"role": "user", "content": mensaje}
        ]
    )

    return jsonify({
        "respuesta": response.choices[0].message.content
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

**Archivo `.env`:**

```env
OPENAI_API_KEY=tu_clave_secreta_aqui
```

---

### 7.3 Hugging Face API

**Hugging Face** ofrece miles de modelos abiertos y una API de inferencia.

**Obtener API Key:**

1. Crear cuenta en https://huggingface.co/
2. Ir a Settings → Access Tokens
3. Crear token con permisos de lectura

**Instalar librería:**

```bash
pip install huggingface_hub requests python-dotenv
```

**Ejemplo con Hugging Face:**

```python
# app.py
import os
import requests
from dotenv import load_dotenv
from flask import Flask, request, jsonify

load_dotenv()

app = Flask(__name__)

HUGGINGFACE_API_URL = "https://api-inference.huggingface.co/models"
HUGGINGFACE_API_KEY = os.getenv("HUGGINGFACE_API_KEY")

headers = {"Authorization": f"Bearer {HUGGINGFACE_API_KEY}"}

@app.route('/api/sentimiento', methods=['POST'])
def analisis_sentimiento():
    data = request.get_json()
    texto = data.get('texto', '')

    # Usar modelo de análisis de sentimiento
    response = requests.post(
        f"{HUGGINGFACE_API_URL}/facebook/bart-large-mnli",
        headers=headers,
        json={"inputs": texto}
    )

    resultado = response.json()
    return jsonify({
        "texto": texto,
        "resultado": resultado
    })

@app.route('/api/generar', methods=['POST'])
def generar_texto():
    data = request.get_json()
    prompt = data.get('prompt', '')

    response = requests.post(
        f"{HUGGINGFACE_API_URL}/gpt2",
        headers=headers,
        json={"inputs": prompt, "parameters": {"max_length": 100}}
    )

    resultado = response.json()
    return jsonify({
        "prompt": prompt,
        "generado": resultado[0]['generated_text'] if resultado else ""
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

---

### 7.4 Google AI (Gemini)

**Google AI** ofrece el modelo Gemini y otros servicios.

**Obtener API Key:**

1. Ir a https://aistudio.google.com/
2. Crear proyecto
3. Generar API key

**Instalar librería:**

```bash
pip install google-generativeai python-dotenv
```

**Ejemplo con Gemini:**

```python
# app.py
import os
import google.generativeai as genai
from dotenv import load_dotenv
from flask import Flask, request, jsonify

load_dotenv()

app = Flask(__name__)

genai.configure(api_key=os.getenv("GOOGLE_API_KEY"))

model = genai.GenerativeModel("gemini-pro")

@app.route('/api/chat', methods=['POST'])
def chat():
    data = request.get_json()
    mensaje = data.get('mensaje', '')

    response = model.generate_content(mensaje)

    return jsonify({
        "respuesta": response.text
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

---

### 7.5 Manejo Seguro de Claves API

**Nunca hardcodear claves API:**

```python
# ❌ MALO: Clave hardcodeada
API_KEY = "sk-abc123def456..."

# ✅ BUENO: Usar variables de entorno
import os
API_KEY = os.getenv("OPENAI_API_KEY")
```

**Usar `.env` para desarrollo:**

```env
# .env
OPENAI_API_KEY=sk-proj-abc123...
HUGGINGFACE_API_KEY=hf_abc123...
GOOGLE_API_KEY=AIzaSy...
```

**Cargar `.env` con python-dotenv:**

```python
from dotenv import load_dotenv
load_dotenv()  # Carga las variables del archivo .env

import os
api_key = os.getenv("OPENAI_API_KEY")
```

**Nunca subir `.env` a Git:**

```gitignore
# .gitignore
.env
venv/
__pycache__/
*.pyc
```

---

### 7.6 Casos de Uso: Análisis de Sentimiento

**Análisis de sentimiento con OpenAI:**

```python
# app.py
import os
from openai import OpenAI
from dotenv import load_dotenv
from flask import Flask, request, jsonify

load_dotenv()

app = Flask(__name__)
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

@app.route('/api/sentimiento', methods=['POST'])
def analisis_sentimiento():
    data = request.get_json()
    texto = data.get('texto', '')

    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {
                "role": "system",
                "content": "Analiza el sentimiento del texto. Responde solo con: positivo, negativo o neutral."
            },
            {"role": "user", "content": texto}
        ],
        max_tokens=10
    )

    sentimiento = response.choices[0].message.content.strip().lower()

    return jsonify({
        "texto": texto,
        "sentimiento": sentimiento
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

**Probar:**

```bash
curl -X POST http://localhost:5001/api/sentimiento \
  -H "Content-Type: application/json" \
  -d '{"texto": "Este producto es excelente, me encanta"}'
```

---

### 7.7 Casos de Uso: Generación de Texto

**Generación de texto con OpenAI:**

```python
@app.route('/api/generar', methods=['POST'])
def generar_texto():
    data = request.get_json()
    prompt = data.get('prompt', '')
    max_tokens = data.get('max_tokens', 200)

    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "Eres un asistente útil."},
            {"role": "user", "content": prompt}
        ],
        max_tokens=max_tokens
    )

    return jsonify({
        "prompt": prompt,
        "respuesta": response.choices[0].message.content
    })
```

**Ejemplo de uso:**

```bash
curl -X POST http://localhost:5001/api/generar \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Escribe un resumen de la Revolución Francesa", "max_tokens": 300}'
```

---

### 7.8 Casos de Uso: Clasificación de Texto

**Clasificación con Hugging Face:**

```python
@app.route('/api/clasificar', methods=['POST'])
def clasificar():
    data = request.get_json()
    texto = data.get('texto', '')

    response = requests.post(
        f"{HUGGINGFACE_API_URL}/facebook/bart-large-mnli",
        headers=headers,
        json={"inputs": texto}
    )

    resultado = response.json()

    # Extraer las etiquetas y puntuaciones
    etiquetas = resultado[0].keys()
    puntuaciones = resultado[0].values()

    # Encontrar la etiqueta con mayor puntuación
    mejor_etiqueta = max(zip(etiquetas, puntuaciones), key=lambda x: x[1])

    return jsonify({
        "texto": texto,
        "clasificacion": mejor_etiqueta[0],
        "confianza": mejor_etiqueta[1]
    })
```

---

### 7.9 Integrar IA con Symfony

**Servicio de IA en Symfony:**

```php
// src/Service/AiService.php
<?php

namespace App\Service;

use Symfony\Contracts\HttpClient\HttpClientInterface;

class AiService
{
    private HttpClientInterface $httpClient;
    private string $openaiApiKey;

    public function __construct(HttpClientInterface $httpClient)
    {
        $this->httpClient = $httpClient;
        $this->openaiApiKey = $_ENV['OPENAI_API_KEY'];
    }

    public function analizarSentimiento(string $texto): string
    {
        $response = $this->httpClient->request('POST', 'https://api.openai.com/v1/chat/completions', [
            'json' => [
                'model' => 'gpt-3.5-turbo',
                'messages' => [
                    [
                        'role' => 'system',
                        'content' => 'Analiza el sentimiento del texto. Responde solo con: positivo, negativo o neutral.'
                    ],
                    [
                        'role' => 'user',
                        'content' => $texto
                    ]
                ],
                'max_tokens' => 10
            ],
            'headers' => [
                'Authorization' => 'Bearer ' . $this->openaiApiKey,
                'Content-Type' => 'application/json',
            ],
        ]);

        $data = $response->toArray();
        return $data['choices'][0]['message']['content'] ?? 'neutral';
    }

    public function generarTexto(string $prompt, int $maxTokens = 200): string
    {
        $response = $this->httpClient->request('POST', 'https://api.openai.com/v1/chat/completions', [
            'json' => [
                'model' => 'gpt-3.5-turbo',
                'messages' => [
                    ['role' => 'system', 'content' => 'Eres un asistente útil.'],
                    ['role' => 'user', 'content' => $prompt]
                ],
                'max_tokens' => $maxTokens
            ],
            'headers' => [
                'Authorization' => 'Bearer ' . $this->openaiApiKey,
                'Content-Type' => 'application/json',
            ],
        ]);

        $data = $response->toArray();
        return $data['choices'][0]['message']['content'] ?? '';
    }
}
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

    // Analizar sentimiento
    $sentimiento = $aiService->analizarSentimiento($tema->getContenido());

    // Si el sentimiento es negativo, advertir al usuario
    if (strtolower($sentimiento) === 'negativo') {
        $this->addFlash('warning', 'Tu mensaje tiene un tono negativo. ¿Estás seguro?');
    }

    $entityManager->persist($tema);
    $entityManager->flush();

    return $this->redirectToRoute('tema_show', ['id' => $tema->getId()]);
}
```

**Configuración:**

```env
# .env
OPENAI_API_KEY=tu_clave_secreta_aqui
```

---

### 7.10 Moderación Automática

**Moderar contenido del foro con IA:**

```php
// src/Service/ModerationService.php
<?php

namespace App\Service;

use Symfony\Contracts\HttpClient\HttpClientInterface;

class ModerationService
{
    private HttpClientInterface $httpClient;
    private string $aiServiceUrl;

    public function __construct(HttpClientInterface $httpClient)
    {
        $this->httpClient = $httpClient;
        $this->aiServiceUrl = $_ENV['AI_SERVICE_URL'] ?? 'http://localhost:5001';
    }

    public function moderarContenido(string $texto): array
    {
        try {
            $response = $this->httpClient->request('POST', "{$this->aiServiceUrl}/api/moderar", [
                'json' => ['texto' => $texto],
            ]);

            return $response->toArray();
        } catch (\Exception $e) {
            // Si el servicio no está disponible, permitir el contenido
            return ['aprobado' => true, 'motivo' => 'Servicio no disponible'];
        }
    }
}
```

```php
// src/Controller/TemaController.php
#[Route('/tema/nuevo', name: 'tema_nuevo')]
#[IsGranted('IS_AUTHENTICATED_FULLY')]
public function nuevo(
    Request $request,
    EntityManagerInterface $entityManager,
    ModerationService $moderationService
): Response {
    $contenido = $request->request->get('contenido');

    // Moderar contenido
    $moderacion = $moderationService->moderarContenido($contenido);

    if (!$moderacion['aprobado']) {
        $this->addFlash('error', 'Tu mensaje no ha sido aprobado: ' . $moderacion['motivo']);
        return $this->redirectToRoute('temas');
    }

    // Crear tema...
}
```

---

## 💻 Ejercicios

### Ejercicio 7.1: Configurar OpenAI

1. Crear cuenta en OpenAI
2. Obtener API key
3. Crear archivo `.env` con la clave
4. Probar una llamada a la API con Python

```python
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": "¿Qué es la IA?"}]
)

print(response.choices[0].message.content)
```

---

### Ejercicio 7.2: Microservicio de Sentimiento con OpenAI

Crear un microservicio en Flask que use OpenAI para analizar sentimiento:

```python
from flask import Flask, request, jsonify
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

@app.route('/api/sentimiento', methods=['POST'])
def analisis_sentimiento():
    data = request.get_json()
    texto = data.get('texto', '')

    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "Analiza el sentimiento. Responde solo: positivo, negativo o neutral."},
            {"role": "user", "content": texto}
        ],
        max_tokens=10
    )

    return jsonify({
        "texto": texto,
        "sentimiento": response.choices[0].message.content.strip().lower()
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

---

### Ejercicio 7.3: Integrar con Symfony

1. Crear el servicio `AiService` en Symfony
2. Llamar a OpenAI desde Symfony
3. Mostrar el resultado en una plantilla

---

### Ejercicio 7.4: Clasificación con Hugging Face

Crear un microservicio que clasifique texto en categorías:

```python
@app.route('/api/clasificar', methods=['POST'])
def clasificar():
    data = request.get_json()
    texto = data.get('texto', '')

    response = requests.post(
        "https://api-inference.huggingface.co/models/facebook/bart-large-mnli",
        headers={"Authorization": f"Bearer {os.getenv('HUGGINGFACE_API_KEY')}"},
        json={"inputs": texto}
    )

    resultado = response.json()
    mejor = max(resultado[0].items(), key=lambda x: x[1])

    return jsonify({
        "texto": texto,
        "categoria": mejor[0],
        "confianza": mejor[1]
    })
```

---

### Ejercicio 7.5: Proyecto - Foro con IA

Ampliar el foro para incluir:

1. Análisis de sentimiento en cada tema
2. Moderación automática de contenido
3. Sugerencias de respuestas con IA
4. Resumen automático de temas largos

---

## 🏗️ Proyecto Práctico: Microservicio de IA para el Foro

**Objetivo:** Crear un microservicio de IA que analice sentimiento y modere contenido.

**Pasos:**

1. **Obtener API keys:**

```env
# .env
OPENAI_API_KEY=tu_clave_aqui
HUGGINGFACE_API_KEY=tu_clave_aqui
```

2. **Crear microservicio en Flask:**

```python
# app.py
from flask import Flask, request, jsonify
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

@app.route('/api/sentimiento', methods=['POST'])
def analisis_sentimiento():
    data = request.get_json()
    texto = data.get('texto', '')

    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "Analiza el sentimiento. Responde solo: positivo, negativo o neutral."},
            {"role": "user", "content": texto}
        ],
        max_tokens=10
    )

    return jsonify({
        "texto": texto,
        "sentimiento": response.choices[0].message.content.strip().lower()
    })

@app.route('/api/moderar', methods=['POST'])
def moderar():
    data = request.get_json()
    texto = data.get('texto', '')

    # Usar OpenAI para moderar
    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "¿Este texto es apropiado para un foro educativo? Responde solo: SI o NO."},
            {"role": "user", "content": texto}
        ],
        max_tokens=5
    )

    respuesta = response.choices[0].message.content.strip().upper()
    aprobado = respuesta == "SI"

    return jsonify({
        "aprobado": aprobado,
        "motivo": "Texto aprobado" if aprobado else "Texto no apropiado"
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

4. **Integrar con Symfony** (servicio que llama al microservicio)

5. **Probar la integración completa**

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ Qué es IA como Servicio (AIaaS)
- ✅ Consumir APIs de OpenAI, Hugging Face y Google AI
- ✅ Manejar claves API de forma segura con `.env`
- ✅ Casos de uso: sentimiento, generación, clasificación
- ✅ Integrar IA en aplicaciones Symfony
- ✅ Moderación automática de contenido

**Próxima unidad:** UD08 - Arquitecturas Alternativas: Backend-as-a-Service (BaaS)

---

## 🔗 Recursos Adicionales

- [OpenAI Documentation](https://platform.openai.com/docs/)
- [Hugging Face Documentation](https://huggingface.co/docs)
- [Google AI Documentation](https://ai.google.dev/)
- [Python Requests](https://requests.readthedocs.io/)
- [python-dotenv](https://python-dotenv.readthedocs.io/)
