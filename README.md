# TPFINAL-ED3-GRUPO3
Participantes: Bolatti Agustin, Levang Branko U. , Muñoz Facundo


##  1. Descripción General del Proyecto

El sistema implementa un **control de barrera de peaje automatizado** sobre el microcontrolador LPC1769, que resuelve el problema de gestionar el paso vehicular sin intervención manual, validando una condición de "tarifa" antes de habilitar el acceso. La llegada de un vehículo se detecta mediante una interrupción externa (EINT0) que dispara una medición analógica vía ADC, cuya conversión es transferida a memoria mediante GPDMA sin intervención de la CPU. Según el valor leído frente a un umbral configurado, el sistema decide abrir o no la barrera, controlando un servomotor mediante PWM generado por software. Una vez abierta, un sensor de presencia y un timeout de seguridad de 15 segundos garantizan que la barrera se cierre automáticamente, ya sea por el paso efectivo del vehículo o por inactividad prolongada, evitando que quede abierta indefinidamente.

El proyecto está dirigido a un contexto **académico**, como demostración integrada de periféricos del LPC1769 (GPIO, EXTI, ADC, GPDMA, Timers y UART) trabajando en conjunto dentro de un sistema embebido con lógica de estados. Toda la operación del sistema —estado actual de la barrera, últimas 5 mediciones del ADC, estado del DMA y habilitación de la detección— se reporta en tiempo real por UART0 cada 1 segundo, y además permite el control remoto básico (habilitar/deshabilitar detección) mediante comandos de texto enviados por la misma interfaz, lo que facilita el monitoreo y la depuración durante las pruebas en banco.

### Alcances del Proyecto (¿Qué hace y qué NO hace el sistema?)

**El sistema SÍ es capaz de:**
- Detectar la llegada de un vehículo mediante un pulsador conectado a EINT0 (P2.10).
- Medir una señal analógica (tarifa simulada) por ADC y transferirla a memoria vía GPDMA sin uso de CPU.
- Decidir la apertura de la barrera comparando la medición contra un umbral fijo.
- Controlar un servomotor mediante PWM generado por software (TIMER0) para abrir y cerrar la barrera.
- Detectar la salida del vehículo mediante polling periódico de un sensor de presencia (TIMER3).
- Cerrar automáticamente la barrera por timeout de seguridad (15 s) si el vehículo no completa el paso (TIMER2).
- Mantener un historial circular de las últimas 5 mediciones del ADC.
- Transmitir el estado completo del sistema por UART0 cada 1 segundo (TIMER1): estado de la barrera, valor ADC, historial de mediciones, estado del DMA y estado de la detección.
- Recibir comandos por UART (`on`/`off`) para habilitar o deshabilitar la detección de vehículos.
- Indicar visualmente el estado mediante LEDs (barrera abierta/cerrada).
- Manejar errores de DMA, reportándolos y re-habilitando la detección automáticamente.

**El sistema NO incluye (Fuera de alcance):**
- Almacenamiento local de datos (Data Logging) en tarjeta SD ni en memoria no volátil.
- Conectividad inalámbrica Wi-Fi/Bluetooth.
- Gestión real de tarifas o cobro (cálculo de monto, transacciones, registro de pagos).
- Detección de vehículos por sensores reales (cámara, lazo inductivo, ultrasonido); se simula con pulsador.
- Múltiples carriles o barreras simultáneas.
- Protocolo de comunicación estructurado (tramas con checksum, JSON, etc.) sobre UART.
- Realimentación de posición real del servomotor (no hay verificación física del ángulo alcanzado).
- Interfaz gráfica o display local (LCD/OLED); toda la salida es por UART hacia una terminal.
- Watchdog timer para recuperación automática ante fallos de firmware.
- Calibración dinámica del umbral del ADC (valor fijo en tiempo de compilación).

###  Posibles Etapas Siguientes (Líneas Futuras)

- Migrar el circuito de protoboard a un circuito impreso (PCB) diseñado bajo normas de compatibilidad electromagnética (EMC).
- Implementar modos de bajo consumo (Sleep) administrados por hardware para permitir el uso de baterías o backup ante cortes de energía.
- Diseñar una interfaz gráfica (GUI) en Python o una app móvil para la visualización remota de las variables y el historial de pasos.
- Incorporar un sensor real de detección vehicular (lazo inductivo o cámara con procesamiento de imagen) en lugar del pulsador simulado.
- Agregar un módulo de comunicación inalámbrica (Wi-Fi/LoRa) para reportar el estado a un sistema central de gestión de peajes.
- Implementar un protocolo de comunicación estructurado (con checksum o framing) para mayor robustez frente a ruido en la UART.
- Sumar un watchdog timer por hardware para recuperación automática ante cuelgues.


