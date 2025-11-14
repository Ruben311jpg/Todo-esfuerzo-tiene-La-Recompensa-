---
title: "Seminario Asignatura"
subtitle: "Bases de Datos"
author: "Rubén Peña, Daniel Navajas, Andrés Carballo"
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
    id_paciente SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    edad INTEGER CHECK (edad >= 0), -- La edad no puede ser negativa
    sexo VARCHAR(10) CHECK (genero IN ('Masculino', 'Femenino')), -- Género debe ser uno de estos valores
    peso DECIMAL(5,2) CHECK (peso > 0), -- Peso debe ser mayor a 0
    ciudad VARCHAR(30) DEFAULT 'Burgos',
    consumo_promedio_ml/semana smallint CHECK (consumo_promedio_ml > 0),
    edad_comienzo_consumo smallint
);
")
```



Ahora agregamos los datos de los pacientes


``` r
# Insertar datos en 'pacientes'
dbExecute(con, "
INSERT INTO pacientes (nombre, edad, genero, peso, altura) 
VALUES 
    (71453722V, 1, 'Bud Abbott', 45, 'Masculino', 80.5, default, 300, 18),
    (71454398L, 2, 'Lou Costello', 30, 'Masculino', 65.3, 'Burgos', 250, 17),
    (71456349S, 3, 'Andrés Caraballo', 60, 'Femenino', 70.2, 'Melilla', 400, 13),
    (71309450C, 4, 'César Ausin' 20, 'Masculino', 68.2, default, 300, 16),
    (15368752Z, 5, 'Gonzalo Villacorta', 65, 'Masculino', 75.4, 'Palencia', 190, 18),
")

```

Mostramos la tabla


``` sql
SELECT * FROM pacientes; 
```


<div class="knitsql-table">


Table: 5 records

| DNI       | ID | Nombre             | Edad | Sexo      | Peso | Ciudad   | Consumo actual | Edad comienzo consumo |
| --------- | -- | ------------------ | ---- | --------- | ---- | -------- | -------------- | --------------------- |
| 71453722V | 1  | Bud Abbott         | 45   | Masculino | 80.5 | default  | 300            | 18                    |
| 71454398L | 2  | Lou Costello       | 30   | Masculino | 65.3 | Burgos   | 250            | 17                    |
| 71456349S | 3  | Andrés Caraballo   | 60   | Femenino  | 70.2 | Melilla  | 400            | 13                    |
| 71309450C | 4  | César Ausin        | 20   | Masculino | 68.2 | default  | 300            | 16                    |
| 15368752Z | 5  | Gonzalo Villacorta | 65   | Masculino | 75.4 | Palencia | 190            | 18                    |


</div>

#### **Tabla consultas**

Contendrá los registros de consultas médicas.


``` r
# Crear tabla 'consultas'
dbExecute(con, "
CREATE TABLE consultas (
    id_consulta SERIAL PRIMARY KEY,
    id_paciente INTEGER REFERENCES pacientes(id_paciente), -- Clave foránea a 'pacientes'
    fecha DATE NOT NULL,
    diagnostico VARCHAR(255),
    CONSTRAINT chk_fecha CHECK (fecha <= CURRENT_DATE) -- La fecha de consulta no puede ser futura
);
")
```

```
## [1] 0
```

Ahora agregamos valores a las tablas de consultas


``` r
# Insertar datos en 'consultas'
dbExecute(con, "
INSERT INTO consultas (id_paciente, fecha, diagnostico) 
VALUES 
    (1, '2024-01-10', 'Hipertensión'),
    (2, '2024-02-15', 'Diabetes Tipo 2'),
    (3, '2024-03-01', 'Insuficiencia Cardíaca');
")
```

```
## [1] 3
```

Mostramos la tabla


``` sql
SELECT * FROM consultas; 
```


<div class="knitsql-table">


Table: 3 records

|id_consulta | id_paciente|fecha      |diagnostico            |
|:-----------|-----------:|:----------|:----------------------|
|1           |           1|2024-01-10 |Hipertensión           |
|2           |           2|2024-02-15 |Diabetes Tipo 2        |
|3           |           3|2024-03-01 |Insuficiencia Cardíaca |

</div>

#### **Tabla tratamientos**

Contendrá los tratamientos que se administran a los pacientes.


``` r
# Crear tabla 'tratamientos'
dbExecute(con, "
CREATE TABLE tratamientos (
    id_tratamiento SERIAL PRIMARY KEY,
    id_consulta INTEGER REFERENCES consultas(id_consulta), -- Clave foránea a 'consultas'
    nombre_tratamiento VARCHAR(100) NOT NULL,
    duracion_dias INTEGER CHECK (duracion_dias > 0), -- La duración del tratamiento debe ser mayor a 0
    dosis_mg DECIMAL(5,2) CHECK (dosis_mg > 0) -- La dosis debe ser mayor a 0
);
")
```

```
## [1] 0
```

Ahora agregamos valores a la tabla de tratamientos


``` r
# Insertar datos en 'tratamientos'
dbExecute(con, "
INSERT INTO tratamientos (id_consulta, nombre_tratamiento, duracion_dias, dosis_mg) 
VALUES 
    (1, 'Enalapril', 30, 10.5),
    (2, 'Metformina', 60, 850.0),
    (3, 'Furosemida', 15, 40.0);
")
```

```
## [1] 3
```

Mostramos la tabla


``` sql
SELECT * FROM tratamientos; 
```


<div class="knitsql-table">


Table: 3 records

|id_tratamiento | id_consulta|nombre_tratamiento | duracion_dias| dosis_mg|
|:--------------|-----------:|:------------------|-------------:|--------:|
|1              |           1|Enalapril          |            30|     10.5|
|2              |           2|Metformina         |            60|    850.0|
|3              |           3|Furosemida         |            15|     40.0|

</div>

### Pregunta 1

1.  ¿Cuáles son los nombres de los pacientes y los tratamientos que están recibiendo?


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

2.  Cuántas consultas han sido diagnosticadas con cada tipo de diagnóstico?


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

3.  ¿Cuál es la dosis promedio de tratamiento que están recibiendo los pacientes en cada diagnóstico?


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
## [1] "consultas"    "pacientes"    "tratamientos"
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

