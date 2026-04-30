# Práctica: Comunicación entre Contenedores MySQL y phpMyAdmin mediante Redes Docker

---

## 1. Título

**Implementación de una Red Docker Personalizada para la Comunicación entre Contenedores MySQL y phpMyAdmin**

---

## 2. Tiempo de Duración

**Tiempo estimado:** 60 minutos

---

## 3. Fundamentos

Docker permite ejecutar múltiples servicios de forma aislada mediante contenedores. Sin embargo, para que dos o más contenedores puedan comunicarse entre sí, es necesario que compartan una **red Docker**. Por defecto, los contenedores no se ven entre ellos a menos que se conecten explícitamente a una red común.

Docker ofrece distintos tipos de redes. La más utilizada para comunicación entre contenedores es la red de tipo **bridge**, que actúa como un switch virtual interno. Cuando se crea una red personalizada de tipo bridge, Docker asigna a cada contenedor un nombre de host igual a su nombre de contenedor, lo que permite que los servicios se referencien entre sí por nombre en lugar de por dirección IP. Esto es especialmente útil para configurar herramientas como phpMyAdmin, que necesita conocer la dirección del servidor MySQL al que debe conectarse.

**MySQL** es uno de los sistemas de gestión de bases de datos relacionales más populares del mundo. Su imagen oficial en Docker Hub permite levantar un servidor de base de datos en segundos, configurando credenciales y base de datos inicial mediante variables de entorno.

**phpMyAdmin** es una herramienta web de administración para MySQL y MariaDB. Permite gestionar bases de datos, tablas, usuarios y ejecutar consultas SQL desde una interfaz gráfica en el navegador, sin necesidad de usar la línea de comandos. Al ejecutarse en un contenedor, phpMyAdmin necesita conocer el host del servidor MySQL; gracias a la red Docker personalizada, se puede indicar simplemente el nombre del contenedor MySQL como host.

La combinación de ambos servicios en contenedores conectados por una red personalizada es un patrón muy común en entornos de desarrollo, ya que permite tener un entorno completo de base de datos y administración funcionando de forma aislada, reproducible y sin instalar nada directamente en el sistema operativo anfitrión (Docker Inc., 2024).

---

## 4. Conocimientos Previos

Para realizar esta práctica el estudiante debe tener claros los siguientes temas:

- Comandos básicos de Linux / terminal: ejecución de comandos, uso de flags.
- Conceptos fundamentales de Docker: imágenes, contenedores y su ciclo de vida.
- Comandos básicos de Docker: `docker run`, `docker ps`, `docker stop`, `docker rm`.
- Manejo básico del navegador web para acceder a interfaces locales (`localhost`).
- Noción básica de redes: puertos, host, protocolo HTTP.
- Conceptos básicos de bases de datos: qué es una base de datos, una tabla y un registro.

---

## 5. Objetivos a Alcanzar

- Crear un contenedor MySQL configurando credenciales mediante variables de entorno.
- Crear un contenedor phpMyAdmin y enlazarlo al servidor MySQL.
- Crear una red personalizada Docker de tipo bridge para permitir la comunicación entre ambos contenedores.
- Conectar los dos contenedores a la red creada y verificar su comunicación.
- Administrar una base de datos de prueba desde la interfaz web de phpMyAdmin.

---

## 6. Equipo Necesario

| Recurso | Especificación |
|---|---|
| **Computador** | Windows 10/11, Linux (Ubuntu 20.04+) o macOS 12+ |
| **Docker Engine** | v24.0 o superior |
| **Docker Desktop** | v4.20 o superior (Windows / macOS) |
| **Navegador web** | Chrome, Firefox o Edge (versión reciente) |
| **Terminal** | Bash, Zsh o PowerShell 7+ |
| **Conexión a Internet** | Para descargar las imágenes desde Docker Hub |

---

## 7. Material de Apoyo

