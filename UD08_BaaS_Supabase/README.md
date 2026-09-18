# UD08: Arquitecturas Alternativas: Backend-as-a-Service (BaaS) ✨

## 🎯 Objetivos

Al finalizar esta unidad, el alumno será capaz de:

- Explicar qué es Backend-as-a-Service (BaaS)
- Comparar BaaS con backend tradicional
- Configurar Supabase como alternativa a Firebase
- Implementar autenticación de usuarios con Supabase
- Usar base de datos en tiempo real (Postgres)
- Almacenar ficheros con Supabase Storage

---

## 📖 Contenidos

### 8.1 ¿Qué es BaaS?

**BaaS** (Backend-as-a-Service) proporciona un backend completo listo para usar, sin necesidad de gestionar servidores.

```
┌─────────────────────────────────────────────────────────┐
│                  Aplicación Frontend                      │
│              (JavaScript, React, Vue, etc.)               │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                    BaaS Provider                          │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Auth    │  │   DB     │  │  Storage │              │
│  │ (Login)  │  │(Postgres) │  │ (Files)  │              │
│  └──────────┘  └──────────┘  └──────────┘              │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │ Realtime     │  │  Functions   │                    │
│  │ (WebSockets) │  │  (Serverless)│                    │
│  └──────────────┘  └──────────────┘                    │
└─────────────────────────────────────────────────────────┘
```

**Proveedores BaaS:**

| Proveedor | Base de datos | Autenticación | Storage | Realtime |
| --- | --- | --- | --- | --- |
| **Supabase** | PostgreSQL | Sí | Sí | Sí |
| Firebase | Firestore | Sí | Sí | Sí |
| Appwrite | PostgreSQL | Sí | Sí | Sí |
| AWS Amplify | DynamoDB | Sí | Sí | Sí |

---

### 8.2 Ventajas y Desventajas de BaaS

**Ventajas:**

- ✅ Desarrollo más rápido (backend listo)
- ✅ Sin gestión de servidores
- ✅ Escalabilidad automática
- ✅ Autenticación incluida
- ✅ Base de datos en tiempo real
- ✅ Open Source (Supabase)

**Desventajas:**

- ❌ Dependencia del proveedor (vendor lock-in)
- ❌ Menos control sobre la infraestructura
- ❌ Coste puede aumentar con mucho uso
- ❌ Limitaciones personalización

---

### 8.3 Introducción a Supabase

**Supabase** es una alternativa Open Source a Firebase, basada en PostgreSQL.

**Características:**

- Base de datos PostgreSQL completa
- Autenticación de usuarios
- Almacenamiento de ficheros (Storage)
- Tiempo real (Realtime)
- Funciones serverless (Edge Functions)
- API REST y GraphQL automática

**Crear proyecto:**

1. Ir a https://supabase.com/
2. Crear cuenta
3. Crear nuevo proyecto
4. Obtener credenciales

---

### 8.4 Configuración de Supabase

**Instalar SDK de Supabase:**

```bash
# Para JavaScript (frontend)
npm install @supabase/supabase-js

# Para Python
pip install supabase

# Para PHP
composer require supabase/php
```

**Obtener credenciales:**

```javascript
// config/supabase.js
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = 'https://tu-proyecto.supabase.co'
const supabaseKey = 'tu-anon-key-aqui'

export const supabase = createClient(supabaseUrl, supabaseKey)
```

**Archivo `.env`:**

```env
SUPABASE_URL=https://tu-proyecto.supabase.co
SUPABASE_ANON_KEY=tu-anon-key-aqui
SUPABASE_SERVICE_ROLE_KEY=tu-service-role-key-aqui
```

---

### 8.5 Autenticación con Supabase

**Registro de usuario:**

```javascript
// auth/register.js
import { supabase } from './config/supabase'

async function registrar(email, password, nombre) {
    const { data, error } = await supabase.auth.signUp({
        email: email,
        password: password,
    })

    if (error) {
        console.error('Error al registrar:', error.message)
        return
    }

    // Guardar datos adicionales en la tabla profiles
    const { error: profileError } = await supabase
        .from('profiles')
        .insert([{
            id: data.user.id,
            nombre: nombre,
            email: email
        }])

    if (profileError) {
        console.error('Error al crear perfil:', profileError.message)
    }
}
```

