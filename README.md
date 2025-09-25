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

| rolname                     | rolsuper | rolcreaterole | rolcreatedb | rolcanlogin |
| --------------------------- | -------- | ------------- | ----------- | ----------- |
| pg_database_owner           | False    | False         | False       | False       |
| pg_read_all_data            | False    | False         | False       | False       |
| pg_write_all_data           | False    | False         | False       | False       |
| pg_monitor                  | False    | False         | False       | False       |
| pg_read_all_settings        | False    | False         | False       | False       |
| pg_read_all_stats           | False    | False         | False       | False       |
| pg_stat_scan_tables         | False    | False         | False       | False       |
| pg_read_server_files        | False    | False         | False       | False       |
| pg_write_server_files       | False    | False         | False       | False       |
| pg_execute_server_program   | False    | False         | False       | False       |
| pg_signal_backend           | False    | False         | False       | False       |
| pg_checkpoint               | False    | False         | False       | False       |
| pg_maintain                 | False    | False         | False       | False       |
| pg_use_reserved_connections | False    | False         | False       | False       |
| pg_create_subscription      | False    | False         | False       | False       |
| mi_usuario                  | True     | True          | True        | True        |
| admin_biblio                | False    | False         | False       | True        |
| usuario_biblio              | False    | False         | False       | True        |
| lectores                    | False    | False         | False       | False       |

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

Resultado:

| Título                            | Nombre                 |
| --------------------------------- | ---------------------- |
| Cien años de soledad              | Gabriel García Márquez |
| El amor en los tiempos del cólera | Gabriel García Márquez |
| La casa de los espíritus          | Isabel Allende         |
| Paula                             | Isabel Allende         |
| Ficciones                         | Jorge Luis Borges      |
| La ciudad y los perros            | Mario Vargas Llosa     |
| Conversación en La Catedral       | Mario Vargas Llosa     |
| Rayuela                           | Julio Cortázar         |

### 5b. Mostrar los préstamos que aún no tienen fecha de devolución.

```sql
SELECT *
FROM prestamos
WHERE fecha_devolucion IS NULL;
```

Resultado:

| id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario |
| ----------- | -------- | -------------- | ---------------- | ------------------- |
| 2           | 3        | 2025-09-05     | NULL             | Carlos Gómez        |
| 4           | 7        | 2025-09-10     | NULL             | Miguel Torres       |

### 5c. Obtener los autores que tienen más de un libro registrado.

```sql
SELECT nombre
FROM (
  SELECT id_autor
  FROM autores NATURAL JOIN libros
  GROUP BY id_autor
  HAVING COUNT(*) > 1)
 NATURAL JOIN autores;
```

Resultado:

| nombre                 |
| ---------------------- |
| Gabriel García Márquez |
| Isabel Allende         |
| Mario Vargas Llosa     |

## 6. Consultas con agregación

### 6a. Calcular el número total de préstamos realizados.

```sql
SELECT COUNT(*) num_prestamos
FROM prestamos;
```

Resultado:

| num_prestamos |
| ------------- |
| 5             |

### 6b. Obtener el número de libros prestados por cada usuario.

```sql
SELECT usuario_prestatario, COUNT(*)
FROM prestamos
GROUP BY usuario_prestatario;
```

Resultado:

| usuario_prestatario | count |
| ------------------- | ----- |
| Lucía Fernández     | 1     |
| Ana Pérez           | 1     |
| Miguel Torres       | 1     |
| Sofía Ramírez       | 1     |
| Carlos Gómez        | 1     |

## 7. Modificación de datos

### 7a. Actualizar la fecha de devolución de un préstamo pendiente.

```sql
UPDATE prestamos
SET fecha_devolucion = CURRENT_DATE
WHERE id_prestamo = 2;
```

Tabla modificada:

| id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario |
| ----------- | -------- | -------------- | ---------------- | ------------------- |
| 1           | 1        | 2025-09-01     | 2025-09-15       | Ana Pérez           |
| 3           | 5        | 2025-09-07     | 2025-09-20       | Lucía Fernández     |
| 4           | 7        | 2025-09-10     | NULL             | Miguel Torres       |
| 5           | 8        | 2025-09-12     | 2025-09-22       | Sofía Ramírez       |
| 2           | 3        | 2025-09-05     | 2025-09-24       | Carlos Gómez        |

### 7b. Eliminar un libro y comprobar el efecto en la tabla de préstamos (usar ON DELETE CASCADE o justificar el comportamiento).

```sql
DELETE FROM libros
WHERE id_libro=1;
```

