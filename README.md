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

1.  **Tabla pacientes**: Contendrá los datos de los paciente incluyendo la cantidad media de consumo de alcohol y edad de inicio de consumo.
2.  **¿Cuál es el número total de diagnósticos, el número de diagnósticos relacionados con el alcohol y el número de diagnósticos no relacionados con el alcohol que presenta cada paciente?
3.  **¿Cuántas evaluaciones psicológicas tiene cada paciente, cuántas de ellas presentan puntuación alta, cuántas presentan puntuación baja y cuál es su puntuación media?
## Objetivos/Preguntas

## Metodología y Resultados

Código del seminario

### Creación de tablas

#### **Tabla pacientes**

Contendrá los datos de los pacientes.


``` r
# Crear tabla 'pacientes'
dbExecute(con, "
CREATE TABLE pacientes (
    DNI CHAR(9) UNIQUE NOT NULL,
    id_paciente SMALLINT PRIMARY KEY,
    nombre VARCHAR(100),
    edad INTEGER CHECK (edad >= 0), -- La edad no puede ser negativa
    sexo VARCHAR(10) CHECK (sexo IN ('Masculino', 'Femenino')), -- Género debe ser uno de estos valores
    peso DECIMAL(5,2) CHECK (peso > 0), -- Peso debe ser mayor a 0
    ciudad VARCHAR(30) DEFAULT 'Burgos',
    consumo_promedio_ml smallint CHECK (consumo_promedio_ml > 0), -- Consumo de alcohol medido en mililitros/semana
    edad_comienzo_consumo smallint CHECK (edad_comienzo_consumo <= edad), -- Que nadie comience a consumir después de su edad acutal
    CONSTRAINT ID_no_Asociado UNIQUE (dni, id_paciente)
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
    ('15368752Z', 5, 'Gonzalo Villacorta', 65, 'Masculino', 75.4, 'Palencia', 190, 18),
    ('71459876A', 6, 'María López',27, 'Femenino',62.0, 'Burgos',120, 12),
    ('71451234B', 7, 'Javier Martín',50, 'Masculino', 88.3, 'Valladolid', 500, 16),
    ('71452345C', 8, 'Lucía Sánchez',22, 'Femenino',  55.1, 'Burgos',80, 18),
    ('71453421D', 9,  'Sergio Ortega', 41, 'Masculino', 82.0,  'León', 220, 20),
    ('71455678E', 10, 'Elena Martínez',33, 'Femenino',  59.5,  'Burgos', 150, 18);
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
    DNI CHAR(9) NOT NULL,
    id_paciente SMALLINT NOT NULL,
    fecha DATE NOT NULL,
    enfermedad VARCHAR(40),
    hospital VARCHAR(40) DEFAULT 'HUBU',
    doctor VARCHAR(40),
    CONSTRAINT chk_fecha CHECK (fecha <= CURRENT_DATE), -- La fecha de consulta no puede ser futura
    FOREIGN KEY (dni, id_paciente)
      REFERENCES pacientes(dni, id_paciente)
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
    (2, '71454398L', 2, '2024-02-15', 'Cirrosis', 'HUBU', 'Manuel Iván Pérez'),
    (3, '71456349S', 3, '2024-03-01', 'Cirrosis', 'Hospital universitario La Paz', 'Manuel Iván Pérez'),
    (4, '15368752Z', 5, '2024-03-25', 'Pancreatitis', 'Río Carrión', 'Gregory House'),
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
    DNI CHAR(9) NOT NULL,
    id_paciente SMALLINT NOT NULL, 
    fecha DATE NOT NULL,
    puntuacion INTEGER CHECK (puntuacion >= 0 and puntuacion <= 10),
    escala_usada VARCHAR(20) DEFAULT 'AUDIT',
    doctor VARCHAR(30),
    CONSTRAINT chk_fecha_diag CHECK (fecha <= CURRENT_DATE), -- La fecha de consulta no puede ser futura
    FOREIGN KEY (dni, id_paciente)
      REFERENCES pacientes(dni, id_paciente)
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
    (5,'15368752Z', 5, '2024-03-30', 7, default, 'Marlon'),
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

1.  Consumo de riesgo por ciudad.
   
Se contará el número total de pacientes que hay en cada ciudad. Además se realaizará una clasificación entre pacientes los pacientes que son de riesgo y los que no. Esta
clasificación se realizará en función del consumo promedio en ml de cada paciente.
El resultado final mostrado en la relación será la cantidad de pacientes por cada ciudad y de entre ellos cuales son de riesgo y cuales no.

``` sql
SELECT ciudad,
  COUNT (*) as total_pacientes,
  COUNT (CASE
            WHEN consumo_promedio_ml >= 300 then 1 end) as pacientes_riesgo,
  COUNT (Case
            WHEN consumo_promedio_ml < 300 then 1 end) as pacientes_no_riesgo
