---
Título: "Relaciones entre el Consumo de Alcohol, Enfermedades Hepáticas y Evaluación Psicológica"
Subtítulo: "Asignatura : Bases de Datos"
Autores: " Daniel Navajas, Rubén Peña, Andrés Carballo"
output: 
  html_document:
    keep_md: true
    self_contained: true
---
## Introducción

Se va a trabajar con tres **relaciones** asociadas al consumo de alcohol y sus enfermedades causadas en diversos pacientes. 

1.  **Tabla pacientes**: Contendrá los datos de los paciente incluyendo la cantidad media de consumo de alcohol y edad de inicio de consumo.
2.  **Tabla diagnostico_medico**: Contendrá los datos relativos a la enfermedad del paciente y la consulta realizada.
3.  **Tabla evaluacion_psicologica**: Contendrá el estado psicológico del paciente y las pruebas realizadas.

## Objetivos/Preguntas

A continuación se responderán las siguientes preguntas:

1.  ¿Con qué edad empezó a consumir cada paciente?
2.  ¿Cuántos pacientes fueron diagnosticados con las diferentes enfermedades?
3.  ¿Cuál es la puntuación promedio que obtienen los pacientes en cada evaluación?

## Metodología y Resultados

Código del seminario

### Creación de tablas

#### **Tabla pacientes**

Contendrá los datos de los pacientes.


``` r
# Crear tabla 'pacientes'
dbExecute(con, "
CREATE TABLE pacientes (
    DNI CHAR(9) UNIQUE,
    id_paciente SMALLINT PRIMARY KEY,
    nombre VARCHAR(100),
    edad INTEGER CHECK (edad >= 0), -- La edad no puede ser negativa
    sexo VARCHAR(10) CHECK (sexo IN ('Masculino', 'Femenino')), -- Género debe ser uno de estos valores
    peso DECIMAL(5,2) CHECK (peso > 0), -- Peso debe ser mayor a 0
    ciudad VARCHAR(30) DEFAULT 'Burgos',
    consumo_promedio_ml smallint CHECK (consumo_promedio_ml > 0), -- Consumo de alcohol medido en mililitros/semana
    edad_comienzo_consumo smallint CHECK (edad_comienzo_consumo <= edad) -- Que nadie comience a consumir después de su edad acutal
);
")
```



Ahora agregamos los datos de los pacientes


``` r
# Insertar datos en 'pacientes'
dbExecute(con, "
INSERT INTO pacientes (DNI, id_paciente, nombre, edad, sexo, peso, ciudad, consumo_promedio_ml, edad_comienzo_consumo) 
VALUES 
    ('71453722V', 1, 'Bud Abbott', 45, 'Masculino', 80.5, default, 300, 18),
    ('71454398L', 2, 'Lou Costello', 30, 'Masculino', 65.3, 'Burgos', 250, 17),
    ('71456349S', 3, 'Andrés Caraballo', 60, 'Femenino', 70.2, 'Melilla', 400, 13),
    ('71309450C', 4, 'César Ausin', 20, 'Masculino', 68.2, default, 300, 16),
    ('15368752Z', 5, 'Gonzalo Villacorta', 65, 'Masculino', 75.4, 'Palencia', 190, 18);
    ('71459876A', 6, 'María López',       27, 'Femenino',  62.0, 'Burgos',     120, 12),
    ('71451234B', 7, 'Javier Martín',     50, 'Masculino', 88.3, 'Valladolid', 500, 16),
    ('71452345C', 8, 'Lucía Sánchez',     22, 'Femenino',  55.1, 'Burgos',      80, 18);
    ('71453421D', 9,  'Sergio Ortega',     41, 'Masculino', 82.0,  'León',     220, 20),
    ('71455678E', 10, 'Elena Martínez',    33, 'Femenino',  59.5,  'Burgos',   150, 18);
")

```

Mostramos la tabla


``` sql
SELECT * FROM pacientes; 
```


