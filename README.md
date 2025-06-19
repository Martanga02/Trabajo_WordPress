# Despliegue de WordPress y Base de Datos con Docker

**Autora: Martina Flores Tosini**

**Proyecto: Martina-docker**

**Curso: Virtualización – Despliegue de WordPress con Docker**


## Descripción del entorno

Este entorno despliega un sitio WordPress llamado **Martina-docker** conectado a una base de datos MySQL utilizando Docker Compose. Se usan volúmenes para persistencia, y el entorno fue construido y personalizado para permitir la carga de archivos de gran tamaño.

## Servicios utilizados

- **db**: Contenedor basado en la imagen oficial `mysql:5.7`
- **wordpress**: Imagen personalizada a partir de `wordpress:latest`, construida con Dockerfile propio.

## Archivos entregados

- `docker-compose.yml`: define los servicios y sus configuraciones. (Se encuentra en el ZIP)
- `Dockerfile`: modifica parámetros de PHP para permitir archivos grandes. (Se encuentra en el ZIP)
- `README.md`: documentación del entorno, problema detectado y solución aplicada.

## Instrucciones para reproducir el entorno

1. Clonar o copiar el proyecto.
2. En la carpeta raíz, ejecutar:

bash
docker compose up -d --build

3. Acceder desde el navegador a:
http://localhost:8000

4. Completar la instalación inicial de WordPress (nombre de usuario, contraseña, etc.).

Prueba de error al subir archivos grandes
Durante las pruebas iniciales se intentó subir un archivo de 400 MB usando el panel de medios de WordPress. Como era de esperarse, se presentó el siguiente error:

“El archivo excede el tamaño máximo permitido en php.ini”

Esto se debe a los valores por defecto de PHP en la imagen wordpress:latest, que son muy bajos (normalmente 2 MB).

Solución aplicada
Para resolver el problema, se creó un Dockerfile que extiende la imagen original y modifica la configuración de PHP:

Contenido del Dockerfile:
FROM wordpress:latest

RUN echo "upload_max_filesize = 512M\npost_max_size = 512M" > /usr/local/etc/php/conf.d/uploads.ini

Luego se modificó docker-compose.yml para usar build: . en lugar de image: wordpress:latest.

Finalmente, se reconstruyó el entorno desde cero:
docker compose down -v
docker compose up -d --build

Después de aplicar la solución, se volvió a intentar la carga del archivo de 400 MB, y funcionó correctamente. Se adjunta captura como evidencia .
![WhatsApp Image 2025-06-19 at 14 59 33](https://github.com/user-attachments/assets/cbb77a17-3445-4445-864b-bd4ebffc5c05)
![WhatsApp Image 2025-06-19 at 14 59 34](https://github.com/user-attachments/assets/d14e3aac-1cc4-4d3d-94fe-d5ec380d5b73)


**Errores encontrados y aprendizajes**
Durante el proceso, se presentaron algunos desafíos que se resolvieron:

❌ Al principio, el navegador no accedía a localhost:8000 porque en docker-compose.yml el puerto estaba expuesto como 8080.
✔️ Se corrigió cambiando 8080:80 a 8000:80.

❌ WordPress mostraba "Error establishing a database connection".
✔️ El volumen de MySQL estaba “sucio”, así que se borró todo con down -v y se corrigieron las variables de entorno.

❌ Al hacer docker compose up, aparecía un error por claves duplicadas en el servicio wordpress.
✔️ Se reemplazó toda la sección wordpress: con la versión corregida (sin duplicar restart ni environment).

❌ No se encontraba el archivo de prueba archivo_prueba.mp4 desde el navegador para subirlo.
✔️ Se usó find para localizarlo en el sistema y se subió exitosamente desde WordPress.

Estos errores me ayudaron a entender cómo funciona la persistencia, el build de imágenes personalizadas y la configuración interna de PHP en entornos Docker.


