- [15.1 Creación de Vistas](#151-creación-de-vistas)
  - [Ver](#ver)
  - [¿Por Qué utilizar Vistas?](#por-qué-utilizar-vistas)
  - [Creación de Vistas](#creación-de-vistas)
  - [Instrucciones para la Creación de Vistas](#instrucciones-para-la-creación-de-vistas)
  - [Funciones de CREATE VIEW](#funciones-de-create-view)
    - [Vista Simple](#vista-simple)
    - [Vista Compleja](#vista-compleja)
  - [Modificación de Vistas](#modificación-de-vistas)
- [15.2 Operaciones DML y Vistas](#152-operaciones-dml-y-vistas)
  - [Sentencias DML y Vistas](#sentencias-dml-y-vistas)
  - [Control de Vistas](#control-de-vistas)
    - [Vistas con la Opción CHECK](#vistas-con-la-opción-check)
    - [Vistas con READ ONLY](#vistas-con-read-only)
  - [Restricciones de DML](#restricciones-de-dml)
- [15.3 Gestión de Vistas](#153-gestión-de-vistas)
  - [Supresión de una Vista](#supresión-de-una-vista)
  - [Vistas en Línea](#vistas-en-línea)
  - [Análisis de N principales](#análisis-de-n-principales)

- [Volver al inicio](index.html)

## 15.1 Creación de Vistas

Manipular una base de datos adecuadamente requiere de un moderado nivel de
capacitación y conocimiento específico que no se traduce rápidamente en las necesidades inmediatas que tienen las organizaciones. Como DBA, sus prioridades deben estar en tareas específicas del dominio como determinar quiénes acceso a qué datos,
de qué forma recuperarlos y la integridad de la entrada de los datos, pero también tiene que afrontar el desafío de hacer
la operabilidad lo menos complicada posible para un interesado de la organización realizar las operaciones que necesite.
Las vistas son una representación virtual de tablas personalizadas para cumplir requisitos de usuario específicos.

### Ver

Una vista, como una tabla, es un objeto de la BD. Sin embargo,
las vistas no son tablas "reales", sino representaciones lógicas
de tablas existentes u otras vistas.
**Las vistas no contienen datos propios**, son como ventanas por
las que se pueden ver o cambiar los datos de las tablas.

**Las tablas sobre las que se crea una vista se denominan tablas base.** La vista es una consulta almacenada como una sentencia
`SELECT` en el diccionario de datos.

```sql
-- sentencia para crear la vista
CREATE VIEW view_employees AS
SELECT employee_id, first_name, last_name, email
FROM employees
WHERE employee_id BETWEEN 100 AND 124;

-- visualizando el contenido de la vista
SELECT * FROM view_employees;
```

| EMPLOYEE_ID | FIRST_NAME | LAST_NAME | EMAIL    |
| ----------- | ---------- | --------- | -------- |
| 100         | Steven     | King      | SKING    |
| 101         | Neena      | Kochhar   | NKOCHHAR |
| 102         | Lex        | De Haan   | LDEHAAN  |
| 124         | Kevin      | Mourgos   | KMOURGOS |
| 103         | Alexander  | Hunold    | AHUNOLD  |
| 104         | Bruce      | Ernst     | BERNST   |
| 107         | Diana      | Lorentz   | DLORENTZ |

### ¿Por Qué utilizar Vistas?

- Las vistas restringen el acceso a la tabla base porque la
  vista puede mostrar columnas selectivas de la tabla.

- Las vistas se pueden utilizar para reducir la complejidad de la
  ejecución de las consultas basadas en sentencias `SELECT` complicadas.

- Por ejemplo, el creador de la vista puede ocnstruir sentencias de unión que
  recuperen datos de varias tablas.

- El usuario de la vista no ve el código subyacente ni cómo crearlo, sino que
  mediante la vista interactúa con la BD con consultas simples.

- Las vistas pueden recuperar datos de varias tablas, proporcionando
  independencia de datos a los usuarios.

- Los usuarios pueden ver los mismos datos de distintas formas.

- Las vistas proporcionan a los grupos de usuarios acceso a los
  datos según unos permisos o criterios concretos.

### Creación de Vistas

Para crear una vista, debe embebir una subconsulta dentro de una
sentencia `CREATE VIEW` de la siguiente forma:

```sql
CREATE [OR REPLACE] [FORCE|NO FORCE]
VIEW view_name [(alias [, alias]...)]
AS subquery
[WITH CHECK OPTION [CONSTRAINT constraint]]
[WITH READ ONLY [CONSTRAINT constraint]]
```

Aquí un desgloce de las opciones posibles:

|                     |                                                                                                 |
| ------------------- | ----------------------------------------------------------------------------------------------- |
| `OR REPLACE`        | Vuelve a crear la vista si ya existe                                                            |
| `FORCE`             | Crea la vista independientemente de si existen o no las tablas base                             |
| view_name           | Nombre específico de la vista                                                                   |
| alias               | Un nombre para cada expresión seleccionada por la consulta de la vista                          |
| subquery            | Es una sentencia `SELECT` completa; puede utilizar alias para las columnas de la lista `SELECT` |
| `WITH CHECK OPTION` | Especifica que las filas son accesibles para la vista desúés de una inserción o actualización   |
| `CONSTRAINT`        | Nombre asignado a la restricción `CHECK OPTION`                                                 |
| `WITH READ ONLY`    | Garantiza que no se puede realizar ninguna operación DML en esta vista.                         |

Por ejemplo:

~~~sql
-- Creando la vista
CREATE OR REPLACE VIEW view_euro_countries AS
  SELECT country_id, region_id, country_name, capitol
  FROM wf_countries
  WHERE location LIKE '%Europe';

-- Mostrando los datos de la vista
SELECT * FROM view_euro_countries ORDER BY country_name;
~~~

Las sentencias anteriores devuelven el siguiente juego de datos:

|COUNTRY_ID|REGION_ID|COUNTRY_NAME|CAPITOL|
|---|---|---|---|
|22|155|Bailwick Of Guernsey|Saint Peter Port|
|203|155|Bailwick Of Jersey|Saint Helier|
|387|39|Bosnia and Herzegovina|Saravejo|
|420|151|Czech Republic|Prague|
|298|154|Faroe Islands|Torshavn|
|49|155|Federal Republic of Germany|Berlin|
|33|155|French Republic|Paris|
|...|...|...|...|

### Instrucciones para la Creación de Vistas

- **La subconsulta que defina la vista no puede contener ninguna sintaxis
  `SELECT` compleja**

- Por razones de rendimiento, la subconsulta que define la vista no debe
  contener ninguna cláusula `ORDER BY`, es mejor especificarla al recuperar los
  datos de la vista.

- Puede utilizar la opción `OR REPLACE` para cambiar la definición de la vista
  sin tener que borrarla o volver a otorgarle privilegios previos.

- Se pueden utilizar alias para los nombres de columna en la subconsulta.

### Funciones de CREATE VIEW

Existen dos clases de vistas: simples y complejas. En la tabla se resumen las
funciones de cada vista:

|Función|Vistas Simples|Vistas Complejas|
|---|---|---|
|Número de tablas utilizadas para derivar datos|Una|Una o más|
|Puede contener funciones|No|Sí|
|Puede contener grupos de datos|No|Sí|
|Puede realizar operaciones DML en una vista|Sí|No siempre|

#### Vista Simple

La siguiente es una demostración de cómo crear una vista simple.
La subconsulta deriva datos a partir **DE UNA ÚNICA TABLA** y **NO CONTIENE NINGUNA FUNCIÓN DE UNIÓN NI NINGUNA FUNCIÓN DE GRUPO.**

**Puesto que es una vista simple, una operación DML que afectan la tabla base
se podrían realizar en la vista.**

~~~sql
CREATE OR REPLACE VIEW view_euro_countries AS
  SELECT country_id, region_id, country_name, capitol
  FROM wf_countries
  WHERE location LIKE '%Europe';
~~~

Los nombres de columna en la sentencia `SELECT` pueden tener
alias como se muestra a continuación.

Tenga en cuenta que los alias también se pueden enumerar después
de la sentencia `CREATE VIEW` y antes de la subconsulta `SELECT`:

~~~sql
-- Primera forma de utilizar aliases en una vista
CREATE OR REPLACE VIEW view_euro_countries AS
  SELECT country_id AS "ID", country_name AS "Country",
    capitol AS "Capitol City"
  FROM wf_countries
  WHERE location LIKE '%Europe'
  
-- Segunda forma de utilizar aliases en una vista
CREATE OR REPLACE VIEW 
view_euro_countries("ID","Country","Capitol City")
AS SELECT country_id, region_id, country_name, capitol
  FROM wf_countries
  WHERE location LIKE '%Europe';
~~~

Es posible crear una vista independientemente de si existen o 
no las tablas base. La palabra `FORCE` obliga a `CREATE VIEW` a
crear una vista aunque no sea válida. Esto puede ser útil 
para la fase de desarrollo de una BD, especialmente si está 
esperando que en breve se le otorguen los priviliegios necesarios
para el objeto al que se hace referencia.

#### Vista Compleja

Las vistas complejas son vistas que pueden contener funciones de
grupo y uniones. El siguiente ejemplo crea una vista que deriva
datos de dos tablas:

~~~sql
CREATE OR REPLACE VIES view_euro_countries
  ("ID","Country","Capitol City","Region")
AS SELECT c.country, c.country_name, c.capitol, r.region_name
  FROM wf_countries c JOIN wf_world_regions r
  USING (region_id)
  WHERE location LIKE '%Europe';

SELECT * FROM view_euro_countries;
~~

El juego de sentencias anterior devuelve los siguientes  
resultados:

|ID|Country|Capitol City|Region|
|---|---|---|---|
|375|Republic of Belarus|Minsk|Eastern Europe|
|48|Republic of Poland|Warsaw|Eastern Europe|
|421|Slovak Republic|Bratislava|Eastern Europe|
|36|Republic of Hungary|Budapest|Eastern Europe|
|90|Republic of Turkey|Ankara|Eastern Europe|
|40|Romania|Bucharest|Eastern Europe|
|373|Republic of Moldova|Chisinau|Eastern Europe|
|370|Republic of Lithuania|Vilnius|Eastern Europe|
|371|Republic of Latvia|Riga|Eastern Europe|
|372|Republic of Estonia|Tallinn|Eastern Europe|
|...|...|...|...|

Las funciones de grupo también se pueden agregar a sentencias
de vistas complejas, por ejemplo:

~~~sql
CREATE OR REPLACE VIEW view_high_pop
  ("Region ID","Highest population")
AS SELECT region_id, MAX(population)
  FROM wf_countries
  GROUP BY region_id;

SELECT * FROM view_high_pop;
~~~

Las sentencias anteriores devuelven:

|Region ID|Highest population|
|---|---|
|5|188078227|
|9|20264082|
|11|131859731|
|13|107449525|
|14|74777981|
|15|78887007|
|17|62660551|
|18|44187637|
|21|298444215|
|...|...|

### Modificación de Vistas

Para modificar una vista existente sin tener que borrarla y
recrearla, utilice la opción `OR REPLACE` en la sentencia `CREATE VIEW`. La vista antigua se sustituye por la nueva:

~~~sql
-- nótese el uso de OR REPLACE antes del VIEW
CREATE OR REPLACE VIEW view_euro_countries AS
  SELECT country_id AS "ID", country_name AS "Country",
    capitol AS "Capitol City"
  FROM wf_countries
  WHERE location LIKE '%Europe'
~~~

## 15.2 Operaciones DML y Vistas

Además de simplificar el acceso de usuario a datos incluidos en
una o más tablas de la BD, las vistas también permiten a los 
usuarios realizar cambios en las tablas subyacentes.
Como DBA, puede que le interese poner restricciones en 
determinadas vistas de datos para conservar la integridad de  
toda la BD.

### Sentencias DML y Vistas

Las operaciones DML `INSERT`, `UPDATE` y `DELETE` se pueden
realizar en vistas simples. Estas operaciones se pueden utilizar
para cambiar los datos en las tablas subyacentes. Si se crea
una vista que permita a los usuarios ver información restringida
mediante la cláusula `WHERE` los usuarios seguirán pudiendo 
realizar operaciones DML en todas las columnas de la vista.

Por ejemplo, la siguiente vista de la tabla de empleados se ha  
creado para ser utilizada por los jefes del departamento 50:

~~~sql
CREATE VIEW view_dept50
AS SELECT department_id,
employee_id, first_name, last_name,
salary
  FROM copy_employees WHERE department_id=50;

SELECT * FROM view_dept50;
~~~

Las sentencias anteriores retornan:

|DEPARTMENT_ID|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|SALARY|
|---|---|---|---|---|
|50|124|Kevin|Mourgos|5800|
|50|141|Trenna|Rajs|3500|
|50|142|Curtis|Davies|3100|
|50|143|Randall|Matos|2600|
|50|144|Peter|Vargas|2500|

### Control de Vistas

Mediante la vista como se ha indicado, es posible **INSERTAR, 
ACTUALIZAR y SUPRIMIR**  información de todas las filas de la  
vista, incluso aunque el resultado sea que una fila ya no forma
parte de la vista. Puede ser que esto no era lo que deseaba el
DBA cuando creó la vista. Para controlar el acceso a los datos,
se pueden utilizar las opciones `CHECK OPTION` y `READ ONLY`
junto a `CREATE VIEW`.

#### Vistas con la Opción CHECK

La vista se define `WITH CHECK OPTION`:

~~~sql
CREATE VIEW view_dept50
AS SELECT department_id,
employee_id, first_name, last_name,
salary
  FROM copy_employees WHERE department_id=50;
~~~

Mediante la vista anterior, se puede cambiar el departamento
de un empleado:

~~~sql
UPDATE view_dept50
SET department_id=90
WHERE employee_id=124;
-- esta actualización se realizaría exitosamente aunque este
-- empleado ya no formará parte de la vista.
~~~

**Con `WITH CHECK OPTION` se garantiza que las operaciones DML
realizadas en la vista se mantengan en el dominio de la vista.**

Por ejemplo, cualquier intento de cambiar el número de departamento de una fila de la vista fallará porque viola la
restricción `WITH CHECK OPTION`:

~~~sql
-- el mismo ejemplo de antes, pero ahora se le agregará una
-- restricción CHECK
CREATE VIEW view_dept50
AS SELECT department_id,
employee_id, first_name, last_name,
salary
  FROM copy_employees WHERE department_id=50;
-- ahora cualquier cambio que un registro de esta vista sufra 
-- tendrá que cumplir la restricción de que siga dentro del
-- dominio de la vista, o sea que cumpla la condición del WHERE
WITH CHECK OPTION CONSTRAINT view_dept50_check;
~~~

Ahora, al tratar de realizar el siguiente cambio:

~~~sql
UPDATE view_dept50
SET department_id=90
WHERE employee_id=124;
~~~

Que sacaría al empleado 124 del dominio de la lista (aquellos 
que cumplen con `department_id=50`) entonces el servidor
devolvería un error.

#### Vistas con READ ONLY

La opción `WITH READ ONLY` garantiza que no se produzca ninguna
operación DML en la vista. Cuaqluier intento de ejecutar una
sentencia `INSERT`, `UPDATE` o `DELETE`

~~~sql
-- la misma vista del ejemplo anterior, pero utilizando
-- WITH READ ONLY
CREATE VIEW view_dept50
AS SELECT department_id,
employee_id, first_name, last_name,
salary
  FROM copy_employees WHERE department_id=50;
-- ahora no se podrán modificar los registros que exponga la
-- vista.
WITH READ ONLY;
~~~

Cualquier intento de ejecutar una sentencia DML producirá un
error de Oracle Server.

### Restricciones de DML

- Las vistas simples y complejas difieren en su capacidad para
permitir operaciones DML en una vista.

- Para las vistas simples, las operaciones DML pueden realizarse
en la vista.

- **Para las vistas complejas, las operaciones DML no siempre 
están permitidas.**

En general, si una vista compleja puede o no soportar operaciones
DML depdende de las siguientes reglas:

- **No se puede eliminar una fila desde una tabla base si la 
vista contiene algo de lo siguiente:**
  
  - Funciones de grupo
  - Cláusula `GROUP BY`
  - La palabra clave `DISTINCT`
  - La palabra clave `ROWMNUM` de pseudocolumna

- **No se puede modificar datos de una vista si la vista 
contiene:**

  - Funciones de grupo
  - Cláusula `GROUP BY`
  - La palabra clave `DISTINCT`
  - La palabra clave `ROWMNUM` de pseudocolumna
  - Columnas definidas por expresiones

- **No se pueden agregar datos en una vista si esta:**
  
  - Funciones de grupo
  - Cláusula `GROUP BY`
  - La palabra clave `DISTINCT`
  - La palabra clave `ROWMNUM` de pseudocolumna
  - Incluye columnas definidas por expresiones
  - No incluye columnas `NOT NULL` en las tablas base
  
## 15.3 Gestión de Vistas

Es posible que un empleado haya tenido acceso a información sensible mediante
una vista, y se deba asegurar de que esa vista no sea accesible más tiempo de
lo necesario. Las vistas son creadas con fines específicos, cuando ya no se
necesitan o se desean modificar, existen medios para realizar los cambios
necesarios.

### Supresión de una Vista

Puesto que una vista no contiene datos propios, **su eliminación no afecta los datos de las tablas subyacentes.**

Si la vista se ha utilizado para hacer operaciones DML en el 
pasado, esos cambios se mantienen en las tablas base. Suprimir
una vista simplemente elimina su definición de la BD. Sólo el
creador o un usuario de la BD con el privilegio `DROP ANY VIEW`
pueden eliminar una vista. La sintáxis SQL para eliminar una 
vista es:

~~~sql
DROP VIEW viewname;
~~~

### Vistas en Línea

Las vistas en línea también se denominan **subconsultas en la
cláusula `FROM`**. Inserte una subconsulta en la cláusula `FROM`
como si la subconsulta fuera un nombre de tabla. Las vistas en
línea se utilizan para simplificar consultas complejas mediante
la eliminación de operaciones de unión y la condensación de
varias consultas en una sola.

En el siguiente ejemplo, la cláusula `FROM` contiene una 
sentencia `SELECT` que recupera datos como cualquier sentencia
`SELECT`. A los datos devueltos por la subconsulta se les asigna
un alias `d` que a continuación se utiliza junto con la consulta
principal para devolver columnas seleccionadas de ambos orígenes
de la consulta:

~~~sql
SELECT e.last_name, e.salary, e.department_id, d.maxsal
FROM employeees e, (
  SELECT department_id, MAX(salary) maxsal
  FROM employees GROUP BY department_id
) d
WHERE e.department_id = d.department_id
AND e.salary = d.maxsalary;
~~~

### Análisis de N principales

El análisis de N principales es una operación SQL utilizada
para clasificar resultados. El uso del análisis de N principales
resulta útil cuando desea recuperar los n registros principales 
de un juego de resultados devuelto por una consulta. Por ejemplo,
la siguiente sentencia regresa los cinco primeros empleados con
la fecha de contratación más antigua:

~~~sql
SELECT ROWNUM AS "Longest employed", last_name, hire_date
FROM employees WHERE ROWNUM <=5
ORDER BY hire_date;
~~~

La sentencia anterior devuelve:

|Longest employed|LAST_NAME|HIRE_DATE|
|---|---|---|
|1|King|17-Jun-1987|
|4|Whalen|17-Sep-1987|
|2|Kochhar|21-Sep-1989|
|3|De Haan|13-Jan-1993|
|5|Higgins|07-Jun-1994|

El análisis de N principales utiliza una vista en línea para
devolver un juego de resultados. Puede utilizar `ROWNUM` en su
consulta para asignar un número de fila al juego de resultados.
La siguiente es una versión del ejemplo anterior, pero haciendo
un análisis de N principales:

~~~sql
SELECT ROWNUM AS "Longest employed", last_name, hire_date
FROM (
  -- primero se selecciona la lista de hire_date y last_name
  SELECT last_name, hire_date
  FROM employees
  -- luego se ordena por antigüedad ascendentemente
  ORDER BY hire_date;
)
-- finalmente se restringe el número devuelto de filas
WHERE ROWNUM <=5;
~~~

La sentencia anterior devuelve:

|Longest employed|LAST_NAME|HIRE_DATE|
|---|---|---|
|1|King|17-Jun-1987|
|2|Whalen|17-Sep-1987|
|3|Kochhar|21-Sep-1989|
|4|Hunold|03-Jan-1990|
|5|Ernst|27-May-1991|
