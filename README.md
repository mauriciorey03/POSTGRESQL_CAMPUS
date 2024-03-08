### CREACIÓN DE TABLAS  POSTGRESQL


```postgresql
CREATE TYPE sexo AS ENUM ('H','M');
CREATE TYPE tipo_persona AS ENUM ('Alumno','Profesor');
CREATE TYPE tipo_asignatura AS ENUM ('Básica','Obligatoria','Optativa');

CREATE TABLE IF NOT EXISTS departamento (
	id_departamento SERIAL PRIMARY KEY,
	nombre VARCHAR(50)
);

CREATE TABLE IF NOT EXISTS grado (
	id_grado SERIAL PRIMARY KEY,
	nombre VARCHAR(100)
);

CREATE TABLE IF NOT EXISTS persona (
	id_persona SERIAL PRIMARY KEY,
	nif VARCHAR(9),
	nombre VARCHAR (25),
	apellido1 VARCHAR(50),
	apellido2 VARCHAR(50),
	ciudad VARCHAR(25),
	direccion VARCHAR(50),
	telefono INTEGER,
	year_born DATE,
	sexo sexo,
	tipo_persona tipo_persona	
);

CREATE TABLE IF NOT EXISTS profesor (
	id_profesor INTEGER REFERENCES persona(id) UNIQUE,
	id_departamento INTEGER REFERENCES departamentos(id)
);


CREATE TABLE IF NOT EXISTS curso_escolar (
	id_curso_escolar SERIAL PRIMARY KEY,
	anyo_inicio DATE,
	anyo_fin DATE
);

CREATE TABLE IF NOT EXISTS asignatura (
	id_asignatura SERIAL PRIMARY KEY,
	nombre VARCHAR(100),
	creditos FLOAT,
	tipo_asignatura tipo_asignatura,
	curso INTEGER,
	cuatrimestre INTEGER,
	id_profesor INTEGER REFERENCES profesor(id_profesor),
	id_grado INTEGER REFERENCES grado(id)
);

CREATE TABLE IF NOT EXISTS alumno_se_matricula_asignatura (
	id_alumno INTEGER REFERENCES persona(id),
	id_asignatura INTEGER REFERENCES asignatura(id),
	id_curso_escolar INTEGER REFERENCES curso_escolar(id)
);
```


### CONSULTAS PROPUESTAS POSTGRESQL

1. Devuelve un listado con el primer apellido, segundo apellido y el nombre de todos los alumnos. El listado deberá estar ordenado alfabéticamente de menor a mayor por el primer apellido, segundo apellido y nombre.

    ```postgresql
    SELECT apellido1, apellido2, nombre
    FROM persona
    WHERE tipo = 'alumno'
    ORDER BY apellido1, apellido2
    ```

2. Averigua el nombre y los dos apellidos de los alumnos que **no** han dado de alta su número de teléfono en la base de datos.

    ```postgresql
    SELECT nombre, CONCAT(apellido1, " ", apellido2) apellido, telefono
    FROM persona
    WHERE tipo = 'alumno' AND telefono IS NULL
    ```

3. Devuelve el listado de los alumnos que nacieron en `1999`.

    ```sql
    SELECT nombre, apellido1, apellido2, fecha_nacimiento
    FROM persona
    WHERE YEAR_BORN(fecha_nacimiento) = 1999
    ```

4. Devuelve el listado de `profesores` que **no** han dado de alta su número de teléfono en la base de datos y además su nif termina en `K`.

    ```postgresql
    SELECT nombre, apellido1, nif, tipo, telefono
    FROM persona
    WHERE nif LIKE '%K' AND tipo = 'profesor' AND telefono IS NULL
    ```

5. Devuelve el listado de las asignaturas que se imparten en el primer cuatrimestre, en el tercer curso del grado que tiene el identificador `7`.

    ```postgresql
    SELECT nombre, cuatrimestre, curso, id_grado
    FROM asignatura
    WHERE cuatrimestre = 1 AND curso = 3 AND id_grado=7
    ```

6. Devuelve un listado con los datos de todas las **alumnas** que se han matriculado alguna vez en el `Grado en Ingeniería Informática (Plan 2015)`.

    ```postgresql
    SELECT DISTINCT P.nombre, P.aPellido1, P.sexo
    FROM persona P
    INNER JOIN alumno_se_matricula_asignatura MA ON P.id = MA.id_alumno
    INNER JOIN asignatura A ON A.id = MA.id_asignatura
    INNER JOIN grado G ON A.id_grado = G.id
    WHERE sexo = 'M' AND A.id_grado=4
    ```

