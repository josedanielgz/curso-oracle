Parte 12

* [Volver al inicio](index.html)

# 12.1 Sentencias INSERT

Las bases de datos son altamente dinámicas, están constantemente en el proceso
de inserción, actualización y supresión de datos, pues de no ser así, las BBDD
serían poco útiles. Las sentencias de lenguaje de manipulación de datos, o DML,
sirven para realizar cambios en una BBDD.

Se recomienda realizar una copia de las tablas originales antes de practicar sentencias de inserción riesgosas, y utilizar esa copia. En el peor de los casos, siempre se puede restaurar desde la tabla original.

### Sintaxis para Crear una Copia de una Tabla

Cree la sintaxis de la tabla:

~~~sql
CREATE TABLE copy_tablename AS (SELECT * FROM tablename);
~~~

Por ejemplo:

~~~sql
CREATE TABLE copy_employees AS (SELECT * FROM employees);
CREATE TABLE copy_departments AS (SELECT * FROM departments);
~~~

Puede verificar y ver la copia de la tabla utilizando `DESCRIBE` y `SELECT`.

~~~sql
DESCRIBE copy_emplyees;
SELECT * FROM copy_employees;
~~~

### INSERT

La sentencia `INSERT` se utiliza para agregar una nueva fila a una tabla. Esta tiene tres valores: el nombre de la tabla, los nombres de las columnas de la tabla que se van a rellenar y los valores correspondientes para cada columna.

Por ejemplo, para insertar los siguientes datos de un nuevo departamento en la tabla `copy_departments`:

|DEPARTMENT_ID|DEPARTMENT_NAME|MANAGER_ID|LOCATION_ID|
|---|---|---|---|
|200|Human Resources|205|1500|

Se utiliza la siguiente sentencia:

~~~sql
INSERT INTO copy_departments 
  (department_id, department_name, manager_id, location_id)
VALUES (200,'Human Resources',205,1500);
~~~

