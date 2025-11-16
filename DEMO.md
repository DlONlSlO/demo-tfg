# DEMO
### Markdown del proceso de instalación de Snort en un equipo reciclado con sistema operativo Debian 12.5 (Bookworm)

# Instalación y Configuración de Snort en Debian

<p align="center">
  <img src="https://www.snort.org/assets/Snort_fulllogo.png" width="240" valign="middle" style="margin-right: 20px;">
  <img src="https://www.debian.org/logos/openlogo-nd.svg" width="150" valign="middle">
</p>

A continuación, se escriben los pasos para instalar el Sistema de Detección de Intrusos (IDS) **Snort** en **Debian 12.5 (Bookworm)** y configurar una regla para detectar detectar paquetes ICMP (ping).



## 1. Requisitos Previos

### Paso 1.1: Actualizar el Sistema

Actualizar el índice de paquetes, para maximizar la compatibilidad de las librerías del sistema con Snort.

``` 
sudo apt update
```
Instalar la actualizaciones disponibles para los paquetes ya instalados.
```
sudo apt upgrade -y 
```

### Paso 1.2: Instalar Herramientas Base de Compilación

Snort se compila desde código, por lo que se requieren herramientas de desarrollo.

``` 
sudo apt install -y build-essential bison flex pkg-config
```
>**build-essential**: Instala gcc, g++, make (el compilador y herramientas base).

>**bison**: Necesario para generar analizadores sintácticos.

>**flex**: Generador de analizadores léxicos. Complementa a Bison.

>**pkg-config**: Permite al compilador encontrar librerías instaladas.

### Paso 1.3: Instalar Dependencias Necesarias

Estas bibliotecas permiten compilar los módulos de captura, análisis y procesamiento.

``` 
sudo apt install -y libpcap-dev libpcre3-dev libdumbnet-dev zlib1g-dev liblzma-dev libssl-dev libnghttp2-dev libtirpc-dev
```
>**libpcap-dev** Header files para capturar paquetes.

>**libpcre3-dev** Header files para expresiones regulares utilizadas por Snort.

>**libdumbnet-dev** Manipulación de paquetes IP/TCP/UDP.

>**zlib1g-dev** Necesaria para compilar soporte de compresión.

>**liblzma-dev** Compresión LZMA/XZ, usada por el decodificador.

>**libssl-dev** Header files de OpenSSL necesarios para TLS/SSL.

>**libnghttp2-dev** Soporte HTTP/2 (requiere headers).

>**libtirpc-dev** Header files RPC modernos requeridos por LibDAQ.

### Paso 1.4: Instalar LibDAQ (Data Acquisition Library)

Snort no puede capturar paquetes directamente, utiliza para ello una librería llamada LibDAQ que actúa como capa de abstracción entre el IDS y las interfaces de red reales.

Para descargarlo utilicé:

```
cd /tmp
wget https://www.snort.org/downloads/snort/daq-2.0.7.tar.gz
tar -xvzf daq-2.0.7.tar.gz
cd daq-2.0.7/
```

Utilizando un directorio temporal `tmp` para descargar los archivos `tar` de los repositorios oficiales de Snort y luego extraerlos allí, para proceder a compilarlos.

```
./configure
```
Utilizando `./configure` que es un script, que se encarga de chequear las dependencias, las rutas de las librerías compiladores y de generar los archivos para finalmente compilar utilizando `make`.
```
make
```
Utiliza las instrucciones generadas por `./configure` para compilar finalmente todos los módulos de LibDAQ.
Una vez compilado se instalan los binarios y librerías resultantes en las rutas del sistema con:

```
sudo make install
```

## 2. Instalar Snort


Al igual que LibDAQ, para descargar el código fuente empleé los repositorios oficiales de Snort:

```
cd /tmp
wget https://www.snort.org/downloads/snort/snort-2.9.20.tar.gz
tar -xvzf snort-2.9.20.tar.gz
cd snort-2.9.20/
```

Aplicando el mismo procedimiento de instalación:

```
./configure
```
Con la salvedad de que antes de compilar, se indican variables de entorno, para maximizar la compatibilidad de la versión 2.9.20 de Snort con Debian 12.5.
```
export CFLAGS="-I/usr/include/tirpc"
export CPPFLAGS="-I/usr/include/tirpc"
export LDFLAGS="-ltirpc"
```
>**CFLAGS** Define opciones para el compilador C y agrega la ruta donde están los headers.

>**CPPFLAGS** Es similar a CFLAGS pero aplicado al preprocesador de C.

>**LDFLAGS** Se utiliza durante el proceso de linking.

> [!WARNING] 
> Este paso corrige la compatibilidad y permite continuar sin errores en la compilación.

Ahora si pude compilar normalmente con:
```
make
sudo make install
```
> [!TIP]
> Opcional verificar la instalación de Snort con: 
> ```
> sudo ldconfig
> snort -V
> ```
> **sudo ldconfig** reconstruye el cache y registra todas las librerías recién instaladas.
> **snort -V** muestra la versión y más datos, indicando una correcta compilación.



## 3. Configurar Snort

### Paso 3.1: Crear la Estructura de Directorios y Archivos para Snort

Snort requiere una serie de directorios para almacenar configuración, reglas, logs, listas blancas, listas negras y módulos dinámicos