- 📘 [Documentación oficial de Docker — Networking](https://docs.docker.com/network/)
- 🐬 [Imagen oficial de MySQL en Docker Hub](https://hub.docker.com/_/mysql)
- 🛠️ [Imagen oficial de phpMyAdmin en Docker Hub](https://hub.docker.com/_/phpmyadmin)
- 🗒️ [Cheat Sheet de Docker](https://docs.docker.com/get-started/docker_cheatsheet.pdf)
- 📖 Guía de asignatura de Administración de Servidores.

---

## 8. Procedimiento

**Paso 1: Verificar que Docker está activo**

Confirmar que Docker está instalado y corriendo antes de iniciar:

```bash
docker --version
docker info
```

---

**Paso 2: Descargar las imágenes necesarias**

Descargar previamente las imágenes de MySQL y phpMyAdmin desde Docker Hub:

```bash
docker pull mysql:8.0
docker pull phpmyadmin:latest
```

---

**Paso 3: Crear la red personalizada**

Crear una red Docker de tipo bridge con el nombre `db_network`:

```bash
docker network create db_network
```

Verificar que la red fue creada:

```bash
docker network ls
```

---

**Paso 4: Crear el contenedor MySQL**

Levantar el contenedor de MySQL con las credenciales necesarias:

```bash
docker run -d \
  --name mysql_server \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=admin123 \
  -p 3306:3306 \
  mysql:8.0
```

| Variable de entorno | Descripción |
|---|---|
| `MYSQL_ROOT_PASSWORD` | Contraseña del usuario root de MySQL |
| `MYSQL_DATABASE` | Nombre de la base de datos inicial que se crea automáticamente |
| `MYSQL_USER` | Usuario adicional creado al iniciar |
| `MYSQL_PASSWORD` | Contraseña del usuario adicional |

---

**Paso 5: Crear el contenedor phpMyAdmin**

Levantar el contenedor de phpMyAdmin, indicando como host el nombre del contenedor MySQL:

```bash
docker run -d \
  --name phpmyadmin_server \
  -e PMA_HOST=mysql_server \
  -e PMA_PORT=3306 \
  -p 8080:80 \
  phpmyadmin:latest
```

> La variable `PMA_HOST` debe contener el **nombre del contenedor MySQL** (`mysql_server`), ya que dentro de la red Docker los contenedores se identifican por su nombre.

---

**Paso 6: Conectar ambos contenedores a la red `db_network`**

```bash
docker network connect db_network mysql_server
docker network connect db_network phpmyadmin_server
```

Verificar que ambos contenedores están conectados a la red:

```bash
docker network inspect db_network
```



**Paso 7: Acceder a phpMyAdmin desde el navegador**

Abrir el navegador e ingresar a:

```
http://localhost:8080
```
<img width="784" height="567" alt="php" src="https://github.com/user-attachments/assets/a58229d0-eb46-4004-b1c3-2e59912874a8" />




**Paso 8: Crear una base de datos de prueba desde phpMyAdmin**

Una vez dentro de la interfaz de phpMyAdmin:

1. Hacer clic en **"Nueva"** en el panel izquierdo.
2. Escribir el nombre `practica_db` en el campo de nombre de base de datos.
3. Seleccionar la cotejación `utf8mb4_unicode_ci`.
4. Hacer clic en **"Crear"**.
<img width="1366" height="633" alt="base" src="https://github.com/user-attachments/assets/4efc764a-cec2-4584-89c0-d4522c6a6786" />



**Paso 9: Crear una tabla y verificar la comunicación**

Dentro de `practica_db`, crear una tabla de prueba usando la pestaña **SQL**:

```sql
CREATE TABLE usuario (
    id       INT AUTO_INCREMENT PRIMARY KEY,
    nombre   VARCHAR(100) NOT NULL,
    correo   VARCHAR(100) NOT NULL
);

INSERT INTO usuario (nombre, correo) VALUES ('Ana Torres', 'ana@example.com');
INSERT INTO usuario (nombre, correo) VALUES ('Luis Mora',  'luis@example.com');

SELECT * FROM usuario;
```
<img width="1109" height="550" alt="data" src="https://github.com/user-attachments/assets/cdaaf748-6f18-4741-934c-5ead4b6b1acf" />


## 9. Resultados Esperados

Al finalizar la práctica se habrán obtenido los siguientes resultados:

- Dos contenedores en ejecución (`mysql_server` y `phpmyadmin_server`) visibles con `docker ps`.
- Una red personalizada `db_network` de tipo bridge con ambos contenedores conectados.
- Acceso funcional a la interfaz web de phpMyAdmin en `http://localhost:8080`.
- Base de datos `practica_db` creada desde la interfaz gráfica, con una tabla `usuario` y registros insertados.
- Comunicación verificada entre phpMyAdmin y MySQL gracias a la red Docker personalizada, sin necesidad de exponer el contenedor MySQL directamente al exterior.

---

## 10. Evidencias
<img width="605" height="323" alt="pgadmin" src="https://github.com/user-attachments/assets/9fdbf702-7625-41cf-9f18-9027beb38fa2" />
<img width="498" height="469" alt="phpxd" src="https://github.com/user-attachments/assets/aedd5e96-7d2c-45f9-b671-4b70aea33df5" />



## 11. Bibliografía

Docker Inc. (2024). *Networking overview*. Docker Documentation. https://docs.docker.com/network/

Docker Inc. (2024). *mysql - Official Image*. Docker Hub. https://hub.docker.com/_/mysql

Docker Inc. (2024). *phpmyadmin - Official Image*. Docker Hub. https://hub.docker.com/_/phpmyadmin

The phpMyAdmin Project. (2024). *phpMyAdmin documentation*. https://www.phpmyadmin.net/docs/

Oracle Corporation. (2024). *MySQL 8.0 Reference Manual*. https://dev.mysql.com/doc/refman/8.0/en/

Kane, S., & Matthias, K. (2023). *Docker: Up & running* (3rd ed.). O'Reilly Media. https://www.oreilly.com/library/view/docker-up/9781098131814/

---

*Documento elaborado con fines académicos — Administración de Servidores*
