# Entorno de Desarrollo Docker LAMP Legacy (Symfony 1.4 / PHP 7.2)

Este repositorio contiene la configuración necesaria para levantar un entorno de desarrollo local dockerizado, diseñado específicamente para mantener proyectos legacy (como **Autogestion** y **eUNSTA**) en equipos modernos (incluyendo Mac con Apple Silicon M1/M2/M3).

## 🚀 Requisitos Previos

* [Docker Desktop](https://www.docker.com/products/docker-desktop) instalado y corriendo.
* (Opcional) Un disco externo si tienes poco espacio, ya que configuraremos los volúmenes para persistir datos fuera del contenedor.

## 🛠️ Instalación y Puesta en Marcha

### 1. Clonar el repositorio
Clona este repo en tu máquina o disco externo:
```bash
git clone <url-de-este-repo> entorno-lamp
cd entorno-lamp
```

### 2. Crear carpetas de trabajo
Como estas carpetas están ignoradas por git, debes crearlas manualmente:
```bash
mkdir www
mkdir mysql_data
```
* **www:** Aquí harás `git clone` de los proyectos (Autogestion, eUNSTA, etc.).
* **mysql_data:** Aquí se guardará la BD. Si borras esta carpeta, pierdes los datos.

### 3. Levantar el entorno
```bash
docker-compose up -d --build
```

### 4. Configurar tu archivo Hosts
Para acceder a los proyectos por nombre (ej: `alumnos.local`), edita tu archivo hosts:
* **Mac/Linux:** `sudo nano /etc/hosts`
* **Windows:** Ejecutar Notepad como Admin y abrir `C:\Windows\System32\drivers\etc\hosts`

Agrega:
```text
127.0.0.1 alumnos.local
127.0.0.1 mi-otro-proyecto.local
```

---

## ⚙️ Configuración de Proyectos (Symfony 1.4)

### 1. Conexión a Base de Datos (`databases.yml`)
**IMPORTANTE:** Dentro de Docker, `localhost` no es la base de datos. Debes usar el host **`db`**.

Edita `config/databases.yml` en tu proyecto:
```yaml
all:
  doctrine: # o propel
    param:
      dsn:      mysql:host=db;dbname=nombre_bd
      username: root
      password: root
```

### 2. Assets de Symfony (Fix visual)
Si ves el login sin estilos o errores 404 en `/sf/`, debes crear el enlace simbólico dentro del contenedor:

```bash
docker exec -it lamp_symfony_web bash
cd /var/www/html/TuProyecto/web
ln -s ../lib/vendor/symfony/data/web/sf sf
exit
```

### 3. Permisos y Caché
Si tienes errores 500 inexplicables, suele ser permisos o caché vieja. Ejecuta:
```bash
docker exec -it lamp_symfony_web bash -c "cd /var/www/html/TuProyecto && ./symfony cc && chmod -R 777 cache log"
```

---

## 🗄️ Base de Datos

### Credenciales
* **Host:** `127.0.0.1` (desde Workbench/DBeaver) o `db` (desde dentro de los proyectos PHP).
* **User:** `root`
* **Pass:** `root`
* **Puerto:** `3306`

### Importar un Backup (.sql)
Coloca tu archivo `.sql` en una ruta conocida y ejecuta:
```bash
# Ejemplo: Crear la BD e importar
docker exec -i lamp_symfony_db mysql -u root -proot -e "CREATE DATABASE IF NOT EXISTS nombre_bd;"
docker exec -i lamp_symfony_db mysql -u root -proot nombre_bd < /ruta/a/tu/backup.sql
```
*Nota: MySQL está configurado en modo `lower_case_table_names=1`, por lo que ignora mayúsculas/minúsculas en nombres de tablas, ideal para evitar conflictos entre Windows/Mac/Linux.*

---

## 🐛 Troubleshooting Común

* **Error "Connection Refused":** Revisa que en `databases.yml` el host sea `db` y no `localhost`.
* **Error "ONLY_FULL_GROUP_BY":** Ya está solucionado en el docker-compose con el flag `--sql-mode=""`.
* **Mac M1/M2:** El contenedor de BD corre emulado (`linux/amd64`), puede tardar unos segundos más en iniciar la primera vez.