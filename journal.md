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