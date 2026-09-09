# 🪩 Clase 05 
Miércoles 09 de septiembre de 2026

## 🎯 Avances para esta clase...
Con la corrección de los profesores en la clase pasada (02/09), me recomendaron en vez de hacer funcionar el sistema por separado, hacer el sistema IoT completo. La clave es que el dispositivo físico haga pocas cosas y que el procesamiento complejo ocurra en la nube.

### ⚙️ La arquitectura se vería así

```
┌──────────────────────────────────────────────┐
│              DISPOSITIVO UMBRAL              │
│                                              │
│ Tarjeta → Sensor IR → ESP32-CAM → fotografía │
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

### 💡 ¿Qué es Supabase Edge Function?
Una Edge Function es como un asistente digital que vive en la nube y al que se le asigna tareas muy específicas. En mi caso, cada vez que la imagen sea capturada, esta la leerá y transcribirá lo que dice y este "asistente" lo enviará a la app. 

En resumen, es una herramienta para que los programadores creen funciones rápidas y seguras que hagan que las interacciones de paginas web, apps, etc. funcionen de forma instantánea sin que el usuario note que hay muchas cosas sucediendo detrás de un click por ejemplo.

### 💡 ¿Qué es una API?
...

# 🛠️ El paso a paso
1. **Definir el formato físico de la tarjeta**. En este caso será de 10 x 8 cm. Tendrá 5 líneas de escritura, fondo mate y de color blanco. Para que el OCR pueda leer bien la escritura.
   
1. **La cámara debe estar en una posición fija**, ya que la tarjeta se deslizará horizontalmente hasta el tope. Para eso construiré una carcasa de cartón o similar para crear esta "mini cabina de escaneo". La cámara estará arriba, la tarjeta abajo y la iluminación debe estar controlada, (la misma cámara del ESP32 trae un flash).

1. El sensor infrarrojo confirma que la tarjeta llegó, es decir, cuando el sensor IR cambia de estado, el ESP32 sabe que la tarjeta está presente y dispara la secuencia.

1. El ESP32- CAM toma una foto jpeg. Esta cámara tiene una resolución de 1600 x 1200 px por lo que saldrá una imagen nítida a poca distancia. (La idea es que el apagado paulatino de Umbral comience cuando la imagen ya haya sido recibida correctamente por el servidor, ya que así el ritual visual también comunica que el pendiente realmente fue transferido).
   
1. El ESP32 envía la imagen a la API (ya creada) mediante wifi. Esto se iría a una Supabase mediante un ``` POST ``` que básicamente es enviar la información al servidor, es decir una petición. La función de Supabase funciona como intermediaria.

1. La API guarda la captura de la imagen. Primero la debe guardar en un bucket o contenedor de Supabase Storage. Esto es muy útil porque así puedo compara el texto original escrito con el resultado del OCR y medir qué tan bien está funcionando el reconocimiento.

1. La API manda esa fotografía a Google Cloud Vision, le pediremos ``` DOCUMENT_TEXT_EDITION``` que está pensado para extraer texto de documentos y también soporta escritura en manuscrita. Un ejemplo de esto sería:

```
LO QUE DICE LA FOTOGRAFÍA:

□ Terminar presentación
□ Comprar materiales
□ Enviar avance

