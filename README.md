# Mejores Prácticas para Trabajar con Docker

Este documento reúne las mejores prácticas para trabajar con Docker, desde la elección de imágenes adecuadas hasta la optimización de tus Dockerfiles y contenedores. Seguir estas recomendaciones te ayudará a crear aplicaciones más seguras, ligeras y eficientes.


## 1. Usar Imágenes Docker Oficiales

Las imágenes oficiales de Docker están mantenidas por Docker y por los mantenedores de las aplicaciones. Estas imágenes son generalmente más seguras y confiables que las imágenes no oficiales. Siempre que sea posible, utiliza imágenes oficiales como base para tus contenedores.

  Ejemplo:

    FROM node:16-alpine

## 2. Considerar una Versión con Etiqueta

Es importante no usar latest como etiqueta para las imágenes Docker, ya que esto puede llevar a inconsistencias y problemas de seguridad. En su lugar, es mejor utilizar una etiqueta de versión específica. Esto te da más control sobre qué versión de la imagen estás utilizando y te protege de cambios inesperados.

  Mala práctica:

    FROM node:latest

Buena práctica:

    FROM node:16-alpine

Esto asegura que siempre uses la versión específica que necesitas y te ayuda a evitar problemas derivados de cambios no controlados.

## 3. Elegir Imágenes Mínimas con Alpine

Alpine Linux es una distribución minimalista, ligera y de alto rendimiento. Usar imágenes basadas en Alpine puede reducir significativamente el tamaño de tus imágenes Docker, lo que lleva a tiempos de construcción más rápidos y a una mejor eficiencia en el uso del almacenamiento y la red.

  Ejemplo:

    FROM python:3.9-alpine

Si tu imagen base no tiene soporte para Alpine, investiga si existe una versión ligera o más optimizada. Esto no solo ahorra espacio, sino que también puede mejorar la seguridad al reducir la superficie de ataque.

## 4. Usar Multi Stage Builds

Los Multi Stage Builds permiten crear imágenes más pequeñas y eficientes, separando las fases de construcción de las de ejecución. Puedes usar una imagen para construir tu aplicación y otra más ligera para ejecutarla, eliminando dependencias y archivos innecesarios del contenedor final.

  Ejemplo de Dockerfile con Multi Stage Build:

    # Etapa 1: Construcción
    FROM node:16-alpine AS builder
    WORKDIR /app
    COPY . .
    RUN npm install && npm run build

    # Etapa 2: Imagen final
    FROM nginx:alpine
    COPY --from=builder /app/build /usr/share/nginx/html

En este ejemplo, el primer contenedor (builder) se encarga de construir la aplicación, mientras que el segundo contenedor utiliza la imagen de nginx para servir los archivos estáticos generados, sin incluir las dependencias de desarrollo.

## 5. Borrar el Caché de Docker Desktop

A medida que trabajas con Docker, se acumulan muchas capas de caché que pueden ocupar espacio y hacer que Docker funcione más lentamente. Para mantener tu sistema eficiente, es importante eliminar regularmente las imágenes y contenedores no utilizados. Puedes hacerlo con los siguientes comandos:

  Eliminar contenedores detenidos:

    docker container prune

Eliminar imágenes no utilizadas:

    docker images prune -a

Eliminar todos los recursos no utilizados (contenedores, redes, imágenes, etc.):

    docker system prune -a

Realiza estos mantenimientos con frecuencia para evitar el consumo innecesario de espacio y mejorar el rendimiento de Docker.

## 6. Crear Usuarios Dentro del Dockerfile

Por razones de seguridad, siempre que sea posible, deberías crear un usuario no privilegiado dentro de tu Dockerfile. Ejecutar procesos como root dentro del contenedor puede exponer tu aplicación a vulnerabilidades de seguridad.

  Ejemplo de Dockerfile con un usuario no privilegiado:

    FROM node:16-alpine

    # Crear un usuario no root
    RUN addgroup -S appgroup && adduser -S appuser -G appgroup

    # Establecer el directorio de trabajo
    WORKDIR /app

    # Copiar archivos y cambiar la propiedad a appuser
    COPY . .
    RUN chown -R appuser:appgroup /app

    # Cambiar a appuser
    USER appuser

    # Ejecutar la aplicación
    CMD ["npm", "start"]

De este modo, tu aplicación no corre con privilegios de root, mejorando la seguridad.

## 7. Usar Distroless

Las imágenes distroless están diseñadas para ser lo más ligeras posible, incluyendo solo lo necesario para ejecutar la aplicación. No contienen herramientas de shell, gestores de paquetes ni otros componentes de sistema operativo que no sean necesarios para la ejecución de la aplicación.

Utilizar imágenes distroless reduce la superficie de ataque y el tamaño de la imagen, lo que mejora la seguridad y la eficiencia.

  Ejemplo:

    FROM gcr.io/distroless/nodejs:16

Ten en cuenta que el uso de distroless puede requerir una estructura de ejecución diferente, ya que no incluye herramientas de depuración como bash o sh.

## 8. Otras Buenas Prácticas

### - Minimiza las capas en tu Dockerfile

Cada instrucción en el Dockerfile crea una nueva capa en la imagen. Usa instrucciones como `RUN`, `COPY` o `ADD` de manera eficiente para minimizar el número de capas y, por lo tanto, el tamaño de la imagen.

Mala práctica:

      RUN apt-get update
      RUN apt-get install -y curl

Buena práctica:

      RUN apt-get update && apt-get install -y curl

De esta manera, reduces el número de capas y haces que tu imagen sea más ligera.

### - Usa .dockerignore

Para evitar que Docker copie archivos innecesarios (como archivos de configuración locales o dependencias no relevantes), usa un archivo .dockerignore para excluirlos del contexto de construcción. Esto reduce el tamaño de la imagen final y mejora la eficiencia.

  Ejemplo de .dockerignore:

    node_modules
    *.log
    .git

Este archivo debe incluir los directorios y archivos que no son necesarios para construir tu contenedor. Por ejemplo, las dependencias de desarrollo locales o archivos temporales generados por tu editor de código.

### - Mantén tus imágenes actualizadas

Revisa regularmente las imágenes base y actualízalas para corregir vulnerabilidades de seguridad. Las imágenes desactualizadas pueden tener fallos de seguridad que podrían exponer tu aplicación a riesgos. Usa herramientas como Docker Scan para analizar vulnerabilidades de las imágenes.

  Comando para escanear vulnerabilidades:

    docker scan <nombre-imagen>

Además, asegúrate de actualizar las versiones de las imágenes base en tu Dockerfile para obtener las últimas actualizaciones y correcciones de seguridad.

### - No incluyas información sensible en tus imágenes

No pongas secretos, claves o contraseñas directamente en el Dockerfile. Esto puede exponer información sensible si la imagen se comparte o se sube a repositorios públicos. En su lugar, utiliza mecanismos más seguros, como Docker Secrets o Variables de Entorno.

  Mala práctica:

    ENV DB_PASSWORD=supersecreta

Buena práctica: Usa Docker Secrets o Variables de Entorno:

Usar Docker Secrets:

    docker secret create db_password db_password.txt

### - Usar Variables de Entorno en el docker-compose.yml:

        version: '3'
        services:
          myapp:
            image: myapp:latest
            environment:
              - DB_PASSWORD=${DB_PASSWORD}

De esta forma, mantienes la seguridad de tu aplicación al no exponer directamente la información sensible dentro de las imágenes.