<div class="knitsql-table">


Table: 5 records

| DNI       | id_paciente | Nombre             | Edad | Sexo      | Peso | Ciudad     | Consumo actual   | Edad inicio |
|-----------|-------------|--------------------|------|-----------|------|------------|------------------|-------------|
| 71453722V | 1           | Bud Abbott         | 45   | Masculino | 80.5 | Burgos     | 300              | 18          |
| 71454398L | 2           | Lou Costello       | 30   | Masculino | 65.3 | Burgos     | 250              | 17          |
| 71456349S | 3           | Andrés Caraballo   | 60   | Femenino  | 70.2 | Melilla    | 400              | 13          |
| 71309450C | 4           | César Ausin        | 20   | Masculino | 68.2 | Burgos     | 300              | 16          |
| 15368752Z | 5           | Gonzalo Villacorta | 65   | Masculino | 75.4 | Palencia   | 190              | 18          |
| 71459876A | 6           | María López        | 27   | Femenino  | 62.0 | Burgos     | 120              | 12          |
| 71451234B | 7           | Javier Martín      | 50   | Masculino | 88.3 | Valladolid | 500              | 16          |
| 71452345C | 8           | Lucía Sánchez      | 22   | Femenino  | 55.1 | Burgos     | 80               | 18          |
| 71453421D | 9           | Sergio Ortega      | 41   | Masculino | 82.0 | León       | 220              | 20          |
| 71455678E | 10          | Elena Martínez     | 33   | Femenino  | 59.5 | Burgos     | 150              | 18          |

</div>

#### **Tabla diagnostico_medico**


``` r
# Crear tabla 'diagnostico_medico'
dbExecute(con, "
CREATE TABLE diagnostico_medico(
    id_diagnostico smallint PRIMARY KEY,
    DNI CHAR(9) NOT NULL REFERENCES pacientes(dni),
    id_paciente SMALLINT REFERENCES pacientes(id_paciente), -- Clave ajena a 'pacientes'
    fecha DATE NOT NULL,
    enfermedad VARCHAR(40),
    hospital VARCHAR(40) DEFAULT 'HUBU',
    doctor VARCHAR(40),
    CONSTRAINT chk_fecha CHECK (fecha <= CURRENT_DATE) -- La fecha de consulta no puede ser futura
);
")
```

Ahora agregamos valores a las tablas de consultas


``` r
# Insertar datos en 'diagnostico_medico'
dbExecute(con, "
INSERT INTO diagnostico_medico (id_diagnostico, DNI, id_paciente, fecha, enfermedad, hospital, doctor) 
VALUES 
    (1, '71453722V', 1, '2024-01-10', 'Hepatitis', default, 'Shaun Murphy'),
    (2, '71454398L', 2, '2024-02-15', 'Cirrosis', HUBU, 'Manuel Iván Pérez'),
    (3, '71456349S', 3, '2024-03-01', 'Cirrosis', 'Hospital universitario La Paz', 'Manuel Iván Pérez'),
    (4, '15368752Z', 5, '2024-03-25', 'Pancreatitis', 'Río Carrión', 'Gregory House');
    (5, '71459876A', 6, '2024-04-10', 'Gastritis','12 de Octubre', 'Julia González'),
    (6, '71451234B', 7, '2024-02-28', 'Cirrosis','HUBU', 'Manuel Iván Pérez'),
    (7, '71451234B', 7, '2024-05-15', 'Encefalopatía hepática', 'HUBU', 'Manuel Iván Pérez'),
    (8, '71452345C', 8, '2024-06-01', 'Trastorno de ansiedad','12 de Octubre', 'Julia González'),
    (9,  '71453421D', 9,  '2024-07-15', 'Hígado graso alcohólico', 'HUBU', 'Julia González'),
    (10, '71455678E', 10, '2024-08-01', 'Gastritis alcohólica', default, 'Shaun Murphy');
")
```


Mostramos la tabla


