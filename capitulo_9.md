 Oracle Academy: Database Programing with SQL, Parte 9

* [Volver al inicio](index.html)

- [9.1 Uso de las Cláusulas Group By y Having](#91-uso-de-las-cláusulas-group-by-y-having)
  - [Cláusula GROUP BY](#cláusula-group-by)
  - [COUNT](#count)
  - [Cláusula WHERE](#cláusula-where)
  - [Grupos dentro de Grupos](#grupos-dentro-de-grupos)
  - [Anidamiento de Funciones de Grupo](#anidamiento-de-funciones-de-grupo)
  - [HAVING](#having)
- [9.2 Uso de las Operaciones Roullup y Cube, y Grouping Sets](#92-uso-de-las-operaciones-roullup-y-cube-y-grouping-sets)
  - [Fórmula de Resultado de ROLLUP](#fórmula-de-resultado-de-rollup)
  - [Sin ROLLUP](#sin-rollup)
  - [CUBE](#cube)
  - [GROUPING SETS](#grouping-sets)
  - [Funciones GROUPING](#funciones-grouping)
- [9.3 Uso de los Operadores SET](#93-uso-de-los-operadores-set)
  - [UNION](#union)
  - [INTERSECT](#intersect)
  - [MINUS](#minus)
  - [Ejemplos de Operadores SET](#ejemplos-de-operadores-set)

## 9.1 Uso de las Cláusulas Group By y Having

Existen casos en los que además de formar grupos con los datos también se
requiere clasificar esos grupo, por ejemplo, al intentar averiguar el promedio
de altura de los estudiantes de cada curso en una institución educativa. Este
problema podría resolverse utilizando la siguiente consulta:

~~~sql
SELECT AVG(height) FROM students WHERE year_in_school=:insert_year_in_school;
~~~

Pero, a parte de ser una sentencia que probablemente no funcionaría fuera de
Oracle APEX, habría que insertar cada curso por separado a mano. La otra opción
es crear una sentencia individual para cada `year_in_school`, pero eso es
engorroso. Por suerte las cláusulas `GROUP BY` y `HAVING` pueden utilizarse juntas
para manipular las funciones de grupo. Por ejemplo:

~~~sql
SELECT department_id, AVG(salary) FROM employees GROUP BY department_id ORDER
BY department_id;
~~~

La sentencia anterior calcula el promedio de salario de los empleados de cada
departamento **PERO AGRUPANDO TODOS LOS EMPLEADOS POR SU DEPARTAMENTO.** Esto
arrojaría la siguente respuesta:

|DEPARTMENT_ID|AVG(SALARY)|
|---|---|
|10|4400|
|20|9500|
|50|3500|
|60|6400|
|80|10033.33333333...|
|90|10033.33333333...|
|110|10150|
|-|7000|

**Lo que sucede es que, primero se agrupan todos los empleados por
`department_id`, y luego se le aplica la función `AVG` a cada grupo.**

Otro problema similar, averiguar cuál es el salario máximo entre los empleados
por cada departamento.

~~~sql
SELECT MAX(salary) FROM employees GROUP BY department_id;
~~~

|MAX(SALARY)|
|---|
|7000|
|24000|
|13000|
|...|

### Cláusula GROUP BY

¿Pero qué pasa si se necesita saber de qué departamento proviene cada salario
máximo? Eso es posible, sólo hay que incluir la columna `department_id` en la
cláusula `GROUP BY`.

~~~sql
SELECT department_id,MAX(salary) FROM employees GROUP BY department_id;
~~~

|DEPARTMENT_ID|MAX(SALARY)|
|---|---|
|-|7000|
|90|24000|
|20|13000|
|...|...|

Aunque hay una salvedad: **CUALQUIER COLUMNA QUE NO FORME PARTE DE UNA FUNCIÓN
DE GRUPO Y SE ENCUENTRE EN LA CLÁUSULA `SELECT` DEBE HACER PARTE DE LA CLÁUSULA
`GROUP BY`, en caso contrario se generará un error:**

~~~sql
-- la siguiente sentencia devuelve error por el las_name en la cláusula SELECT
SELECT job_id, last_name, AVG(salary) FROM employees GROUP BY job_id;
~~~

Otros ejemplos del uso de `GROUP BY`:

Redondear la media a un número entero:

~~~sql
SELECT region_id, ROUND(AVG(population)) AS population, FROM wf_countries GROUP
BY region_id ORDER BY region_id;
~~~

Cuente el número de idiomas hablados para todos los países:

~~~sql
SELECT country_id, COUNT(language_id) AS "Number of languages" FROM
wf_spoken_languages GROUP BY country_id;
~~~

### COUNT

En este ejemplo se muestra cuántos países hay en cada región.

~~~sql
SELECT COUNT(country_name), region_id FROM wf_countries GROUP BY region_id
ORDER BY region_id;
~~~

Esta consulta retorna la siguiente tabla:

|COUNT(COUNTRY_NAME)|REGION_ID|
|---|---|
|15|5|
|28|9|
|21|11|
|8|13|
|7|14|
|8|15|
|5|17|
|17|18|

Aunque por regla general (incluso si es poco probable en este caso) es mejor
utilizar `COUNT(*)` para eliminar cualquier posibilidad de encontrarse con
nulos.

~~~sql
SELECT COUNT(*), region_id FROM wf_countries GROUP BY region_id
ORDER BY region_id;
~~~

### Cláusula WHERE

También se pueden utilizar cláusulas WHERE para excluir las filas **antes de
que estas formen grupos. NO SE PUEDEN USAR CLÁUSULAS WHERE DESPUÉS DE UN `GROUP
BY`**

~~~sql
SELECT department_id, MAX(salary) FROM employees WHERE last_name!='King' GROUP
BY department_id;
~~~

Eso devolvería:

|DEPARTMENT_ID|MAX(SALARY)|
|---|---|
|-|7000|
|90|24000|
|20|13000|
|...|...|

### Grupos dentro de Grupos

A veces se debe dividir los grupos en subgrupos aún más pequeños, por ejemplo,
para agrupar todos los empleados por su departamento, y luego, dentro de cada
departamento, agruparlos por trabajo. Tal caso se puede apreciar en la
siguiente consulta:

~~~sql
-- nótese los dos argumentos de la cláusula GROUP BY.
-- primero está agrupando a todos los empleados por departamantos
-- luego, cuando estén agrupados, los subagrupa por trabajo
SELECT department_id, job_id, count(*) FROM employees WHERE department_id>40
GROUP BY department_id, job_id;
~~~

Dicha consulta devuelve:

|DEPARTMENT_ID|JOB_ID|COUNT(*)|
|---|---|---|
|110|AC_ACCOUNT|1|
|50|ST_CLERK|4|
|80|SA_REP|2|
|90|AD_VP|2|
|50|ST_MAN|1|
|...|...|...|

### Anidamiento de Funciones de Grupo

Las funciones de grupo se pueden anidar **EN UNA PROFUNDIDAD DE DOS** cuando se
utilice un `GROUP BY`. Por ejemplo:

~~~sql
-- máximo dos funciones de grupo anidadas a la vez
SELECT MAX(AVG(salary)) FROM employees GROUP BY department_id;
~~~

La consulta anterior devuelve un valor: el salario promedio más alto entre
todos los departamentos.

### HAVING

Existen escenarios donde se nota quea cláusula WHERE tiene sus limitaciones a
la hora de trabajar con grupos. Por ejemplo, a primera vista no parece difícil
resolver el problema de encontrar el salario más alto de cada departamento que
tenga más de un empleado, PERO **la siguiente sentencia NO es
posible realizarla:**

~~~sql
-- NO SE PUEDEN UTILIZAR FUNCIONES DE GRUPO EN UN WHERE CON UN GROUP BY
SELECT department_id, MAX(salary) FROM employees WHERE count(*)>1 GROUP BY
department_id;
-- arroja error
~~~

Por suerte existe una cláusua especial para restringir los grupos, `HAVING`. En
una consulta con una cláusula `GROUP BY` y `HAVING`, primero se crean los
grupos y luego se filtran. La forma correcta de realizar la consulta anterior,
sería:

~~~sql
SELECT department_id, MAX(salary) FROM employees GROUP BY department_id HAVING
COUNT(*)>1 ORDER BY department_id;
~~~

|DEPARTMENT_ID|MAX(SALARY)|
|---|---|
|20|13000|
|50|5800|
|60|9000|
|80|11000|
|90|24000|
|110|12000|

La siguiente consulta devuelve la población media de los países de cada región,
pero sólo de los grupos de regiones con una población superior a 300 mil:

~~~sql
SELECT region_id, ROUND(AVG(population)) FROM wf_countries GROUP BY region_id
HAVING MIN(population)>300000 ORDER BY region_id;
~~~

**Se recomienda seguir el siguiente orden a la hora de utilizar las cláusulas para
manipular grupos:**

~~~sql
SELECT colummn, group_function FROM table WHERE ... GROUP BY ... HAVING ...
ORDER BY ...;
~~~

## 9.2 Uso de las Operaciones Roullup y Cube, y Grouping Sets

En ocasiones, además de obtener los datos agrupados, se requiere realizar
cálculos con ellos, algunos de ellos incluso acumulativos. Por ejemplo,
evaluando el caso de la altura promedio de los estudiantes de una institución
por curso:

~~~sql
SELECT year_in_school, AVG(height) FROM students GROUP BY year_in_school;
~~~

¿Pero qué ocurre si, una vez que haya seleccionado sus grupos y calculado los
valores agregados en esos grupos, también deseara los subtotales por grupo y
una suma total de todas las filas seleccionadas?. Existe la posibilidad de
importar los resultados en otros métodos externos como una hoja de cálculo o
una calculadora. Pero convenientemente, Oracle Server cuenta con algunas
extensiones de la cláusula GROUP BY creadas específicamente con este fin:
`ROLLUP`, `CUBE` y `GROUPING SETS`.

En las consultas con `GROUP BY` a menudo se deben producir subtotales y
totales, eso puede hacerse con `ROLLUP`. Esta sentencia crea subtotales que se
acumulan desde el nivel más detallado hasta la suma total, siguiendo la lista
de agrupamiento especificada en la cláusula `GROUP BY`.

`ROLLUP` utiliza una lista ordenada de las columnas del agrupamiento en su
lista de argumentos. En primer lugar, calcula los valores de agregación
estándar especificados en la cláusula `GROUP BY`. A continuación crea subtotales
de nivel superior de forma progresiva, de derecha a izquierda  a través de la
lista de columnas del agrupamiento. Finalmente, crea la suma total.

Por ejemplo, la siguiente consulta:

~~~sql
SELECT department_id, job_id, SUM(salary) FROM employees WHERE department_id<50
GROUP BY ROLLUP(department_id, job_id);
~~~

Devuelve la siguiente tabla:

|DEPARTMENT_ID|JOB_ID|SUM(SALARY)|
|---|---|---|
|10|AD_ASST|4400|
|**10**|-|**4400**|
|20|MK_MAN|1300|
|20|MK_REP|6000|
|**20**|-|**19000**|
|-|-|**23400**|

Prestar atención a las filas que fueron resaltadas en la elaboración del
documento:

* La segunda fila de la tabla es el subtotal del `dept_id` 10
* La penúltima fila es el subtotal del `dept_id` 20
* La última fila es la suma total de ambos subtotales.

### Fórmula de Resultado de ROLLUP

El número de columnas o expresiones que aparecen en la lista de argumentos de
`ROLLUP` determina el número de agrupamientos. La fórmula es `(número de
columnas)+1` donde el número de columnas es la cantidad de argumentos pasados a
`ROLLUP`. En el ejemplo anterior, se le pasaron dos argumentos a `ROLLUP`,
`department_id` y `job_id`, por eso se generaron tres filas de resultado (la
segunda, la penúltima y la última).

### Sin ROLLUP

¿Qué aspecto hubiera tenido la consulta anterior, pero sin haber utilizado
ROLLUP?

~~~sql
SELECT department_id, job_id, SUM(salary) FROM employees WHERE department_id<50
GROUP BY (department_id, job_id);
~~~

La salida se vería así

|DEPARTMENT_ID|JOB_ID|SUM(SALARY)|
|---|---|---|
|10|AD_ASST|4400|
|20|MK_MAN|1300|
|20|MK_REP|6000|

Para obtener los subtotales de ROLLUP habría falta hacer varias consultas más
sobre esta tabla temporal.

### CUBE

`CUBE` es una extensión de la cláusula `GROUP BY`, que genera informes de
tabulación cruzada, que se puede aplicar a todas las funciones de agregación.
Las columnas especificadas en la cláusula `GROUP BY` están incluídas en
referencias cruzadas para crear un superjuego de grupos. Las funciones de
agregación especificadas en la lista `SELECT` se aplican a estos grupos para
crear valores de resumen para las filas superagregadas adicionales.

Todas las combinaciones posibles de filas se agregan con `CUBE`, si tiene n
columnas en la cláusula GROUP BY, habrá 2n posibles combinaciones de
superagregados. Matemáticamente, estas combinaciones forman un cubo de n
dimensiones, de ahí el nombre del operador. Esta cláusula se utiliza a menudo
en las consultas que usan columnas de tablas independientes.

Un ejemplo práctico sería consultar la tabla de ventas de una compañía como
Amazon. Un informe de tabulación cruzada normalmente solicitado podría incluis
subtotales para todas las combinaciones posibles de las ventas de un mes,
región y producto.

En la siguiente consulta, las filas resaltadas las genera la operación `CUBE`:

~~~sql
SELECT department_id, job_id, SUM(salary) FROM employees WHERE department_id<50
GROUP BY CUBE(department_id,job_id);
~~~

|DEPARMENT_ID|JOB_ID|SUM(SALARY)|
|---|---|---|
|-|-|**23400**|
|-|**MK_MAN**|**13000**|
|-|**MK_REP**|**60000**|
|-|**AD_ASST**|**4400**|
|**10**|-|**4400**|
|10|AD_ASST|4400|
|**20**|-|**19000**|
|20|MK_MAN|13000|
|20|MK_REP|6000|

Las filas resaltadas de la tabla representan:

* Primera fila: Total del informe
* Segunda fila: Subtotal de MK_MAN
* Tercera fila: Subtotal de MK_REP
* Cuarta fila: Subtotal de AD_ASST
* Quinta fila: Subtotal de departamento 10
* Septima fila: Subtotal de departamento 20:

### GROUPING SETS

Esta es otra extensión de la cláusula `GROUP BY`, se utiliza para especificar
varias agrupaciones de datos, lo que proporciona la funcionalidad de tener
varias cláusulas `GROUP BY` en la misma sentencia `SELECT`.

Si desea ver los datos de la tabla `EMPLOYEES` agrupados por `(department_id,manager_id)`
pero también por `(department_id,manager_id)`, y por `(job_id,manager_id)`,
esto se puede lograr con `GROUPING SETS` sin necesidad de reescribir la
sentencia con tres `GROUP BY` diferentes.

La siguiente sentencia hace uso de `GROUPING SETS` para resolver el problema
planteado anteriormente:

~~~sql
SELECT department_id, job_id, manager_id, SUM(salary) FROM employees WHERE
department_id<50 GROUP BY GROUPING SETS
((job_id, manager_id),(department_id, job_id),(department_id, manager_id));
~~~

|DEPARTMENT_ID|JOB_ID|MANAGER_ID|SUM(SALARY)|
|---|---|---|---|
|-|MK_MAN|100|1000|
|-|MK_MAN|201|6000|
|-|AD_ASST|101|4400|
|10|AD_ASST|-|4400|
|20|MK_MAN|-|13000|
|20|MK_REP|-|6000|
|10|-|101|19000|
|20|-|100|13000|
|20|-|201|6000|

* Las primeras tres filas don del grupo (job_id, manager_id)
* Las filas de la cuarta a la sexta son del grupo (department_id, job_id)
* Las filas de las séptima a la novena son de (department_id, manager_id)

### Funciones GROUPING

Al utilizar `ROLLUP` o `CUBE` para crear informes con subtotales muy a menudo
también tiene que poder saber qué filas de la salida son las reales y cuáles
son producidas por `ROLLUP` o `CUBE`. Esto es algo que se puede resolver con
`GROUPING`. Esta función sólo acepta una columna como argumento, y devuelve 1
para una fila que ha sido calculada o 0 para una que ha sido devuelta de la
BD.

~~~sql
SELECT department_id, job_id, SUM(salary)
  GROUPING(department_id) AS "Dept sub total",
  GROUPING(job_id) AS "Job sub total"
FROM employees WHERE department_id<50
GROUP BY CUBE(department_id,job_id);
~~~

|DEPARMENT_ID|JOB_ID|SUM(SALARY)|Dept sub total|Job sub total|
|---|---|---|---|---|
|-|-|**23400**|1|1|
|-|**MK_MAN**|**13000**|1|0|
|-|**MK_REP**|**60000**|1|0|
|-|**AD_ASST**|**4400**|1|0|
|**10**|-|**4400**|0|1|
|10|AD_ASST|4400|0|0|
|**20**|-|**19000**|0|1|
|20|MK_MAN|13000|0|0|
|20|MK_REP|6000|0|0|

## 9.3 Uso de los Operadores SET

Los operadores SET se utilizan para combinar los resultados de diferentes
sentencias SELECT en una sola salida de resultado.

A veces se desea una salida de más de una tabla. Unir dos tablas devuelve un juego de filas que cumple con los criterios de unión, pero a veces los criterios de unión no son suficiente. Los operadores `SET` abarcan más. Pueden devolver las filas que se han encontrado en varias sentencias `SELECT`, las que estén en una tabla y no en otra, o bien las filas comunes en ambas sentencias.

Estos tienen reglas específicas:

* **EL NÚMERO DE COLUMNAS Y LOS TIPOS DE DATOS DEBEN SER IDÉNTICOS en todas las sentencias SELECT utilizadas en la consulta, aunque NO TIENEN QUE LLAMARSE IGUAL.**
* **LOS NOMBRES DE COLUMNA de la salida SE TOMAN de los nombres de columna DEL PRIMER `SELECT`**

Para explicar los operadores `SET`, se utilizarán las siguiente listas:

~~~sql
A={a_id:1,2,3,4,5}
B={b_id:4,5,6,7,8}
~~~

### UNION

El operador `UNION` devuelve todas las filas de ambas tablas, después de eliminar los duplicados:

~~~sql
SELECT a_id FROM a UNION SELECT b_id FROM b;
-- devuelve "1,2,3,4,5,6,7,8"
~~~

Para devolver todas las filas coincidentes en ambas tablas sin eliminar los duplicados, hay que agregar el operador `ALL`:

~~~sql
SELECT a_id FROM a UNION ALL SELECT b_id FROM b;
-- devuelve "1,2,3,4,5,4,5,6,7,8" porque el 4 y el 5 son los elemntos
-- coincidentes en ambas tablas y aparecen una vez en cada tabla.
~~~

### INTERSECT

El operador `INTERSECT` devuelve todas las filas comunes a ambas tablas.

~~~sql
SELECT a_id FROM a INTERSECT SELECT b_id FROM b;
-- devuelve "4,5"
~~~

### MINUS

El operador `MINUS` devuelve todas las filas encontradas en la tabla de la
primera sentencia (de izquierda a derecha) pero no de la segunda.

~~~sql
SELECT a_id FROM a MINUS SELECT b_id FROM b;
-- el resultado sería 1,2,3. Los elementos de A que no están en B
~~~

Para obtener los elementos que están en B pero no en A, habría que consultar a
B primero:

~~~sql
SELECT b_id from B MINUS SELECT a_id FROM a;
-- retorna 6,7,8. Los valores de B que no están en A
~~~

### Ejemplos de Operadores SET

Considere la tabla employees que tiene una fecha de contratación, un ID de
empleado y un ID de cargo.

La tabla del historial de cargos contiene el ID del empleado y del cargo, pero
no tiene una columna con fecha de contratación.

A primera vista no se puede usar un `SET` con todo lo necesario, las dos tablas comparten ID del empleado e ID de cargo, pero el historial de cargos no tiene fecha de inicio.

Sin embargo, se puede utilizar la función `TO_CHAR(NULL)` para crear columnas coincidentes,
como en el siguiente ejemplo

~~~sql
SELECT hire_date, employee_id, job_id FROM employees
  UNION
SELECT TO_DATE(NULL), employee_id, job_id FROM job_history
ORDER BY employee_id;
~~~

El truco consiste en generar una columna artifical usando las funciones de conversión
de acuerdo con los tipos de datos de las columnas que se necesiten en la otra
tabla para que los `SET` puedan realizarse correctamente.

**Nótese que para ordenar toda la consulta sólo sigue haciendo falta colocar un
`ORDER BY` al final de toda la sentencia.**

~~~sql
SELECT hire_date, employee_id, TO_DATE(NULL) AS start_date, TO_DATE(NULL) AS
end_date, job_id, department_id FROM employees
  UNION
SELECT TO_DATE(NULL), employee_id, start_date, end_date, job_id, department_id
FROM job_history
ORDER BY employee_id;
~~~

El resultado final de esa sentencia sería:

|HIRE_DATE|EMPLOYEE_ID|START_DATE|END_DATE|JOB_ID|DEPARTMENT_ID|
|---|---|---|---|---|---|
|17-Jun-1987|100|-|-|AD_PRES|90|
|21-Sep-1989|101|-|-|AD_VP|90|
|-|101|21-Sep-1989|27-Oct-1993|AC_ACCOUNT|110|
|-|101|28-Oct-1993|15-Mar-1997|AC_MGR|110|
|13-Jan-1993|102|-|-|AD_VP|90|
|-|102|13-Jan-1993|24-Jul-1998|IT_PROG|60|
|03-Jan-1990|103|-|-|IT_PROG|60|
|21-May-1991|104|-|-|IT_PROG|60|
|07-Feb-1999|107|-|-|IT_PROG|60|
|-|114|24-Mar-1998|31-Dec-1999|ST_CLERK|50|
|-|122|01-Jan-1999|31-Dec-1999|ST_CLERK|50|
|16-Nov-1999|124|-|-|ST_MAN|50|
|17-Oct-1995|141|-|-|ST_CLERK|50|
|29-Jan-1997|142|-|-|ST_CLERK|50|
|15-Mar-1998|143|-|-|ST_CLERK|50|
|...|...|...|...|...|...|
|...|...|...|...|...|...|

