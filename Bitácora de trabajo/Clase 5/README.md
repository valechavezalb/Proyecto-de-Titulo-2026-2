# Clase 05 
Miércoles 09 de septiembre de 2026

## Avances para esta clase...
Con la corrección de los profesores en la clase pasada (02/09), me recomendaron en vez de hacer funcionar el sistema por separado, hacer el sistema IoT completo. La clave es que el dispositivo físico haga pocas cosas y que el procesamiento complejo ocurra en la nube.

### La arquitectura se vería así

```
┌──────────────────────────────────────────────┐
│              DISPOSITIVO UMBRAL              │
│                                              │
│ Tarjeta → Sensor IR → ESP32-CAM → fotografía│
│                           │                  │
│                           └── Wi-Fi          │
└───────────────────────────────│──────────────┘
                                ↓
                         INTERNET / HTTPS
                                ↓
┌──────────────────────────────────────────────┐
│                    NUBE                      │
│                                              │
│         API / Supabase Edge Function         │
│                    ↓                         │
│           guarda fotografía                  │
│                    ↓                         │
│        Google Cloud Vision OCR API           │
│                    ↓                         │
│          devuelve texto escrito              │
│                    ↓                         │
│       separa texto en pendientes             │
│                    ↓                         │
│       guarda pendientes en base de datos     │
└───────────────────────────────│──────────────┘
                                ↓
┌──────────────────────────────────────────────┐
│                     APP                      │
│                                              │
│           consulta base de datos             │
│                    ↓                         │
│            muestra pendientes                │
│                    ↓                         │
│       editar · eliminar · priorizar          │
└──────────────────────────────────────────────┘
```
Esto es lo que pasaría en segundo plano, el ESP32-CAM no hace OCR. Fotografía y manda la imagen, la API en la nube coordina el procesamiento, el OCR interpreta, la base de datos guarda y la app recupera esta información para dársela al usuario en pantalla.

Para la nube tengo pensado en usar Supabase + Google Cloud Vision. Supabase entrega una base de datos PostegreSQL, almacenamiento de imágenes, API y Edge Functions, estan funciones pueden recibir solicitudes HTTP, trabajar con Storage/ Database y llamar servicios externos manteniend las credenciales como secretos del servidor. Google Cloud Vision soporta específicamente OCR de escritura manuscrita mediante ``` DOCUMENT_TEXT_DETECTION```.

# El paso a paso
1. Definir el formato físico de la tarjeta. En este caso será de 10 x 8 cm. Tendrá 5 líneas de escritura, 