``` sql
SELECT * FROM diagnostico_medico; 
```


<div class="knitsql-table">


Table:  records

| id_diagnostico | DNI        | id_paciente | Fecha       | Enfermedad                | Hospital                      | Doctor             |
|----------------|------------|-------------|-------------|---------------------------|-------------------------------|--------------------|
| 1              | 71453722V  | 1           | 2024-01-10  | Hepatitis                 | HUBU                          | Shaun Murphy       |
| 2              | 71454398L  | 2           | 2024-02-15  | Cirrosis                  | HUBU                          | Manuel Iván Pérez  |
| 3              | 71456349S  | 3           | 2024-03-01  | Cirrosis                  | Hospital universitario La Paz | Manuel Iván Pérez  |
| 4              | 15368752Z  | 5           | 2024-03-25  | Pancreatitis              | Río Carrión                   | Gregory House      |
| 5              | 71459876A  | 6           | 2024-04-10  | Gastritis                 | 12 de Octubre                 | Julia González     |
| 6              | 71451234B  | 7           | 2024-02-28  | Cirrosis                  | HUBU                          | Manuel Iván Pérez  |
| 7              | 71451234B  | 7           | 2024-05-15  | Encefalopatía hepática    | HUBU                          | Manuel Iván Pérez  |
| 8              | 71452345C  | 8           | 2024-06-01  | Trastorno de ansiedad     | 12 de Octubre                 | Julia González     |
| 9              | 71453421D  | 9           | 2024-07-15  | Hígado graso alcohólico   | HUBU                          | Julia González     |
| 10             | 71455678E  | 10          | 2024-08-01  | Gastritis alcohólica      | HUBU                          | Shaun Murphy       |



</div>

#### **Tabla evaluacion_psicologica**


``` r
# Crear tabla 'evaluacion_psicologica'
dbExecute(con, "
CREATE TABLE evaluacion_psicologica (
    id_evaluacion SMALLINT PRIMARY KEY,
    DNI CHAR(9) NOT NULL REFERENCES pacientes(dni),
    id_paciente SMALLINT REFERENCES pacientes(id_paciente), 
    fecha DATE NOT NULL,
    puntuacion INTEGER CHECK (puntuacion >= 0 and puntuacion <= 10),
    escala_usada VARCHAR(20) DEFAULT 'AUDIT',
    doctor VARCHAR(30),
    CONSTRAINT chk_fecha CHECK (fecha <= CURRENT_DATE) -- La fecha de consulta no puede ser futura
);
")
```

Ahora agregamos valores a la tabla de las evaluaciones


``` r
# Insertar datos en 'evaluacion_psicologica'
dbExecute(con, "
INSERT INTO evaluacion_psicologica (id_evaluacion, DNI, id_paciente, fecha, puntuacion, escala_usada, doctor) 
VALUES 
    (1,'71453722V', 1, '2024-01-18', 8, default, 'Judith Becker'),
    (2,'71454398L', 2, '2024-02-20', 6, 'AUDIT-C', 'Samuel Vicario'),
    (3,'71456349S', 3, '2024-03-10', 9, 'AUDIT', 'Patricio Estrella'),
    (4,'71309450C', 4, '2024-03-20', 5, 'AUDIT-C', 'Judith Becker'),
    (5,'15368752Z', 5, '2024-03-30', 7, default, 'Marlon');
    (6, '71459876A', 6, '2024-04-20', 4, 'AUDIT-C', 'Victoria Prieri'),
    (7, '71451234B', 7, '2024-03-05', 9, 'AUDIT', 'Judith Becker'),
    (8, '71451234B', 7, '2024-06-10', 8, 'AUDIT', 'Victoria Prieri'),
    (9, '71452345C', 8, '2024-06-10', 3, 'AUDIT-C', 'Patricio Estrella');
")
```

Mostramos la tabla


``` sql
SELECT * FROM evaluacion_psicologica; 
```


