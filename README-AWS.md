
---


# Tutorial 05: Despliegue de Django con Docker en AWS - Evidencia

## Introducción

Este documento sirve como evidencia de la realización del Tutorial 05, enfocado en el despliegue de una aplicación Django utilizando Docker sobre Amazon Web Services (AWS). El objetivo es mostrar paso a paso la creación de una instancia EC2, la instalación de Docker, la creación de una base de datos con RDS (MySQL), la configuración de un entorno de desarrollo con variables de entorno, y el despliegue final de la aplicación Django contenerizada.

## Autor

- **Nombre:** Maximiliano Bustamante  
- **Programa:** Ingeniería de Sistemas  
- **Universidad:** Universidad EAFIT

---

## Índice

- Antes de iniciar
- A. Acceso y creación de la instancia EC2
- B. Instalación de Docker y configuración de base de datos RDS
- C. Descarga y ejecución del proyecto Django con Docker
- Pregunta Final
- Consideraciones Finales (RDS)
- Comandos Adicionales Útiles
- Conclusiones
- Autoría

---

## Antes de iniciar

- Se completaron los tutoriales anteriores.
- Este taller muestra un ejemplo de despliegue de una aplicación Django con Docker en AWS.

---

## A. Acceso y creación de la instancia EC2

**A1.** Solicitar acceso al docente para AWS Academy.

**A2.** Ingresar a [AWS Academy](https://www.awsacademy.com/) con cuenta estudiantil.

**A3.** Acceder a la sección LMS → Cursos → Curso `TEIS-2025-1`.

**A4.** Ir a "Modules" y lanzar el AWS Academy Learner Lab.

**A5.** Iniciar el laboratorio → Esperar luz verde → Click en AWS.

**A6.** Buscar "EC2" y dar click en "Launch Instance".

**A7.** Configurar la instancia:

- Amazon Linux 2023 AMI
- Tipo: t2.micro
- Key Pair: *Proceed without key pair*
- Red: Activar HTTP y HTTPS

**A8.** Lanzar instancia y esperar a que esté disponible.

**A9.** Ir a "Instances", seleccionar la instancia → "Connect" → Usar "EC2 Instance Connect" → "Connect".

<!-- Insertar screenshot de conexión exitosa -->
![Conexión EC2](ruta/a/screenshot_conexion_ec2.png)

---

## B. Instalación de Docker y configuración de base de datos

**B1.** Instalar y habilitar Docker:

```bash
sudo dnf update -y
sudo dnf install -y docker
sudo systemctl enable --now docker
docker --version
```

<!-- Screenshot Docker version -->
![Docker instalado](ruta/a/screenshot_docker_version.png)

**B2.** Desde la consola AWS, buscar "RDS" → "Create database".

**B3.** Configuración de la base de datos:

- Tipo: MySQL
- Versión: 8.0
- Modo: Free Tier
- Identificador: teis20251
- Usuario: admin
- Contraseña: (segura y propia)
- Click en "Create database"

**B4.** Acceder al “VPC Security Group” de la base de datos:

- Ir a "Edit inbound rules"
- Añadir regla que permita acceso desde IP pública de EC2

**B5.** Instalar cliente MySQL (MariaDB):

```bash
sudo dnf install -y mariadb105
```

**B6.** Conectarse a la base de datos:

```bash
mysql -h <ENDPOINT_DB> -P 3306 -u admin -p
```

Ingresar contraseña → Crear la base de datos:

```sql
CREATE DATABASE djangodocker;
```

Salir con `exit`.

---

## C. Descarga y ejecución del proyecto Django con Docker

**C1.** Probar Docker con contenedor de ejemplo:

```bash
sudo docker container run hello-world
```

**C2.** Instalar git:

```bash
sudo dnf install git -y
```

**C3.** Descargar proyecto de ejemplo:

```bash
git clone https://github.com/Nram94/djangoDocker.git
cd djangoDocker
```

**C4.** Copiar archivo de entorno:

```bash
cp .env.example .env
```

**C5.** Editar `.env` con:

```bash
nano .env
```

Modificar con:

- DB_HOST: endpoint de RDS
- DB_DATABASE: djangodocker
- DB_USERNAME: admin
- DB_PASSWORD: (la usada anteriormente)

Guardar con `Ctrl+X`, luego `Y` y `Enter`.

<!-- Screenshot del .env modificado -->
![Archivo .env](ruta/a/screenshot_env_modificado.png)

**C6.** Construir imagen Docker:

```bash
sudo docker image build -t django-app .
```

**C7.** Ejecutar contenedor:

```bash
sudo docker container run -d --name django-docker -p 80:80 django-app
```

**C8.** Acceder a la app desde el navegador:

- Obtener IP pública de EC2 desde consola AWS
- Ir a `http://<IP_PUBLICA_EC2>/public`

<!-- Screenshot acceso web -->
![Acceso app Django](ruta/a/screenshot_app_django.png)

---

## Pregunta Final

**¿Qué ventaja tiene utilizar Docker para desplegar un proyecto en AWS versus hacerlo de forma manual en la VM?**

**Respuesta:** Docker permite empacar la aplicación y todas sus dependencias en una imagen portátil, garantizando que se ejecute de forma consistente en cualquier entorno que soporte Docker. Esto elimina problemas comunes de configuración y facilita el escalado, despliegue y mantenimiento. En contraste, la instalación manual en una VM es propensa a errores humanos, depende del sistema operativo, y no es fácilmente replicable.

---

## Consideraciones Finales (RDS)

> ⚠️ ¡IMPORTANTE! AWS RDS puede consumir créditos rápidamente.

- Una vez finalizado el tutorial:
  - Detener el contenedor de Docker
  - Borrar la base de datos desde la consola RDS
  - Detener la instancia EC2 si no se seguirá usando

---

## Comandos Adicionales Útiles

- Eliminar contenedor:

```bash
sudo docker container rm django-docker -f
```

- Limpiar recursos Docker:

```bash
sudo docker system prune -a
```

- Acceder a bash del contenedor:

```bash
sudo docker exec -it django-docker bash
```

---

## Conclusiones

Este tutorial permitió comprender el proceso completo de despliegue de una aplicación Django con Docker en la nube utilizando AWS. Se abordaron prácticas esenciales como la creación de infraestructura (EC2 y RDS), la instalación de herramientas necesarias (Docker, Git, MariaDB), y la configuración del entorno para correr la aplicación. La contenerización simplifica la portabilidad y confiabilidad del despliegue, siendo una herramienta clave para ambientes de desarrollo modernos y escalables. También se reflexionó sobre la importancia del monitoreo de costos y la administración responsable de recursos en la nube.

---

## Autoría

- **Realizado por:** Maximiliano Bustamante  
- **Programa:** Ingeniería de Sistemas  
- **Universidad:** Universidad EAFIT
```

---
