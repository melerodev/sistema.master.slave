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
![alt text](/img/image.png)</br>
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
`sudo systemctl status named`

Si todo ha salido bien nos tendrá que devolver algo así:

![alt text](/img/image1.png)

Ahora lo que haremos será poner la MV de `tierra` como maestro con el nombre `tierra.sistema.test` y tendrá autoridad sobre la zona directa e inversa. Para ello uilizaremos el comando abriremos el archivo de configuración local de Bind9 con el comando `sudo nano /etc/bind/named.conf.local` y lo dejaremos de la siguiente forma: 
```
// Zona directa
zone "sistema.test" {
    type master;                        // Es el servidor maestro
    file "/etc/bind/db.sistema.test";   // Archivo de configuración de la zona directa
};

// Zona inversa
zone "57.168.192.in-addr.arpa" {
    type master;                        // Es el servidor maestro
    file "/etc/bind/db.192.168.57";     // Archivo de configuración de la zona inversa
};
```

Ahora lo que haremos será crear el archivo de zona directa con los comandos:
`sudo cp /etc/bind/db.local /etc/bind/db.sistema.test`
`sudo nano /etc/bind/db.sistema.test`

Y dentro de este pondremos el siguiente bloque de código:
```
$TTL    604800
@       IN      SOA     tierra.sistema.test. admin.sistema.test. (
                          2024111801 ; Serial
                          604800     ; Refresh
                          86400      ; Retry
                          2419200    ; Expire
                          604800 )   ; Negative Cache TTL
;
@       IN      NS      tierra.sistema.test.
@       IN      NS      venus.sistema.test.
tierra  IN      A       192.168.57.103
venus   IN      A       192.168.57.102
marte   IN      A       192.168.57.104
mail    IN      CNAME   marte
ns1     IN      CNAME   tierra
ns2     IN      CNAME   venus
```

Ahora crearemos el archivo de zona inversa con los comandos:
`sudo cp /etc/bind/db.127 /etc/bind/db.192.168.57`
`sudo nano /etc/bind/db.192.168.57`

Y dentro de este pondremos el contenido:
```
$TTL    604800
@       IN      SOA     tierra.sistema.test. admin.sistema.test. (
                          2024111801 ; Serial
                          604800     ; Refresh
                          86400      ; Retry
                          2419200    ; Expire
                          604800 )   ; Negative Cache TTL
;
@       IN      NS      tierra.sistema.test.
@       IN      NS      venus.sistema.test.
103     IN      PTR     tierra.sistema.test.
102     IN      PTR     venus.sistema.test.
104     IN      PTR     marte.sistema.test.
```

Ahora nos aseguraremos de que los archivos tinen los permisos correctos con el comando `sudo chown bind:bind /etc/bind/db.sistema.test /etc/bind/db.192.168.57`.

Después de eso lo que haremos será comprobar que los archivos estén todos "OK", con los comandos.
`sudo named-checkzone sistema.test /etc/bind/db.sistema.test`
`sudo named-checkzone 57.168.192.in-addr.arpa /etc/bind/db.192.168.57`

Tras recibir una respuesta de este estílo, significará que todo está bien:
```
sudo named-checkzone 57.168.192.in-addr.arpa /etc/bind/db.192.168.57
zone sistema.test/IN: loaded serial 2024111801
OK
zone 57.168.192.in-addr.arpa/IN: loaded serial 2024111801
OK
```

Ahora lo que haremos será reiniciar Bind9 y comprobar su estado.
`sudo systemctl restart bind9`
`sudo systemctl status bind9`