<div class="knitsql-table">


Table: 3 records

| id_evaluacion | DNI        | id_paciente | Fecha       | Puntuación | Escala usada | Doctor            |
|---------------|------------|-------------|-------------|------------|--------------|-------------------|
| 1             | 71453722V  | 1           | 2024-01-18  | 8          | AUDIT        | Judith Becker     |
| 2             | 71454398L  | 2           | 2024-02-20  | 6          | AUDIT-C      | Samuel Vicario    |
| 3             | 71456349S  | 3           | 2024-03-10  | 9          | AUDIT        | Patricio Estrella |
| 4             | 71309450C  | 4           | 2024-03-20  | 5          | AUDIT-C      | Judith Becker     |
| 5             | 15368752Z  | 5           | 2024-03-30  | 7          | AUDIT        | Marlon            |
| 6             | 71459876A  | 6           | 2024-04-20  | 4          | AUDIT-C      | Victoria Prieri   |
| 7             | 71451234B  | 7           | 2024-03-05  | 9          | AUDIT        | Judith Becker     |
| 8             | 71451234B  | 7           | 2024-06-10  | 8          | AUDIT        | Victoria Prieri   |
| 9             | 71452345C  | 8           | 2024-06-10  | 3          | AUDIT-C      | Patricio Estrella |


</div>

### Pregunta 1

1.  ¿Con qué edad empezó a consumir cada paciente?

``` sql
SELECT pacientes.nombre, tratamientos.nombre_tratamiento
FROM pacientes 
JOIN consultas ON pacientes.id_paciente = consultas.id_paciente
JOIN tratamientos ON consultas.id_consulta = tratamientos.id_consulta;
```


<div class="knitsql-table">


Table: 3 records

|nombre          |nombre_tratamiento |
|:---------------|:------------------|
|Juan Pérez      |Enalapril          |
|Ana López       |Metformina         |
|Carlos Martínez |Furosemida         |

</div>

### Pregunta 2

2.  ¿Cuántos pacientes fueron diagnosticados con las diferentes enfermedades?


``` sql
SELECT consultas.diagnostico, COUNT(consultas.id_consulta) AS total_consultas
FROM consultas 
GROUP BY consultas.diagnostico;
```


<div class="knitsql-table">


Table: 3 records

|diagnostico            | total_consultas|
|:----------------------|---------------:|
|Insuficiencia Cardíaca |               1|
|Hipertensión           |               1|
|Diabetes Tipo 2        |               1|

</div>

### Pregunta 3

3.  ¿Cuál es la puntuación promedio que obtienen los pacientes en cada evaluación?


``` sql
SELECT consultas.diagnostico, AVG(tratamientos.dosis_mg) AS dosis_promedio
FROM consultas 
JOIN tratamientos ON consultas.id_consulta = tratamientos.id_consulta
GROUP BY consultas.diagnostico;
```


<div class="knitsql-table">


Table: 3 records

|diagnostico            | dosis_promedio|
|:----------------------|--------------:|
|Insuficiencia Cardíaca |           40.0|
|Hipertensión           |           10.5|
|Diabetes Tipo 2        |          850.0|

</div>



# Finalizando el documento

Para finalizar el documento (y no entorpecer con las prácticas), es necesario crear un código que nos permita **borrar todas las tablas creadas**.


``` r
# Obtener la lista de todas las tablas en la base de datos
tables <- dbListTables(con)
print(tables)
```

```
## [1] "pacientes"    "diagnostico_medico"    "evaluacion_psicologica"
```

``` r
# Borrar todas las tablas
for (table in tables) {
  dbExecute(con, paste0("DROP TABLE IF EXISTS ", table, " CASCADE;"))
}

# Verificar que no queden tablas
tables_after <- dbListTables(con)
print(tables_after)
```

```
## character(0)
```

Finalmente, desconectamos la base de datos local.


``` r
# Cerrar la conexión
dbDisconnect(con)
```