**Login de usuario:**

```javascript
// auth/login.js
import { supabase } from './config/supabase'

async function login(email, password) {
    const { data, error } = await supabase.auth.signInWithPassword({
        email: email,
        password: password,
    })

    if (error) {
        console.error('Error al iniciar sesión:', error.message)
        return
    }

    console.log('Usuario logueado:', data.user)
}
```

**Logout:**

```javascript
// auth/logout.js
import { supabase } from './config/supabase'

async function logout() {
    const { error } = await supabase.auth.signOut()
    if (error) {
        console.error('Error al cerrar sesión:', error.message)
    }
}
```

**Obtener usuario actual:**

```javascript
// auth/current-user.js
import { supabase } from './config/supabase'

async function obtenerUsuarioActual() {
    const { data: { user } } = await supabase.auth.getUser()
    return user
}

// Escuchar cambios de autenticación
supabase.auth.onAuthStateChange((event, session) => {
    if (session) {
        console.log('Sesión activa:', session.user.email)
    } else {
        console.log('Sesión cerrada')
    }
})
```

---

### 8.6 Base de Datos con Supabase

**Crear tabla en Supabase:**

1. Ir a Table Editor en el dashboard de Supabase
2. Crear nueva tabla `tareas`

**Estructura de la tabla `tareas`:**

| Columna | Tipo | Descripción |
| --- | --- | --- |
| id | uuid (PK) | ID automático |
| titulo | text | Título de la tarea |
| descripcion | text | Descripción |
| completada | boolean | Estado de la tarea |
| usuario_id | uuid (FK) | Usuario propietario |
| creado_en | timestamp | Fecha de creación |

**Insertar datos:**

```javascript
// tareas/create.js
import { supabase } from './config/supabase'

async function crearTarea(titulo, descripcion, usuarioId) {
    const { data, error } = await supabase
        .from('tareas')
        .insert([{
            titulo: titulo,
            descripcion: descripcion,
            completada: false,
            usuario_id: usuarioId,
            creado_en: new Date().toISOString()
        }])
        .select()

    if (error) {
        console.error('Error al crear tarea:', error.message)
        return
    }

    console.log('Tarea creada:', data[0])
}
```

**Consultar datos:**

```javascript
// tareas/list.js
import { supabase } from './config/supabase'

async function listarTareas() {
    const { data, error } = await supabase
        .from('tareas')
        .select('*')
        .order('creado_en', { ascending: false })

    if (error) {
        console.error('Error al listar tareas:', error.message)
        return
    }

    console.log('Tareas:', data)
    return data
}
```

**Actualizar datos:**

```javascript
// tareas/update.js
import { supabase } from './config/supabase'

async function marcarCompletada(id, completada) {
    const { error } = await supabase
        .from('tareas')
        .update({ completada: completada })
        .eq('id', id)

    if (error) {
        console.error('Error al actualizar:', error.message)
    }
}
```

**Eliminar datos:**

```javascript
// tareas/delete.js
import { supabase } from './config/supabase'

async function eliminarTarea(id) {
    const { error } = await supabase
        .from('tareas')
        .delete()
        .eq('id', id)

    if (error) {
        console.error('Error al eliminar:', error.message)
    }
}
```

---

### 8.7 Tiempo Real con Supabase

**Escuchar cambios en tiempo real:**