```
Y en el código se vería algo así:

```
{
  "text": "Terminar presentación\nComprar materiales\nEnviar avance"
}
```

1. La nube transforma ese texto en pendientes. Como en la tarjeta tendrá unas 5 o 6 líneas, serám 5 o 6 pendientes, por lo que basta con dividir el resultado OCR por saltos de línea. Como ejemplo quedaría algo así:

```
"Terminar presentación"
"Comprar materiales"
"Enviar avance"
```
Y esto se transforma en tres registros.

1. Los pendientes se guardan en una Supabase. Para el MVP que generaré para posteriormente testearlo, bastaría una tabla ``` tasks``` que tendrá un id, el texto, fecha de creación, programado para, si está completado o no, prioridad, fuente, número de captura. Y otra tabla en la que irían las capturas ```captures``` con la id de la imagen original, la foto original, el texto completo detectado, el estado (para saber si existe un error), la fecha y hora de creación.

Esto para asegurarme de que el procesamiento interno actúe de forma clara y expedita.

1. La app consulta Supabase. Para la app utilizaré Figma y FlutterFlow + Supabase, así puedo hacer que la interfaz visualmente y conectarla a las tablas de Supabase.

## 🛠️ El procesamiento interno de Umbral
El funcionamiento interno funciona como una interfaz de captura, mientras que el procesamiento de información ocurre en una infraestructura en segundo plano en la nube. Al detectar la tarjeta, el sistema captura la imagen y la envía mediante wifi a una API. La API almacena temporalmente la captura y la deriva a un servicio OCR para reconocer la escritura manuscrita. El texto resultante se estructura en pendientes individuales y se almacena en una base de datos asociada a la jornada siguiente. Finalmente, la aplicación consulta estos datos y los presenta al usuario para su organización matutina.

Así se vería el flujo a grandes rasgos: input físico → captura → transmisión → procesamiento → almacenamiento → recuperación → interfaz.


## 📲 La conexión de las interfaces
Como ya tengo las pantallas previamente diseñadas en Figma, puedo usarla como base visual, llevar estas pantallas a FlutterFlow y después concetar cada elemento a Supabase.

El flujo se vería algo así: Figma → FlutterFlow → Supabase → datos reales de Umbral 

### 💡 ¿Qué es FLutterFlow?
Es una herramienta de diseño visual Low code que permite crear aplicaciones móviles y web reales, arrastrando y soltando elementos, sin necesidad de saber programar desde cero. Entonces:

1. FlutterFlow: es la interfaz funcional de la app.
   
2. Petición HTTPs POST: cuando el usuario anotó sus pendientes en la hoja y en segundo plano se transcriben para que se visualicen en la app. (Datos de forma segura enviados del servidor a la app).

3. Supabase Edge Function: el mediador que hace que la información de los pendientes se vea reflejada en la app.

* Hay que tener el cuenta que flutterflow cuenta con planes de pago, que por ende no todas las funciones estrán disponibles en todos los planes.

### 🔌 La conexión 
- Primero hay que preparar el archivo en figma. Debe estar ordenado y organizado con cada una de sus pantallas. Además dentro de cada pantalla deben estar los elementos en Auto Layout, para que FlutterFlow pueda interpretar mejor la estructura.
  
- FlutterFlow permite crear una página usando directamente un Figma Frame URL. Para hacerlo hay que seguir lo siguiente: **FlutterFlow → Create Page → Import from Figma → conectar cuenta Figma → pegar URL del Frame → Import → Generate**.

Además puedo importar desde mi figma el sistema visual que estoy utilizando, los colores y la tipografía, desde Theme Settings → Design System → Connect to Figma.

- Después la interfaz dejará de ser una maqueta, los textos que están estáticos en la figma dejarán de serlo cuando pasen a flutterflow. Ahí tenemos que decir algo como : "este texto no debe decir siempre _Terminar presentación_, tiene que mostrar lo que venga desde Supabase. Ahí es donde conectamos la interfaz con la base de datos.

- Creamos la tabla en Supabase con las categorías indicadas más arriba.

- Conectamos supabase con flutterflow. La forma más sencilla es FlutterFlow → Settings & Integrations → Supabase → Connect with Supabase OAuth. Autorizo la cuenta, selecciono el proyecto y flutter se conecta. Después cada vez que se modifiquen las tablas de supabase, hay que usar Get Schema para que flutter actualice la estructura. Enonces flutter entenderá que hay una tabla llamada ```tasks``` y tiene un campo ```text```, ```priority```, etc.  

- Conectamos la pantalla principal de mi app. Por ejemplo, selecciono la lista donde aparecen los pendientes y hago un backend query a la tabla de pendientes o ```tasks```. FlutterFlow permite consultar las tablas de supabase directamente desde las páginas o widgets.

Entonces la app ya no muestra:
```
○ Lorem ipsum
○ Lorem ipsum
○ Lorem ipsum

```

En su lugar veremos esto:
```
SUPABASE
   ↓

task 1
"Terminar presentación"

task 2
"Comprar materiales"

task 3
"Enviar avance"

   ↓
FLUTTERFLOW

○ Terminar presentación
○ Comprar materiales
○ Enviar avance

```

Y eso ocurre automáticamente.

- Luego conectamos los botones y aquí entra programación pero flutter lo hace más visual. Por ejemplo al completar la tarea, osea hacerle check, selecciono el botón y agrego:  Action → Supabase → Update Row y ```completed = true ```.

Si quiero editar el texto de la app porque me equivoqué de pendiente, sería algo como ``` Update Row → text ```

- Ahora aparece Umbral. Lo más importante es entender que la ESP32-CAM no necesita hablar directamente con la interfaz de figma, porque el punto de encuentro será Supabase. Es decir:

```
EN LA NOCHE

TARJETA
   ↓
ESP32-CAM
   ↓
API
   ↓
OCR
   ↓
"Terminar presentación"
   ↓
SUPABASE
   ↓
tasks
```

```
AL DÍA SIGUIENTE EN LA MAÑANA

SUPABASE
   ↓
FLUTTERFLOW
   ↓
TU INTERFAZ DE FIGMA
   ↓

Buenos días

○ Terminar presentación

```

Esto es importante porque mantiene el sistema desacoplado. Porque el dispositivo escribe datos, la app los lee y los modifica y la supabase está en medio.

- Para el MVP que haré preliminarmente no es necesario que contemple absolutamente todas las pantallas. Por que la que más importa es crear manualmente en Supabase una tarea llamada por ejemplo "comprar materiales" y conseguir que aparezca automáticamente dentro de la interfaz en la parte de los pendientes. Una vez conseguido eso, conectaré el OCR.


# 🛠️ Los materiales que compré para comenzar con el prototipo
Los siguientes componentes corresponden a la primera versión funcional de Umbral Nocturno. El objetivo de es es permitir detectar la inserción de la tarjeta de pendientes, capturar su contenido, procesar la interacción y entregar retroalimentación mediante luz.




