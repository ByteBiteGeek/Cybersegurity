# Trust.
- Dificultad facil.

## Escaneo con nmap.
<p>Realizamos escaneo con nmap para observar que puertos tenemos abiertos, servicio y versiones</p>

```
nmap -sCV -sS -n -Pn -p- [IP]

```
## Fuzzing
<p>Una vez que encontremos los puerto procederemos a utilizar la herramienta "gobuster" cuyo objetivo es realizar ataque de fuerza bruta
sobre el servidor, clonandolo desde git</p>

```
git clone https://github.com/OJ/gobuster.git

```

<p>Ahora debemos copilar el codigo para crear un ejecutable de gobuster debemos estar dentro del directorio de "gobuster" y luego
y luego copilar el codigo.</p>

```
go build

```

### En caso de que no el comando "go" no exista se debe instalar go que es el lenguaje de programacion en el cual esta escrito gobuster.

```
sudo apt update
sudo apt install golang

```

<p>Y ahora hacer falta un diccionario para utilizar, en este caso usaré el de "Seclist"</p>

```
git clone https://github.com/danielmiessler/SecLists.git

```

<p>Y posteriormente ejecutar el comando desde dentro del directorio gobuster (tener en cuenta que tambien tengo go instalado allí)</p>

```
./gobuster dir -u [IP] -w SecLists/Discovery/Web-Content/common.txt -x .php,.sh,.py,.txt

```

<p>Finalmente obtenemos lo siguiente:</p>

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.18.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd.php        (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/.hta                 (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/.htaccess.php        (Status: 403) [Size: 275]
/.hta.php             (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]
/server-status        (Status: 403) [Size: 275]
Progress: 9478 / 9478 (100.00%)
===============================================================
Finished
===============================================================


```