```javascript
// tareas/realtime.js
import { supabase } from './config/supabase'

// Escuchar cambios en la tabla tareas
const subscription = supabase
    .channel('tareas-channel')
    .on('postgres_changes', {
        event: '*',
        schema: 'public',
        table: 'tareas'
    }, (payload) => {
        console.log('Cambio detectado:', payload)

        if (payload.eventType === 'INSERT') {
            console.log('Nueva tarea:', payload.new)
            // Actualizar la UI con la nueva tarea
        } else if (payload.eventType === 'UPDATE') {
            console.log('Tarea actualizada:', payload.new)
            // Actualizar la UI con la tarea actualizada
        } else if (payload.eventType === 'DELETE') {
            console.log('Tarea eliminada:', payload.old)
            // Eliminar la tarea de la UI
        }
    })
    .subscribe()

// Dejar de escuchar
// subscription.unsubscribe()
```

**Aplicación de tareas colaborativas en tiempo real:**

```javascript
// app.js
import { supabase } from './config/supabase'

class TareaApp {
    constructor() {
        this.tareas = []
        this.inicializar()
    }

    async inicializar() {
        // Obtener usuario actual
        const { data: { user } } = await supabase.auth.getUser()
        if (!user) {
            window.location.href = '/login.html'
            return
        }

        // Cargar tareas
        await this.cargarTareas()

        // Escuchar cambios en tiempo real
        this.escucharCambios()
    }

    async cargarTareas() {
        const { data, error } = await supabase
            .from('tareas')
            .select('*')
            .eq('usuario_id', (await supabase.auth.getUser()).data.user.id)
            .order('creado_en', { ascending: false })

        if (error) {
            console.error('Error:', error.message)
            return
        }

        this.tareas = data
        this.renderizarTareas()
    }

    escucharCambios() {
        supabase
            .channel('tareas-channel')
            .on('postgres_changes', {
                event: '*',
                schema: 'public',
                table: 'tareas',
                filter: `usuario_id=eq.${(await supabase.auth.getUser()).data.user.id}`
            }, (payload) => {
                this.cargarTareas()
            })
            .subscribe()
    }

    renderizarTareas() {
        const container = document.getElementById('tareas-container')
        container.innerHTML = ''

        this.tareas.forEach(tarea => {
            const div = document.createElement('div')
            div.className = 'tarea'
            div.innerHTML = `
                <input type="checkbox" ${tarea.completada ? 'checked' : ''}
                       onchange="marcarCompletada(${tarea.id}, ${!tarea.completada})">
                <span class="titulo ${tarea.completada ? 'completada' : ''}">${tarea.titulo}</span>
                <button onclick="eliminarTarea(${tarea.id})">🗑️</button>
            `
            container.appendChild(div)
        })
    }
}

// Iniciar aplicación
const app = new TareaApp()
```

---

### 8.8 Almacenamiento con Supabase

**Subir ficheros:**

```javascript
// storage/upload.js
import { supabase } from './config/supabase'

async function subirFichero(fichero, nombreBucket) {
    const nombreFichero = `${Date.now()}-${fichero.name}`

    const { data, error } = await supabase.storage
        .from(nombreBucket)
        .upload(nombreFichero, fichero)

    if (error) {
        console.error('Error al subir:', error.message)
        return
    }

    // Obtener URL pública
    const { data: { publicUrl } } = supabase.storage
        .from(nombreBucket)
        .getPublicUrl(nombreFichero)

    console.log('URL pública:', publicUrl)
    return publicUrl
}

// Uso
const input = document.getElementById('fichero-input')
input.addEventListener('change', async (event) => {
    const fichero = event.target.files[0]
    const url = await subirFichero(fichero, 'imagenes')
    console.log('Fichero subido:', url)
})
```

**Listar ficheros:**

```javascript
// storage/list.js
import { supabase } from './config/supabase'

async function listarFicheros(bucket) {
    const { data, error } = await supabase.storage
        .from(bucket)
        .list()

    if (error) {
        console.error('Error:', error.message)
        return
    }

    console.log('Ficheros:', data)
    return data
}
```

**Eliminar ficheros:**

```javascript
// storage/delete.js
import { supabase } from './config/supabase'

async function eliminarFichero(nombreFichero, bucket) {
    const { error } = await supabase.storage
        .from(bucket)
        .remove([nombreFichero])

    if (error) {
        console.error('Error:', error.message)
    }
}
```

---

