# sistema.master.slave
<hr style="color: white; border-radius: 5px;">
Dentro del directorio `.vagrant` crearemos un `Vagrantfile` en que el crearemos dos MV:
-   Máquina maestro: `tierra.sistema.test` (Debian texto, IP: `192.168.57.103`).
-   Máquina esclavo: `venus.sistema.test` (Debian texto, IP: `192.168.57.102`).

No se deberá tener bajo control de versiones el directorio `.vagrant` ni los **ficheros de backup**.

Después ejecutaremos el comando `vagrant up` para crear, configurar e iniciar las MV. Tras esto mismo ejecutaremos el comando `vagrant ssh <nombre mv>`, que en este caso nos conectaremos a la máquina maestra `tierra`.

Ahora lo que haremos será activar solamente la **escucha del servidor para el protocolo IPv4**. Para ello nos conectaremos a la MV e instalaremos e instalaremos *Bind9*, que es un softaware de servidor DNS utilizado para resolver nombre de dominio y de entra todas sus funciones, hay una que nos permitirá hacer lo que nosotros queremos.

Antes de instalar cualquier programa en Linux, siempre es recomendable actualizar los paquetes mediante `sudo apt-get update`.

Para ello primero tendrémos que instalarlo mediante el comando `sudo apt-get install bind9 bind9utils bind9-doc` dentro la MV. Ahora modificaremos el archivo general de configuración `sudo nano /etc/default/named` y pondremos el valor `OPTIONS` que solo use IPv4 de esta forma:
`OPTIONS = "-u bind -4"`.

![alt text](/img/image.png)

Y guardaremos los cambios.

Ahora modificaremos el archivo `named.conf.options` utilizando el comando `sudo nano /etc/bind/named.conf.options` y pondremos `dnssec-validation <valor>;` en *yes* para activar la validación de DNS en .

Ahora permitiremos que los servidores permitan consultas recursivas sólo a los ordenadores en la red `127.0.0.0/8` y en la red `192.168.57.0/24`, para ello utilzaremos la opción de control de acceso o `acl` añadiendo las siguientes líneas de código al archivo de configuración:
```
acl "redes_permitidas" {
    127.0.0.0/8;        // Loopback (localhost)
    192.168.57.0/24;    // Red local específica
};
```

El archivo nos quedaría tal que así:
```
acl "redes_permitidas" {
    127.0.0.0/8;        // Loopback (localhost)
    192.168.57.0/24;    // Red local específica
};

options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation yes;

        listen-on-v6 { any; };
};
```

Ahora reiniciaremos el servidor y después comprobaremos su estado:
`sudo systemctl restart named`
`sudosystemctl status named`
