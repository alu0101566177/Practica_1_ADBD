# Informe práctica 1 ADBD

## 1. Creación de la base de datos

```sql
CREATE DATABASE biblioteca;
```

## 2. Gestión de usuarios

### 2a. Creación de usuarios

```sql
CREATE USER admin_biblio WITH PASSWORD '123';
-- Se le conceden todos los permisos sobre la base de datos
GRANT ALL PRIVILEGES ON DATABASE biblioteca TO admin_biblio;

CREATE USER usuario_biblio WITH PASSWORD '123';
-- Se le da permisos para acceder a la base de datos
-- y hacer consultas tipo select
GRANT CONNECT ON DATABASE biblioteca TO usuario_biblio;
GRANT USAGE ON SCHEMA public TO usuario_biblio;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO usuario_biblio;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO lectores;
```

### 2b. Creación del rol lectores

```sql
CREATE ROLE lectores;

GRANT CONNECT ON DATABASE biblioteca TO lectores;
GRANT USAGE ON SCHEMA public TO lectores;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO lectores;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO lectores;
```

### 2c. Asignar el rol lectores a usuario_biblio

```sql
GRANT lectores TO usuario_biblio;
```

### 2d. Comprobar usuarios creados en el sistema

```sql
SELECT rolname, rolsuper, rolcreaterole, rolcreatedb, rolcanlogin
FROM pg_roles;
```

Resultado:

```csv
"rolname","rolsuper","rolcreaterole","rolcreatedb","rolcanlogin"
"pg_database_owner",False,False,False,False
"pg_read_all_data",False,False,False,False
"pg_write_all_data",False,False,False,False
"pg_monitor",False,False,False,False
"pg_read_all_settings",False,False,False,False
"pg_read_all_stats",False,False,False,False
"pg_stat_scan_tables",False,False,False,False
"pg_read_server_files",False,False,False,False
"pg_write_server_files",False,False,False,False
"pg_execute_server_program",False,False,False,False
"pg_signal_backend",False,False,False,False
"pg_checkpoint",False,False,False,False
"pg_maintain",False,False,False,False
"pg_use_reserved_connections",False,False,False,False
"pg_create_subscription",False,False,False,False
"mi_usuario",True,True,True,True
"admin_biblio",False,False,False,True
"usuario_biblio",False,False,False,True
"lectores",False,False,False,False
```

### 2e. Editar la contraseña de usuario_biblio

```sql
ALTER USER usuario_biblio WITH PASSWORD 'nueva contraseña';
```

### 2f. Configurar permisos para que usuario_biblio no pueda eliminar registros

```sql
REVOKE DELETE ON ALL TABLES IN SCHEMA public FROM usuario_biblio;
REVOKE DELETE ON ALL TABLES IN SCHEMA public FROM lectores;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    REVOKE DELETE ON TABLES FROM lectores;
```

## 3. Creación de las tablas

```sql
-- Esquema de la base de datos biblioteca

CREATE TABLE autores (
  id_autor SERIAL PRIMARY KEY,
  nombre TEXT NOT NULL,
  nacionalidad TEXT
);

CREATE TABLE libros (
  id_libro SERIAL PRIMARY KEY,
  titulo TEXT NOT NULL,
  ano_publicacion INTEGER NOT NULL,
  id_autor INTEGER REFERENCES autores ON DELETE SET NULL -- Se marca como autor desconocido si se elimina la relacion
);

CREATE TABLE prestamos (
  id_prestamo SERIAL PRIMARY KEY,
  id_libro INTEGER NOT NULL REFERENCES libros ON DELETE CASCADE, -- Si se elimina un libro, tambien todos los prestamos relacionados a el
  fecha_prestamo DATE NOT NULL,
  fecha_devolucion DATE,
  usuario_prestatario TEXT NOT NULL -- Nombre del usuario
);
```

## 4. Inserción de datos de ejemplo

```sql
INSERT INTO autores (nombre, nacionalidad) VALUES
('Gabriel García Márquez', 'Colombiana'),
('Isabel Allende', 'Chilena'),
('Jorge Luis Borges', 'Argentina'),
('Mario Vargas Llosa', 'Peruana'),
('Julio Cortázar', 'Argentina');

INSERT INTO libros (titulo, ano_publicacion, id_autor) VALUES
('Cien años de soledad', 1967, 1),
('El amor en los tiempos del cólera', 1985, 1),
('La casa de los espíritus', 1982, 2),
('Paula', 1994, 2),
('Ficciones', 1944, 3),
('La ciudad y los perros', 1963, 4),
('Conversación en La Catedral', 1969, 4),
('Rayuela', 1963, 5);

INSERT INTO prestamos (id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario) VALUES
(1, '2025-09-01', '2025-09-15', 'Ana Pérez'),
(3, '2025-09-05', NULL, 'Carlos Gómez'), -- aún no devuelto
(5, '2025-09-07', '2025-09-20', 'Lucía Fernández'),
(7, '2025-09-10', NULL, 'Miguel Torres'), -- aún no devuelto
(8, '2025-09-12', '2025-09-22', 'Sofía Ramírez');
```

## 5. Consultas básicas

### 5a. Listar todos los libros con su autor correspondiente.

```sql
SELECT titulo, nombre
FROM libros NATURAL JOIN autores;
```