```
sudo mkdir -p /etc/snort
sudo mkdir -p /etc/snort/rules
sudo mkdir -p /etc/snort/so_rules
sudo mkdir -p /etc/snort/preproc_rules
sudo mkdir -p /var/log/snort
sudo mkdir -p /usr/local/lib/snort_dynamicrules
```

Y también requiere archivos por defecto aunque se encuentren vacíos, sin ellos podría presentar errores en su ejecución.

```
sudo touch /etc/snort/rules/white_list.rules
sudo touch /etc/snort/rules/black_list.rules
sudo touch /etc/snort/rules/local.rules
```
Además, se copian estos archivos de configuración, desde el código fuente:
```
sudo cp *.conf* /etc/snort/
sudo cp *.map /etc/snort/
```



### Paso 3.2: Editar el Archivo Principal de Configuración de Snort

Para ello con el editor `nano` se edita `snort.conf`, este archivo controla todo rutas, variables de red, preprocesadores, reglas, decodificadores, la red que se quiere proteger, la red externa, las listas blanca y negra, etc.
```
sudo nano /etc/snort/snort.conf
```
Dentro del archivo modifiqué las líneas:

```
ipvar HOME_NET 0.0.0.0/00  
ipvar EXTERNAL_NET !$HOME_NET
var RULE_PATH /etc/snort/rules
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules
var WHITE_LIST_PATH /etc/snort/rules
var BLACK_LIST_PATH /etc/snort/rules
```
>**ipvar HOME_NET 0.0.0.0/00**: La red que se quiere proteger.

>**ipvar EXTERNAL_NET !$HOME_NET**: Indicar que todo lo que no es HOME_NET, es red externa.

>**var RULE_PATH /etc/snort/rules; var SO_RULE_PATH /etc/snort/so_rules y var PREPROC_RULE_PATH /etc/snort/preproc_rules**: Reemplacé por las rutas donde están cada una de estas reglas (ver 3.1).

>**var WHITE_LIST_PATH /etc/snort/rules y var BLACK_LIST_PATH /etc/snort/rules**: Reemplacé por las rutas donde están estas listas (ver 3.1).

> [!IMPORTANT] 
> Guardar en `nano` con `Ctrl + O`, presionar `Enter` y luego salir con `Ctrl + X`.


## 4. Configurar Reglas


### Paso 4.1: Crear Reglas
Para ello con el editor `nano` se edita `local.rules`, es el archivo destinado a reglas creadas por el administrador, es decir reglas que no provienen de los repositorios oficiales.
```
sudo nano /etc/snort/rules/local.rules
```
Dentro del archivo agregué la línea:

```
alert icmp any any -> $HOME_NET any (msg:"PING DETECTADO"; sid:1000001; rev:1;)
```
Explicación de la regla:

>**alert** Es la acción, Snort generará una alerta cuando la condición se cumpla.

>**icmp** Es el protocolo, en este caso es el utilizado por ping, traceroute, etc.

>**any** Cualquier IP de origen.

>**any** Es el puerto aunque icmp no utiliza se escribe por sintaxis.

>**$HOME_NET** Red protegida definida en `snort.conf`.

>**any** El puerto nuevamente por sintaxis.

>**msg:"PING DETECTADO"** Es el mensaje que muestra la alerta.

>**sid:1000001** Identificador de la regla (Snort ID).

>**rev:1** Revisión de la regla.

> [!NOTE] 
> Los SID de las reglas personalizadas deben comenzar desde 1,000,000.

Una vez creada la regla cerrar el editor, siguiendo los pasos de la última línea de 3.2

### Paso 4.2: Comentar Reglas Oficiales
Para evitar conflictos es necesario deshabilitar reglas que vienen por defecto en Snort, algunas de ellas desactualizadas, para esto nuevamente edité el archivo `snort.conf`, comentando todas las reglas que no fueran las personalizadas (`include $RULE_PATH/local.rules`), poniendo delante de cada linea el caracter `#` , como por ejemplo en las siguientes:
```
#include $RULE_PATH/app-detect.rules
#include $RULE_PATH/attack-responses.rules
#include $RULE_PATH/backdoor.rules
#include $RULE_PATH/bad-traffic.rules
...
```

Y nuevamente cerré el editor, siguiendo los pasos de la última línea de 3.2

## 5. Prueba

Por último ejecuté Snort para verificar el funcionamiento:

```
sudo snort -A console -q -c /etc/snort/snort.conf -i enp5s0
```
>**sudo** Privilegios necesarios para que Snort pueda capturar los paquetes.

>**Snort** Ejecuta el IDS.

>**-A console** Muestra las alertas directamente en la terminal.

>**-q** Modo silencioso, no muestra información innecesaria en la consola.

>**-c /etc/snort/snort.conf** El archivo de configuración que se va a utilizar.

>**-i enp5s0** Interfaz de red que Snort va a monitorear.

Al momento de hacer un ping desde otra computadora, Snort mostró el mensaje en consola "PING DETECTADO", junto con la IP origen, confirmando el resultado esperado de esta prueba, Snort está monitoreando correctamente la interfaz Ethernet `enp5s0`; la regla personalizada en `local.rules` está siendo cargada; el IDS detecta paquetes ICMP según lo configurado; el tráfico coincide con la condición de la regla y las alertas se muestran correctamente en consola.


Autor: Jorge Dionisio Velazquez
Fecha: 16/11/2025
