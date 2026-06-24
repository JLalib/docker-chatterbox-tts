# 🗣️  Chatterbox TTS Docker - API de Text-to-Speech con Clonación de Voz

**Chatterbox TTS Docker** es la implementación en contenedores de **Chatterbox TTS**, una potente API local de texto a voz (TTS) completamente compatible con la API de OpenAI. Su característi
ca estrella es la capacidad de **clonar cualquier voz** con una muestra de audio de tan solo 10 segundos, permitiendo generar speech natural y multilingüe sin depender de servicios en la nube
 costosos.

---

## ✨  Características principales

- **Clonación de Voz Zero-Shot**: Clona tu propia voz o cualquier otra con una muestra de audio de ~10-30 segundos.
- **Compatibilidad con OpenAI**: Funciona como un reemplazo directo (*drop-in replacement*) para cualquier aplicación que utilice la API de TTS de OpenAI.
- **Soporte Multilingüe**: Generación de voz en **22 idiomas** diferentes.
- **Soporte de Hardware Flexible**: Optimizado para **GPU NVIDIA (CUDA)** y **AMD (ROCm)**, con soporte para **CPU** (aunque con menor rendimiento).
- **Frontend React incluido**: Interfaz web intuitiva para generar audio y gestionar voces sin usar la terminal.
- **API de alto rendimiento**: Basada en **FastAPI**, ofreciendo respuestas rápidas y documentación automática (Swagger).
- **Control Paramétrico**: Ajuste de exageración emocional, peso de fidelidad (CFG weight) y semillas para resultados consistentes.
- **Procesamiento Inteligente**: Fragmentación automática de textos largos para evitar cortes en el audio.

---

## ⚠️  Requisitos previos

- **Docker** y **Docker Compose** (versión 2.x o superior).
- **4-8 GB RAM** (dependiendo del modelo y hardware).
- **10+ GB de espacio en disco** (para modelos y librería de voces).
- **GPU (Altamente recomendada)**: NVIDIA (6GB+ VRAM) o AMD ROCm. El uso en CPU es posible pero significativamente más lento.
- **Puerto 4123** disponible.
- **Muestra de voz**: Un archivo `.mp3` o `.wav` de 10-30 segundos de voz clara y sin ruido.

---

## ⚙️  Configuración con Docker Compose

### 1. Clona este repositorio y accede al directorio:
```bash
git clone https://github.com/JLalib/docker-chatterbox-tts.git
cd chatterbox-tts-docker
```

### 2. Prepara tu muestra de voz
Para que la aplicación use tu voz por defecto, coloca tu archivo de audio en la raíz del proyecto con el nombre `voice-sample.mp3`:
```bash
cp /ruta/a/tu/grabacion.mp3 ./voice-sample.mp3
```

### 3. Archivo `docker-compose.yml`
Crea el archivo `docker-compose.yml` con la siguiente configuración:

```yaml
version: '3.8'

services:
  chatterbox-tts:
    build:
      context: .
      dockerfile: Dockerfile # Asumiendo que usas el Dockerfile del repo oficial
    container_name: chatterbox-tts-api
    restart: unless-stopped
    ports:
      - "4123:4123"
    volumes:
      - ./voice-sample.mp3:/app/voice-sample.mp3:ro
      - chatterbox-models:/cache
      - chatterbox-voices:/voices
    environment:
      - DEVICE=${DEVICE} # 'cuda' para NVIDIA, 'cpu' para procesador
      - EXAGGERATION=${EXAGGERATION}
      - CFG_WEIGHT=${CFG_WEIGHT}
      - VOICE_SAMPLE_PATH=/app/voice-sample.mp3
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

volumes:
  chatterbox-models:
  chatterbox-voices:
```

### 4. Configura las variables de entorno
Crea un archivo `.env` para gestionar la configuración:

```env
# Hardware: 'cuda' (Recomendado) o 'cpu'
DEVICE=cuda

# Control de voz (Valores sugeridos 0.0 a 1.0)
EXAGGERATION=0.7
CFG_WEIGHT=0.4
```

### 5. Despliega el servicio
```bash
docker compose up -d
```

