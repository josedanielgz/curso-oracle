Oracle Academy: Database Programing with SQL, Parte 2

* [Volver al inicio](index.html)

- [2.1: Columnas, Caracteres y Filas](#21-columnas-caracteres-y-filas)
  - [Comando `DESCRIBE`](#comando-describe)
  - [Operador de Concatenación](#operador-de-concatenación)
  - [Concatenación de Valores Literales](#concatenación-de-valores-literales)
  - [Eliminar filas duplicadas con `DISTINCT`](#eliminar-filas-duplicadas-con-distinct)
- [2.2 Limitación de Filas Seleccionadas](#22-limitación-de-filas-seleccionadas)
  - [Sentencia SELECT y WHERE](#sentencia-select-y-where)
  - [Operadores de comparación](#operadores-de-comparación)
  - [Cadenas de caracteres y fechas en la cláusula WHERE](#cadenas-de-caracteres-y-fechas-en-la-cláusula-where)
- [2.3 Operadores de Comparación](#23-operadores-de-comparación)
  - [Cláusula BETWEEN ... AND](#cláusula-between--and)
- [Cláusula IN](#cláusula-in)
- [Cláusula LIKE](#cláusula-like)
- [Condición IS [NOT] NULL](#condición-is-not-null)

## 2.1: Columnas, Caracteres y Filas

SQL cuenta con múltiples funciones incorporadas para facilitar el trabajo de listar grandes conjuntos anidados de datos y personalizar la salida, entre las cuales está la concatenación, el manejo de valores literales (como fechas, números y caracteres) y eliminar filas duplicadas.

### Comando `DESCRIBE`

También abreviado como `DESC` en algunos RDBMS, sirve para mostrar una descripción rápida de la estructura de las tablas: su nombre, columnas, tipos de datos de dichas columnas, las claves (primarias o ajenas) y las columnas que pueden contener valores nulos, entre otros detalles que son de utilidad a la hora de insertar nuevos datos en una tabla.

~~~sql
DESC <tabla>;
~~~

Por ejemplo

~~~sql
DESC departments;
~~~

|Table|Column|Data Type|Length|Precision|Scale|Primary Key| Nullable|Default|Comment|
|---|---|---|---|---|---|---|---|---|---| 
|DEPARTMENTS|DEPARTMENT_ID|NUMBER|-|4|0|1|-|-|-|
|-|DEPARTMENT_NAME|VARCHAR2|30|-|-|-|-|-|-|
|-|MANAGER_ID|NUMBER|-|6|0|-| |-|-|         
|-|LOCATION_ID|NUMBER|-|4|0|-| |-|-|

### Operador de Concatenación

La concatenación es el acto de enlazar o conectar elementos juntos en una serie. El símbolo de concatenación en este contexto son dos "líneas verticales" o `||`. Los valores a ambos lados del operador se combinan en una sola columna de salida. Se utiliza de la siguiente forma.

~~~sql
string 1 || string2 || string_n
~~~

El valor resultado de una concatenación es una cadena de caracteres.

En SQL, el operador de concatenación puede enlazar columnas a otras columnas, expresiones aritméticas o valores constantes para crear una expresión de caracteres. Un uso característico de esta operación es dar legibilidad a las salidas de las consultas, por ejemplo:

~~~sql
SELECT department_id||' '||department_name FROM departments;
~~~

|DEPARTMENT_ID\|\|''\|\|DEPARTMENT_NAME|
|---| 
|10 Administration|
|20 Marketing|
|50 Shipping|
|60 IT|
|...|

_Ejemplo de concatenación donde se presentan el código y nombre del departamento separados por un caracter en blanco o espacio._

Los aliases ayudan a reducir la verbosidad del operador de concatenación en la presentación final, para que los argumentos de `SELECT` no aparezcan en la cabecera de la columna:

~~~sql
SELECT department_id||' '||department_name AS "Department Info" FROM departments;
~~~

|Department Info|
|---| 
|10 Administration|
|20 Marketing|
|50 Shipping|
|60 IT|
|...|

_El ejemplo anterior pero utilizando el alias "Department Info"._

### Concatenación de Valores Literales

Un valor literal es un dato fijo, puede ser un carácter, número o fecha. Se puede utilizar valores literales en las sentencias SQL para elaborar frases o sentencias. Estos se incluyen en la cláusula `SELECT` con el operador de concatenación. Siempre se debe usar comillas simples para delimitar caracteres y fechas, aunque no con los números. Los valores literales se devuelven para cada fila regresada por la consulta.

Por ejemplo:

~~~sql
--- El espacio después y antes de la primera y segunda comilla
-- respectivamente sirve para hacer la presentación más uniforme,
-- de forma last_name y salary no queden pegados.
SELECT last_name || ' has a monthly salary of ' || salary ||'dollars.' AS "Pay" FROM employees;
~~~

Devuelve:

|PAY|
|---|
|King has a monthly salary of 24000 dollars|
|Kochhar has a monthly salary of 17000 dollars|
|De Haan has a monthly salary of 17000 dollars|
|Whalen has a monthly salary of 4400 dollars|
|Higgins has a monthly salary of 12000 dollars|
|Gietz has a monthly salary of 8300 dollars|

También pueden usarse valores numéricos como literales, por ejemplo:

~~~sql
SELECT last_name || ' has a ' || 1 || ' year salary of ' || salary*12 || ' dollars.' AS "Pay" FROM employees;
~~~

|PAY|
|---|
|King has a 1 year salary of 288000 dollars|
|Kochhar has a 1 year salary of 204000 dollars|
|De Haan has a 1 year salary of 204000 dollars|
|Whalen has a 1 year salary of 52800 dollars|
|Higgins has a 1 year salary of 144000 dollars|

### Eliminar filas duplicadas con `DISTINCT`

Se puede utilizar DISTINCT en caso de que necesitemos saber cuántas instancias únicas de algo existen. Por ejemplo, para saber la lista de departamentos en el sistema que tienen algún empleado:

~~~sql
SELECT department_id FROM employees;
~~~

La siguiente sentencia nos devolvería el código del departamente de **cada empleado**, porque a menos que se especifique, la salida de una consulta no elimina las filas duplicadas. Para hacer eso, hay que usar `DISTINCT`:

~~~sql
-- DISTINCT debe ubicarse directamente luego de la cláusula SELECT
SELECT DISTINCT department_id FROM employees;
~~~

El cualificador DISTINCT afecta a todas las columnas enumeradas y devuelve todas las combinaciones de columnas diferentes de las que devuelve la cláusula `SELECT`.

## 2.2 Limitación de Filas Seleccionadas

En SQL se puede delimitar la cantidad de información devuelta por una sentencia, lo cual es muy importante desde un punto de vista de un negocio; por ejemplo, es una manera de ahorrar recursos cuando sólo se necesitan algunos datos selectos de una consulta que normalmente devolvería millones de resultados.

### Sentencia SELECT y WHERE

La cláusula `WHERE` es una cláusula opcional que se agrega al final de las sentencias SQL de consulta, y permite delimitar el número de elementos devueltos por una consulta de acuerdo con una serie de condiciones:

~~~sql
SELECT *|{[DISTINCT] <columna> | <expresion> [AS] ...} FROM <tabla> [WHERE <condiciones>];
~~~

Las cláusulas `WHERE` incluyen una o más condiciones que deben cumplirse e, inmediatamente después, le sigue la cláusula `FROM` en una sentencia SQL.

**NOTA: No se pueden utilizar aliases en la cláusula WHERE**

Por ejemplo

~~~sql
--- Nótese el uso del operador de comparación '='
SELECT employee_id, first_name, last_name FROM employees WHERE employee_id = 101;
~~~

Devuelve:

|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|
|---|---|---|
|101|Neena|Kochhar|

### Operadores de comparación

En las cláusulas WHERE se pueden utilizar ciertos operadores de comparación para contrastar varias expresiones (un par por cada comparador utilizado). Estos son:

* = Igual que
* \> Mayor que
* \>= Mayor igual que
* < Menor que
* <= Menor igual que
* <> No es igual que (también sirven != o ^=)

Generalmente estos operadores son utilizados con valores numéricos, por ejemplo:

~~~sql
SELECT employee_id, last_name, department_id FROM employees WHERE department_id = 90;
~~~

Devuelve

|EMPLOYEE_ID|LAST_NAME|DEPARTMENT_ID|
|---|---|---|
|100|King|90|
|101|Kochhar|90|
|102|De Hann|90|

Sin embargo, también es posible utilizar fechas y cadenas de caracteres, siempre y cuando estén rodeadas de comillas simples `''`.

### Cadenas de caracteres y fechas en la cláusula WHERE

Las cadenas de caracteres son sensibles a mayúsculas y minúsculas cuando se utilizan en la cáusula `WHERE`.

Por ejemplo, en la siguiente consulta:

~~~sql
SELECT first_name, last_name FROM employees WHERE last_name = 'taylor';
~~~

Debido a que los apellidos de la tabla están capitalizados correctamente (o sea que la primera letra es mayúscula y el resto es minúscula), ningún resultado será resuelto.

Oracle SQL, al igual que otros RDBMS, cuenta con funciones internas para manipular caracteres que falicitan la tarea de lidiar con estos errores de sensibilidad, las cuales serán expuestas más adelante, como `UPPER`, `LOWER` e `INITCAP`.

Las fechas, salvo con la salvedad de que estén encerradas entre comillas y el formato sea válido (algo que se verá más adelante) pueden utilizarse con los operadores de comparación de la misma forma que los números.

Para recapitular, estos son algunos ejemplos del uso correcto de los operadores de comparación:

~~~sql
WHERE hire_date < '01-Jan-2000';
WHERE salary >=6000;
WHERE job_id = 'IT_PROG';
~~~

## 2.3 Operadores de Comparación

Las comparaciones son algo que se utilizan rutinariamente incluso en actividades cotidianas de las que no somos concientes, por lo que no es de sorprenderse que estas también existan en SQL. Con las condiciones de comparación, se pueden delimitar los datos devueltos por una sentencia de búsqueda a los que cumplan determinadas condiciones.
Además de los comparadores lógicos mencionados anteriormente (`>,<,=` y sus variaciones) SQL también ofrece las palabras reservadas `BETWEEN`, `IN` y `LIKE`.

### Cláusula BETWEEN ... AND

Una cláusula `BETWEEN` sirve para delimitar los valores devueltos por una consulta que estén dentro de un par de valores A y B, **incluyendo esos dos valores.**

Por ejemplo, la siguiente sentencia:

~~~sql
-- Nótese que el orden en que se enumeran A y B es importante, el
-- límite inferior siempre debe ir primero.
SELECT last_name, salary FROM employees WHERE salary BETWEEN 9000 AND 11000;
~~~

Devuelve los siguientes resultados:

|LAST_NAME|SALARY|
|---|---|
|Zlotkey|10500|
|Abel|11000|
|Hunold|9000|

Se puede decir que esta cláusula es una forma resumida de utilizar `>=` y `<=`, de la siguiente forma:

~~~sql
SELECT last_name, salary FROM employees WHERE salary >= 9000 AND salary <= 11000;
~~~

## Cláusula IN

Una cláusula `IN` (también conocida como condición de miembro) puede utilizarse para determinar si uno de los valores está incluído en un juego de valores específico, sean estos valores literales o resultados de otras consultas. Un ejemplos de su uso en la vida real podría ser determinar si los números de teléfono internacionales de una lista de persona pertenecen a un país específico, basándose en su código de llamada.

En el siguiente ejemplo práctico:

~~~sql
SELECT city, state_province, country_id FROM locations WHERE country_id IN('UK','CA');
~~~

Se retornan los siguientes valores:

|CITY|STATE_PROVINCE|COUNTRY_ID|
|---|---|---|  
|Toronto|Ontario|CA|
|Oxford|Oxford|UK|

## Cláusula LIKE

Esta cláusula es de utilidad a la hora de buscar coincidencias con caracteres, fechas o patrones de números incluso si sólo se tiene una parte del patrón de coincidencia. Esto tiene aplicación, por ejemplo, a la hora de encontrar los apellidos de una persona que empiecen por una letra específica.

**El operador `LIKE` goza de dos caracteres denominados comodines, que son `%` (cero o más caracteres) y `_` (un único caracater).** Por ejemplo, la siguiente sentencia:

~~~sql
SELECY last_name FROM employees WHERE last_name LIKE '_o%';
~~~ 

Retorna los siguientes valores

|LAST_NAME|
|---|
|Kochhar|
|Lorentz|
|Mourgos|

Esto es, los apellidos de los empleados cuyo apellido (valga la redundancia) tenga una letra o antecedida por una inicial, más un número indefinido (cero o más) de caracteres después de la o, que es lo que vendría a representar `_o%`.

**NOTA: Dado el caso especial en que el patrón de caracteres que esté intentando buscar contenga alguno de los comodines `_` o `%`, se debe "escapear" o indicar que dicho caracter debe identificarse como un caracter literal, esto se hace con una cláusula `ESCAPE`, la cual funciona de la siguiente manera:**

~~~sql
SELECT last_name, job_id FROM employees WHERE job_id LIKE '%\_R%' ESCAPE '\';
~~~

En este ejemplo, se está enviando un patrón de texto que contiene el caracter `_` y se está indicando que este debe interpretarse literalmente porque le antecede un caracter `\` (o sea que lo está "escapeando").

## Condición IS [NOT] NULL

Oracle, al igual que otros RDBMS, provee medidas para determinar y probar los valores nulos, los cuales son, recapitulando, valores desconocidos, sin asignar o que no se pueden aplicar. Los casos de uso donde se deben determinar no escasean. Por ejemplo, determinar el número de clientes que no hayan registrado un número de teléfono.

Mediante la condición `IS NULL` se puede probar si un campo alberga un valor nulo, de igual manera también existe una condición `IS NOT NULL` para comprobar si un dato está disponible (o sea, que no es nulo).
Por ejemplo:

~~~sql
SELECT last_name, manager_id FROM employees WHERE manager_id IS NULL;
~~~
Devuelve los datos de los empleados que no tengan un jefe.

|LAST_NAME|MANAGER_ID|
|---|---|
|King|-|

King es el presidente de la compañía, entonces no tiene jefe.

Y el siguiente ejemplo:

~~~sql
SELECT last_name, comission_pct FROM employees WHERE comission_pct IS NOT NULL;
~~~

Devuelve los siguientes resultados:

|LAST_NAME|COMISSION_PCT|
|---|---|
|Zlotkey|.2|  
|Abel|.3|  
|Taylor|.2|  
|Grant|.15|

Que son todos los empleados que tienen asignada un porcentaje de comisión.