Otra forma de agregarlos es implícitamente, omitiendo los nombres de columna,
**pero los valores de cada columna deben coincidir exactamente con el orden por
defecto en el que aparecen en la tabla (como se muestra en `DESCRIBE` y se debe
proporcionar un valor para cada columna.**

La sentencia `INSERT` en este ejemplo se ha escrito sin nombrar explícitamente las columnas:

~~~sql
INSERT INTO copy_departments VALUES
  (210,'Estate Management',102,1700);
~~~

Pero para mayor claridad, es mejor utilizar los nombres de columna en una cláusula `INSERT`.

### Comprobación de la tabla en primer lugar

Antes de insertar datos en una tabla, debe comprobar varios detalles de la
tabla. La sentencia `DESCRIBE` devuelve una descripción de la estructura de la tabla y el gráfico de resúmen de la tabla. Esta es la descripción de la tabla `copy_departments`:

|Column|Data Type|Length|Precision|Scale|Primary Key|Nullable|
|---|---|---|---|---|---|---|
|DEPARTMENT_ID|NUMBER|-|4|0|-|Yes|
|DEPARTMENT_NAME|VARCHAR2|30|-|-|-|-|
|MANAGER_ID|NUMBER|-|6|0|-|Yes|
|LOCATION_ID|NUMBER|-|4|0|-|Yes|

El resumen de una tabla proporciona información sobre cada columna de la tabla como:

* permiso de valores duplicados
* tipos de dato permitido
* cantidad de datos permitida
* permiso de valores NULL

La columna Tipo de dato para los tipos de dato de caracteres especifica entre el paréntesis el número máximo de caracteres permitidos. Por ejemplo, en la siguiente tabla:

|Column|Data Type|Length|Precision|Scale|Primary Key|Nullable|
|---|---|---|---|---|---|---|
|EMPLOYEE_ID|NUMBER|-|6|0|1|-|
|FIRST_NAME|VARCHAR2|20|-|-|-|Yes|
|LAST_NAME|VARCHAR2|25|-|-|-|-|
|EMAIL|VARCHAR2|25|-|-|-|-|
|PHONE_NUMBER|VARCHAR2|25|-|-|-|Yes|
|HIRE_DATE|DATE|7|-|-|-|-|
|JOB_ID|VARCHAR2|10|-|-|-|-|
|SALARY|NUMBER|-|8|2|-|Yes|
|COMMISSION_PCT|NUMBER|-|2|2|-|Yes|
|MANAGER_ID|NUMBER|-|6|0|-|Yes|
|DEPARTMENT_ID|NUMBER|-|4|0|-|Yes|
|BONUS|VARCHAR2|5|-|-|-|Yes|

`first_name` es de tipo `VARCHAR2(20)`, o sea que se pueden introducir hasta 20 caracteres en esa columna.

Para los datos `NUMBER`, los paréntesis especifican la precisión y la escala.
Esto es el número total de dígitos y el número de dígitos a la derecha de la
posición decimal, respectivamente.

Por ejemplo, la columna `SALARY` permite números con una precisión de 8 y una escala de 2, o sea que el valor máximo permitido es 999999,99.

### Inserción de filas con Valores Nulos

La sentencia `INSERT` no necesita especificar todas las columnas: se pueden excluir las columnas con valores nulos.

Si a todas las columnas que necesitan un valor se les asigna un valor, la inserción funciona.

En nuestro ejemplo, la columna `EMAIL` se define como una columna `NOT NULL`. Un intento implícito de agregar valores a la tabla como el que se muestra a continuación generará un error:

~~~sql
INSERT INTO copy_employees
  (employee_id, first_name, last_name, phone_number, hire_date, job_id, salary);
VALUES
  (302,'Grigorz','Polanski','8586667641','15-Jun-2017','IT_PROG',4200);
-- generará error porque la columna EMAIL no se ha mencionado explícitamente, y
-- esta no acepta valores nulos
~~~

Las inserciones implícitas insertan automáticamente un valor nulo en las columnas que lo permiten. Se puede utilizar la palabra `NULL` para agregarlos explícitamente.

Para especificar cadenas vacías y/o fechas que faltan, utilice
comillas simples vacías, así `''`:

~~~sql
INSERT INTO copy_employees
  (employee_id, first_name, last_name, email, phone_number, hire_date, job_id, salary);
VALUES
  (302,'Grigorz','Polanski','gpolanski','','15-Jun-2017','IT_PROG',4200);
~~~

La sentencia anterior inserta el siguiente empleado:

|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|EMAIL|PHONE_NUMBER|HIRE_DATE|JOB_ID|SALARY|
|---|---|---|---|---|---|---|---|
|302|Grigorz|Polanski|gpolanski|-|15-Jun-2017|IT_PROG|4200|

### Inserción de Valores Especiales

Los valores especiales como `SYSDATE` y `USER` se pueden introducir en la lista
`VALUES` de una sentencia `INSERT`.
`SYSDATE` colocará las fechas y hota actuales en una columna, `USER` insertará el nombre de usuario de la sesión actual, que es `OAE_PUBLIC_USER` en Oracle APEX.

~~~sql
INSERT INTO copy_employees
  (employee_id, first_name, last_name, email, phone_number, hire_date, job_id, salary);
VALUES
  (304,'Test',USER,'t_user','4159982010',SYSDATE,'ST_CLERK',2500);
~~~

### Inserción de Valores de Fecha Específicos

El modelo de formato por defecto para tipos de dato de fecha es DD-Mes-AAAA. Con este formato de fecha, la hora por defecto de medianoche (00:00:00) también se incluye.

En la sección anterior, hemos aprendido a cómo utilizar la función `TO_CHAR` para convertir una fecha en una cadena de caracteres cuando queremos recuperar y mostrar un valor de fecha con un formato que no es el formato por defecto. Por ejemplo:

~~~sql
SELECT first_name, TO_CHAR(hire_date,'Montrh, fmdd, YYYY')
FROM employees WHERE employee_id=101;
-- Devuelve September, 21, 1989
~~~

Del mismo modo, **si deseamos insertar una fila con un formato que no sea el
formato por defecto para una colummna de fecha, debemos utilizar la función
`TO_DATE` para convertir el valor de fecha (cadena de caracteres) en una fecha.**

~~~sql
INSERT INTO copy_employees
  (employee_id,first_name,last_name,email,phone_number,hire_date,job_id,salary);
VALUES
  (301,'Katie','Hernandez','khernandez','8586667641',
    TO_DATE('July 8, 2017', 'Month fmdd, yyyy'),
    'MK_REP',4200);
~~~

**También es posible sustituir la hora por defecto:**

~~~sql
INSERT INTO copy_employees
  (employee_id,first_name,last_name,email,phone_number,hire_date,job_id,salary);
VALUES
  (303,'Angelina','Wright','awright','4159982010',
    TO_DATE('July 10, 2017 17:20', 'Month fmdd, yyyy HH24:MI'),
    'MK_REP',3600);
~~~

La sentencia anterior inserta el siguiente registro:
|FIRST_NAME|LAST_NAME|Date and Time|
|---|---|---|
|Angelina|Wright|10-Jul-2017 17:20|

### Uso de una Subconsulta para Copiar Filas

Cada sentencia `INSERT` hasta el momento sólo ha agregado una fila a la vez.
Sin embargo, es posible utilizar subconsultas dentro de una sentencia `INSERT`
para copiar filas en masa desde otra tabla. Todos los resultados de dicha subconsulta se insertan en la tabla. Por lo tanto, podemos copiar 100 filas (o 1000 filas) con una subconsulta de varias filas en `INSERT`.
**No se necesita una cláusula `VALUES` al insertar filas de este modo, el motor
puede determinar que los valores insertados son exactamente los mismos
devueltos por la subconsulta.**. Por ejemplo:

~~~sql
INSERT INTO sales_reps(id,name,salary,commission_pct);
  SELECT employee_id, last_name, salary, commission_pct
  FROM employees
  WHERE job_id LIKE '%REP%';
~~~

En el ejemplo anterior, una nueva tabla llamada `SALES_REP` se rellena con copias de algunas de las filas y columnas de la tabla empleado, aquellas que tengan identificadores de trabajo como `%REP%`.

El número de columnas y sus tipos de dato de la lista de columnas de la cláusula `INSERT` deben coincidir con el número de columnas y sus tipos de dato en la subconsulta. La subconsulta no está incluída entre paréntesis como se hace con las subconsultas en la cláusula `WHERE` de una sentencia `SELECT`.

Si deseamos copiar todos los datos (filas y columnas), la sintaxis es aún más simple:

~~~sql
INSERT INTO sales_rep SELECT * FROM employees;
~~~

**Esto sólo funcionará si ambas tablas tienen el mismo número de colummnas que coincida con los tipos de dato y que estén en el mismo orden.**

## 12.1 Actualización de Valores de Columna y Supresión de Filas

Las bases de datos casi nunca se mantienen estáticas, y la actualización de los datos es uno de los trabajos de un administrador de BBDD (o DBA).
La sentencia `UPDATE` se utiliza para modificar filas existentes de una tabla. Esta sentencia necesita cuatro valores.
* nombre de la tabla
* nombre de las columnas cuyos valores se van a modificar
* un nuevo valor para cada una de las columnas que se están modificando
* una condición que identifique las filas de la tabla que se van a modificar
**El nuevo valor para una columna puede ser el resultado de una subconsulta de una sola fila.**

Por ejemplo:

~~~sql
-- Se recomienda que la sentencia UPDATE aparezca sola
-- en una línea
UPDATE copy_employees SET phone_number = '123456' WHERE employee_id = 103;
~~~

La sentencia anterior actualiza el número de teléfono de un empleado, de forma qu queda así:

|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|PHONE_NUMBER|
|---|---|---|---|
|**303**|Angelina|Wright|**123456**|

Podemos cambiar varias columnas y/o filas en una sola sentencia `UPDATE`. En este ejemplo, se cambia el número de teléfono y el apellido de dos empleados:

~~~sql
UPDATE copy_employees 
SET phone_nummber='654321',last_name='Jones'
WHERE employee_id>=303;
~~~

La anterior sentencia produce el siguiente cambio:

|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|PHONE_NUMBER|
|---|---|---|---|
|**303**|Angelina|**Jones**|**654321**|
|**303**|Test|**Jones**|**654321**|

**TENGA CUIDADO AL ACTUALIZAR LOS VALORES DE COLUMNA, SI LA CLÁUSULA `WHERE` SE OMITE, TODAS LAS FILAS DE LA TABLA SE ACTUALIZARÁN**

~~~sql
UPDATE copy_employees 
SET phone_nummber='654321',last_name='Jones';
~~~

La sentencia anterior produce los siguientes cambios:

|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|PHONE_NUMBER|
|---|---|---|---|
|100|Steven|Jones|654321|
|101|Neena|Jones|654321|
|102|Lex|Jones|654321|
|200|Jennifer|Jones|654321|
|205|Shelley|Jones|654321|
|206|William|Jones|654321|
|149|Eleni|Jones|654321|
|174|Ellen|Jones|654321|
|...|...|...|...|

### Actualización de una Columna con un valor de una Subconsulta

Podemos utilizar el resultado de una subconsulta de una sola fila
para proporcionar el valor nuevo para una consulta actualizada.
Por ejemplo:

~~~sql
UPDATE copy_employees
SET salary = (SELECT SALARY
  FROM copy_employees WHERE employee_id = 100)
WHERE employee_id = 101;
~~~

En el ejemplo anterior se cambia el salario del empleado 101 al mismo salario que otro empleado (el 100).

|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|SALARY|
|---|---|---|---|
|100|Steven|King|24000|
|101|Neena|Kochhar|24000|

### Actualización de Dos columnas con Dos sentencias de Subconsulta

Para actualizar varias columnas en una sentencia `UPDATE` se puede
escribir varias subconsultas de una sola fila, una para cada columna.
Por ejemplo, la siguiente sentencia `UPDATE` cambia el salario de un empleado y el
identificador de cargo `id=206` a los mismos valores que otro empleado `id=205`

~~~sql
UPDATE copy_employees
SET salary = (
  SELECT salary FROM copy_employees
  WHERE employee_id = 205)
), job_id = (
  SELECT job_id FROM copy_employees
  WHERE employee_id=205
)
WHERE employee_id=206;
~~~

