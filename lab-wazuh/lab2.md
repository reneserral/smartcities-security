# Laboratorio 2: Wazuh

## Preparación de la VM

Para realizar esta práctica se usará PowerShell si estamos en Windows o `bash` o similar en Linux o MacOS.

Primero creamos un directorio para la práctica:
```
mkdir lab2-wazuh
cd lab2-wazuh
```

A continuación descargamos la última versión del Vagrantfile del repositorio:
```
wget https://raw.githubusercontent.com/reneserral/smartcities-security/main/lab-certificados/Vagrantfile
```

Ahora ya podemos arrancar la VM:
```
vagrant up wazuh
```

Este proceso puede llegar a tardar 20 minutos y requiere 4GB de memoria RAM disponible en el equipo.

## Navegando por Wazuh
Si todo ha ido bien debería estar el sistema funcionando.

**Nota**: dependiendo del equipo si intentamos acceder a la web directamente puede ser que todavía no esté en funcionamiento, en este caso conviene esperar unos segundos.

Para acceder abrimos un navegador a: https://192.168.56.12/

El login y contraseña por defecto son:
```
login: kibanaserver
password: kibanaserver
```

### Añadiendo un agente
Para instalar un agente iniciamos otra máquina virtual, puede ser la del laboratorio 1:
```
vagrant up apache
```

Para instalar un agente solo es necesario ejecutar los siguientes comandos:
```
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo su
WAZUH_MANAGER="192.168.56.12" apt install wazuh-agent
exit
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Una vez instalado si refrescamos el navegador deberíamos ver el agente instalado. Ahora podemos navegar por las páginas para ver las opciones y la información que se puede obtener.

### 5.1.  Modificación de ficheros
En este ejemplo vamos a configurar el monitor de ficheros para que reporte cambios en dichos archivos. Vamos a configurar que la comprobación de integridad se realice cada 120 segundos (por defecto son cada 12h – 43200 segundos). Para ello solo hace falta conectarse a la máquina wazuh:
```
vagrant ssh wazuh
```

Y editar como root la configuración de ossec:
```
sudo nano /var/ossec/etc/ossec.conf
```

Ir a la sección de syscheck, allá habilitamos el servicio y cambiamos la frecuencia a 120
```
<disabled>no</disabled>
<frequency>43200</frequency>
Otro punto importante es habilitar la sincronización, en la misma sección syscheck:
<synchronization>
  <enabled>yes</enabled>
  <interval>5m</interval>
  <max_interval>1h</max_interval>
  <max_eps>10</max_eps>
</synchronization>
```

Nos fijamos también en algunos directorios que se monitorizan: `/etc`, `/usr/bin`...

Ahora guardamos los cambios y reiniciamos el servicio:
```
sudo systemctl restart wazuh-manager
```

Comprobamos que está activo:
```
systemctl status wazuh-manager
● wazuh-manager.service - Wazuh manager
   Loaded: loaded (/usr/lib/systemd/system/wazuh-manager.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-09-22 08:47:45 UTC; 24s ago
  Process: 4424 ExecStop=/usr/bin/env ${DIRECTORY}/bin/ossec-control stop (code=exited, status=0/SUCCESS)
  Process: 4671 ExecStart=/usr/bin/env ${DIRECTORY}/bin/ossec-control start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/wazuh-manager.service
           ├─4848 /var/ossec/framework/python/bin/python3 /var/ossec/api/scripts/wazuh-apid.py
           ├─4890 /var/ossec/bin/ossec-authd
           ├─4966 /var/ossec/bin/wazuh-db
           ├─5051 /var/ossec/bin/ossec-execd
           ├─5067 /var/ossec/bin/ossec-analysisd
           ├─5171 /var/ossec/bin/ossec-syscheckd
           ├─5251 /var/ossec/bin/ossec-remoted
           ├─5345 /var/ossec/bin/ossec-logcollector
           ├─5425 /var/ossec/bin/ossec-monitord
           └─5446 /var/ossec/bin/wazuh-modulesd
```
Ahora vamos a `apache` y creamos/modificamos algunos ficheros en `/etc`, por ejemplo creando un usuario con adduser, esperamos unos minutos y debería mostrarse en la UI rápidamente.