7. Devuelve un listado con todas las asignaturas ofertadas en el `Grado en Ingeniería Informática (Plan 2015)`.

    ```postgresql
    SELECT nombre as "Asignaturas del Grado de Ing. Informática"
    FROM asignatura
    WHERE id_grado=4
    ```

8. Devuelve un listado de los `profesores` junto con el nombre del `departamento` al que están vinculados. El listado debe devolver cuatro columnas, `primer apellido, segundo apellido, nombre y nombre del departamento.` El resultado estará ordenado alfabéticamente de menor a mayor por los `apellidos y el nombre.`

    ```postgresql
    SELECT Pe.tipo, Pe.apellido1, Pe.apellido2, Pe.nombre, D.nombre
    FROM persona Pe
    INNER JOIN profesor Pro ON Pe.id = Pro.id_profesor
    INNER JOIN departamento D ON Pro.id_departamento = D.id_departamento
    WHERE tipo= 'profesor'
    ORDER BY Pe.apellido1, Pe.apellido2, Pe.nombre
    ```

9. Devuelve un listado con el nombre de las asignaturas, año de inicio y año de fin del curso escolar del alumno con nif `26902806M`.

    ```postgresql
    SELECT DISTINCT A.nombre, CE.anyo_inicio, CE.anyo_fin
    FROM asignatura A
    INNER JOIN alumno_se_matricula_asignatura MA ON A.id = MA.id_asignatura
    INNER JOIN curso_escolar CE ON CE.id = MA.id_curso_escolar
    INNER JOIN persona P ON P.id = MA.id_alumno
    WHERE nif = '26902806M'
    ```

10. Devuelve un listado con el nombre de todos los departamentos que tienen profesores que imparten alguna asignatura en el `Grado en Ingeniería Informática (Plan 2015)`.

     ```postgresql
     SELECT DISTINCT D.nombre "DEPARTAMENTOS", A.nombre ASIGNATURA, P.nombre PROFESOR
     FROM departamento D
     INNER JOIN profesor PRO ON D.id = PRO.id_departamento
     INNER JOIN asignatura A ON A.id_profesor = PRO.id_profesor
     INNER JOIN persona P ON P.id = PRO.id_profesor
     ```

11. Devuelve un listado con todos los alumnos que se han matriculado en alguna asignatura durante el curso escolar 2018/2019.

      ```postgresql
      SELECT DISTINCT P.nombre Alumno, A.nombre, CE.anyo_inicio
      FROM asignatura A
      INNER JOIN alumno_se_matricula_asignatura MA ON A.id = MA.id_asignatura
      INNER JOIN curso_escolar CE ON CE.id = MA.id_curso_escolar
      INNER JOIN persona P ON P.id = MA.id_alumno
      WHERE CE.anyo_inicio BETWEEN 2018 AND 2019
      ```

12. Devuelve un listado con los nombres de **todos** los profesores y los departamentos que tienen vinculados. El listado también debe mostrar aquellos profesores que no tienen ningún departamento asociado. El listado debe devolver cuatro columnas, nombre del departamento, primer apellido, segundo apellido y nombre del profesor. El resultado estará ordenado alfabéticamente de menor a mayor por el nombre del departamento, apellidos y el nombre.

      ```postgresql
      SELECT D.nombre as departamento,P.apellido1,P.apellido2,P.nombre FROM persona P
      INNER JOIN profesor PR on P.id=PR.id_profesor
      INNER JOIN departamento D on PR.id_departamento=D.id
      ORDER BY D.nombre,P.apellido1,P.apellido2,P.nombre ASC;
      ```

13. Devuelve un listado con los profesores que no están asociados a un departamento. Devuelve un listado con los departamentos que no tienen profesores asociados.

      ```postgresql
      SELECT P.apellido1, P.apellido2, P.nombre, P.tipo, D.nombre
      FROM persona P
      INNER JOIN profesor Pro ON P.id = Pro.id_profesor
      INNER JOIN departamento D ON Pro.id_departamento = D.id
      WHERE D.nombre IS NULL
      
      #-SEGUNDA PARTE
      
      SELECT D.nombre
      FROM departamento D
      WHERE D.id NOT IN 
      (SELECT D.id 
      FROM profesor P 
      INNER JOIN Departamento D ON P.id_departamento = D.id)
      
      ```

