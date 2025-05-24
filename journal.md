# Journal
En este documento describo como voy avanzando en el desarrollo del proyecto.

## 2025-05-22
Elegí Fedora, empezaré por probar digiKam, que me pude una base de datos, lo que me llevó a instalarla en docker compose con portainer y ponerlo a competir con immich, creo que necesitaré ambos.

Primero toca insalar docker engine, no me sirvieron las instrucciones de ChatGPT, pero me sirvieron las instrucciones de la página de Docker: https://docs.docker.com/engine/install/fedora/

Seguí las instrucciones hasta habilitar Docker y correr el Hello World. Es super importante!!

Hay que crear el doirectorio:

    mkdir -p ~/portainer && cd ~/portainer

Hay que crear el Yaml para la configuración de Portainer

    nano docker-compose.yml

Le puse esto al documento

    version: '3.8'

    services:
    portainer:
        image: portainer/portainer-ce:latest
        container_name: portainer
        restart: always
        ports:
        - "8000:8000"
        - "9443:9443"   # HTTPS (interfaz web)
        - "9000:9000"   # HTTP (interfaz web legacy)
        volumes:
        - /var/run/docker.sock:/var/run/docker.sock
        - portainer_data:/data

    volumes:
    portainer_data:

Luego arranqué el servicio

    docker compose up -d