Este es el cambio realizado:

|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|SALARY|JOB_ID|
|---|---|---|---|---|
|205|Shelley|Higgins|12000|AC_MGR|
|206|William|Gietz|12000|AC_MGR|

### Actualización de Filas con Subconsulta Correlacionada

Como ya sabe, las subconsultas pueden ser independientes o correlacionadas. En una subconsulta correlacionada, se
actualiza una fila de una tabla según una selección de la misma tabla.
En el siguiente ejemplo, se ha creado una copia de la columna de nombre de departamento en la tabla de empleados. A continuación, los datos de la tabla de departamentos original se han recuperado, copiado y utilizado para rellenar la cpia de la columna en la tabla `copy_employees`.

~~~sql
ALTER TABLE copy_employees ADD (department_name VARCHAR2(30) NOT NULL);

UPDATE copy_employees e SET e.department_name = (
  SELECT d.department_name
  FROM departments d
  WHERE e.department_id =
  d.department_id
);
~~~

### DELETE
La sentencia `DELETE` se utiliza para eliminar filas existentes
de una tabla. La sentencia necesita dos valores:

* nombre de la tabla
* condición que identifica las filas que se van a suprimir

En el ejemplo, se utiliza la copia de la tabla de empleados para
suprimir una fila: el empleado con el número de identificador 303:

~~~sql
DELET from copy_employees WHERE employee_id = 303; 
~~~

**SI NO ESPECIFICA UNA CLÁUSULA `WHERE`, SE SUPRIMEN TODAS LAS FILAS DE LA TABLA.**

### Subconsultas DELETE

También se pueden utilizar subconsultas en sentencias `DELETE`. En este ejemplo, se suprimen las filas de la tabla de empleados para todos los empleados que trabajan en el departamento de envíos:

~~~sql
DELETE FROM copy_employees WHERE department_id =
  (SELECT department_id FROM departments
    WHERE department_name = 'Shipping');
-- Esto elimina 5 filas
~~~

El siguiente ejemplo elimina filas de todos los empleados que
trabajan para un gerente que administra menos de 2 empleados:

~~~sql
DELETE FROM copy_employees e
WHERE e.manager_id IN (
  SELECT d.manager_id
  FROM employees d
  HAVING COUNT(d.department_id)<2
  GROUP BY d.manager id
);
~~~

### Errores de Restricción de Integridad

Las restricciones de integridad garantizan que los datos se ajusten a un juego anidado de reglas. Las restricciones se comprueban cada vez que se ejecuta una sentencia DML que las pueda romper; de ser el caso, la tabla no se actualiza y devuelve un error.

Por ejemplo:

