# Servidor en Raspberry PI

Estos son los pasos para configurar una Raspberry PI como servidor local de tal manera que se puedan probar aplicaciones y flujos de trabajo en un ambiente enteramente local que se parece al flujo final usando un servidor VPS que tiene instalado Ubuntu y NGINX.

## Partes

- Raspberry PI 3 B+.
- Micro SD 16 Gb Sandisk Clase 10.

## Instalación

Desde un computador (distinto a la Raspberry), instalar [Raspberry Pi Imager](https://www.raspberrypi.org/software/) para instalar en la SD el sistema operativo. Conectar la SD al computador y seguir los pasos en **Raspberry Pi Imager** para instalar el OS:

[Ubuntu Server 20.04.2 LTS 32-bit](https://ubuntu.com/download/raspberry-pi)

Al terminar el proceso de instalación en la SD, conectarla a la Raspberry PI y prenderla.

En la primera iniciada se va a detener en `ubuntu login: _` pero hay que esperar un momento y se va a iniciar otro proceso de configuración automáticamente llamado [Cloud-init](https://help.ubuntu.com/community/CloudInit#:~:text=cloud%2Dinit%20is%20the%20Ubuntu,Ubuntu%20images%20available%20on%20EC2.). Hay que esperar unos minutos a que corra ese proceso.

Al principio pensé que se quedaba estancado luego de que salía `Cloud-init v. 20.4.1-Oubuntu1~20.04.1 finished at ...` pero al presionar Enter en el teclado pasó al `Ubuntu login:`.

Las credenciales iniciales son:

```md
usuario; ubunbtu
password: ubuntu
```

Al poner estas credenciales nos va a pedir que cambiemos la clave.

## WIFI

Siguiendo [este tutorial](https://linuxconfig.org/ubuntu-20-04-connect-to-wifi-from-command-line), esta es la configuración que me sirve:

```sh
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
    wifis:
        wlan0:
            optional: true
            access-points:
                "SSID-NAME-HERE":
                    password: "PASSWORD-HERE"
            dhcp4: true
```

Luego de configurar el WIFI, Ubuntu va a realizar una serie de actualizaciones e instalaciones con `APT`. Esto toma un tiempo y si queremos instalar cosas nos va a dar un error parecido a:

```sh
Waiting for cache lock: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 3539 (unattended-upgr)
```

Hay que esperar!

Podemos revisar que processos estan usando `APT` con el comando:

```sh
ps aux | grep -i apt
```

## Conexión remota SSH

EL `Cloud-init` se encargó de instalar ssh entonces podemos conectarnos al servidor desde otro computador. Para comprobar que el `ssh` este activado podemos correr el comando:

```sh
sudo systemctl status ssh
```

Y debe decir en parte del log `active (runing)`.

En la Raspberry, ver el IP de la placa con:

```sh
ip a
```

En este caso esta conectado al wifi con `wlan0` (que configuramos antes en la sección WIFI) y el ip debe estar en la sección `inet 192.168.0.9` - Este numero es diferente en cada red.

Desde otro computador, usar SSH para conectarse a la Raspberry.

```sh
ssh ubuntu@192.168.0.9
```

## Configuración NGINX

Instalación

```sh
sudo apt install nginx
```

Iniciar el servidor

```sh
sudo /etc/init.d/nginx start
```

Esto debería poner una página web en la IP de la Raspberry, entonces si vamos al explorador a la url: `http://192.168.0.9/` debería salir una página que dice "Welcome to nginx!".

## Instalar Docker

Actualizar el sistema:

```sh
sudo apt-get update && sudo apt-get upgrade
```

Instalar Docker y Docker Compose:

```sh
sudo apt install docker.io docker-compose
```

Iniciar Docker y activar que se inicie automáticamente al reiniciar la Raspberry.

```sh
sudo systemctl enable --now docker
```

Comprobar instalación

```sh
sudo docker version
```

```sh
sudo docker-compose version
```

## Conexión entre computador y servidor

[Crear una conexión sin clave](https://help.dreamhost.com/hc/en-us/articles/216499537-How-to-configure-passwordless-login-in-Mac-OS-X-and-Linux)

```sh
ssh-keygen -t rsa -b 4096
```

```sh
sh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@192.168.0.9
```

## Contexto

Con las opciones de contexto en docker podemos correr comandos desde nuestro computador en el servidor.

```sh
docker context create rpi --docker host=ssh://ubuntu@192.168.0.9
```

Podemos comprobar la conexión viendo la versión en el servidor:

```sh
docker --context rpi version
```

## Iniciar contenedor en servidor

```sh
docker-compose --context rpi up -d
```