Si todo sale bien hay que entrar al siguiente url para hacer la primer configuración [http://localhost:9000](http://localhost:9000). Hay que hacerlo rápido, y guardar la contraseña porque se usará mucho.

Ahora estoy configurando MySQL. Al momento de definir el contenedor tengo lo siguiente 

    version: '3.8'

    services:
    mariadb:
        image: mariadb:10.11
        container_name: central-mariadb-local
        restart: unless-stopped
        environment:
        MYSQL_ROOT_PASSWORD: lumaLuma34
        ports:
        - "3307:3306"  # expone en el host el puerto 3307 para que no choque con otros
        volumes:
        - central_mariadb_data:/var/lib/mysql

    volumes:
    central_mariadb_data:


Luego entré al contenedor en la seccion containers, luego console y luego el comando siguiente:

    mysql -u root -p

Ahí adentro creo un usuario y contraseña y doy permisos

    CREATE DATABASE digikam CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE USER 'digikam_user'@'%' IDENTIFIED BY 'digipass';
    GRANT ALL PRIVILEGES ON digikam.* TO 'digikam_user'@'%';
    FLUSH PRIVILEGES;

Este usuario con permisos es necesario para que DigiKam Funcione, está en su documentación. La instalación pide nombre de base de datos de CoreDB, Thumbs, FaceDb y Similarity DB. Los cuales deben crearse previamente de forma manual.

Los creé con los suigientes comandos ya en la terminal del contenedor:

    CREATE DATABASE digikam CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE DATABASE digikam_thumbs CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE DATABASE digikam_faces CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE DATABASE digikam_similarity CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

    GRANT ALL PRIVILEGES ON digikam.* TO 'digikam_user'@'%';
    GRANT ALL PRIVILEGES ON digikam_thumbs.* TO 'digikam_user'@'%';
    GRANT ALL PRIVILEGES ON digikam_faces.* TO 'digikam_user'@'%';
    GRANT ALL PRIVILEGES ON digikam_similarity.* TO 'digikam_user'@'%';
    FLUSH PRIVILEGES;

En este punto me dijo que no tengo algunos drivers, así que hay que instalarlos y reniniciar:

    sudo dnf install qt5-qtbase-mysql
    sudo dnf install qt6-qtbase-mysql

Esto a continuación es lo que responde ChatGPT para comprobar que se creó todo de forma correcta

🧠 Qué necesita digiKam según la segunda imagen:

        Un usuario (digikam_user)

        Una contraseña (digipass)

        Una base de datos (digikam)

        Que ese usuario tenga todos los privilegios sobre esa base

✅ Paso 1: Entra a la consola del contenedor MariaDB desde Portainer

        Ve a Containers

        Haz clic en central-mariadb-local

        Ve a la pestaña Console

        Usa /bin/bash o /bin/sh → Connect

🛠️ Paso 2: Conéctate a MySQL como root

    mysql -u root -p

(Escribe la contraseña que pusiste como MYSQL_ROOT_PASSWORD en el stack, por ejemplo: supersegura123)

🧪 Paso 3: Comprueba el usuario y permisos. Verifica que el usuario exista:

    SELECT user, host FROM mysql.user;

Debes ver algo como:

    +---------------+------+
    | user          | host |
    +---------------+------+
    | digikam_user  | %    |
    | root          | %    |
    +---------------+------+
Verifica que tenga acceso a su base:

    SHOW GRANTS FOR 'digikam_user'@'%';

Debería mostrar:

    GRANT ALL PRIVILEGES ON `digikam`.* TO 'digikam_user'@'%'

🧱 Si no aparece o está incompleto:

Ejecuta:

    CREATE DATABASE IF NOT EXISTS digikam CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE USER IF NOT EXISTS 'digikam_user'@'%' IDENTIFIED BY 'digipass';
    GRANT ALL PRIVILEGES ON digikam.* TO 'digikam_user'@'%';
    FLUSH PRIVILEGES;

✅ Paso 4: Sal y prueba desde tu sistema Fedora

Ahora puedes probar desde terminal:

    mysql -h 127.0.0.1 -P 3307 -u digikam_user -p

Te pedirá la contraseña digipass
Luego, si entras, ejecuta:

    USE digikam;

Si no hay errores, la conexión desde digiKam debería funcionar también ✅

Para poder correr esto es necesario tener instalado el cliente de María DB en la terminal de Fedora

    sudo dnf install mariadb

Una vez instalado, es posible comprobar la instalación

    mysql -h 127.0.0.1 -P 3307 -u digikam_user -p

Esto debe iniciar la consola de MariaDB. En caso contrario, verificar que  haya conexión.

    ss -tuln | grep 3307

Si la hay, el problema es el password.

Se espera que salga algo parecido a lo siguiente:

    LISTEN 0 128 127.0.0.1:3307 ...

Comprueba que esté corriendo Docker

    docker ps

La respuesta debe ser tipo

    central-mariadb-local   mariadb:10.11   ...   0.0.0.0:3307->3306/tcp

Verifica que el puerto esté abierto en Fedora

    ss -tuln | grep 3307

Debes ver algo parecido a esto

    LISTEN 0 128 127.0.0.1:3307 ...

## 2025-05-23
Entra al contenedor desde Portainer (Console → /bin/bash)

Instala net-tools si no tienes netstat:

    apt update && apt install net-tools  # o usa 'apk' si es Alpine

Luego

    netstat -tuln | grep 3306

Se espera el siguiente resultado

    tcp  0  0 0.0.0.0:3306  0.0.0.0:*  LISTEN

Asegúrate de que bind-address no esté restringiendo

A veces MariaDB solo escucha en localhost dentro del contenedor. Puedes forzarlo a escuchar en todas las interfaces.

Entra al contenedor. Edita o crea el archivo 

    /etc/mysql/my.cnf o /etc/my.cnf 
    
(según la distro base, para fedora es la primer opción)

Agrega o modifica:

    bind-address=0.0.0.0

Reinicia el contenedor:

    docker restart central-mariadb-local

✅ Prueba final desde Fedora

    mysql -h 127.0.0.1 -P 3307 -u digikam_user -p

Si te deja entrar, entonces digiKam también podrá hacerlo.

Configurar el bind adress

✅ Solución: crear el archivo con echo y redirección

Desde dentro del contenedor, ejecuta:

    echo -e "[mysqld]\nbind-address = 0.0.0.0" > /etc/mysql/mariadb.conf.d/99-bind.cnf

✅ Luego, reinicia el contenedor desde Fedora:

    docker restart central-mariadb-local

🔍 Verifica que MariaDB escucha en todas las interfaces

Después del reinicio, entra de nuevo al contenedor y corre:

    netstat -tuln | grep 3306

Si no tienes netstat, prueba:

    ss -tuln | grep 3306

Esperas ver algo como:

    LISTEN 0 128 0.0.0.0:3306 ...

Luego recibí un error en DigiKam, que no tenía el driver, ChatGPT me dijo que era el driver de Qt y MariaDB lo que hacía falta, así que me preguntó si quería instalarlo y le dije que sí y ya funcionó todo.

El comando fue este

    QT_DEBUG_PLUGINS=1 digikam

Lo que tenía que instalar era sudo 
    
    dnf install qt5-qtbase-mysql qt6-qtbase-mysql
