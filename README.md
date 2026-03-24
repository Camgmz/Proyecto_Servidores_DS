# Smart Data Center

El siguiente proyecto muestra la implementación de un Smart Data Center con arquitectura IoT. El sistema utiliza Device Shadow para gestionar actuadores según la telemetría recolectada por sensores ambientales, e incluye un dashboard local en Node-RED y un flujo de alertas críticas (detección de incendios e intrusos) gestionado por SNS. Toda la actividad y los logs del sistema se almacenan de forma persistente en una base de datos NoSQL con DynamoDB.

## Instrucciones de conexión

### a) Node-RED

1. Inicializar Node-RED en la consola del sistema mediante el comando <node-red>.

2. Acceder al servidor mediante el puerto  http://127.0.0.1:1880/

3. Importar el flujo Smart_Datacenter_flow_FINAL.json

4. Seleccionar uno de los nodos MQTT y crear un nuevo broker para la conexión con AWS IoT Core, con las siguientes características:

	- Server: Endpoint de AWS
	- Port: 8883
	- Protocol: MQTTV 3.1.1

Asimismo, habilitar la opción "Use TLS" y crear una configuración TLS nueva. Ingresar los archivos de certificado (.pem.crt) , llave privada (.pem.key) y AmazonCA1.pem del objeto (thing) de AWS IoT Core con el que se llevará a cabo la simulación.

5. Una vez configurado, seleccionar el mismo broker para los siguientes nodos:
	- AWS --> In
	- AWS --> Out
	- Node-RED --> DynamoDB
	- Wokwi --> Node-RED

6. Hacer click en Deploy.

7. Para visualizar el dashboard, acceder al puerto  http://127.0.0.1:1880/ui

### b) Wokwi

1. Ingresar al enlace: https://wokwi.com/projects/459396131267350529

2. Dar click en "Start the simulation".

3. Una vez compilado, en el monitor serial se mostrarán los siguientes mensajes para confirmar la conexión a WiFi y a MQTT.


"Conectando a WiFi... conectado!"
"Intentando conectar a MQTT... conectado!"

## Ejecución de eventos

### a) Dashboard Node-RED

Dentro del dashboard local, en el grupo "Eventos", se encuentran cuatro botones:

### 1) ACTIVAR HVAC DE EMERGENCIA ###: Incrementa un grado de temperatura y 2% de humedad cada segundo, por 16 segundos, hasta superar los límites superiores ( temperatura > 32°C y humedad > 80%). La condición se sostiene por 5 segundos antes de disparar el encendido del HVAC de emergencia. La condición se mantiene por 10 segundos, y eventualmente tanto temperatura como humedad decrecen hasta regresar al interior de los parámetros seguros.

2) ACTIVAR GENERADOR: Decrementa 5% de batería por segundo hasta llegar a 15%. La condición se mantiene por 5 segundos antes de disparar el encendido del generador. 

La condición se mantiene por 10 segundos, y eventualmente el porcentaje de batería regresa al interior de los parámetros aceptables.

3) ALERTA INTRUSO: Envía una alerta de intruso. Se muestra un pop up de alerta en el dashboard, la puerta se bloquea y el sistema envía un mensaje de alerta de intruso.

La alerta de intruso también puede activarse por medio del input de credenciales. En el grupo Control de Acceso, en el campo "credencial" puede ingresarse texto.

La condición para autorizar el acceso es que la credencial ingresada termine en los caracteres E32. Según esta condición, se despliegan los siguientes eventos:

	a) Acceso autorizado: La credencial es correcta, se muestra un pop up de "Acceso autorizado" en el dashboard y la puerta se abre.
	b) Acceso no autorizado: La credencial es incorrecta, y el contador "Intentos" incrementa en 1. La puerta permanece cerrada.
	c) Alerta de intruso: Cuando el contador "Intentos" llega a 3, se activa la alerta de intruso. Aparece un pop up de alerta en el dashboard, la puerta se bloquea y el sistema envía un mensaje de alerta de intruso.

4) ALERTA INCENDIO: Incrementa el nivel de CO por 4 ppm cada segundo, hasta superar el límite superior (CO > 50 ppm). 

La condición se mantiene por 5 segundos antes de de disparar la alerta de incendio. Se muestran dos pop ups en el dashboard: uno correspondiente a la alerta de incendio y otro sobre el encendido del extintor, se enciende el sistema de extinción, el sistema envía un mensaje de la alerta de incendio y la puerta se abre, independientemente del estado anterior (bloqueada o cerrada).

La condición se mantiene por 10 segundos, y gradualmente el nivel de CO regresa por debajo del límite superior.


### b) Simulación ESP32 en Wokwi

El circuito simulado en Wokwi representa el mecanismo de acceso y alerta de intruso con credencial, y establece comunicación con el dashboard local mediante el protocolo MQTT.

El circuito cuenta con tres push button de colores verde, rojo y azul; cada uno representa el input de una credencial correcta, incorrecta y reset del sistema respectivamente.

Al presionar el botón verde, se envía una credencial correcta a Node-RED y se enciende el LED verde del circuito.

Al presionar el botón rojo, se envía una credencial incorrecta y un incremento al contador de intentos a Node-RED. El LED amarillo se enciende al presionar este botón.

Una vez que el contador de intentos llegue a 3, se envía una alerta de intruso a Node-RED y se enciende el LED rojo.

El botón azul reinicia el contador y el mensaje enviado a Node-RED.

## Diagrama de flujo
<img width="686" height="453" alt="diagrama" src="https://github.com/user-attachments/assets/86c889ac-d614-4235-baac-6a5ea2608aaf" />

