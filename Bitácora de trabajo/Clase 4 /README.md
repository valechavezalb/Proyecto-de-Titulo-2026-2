# Clase 04 
Miércoles 02 de septiembre de 2026

## Apuntes de la clase 
Revisión de avances de la semana 

### Arquitectura de la app
Pantalla de Pendientes, debe ser la vista rápida del día:
- se muestran todos los pendientes que se anotaron la noche anterior.
- previsualización de los pendientes anteriores.
- capacidad de editar para agregar o eliminar.
- botón de ver todos los pendientes.

**Pantalla de Ver todos los pendientes:**
- aquí se despliegan en una lista el historial de los pendientes para revisarlos, etc.


**Pantalla de Registro o tracker:**
- se muestra el calendario en formato mes con puntitos que indican que los días que has escrito pendientes, independientemente de que si se completaron o no.
- un botón que contiene "Mi progreso" que es donde se mostrarán estadísticas de productividad, insignias y reforzamientos positivos que permitan crear un hábito.

**Pantalla de "Mi progreso"**
- Se muestran las estadísticas como racha actual, mejor racha y cuántas veces ha escrito en el mes.

**Pantalla de Rutina y Personalización**
- vista previa del dispositivo personalizado.
- color.
- luminosidad
- inicio de cierre actual.
- botón de editar horario.

**Pantalla de edición de horarios en tres momentos:**
- horario de cierre: hora en la que se marcará el cierre del día, donde el usuario deberá escribir sus pendientes para declarar el fin de la jornada productiva.
- revisión matutina: hora en la que recordará abrir la app para ver los el "itinerario" del día.
- seguimiento durante el día: recordatorio para revisar avances (notificación en el celular, sin hostiga)

#### Con esto ya tenemos:
- Registro / Home → Mi progreso → Insignias / rachas

- Pendientes → Pendientes actuales → Añadir / editar → Todos los pendientes → check durante el día → notificaciones

- Rutina → personalización del objeto (color e intensidad) → configuración de horarios → cierre / revisión matutina / seguimiento
  
- Y todos alimentan el mismo ciclo:
NOCHE escribo y deposito → MAÑANA recupero y organizo → DÍA realizo, hago check y recibo feedback → NOCHE vuelvo a descargar lo pendiente.

## Diagrama de flujo de la aplicación
[insertar imagen]

## Arquitectura de la aplicación
[insertar imagen]

# Funcionamiento del sistema

1. La luz del dispositivo está encendida, indicando que el cierre del día todavía está pendiente.
2. El usuario escribe en una tarjeta los pendientes que quiere dejar para el día siguiente.
3. Deposita la tarjeta en el dispositivo.
4. El dispositivo detecta que la tarjeta fue ingresada y comienza a apagar la luz paulatinamente, funcionando como señal de transición hacia el descanso.
5. Mientras el usuario ya puede dar por cerrada su jornada, comienza el procesamiento en segundo plano:
- se captura/digitaliza la información escrita en la tarjeta.
- se procesa la escritura para identificar el contenido.
- se reconocen los pendientes escritos.
- la información se transforma en datos utilizables por el sistema.
- los pendientes quedan almacenados y asociados al día siguiente.
- los datos se sincronizan con la aplicación.

6. La luz termina de apagarse, indicando que el cierre del día está completo. Idealmente, desde aquí el usuario ya no necesita seguir interactuando con tecnología.
7. Durante la noche, el sistema mantiene la información procesada y preparada, sin requerir ninguna acción del usuario.
8. A la mañana siguiente, la app recupera los pendientes provenientes de la tarjeta y los muestra como tareas del día.
9. El usuario puede entonces revisarlos, editar, eliminar, agregar o priorizar tareas antes de comenzar su jornada.

Una vez listo el funcionamiento del sistema, comenzaremos con el prototipado e ideación. Para eso solo tengo una certeza: al insertar la tarjeta necesito que sea de manera horizontal en vez de vertical, para que así, el escaneo de la tarjeta sea mejor, ya que tendrá topes que guíen el recorrido de la tarjeta y que el reconocimiento sea exitoso.

**El gesto visible sería: la luz se enciende → escribir → depositar → ver cómo la luz se apaga**

Entonces un flujo resumido sería: Tarjeta ingresada → detección → captura de escritura → digitalización/OCR → interpretación de texto → identificación de pendientes → almacenamiento → asignación al día siguiente → sincronización dispositivo/app → visualización matutina.

## Componentes del prototipado

- Placa ESP32 de 30 pines wifi/bt
- 2 Sensores infrarrojo TCRT5000
- Anillo LED RGB WS2812B
- Protoboard grande 830 puntos 
- Cables macho-macho hembra-macho
- Fuente 5V/ 3A
- Resistencias surtidas 330-470
- Condensador electrolítico 1000 uf
- Interruptor on/off

Para comenzar a probar si el sistema reconoce la tarjeta insertada, sería algo así:

Tarjeta
↓
Sensor IR detecta tarjeta
↓
ESP32 recibe señal
↓
ESP32 activa secuencia de cierre
↓
WS2812B baja progresivamente su luminosidad
↓
Luz apagada = jornada cerrada

Esto ya podría testearlo con usuarios



  

