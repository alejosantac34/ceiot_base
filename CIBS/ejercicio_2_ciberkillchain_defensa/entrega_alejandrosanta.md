# Ejercicio CiberKillChain - Defensa

# Alumno

**Alejandro Santa Calderón**

**Facultad de ingeniería UBA**

## Enunciado

Desarrollar la defensa en función del ataque planteado en orden inverso, mencionar una medida de detección y una de mitigación, sólo lo más importante, considerar recursos limitados. No es una respuesta a un incidente, hay que detectar el ataque independientemente de la etapa.


## Resolución


Para el proceso de resolución tendremos dos consideraciones, una metodologia que se anticipe desde el diseño a posibles ataques de este tipo (proactiva) y otra que sea reactiva y que intervenga al momento de aparecer dicho ataque.

Se planteo instalar medidas de detección y mitigación robustas, desde el diseño y la operación, considerando la naturaleza desatendida de la subestación y los recursos limitados, contra el ataque simulado al sistema de monitoreo.

## Defensa Proactiva y Reactiva (Inversa) con Técnicas de la Matriz ATT&CK


### Defensa desde el Diseño y Construcción (Proactivo)

Como el constructor de este sistema, integré la seguridad desde el inicio. Para **proteger el dispositivo de monitoreo (sensores)**, implementé una firma digital básica para cada lectura. Así, la unidad central, al recibir los datos, siempre verificará su integridad, dificultando la inyección de lecturas falsas sin conocer mi clave secreta, la cual reside segura en el hardware de cada sensor y en la unidad central. Además, la comunicación de los sensores se realiza a través de un protocolo dedicado y una red físicamente aislada de la red principal, utilizando cableado específico o una red inalámbrica encriptada y autenticada.

Para la **Raspberry Pi 3B / computador industrial (unidad central)**, opté por un sistema operativo Linux minimalista, instalando solo lo esencial y aplicando un hardening básico: deshabilité servicios innecesarios, configuré un firewall local restrictivo (solo permitiendo la comunicación con los sensores y el SAS), y bloqueé el acceso root remoto. También implementé un monitoreo de integridad básico de los archivos críticos del sistema y de la aplicación, utilizando un script simple que verifica hashes al inicio para alertarme sobre cualquier cambio no autorizado. Finalmente, habilité un registro detallado de eventos del sistema y de la aplicación, almacenándolos localmente.

En cuanto al **SAS de la subestación**, la unidad central se comunica a través de protocolo IEC 61850 e, idealmente, en una red lógicamente separada. En el SAS, configuré reglas de firewall para aceptar solo las conexiones necesarias desde mi unidad central. Si el SAS lo permite, la comunicación se diseñó con autenticación mutua para que ambos sistemas se verifiquen antes de intercambiar datos.

### Defensa Reactiva (Inversa de la Cyber Kill Chain)


## 7.Actions on Objectives.

Dado que la subestación opera sin personal diario, la detección manual es inviable en tiempo real.

**Detección:**

Desde mi rol como programador, implementé un **análisis avanzado de anomalías en los datos de los sensores (T1565.003)** directamente en la unidad central. Analizo las series de tiempo de las lecturas para detectar patrones inusuales o cambios bruscos que no concuerdan con el comportamiento ambiental típico. Por ejemplo, si los valores de humedad permanecen estáticos dentro de un rango "normal" durante horas cuando deberían variar, esto generará una alerta. Además, si la conectividad lo permite, programé la unidad central para **validar las lecturas con datos ambientales externos (T1565)** de APIs meteorológicas confiables. Una discrepancia significativa, como una baja humedad reportada por mis sensores cuando las estaciones cercanas indican alta humedad, activará una alarma.

**Mitigación:**

Como arquitecto del sistema, diseñé un **protocolo de alerta automatizado y escalada (Procedimiento)**. Si se detectan anomalías significativas en los datos o inconsistencias con fuentes externas, el sistema enviará automáticamente alertas detalladas al personal de supervisión por SMS, correo electrónico o una plataforma centralizada, indicando la anomalía y la hora. Además, implementé un **modo de operación segura (Fail-Safe) (T1565)**. Si las anomalías persisten y sugieren manipulación, el sistema generará una alerta de posible riesgo de omisión de lavado, incluso si las lecturas están dentro de rangos nominales, requiriendo una inspección manual obligatoria antes del próximo lavado programado.

