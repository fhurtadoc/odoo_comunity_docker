# odoo_comunity_docker
dockercomposer with odoo comunity for deploy 
Guía de Despliegue Local de Odoo 17 con Docker Compose
Esta guía detalla la configuración, solución de errores y pasos necesarios para levantar un entorno de desarrollo local de Odoo 17 Community utilizando Docker y Docker Compose con PostgreSQL 15.
1. Resumen de la Configuración y Solución de Problemas
Durante el proceso de configuración se resolvieron los siguientes inconvenientes clave:
Resolución de nombres de red (DNS de Docker): Se corrigió el valor de la variable de entorno HOST en el contenedor de Odoo. Inicialmente apuntaba a odoo, pero fue modificado a db para coincidir con el nombre del servicio de PostgreSQL dentro de la red interna de Docker.
Permisos en el volumen de PostgreSQL: Se solucionó el error initdb: error: could not change permissions of directory "/var/lib/postgresql/data": Operation not permitted reemplazando el montaje de carpeta local por un volumen nombrado administrado directamente por Docker (postgres-db-data).
Centralización de credenciales: Se eliminaron las credenciales hardcodeadas dentro de docker-compose.yml y se delegó el control de variables críticas de entorno al archivo .env.
2. Requisito Fundamental: Archivo .env
Es estrictamente obligatorio crear un archivo llamado .env en el mismo directorio donde se encuentra el archivo docker-compose.yml. Este archivo define las credenciales secretas y parámetros de conexión para PostgreSQL y Odoo.
Ejemplo de contenido para el archivo .env:
POSTGRES_DB=postgres
POSTGRES_USER=odoo
POSTGRES_PASSWORD=Mathi142014
3. Archivo docker-compose.yml
A continuación se presenta la estructura final y limpia de docker-compose.yml utilizando las variables del archivo .env:
services:
  web:
    image: odoo:17.0
    container_name: odoo_app
    depends_on:
      - db
    ports:
      - "8069:8069"
      - "8072:8072"
    volumes:
      - odoo-web-data:/var/lib/odoo
      - ./config:/etc/odoo
      - ./addons:/mnt/extra-addons
    environment:
      - HOST=db
      - USER=${POSTGRES_USER}
      - PASSWORD=${POSTGRES_PASSWORD}
    restart: always
    networks:
      - my_network

  db:
    image: postgres:15
    container_name: odoo_db
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres-db-data:/var/lib/postgresql/data
    restart: always
    networks:
      - my_network

volumes:
  odoo-web-data:
  postgres-db-data:

networks:
  my_network:
    driver: bridge
4. Estructura del Proyecto
Asegúrate de tener la siguiente estructura de carpetas en tu entorno local:
.
├── .env
├── docker-compose.yml
├── addons/          # Tus módulos y addons personalizados de Odoo
└── config/          # Archivos de configuración opcionales (odoo.conf)
5. Comandos para Iniciar y Administrar los Servicios
Iniciar los contenedores en segundo plano:
docker compose up -d
Verificar que los contenedores estén corriendo:
docker ps
Ver los logs en tiempo real:
# Logs de Odoo
docker logs -f odoo_app

# Logs de PostgreSQL
docker logs -f odoo_db
Detener los contenedores:
docker compose down
Reiniciar y recrear contenedores (en caso de cambios de red o .env):
docker compose down
docker compose up -d --force-recreate
6. Acceso a la Aplicación
Una vez que los contenedores reporten el estado Up, abre tu navegador e ingresa a:
http://localhost:8069 o http://127.0.0.1:8069/
Al acceder por primera vez, verás el formulario de inicialización de Odoo donde podrás crear la primera base de datos del sistema utilizando el correo y contraseña de tu elección.