Tabla modificada (prestamos):

| id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario |
| ----------- | -------- | -------------- | ---------------- | ------------------- |
| 3           | 5        | 2025-09-07     | 2025-09-20       | Lucía Fernández     |
| 4           | 7        | 2025-09-10     | NULL             | Miguel Torres       |
| 5           | 8        | 2025-09-12     | 2025-09-22       | Sofía Ramírez       |
| 2           | 3        | 2025-09-05     | 2025-09-24       | Carlos Gómez        |

## 8. Creación de vistas

### 8a. Crear una vista llamada vista_libros_prestados que muestre: título del libro, autor y nombre del prestatario.

```sql
CREATE VIEW vista_libros_prestados AS
SELECT titulo, nombre AS autor, usuario_prestatario AS prestatario
FROM prestamos NATURAL JOIN libros NATURAL JOIN autores;
```

### 8b. Conceder permisos de consulta sobre esta vista únicamente a usuario_biblio.

```sql
GRANT SELECT ON vista_libros_prestados TO usuario_biblio;
```

## 9. Funciones y consultas avanzadas

### 9a. Crear una función que reciba el nombre de un autor y devuelva todos los libros escritos por él.

```sql
CREATE OR REPLACE FUNCTION libros_de(autor TEXT)
RETURNS TABLE(
    id_libro INTEGER,
    titulo TEXT,
    ano_publicacion INTEGER
) AS $$
BEGIN
    RETURN QUERY
    SELECT l.id_libro, l.titulo, l.ano_publicacion
    FROM libros l
    WHERE l.id_autor IN (SELECT a.id_autor FROM autores a WHERE a.nombre = autor);
END;
$$ LANGUAGE plpgsql;
```

```sql
SELECT * FROM libros_de('Isabel Allende');
```

Resultado:

| id_libro | titulo                   | ano_publicacion |
| -------- | ------------------------ | --------------- |
| 3        | La casa de los espíritus | 1982            |
| 4        | Paula                    | 1994            |

### 9b. Crear una consulta que devuelva los tres libros más prestados.

Insertamos más prestamos de prueba:

```sql
INSERT INTO prestamos (id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario) VALUES
(3, '2025-09-01', '2025-09-15', 'Ana Pérez'),
(8, '2025-09-12', '2025-09-22', 'Sofía Ramírez');
```

```sql
SELECT *
FROM libros
WHERE id_libro IN (
  SELECT id_libro
  FROM prestamos
  GROUP BY id_libro
  ORDER BY COUNT(*) DESC
  LIMIT 3);
```

Resultado:

| id_libro | titulo                   | ano_publicacion | id_autor |
| -------- | ------------------------ | --------------- | -------- |
| 3        | La casa de los espíritus | 1982            | 2        |
| 8        | Rayuela                  | 1963            | 5        |
| 5        | Ficciones                | 1944            | 3        |

## 10. Exportación e importación de datos

### 10a. Exportar el contenido de la tabla libros a un archivo CSV.

```sql
COPY libros TO '/tmp/libros.csv' WITH CSV HEADER;
```

Resultado:

```
id_libro,titulo,ano_publicacion,id_autor
2,El amor en los tiempos del cólera,1985,1
3,La casa de los espíritus,1982,2
4,Paula,1994,2
5,Ficciones,1944,3
6,La ciudad y los perros,1963,4
7,Conversación en La Catedral,1969,4
8,Rayuela,1963,5
```

### 10b. Importar datos adicionales de autores desde un archivo CSV externo.

```sql
COPY autores(nombre, nacionalidad) FROM '/tmp/autores_adicionales.csv' WITH CSV HEADER;
```

Resultado:

| id_autor | nombre                 | nacionalidad   |
| -------- | ---------------------- | -------------- |
| 1        | Gabriel García Márquez | Colombiana     |
| 2        | Isabel Allende         | Chilena        |
| 3        | Jorge Luis Borges      | Argentina      |
| 4        | Mario Vargas Llosa     | Peruana        |
| 5        | Julio Cortázar         | Argentina      |
| 6        | Paulo Coelho           | Brasileña      |
| 7        | J.K. Rowling           | Británica      |
| 8        | Stephen King           | Estadounidense |
| 9        | Haruki Murakami        | Japonesa       |
| 10       | Toni Morrison          | Estadounidense |
| 11       | George Orwell          | Británico      |
| 12       | Virginia Woolf         | Británica      |
| 13       | Miguel de Cervantes    | Española       |
| 14       | Franz Kafka            | Checa          |
| 15       | Leo Tolstoy            | Rusa           |
