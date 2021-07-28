Oracle Academy: Database Programing with SQL, Parte 8

* [Volver al inicio](index.html)

- [8.1 Funciones de Grupo](#81-funciones-de-grupo)
    - [MIN](#min)
    - [MAX](#max)
    - [SUM](#sum)
    - [AVG](#avg)
    - [COUNT](#count)
    - [VARIANCE](#variance)
    - [STDDEV](#stddev)
  - [Funciones de Grupo y NULL](#funciones-de-grupo-y-null)
  - [Reglas para Funciones de Grupo](#reglas-para-funciones-de-grupo)
- [8.2 COUNT, DISTINCT y NVL](#82-count-distinct-y-nvl)
  - [Valores COUNT Y NULL](#valores-count-y-null)
  - [COUNT All Rows](#count-all-rows)
  - [DISTINCT](#distinct)
  - [Uso de DISTINCT](#uso-de-distinct)
  - [DISTINCT y COUNT](#distinct-y-count)
  - [NVL](#nvl)

## 8.1 Funciones de Grupo

Oracle cuenta con siete funciones de grupo. Estas funciones permiten devolver
resultados únicos en función de un conjunto de filas o una tabla completa. Las
funciones de grupo funcionan en un juego de filas para proporcionar un
resultado por grupo. Existen casos de uso en los que es necesario hacer esto, como al momento de
calcular el promedio de edad en un aula de estudiantes.

Las funciones de grupo se utilizan en la cláusula `SELECT`:

~~~sql
SELECT column, group_function(column), ... FROM table WHERE condition GROUP
BY column;
~~~

Y estas son:

#### MIN

Retorna el valor mínimo entre un grupo de valores de cualquier tipo de dato.
Ejemplo:

~~~sql
SELECT MIN(life_expect_at_birth) FROM wf_countries;
-- Retorna 32,62

SELECT MIN(hire_date) FROM employees;
-- Retorna 17-Jun-1987

SELECT MIN(country_name) FROM wf_countries;
-- Retorna Anguilla, porque es la primera palabra en órden alfabético.
~~~

#### MAX

Retorna el valor máximo entre un grupo de valores de cualquier tipo de dato.

~~~sql
SELECT MAX(life_expect_at_birth) FROM wf_countries;
-- Retorna 83,51

SELECT MAX(hire_date) FROM employees;
-- Retorna 29-Jan-2000

SELECT MAX(country_name) FROM wf_countries;
-- Retorna Sáhara Occidental, porque es la última palabra en órden alfabético.
~~~

#### SUM

Returna el total o la suma de un grupo de valores numéricos.

~~~sql
SELECT SUM(area) FROM wf_countries WHERE region_id=29;
-- Da como resultado 241424

SELECT SUM(salary) FROM employees WHERE department_id=90;
-- Da como resultado 58000
~~~

#### AVG

Retorna el valor promedio o la media de un conjunto de valores numéricos.

~~~sql
SELECT AVG(area) FROM wf_countries WHERE region_id=29;
-- Da como resultado 9659,29

SELECT ROUND(AVG(salary),2) FROM employees WHERE department_id=90;
-- Da como resultado 19333,33
~~~

#### COUNT

Cuenta el número de filas pasadas como argumento.

#### VARIANCE

Se utiliza en columnas con datos numéricos para calcular la
difusión de datos en torno a la media. Por ejemplo, si la nota media para una
clase en la última prueba fue de 82% y las puntuaciones de los alumnos
oscilan entre el 40% y el 100%, la varianza sería mayor que si las notas
oscilaran entre un 78% y un 88%

~~~sql
SELECT ROUND(VARIANCE(life_expect_at_birth),4) FROM wf_countries;
-- Da 143,2394
~~~

#### STDDEV

Similar a la varianza, pero la desviacióon estándar mide la
difusión de los datos. Para dos juegos de datos con aproximadamente la misma
media, cunato mayor sea la difusión, mayor será la desviación estándar.

~~~sql
SELECT ROUND(STDDEV(life_expect_at_birth),4) FROM wf_countries;
-- Da 11,9683
~~~

**Las funciones de grupo NO PUEDEN UTILIZARSE EN LA CLÁUSULA `WHERE`, esto
arroja un error.**

### Funciones de Grupo y NULL

**Las funciones de grupo ignoran los valores `NULL`**. Por ejemplo:

~~~sql
SELECT AVG(commission_pct) FROM employees;
--Devuelve .2125
~~~

Pero si miramos la tabla `employees`:

|LAST_NAME|COMMISSION_PCT|
|---|---|
|King|-|
|Kochhar|-|
|De Haan|-|
|Whalen|-|
|Higgins|-|
|Gietz|-|
|Zlotkey|.2|
|Abel|.3|
|Taylor|.2|
|Grant|.15|
|Mourgos|-|
|...|...|

Sólo algunos empleados tienen comisión, los demás tienen valores nulos. Estos
valores no son tenidos en cuenta por `AVG` (es decir, sólo calculó el promedio
entre los que sí tenían comisión), esto puede ser un problema si se necesita precisión.

### Reglas para Funciones de Grupo

Más de una función de grupo se pueden usar a la vez en una cláusula `SELECT`:

~~~sql
SELECT MAX(salary) AS "Max", MIN(salary) AS "Min", MIN(employee_id) AS "Emp"
FROM employees WHERE department_id=60;
~~~

|Max|Min|Emp|
|---|---|---|
|9000|4200|103|

Las funciones de grupo ignoran valores nulos y no pueden utilizarse en la
cláusula `WHERE`. Sólo `MIN`, `MAX` y `COUNT` pueden usarse en cualquier tipo de dato. EL resto
sólo funciona en datos de tipo numérico.

## 8.2 COUNT, DISTINCT y NVL

Las funciones de grupo aceleran bastante operaciones de negocio que de otra
forma requerirían contar registros individuales de forma manual. `COUNT` es una función que permite determinar
**el número de valores no nulos** dentro de una columna:

~~~sql
SELECT COUNT(job_id) FROM employees;
-- Devuelve 20
~~~

### Valores COUNT Y NULL

Retomando el ejemlpo anterior sobre el campo `commission_pct` de la tabla
`employees`, se sabe que hay 20 empleados en la tabla `employees`, pero si
contamos usando `commission_pct`

~~~sql
SELECT COUNT(commission_pct) FROM employees;
-- Devuelve 4
~~~

Podemos ver que `COUNT` está ignorando a la gran mayoría de empleados que no tienen `commission_pct`.

### COUNT All Rows

Se puede contar absolutamente todas las filas de una tabla utilizando
`COUNT(*)`, ya que este considera el juego de columnas completo de toda la
fila, de las cuales tiene siempre debe haber mínimo un valor no nulo por
respeto a la integridad entidad.

La siguiente sentencia devuelve cuántos empleados fueron contratados antes del
1° de Enero de 1996:

~~~sql
SELECT COUNT(*) FROM employees WHERE hire_date<'01-Jan-1996';
-- Devuelve 9
~~~

### DISTINCT

La palabra clave `DISTINCT` se utiliza para devolver valores o combinaciones de
**NO DUPLICADOS** en una consulta. Por ejemplo:

~~~sql
SELECT DISTINCT job_id FROM employees;
~~~

Devuelve:

|JOB_ID|
|---|
|AC_ACCOUNT|
|AC_MGR|
|AD_ASST|
|AD_PRES|
|AD_VP|
|IT_PROG|
|MK_MAN|
|...|

Con un juego de columnas no duplicadas:

~~~sql
SELECT DISTINCT job_id, department_id FROM employees;
~~~

Devuelve:

|JOB_ID|DEPARTMENT_ID|
|---|---|
|IT_PROG|60|
|SA_REP|80|
|ST_MAN|50|
|AD_VP|90|
|AD_ASST|10|
|MK_MAN|20|
|MK_REP|20|
|SA_MAN|80|
|SA_REP|-|
|...||

Aunque realmente no existen duplicados entre la combinación de `job_id` y
`department_id`, incluso si cada columna individual tiene duplicados:

~~~sql
SELECT job_id, department_id FROM employees;
~~~

Devuelve:

|JOB_ID|DEPARTMENT_ID|
|---|---|
|IT_PROG|60|
|SA_REP|80|
|ST_MAN|50|
|AD_VP|90|
|AD_ASST|10|
|MK_MAN|20|
|MK_REP|20|
|SA_MAN|80|
|SA_REP|-|
|...||

### Uso de DISTINCT

**La función `DISTINCT` puede utilizarse con todas las funciones de grupo, de
tal forma que LAS FUNCIONES DE GRUPO SÓLO TENGAN EN CUENTA LOS VALORES NO
DUPLICADOS.**

Por ejemplo, las siguientes sentencias arrojan ambas resultados distintos:

~~~sql
SELECT SUM(salary) FROM employees WHERE department_id=90;
-- Da 58000

SELECT SUM(DISTINCT salary) FROM employees WHERE department_id=90;
-- Da 41000
~~~

Porque la segunda sentencia está ignorando los empleados que tengan el mismo
salario, sólo está sumando un valor de salario de los repetido.

### DISTINCT y COUNT

Al utilizar `DISTINCT` y `COUNT` en la misma cláusula, se obtiene **el número
de valores NO DUPLICADOS de una columna**

Por ejemplo:

~~~sql
SELECT COUNT(DISTINCT job_id) FROM employees;
-- Devuelve 12. Esto respondería la pregunta de ¿Cuántos trabajos diferentes
-- tienen asignados los empleados?

SELECT COUNT(job_id) FROM employees;
-- Devuelve 18. Esto respondería la pregunta de ¿Cuántos importes de salarios
-- diferentes se pagan a los empleados?
~~~

### NVL

En algunos casos es necesario incluir valores nulos en funciones de grupo, por
ejemplo, al determinar el número medio de pedidos de clientes servvidos cada
día en un restaurante, para saber cuánta comida pedir al mes. Puede que el
dueño decida que incluir los días en que el establecimiento está cerrado (ergo,
**no sirve comida a nadie**) genere un indicador más fiable. Esta sería la
consulta adecuada para esta situación:

~~~sql
-- Los dias que no se sirva nada se suman como un 0 al promedio.
SELECT AVG(NVL(customer_orders,0));
~~~

Un ejemplo con la tabla `employees`:

~~~sql
SELECT AVG(commission_pct) FROM employees;
-- Devuelve .2125

SELECT AVG(NVL(commission_pct,0)) FROM employees;
-- Devuelve .0425, porque los nulos son contados como 0 para calcular el promedio
~~~
