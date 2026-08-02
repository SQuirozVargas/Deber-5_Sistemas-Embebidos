# Tarea #5 - Sistemas Embebidos: ESP32 con UART, I2C y FreeRTOS

Nombre: Saúl Alejandro Quiroz Vargas
Materia: Sistemas Embebido
Paralelo: 1

Este repositorio contiene los tres ejercicios de la Tarea #5, enfocados en la
comunicación serial UART, el protocolo I2C y la ejecución concurrente de tareas
con FreeRTOS sobre el microcontrolador ESP32.


## Índice
- Ejercicio 1 - Comunicación Serial con UART2
- Ejercicio 2 - Sistema Multitarea con FreeRTOS
- Ejercicio 3 - Sistema Integrado UART + FreeRTOS + I2C
- [Enlaces]
- Enlace del ejercicio 1: https://wokwi.com/projects/471199744503740417
- Enlace del ejercicio 2: https://wokwi.com/projects/471200832395098113
- Enlace del ejercicio 3: https://wokwi.com/projects/471201654473482241
- Video de Youtube: https://youtu.be/kuNR116xm48

# Ejercicio 1
# Comunicación Serial Avanzada con UART2 (ESP-IDF)

Framework: ESP-IDF

# Descripción
Aplicación que implementa un sistema de comandos seriales sobre el puerto
**UART2** del ESP32, utilizando los drivers nativos de ESP-IDF. El sistema recibe
comandos de texto, los interpreta y responde de forma estructurada.

# Funcionamiento del sistema
El programa inicializa UART2 (pines TX2=GPIO17, RX2=GPIO16) con un baud rate
configurable. En un bucle no bloqueante, va leyendo carácter por carácter hasta
detectar un fin de línea, momento en el que arma el comando completo y lo procesa.
Según el comando recibido, ejecuta la acción correspondiente y devuelve una
respuesta estructurada por el mismo puerto UART.

# Comandos disponibles
| Comando   | Acción                                                      |
|-----------|------------------------------------------------------------ | 
| `status`  | Devuelve el estado del sistema                              |
| `led on`  | Enciende el LED físico del pin GPIO2                        |
| `led off` | Apaga el LED                                                |
| `info`    | Muestra puerto, baud rate y contador de comandos            |
| `reset`   | Reinicia las variables internas                             |

# Características técnicas
- Uso exclusivo de **UART2** con drivers nativos ESP-IDF (`uart_driver_install`,
  `uart_param_config`, `uart_set_pin`, `uart_write_bytes`).
- Manejo de buffers de RX/TX de 1024 bytes.
- Lectura **no bloqueante**.
- Código estructurado en funciones.
- LED físico conectado a GPIO2 mediante resistencia de 220Ω.

#### Arquitectura
En el terminal: UART--> [TareaUART/Lectura] --> [process_command()]
Fisicamente: [LED GPIO2] [Estado] [Respuesta UART2]

# Ejercicio 2
# Sistema Multitarea con FreeRTOS

Framework: Arduino

# Descripción
Sistema multitarea que ejecuta **tres tareas concurrentes** mediante FreeRTOS,
cada una con su propia frecuencia y prioridad.

# Funcionamiento del sistema
En el arranque se crean tres tareas independientes con `xTaskCreate()`. Cada una
corre en su propio ciclo, usando `vTaskDelay()` para marcar su ritmo. El
planificador de FreeRTOS reparte el tiempo de CPU entre ellas según su prioridad,
permitiendo que se ejecuten de forma concurrente sin bloquearse entre sí.

# Tareas implementadas
| Tarea         | Prioridad | Frecuencia  | Función                                      |
|---------------|-----------|-------------|--------------------------------------------- |
| TareaSensor   | 3         | cada 1 s    | Lee un sensor virtual con valores aleatorios.|
| TareaLED      | 2         | cada 500 ms | Parpadea un LED en GPIO2                     |
| TareaReporte  | 1         | cada 2 s    | Envía el valor del sensor al monitor         |

# Justificación de FreeRTOS frente a un loop tradicional
Con un único `loop()` habría que entrelazar manualmente los tiempos (por ejemplo
con `millis()`), y cualquier retardo en una parte bloquearía el resto del código.
Con FreeRTOS cada tarea corre de forma independiente a su propia frecuencia,
mejorando la organización, la capacidad de respuesta y aprovechando los dos
núcleos del ESP32.

# Compilación y ejecución
1. Abrir en Wokwi con plantilla **Arduino**.
2. Compilar y simular.
3. Observar el LED parpadeando y los reportes periódicos en el Serial Monitor.



# Ejercicio 3
# Sistema Integrado UART + FreeRTOS + I2C (OLED)

Framework: Arduino

# Descripción
Sistema completo que integra **comunicación UART**, **multitarea con FreeRTOS** y
una **pantalla OLED SSD1306 por I2C**. Los comandos recibidos por UART se procesan
en tareas dedicadas comunicadas mediante una **cola (Queue)** de FreeRTOS, y la
OLED muestra un gráfico de línea de tiempo con la actividad de las tareas.

# Funcionamiento del sistema
La tarea de UART escucha los comandos entrantes y los coloca en una cola. La tarea
de control toma los comandos de esa cola y ejecuta la acción correspondiente. En
paralelo, una tarea de sensor genera lecturas periódicas y una tarea de display
actualiza continuamente el gráfico en la pantalla OLED, dibujando en una línea de
tiempo cuándo se activa cada tarea.

# Comandos disponibles
| Comando   | Acción                                              |
|-----------|-----------------------------------------------------|
| `led on`  | Enciende el LED en GPIO2                            |
| `led off` | Apaga el LED                                        |
| `status`  | Muestra el estado del LED y el contador de comandos |

# Tareas implementadas
| Tarea         | Prioridad | Función                                            |
|---------------|-----------|----------------------------------------------------|
| TareaUART     | 3         | Recibe comandos por UART y los envía a la cola     |
| TareaControl  | 2         | Toma comandos de la cola y ejecuta acciones        |
| TareaSensor   | 1         | Lectura periódica de un sensor virtual             |
| TareaDisplay  | 1         | Actualiza el gráfico de línea de tiempo en la OLED |

# Comunicación entre tareas
Se utiliza una cola de FreeRTOS  para pasar los comandos desde la tarea que lee el U
ART hasta la tarea que los ejecuta, garantizando una comunicación segura y ordenada 
entre tareas.

# Conexiones de la OLED (I2C)
| OLED | ESP32   |
|------|---------|
| VCC  | 3V3     |
| GND  | GND     |
| SDA  | GPIO21  |
| SCL  | GPIO22  |

# Arquitectura
[Terminal] --UART--> [TareaUART] --Queue--> [TareaControl] --> [LED / Estado]

[TareaSensor] -----------------------------> [Historial de actividad]

[TareaDisplay] --I2C--> [OLED SSD1306]

# Compilación y ejecución
1. Abrir en Wokwi con plantilla **Arduino**.
2. En el **Library Manager**, agregar: `Adafruit SSD1306` y `Adafruit GFX Library`.
3. Compilar y simular.
4. Escribir comandos en el Serial Monitor y observar el gráfico en la OLED.