~~~sql
UPDATE copy_employees
SET last_name = (
  SELECT last_name
  FROM copy_employees
  WHERE employee_id = 999
)
WHERE employee_id = 101;
~~~

Aquí se viola una restricción `NOT NULL` porque `last_name` tiene una restricción de no nulo e `id=999` no existe (entonces esa subconsulta regresa nulo).

Los siguientes son ejemplos de comprobación de restricciones de integridad. Considere que la tabla `employees` tiene una restricción de clave ajena en la columna department_id de la tabla `departments`, que garantiza que cada empleado pertenezca a un departamento válido, y que en la tabla `departments` existen los departamentos 10 y 20, pero no 15:

~~~sq
-- Esta falla
UPDATE employees SET department_id = 15 WHERE employee_id = 100;

-- Esta técnicamente no falla
DELETE FROM departments WHERE department_id = 10;

-- Pero esta fallará si la anterior sentencia se ejecuta
UPDATE employees SET department_id = 10
WHERE department_id = 20;
~~~

**CREAR UNA TABLA MEDIANTE `CREATE TABLE ... AS` TRANSFIERE LAS FILAS Y LAS RESTRICCIONES NULAS, PERO NO LAS RESTRICCIONES DE CLAVE PRIMARIA-AJENA, POR LO TANTO NO EXISTE NINGUNA RESTRICCIÓN DE INTEGRIDAD EN LAS TABLAS COPIADAS, ESTAS DEBEN ADICIONARSE MANUALMENTE.**

### Cláusula FOR UPDATE en una Sentencia SELECT

Al emitir una sentencia `SELECT` en una tabla de BBDD, no se emite ningún
bloqueo en la BBDD en las filas devueltas por la consulta realizada. **Sin
embargo, a veces es necesario asegurarse de que nadie más pueda actualizar o
suprimir los registros que la consulta está devolviendo mientras trabaja en
esos registros. Este es el propósito de la cláusula `FOR UPDATE`.** En cuanto se ejecuta la consulta, la BBDD emitirá bloqueos de nivel de fila exclusivos en toda las filas devueltas por una sentencia `SELECT`, que se mantendrán hasta que emita un comando `COMMIT` o `ROLLBACK`. **Cabe destacar que la instancia en línea/alojado de APEX se confirmará automáticamente y el bloqueo de nivel de fila no se realizará.**

Si se utiliza una cláusula `FOR UPDATE` en una sentencia `SELECT` con varias tablas en ella, todas las filas de todas las tablas se bloquearán. Por ejemplo:

~~~sql
SELECT e.employee_id, e.salary, d.department_name
FROM employees e JOIN departments d
USING (department_id)
WHERE job_id = 'ST_CLERK' AND location id = 1500
  FOR UPDATE -- Atención a esta parte
ORDER BY e.employee_id;
~~~

|EMPLOYEE_ID|SALARY|DEPARTMENT_NAME|
|---|---|---|
|141|3500|Shipping|
|142|3100|Shipping|
|143|2600|Shipping|
|144|2500|Shipping|

Ahora esasa cuatro filas están bloqueadas por el usuario que ha ejecutado la sentencia `SELECT` hasta que ese usuario haga `COMMIT` y `ROLLBACK`.

### Valores DEFAULT, MERGE e Inserciones en Varias Tablas

Hasta ahora ha estado actualizando e insertando registros mediante sentencias
individuales `INSERT`. Esto puede quedarse cortos en casos donde se requiere
manejar un volumen grande de información. En una empresa real pueden haber
almacenes de datos para almacenar los registros de ventas, de clientes, de
nómina, de contabilidad y de personal, entre otros, donde los datos procedan de
varios orígenes y sean gestionados por varias personas.

