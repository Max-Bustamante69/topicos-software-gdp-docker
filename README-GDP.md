
# Tutorial 05: Despliegue de Django con Docker en GCP - Evidencia

## Introducción

Este documento sirve como evidencia de la realización del Tutorial 05, enfocado en el despliegue de una aplicación Django utilizando Docker sobre Google Cloud Platform (GCP). El objetivo es mostrar paso a paso la configuración de la infraestructura necesaria en GCP, la instalación de Docker, la configuración de una base de datos Cloud SQL y el despliegue final de la aplicación Django contenerizada.

## Autor

*   **Nombre:** Maximiliano Bustamante
*   **Programa:** Ingeniería de Sistemas
*   **Universidad:** Universidad EAFIT

## Índice

1.  [Antes de iniciar](#antes-de-iniciar)
2.  [A. Creando la máquina virtual](#a-creando-la-máquina-virtual)
3.  [B. Accediendo a la máquina virtual](#b-accediendo-a-la-máquina-virtual)
4.  [C. Instalar Docker](#c-instalar-docker)
5.  [D. Instalar la base de datos en Cloud SQL](#d-instalar-la-base-de-datos-en-cloud-sql)
6.  [E. Configurar proyecto Django con Docker](#e-configurar-proyecto-django-con-docker)
7.  [F. Correr proyecto Django con Docker](#f-correr-proyecto-django-con-docker)
8.  [Pregunta Final](#pregunta-final)
9.  [Consideraciones Finales (Cloud SQL)](#consideraciones-finales-cloud-sql)
10. [Comandos Adicionales Útiles](#comandos-adicionales-útiles)
11. [Conclusiones](#conclusiones)
12. [Autoria](#autoria)

---

## Antes de iniciar

*   Se completaron los tutoriales anteriores.
*   Este taller muestra un ejemplo de despliegue de una aplicación Django con Docker en GCP.

---

## A. Creando la máquina virtual

**A1.** Conexión a la cuenta de Google Cloud usando el correo EAFIT.
*   URL: `https://console.cloud.google.com/`

<!-- AQUÍ VA TU SCREENSHOT: Pantalla de inicio de GCP o similar -->
![image](https://github.com/user-attachments/assets/77aec3ef-d414-4a77-82a9-01df1b70d0b5)


**A2.** Habilitar la API de "Compute Engine".

<!-- AQUÍ VA TU SCREENSHOT: Habilitando la API de Compute Engine -->
![image](https://github.com/user-attachments/assets/428233ee-205d-48eb-bbe3-1f4280942131)


**A3.** Navegar a "VM instances” -> “Create instance".

<!-- AQUÍ VA TU SCREENSHOT: Navegación a Crear Instancia VM -->
![image](https://github.com/user-attachments/assets/b8c2eb99-44b7-4a17-989e-e6f08fda3a57)


**A4.** Configurar la instancia con los siguientes valores (o similares según el tutorial):
*   **Name:** `instance-docker`
*   **Region:** `us-central1`
*   **Zone:** `us-central1-a`
*   **Machine type:** `e2-micro` (2 vCPU, 1 GB memory)
*   **Boot disk:** Debian GNU/Linux 10 (buster), 10 GB balanced persistent disk
*   **Firewall:** Allow HTTP traffic, Allow HTTPS traffic
*   Verificar costo estimado mensual (~$7).

<!-- AQUÍ VA TU SCREENSHOT: Configuración de la instancia VM -->
![image](https://github.com/user-attachments/assets/1a5bcd9f-f3d4-4a63-bf00-f6963c94791b)
![image](https://github.com/user-attachments/assets/d57df19c-530e-46c7-8988-54e1d8859773)



**A5.** Esperar a que la instancia esté disponible en la lista.

<!-- AQUÍ VA TU SCREENSHOT: Lista de instancias VM con la nueva instancia creada -->
![image](https://github.com/user-attachments/assets/24a2ca1e-fa05-4518-8e48-46262b065602)


---

## B. Accediendo a la máquina virtual

**B1.** Click sobre el nombre de la instancia creada (`instance-docker`).

**B2.** Seleccionar “SSH” -> “Open in browser window”.

<!-- AQUÍ VA TU SCREENSHOT: Ventana de terminal SSH abierta en el navegador -->
![image](https://github.com/user-attachments/assets/0974bca3-d4d5-4317-a56f-f5169011d98a)


---

## C. Instalar Docker

**C1.** Ejecutar los siguientes comandos en la terminal SSH para instalar Docker en Debian:

```bash
# Actualizar lista de paquetes
sudo apt-get update

# Instalar paquetes necesarios para usar repositorios sobre HTTPS
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common

# Añadir la clave GPG oficial de Docker
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

# Añadir el repositorio estable de Docker
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian buster stable"

# Actualizar lista de paquetes nuevamente
sudo apt-get update

# Instalar Docker Engine, CLI y Containerd
sudo apt-get install -y docker-ce docker-ce-cli containerd.io


<!-- AQUÍ VA TU SCREENSHOT: Ejecución de los comandos de instalación de Docker -->
![Screenshot Instalación Docker](ruta/a/screenshot_install_docker.png)

**C2.** Verificar que Docker quedó instalado:

```bash
docker -v
```

<!-- AQUÍ VA TU SCREENSHOT: Salida del comando 'docker -v' -->
![image](https://github.com/user-attachments/assets/90efc66c-1245-481f-bb67-b96fd88e290d)


---

## D. Instalar la base de datos en Cloud SQL

**D1.** En GCP, navegar a la sección "SQL".

<!-- AQUÍ VA TU SCREENSHOT: Navegación a Cloud SQL -->
![image](https://github.com/user-attachments/assets/da4c2c32-fa24-4933-b400-ae2e60255c49)


**D2.** Click en "Create Instance".

**D3.** Seleccionar "MySQL".

**D4.** Configurar la instancia de base de datos:
*   **Instance ID:** `laravelbd` (o el nombre que prefieras)
*   **Password:** Establecer una contraseña segura para el usuario `root`.
*   **Database version:** MySQL 8.0 (o la versión indicada)
*   **Choose configuration:** Seleccionar "Sandbox" (o "Development" si no aparece Sandbox).
*   **Zonal availability:** Single zone (para minimizar costos en el tutorial).
*   **Machine type:** Lightweight (1 vCPU, 3.75 GB) o similar.
*   **Storage type:** HDD (o SSD si se prefiere, HDD es más económico).
*   **Storage capacity:** 10 GB.
*   **Enable automatic storage increases:** Desmarcar o marcar según preferencia (marcado en el tutorial).
*   **Automate backups:** Desmarcar para este tutorial para evitar costos (el resumen final lo muestra deshabilitado).

<!-- AQUÍ VA TU SCREENSHOT: Configuración de la instancia Cloud SQL -->
![image](https://github.com/user-attachments/assets/60895341-d72e-4968-a151-c9e3dcef42d9)


**D5.** Revisar el resumen y click en "Create Instance".

<!-- AQUÍ VA TU SCREENSHOT: Resumen de la instancia Cloud SQL antes de crear -->
![image](https://github.com/user-attachments/assets/e27b770b-16f6-4511-8a9d-3128f57e586a)


**D6.** Esperar a que la instancia de base de datos esté creada (puede tomar varios minutos).

<!-- AQUÍ VA TU SCREENSHOT: Instancia Cloud SQL creada en la lista -->
![image](https://github.com/user-attachments/assets/4a72ed9a-a4ea-452f-ac80-245f0f3103b9)



**D7.** Crear la base de datos para el proyecto dentro de la instancia SQL:
*   Ir a la instancia `laravelbd`.
*   Ir a la pestaña "Databases".
*   Click en "Create database".
*   **Database Name:** `djangoproject` (o el nombre que usará tu app).

<!-- AQUÍ VA TU SCREENSHOT: Creación de la base de datos 'djangoproject' -->
![image](https://github.com/user-attachments/assets/11c60278-498f-43e5-b364-fd1b2a3276ed)


**D8.** Configurar el acceso a la red:
*   Ir a la pestaña "Connections".
*   Ir a la sección "Networking".
*   Click en "Add Network".
*   **Name:** `all` (o un nombre descriptivo).
*   **Network:** `0.0.0.0/0` (Permite el acceso desde cualquier IP - **¡CUIDADO! No recomendado para producción**).
*   Click en "Done" y luego "Save".

<!-- AQUÍ VA TU SCREENSHOT: Configuración de red para permitir 0.0.0.0/0 -->
![image](https://github.com/user-attachments/assets/d1088e4e-4c14-4f63-bdfa-ed7ca8d27b35)


---

## E. Configurar proyecto Django con Docker

**E1.** Conectarse por SSH a la instancia Docker (`instance-docker`). Verificar que no hay contenedores corriendo (si es la primera vez):

```bash
sudo docker container ls
```

<!-- AQUÍ VA TU SCREENSHOT: Salida de 'sudo docker container ls' (probablemente vacía) -->
![image](https://github.com/user-attachments/assets/94793735-444c-45e0-a026-ef16b50b2132)


**E2.** Ejecutar el contenedor "hello-world" para verificar la instalación de Docker:

```bash
sudo docker container run hello-world
```

<!-- AQUÍ VA TU SCREENSHOT: Salida del comando 'hello-world' -->
![image](https://github.com/user-attachments/assets/bed8a67c-82e1-43d0-9c62-d834d31e22c1)


**E3.** Descargar el proyecto Django de ejemplo:

```bash
git clone https://github.com/Nram94/djangoDocker.git
```

<!-- AQUÍ VA TU SCREENSHOT: Salida del comando 'git clone' -->
![image](https://github.com/user-attachments/assets/b8985247-44ff-4e66-a43b-683bfc089246)


**E4.** Ubicarse en la carpeta del proyecto:

```bash
cd djangoDocker
ls
```

<!-- AQUÍ VA TU SCREENSHOT: Salida del comando 'ls' dentro de la carpeta djangoDocker -->
![Screenshot LS Proyecto](ruta/a/screenshot_ls_proyecto.png)

**E5.** Renombrar el archivo de ejemplo de variables de entorno:

```bash
sudo mv .env.example .env
```

**E6.** Modificar el archivo `.env` con los datos de la base de datos Cloud SQL:
*   Usar `nano` o `vim`: `sudo nano .env`
*   Encontrar la IP pública de la instancia Cloud SQL en la consola de GCP (Overview de la instancia SQL).
*   Reemplazar los valores:
    *   `DB_HOST`: IP pública de tu instancia Cloud SQL.
    *   `DB_DATABASE`: `djangoproject` (o el nombre que le diste).
    *   `DB_USERNAME`: `root` (usuario por defecto de Cloud SQL).
    *   `DB_PASSWORD`: La contraseña que estableciste para el usuario `root`.
*   Guardar los cambios (en `nano`: Ctrl+X, luego 'Y', luego Enter).

<!-- AQUÍ VA TU SCREENSHOT: Edición del archivo .env (mostrando las líneas modificadas) -->
![image](https://github.com/user-attachments/assets/7d787fab-f5db-4a74-8646-88ad14dd1593)

<!-- AQUÍ VA TU SCREENSHOT: Consola GCP mostrando la IP pública de Cloud SQL -->
![image](https://github.com/user-attachments/assets/28dae312-17fd-4484-9fb3-8cdbb837cd9b)


---

## F. Correr proyecto Django con Docker

**F1.** Verificar que existe el archivo `Dockerfile` en la raíz del proyecto:

```bash
ls
```

<!-- AQUÍ VA TU SCREENSHOT: Salida de 'ls' mostrando el Dockerfile -->
![image](https://github.com/user-attachments/assets/433d6304-4575-42a2-a6d1-fe1d3b30392d)


**F2.** Crear la imagen Docker para la aplicación Django:

```bash
# Asegúrate de estar en el directorio /djangoDocker
sudo docker image build -t django-app .
```

<!-- AQUÍ VA TU SCREENSHOT: Proceso de build de la imagen Docker -->
![image](https://github.com/user-attachments/assets/96f75242-34cb-4cbc-bb58-95c326e600a4)


**F3.** Desplegar un contenedor con la imagen creada:
*   `-d`: Correr en modo detached (background).
*   `--name django-docker`: Nombre del contenedor.
*   `-p 80:80`: Mapear el puerto 80 del host al puerto 80 del contenedor (el tutorial muestra 80:80 en la imagen final, aunque el texto dice 8000:8000. Usaremos 80:80 para que coincida con el acceso final).
*   `django-app`: Nombre de la imagen a usar.

```bash
sudo docker container run -d --name django-docker -p 80:80 django-app
```

<!-- AQUÍ VA TU SCREENSHOT: Ejecución del comando 'docker run' -->
![image](https://github.com/user-attachments/assets/38492283-2481-4a5f-8cd5-87b37ae30fa3)

<!-- AQUÍ VA TU SCREENSHOT: Salida de 'sudo docker ps' mostrando el contenedor corriendo -->
![image](https://github.com/user-attachments/assets/b913f414-f090-43e5-9b7c-3152937c3430)


**F4.** (Opcional/Alternativo según el tutorial) Lanzar la instancia usando Docker Compose (si el proyecto está configurado para ello):

```bash
# Este comando podría ser necesario si el proyecto usa docker-compose.yml
# sudo docker compose up -d
```
*(Nota: El paso F3 ya inicia el contenedor. `docker compose up` se usaría si hubiera un archivo `docker-compose.yml` para orquestar servicios, lo cual no se detalla explícitamente para este paso en el tutorial, pero se menciona)*.

**F5.** Acceder a la aplicación desde el navegador:
*   Obtener la IP externa de la instancia VM (`instance-docker`) desde la consola de GCP.
*   Abrir `http://EXTERNAL_IP_VM` (o `http://EXTERNAL_IP_VM:80`).

<!-- AQUÍ VA TU SCREENSHOT: Aplicación Django corriendo en el navegador -->
![image](https://github.com/user-attachments/assets/743debe7-631e-489d-8dba-e8f5abf1dbeb)


**(Opcional)** Registrarse y verificar que funciona.

<!-- AQUÍ VA TU SCREENSHOT: (Opcional) Aplicación después de registrarse/loguearse -->
![image](https://github.com/user-attachments/assets/b5d7c374-adf9-4374-a2df-034a58a17444)


---

## Pregunta Final

*   **¿Qué ventaja tiene utilizar Docker para desplegar un proyecto en GCP versus la forma inicial en la que lo hicimos?**

    <!-- AQUÍ VA TU RESPUESTA -->
    *Respuesta:* La principal ventaja de usar Docker es la **portabilidad y consistencia del entorno**. Con Docker, empaquetamos la aplicación y todas sus dependencias (librerías, versiones específicas de Python, etc.) en una imagen. Esta imagen se puede ejecutar de la misma manera en cualquier máquina que tenga Docker instalado (local, GCP, AWS, etc.), eliminando problemas de "funciona en mi máquina". Esto simplifica enormemente el despliegue, reduce errores de configuración entre entornos (desarrollo, pruebas, producción) y facilita la escalabilidad, ya que lanzar nuevas instancias de la aplicación es tan simple como ejecutar un nuevo contenedor a partir de la misma imagen. En contraste, la forma inicial (probablemente instalando todo directamente en la VM) es propensa a errores, dependiente de la configuración manual de la VM y más difícil de replicar consistentemente.

---

## Consideraciones Finales (Cloud SQL)

**¡OJO! "CLOUD SQL” CONSUME MUCHOS CRÉDITOS.**
*   **Acción:** Una vez terminado el taller, **detener** la instancia de Cloud SQL (`laravelbd`) para evitar gastos innecesarios. Ir a la instancia en la consola GCP y hacer click en "STOP".

<!-- AQUÍ VA TU SCREENSHOT: Instancia Cloud SQL detenida -->
![image](https://github.com/user-attachments/assets/b097b6b1-e638-436c-894a-824b387c5a85)


---

## Comandos Adicionales Útiles

*   **Borrar todo lo que Docker haya creado (contenedores parados, redes, imágenes no usadas, caché de build):**
    ```bash
    sudo docker system prune -a
    ```
*   **Borrar forzadamente el contenedor `django-docker`:**
    ```bash
    sudo docker container rm django-docker -f
    ```
*   **Conectarse a la terminal (bash) dentro del contenedor `django-docker`:**
    ```bash
    sudo docker exec -it django-docker bash
    ```

---

## Conclusiones

La realización de este tutorial permitió adquirir experiencia práctica en el despliegue de aplicaciones web modernas utilizando tecnologías clave como Django, Docker y Google Cloud Platform. Se comprendió el proceso de aprovisionamiento de infraestructura en la nube (VMs y bases de datos gestionadas), la importancia de la contenerización con Docker para asegurar entornos consistentes y la configuración necesaria para conectar una aplicación Django a una base de datos externa. El uso de Docker simplifica notablemente el proceso de despliegue y gestión de dependencias en comparación con métodos tradicionales. Fue crucial también aprender sobre la gestión de costos en la nube, como la necesidad de detener servicios como Cloud SQL cuando no están en uso activo.

---

## Autoria

*   **Realizado por:** Maximiliano Bustamante
*   **Programa:** Ingeniería de Sistemas
*   **Universidad:** Universidad EAFIT

---
```
