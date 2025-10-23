# Taller2-comunicaciones-industriales
## integrantes: camila perez, santiago hernandez, diego alejandro rodriguez.
## Primer punto 
PROFIBUS (PROcess Fleld Bus): Se determina como un Bus de campo serial direccionado principalmente a la automatización industrial.  

-> Topologia: Su topologia es de bus multipunto con terminación en extremos.  

-> Ventajas: Buen control ciclico, es ampliamente utilizado en PLCs y tambien en sensores industriales.  

-> Capa: Fisica y de enlace  

-> Casos de uso: Comunicaciones entre E/S remotas, comunicaciones entre PLCs y módulos en plantas industriales.  

PROFINET: Se determina como ethernet industrial definido por PROFIBUS y PROFINET, sobre un módelo de Ethernet estandar pero con perfiles para tiempo real, además de que su velocidad tipica se encuentra entre 100Mbps.   

-> Topologia: De por si entre lsa topologias comunes se encuentra la de estrella y anillo.  

-> Ventajas: Se caracteriza por tener alta velocidad, soporte de servicios, integración sencilla con redes IT y tambien diagnosticos avanzados.  

-> Capa: Capa fisica basada en ethernet.  

-> Casos de uso: Se suele utilizar para el control de fábricas modernas, sistemas que necesitan mucho ancho de banda y HMI/SCADA.  

-> Desventajas: Requiere switches y por lo tanto, suele tener mayor complejidad en la configuración.   

## Segundo punto:   

Para el segundo punto se utilizo una raspberry pi pico 2w y un servo motor.
![raspberry](Taller2/raspberry.png)
![servo Motor](Taller2/servoMotor.jpg)

En esta actividad se desarrolló un sistema de control para un servo motor SG90 utilizando una Raspberry Pi Pico 2W conectada a través de un módulo MAX485 para comunicación industrial mediante RS-485. El objetivo fue demostrar los tres modos de comunicación (Simplex, Half Dúplex y Full Dúplex) y observar cómo afectan el intercambio de información entre dispositivos.

 Descripción general del código

El programa fue desarrollado en MicroPython, aprovechando las librerías machine, utime y _thread para manejar los periféricos y la multitarea del microcontrolador.
El archivo principal contiene la clase ServoControlPico, encargada de inicializar el hardware, configurar la UART y controlar el servo por medio de señales PWM.

Principales configuraciones:

UART RS-485:

TX → GPIO0

RX → GPIO1

DE → GPIO2 (Driver Enable)

RE → GPIO3 (Receiver Enable)

Velocidad de comunicación: 9600 baudios


Servo PWM:

GPIO4 como salida PWM a 50 Hz

Rango de 0° a 180° con conversión a ciclo útil entre 0.5 ms y 2.5 ms




---

 Esquema de conexión

Raspberry Pi Pico 2W → Módulo MAX485

GPIO0 → DI  
GPIO1 → RO  
GPIO2 → DE  
GPIO3 → RE  
3.3V  → VCC  
GND   → GND

MAX485 → Bus RS-485

A → A  
B → B

Raspberry Pi Pico 2W → Servo SG90

GPIO4 → Señal (naranja)  
5V    → VCC (rojo)  
GND   → GND (marrón)

> Se añadieron resistencias de 120 Ω en los extremos del bus y un capacitor de 100 nF entre VCC y GND del módulo MAX485 para estabilidad eléctrica.






 Funcionamiento por modos

El código implementa tres modos de comunicación que pueden ejecutarse desde el menú principal o mediante las funciones de demostración:

1. Modo Simplex

Solo se transmite información del maestro hacia el esclavo.

La función demo_simplex() envía una secuencia de comandos que cambian el ángulo del servo (0°, 45°, 90°, 135°, 180°…).

El sistema usa set_transmit_mode() para mantener DE y RE en nivel alto (solo transmisión).

Ideal para pruebas donde solo se necesita enviar datos.


2. Modo Half Dúplex

En este modo la comunicación se alterna entre transmisión y recepción.

Primero se activa set_transmit_mode() para enviar el comando con el ángulo (SET:valor), luego se cambia a set_receive_mode() para escuchar una confirmación o respuesta.

Se utiliza la función demo_half_duplex(), donde se envía un nuevo ángulo y se espera la confirmación con receive_data().

Es el modo más usado en RS-485, ya que aprovecha una sola línea para ambos sentidos, alternando los estados de DE y RE.


3. Modo Full Dúplex

Se realiza transmisión y recepción simultáneamente (requiere dos módulos RS-485 o dos Picos).

La función demo_full_duplex() crea un hilo secundario con _thread.start_new_thread() que ejecuta continuous_receiver(), permitiendo recibir datos en paralelo mientras se transmiten comandos.

Cada mensaje enviado con formato POS:valor es respondido con ACK:valor, validando la comunicación en ambos sentidos.





 Flujo del programa principal

Al ejecutar el script, se muestra un menú que permite seleccionar entre control manual o demostraciones automáticas.
Por ejemplo, al elegir la opción “Demo Completa”, se ejecuta la secuencia:

1. demo_simplex()


2. demo_half_duplex()


3. demo_full_duplex()



Durante la ejecución, el sistema reporta el número de mensajes enviados (TX), recibidos (RX), el modo actual, y la posición del servo.
Al finalizar, se llama a cleanup() para detener el PWM y dejar el servo en posición central.




 Resultado esperado

En Simplex, el servo se mueve siguiendo una secuencia sin recibir respuesta.

En Half Dúplex, el sistema envía un comando y espera confirmación del otro extremo antes de continuar.

En Full Dúplex, se observan transmisiones y recepciones simultáneas, demostrando comunicación bidireccional continua.


Este código demuestra el principio de operación del estándar RS-485 en sistemas industriales, aplicando conceptos de comunicación serial diferencial, control de flujo, sincronización y direccionamiento, todo implementado en un entorno embebido real con Raspberry Pi Pico 2W.


