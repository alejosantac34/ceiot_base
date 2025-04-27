# Ejercicio CiberKillChain - Ataque


# Alumno

Alejandro Santa Calderón
Facultad de ingeniería UBA

# Sistema de monitoreo de corriente de fuga y contaminación.

* El sistema de monitoreo de corriente de fuga en una subestación eléctrica, permite identificar el incremento de corrientes de perdidas por el aislamiento de los equipos, producto del incremento de contaminantes conductivos, que al mezclarse con la humedad relativa del ambiente genera "electrolito" conductivos, aptos para que corrientes fluyan entre el elemento energizado y tierra. Al incrementarse estas corrientes representativamente, pueden producir un "cortocircuito" en el sistema eléctrico, generando actuación de las protecciones eléctricas, disparo de circuitos, indisponibilidades de los activos de trasmisión o distribución que se ven reflejadas en Energía No Suministrada (ENS) a los clientes finales y perdidas millonarias al transmisor o distribuidor a través de multas.

El sistema consiste en sensorica para las variables mencionadas, que trasmite señales a través de protocolo LoRaWAN a una Raspberry Pi 3B o Computador industrial instalado en el edificio de control de la subestación que no solo procesa los datos de las mediciones, sino que sirve de Broker para trasmitir los datos al SAS de la subestación bajo protocolo IEC 61850

![Imagen del dispositivo.](doc/disp.jpg)

# Objetivo

 * Sabotear el sistema de monitoreo de corriente de fuga en una subestación eléctrica desatendida, para generar una falla por contaminación que produzca indisponibilidades y costos a la compañía

# Resolución

Los siguientes son los pasos del ataque.

## 1.Reconnaissance.

Como atacante, mi primer paso es **Descubrimiento (T1595)** de la infraestructura objetivo. Inicialmente, me enfocaré en obtener información pública sobre la subestación eléctrica y sus sistemas. Utilizaré **OSINT (T1595.002)** para rastrear informes técnicos, diagramas de red publicados accidentalmente, o incluso información sobre los proveedores de los sistemas de monitoreo. Buscaré manuales de usuario o especificaciones técnicas que me den detalles sobre los rangos de operación aceptables de los sensores y el protocolo de comunicación utilizado.

Luego, intentaré un **Escaneo de Red Activo (T1046)** desde ubicaciones remotas que puedan tener visibilidad hacia la red de la subestación (si está expuesta de alguna manera, aunque sea indirectamente a través de VPNs de proveedores). Esto me permitirá identificar posibles puntos de entrada y los servicios que están corriendo. Si encuentro algún portal web o servicio expuesto, realizaré un **Escaneo de Vulnerabilidades (T1595.001)** para identificar posibles debilidades en las aplicaciones o servicios.

Mi objetivo final en esta fase es comprender la arquitectura del sistema de monitoreo, los tipos de sensores utilizados, los rangos de valores normales, y cualquier posible punto débil que pueda explotar en las siguientes etapas.


## 2.Weaponization.

Con la información recopilada, ahora toca el **Armamento (T1588)**. Basándome en los posibles protocolos de comunicación identificados (quizás Modbus, SNMP u otro protocolo industrial), crearé una **Carga Útil Maliciosa (T1588.006)** que me permita enviar comandos alterados a la unidad central de monitoreo. Esta carga útil estará diseñada específicamente para inyectar valores de humedad, temperatura y corriente de fuga que se encuentren dentro de los rangos de operación "normales" pero que sean significativamente diferentes de los valores reales.

Para la **Entrega (TA0003)** posterior, prepararé un **Paquete Malicioso (T1588.003)**. Si identifiqué una vulnerabilidad en algún servicio web o aplicación expuesta durante el reconocimiento, podría empaquetar mi carga útil dentro de un exploit para esa vulnerabilidad. Sin embargo, considerando la naturaleza sensible de la subestación, una **Ingeniería Social (TA0011)** podría ser un vector de entrega más sutil y efectivo a largo plazo.

Por ahora, me concentraré en refinar mi carga útil maliciosa para asegurar que los datos falsificados sean creíbles y no activen alarmas inmediatas basadas en valores fuera de rango obvios.

## 3.Delivery.

Mi siguiente paso es la **Entrega (T1091)**. Aunque una explotación directa a través de una vulnerabilidad web (si la hubiera) sería rápida, la **Ingeniería Social (T1534)** me ofrece una mayor probabilidad de éxito a largo plazo sin generar alertas tempranas.

Crearé un **Correo Electrónico de Phishing (T1534.001)** dirigido a personal de mantenimiento o supervisores de la subestación. El correo electrónico contendrá un **Archivo Adjunto Malicioso (T1566.001)** o un **Enlace Malicioso (T1566.002)** disfrazado de un documento técnico urgente, una actualización de software legítima o una alerta de seguridad falsa.

