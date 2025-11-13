# DEMO
### Es un Markdown del proceso de configuración de Snort dentro de Debian 12.5 (Bookworm)

# Instalación y Configuración de Snort en Debian

A continuación se escriben los pasos para instalar el Sistema de Detección de Intrusos (IDS) **Snort** en **Debian 12.5 (Bookworm)** y configurar una regla para detectar escaneos de puertos.



## 1. Requisitos Previos

Acceso con privilegios de `sudo` o como usuario `root`.



## 2. Instalación de Snort

### Paso 2.1: Actualizar el Sistema

Actualizar la lista de paquetes:

``` 
sudo apt update
sudo apt upgrade -y 
```

### Paso 2.2: Instalar Snort

Instalar Snort y las dependencias necesarias:

``` 
sudo apt install snort -y
```

>[!NOTE]
>Durante la instalación va a pedir que se especifique la interfaz de red que se va a monitorear. (para el caso en la maquina reciclada para este proyecto es: `eth0`)

## 3. Configuración de Regla de Alerta (Escaneo de Puertos)

### Paso 3.1: Definir la Red

Editar el archivo de configuración principal de Snort para definir la Red (`$HOME_NET`).
```
sudo nano /etc/snort/snort.conf
```
Buscar y cambiar el valor de  `$HOME_NET`, por el de la Red. Por ejemplo 192.168.1.0/24:

```
var HOME_NET 192.168.1.0/24 
var EXTERNAL_NET !$HOME_NET
```

### Paso 3.2: Crear e Incluir la Regla Personalizada

- Crear el archivo de reglas
```
sudo nano /etc/snort/rules/local.rules
```
- Añadir la regla de escaneo de puertos
```
alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ALERTA - Posible Escaneo de Puertos (SYN)"; flow:to_server,established; flags:S,12; threshold:type limit, track by_src, count 10, seconds 60; classtype:attempted-recon; sid:1000001; rev:1;)
```
- Incluir el archivo de Reglas en `snort.conf`:
Editar el archivo principal nuevamente (`/etc/snort/snort.conf`) y añadir al final:
```
include $RULE_PATH/local.rules
```

### Paso 3.3: Ejecutar Snort en modo IDS

Para que escuche a través de la interfaz de red que configuré

```
sudo snort -A full -q -c /etc/snort/snort.conf -i eth0
```