Por suerte SQL cuenta con formas más eficaces para actualizar e insertar datos
masivamente, como la combinación de una sentencia `INSERT` y `UPDATE` en un
solo comando atómico, y la posibilidad de recuperar datos de una única
subconsulta e insertarlos en más de una tabla de destino.

### DEFAULT
Cada columna puede tener un valor por defecto especificado, cada vez que se inserte una nueva fila y no se le asigne un valor, se utilizará ese valor asignado en lugar de un nulo. **Este valor `DEFAULT` puede establecerse al crear o modificar la columna de una tabla y puede adquirir un valor literal, o una expresión o función SQL, pero NO PUEDE SER EL NOMBRE DE OTRA COLUMNA.**

Por ejemplo:

~~~sql
CREATE TABLE my_employees (
  hire_date DATE DEFAULT SYSDATE, -- Atención a esta parte
  first_name VARCHAR2(15),
  last_name VARCHAR(15)
);
~~~

La anterior sentencia sirve para crear una tabla de empleados que asigne `hire_date` automáticamente con la fecha del servidor si este campo no se asigna manualmente.

La mejor forma de usar el valor DEFAULT es implícitamente, no
declarándolo en la sentencia `INSERT`, aunque es posible declararo explícitamente:**

~~~sql
-- esta es la forma explícita de usar DEFAULT. Recomendado
-- para dar legibilidad a las sentencias, pero no es necesario
INSERT INTO my_employees (hire_date, first_name, last_name)
VALUES (DEFAULT, 'Angelina', 'Wright');
~~~

### DEFAULT explícito con UPDATE

Se pueden insertar valores por defecto explícitos en las sentencias `INSERT` y `UPDATE`, por ejemplo

~~~sql
UPDATE my_employees
SET hire_date=DEFAULT
WHERE last_name='Wright';
~~~

**Si no se especificó un valor DEFAULT al momento de la creación de
la tabla, `DEFAULT` asigna un valor nulo.**

### MERGE
El uso de la sentencia `MERGE` logra dos tareas al mismo tiempo.
Con `MERGE` se puede insertar y actualizar simultáneamente. Si falta un valor, se inserta uno nuevo, y si este existe pero debe cambiarse, se actualizará. Se necesitan ambos privilegios (`INSERT` y `UPDATE`) en la tabla de destino y privilegios `SELECT` en la tabla de origen. `MERGE` soporta el uso de aliases.

Se lee una fila a la vez de la tabla origen y se comparan con la
de destino según la condición de coincidencia. **Si existe una fila
coincidente en la tabla de destino, la fila de origen se utiliza
para actualizar una o más columnas de la tabla destino coincidente.**

**Si no existe una fila coincidente, los valores de la fila de origen se utilizan para insertar una nueva fila en la tabla destino.**

Por ejemplo:

~~~sql
MERGE INTO copy_emp c USING employees e
  ON (c.employee_id = e.employee_Id)
WHEN MATCHED THEN UPDATE
  SET
    c.last_name = e.last_name,
    c.department_id = e.department_id
WHEN NOT MATCHED THEN INSERT
  VALUES (e.employee_id, e.last_name, e.department_id);
~~~

La sentencia anterior utiliza la tabla `employees` (alias e) como
origen de datos para insertar y actualizar filas en una copia de
la tabla denominada `copy_emp` (alias c).

### Ejemplos de MERGE

Las filas 100 y 103 de `employees` tienen filas coincidentes en
`copy_emp` y por lo tanto, se actualizaron las filas coincidentes
de `copy_emp`. El empleado 142 no tiene ninguna fila coincidente, y por lo tanto se insertó en `copy_emp`.

Para propósitos del ejemplo, estos son los registros de la tabla origen:

|EMPLOYEE_ID|LAST_NAME|DEPARTMENT_ID|
|---|---|---|
|100|King|90|
|103|Hunold|60|
|142|Davies|50|

Estos son los de `copy_emp`