El archivo adjunto o el enlace estarán diseñados para establecer una conexión inicial dentro de la red de la subestación. En lugar de un exploit directo en esta etapa, mi objetivo principal es establecer un **Acceso Inicial (TA0001)** sigiloso para futuras acciones. Por lo tanto, el archivo adjunto podría contener un payload que establezca una **Conexión Remota a través de un Servicio Legítimo (T1090)** como una conexión reverse shell encubierta utilizando un protocolo común (como HTTPS) para evitar la detección por firewalls básicos.


## 4.Exploitation

Una vez que mi payload inicial se ejecute, habré logrado el **Acceso Inicial (T1078)**. Ahora, necesito **Escalar Privilegios (TA0004)** si aterricé con una cuenta de usuario limitada. Utilizaré técnicas como el **Abuso de Tokens (T1134)** o la **Explotación de Vulnerabilidades Locales (T1068)** en el sistema comprometido para obtener credenciales con mayores permisos.

Una vez con privilegios suficientes, mi objetivo es identificar el sistema que gestiona directamente la comunicación con los sensores de humedad, temperatura y corriente de fuga. Esto podría ser un servidor SCADA, una unidad de control local o incluso una aplicación de escritorio específica.

Para interactuar con este sistema, podría intentar **Inyección de Código (T1505)** si encuentro alguna interfaz o aplicación vulnerable que permita la entrada de datos. Sin embargo, mi enfoque principal será **Validar Entrada Inadecuada (T1210)** en la comunicación con los sensores. Si el sistema no valida correctamente los rangos de los datos recibidos (a pesar de estar dentro de los límites "normales"), podré inyectar mis valores alterados directamente en el flujo de datos.

## 5.Installation

Para asegurar mi persistencia y control a largo plazo, necesito **Establecer Persistencia (TA0003)**. Utilizaré técnicas como la creación de **Nuevas Cuentas de Usuario (T1136)** con privilegios administrativos o la modificación de **Tareas Programadas/Trabajos (T1053)** para ejecutar mi payload malicioso periódicamente.

También podría instalar un **Backdoor (T1505)** en el sistema comprometido, como un servicio remoto encubierto o la modificación de servicios legítimos para permitir el acceso no autorizado. Esto me permitirá retomar el control incluso si mi conexión inicial se pierde.

Además, para facilitar mi **Comando y Control (TA0011)** en la siguiente etapa, configuraré un canal encubierto y persistente. Esto podría implicar la modificación de la configuración de firewall local para permitir conexiones desde mi servidor C2 o el uso de **Túneles (T1572)** a través de protocolos legítimos.


## 6.Command & Control

Con mi backdoor instalado y un canal de comunicación establecido, ahora puedo ejercer **Comando y Control (T1041)**. Utilizaré el canal encubierto que configuré para comunicarme con el sistema comprometido desde mi servidor **Servidor de Comando y Control (C2) (T1041)**.

A través de este canal, enviaré los comandos específicos para inyectar los datos alterados al sistema de monitoreo. Utilizaré el protocolo de comunicación nativo del sistema de monitoreo (identificado durante el reconocimiento) para enviar paquetes de datos falsificados que representen valores de humedad, temperatura y corriente de fuga dentro de los rangos aceptables pero que indiquen condiciones seguras cuando en realidad no lo son.

Para evitar la detección, utilizaré **Ofuscación de Datos (T1001)** en mi comunicación C2, disfrazando los comandos como tráfico legítimo. También podría utilizar **Canales de Control Alternativos (T1573)** como el uso de DNS o ICMP para la comunicación en caso de que el canal principal sea bloqueado.

## 7.Actions on Objectives

Finalmente, he llegado a la fase de **Acciones en los Objetivos (T1565)**. Mi objetivo principal es **Manipulación de Datos (T1565.001)** del sistema de monitoreo. Continuaré enviando los datos falsificados de humedad, temperatura y corriente de fuga para que el personal de la subestación crea que las condiciones son normales y seguras.

Como resultado de esta manipulación, el **Lavado de Activos Industriales (T0883)** programado de la subestación no se realizará debido a la falsa sensación de seguridad. Con el tiempo, la acumulación de contaminantes en los aisladores de alta tensión, combinada con condiciones ambientales reales (alta humedad, niebla salina si aplica), aumentará la corriente de fuga hasta alcanzar un punto crítico.

Esta desatención de la carga, causada directamente por la información manipulada en el sistema de monitoreo, conducirá a un **Flashover (T0884)** en los equipos de alta tensión, provocando una falla significativa, interrupción del suministro eléctrico y las consecuentes **Pérdidas de Disponibilidad (T1753)** y **Pérdidas Financieras (T1753.001)** millonarias para la empresa por la interrupción de la carga y los daños a los equipos. Mi objetivo final de sabotaje se habrá cumplido.





  


  