### 8.9 Proyecto: App de Tareas con Supabase

**Objetivo:** Crear una aplicación web de tareas colaborativas usando Supabase.

**Estructura del proyecto:**

```
app-tareas/
├── index.html
├── css/
│   └── style.css
├── js/
│   ├── config/
│   │   └── supabase.js
│   ├── auth/
│   │   ├── login.js
│   │   ├── register.js
│   │   └── logout.js
│   ├── tareas/
│   │   ├── create.js
│   │   ├── list.js
│   │   ├── update.js
│   │   └── delete.js
│   └── app.js
└── README.md
```

**HTML principal:**

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>App de Tareas</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <h1>📋 Mis Tareas</h1>
        <button id="logout-btn">Cerrar Sesión</button>
    </header>

    <main>
        <section id="crear-tarea">
            <h2>Nueva Tarea</h2>
            <input type="text" id="titulo" placeholder="Título de la tarea">
            <textarea id="descripcion" placeholder="Descripción"></textarea>
            <button id="crear-btn">Crear Tarea</button>
        </section>

        <section id="lista-tareas">
            <h2>Mis Tareas</h2>
            <div id="tareas-container"></div>
        </section>
    </main>

    <script type="module" src="js/app.js"></script>
</body>
</html>
```

**JavaScript principal:**

```javascript
// js/app.js
import { supabase } from './config/supabase.js'

class TareaApp {
    constructor() {
        this.tareas = []
        this.inicializarEventos()
        this.verificarSesion()
    }

    async verificarSesion() {
        const { data: { session } } = await supabase.auth.getSession()

        if (!session) {
            window.location.href = 'login.html'
            return
        }

        this.cargarTareas()
        this.escucharCambios()
    }

    inicializarEventos() {
        document.getElementById('crear-btn').addEventListener('click', () => this.crearTarea())
        document.getElementById('logout-btn').addEventListener('click', () => this.cerrarSesion())
    }

    async crearTarea() {
        const titulo = document.getElementById('titulo').value
        const descripcion = document.getElementById('descripcion').value

        if (!titulo) {
            alert('El título es obligatorio')
            return
        }

        const { data: { user } } = await supabase.auth.getUser()

        const { error } = await supabase.from('tareas').insert([{
            titulo,
            descripcion,
            completada: false,
            usuario_id: user.id
        }])

        if (error) {
            console.error('Error:', error.message)
            return
        }

        document.getElementById('titulo').value = ''
        document.getElementById('descripcion').value = ''
    }

    async cargarTareas() {
        const { data: { user } } = await supabase.auth.getUser()

        const { data, error } = await supabase
            .from('tareas')
            .select('*')
            .eq('usuario_id', user.id)
            .order('creado_en', { ascending: false })

        if (error) {
            console.error('Error:', error.message)
            return
        }

        this.tareas = data
        this.renderizarTareas()
    }

    renderizarTareas() {
        const container = document.getElementById('tareas-container')
        container.innerHTML = ''

        this.tareas.forEach(tarea => {
            const div = document.createElement('div')
            div.className = `tarea ${tarea.completada ? 'completada' : ''}`
            div.innerHTML = `
                <input type="checkbox" ${tarea.completada ? 'checked' : ''}
                       onchange="app.marcarCompletada(${tarea.id}, ${!tarea.completada})">
                <div>
                    <strong>${tarea.titulo}</strong>
                    <p>${tarea.descripcion || ''}</p>
                </div>
                <button onclick="app.eliminarTarea(${tarea.id})">🗑️</button>
            `
            container.appendChild(div)
        })
    }

    async marcarCompletada(id, completada) {
        const { error } = await supabase
            .from('tareas')
            .update({ completada })
            .eq('id', id)

        if (error) {
            console.error('Error:', error.message)
        }
    }

    async eliminarTarea(id) {
        const { error } = await supabase
            .from('tareas')
            .delete()
            .eq('id', id)

        if (error) {
            console.error('Error:', error.message)
        }
    }