|EMPLOYEE_ID|LAST_NAME|DEPARTMENT_ID|
|---|---|---|
|100|Smith|40|
|103|Chang|30|

Después del `MERGE` de ambas tablas:

|EMPLOYEE_ID|LAST_NAME|DEPARTMENT_ID|
|---|---|---|
|100|King|90|
|103|Hunold|60|
|142|Davies|50|

### Inserciones en Varias Tablas

Cuando se deben insertar los mimsos datos de origen en más de un
destino, existe la posibilidad de realizaer inserciones en varias tablas. Esto es útil cuando se trabaja en un entorno de
almacén de datos donde es común mover con regularidad datos de
los sistemas operativos a un almacén de datos para la generación de informe
analíticos y análisis. Existen escenarios donde se gestionan cantidades
inmensas de información; por ejemplo, la cantidad de filas que debe gestionar
un proveedor de telefonía diariamente, o las páginas que navega en internet, por la
cantidad de clientes que debe tener.

Puede ser que estas filas deban insertarse en más de una tabla
del almacén de datos, si podemos seleccionarlas una sola vez, pero replicarlas
para cada lugar donde las necesitemos, el rendimiento mejora radicalmente en
comparación con tener que seleccionarlas varias veces.

**La inserción de varias filas puede ser condicional o incondicional; en la primera, Oracle insertará todas las filas
devueltas por la subconsulta en todas las cláusulas de inserción
de tabla encontradas en la sentencia; en la segunda, puede especificar `ALL` o `FIRST`**

#### ALL

Si se especifica `ALL` (el valor por defecto) la base de datos evalúa cada
cláusula `WHEN` independientemente de los resultados de la evaluación de
cualquier otra cláusula `WHEN`.
Para cada cláusula `WHEN` cuya condicion evaluada sea cierta, la BBDD ejecuta la lista de cláusulas `INTO` correspondientes.

#### FISRT

Si especifica `FIRST`, la BBDD evalúa cada cláusula `WHEN` en el orden en el
que aparece la sentencia. Para la primiera cláusula `WHEN` que se evalúe como
cierta, la BBDD ejecuta la cláusula `INTO` correspondiente y omite las
cláusulas `WHEN` posteriores de la fila determinada.

#### ELSE

Para una fila determinada, si la cláusula `WHEN` no se evalúa como cierta, la
BBDD ejecuta las cláusulas `INTO` asociadas con la cláusula `ELSE`. Si no se especifica una cláusula `ELSE`, la BBDD no realiza ninguna acción para dicha fila

### Sintaxis de Inserciones en Varias Tablas

La sintaxis de la sentencia de inserción en varias tablas es la
siguiente:

~~~sql
INSERT ALL INTO clause VALUES clause SUBQUERY;
~~~

Un ejemplo de la sentencia de inserción en varias tablas es el
siguiente:

~~~sql
INSERT ALL
  INTO my_employees
    VALUES (hire_date,first_name,last_name)
  INTO copy_my_employees
    VALUES (hire_date,first_name,last_name)
SELECT hire_date, first_name, last_name
FROM employees;
~~~

La sentencia anterior consulta algunos campos de los registros de la tabla
`employees` y crea dos copias en las tablas `my_employees` y
`copy_my_employees`.

Otro ejemplo más complejo es:

~~~sql
INSERT ALL
  WHEN call_format IN ('tlk','txt','pic') THEN
  INTO all_calls
    VALUES (caller_id, call_timpestamp, call_duration, call_format)
  WHEN call_format IN ('tlk','txt') THEN
  INTO police_record_calls
    VALUES (caller_id, call_timpestamp, recipient_caller)
  WHEN call_duration <50 AND call_type = 'tlk' THEN
  INTO short_calls
    VALUES (caller_id, call_timestamp, call_duration)
  WHEN call_duration >50 AND call_type = 'tlk' THEN
  INTO long_calls
    VALUES (caller_id, call_timestamp, call_duration)
SELECT caller_id, call_timestamp, call_duration, call_format,
  recipient_caller
FROM calls
WHERE TRUNC(call_timestamp) = TRUNC(SYSDATE);
~~~
