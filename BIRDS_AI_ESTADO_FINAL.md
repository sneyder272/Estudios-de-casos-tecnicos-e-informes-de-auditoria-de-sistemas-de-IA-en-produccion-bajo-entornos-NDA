# Case Study — BirdsAI: monitoreo acústico e identificación de aves con IA

> **Proyecto:** BirdsAI — Almacén de Aves de Cali  
> **Contexto:** proyecto académico SENA, Programa de Procesamiento de Datos para Modelos de Inteligencia Artificial (PDIA)  
> **Estado:** alcance académico terminado y flujo técnico E2E validado  
> **Aplicación:** https://birds-ai-cali.web.app

---

## 1. El reto

BirdsAI nació como un prototipo orientado al monitoreo acústico de aves en entornos urbanos de Cali.

El problema no era simplemente clasificar un audio. Había que construir un flujo completo que permitiera capturar audio desde el navegador, conservarlo, asociarlo con sus metadatos, procesarlo mediante IA, ejecutar el procesamiento pesado fuera del navegador y devolver el resultado a la aplicación.

El reto principal terminó siendo de **integración de sistemas**, no solamente de machine learning.

## 2. De demo a sistema funcional

El proyecto evolucionó desde una primera interfaz web hacia una aplicación funcional con servicios cloud, almacenamiento persistente, procesamiento remoto e inteligencia artificial.

```text
Demo de interfaz
      ↓
Aplicación web funcional
      ↓
Firebase + almacenamiento persistente
      ↓
Worker Python desacoplado
      ↓
Docker + BirdNET
      ↓
Cloud Run Jobs
      ↓
Cloud Functions + Cloud Tasks
      ↓
Validación E2E desde una grabación real
```

## 3. Arquitectura

```text
Usuario
  ↓
Web App / Firebase Hosting
  ├── Firebase Storage
  └── Cloud Firestore
          ↓
  onDocumentCreated
          ↓
  Cloud Function 2nd Gen
          ↓
  Cloud Tasks
          ↓
  Cloud Run Job
          ↓
  Worker Python
      ├── FFprobe
      ├── FFmpeg
      └── BirdNET
          ↓
  Firestore
          ↓
  Biblioteca web
```

El navegador no ejecuta BirdNET, FFmpeg ni el procesamiento pesado. Su responsabilidad es capturar y enviar el audio.

## 4. Stack tecnológico

### Frontend
- JavaScript
- Firebase Hosting
- Firebase Authentication
- Cloud Firestore
- Firebase Storage
- MediaRecorder / Web Audio API
- Tailwind CSS

### Backend e IA
- Python 3.12
- BirdNET-Analyzer 2.4.0
- TensorFlow 2.20.0
- Firebase Admin SDK 6.9.0
- FFmpeg / FFprobe
- Docker

### Infraestructura
- Google Cloud Run Jobs
- Google Cloud Functions 2nd Gen
- Cloud Tasks
- Eventarc
- Artifact Registry
- Service Accounts
- Application Default Credentials

## 5. Decisiones técnicas

### Procesamiento desacoplado

Se separó la captura de audio del procesamiento de IA para evitar cargar al navegador con BirdNET, FFmpeg y las dependencias pesadas.

### Worker extensible

La lógica utiliza una abstracción de motor de identificación:

```text
SpeciesIdentificationEngine
        │
        └── BirdNetEngine
```

Esto deja preparada la arquitectura para experimentar posteriormente con un modelo propio.

### Docker

El worker, BirdNET, TensorFlow, FFmpeg y sus dependencias se empaquetan en una imagen reproducible.

### Identidad de infraestructura

El worker utiliza **Application Default Credentials (ADC)** mediante una Service Account. No requiere claves JSON dentro del repositorio ni de la imagen.

### Estados explícitos

```text
pendiente
    ↓
procesando
   ↙   ↘
procesado  error
```

Un audio procesado sin detecciones sigue siendo un resultado válido, no un error técnico.

## 6. Automatización

Cuando se crea un registro en:

```text
avistamientos/{documentId}
```

con `estado = "pendiente"`, se inicia automáticamente:

```text
Firestore
   ↓
Cloud Function
   ↓
Cloud Tasks
   ↓
Cloud Run Job
   ↓
Worker Python
```

Cloud Tasks gestiona los reintentos y la concurrencia, mientras que la Function consulta las ejecuciones activas de Cloud Run para reducir el riesgo de ejecuciones solapadas.

## 7. Captura de audio

Los navegadores pueden aplicar procesamiento orientado a voz humana, como cancelación de eco, supresión de ruido y control automático de ganancia.

Para bioacústica, esas transformaciones pueden eliminar componentes relevantes de una señal de ave. Por ello la captura se configuró con:

```javascript
{
  echoCancellation: false,
  noiseSuppression: false,
  autoGainControl: false
}
```

