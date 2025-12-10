
# 🐳 Docker: Comandos Básicos (incluye redes)

Guía rápida de los comandos esenciales de Docker y el uso de **redes (network)**.

---

## Índice
- Verificación y ayuda
- Imágenes
- Contenedores
- Logs y shell dentro del contenedor
- Construir imágenes (Dockerfile)
- Redes (Docker Network)
- Limpieza
- Notas útiles

---

# Verificación y ayuda

## Ver versión instalada
`docker --version`

## Información del entorno Docker (daemon, storage, redes, etc.)
`docker info`

# Imagenes


## Descargar una imagen
`docker pull <imagen>:<tag>`

## Ejemplo:
`docker pull nginx:latest`

## Listar imágenes descargadas
`docker images`

## Eliminar una imagen
`docker rmi <imagen_o_id>`


# Contenedores


## Listar contenedores en ejecución
`docker ps`

## Listar todos los contenedores (incluye detenidos)
`docker ps -a`

## Crear y ejecutar un contenedor (básico)
`docker run <opciones> <imagen>`

## Ejemplos comunes:
[ ] -d: modo detached (segundo plano)
[ ] -p: mapear puertos (host:contenedor)
[ ] --name: nombre del contenedor

`docker run -d -p 8080:80 --name web-nginx nginx:latest`

## Detener un contenedor
`docker stop <nombre_o_id>`

## Iniciar un contenedor detenido
`docker start <nombre_o_id>`

## Reiniciar un contenedor
`docker restart <nombre_o_id>`

## Eliminar un contenedor (debe estar detenido)
`docker rm <nombre_o_id>`

## Ver detalles de un contenedor
`docker inspect <nombre_o_id>`

# Logs y shell dentro del contenedor

## Ver logs (con seguimiento en tiempo real)
`docker logs -f <nombre_o_id>`

## Ejecutar un comando dentro del contenedor (shell interactivo)
`docker exec -it <nombre_o_id> /bin/bash`

## Si no tiene bash:
`docker exec -it <nombre_o_id> /bin/sh`

# Construir imágenes (Dockerfile)

## Construir una imagen desde el Dockerfile en el directorio actual
`docker build -t <nombre_imagen>:<tag> .`

## Ejemplo con etiqueta y contexto específico
`docker build -t miapp:1.0 ./src`

## Construir pasando argumentos de compilación
`docker build --build-arg ENV=prod -t miapp:prod .`


# Redes (Docker Network)

## Listar redes existentes
`docker network ls`

## Inspeccionar una red (detalles, subred, contenedores conectados)
`docker network inspect <nombre_red>`

## Crear una red bridge (la más común)
`docker network create mi_red`

## Crear red con parámetros (subred, gateway, driver)
```docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --gateway 172.20.0.1 \
  mi_red_custom
```

## Conectar un contenedor a una red
`docker network connect <nombre_red> <nombre_contenedor>`

## Desconectar un contenedor de una red
`docker network disconnect <nombre_red> <nombre_contenedor>`

## Eliminar una red (sin contenedores conectados)
`docker network rm <nombre_red>`

# Usar una red al crear contenedor

## Crear contenedor asignado a una red específica
`docker run -d --name app --network mi_red nginx`

## Asignar alias de red (DNS interno en la red)
`docker run -d --name api --network mi_red --network-alias api.internal nginx`

# Redes comunes (drivers)

## Bridge (por defecto): aislamiento básico para contenedores en el mismo host
`docker network create --driver bridge mi_red_bridge`

## Host: el contenedor usa la red del host (Linux)
`docker run --network host <imagen>`

## None: sin conectividad de red
`docker run --network none <imagen>`

## Overlay: para múltiples hosts (requiere Docker Swarm)
`docker network create --driver overlay --attachable mi_red_overlay`


# Limpieza

## Eliminar contenedores, redes no usadas, imágenes no referenciadas y caché
`docker system prune`

## Versión más agresiva (incluye imágenes no usadas)
`docker system prune -a`

## Eliminar volúmenes no usados (ojo: datos persistentes)
`docker volume prune`

# Notas útiles

[ ] Puertos: -p 8080:80 expone el puerto 80 del contenedor en 8080 del host.
[ ] Variables de entorno: -e CLAVE=valor para pasar config al contenedor.
[ ] Volúmenes (persistencia): -v /ruta/host:/ruta/contenedor.
[ ] Auto-reinicio: --restart unless-stopped para mantener servicios corriendo.
[ ] Salud: agrega HEALTHCHECK en tu Dockerfile y consulta con docker inspect.


## Ejemplo completo con red, volumen, variable y restart
```
docker run -d \
  --name mi-servicio \
  --network mi_red \
  -p 8000:8080 \
  -v /datos/app:/var/lib/app \
  -e ASPNETCORE_ENVIRONMENT=Production \
  --restart unless-stopped \
  mi_imagen:latest

```


# Ejemplo de flujo con redes

## 1) Crear red de aplicación
`docker network create mi_red_app`

## 2) Levantar API en esa red
`docker run -d --name api --network mi_red_app -p 5000:5000 mi_api:latest`

## 3) Levantar frontend y conectarlo a la misma red
`docker run -d --name web --network mi_red_app -p 8080:80 mi_web:latest`

## 4) Inspeccionar la red para ver contenedores conectados
`docker network inspect mi_red_app`
