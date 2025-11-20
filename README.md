# Configuración de Proxy Squid en Ubuntu

Como configurar un proxy Squid en Ubuntu, cubriendo instalación, configuración básica, caché, autenticación y reglas de acceso (ACLs)

### 1. Instalación y Copia de Seguridad de Squid

Instalaremos Squid y haremos una copia de seguridad del archivo de configuración original para trabajar con uno simplificado

```bash
sudo apt update
sudo apt install squid apache2-utils curl git -y

# Detenemos el servicio para configurar
sudo systemctl stop squid

# Hacemos backup del original
sudo mv /etc/squid/squid.conf /etc/squid/squid.conf.backup

# Clonamos el repositorio
git clone https://github.com/rhythmcreative/squid.git

# Entramos al archivo y movemos el squid
cd squid && mv squid.conf /etc/squid/squid.conf

cat /etc/squid/squid.conf

```

### 2. El archivo `squid.conf` para la Demo

Copia y pega esta configuración en tu nuevo archivo `/etc/squid/squid.conf`. Está simplificada para facilitar la explicación de cada sección.

```conf
# --- PUERTO (Punto 1 de la rúbrica) ---
# Puerto estándar. 8080 también es común, pero 3128 es el default de Squid.
http_port 3128

# --- CACHÉ (Punto 2 de la rúbrica) ---
# Usar 256 MB de RAM para objetos en tránsito
cache_mem 256 MB
# Objetos mayores a 4MB no se guardan en RAM
maximum_object_size_in_memory 4096 KB
# Guardar caché en disco: Tipo ufs, ruta, 1GB tamaño, 16 carpetas nivel 1, 256 nivel 2
cache_dir ufs /var/spool/squid 1024 16 256
# Política de reemplazo (opcional explicar)
cache_replacement_policy heap LFUDA

# --- AUTENTICACIÓN (Punto 4 de la rúbrica - Sobresaliente) ---
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Mi Proxy Seguro
auth_param basic credentialsttl 2 hours
acl autenticados proxy_auth REQUIRED

# --- ACLs (Punto 3 de la rúbrica) ---
acl red_local src 192.168.1.0/24  # <--- ¡AJUSTAR LA IP a tu red!
acl sitios_bloqueados dstdomain .facebook.com .tiktok.com
acl horarios_trabajo time MTWHF 08:00-18:00

# --- REGLAS DE ACCESO (Orden importa: Primero bloquear, luego permitir) ---
# 1. Bloquear sitios prohibidos
http_access deny sitios_bloqueados
# 2. Permitir solo usuarios con contraseña
http_access allow autenticados
# 3. Permitir red local (si no usas auth, descomenta esto y comenta la anterior)
# http_access allow red_local
# 4. Denegar todo lo demás (Seguridad por defecto - Punto 7 de la rúbrica)
http_access deny all
```

**Nota Importante:** Ajustar `acl red_local src 192.168.1.0/24` a la dirección IP de tu red local que tenga la maquina

Aqui para crear un usuario de autenticacion (1234)
```conf
sudo htpasswd -c /etc/squid/passwd alumno
```
 Esto crea las carpetas definidas en cache_dir
 ```conf
sudo squid -z
```
Luego iniciamos y probamos
```conf
sudo systemctl start squid
sudo systemctl status squid
```
Para poder ver los logs en tiempo real
```conf
tail -f /var/log/squid/access.log
```
## Prueba en tiempo real
```conf
export http_proxy="http://alumno:1234@127.0.0.1:3128"
```
## Para hacer comprobacion de que la pagina se ha bloqueado
```
curl -I https://www.facebook.com
```
Esto deberia de dar forbiden
```
curl -I https://www.wikipedia.com
```
Y esta si que deberia funcionar sin problemas

## Puntos Clave para la Presentación (5 minutos)

Durante tu presentación, enfócate en explicar cada sección de `squid.conf` de manera concisa:

*   **`http_port`**: El puerto por donde Squid escucha conexiones.
*   **`cache_mem` / `cache_dir`**: Cómo Squid gestiona la caché en RAM y en disco para acelerar la navegación.
*   **Autenticación (`auth_param`, `acl autenticados`)**: Explicar la seguridad mediante usuario y contraseña.
*   **ACLs (`acl red_local`, `acl sitios_bloqueados`, `acl horarios_trabajo`)**: Cómo definir criterios (redes, dominios, horarios) para las reglas.
*   **`http_access`**: La importancia del orden de las reglas para permitir o denegar el acceso. La última línea `http_access deny all` es crucial para la seguridad.

¡Buena suerte con tu presentación!