##   3. Especificaciones Eléctricas, Alimentación y Entorno 
- Tension del sistema: uso general 3.3V, 5V para servomotor
- Alimentacion por USB, Uso de fuente externa para la alimentacion de servomotor
- Consumo medio:
- IDE MCUXpresso IDE v11.8 con LPCOpen v2.10
- Microcontrolador NXP LPC1769 (ARM Cortex-M3)
- Bibliotecas de Terceros y Versiones: LPCOpen / CMSIS (driver library de NXP para la familia LPC17xx)
- Periféricos Avanzados Utilizados:
NVIC — gestión de prioridades e interrupciones (NVIC_SetPriority, NVIC_EnableIRQ para EINT0, TIMER0-3, DMA, UART0, ADC)
GPDMA — transferencia ADC → memoria sin CPU (canal P2M conectado a GPDMA_CONN_ADC)
ADC — conversión por software (ADC_START_NOW), canal 0 (P0.23)
EXTI (EINT0) — interrupción externa por flanco descendente (P2.10)
Timers (TIMER0, TIMER1, TIMER2, TIMER3) — cuatro timers de hardware independientes, cada uno con match channels configurados (no se usa SysTick en este código)
UART0 — comunicación serie con cálculo manual de baud rate (DLL/DLM/FDR) y FIFO habilitado
GPIO — entradas/salidas digitales (servo, LEDs, sensor)
PINSEL — configuración de multiplexado de pines

- Estrategia de Concurrencia: Bare-metal con máquina de estados cooperativa, manejada completamente por interrupciones (sin RTOS). El main() corre un superloop simple que solo atiende flags (uart_comando_pendiente, uart_reporte_pendiente) seteados desde las ISRs; toda la lógica de tiempo real ocurre dentro de los handlers de interrupción:
- 
TIMER0 → genera el PWM del servo por software (match en MR0/MR1), es la de mayor prioridad (0) por ser timing-crítica
TIMER3 → polling del sensor de presencia cada 20 ms, prioridad 1
DMA → finalización de la conversión ADC, prioridad 2
TIMER2 → timeout de seguridad de 15 s, prioridad 3
EINT0 → detección de vehículo, prioridad 4 (la más baja del grupo crítico)
TIMER1 y UART0 → reporte periódico y recepción de comandos, prioridad 5 (la de menor peso, ya que es solo I/O hacia el usuario)

La variable global estado actúa como el estado compartido de la máquina de estados, modificado únicamente dentro de secciones protegidas por la jerarquía de prioridades del NVIC (no hay mutexes ni semáforos porque no hay RTOS).


##  4. Proceso de Integración y Desarrollo 
### Hicimos un supuesto de division de 6 etapas de desarrollo del programa:
  Etapa 1: en esta etapa de planificacion emepzamos a implementar la idea, realizamos diagramas para ver que es lo que tenia que hacer el sistema completo funcionando. Hicimos planificacion del trabajo para las siguientes etapas, que sirvieron como una guia para el avance del proyecto. Incluimos una forma de trabajo por objetivos y no por tiempo 

  Etapa 2 - Pulsador + Servo: Lo primero que teniamos que implementar fue la funcion principal de nuestro sistema. Un servomotor que simula una barrera que se abre y se cierra en raccion a la accion de nuestro pulsador. Para eso quisimos implementar un PWM por software, utilizando un timer, para la movilidad de nuestro motor, ajustando lo s grados que queriamos que girara nuestra barrera.

  Etapa 3 - Pulsador + Servo + ADC: en esta etapa decidimos implementar que el pulsador active al servomotor PERO SOLO cuando el ADC lo permita para ya simular la tarifa del peaje como una condicion.

  Etapa 4 - implementacion de un timeout: en esta altura, el ADC ya deberia tener correcta integracion en el sistema. lo que decidimos agregar como mejora, es un timeout para cerrar la barrera y que no sea controlada totalmente por el pulsador y como un seguro de que la barrera cierre ante cualquier inconsistencia.

  Etapa 4 - Pulsador + Servo + ADC + Sensor: se agregaria un sensor infrarojo que lo que hace es detectar una presencia, por ejemplo un vehiculo. Cuando la presencia es detectada la barrera permanece abierta es espera de la salida del vehiculo. Una vez el sensor deja de detectar presencia, la señal que envia la misma se corta y se manda una orden de cerrar la barrera por mas que el timeout no haya terminado.

  Etapa 5 - comunicacion por UART: decidimos agregar una comunicacion por UART bidireccional para el monitoreo del sistema. Mostraba por pantalla el estado de la interrupcion, si estaba habilitada o no, el estado actual del sistema, y el valor del ADC enviado por GPDMA. Tambien incorportaba una orden de OFF para detener todo el sistema en caso de falla o querer desactivarlo manualmente y a su vez una orden de ON para recuperar el funcionamiento. 

  Etapa 6 - pruebas dee integración: se testeo manualmente todas las funciones nombradas anteriormente como un solo sistema para el control de acciones y correccion de erorres o ajustes finos que requeria (limites, umbrales, tiempos, etc).

  
  