### 6. Accede a la aplicación
- **Interfaz Web**: [http://localhost:4123](http://localhost:4123)
- **Documentación API**: [http://localhost:4123/docs](http://localhost:4123/docs)

---

## 🛠️  Primeros pasos

### 1. Verificar la salud del servicio
Ejecuta el siguiente comando para confirmar que la API está respondiendo:
```bash
curl http://localhost:4123/health
```

### 2. Generar audio desde la UI
1. Abre `http://localhost:4123` en tu navegador.
2. Escribe el texto que deseas convertir a voz.
3. Haz clic en **"Generate Speech"** y escucha tu voz clonada.

### 3. Integración con Open WebUI (LLM Local)
Para que tu chatbot hable con tu voz:
1. Ve a **Settings → Admin Settings → Text-to-Speech**.
2. Selecciona el motor **OpenAI**.
3. API Key: `cualquier-clave` (no es requerida).
4. Base URL: `http://localhost:4123/v1`.
5. Voice: `default`.

---

## 🔒  Seguridad y recomendaciones

### 1. Optimización de Hardware
Si experimentas errores de **"Out of Memory" (OOM)** en la GPU:
- Cambia `DEVICE=cpu` en el archivo `.env` (sacrificarás velocidad por estabilidad).
- Asegúrate de tener instalados los `nvidia-container-toolkit` en tu host Linux.

### 2. Configura HTTPS (Producción)
Para acceso externo seguro, utiliza **Caddy** como reverse proxy:

**`Caddyfile`**:
```
tts.tudominio.com {
    reverse_proxy localhost:4123
}
```

### 3. Gestión de Muestras de Voz
Para cambiar la voz clonada sin reiniciar todo el sistema:
1. Reemplaza el archivo `voice-sample.mp3` en el directorio raíz.
2. Reinicia el contenedor:
   ```bash
   docker compose restart
   ```

---

## 📂  Estructura del proyecto

```
./
├── docker-compose.yml    # Orquestación de la API y Volúmenes
├── .env                  # Configuración de hardware y parámetros de voz
├── voice-sample.mp3      # Muestra de audio para la clonación (Input)
├── chatterbox-models/    # Cache de modelos descargados (Volumen)
├── chatterbox-voices/    # Librería de voces gestionadas (Volumen)
└── README-Chatterbox-TTS-Docker.md  # Este archivo
```

---

## 🔄  Actualización y mantenimiento

### Actualizar Chatterbox TTS
```bash
docker compose pull
docker compose up -d --force-recreate
```

### Comandos útiles
| Comando                          | Descripción                                      |
|----------------------------------|--------------------------------------------------|
| `docker compose logs -f`         | Monitorizar la generación de audio y errores.     |
| `nvidia-smi`                     | Verificar el uso de VRAM de la GPU.               |
| `docker compose restart`         | Reiniciar la API tras cambiar la muestra de voz. |
| `docker stats chatterbox-tts-api`| Ver consumo de RAM y CPU en tiempo real.          |

---

## 📊  Comparativa con alternativas

| Característica               | Chatterbox TTS Docker | Eleven Labs       | Google Cloud TTS | pyttsx3           |
|------------------------------|-----------------------|-------------------|-------------------|-------------------|
| **Autohospedado**            | ✅  Sí                 | ❌  No             | ❌  No             | ✅  Sí             |
| **Clonación de Voz Local**   | ✅  Sí (Zero-shot)      | ✅  Sí (SaaS)      | ❌  No             | ❌  No             |
| **Costo**                   | ✅  Gratis / Open Source| ❌  Suscripción     | ❌  Pago por uso    | ✅  Gratis         |
| **Privacidad**               | ✅  Total              | ❌  Nube           | ❌  Nube           | ✅  Total           |
| **Compatibilidad OpenAI**    | ✅  Sí                 | ❌  No             | ❌  No             | ❌  No             |

---

## 📚  Referencias

- [Repositorio Oficial](https://github.com/travisvn/chatterbox-tts-api)
- [Documentación de la API](https://github.com/travisvn/chatterbox-tts-api/blob/main/docs/API_README.md)
- [Guía de Docker Oficial](https://github.com/travisvn/chatterbox-tts-api/blob/main/docs/DOCKER_README.md)

---
