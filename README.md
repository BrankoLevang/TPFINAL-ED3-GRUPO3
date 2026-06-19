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