14. Devuelve un listado con los profesores que no imparten ninguna asignatura.

      ```postgresql
      SELECT CONCAT(P.nombre,' ', P.apellido1,' ', P.apellido2) AS nombre
      FROM persona P
      INNER JOIN profesor PR ON PR.id_profesor = P.id
      LEFT JOIN asignatura A ON A.id_profesor = PR.id_profesor
      WHERE A.id_profesor IS NULL
      ```

15. Devuelve un listado con las asignaturas que no tienen un profesor asignado.

      ```postgresql
      SELECT A.nombre "ASIGNATURAS SIN DOCENTE", P.id_profesor
      FROM asignatura A
      LEFT JOIN profesor P ON A.id_profesor = P.id_profesor
      WHERE P.id_profesor IS NULL
      ```

16. Devuelve un listado con todos los departamentos que tienen alguna asignatura que no se haya impartido en ningún curso escolar. El resultado debe mostrar el nombre del departamento y el nombre de la asignatura que no se haya impartido nunca.

      ```postgresql
      SELECT DISTINCT D.nombre, A.nombre
      FROM departamento D
      INNER JOIN profesor P ON P.id_profesor = D.id
      INNER JOIN asignatura A ON A.id_profesor = P.id_departamento
      LEFT JOIN alumno_se_matricula_asignatura MA ON A.id = MA.id_asignatura
      ```

17. Devuelve el número total de **alumnas** que hay.

      ```postgresql
      SELECT COUNT(sexo) "Cantidad de estudiantes (Mujeres)"
      FROM persona
      WHERE tipo = 'Alumno'
      ```

18. Calcula cuántos alumnos nacieron en `1999`.

      ```postgresql
      SELECT COUNT(nombre) "CANTIDAD DE PERSONAS NACIDAS EN 1999"
      FROM persona
      WHERE YEAR(fecha_nacimiento) = 1999
      ```

19. Calcula cuántos profesores hay en cada departamento. El resultado sólo debe mostrar dos columnas, una con el nombre del departamento y otra con el número de profesores que hay en ese departamento. El resultado sólo debe incluir los departamentos que tienen profesores asociados y deberá estar ordenado de mayor a menor por el número de profesores.

      ```postgresql
      SELECT D.nombre, count(P.id_profesor)
      FROM profesor P
      INNER JOIN departamento D ON D.id = P.id_departamento
      GROUP BY D.nombre
      ORDER BY COUNT(P.id_profesor) DESC
      ```

20. Devuelve un listado con todos los departamentos y el número de profesores que hay en cada uno de ellos. Tenga en cuenta que pueden existir departamentos que no tienen profesores asociados. Estos departamentos también tienen que aparecer en el listado.

      ```postgresql
      SELECT D.nombre, COUNT(P.id_departamento) AS 'Cantidad de profesores en cada dep'
      FROM departamento D LEFT JOIN profesor P ON P.id_departamento = D.id
      GROUP BY D.nombre
      ORDER BY count(P.id_departamento) DESC
      ```

21. Devuelve un listado con el nombre de todos los grados existentes en la base de datos y el número de asignaturas que tiene cada uno. Tenga en cuenta que pueden existir grados que no tienen asignaturas asociadas. Estos grados también tienen que aparecer en el listado. El resultado deberá estar ordenado de mayor a menor por el número de asignaturas.

       ```postgresql
       
       SELECT G.nombre, COUNT(A.id) AS 'Cantidad de Asignaturas'
       FROM grado G INNER JOIN asignatura A ON G.id = A.id_grado
       GROUP BY G.nombre
       ORDER BY COUNT(A.id) DESC
       ```

22. Devuelve un listado con el nombre de todos los grados existentes en la base de datos y el número de asignaturas que tiene cada uno, de los grados que tengan más de `40` asignaturas asociadas.

      ```postgresql
      SELECT G.nombre, COUNT(A.id) AS 'Cantidad de asignaturas'
      FROM grado G INNER JOIN asignatura A ON G.id = A.id_grado
      GROUP BY G.nombre
      HAVING COUNT(A.id) > 40
      ```