    escucharCambios() {
        supabase
            .channel('tareas-channel')
            .on('postgres_changes', {
                event: '*',
                schema: 'public',
                table: 'tareas'
            }, () => {
                this.cargarTareas()
            })
            .subscribe()
    }

    async cerrarSesion() {
        await supabase.auth.signOut()
        window.location.href = 'login.html'
    }
}

// Iniciar aplicación
const app = new TareaApp()
```

---

## 💻 Ejercicios

### Ejercicio 8.1: Configurar Supabase

1. Crear proyecto en Supabase
2. Obtener URL y API key
3. Crear archivo `.env` con las credenciales
4. Instalar SDK y probar conexión

```javascript
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
    process.env.SUPABASE_URL,
    process.env.SUPABASE_ANON_KEY
)

const { data, error } = await supabase.from('tareas').select('*')
console.log(data)
```

---

### Ejercicio 8.2: Autenticación

Implementar login y registro con Supabase Auth:

```javascript
// Login
const { data, error } = await supabase.auth.signInWithPassword({
    email: 'usuario@email.com',
    password: 'contraseña'
})

// Registro
const { data, error } = await supabase.auth.signUp({
    email: 'usuario@email.com',
    password: 'contraseña'
})
```

---

### Ejercicio 8.3: CRUD con Supabase

Crear una aplicación que permita:

1. Listar registros
2. Crear nuevo registro
3. Actualizar registro
4. Eliminar registro

---

### Ejercicio 8.4: Tiempo Real

Implementar tiempo real para que los cambios se reflejen automáticamente:

```javascript
supabase
    .channel('tareas-channel')
    .on('postgres_changes', {
        event: '*',
        schema: 'public',
        table: 'tareas'
    }, (payload) => {
        console.log('Cambio:', payload)
    })
    .subscribe()
```

---

### Ejercicio 8.5: Proyecto - App Colaborativa

Crear una aplicación de tareas colaborativa con:

1. Autenticación de usuarios
2. CRUD de tareas
3. Tiempo real (cambios instantáneos)
4. Almacenamiento de imágenes (adjuntar fotos a tareas)

---

## 🏗️ Proyecto Práctico: App de Tareas Colaborativa

**Objetivo:** Crear una aplicación web completa con Supabase.

**Pasos:**

1. **Crear proyecto en Supabase**

2. **Crear tabla `tareas`:**

```sql
CREATE TABLE tareas (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    titulo TEXT NOT NULL,
    descripcion TEXT,
    completada BOOLEAN DEFAULT FALSE,
    usuario_id UUID REFERENCES auth.users(id),
    creado_en TIMESTAMP DEFAULT NOW()
);
```

3. **Configurar RLS (Row Level Security):**

```sql
ALTER TABLE tareas ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Usuarios ven sus propias tareas"
ON tareas FOR ALL
USING (auth.uid() = usuario_id);
```

4. **Crear la aplicación frontend con HTML/JS**

5. **Probar la aplicación**

---

## 📝 Resumen

En esta unidad hemos aprendido:

- ✅ Qué es BaaS y sus ventajas/desventajas
- ✅ Supabase como alternativa Open Source a Firebase
- ✅ Autenticación de usuarios con Supabase Auth
- ✅ Base de datos PostgreSQL con Supabase
- ✅ Tiempo real con Supabase Realtime
- ✅ Almacenamiento de ficheros con Supabase Storage
- ✅ Crear una aplicación completa con Supabase

**Próxima unidad:** UD09 - El Paso a Producción: Docker y Despliegue

---

## 🔗 Recursos Adicionales

- [Supabase Documentation](https://supabase.com/docs)
- [Supabase JavaScript Client](https://supabase.com/docs/reference/javascript/introduction)
- [Supabase Auth](https://supabase.com/docs/reference/javascript/auth-signinwithpassword)
- [Supabase Realtime](https://supabase.com/docs/reference/javascript/realtime-subscribe)
- [Supabase Storage](https://supabase.com/docs/reference/javascript/storage)
- [Row Level Security](https://supabase.com/docs/guides/auth/row-level-security)