## 6.Command & Control.

**Detección:**

Como constructor del sistema, programé la unidad central para **monitorear todas las conexiones de red salientes (T1041)**. Si detecta conexiones a IPs o puertos no autorizados o inesperados, especialmente si utilizan protocolos atípicos para comunicarse con el SAS, generará una alerta. También implementé un **análisis básico de los patrones de tráfico (T1041)**, buscando flujos inusuales en frecuencia, tamaño de paquetes o intervalos, que podrían indicar un canal de comando y control encubierto.

**Mitigación:**

Como administrador del sistema, configuré un **firewall de salida restrictivo (T1530)** en la Raspberry Pi/computador industrial. Solo se permiten las conexiones salientes necesarias hacia el SAS y, potencialmente, un servidor de logs seguro conocido. Todo otro tráfico saliente está bloqueado.

## 5.Installation.

**Detección:**

Como programador, implementé un **monitoreo de integridad del sistema reforzado (T1546)**. Utilizo una herramienta que genera alertas en tiempo real ante cualquier modificación no autorizada de archivos críticos, creación de nuevas cuentas o alteraciones en las tareas programadas y servicios del sistema operativo. Además, programé un **detector básico de procesos anómalos (T1053)** que me alertará sobre cualquier proceso desconocido o inesperado que se inicie en la unidad central.

**Mitigación:**

Como administrador del sistema, deshabilité todos los servicios de ejecución remota innecesarios (**T1021**), como SSH y Telnet, en la unidad central. El acceso administrativo está restringido al acceso local seguro (consola).

## 4.Exploitation.

**Detección:**

Como desarrollador de la aplicación de monitoreo, implementé una **validación robusta de todas las entradas (T1210)**, tanto de los sensores como de cualquier interfaz de configuración local. Programé el sistema para detectar y bloquear cualquier entrada que no cumpla con los formatos y rangos esperados, incluso si superficialmente parecen "normales".

**Mitigación:**

Como arquitecto del sistema, seguí **prácticas de desarrollo seguro (T1190)** para prevenir vulnerabilidades comunes como inyecciones de código. Realicé pruebas de seguridad básicas durante el desarrollo.


## 3.Delivery.

**Detección:**

Como administrador de la red (aunque sea básica), configuré un **monitoreo de los logs de acceso (T1566)** a la interfaz web local de la unidad central (si existe). Buscaré intentos de acceso inusuales o fallidos desde IPs no autorizadas.

**Mitigación:**

Como arquitecto de la red, mantuve la **red del sistema de monitoreo estrictamente aislada (T1530)** de cualquier red corporativa o externa no esencial, minimizando las posibles vías de entrega de payloads maliciosos.


## 2.Weaponization.

**Detección:**

Como programador, implementé una **verificación periódica de la integridad del software (T1588)** de la aplicación de monitoreo, comparando los hashes de los archivos ejecutables con valores seguros conocidos.

**Mitigación:**

Como administrador del sistema, establecí un proceso de **actualizaciones de software controladas (T1486)**, realizando las actualizaciones del sistema operativo y la aplicación solo desde fuentes confiables y verificando las firmas digitales de los paquetes antes de la instalación.


## 1.Reconnaissance.

**Detección:**

Como administrador de la red, configuré el **firewall perimetral (si existe) para llevar un registro detallado (T1595, T1046)** de cualquier intento de escaneo o acceso no autorizado a la red de la subestación, incluso desde rangos de IP internos.

**Mitigación:**

Como arquitecto del sistema, trabajé para **minimizar la huella digital (T1595)** de la infraestructura de la subestación, limitando la información técnica compartida públicamente y asegurando que solo los servicios esenciales estén expuestos. También implementé **reglas de firewall básicas (T1595, T1046)** para bloquear el tráfico entrante no solicitado.

Este enfoque, donde mi conocimiento como constructor del sistema se une a mi experiencia en ciberseguridad, me permite diseñar defensas proactivas y reactivas adaptadas a las limitaciones de recursos y a la naturaleza desatendida de la subestación, priorizando la automatización de la detección y la implementación de mecanismos de seguridad desde el diseño.

