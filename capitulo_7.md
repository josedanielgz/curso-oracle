Oracle Academy: Database Programing with SQL, Parte 7

* [Volver al inicio](index.html)

- [7.1 Union Igualitaria y Producto Cartesiano](#71-union-igualitaria-y-producto-cartesiano)
  - [Uniones Propiedad de Oracle](#uniones-propiedad-de-oracle)
  - [Unión Igualitaria](#unión-igualitaria)
  - [Unión de Producto Cartesiano](#unión-de-producto-cartesiano)
  - [Restricción de las Filas de unión](#restricción-de-las-filas-de-unión)
- [7.2 Uniones No Igualitarias y Uniones Externas de Oracle](#72-uniones-no-igualitarias-y-uniones-externas-de-oracle)
  - [Unión Externa](#unión-externa)

## 7.1 Union Igualitaria y Producto Cartesiano

La sección anterior mencionaba la forma en que las BBDD pueden devolver datos
de múltiples tablas utilizando la sintaxis del estándar ANSI/ISO SQL-99.
Originalmente Oracle sólo tenía disponible su propia sintáxis propietaria que
aún sigue viendo uso debido a la gran cantidad de BBDD antiguas que aún pueden
encontrarse en funcionamiento.

La siguiente es una comparativa de ambas versiones:

|Unión Propiedad de Oracle|Equivalente ANSI/ISO SQL-99: 1999|
|---|---|
|Producto castesiano|`CROSS JOIN`|
|Unión igualitaria|`NATURAL JOIN`, `USING`, `ON`|
|Unión no igualitaria|`ON` (si no se usa `=`)|

### Uniones Propiedad de Oracle

Para consultar datos de más de una tabla con la sintaxis propiedad de Oracle,
se puede usar una condición de unión en la cláusula `WHERE`:

~~~sql
SELECT table1.column, table2.column, FROM table1, table2 WHERE table1.column=table2.column;
~~~

**Algo similar a lo que ocurre con los aliases de tabla se puede realizar con
los nombres propios de las tablas mismas, donde se puede usar la sintaxis
`table.column` para eliminar ambigüedades entre columnas que puedan llamarse
igual entre tablas diferentes. Esto, o el uso de aliases, es una práctica
recomendable pues hace que las sentencias sean más fáciles de leer y acelera el
acceso a la BBDD. Esto se conoce como CUALIFICAR SUS COLUMNAS.**

### Unión Igualitaria

Algunas veces denominada unión "simple" o "interna", una unión igualitaria es
una unión de tablas que combina filas con los mismos valores para las columnas
especificadas. Esto sería equivalente en el estándar ANSI/ISO SQL-99 a usar
`NATURAL JOIN`, `USING` y `ON` (si la condición de unión utiliza `=`).
El siguiente ejemplo ilustra la forma en que funciona un unión igualitaria.

~~~sql
-- El qué se pide
SELECT employees.last_name, employees.job_id, jobs.job_title
-- El dónde está
FROM employees, jobs
-- El cómo se obtiene
WHERE employees.job_id=jobs.job_id;
;
~~~

Esto arroja:

|LAST_NAME|JOB_ID|JOB_TITLE|
|---|---|---|---|
|King|AD_PRES|President|
|Kochhar|AD_VP|Administration Vice President|
|De Haan|AD_VP|Administration Vice President|
|Higgins|AC_MGR|Accounting Manager|
|Gietz|AC_ACCOUNT|Public Accountant|
|Zlotkey|SA_MAN|Sales Manager|
|Abel|SA_REP|Sales Representative|
|...|...|...|

Otro ejemplo:

~~~sql
SELECT employees.last_name, departments.department_name FROM employees, d
epartments WHERE employees.department_id=departments.department_id;
~~~

|Whalen|Administration|
|Hartstein|Marketing|
|Fay|Marketing|
|Mourgos|Shipping|
|Rajs|Shipping|
|Davies|Shipping|
|Matos|Shipping|
|...|...|

Un tercer ejemplo de uniones igualitarias, **esta vez usando aliases:**

~~~sql
-- j es el alias de jobs, e el de employees
SELECT last_name, e.job_id, job_title FROM employees e, jobs.j WHERE
e.job_id=j.job_id AND department_id=80;
~~~

Que devuelve:

|LAST_NAME|JOB_ID|JOB_TITLE|
|---|---|---|
|Zlotkey|SA_MAN|Sales Manager|
|Abel|SA_REP|Sales Representative|
|Taylor|SA_REP|Sales Representative|

 **NO es necesario agregar el alias cuando los nombres de columnas no se duplican entre las dos tablas. LOS ALIASES DE TABLA DEBEN USARSE O SIMULTANEAMENTE EN LAS CLÁUSULAS `SELECT` Y `FROM` O EN NINGUNA DE LAS DOS. ASIGNAR UN ALIAS EN EL `FROM` SIN UTILIZARLO EN EL `SELECT` GENERA UN ERROR.**

### Unión de Producto Cartesiano

Si en la cláusula WHERE de dos tablas de una consulta de unión no se ha
especificado ninguna condición de unión o la condición de unión no válida,
Oracle Server devuelve el producto cartesiano de las dos tablas.

El producto cartesiano combina cada fila de la primera tabla con cada fila de
la segunda. Esto sería el equivalente de un `CROSS JOIN` del estándar ANSI/ISO
SQL-99. **Es importante comprobar que las condiciones estén bien planteadas para evitar los productos cartesianos.**

Por ejemplo, la siguiente sentencia regresará un producto cartesiano porque se ha omitido la condición de unión:

|LAST_NAME|DEPARTMENT_NAME|
|---|---|
|Abel|Administration|
|Davies|Administration|
|De Haan|Administration|
|Ernst|Administration|
|Fay|Administration|
|Fay|Administration|
|Gietz|Administration|
|Grant|Administration|
|Hartstein|Administration|
|Higgins|Administration|

`160 rows returned in 0.01 seconds.`

### Restricción de las Filas de unión

Al igual que con las consultas de una tabla, la cláusula `WHERE` puede utilizarse para limitar los registros devueltos en la unión de varias tablas mediante condiciones, esto se puede hacer usando los operadores lógicos como `AND` y `OR`. Tal es el caso del ejemplo de los aliases mencionado anteriormente

~~~sql
-- Nótese la sentencia AND department_id=80
SELECT last_name, e.job_id, job_title FROM employees e, jobs.j WHERE
e.job_id=j.job_id AND department_id=80;
~~~

Además, también se pueden unir más de dos tablas usando la sintaxis de unión, basta con enlistar las tablas correctamente en el `FROM` y cuidar el uso de los aliases y la integridad de las restricciones a evaluar. Por ejemplo, supongamos que necesitamos un informe de nuestros empleados y la ciudad donde está ubicado su departamento. Las tres tablas que necesitamos unir son `departments`, `employees` y `locations`.

~~~sql
SELECT last_name, city FROM employees e, departments d, locations l WHERE e.department_id=d.department_id AND d.location_id=l.location_id;
~~~

Esto devuelve:

|LAST_NAME|CITY|
|---|---|
|Hartstein|Toronto|
|Fay|Toronto|
|Zlotkey|Oxford|
|Abel|Oxford|
|...|...|

## 7.2 Uniones No Igualitarias y Uniones Externas de Oracle

La sintaxis propia de Oracle Server también posee la capacidad de tratar con
uniones no igualitarias de tablas, con las que puede ser posible tratar con
tablas que no tengan ningún campo en común entre sí. Por ejemplo, el caso de
los empleados y el grado al que pertenece su salario. Los salarios de un
empleado pueden clasificarse en grados, determinados por esta tabla:

|GRADE_LEVEL|LOWEST_SAL|HIGHEST_SAL|
|---|---|---|
|A|1000|2999|
|B|3000|5999|
|C|6000|9999|
|D|10000|14999|
|E|15000|24999|
|F|25000|40000|

Ya que no hay ninguna coincidencia entre `employees` y `job_grades`, hay que recurrir a los operadores diferentes a `=`. El que suele ser más eficiente es `BETWEEN... AND`, pero las variaciones de `<` y `>` también se pueden utilizar. Una unión no igualitaria equivale a un ANSI `JOIN ON` cuando no se utiliza `=`.

~~~sql
SELECT last_name, salary, grade_level, lowest_sal, highest_sal FROM employees, job_grades WHERE (salary BETWEEN lowest_sal AND highest_sal);

Esa consulta devolvería:

|LAST_NAME|SALARY|GRADE_LEVEL|LOWEST_SAL|HIGHEST_SAL|
|---|---|---|---|---|
|Vargas|2500|A|1000|2999|
|Matos|2600|A|1000|2999|
|Davies|3100|B|3000|5999|
|Rajs|3500|B|3000|5999|
|Lorentz|4200|B|3000|5999|
|Whalen|4400|B|3000|5999|
|Mourgos|5800|B|3000|5999|
|Fay|6000|C|6000|9999|
|...|...|...|...|...|

### Unión Externa

Una unión externa se utiliza para ver las filas que tengan un valor
correspondiente en otra tabla, más aquellas que no tengan correspondencia en la
otra. Para indicar qué tabla puede tener datos que faltan mediante la sintaxis
de unión de Oracle, se utiliza `(+)` después del nombre de la columna de la
tabla en la cláusula `WHERE` de la consulta.

Por ejemplo, la siguiente consulta devuelve los IDs de departamento y los nombres de departamento, tanto aqueñños que tengan departamento asignado como los que no.

~~~sql
SELECT e.last_name, d.department_id, d.department_name FROM employees e, departments d WHERE e.department_id=d.department_id(+);
~~~

Eso arrojaría:

|LAST_NAME|DEPT_ID|DEPT_NAME|
|---|---|---|---|
|Whalen|10|Administration|
|Harstein|20|Marketing|
|Fay|20|Marketing|
|Mourgos|50|Shopping|
|...|||
|Gietz|110|Accounting|
|-|190|Contracting|

**NO ES POSIBLE TENER EL EQUIVALENTE DE `FULL OUTER JOIN` EN LA SINTAXIS DE ORACLE, PORQUE UTILIZAR `(+)` EN AMBOS LADOS DE LA CONDICIÓN GENERA UN ERROR.**

Las siguientes son las variaciones válidas de la sintaxis

~~~sql
-- Equivale a un RIGHT OUTER JOIN
SELECT table1.column, table2.column FROM table1, table2 WHERE table1.column =
table2.column(+);

-- Equivale a un LEFT OUTER JOIN
SELECT table1.column, table2.column FROM table1, table2 WHERE table1.column(+) =
table2.column;
~~~

**ESTA SENTENCIA ARROJARÍA UN ERROR**

~~~sql
-- NO EXISTE UN EQUIVALENTE DEL FULL OUTER JOIN EN LA SINTAXIS DE ORACLE
SELECT table1.column, table2.column FROM table1, table2 WHERE table1.column(+)
= table2.column(+);
~~~