FROM pacientes
GROUP BY ciudad
ORDER BY ciudad;
```


<div class="knitsql-table">


Table:

| ciudad     | total_pacientes | pacientes_riesgo | pacientes_no_riesgo  |
|:-----------|:----------------|:-----------------|:---------------------|
| Burgos     | 6               | 2                | 4                    |
| León       | 1               | 0                | 1                    |
| Melilla    | 1               | 1                | 0                    |
| Palencia   | 1               | 0                | 1                    |
| Valladolid | 1               | 1                | 0                    |



</div>

### Pregunta 2

2.  Diagnósticos alcohólicos vs no alcohólicos.

Se quieren clasificar los diagnósticos realizados a los pacientes en dos grupos. Diagnósticos propios del alcoholismo o diagnósticos que no tienen por qué ser propios del alcoholismo. Para cada paciente se contabilizarán el número total de diagnósticos, ya que un mismo paciente podrá ser diagnosticado más de una vez. Se clasificarán cada uno de esos diagnósticos de la forma anteriormente explicada.


``` sql
SELECT p.nombre,
  COUNT (*) as total_diagnosticos,
  COUNT (CASE
          WHEN d.enfermedad IN ('Cirrosis', 'Hepatitis', 'Pancreatitis', 'Gastritis alcohólica', 'Hígado graso alcohólico', 'Encefalopatía hepática'),
            then 1 END) AS total_alcoholicos,
  COUNT (CASE
          WHEN d.enfermedad IN ('Gastritis', 'Trastorno de asiedad')
            then 1 END) AS total_no_alcoholicos
FROM pacientes p
JOIN diagnostico_medico d
ON p.id_paciente = d.id_paciente 
GROUP BY p.id_paciente, p.nombre;
```


<div class="knitsql-table">

Table:

| nombre             | total_diagnosticos | total_alcoholicos | total_no_alcoholicos   |
|:-------------------|:-------------------|:-------------------|:----------------------|
| Sergio Ortega      | 1                  | 1                  | 0                     |
| Andrés Caraballo   | 1                  | 1                  | 0                     |
| Gonzalo Villacorta | 1                  | 1                  | 0                     |
| Elena Martínez     | 1                  | 1                  | 0                     |
| María López        | 1                  | 0                  | 1                     |
| Lou Costello       | 1                  | 1                  | 0                     |
| Javier Martín      | 2                  | 2                  | 0                     |
| Bud Abbott         | 1                  | 1                  | 0                     |
| Lucía Sánchez      | 1                  | 0                  | 0                     |


</div>

### Pregunta 3

3.  Evaluaciones "altas" vs "bajas" por paciente.

Para cada paciente que tenga al menos una evaluación psicológica se mostrará el nombre de dicho paciente, el número total de evaluaciones que tiene, cuantas de ellas tienen una puntuación "alta" y cuantas tienen una puntuación "baja". También se mostrará la media de las todas las puntuaciones de cada paciente.

``` sql
SELECT p.nombre,
	COUNT (*) as total_evaluaciones,
	COUNT (CASE
			WHEN e.puntuacion >= 8 THEN 1 END) AS total_altas,
	COUNT (CASE
			WHEN e.puntuacion <8 THEN 1 END) AS total_bajas,
	AVG(e.puntuacion) AS promedio_evaluaciones
		
FROM pacientes p
JOIN evaluacion_psicologica e
ON p.id_paciente = e.id_paciente
GROUP BY p.id_paciente, p.nombre
ORDER BY p.id_paciente;
```


<div class="knitsql-table">


Table:

| nombre             | total_evaluaciones | total_altas | total_bajas | promedio_evaluaciones  |
|:-------------------|:-------------------|:------------|:------------|:-----------------------|
| Bud Abbott         | 1                  | 1           | 0           | 8.0                    |
| Lou Costello       | 1                  | 0           | 1           | 6.0                    |
| Andrés Caraballo   | 1                  | 1           | 0           | 9.0                    |
| César Ausin        | 1                  | 0           | 1           | 5.0                    |
| Gonzalo Villacorta | 1                  | 1           | 0           | 7.0                    |
| María López        | 1                  | 0           | 1           | 4.0                    |
| Javier Martín      | 2                  | 2           | 0           | 8.5                    |
| Lucía Sánchez      | 1                  | 0           | 1           | 3.0                    |



</div>

# Esquema de la base de datos
![Esquema](https://github.com/user-attachments/assets/5859f7e3-7e3e-4cfd-b1ef-ea81f768b311)

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