23. Devuelve un listado que muestre el nombre de los grados y la suma del número total de créditos que hay para cada tipo de asignatura. El resultado debe tener tres columnas: nombre del grado, tipo de asignatura y la suma de los créditos de todas las asignaturas que hay de ese tipo. Ordene el resultado de mayor a menor por el número total de crédidos.

      ```postgresql
      SELECT g.nombre, a.tipo, SUM(a.creditos) AS 'Credits'
      FROM grado g INNER JOIN asignatura a ON g.id = a.id_grado
      GROUP BY a.tipo, g.nombre
      ORDER BY SUM(a.creditos) DESC
      ```

24. Devuelve un listado que muestre cuántos alumnos se han matriculado de alguna asignatura en cada uno de los cursos escolares. El resultado deberá mostrar dos columnas, una columna con el año de inicio del curso escolar y otra con el número de alumnos matriculados.

      ```postgresql
      SELECT ce.anyo_inicio, COUNT(DISTINCT MA.id_alumno)
      FROM persona P INNER JOIN alumno_se_matricula_asignatura MA ON P.id = MA.id_alumno
      INNER JOIN asignatura asig ON asig.id = MA.id_asignatura
      INNER JOIN curso_escolar ce ON ce.id = MA.id_curso_escolar
      GROUP BY ce.anyo_inicio
      ```

25. Devuelve un listado con el número de asignaturas que imparte cada profesor. El listado debe tener en cuenta aquellos profesores que no imparten ninguna asignatura. El resultado mostrará cinco columnas: id, nombre, primer apellido, segundo apellido y número de asignaturas. El resultado estará ordenado de mayor a menor por el número de asignaturas.

      ```postgresql
      SELECT p.id, CONCAT(p.nombre, " ", p.apellido1, " ", p.apellido2) AS PROFESOR, COUNT(a.id_profesor) as "CANTIDAD ASIGNATURAS"
      FROM asignatura a RIGHT JOIN persona p ON a.id_profesor = p.id
      GROUP BY p.id
      ORDER BY 3 DESC
      ```

26. Devuelve todos los datos del alumno más joven.

      ```postgresql
      SELECT *
      FROM persona a 
      WHERE a.fecha_nacimiento = (SELECT MAX(a.fecha_nacimiento)
      FROM persona a)
      ```

27. Devuelve un listado con los profesores que no están asociados a un departamento.

      ```postgresql
      SELECT p.nombre, p.apellido1, p.apellido2
      FROM persona p 
      INNER JOIN profesor PRO ON p.id = PRO.id_profesor
      WHERE p.id_departamento NOT IN 
      (SELECT d.id FROM departamento d)
      ```

28. Devuelve un listado con los departamentos que no tienen profesores asociados.

      ```postgresql
      SELECT d.nombre
      FROM departamento d 
      WHERE d.id NOT IN (SELECT DISTINCT p.id_departamento 
      FROM profesor p)
      ```

29. Devuelve un listado con los profesores que tienen un departamento asociado y que no imparten ninguna asignatura.

      ```postgresql
      SELECT p.nombre, p.apellido1, p.apellido2
      FROM persona p 
      WHERE p.id_departamento IN (SELECT d.id FROM departamento d)
      AND NOT EXISTS (SELECT DISTINCT a.id_profesor FROM asignatura a
      WHERE a.id_profesor = p.id)
      ```

30. Devuelve un listado con las asignaturas que no tienen un profesor asignado.

      ```postgresql
      SELECT a.nombre
      FROM asignatura a 
      WHERE NOT EXISTS (SELECT p.id 
      FROM profesor p
      WHERE p.id = a.id_profesor)
      ```

31. Devuelve un listado con todos los departamentos que no han impartido asignaturas en ningún curso escolar.

     ```postgresql
     SELECT * 
     FROM departamento 
     WHERE id NOT IN 
     (SELECT id_departamento 
     FROM profesor p 
     WHERE EXISTS (SELECT id_profesor 
     FROM asignatura a
     WHERE p.id = a.id_profesor 
     AND NOT EXISTS (SELECT id_asignatura 
     FROM alumno_se_matricula_asignatura ma
     WHERE a.id = ma.id_asignatura) 
     ) 
     )
     ```