# 🎮 Servidor de Minecraft (Java Edition) con Docker

[![Docker](https://img.shields.io/badge/Docker-Required-blue)](https://www.docker.com/)
[![Minecraft](https://img.shields.io/badge/Minecraft-Java%20Edition-green)](https://www.minecraft.net/)

Despliegue completo de un servidor de Minecraft Java Edition utilizando Docker Compose. Esta solución incluye persistencia de datos, gestión de recursos, configuración de administradores y herramientas de mantenimiento.

## 📋 Tabla de Contenidos

- [Introducción](#introducción)
- [Características](#características)
- [Requisitos Previos](#requisitos-previos)
- [Instalación Rápida](#instalación-rápida)
- [Configuración](#configuración)
- [Uso](#uso)
- [Administración](#administración)
- [Troubleshooting](#troubleshooting)
- [Configuración Avanzada](#configuración-avanzada)
- [Referencias](#referencias)

---

## 🎯 Introducción

Este proyecto proporciona una solución completa para desplegar un servidor de Minecraft Java Edition utilizando **Docker Compose**. La configuración está basada en la imagen oficial mantenida por la comunidad [itzg/minecraft-server](https://hub.docker.com/r/itzg/minecraft-server), que automatiza la instalación y configuración del servidor.

### Características

- ✅ **Despliegue rápido**: Configuración lista para usar en minutos
- ✅ **Persistencia de datos**: Los mundos y configuraciones se guardan automáticamente
- ✅ **Gestión de recursos**: Control de memoria RAM para optimizar el rendimiento
- ✅ **Reinicio automático**: El contenedor se reinicia automáticamente si se detiene
- ✅ **Acceso remoto**: Configuración RCON para administración remota
- ✅ **Fácil mantenimiento**: Comandos simples para gestionar el ciclo de vida del servidor

---

## 📦 Requisitos Previos

Antes de comenzar, asegúrate de tener instalado:

| Requisito | Versión Mínima | Descripción |
|-----------|----------------|-------------|
| **Docker Engine** | 20.10+ | Motor de contenedores |
| **Docker Compose** | 2.0+ | Orquestación de contenedores (incluido en Docker Desktop) |
| **RAM disponible** | 4GB+ | Mínimo recomendado para el servidor |
| **Espacio en disco** | 5GB+ | Para datos del servidor y mundos |
| **Conexión a Internet** | - | Para descargar la imagen base |

### Verificar Instalación

```bash
# Verificar Docker
docker --version

# Verificar Docker Compose
docker compose version
```

---

## 🚀 Instalación Rápida

### Paso 1: Clonar o Descargar el Proyecto

```bash
cd 07-docker-minecraft
```

### Paso 2: Iniciar el Servidor

```bash
docker compose up -d
```

### Paso 3: Verificar el Estado

```bash
docker logs -f mc-server
```

Espera hasta ver el mensaje: `Done (X.Xs)! For help, type "help".`

### Paso 4: Conectar al Servidor

Abre Minecraft Java Edition y conecta a:
- **Dirección**: `localhost` (local) o `TU_IP_PUBLICA` (remoto)
- **Puerto**: `25565`

---

## ⚙️ Configuración

### Estructura del Proyecto

```
07-docker-minecraft/
├── docker-compose.yml    # Configuración principal del servicio
└── README.md            # Esta documentación
```

### Archivo docker-compose.yml

El archivo `docker-compose.yml` contiene toda la configuración necesaria:

```yaml
services:
  minecraft-server:
    image: itzg/minecraft-server
    container_name: mc-server
    ports:
      - "25565:25565"
    environment:
      # Aceptación obligatoria del Acuerdo de Licencia de Usuario Final
      - EULA=TRUE
      # Límite de memoria asignada a la JVM (Java Virtual Machine)
      - MEMORY=2G
    volumes:
      # Persistencia de datos (mundos, configuraciones, inventarios)
      - mc-data:/data
    restart: unless-stopped

volumes:
  mc-data:
```

### Desglose de Parámetros Técnicos

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| **image** | String | Imagen Docker `itzg/minecraft-server` que automatiza la instalación de Java y binarios del servidor |
| **container_name** | String | Nombre del contenedor (`mc-server`) para facilitar su identificación |
| **ports** | Mapping | Mapea el puerto `25565` del contenedor al host, permitiendo conexiones externas |
| **EULA=TRUE** | Environment | Variable crítica. Sin ella, el servidor abortará el inicio inmediatamente |
| **MEMORY=2G** | Environment | Gestiona el *Heap Size* de Java. Previene errores de "Out of Memory" |
| **volumes** | Volume | El volumen `mc-data` garantiza persistencia de datos si el contenedor es eliminado |
| **restart** | Policy | `unless-stopped` reinicia automáticamente el contenedor tras reinicios del sistema |

---

## 🎮 Uso

### Comandos Básicos

#### Iniciar el Servidor

```bash
docker compose up -d
```

El parámetro `-d` ejecuta el contenedor en segundo plano (modo *detached*).

#### Ver Logs en Tiempo Real

```bash
docker logs -f mc-server
```

#### Detener el Servidor

```bash
docker compose down
```

Realiza un apagado controlado (*graceful shutdown*), guardando todos los datos.

#### Reiniciar el Servidor

```bash
docker compose restart
```

O para aplicar cambios de configuración:

```bash
docker compose down
docker compose up -d
```

---

## 👨‍💼 Administración

### Gestión de Permisos (Operador/Admin)

Dado que el servidor se ejecuta en un entorno aislado, la consola estándar no es accesible directamente. Para otorgar permisos de administrador (OP) a un usuario, utilizaremos la herramienta `rcon-cli` a través de `docker exec`.

#### Otorgar Permisos de Administrador

```bash
docker exec mc-server rcon-cli op <NOMBRE_USUARIO>
```

**Ejemplo:**
```bash
docker exec mc-server rcon-cli op Steve
```

**Resultado esperado:** `Made Steve a server operator`

#### Revocar Permisos de Administrador

```bash
docker exec mc-server rcon-cli deop <NOMBRE_USUARIO>
```

### Acceso Interactivo a la Consola (RCON)

Para ejecutar comandos interactivamente:

```bash
docker exec -it mc-server rcon-cli
```

Una vez dentro, puedes ejecutar comandos de Minecraft directamente:
- `time set day` - Cambiar la hora del día
- `gamemode creative <usuario>` - Cambiar modo de juego
- `tp <usuario1> <usuario2>` - Teletransportar jugadores
- `kick <usuario> <razón>` - Expulsar un jugador

### Comandos Útiles de Administración

```bash
# Ver jugadores conectados
docker exec mc-server rcon-cli list

# Enviar mensaje a todos los jugadores
docker exec mc-server rcon-cli say "Mensaje de anuncio"

# Cambiar dificultad
docker exec mc-server rcon-cli difficulty normal

# Guardar el mundo manualmente
docker exec mc-server rcon-cli save-all

# Ver información del servidor
docker exec mc-server rcon-cli seed
```

---

## 🔧 Troubleshooting

### El servidor no inicia

**Problema:** El contenedor se detiene inmediatamente después de iniciar.

**Solución:**
1. Verifica los logs: `docker logs mc-server`
2. Asegúrate de que `EULA=TRUE` esté configurado
3. Verifica que el puerto 25565 no esté en uso: `lsof -i :25565`

### Error de memoria insuficiente

**Problema:** El servidor muestra errores de "Out of Memory".

**Solución:**
- Reduce el valor de `MEMORY` en `docker-compose.yml` (ej: `1G`)
- O aumenta la RAM disponible en tu sistema

### No puedo conectarme al servidor

**Problema:** No puedo conectarme desde otro dispositivo.

**Solución:**
1. Verifica que el puerto esté abierto en el firewall
2. Asegúrate de usar la IP correcta (no `localhost` desde otro dispositivo)
3. Verifica que el servidor esté ejecutándose: `docker ps`

### Los datos no se guardan

**Problema:** Al reiniciar el contenedor, se pierden los datos.

**Solución:**
- Verifica que el volumen `mc-data` exista: `docker volume ls`
- Asegúrate de usar `docker compose down` (no `docker stop`) para detener el servidor

### Verificar Estado del Contenedor

```bash
# Ver contenedores en ejecución
docker ps

# Ver todos los contenedores (incluyendo detenidos)
docker ps -a

# Ver información detallada del contenedor
docker inspect mc-server

# Ver uso de recursos
docker stats mc-server
```

---

## 🚀 Configuración Avanzada

### Personalizar la Versión de Minecraft

Agrega la variable de entorno `VERSION`:

```yaml
environment:
  - EULA=TRUE
  - MEMORY=2G
  - VERSION=1.20.1  # Versión específica
```

### Habilitar Modo Creativo por Defecto

```yaml
environment:
  - EULA=TRUE
  - MEMORY=2G
  - MODE=creative
```

### Configurar Dificultad

```yaml
environment:
  - EULA=TRUE
  - MEMORY=2G
  - DIFFICULTY=hard
```

### Habilitar RCON con Contraseña

```yaml
environment:
  - EULA=TRUE
  - MEMORY=2G
  - ENABLE_RCON=true
  - RCON_PASSWORD=tu_contraseña_segura
  - RCON_PORT=25575
```

Luego conecta usando:
```bash
docker exec mc-server rcon-cli -p tu_contraseña_segura
```

### Montar Configuraciones Personalizadas

```yaml
volumes:
  - mc-data:/data
  - ./server.properties:/data/server.properties:ro
  - ./whitelist.json:/data/whitelist.json:ro
```

### Aumentar Memoria para Servidores Grandes

Para servidores con muchos jugadores o mods:

```yaml
environment:
  - MEMORY=4G  # Aumentar según tu hardware
```

**Recomendaciones de memoria:**
- 1-5 jugadores: 2GB
- 5-10 jugadores: 4GB
- 10-20 jugadores: 6GB
- 20+ jugadores: 8GB+

### Backup Manual de Datos

```bash
# Crear backup del volumen
docker run --rm -v mc-data:/data -v $(pwd):/backup alpine tar czf /backup/minecraft-backup-$(date +%Y%m%d).tar.gz -C /data .

# Restaurar backup
docker run --rm -v mc-data:/data -v $(pwd):/backup alpine tar xzf /backup/minecraft-backup-YYYYMMDD.tar.gz -C /data
```

---

## 📚 Referencias

### Recursos Oficiales

- [Docker Hub - itzg/minecraft-server](https://hub.docker.com/r/itzg/minecraft-server)
- [Documentación de la Imagen](https://github.com/itzg/docker-minecraft-server)
- [Minecraft Java Edition](https://www.minecraft.net/)

### Variables de Entorno Disponibles

La imagen `itzg/minecraft-server` soporta muchas variables de entorno. Consulta la [documentación completa](https://github.com/itzg/docker-minecraft-server/blob/master/README.md#environment-variables) para ver todas las opciones disponibles.

### Comandos Docker Útiles

```bash
# Ver logs de las últimas 100 líneas
docker logs --tail 100 mc-server

# Seguir logs en tiempo real
docker logs -f mc-server

# Ejecutar comando dentro del contenedor
docker exec mc-server <comando>

# Acceder al shell del contenedor
docker exec -it mc-server bash

# Ver uso de recursos en tiempo real
docker stats mc-server

# Limpiar recursos no utilizados
docker system prune -a
```

---

## 📝 Notas Adicionales

- **Primera ejecución**: La primera vez que inicies el servidor, puede tardar varios minutos mientras se descarga la imagen y se genera el mundo.
- **Puerto 25565**: Este es el puerto estándar de Minecraft. Asegúrate de que no esté bloqueado por tu firewall.
- **Persistencia**: Todos los datos se guardan en el volumen Docker `mc-data`. Este volumen persiste incluso si eliminas el contenedor.
- **Rendimiento**: El rendimiento del servidor depende de la RAM asignada y la potencia del CPU del host.

---

## 📄 Licencia

Este proyecto utiliza la imagen Docker `itzg/minecraft-server` que está bajo la licencia MIT. Minecraft es una marca registrada de Mojang Studios.

---

## 🤝 Contribuciones

Si encuentras algún problema o tienes sugerencias de mejora, no dudes en crear un issue o pull request.

---

**¡Disfruta tu servidor de Minecraft! 🎮**