## 8. BirdNET como baseline

BirdNET-Analyzer 2.4.0 se utilizó como **baseline funcional**. Primero era necesario disponer de un pipeline completo y verificable antes de plantear el entrenamiento de una CNN propia.

```text
audio real
   ↓
preprocesamiento
   ↓
inferencia
   ↓
resultado estructurado
   ↓
persistencia
   ↓
visualización
```

La CNN propia queda como línea experimental futura.

## 9. Validación E2E

La validación principal se realizó desde la aplicación web:

```text
Grabación real
    ↓
Firebase Storage
    ↓
Firestore: pendiente
    ↓
Trigger automático
    ↓
Cloud Tasks
    ↓
Cloud Run
    ↓
Worker Python
    ↓
BirdNET
    ↓
Firestore
    ↓
Biblioteca web
```

Se obtuvieron ejecuciones automáticas de Cloud Run completadas correctamente y una grabación web real produjo una **identificación positiva de especie**.

Esto demuestra que el pipeline funciona como sistema integrado.

No constituye, por sí solo, una evaluación estadística de precisión biológica del modelo.

## 10. Resultados sin detección

Un análisis exitoso puede terminar sin detecciones:

```json
{
  "estado": "procesado",
  "detections": [],
  "topDetection": null
}
```

La distinción entre `procesado` y `error` permite representar correctamente un resultado negativo del análisis.

## 11. Observación humana e IA

Los campos de observación humana, como `especieUsuario` y `nombreCientifico`, se mantienen separados de las predicciones generadas por BirdNET.

Esto permite conservar ambas fuentes sin presentarlas como si fueran el mismo tipo de evidencia.

## 12. Seguridad

Los registros se asocian al usuario autenticado mediante su `uid`.

Los audios utilizan una estructura como:

```text
audios/{uid}/{timestamp}.webm
```

Las reglas de Firestore y Storage controlan el acceso desde el cliente, mientras que el worker escribe resultados mediante Firebase Admin y la identidad de infraestructura.

## 13. Retos y soluciones

| Reto | Solución |
|---|---|
| Procesamiento pesado en navegador | Worker Python remoto |
| Entorno reproducible | Docker |
| Identificación acústica | BirdNET como baseline |
| Acceso cloud seguro | Service Account + ADC |
| Procesamiento de voz del navegador | Desactivar procesamiento de voz |
| Estados ambiguos | Máquina de estados explícita |
| Ejecuciones simultáneas | Cloud Tasks + guard de Cloud Run |
| Cambio futuro de modelo | `SpeciesIdentificationEngine` |
| Validación | Prueba E2E iniciada desde la Web App |

## 14. Lecciones aprendidas

1. **Un modelo de IA no es el sistema completo.** La inferencia es solamente una etapa del flujo.
2. **Las integraciones importan tanto como el código.** La complejidad apareció principalmente en los límites entre servicios.
3. **Los estados deben representar la realidad.** `procesado` sin detecciones no equivale a `error`.
4. **Los experimentos deben mantenerse separados del flujo operacional.**
5. **La prueba E2E cambia la definición de «funciona».** El criterio final fue una grabación real que atravesara automáticamente todo el pipeline y produjera un resultado persistido.

## 15. Estado final

BirdsAI alcanzó el alcance técnico definido para su etapa académica:

- aplicación web desplegada;
- autenticación;
- captura de audio;
- Firebase Storage;
- Firestore;
- worker Python;
- Docker;
- FFmpeg / FFprobe;
- BirdNET;
- Cloud Run Jobs;
- Cloud Functions 2nd Gen;
- Cloud Tasks;
- Service Account + ADC;
- procesamiento automático;
- estados de procesamiento;
- resultados persistidos;
- validación E2E real;
- documentación técnica.

**Aplicación:** https://birds-ai-cali.web.app

## 16. Repositorio y documentación

- [Repositorio Birds-AI-cali](https://github.com/sneyder272/Birds-AI-cali)
- [Documentación técnica](https://github.com/sneyder272/Birds-AI-cali/blob/main/Documentaci%C3%B3n%20Tecnica.md)
- [Worker Python](https://github.com/sneyder272/Birds-AI-cali/tree/main/backend-python)
- [Arquitectura](https://github.com/sneyder272/Birds-AI-cali/blob/main/docs/arquitectura.md)
- [Verificación local del worker](https://github.com/sneyder272/Birds-AI-cali/blob/main/docs/worker-local-verification.md)
- [Handoff del worker](https://github.com/sneyder272/Birds-AI-cali/blob/main/docs/worker-handoff.md)

---

> **BirdsAI convirtió una grabación acústica realizada desde el navegador en un registro procesado mediante un pipeline cloud desacoplado, reproducible y automatizado, utilizando BirdNET como baseline de identificación.**
