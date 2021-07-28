- [Volver al inicio](index.html)

## 16.1 Trabajar con Secuencias

SQL cuenta con un proceso para generar automáticamente números
únicos que elimina la preocupación por los detalles de números
duplicados, este se maneja a través de un objeto de BD llamado
`SEQUENCE`.

Este es un objeto que se puede usar de forma compartida para
generar números duplicados de manera automática. Varios usuarios
pueden acceder a la misma secuencia. **Generalmente se utilizan
secuencias para crear valores de clave primaria.**

Para asegurar que cada valor de la secuencia sea único, estas se
generan y aumentan (o disminuyen) según una rutina interna de
Oracle. Este objeto ahorra tiempo porque reduce la cantidad de
código a escribir.

**Los números de secuencia se almacenan y generan
independientemente de las tablas, por lo tanto la misma
secuencia se puede utilizar para varias tablas.**

La sintáxis para crear una `SEQUENCE`:

```sql
CREATE SEQUENCE sequence
  [INCREMENT BY n]
  [START WITH n]
  [{MAXVALUE n| NOMAXVALUE}]
  [{MINVALUE n| NOMINVALUE}]
  [{CYCLE|NOCYCLE}]
  [{CACHE n|NOCACHE}];
```

En donde las palabras reservadas significan:

- `secuencia`: nombre del generador de secuencias (objeto)
- `INCREMENT BY n`: especifica el intervalo entre los números de secuencia,
  donde n es un número entero (si esta cláusula se omite, la secuencia se va
  incrementando en uno).

- `START WITH n`: especifica el primer número de secuencia que se va a generar
  (si se omite esta cláusula, la secuencia empieza con 1).

- `MAXVALUE n`: especifica el valor máximo que puede generar una secuencia.

- `NOMAXVALUE`: especifica un valor máximo de `10^27` para una secuencia
  ascendente y -1 para una secuencia descendente (los valores por defecto).

- `MINVALUE n`: especifica el valor mínimo de la secuencia.

- `NOMINVALUE`: especifica un valor máximo de 1 para una secuencia ascendente y
  uno de `-(10^26)` para una secuencia descendente (por defecto).

- `CYCLE`|`NOCYCLE` especifica si la secuencia sigue generando valores después de
  alcanzar su valor máximo o mínimo (`NOCYCLE` es la opción por defecto).

- `CACHE n`|`NOCACHE`: especifica cuántos valores asigna previamente o mantiene
  Oracle Server en la memoria (por defecto, Oracle Server almacena en caché 20
  valores). **Si el sistema falla, los valores se pierden.**

### Creación de Secuencias

En la `SEQUENCE` creda para los corredores del Maratón de Londres, los números se irán incrementando en 1, empezando por el
número 1. En este caso, empezar la secuencia con 1 es el mejor
punto de partida:

```sql
CREATE SEQUENCE runner_id_seq
INCREMENT BY 1
START WITH 1
MAXVALUE 50000
NOCACHE
NOCYCLE;
```

En este contexto `NOCACHE` evita que los valores de `SEQUENCE` se
almacenen en la caché de la memoria, con lo que, en caso de fallo
del sistema, se evitará que se perdieran los números asignados
previamente y retenidos en la memoria.

La opción `NOCYCLE` evita que la numeración vuelva a empezar en 1
si se exceden los 50 mil registros.

**NO UTILICE LA OPCIÓN `CYCLE` SI LA SECUENCIA SE UTILIZA PARA
GENERAR VALORES DE CLAVE PRIMARIA, A MENOS QUE SE DISPONGA DE UN MECANISMO FIABLE QUE SUPRIMA LAS FILAS ANTIGUAS MÁS RÁPIDO DE LO
QUE SE AGREGAN NUEVAS FILAS**

### Confirmación de Secuencias

Para verificar que la secuencia se ha creado, consulte el
diccionario de datos, específcamente `USER_OBJECTS`, de la
siguiente forma:

```sql
SELECT sequence_name, min_value, max_value, increment_by,
last_number FROM user_sequences;
```

El valor last_number cambia dependiendo de si se especificó
`CACHE` o `NOCACHE` en la creación de la secuencia:

- Si se especifica `NOCACHE`, aparece el siguiente número de secuencia
  disponible.

- Si se especifica `CACHE`, muestra el siguiente número disponible de la
  secuencia que no se ha almacenado en caché en la memoria.

### Pseudocolumnas NEXTVAL y CURRVAL

La pseudocolumna `NEXTVAL` se utiliza para extraer números de
secuencia sucesivos de una secuencia especificada. **Debe
cualificar a `NEXTVAL` con el nombre de una columna.**

Al hacer referencia a sequence.`NEXTVAL`, se genera un nuevo
número de secuencia y el actual se sustituye en `CURRVAL`.

En el siguiente ejemplo, se inserta un nuevo departamento en la
tabla `departments`, Utiliza la secuencia `DEPARTMENTS_SEQ` para
generar un nuevo número de departamento:

```sql
INSERT INTO departments
  (department_id, department_name, location_id)
VALUES (departments_seq.NEXTVAL, 'Support', 2500);
```

Supongamos que queremos contratar nuevos empleados para ese
departamento; además ya tenemos de antemano la secuencia
`EMPLOYEE_SEQ` para generar los números de los nuevos empleados.

```sql
INSERT INTO employees
  (employee_id, department_id, ...)
-- se puede usar CURRVAL sobre la secuencia para los
-- números de departamento y averiguar cuál es el ID del
-- último que ha sido agregado
VALUES (employees_seq.NEXVAL, dept_deptid_seq.CURRVAL, ...);
```

### Uso de una Secuencia

Después de crear una secuencia, se generan números secuenciales
para utilizarlos en tablas. Se puede referenciar estos valores
mediante las pseudocolumnas `NEXTVAL` y `CURRVAL`. Estas dos
palabras se pueden utilizar en:

- La lista `SELECT` de una sentencia `SELECT` **que no forme parte
  de una subconsulta.**

- La lista `SELECT` de una subconsulta de la sentencia `INSERT`.

- La cláusula `VALUES` de una sentencia `INSERT`.

- La cláusula `SET` de la sentencia `UPDATE`.

**NO SE PUEDE UTILIZAR `NEXTVAL` Y `CURRVAL` en los siguientes
contextos:**

- La lista `SELECT` de una vista.

- UNa sentencia `SELECT` con la palabra clave `DISTINCT`.

- Una sentencia `SELECT` con las cláusulas `GROUP BY`, `HAVING` o
  `ORDER BY`.

- Una subconsulta en una sentencia `SELECT`, `UPDATE` o
  `DELETE`.

- La expresión `DEFAULT` en una sentencia `CREATE TABLE` o
  `ALTER TABLE`.

Retomando el ejemplo de la maratón de corredores, se ha creado la
siguiente tabla:

```sql
CREATE TABLE runners
(
  runner_id NUMBER(6,0) CONSTRAINT runners_id_pk PRIMARY KEY,
  first_name VARCHAR2(30),
  last_name VARCHAR2(30)
);
```

A la cual le correspondía la siguiente secuencia para la gestión
de las claves primarias:

```sql
CREATE SEQUENCE runner_id_seq
INCREMENT BY 1
START WITH 1
MAXVALUE 50000
NOCACHE
NOCYCLE;
```

Los siguientes son ejemplos de cómo insertar nuevos participantes
en la tabla de corredores:

```sql
INSERT INTO runners
  (runner_id, first_name, last_name)
VALUES (runner_id_seq.NEXTVAL, 'Joanne', 'Everely');

INSERT INTO runners
  (runner_id, first_name, last_name)
VALUES (runner_id_seq.NEXTVAL, 'Adam', 'Curtis');
```

Las dos sentencias DDL anteriores retornan:

| RUNNER_ID | FIRST_NAME | LAST_NAME |
| --------- | ---------- | --------- |
| 1         | Joanne     | Everely   |
| 2         | Adam       | Curtis    |

**Para verificar el valor actual de `runners_id_seq`, se utiliza
`CURRVAL` de la siguiente forma:**

```sql
SELECT runner_id_seq.CURRVAL from DUAL;
```

Las secuencias almacenadas en la caché de la memoria permiten un
acceso más rápido a los valores de secuencia. La caché se llena
la primera vez que se hace referencia a una secuencia, luego las
solicitudes del siguiente valor de secuencia se recuperan de lo
que está en la caché. Después de utilizar el último valor de
secuencia, la siguiente solicitud de la secuencia introduce otra
caché de secuencias en la memoria. 20 es el número por defecto de
los números de secuencia almacenados en caché.

### Números no Secuenciales

Aunque los generadores de secuencias emiten números secuenciales
sin intervalos, esta acción se realiza independientemente de que
se realice una confirmación o un `ROLLBACK`.

Los intervalos (números no secuenciales) se pueden generar:

- Al realizar un rollback de una sentencia que contiene una
  secuencia, se pierde el número.

- Un fallo en el sistema. Si se almacenan los valores de la
  secuencia en la memoria caché y ocurre un fallo en el sistema, esos valores se pierden.

- El uso de la misma secuencia para varias tablas. Si lo hace así, cada tabla
  puede contener intervalos en los números secuenciales.

Si la secuencia se ha creado con `NOCACHE`, es posible ver el
siguiente valor de seciencia disponible sin aumentarlo con la
consulta de tabla `USER_SEQUENCES`.

```sql
SELECT sequence_name, min_value, max_value, last_number AS
"Next number"
FROM USER SEQUENCES
WHERE sequence_name = 'RUNNER_ID_SEQ';
```

La sentencia anterior devuelve:

| SEQUENCE_NAME | MIN_VALUE | MAX_VALUE | Next number |
| ------------- | --------- | --------- | ----------- |
| RUNNER_ID_SEQ | 1         | 50000     | 3           |

### Modificación de una Secuencia

Al igual que con otros objetos de BD que ha creado, `SEQUENCE`
también puede cambiarse mediante la palabra reservada `ALTER`.
Por ejemplo, ¿qué pásaria si el maratón de Londres ha excedido el
número de 50000 corredores y necesita agregar más números?. Se podría modificar la secuencia anterior e incrementar su
`MAXVALUE`:

```sql
ALTER SEQUENCE runner_id_seq
  INCREMENT BY 1
  -- Incrementando la cantidad de elementos en la secuencia
  MAXVALUE 99999
  NOCACHE
  NOCYCLE;
```

Lógicamente, **NO SE PUEDE ESTABLECER UN VALOR `MAXVALUE` MENOR
AL ACTUAL.**

Se aplican algunas reglas para poder usar `ALTER SEQUENCE`:

- Debe ser el propietario o tener el privilegio `ALTER` para
  la secuencia para poder modificarla.

- Sólo se ven afectados por la sentencia `ALTER SEQUENCE` los números de
  secuencia futuros.

- **La opción `START WITH` no se puede cambiar mediante `ALTER SEQUENCE`. La
  secuencia se debe borrar y volver a crear para reiniciar la secuencia en un
  número diferente.**

### Eliminación de Secuencias

Para eliminar una secuencia del diccionario de datos, utilice la
sentencia `DROP SEQUENCE`. Debe tener privilegios `DROP ANY SEQUENCE` o ser el propietario del objeto para poder eliminarlo.

```sql
DROP SEQUENCE runner_id_seq;
```

## 16.2 Índices y Sinónimos

Imagine que está en una librería o biblioteca
e intenta encontrar un libro, pero estos están totalmente desordenados y tiene
que examinar cada libro de cada fila de los gabinetes.

Usualmente, consultar una BD indica realizar una exploración de las tablas
completas, lo que obviamente es un proceso costoso. Afortunadamente Oracle, al
igual que otros servidores de BD cuentan con un mecanismo para hacer más eficaz
el proceso de búsqueda, los índices.

Un índice en Oracle Server es un objeto de esquema que puede acelerar la
recuperación de filas mediante un puntero. Los índices se pueden crear
explícita o automáticamente. **Si no hay un índice en la columna seleccionada, se
produce una exploración de tabla completa.** Un índice proporciona acceso directo
y rápido a las filas de una tabla. Funcionan reduciendo las operaciones de E/S
de disco mediante una ruta de acceso indexada para buscar datos de forma más
rápida.

Los índices son usados y mantenidos automáticamente por Oracle
Server. **Una vez se crea un índice, no será necesaria ninguna
intervención directa del usuario.**

Un `ROWID` es una representación de cadena en base 64 de la dirección de fila
que contiene el identificador de bloque, la ubicación de la fila en el bloque y
el identificador de archivo en la BD. Los índices utilizan los `ROWID` porque son la forma más rápida para acceder a cualquier
fila concreta.

Los índices son lógica y físicamente independientes de la tabla que indexan,
**esto significa que se pueden crear o borrar en cualquier momento sin que
afecten a las tablas base o a otros índices. AUNQUE ELIMINAR UNA TABLA TAMBIÉN ELIMINA TODOS LOS ÍNDICES QUE ESTA TUVIESE.**

### Tipos de índices

Se pueden crear dos tipos de índice en Oracle Server:

- **Índice único:** Oracle Server crea automáticamente este índice al definir
  una restricción de clave `PRIMARY KEY` o `UNIQUE` en la columna de una tabla.
  **El nombre del índice es el proporcionado a la restricción. SI DESEA CREAR
  UN ÍNDICE ÚNICO LO RECOMENDABLE ES CREAR UNA RESTRICCIÓN ÚNICA Y DEJAR QUE SE
  CREE IMPLÍCITAMENTE, AUNQUE PUEDEN CREARSE MANUALMENTE.**

- **Índice no único:** este es un índice que el usuario puede crear para acelerar el acceso a las filas. Por ejemplo, para optimizar las uniones, puede crear un índice en la columna `FOREIGN KEY`, que acelera la búsqueda de las filas coincidentes en la columna `PRIMARY KEY`.

### Creación de Índices

Para crear un índice en una o más columnas, emita la sentencia `CREATE INDEX`:

```sql
CREATE INDEX index_name
ON table_name(column, ..., column)
```

Para crear un índice en su propio esquema, necesita el privilegio `CREATE TABLE`.

Para cualquier otro esquema, necesita el privilegio `CREATE TABLE` en las
tablas sobre las que vaya a crear los índices o el privilegio `CREATE ANY INDEX`. **Los valores nulos no se incluyen en el índice.**

Por ejemplo, si se desea mejorar la velocidad de consulta a la
colummna `REGION_ID` de la tabla `WF_COUNTRIES`:

```sql
CREATE INDEX wf_cont_reg_id_idx
ON wf_countries(region_id);
```

### ¿Cuándo se debe crear un Índice?

**Se debe crear un índice sólo si:**

- La columna contiene una gran cantidad de valores.
- Una columna contiene un gran número de valores nulos.
- Una o más columnas se utilizan con frecuencia en conjunto en una cláusula
  `WHERE` o en una condición de unión.
- La tabla es grande y se espera que la mayoría de las consultas recuperen al
  menos del 2 al 4% de las filas

### ¿Cuándo NO crear un índice?

Hay ocasiones en que los índices no son tan beneficiosos o son directamente
contraproducentes:

- Cada operación DML (`INSERT`, `UPDATE` o `DELETE`) que se realiza en una
  tabla con índices indica actualizar dichos índices. Entre más índices
  asociados a una tabla, más esfuerzo debe hacer una tabla para actualizarlos
  todos después de cada operación.

**Por lo general, no merece la pena crear un índice si:**

- La tabla es pequeña
- No se suelen usar columnas como condición en la consulta.
- Se espera que la mayoría de las consultas recuperen más del 4% de la tabla
- La tabla se actualiza con frecuencia
- Se hace referencia a las columnas indexadas como parte de una expresión.

### Índice Compuesto

Un índice compuesto o "concatenado" es un índice creado en varias
columnas de una tabla. **Las columnas del índice compuesto pueden
aparecer en cualquier orden y no es necesario que sean adyacentes
en la tabla.** Los índices compuestos pueden acelerar la recuperación de datos
para las sentencias `SELECT` en las que la cláusula `WHERE` hace
referencia a todas o la parte inicial de las columnas del índice
compuesto:

```sql
CREATE INDEX emps_name_idx
ON employees(first_name, last_name);
```

**Los valores nulos no se incluyen en el índice compuesto.**

**Para optimizar las uniones, puede crear un índice en la columna `FOREIGN KEY`, que acelera la búsqueda de las filas coindicentes en la columna `PRIMARY KEY`. EL OPTIMIZADOR NO UTILIZA UN ÍNDICE SI LA CLÁUSULA `WHERE` CONTIENE LA
EXPRESIÓN `IS NULL`.**

### Confirmación de Índices

Confirme la existencia de los índices mediante la vista del diccionario
`USER_INDEXES`. También puede comprobar las columnas implicadas mediante la vista `USER_IND_COLUMNS`.

La siguiente consulta es la unión entre la tabla `USER_INDEXES` (nombres de los índices y su unicidad) y la tabla `USER_IND_COLUMNS` (nombres de los índices, nombres de tabla y nombres de colummna):

```sql
SELECT DISTINCT
ic.index_name, ic.column_name, ic.column_position, id.uniqueness
FROM user_indexes id, user_ind_columns ic
WHERE id.table_name = ic.table_name
AND ic.table_name = 'EMPLOYEES';
```

La consulta anterior regresa:

| INDEX_NAME        | COLUMN_NAME   | COLUMN_POSITION | UNIQUENESS |
| ----------------- | ------------- | --------------- | ---------- |
| EMP_EMAIL_UK      | EMAIL         | 1               | UNIQUE     |
| EMP_EMP_ID_PK     | EMPLOYEE_ID   | 1               | UNIQUE     |
| EMP_DEPARTMENT_IX | DEPARTMENT_ID | 1               | NONUNIQUE  |
| EMP_JOB_IX        | JOB_ID        | 1               | NONUNIQUE  |
| EMP_MANAGER_IX    | MANAGER_ID    | 1               | NONUNIQUE  |
| EMP_NAME_IX       | LAST_NAME     | 1               | NONUNIQUE  |
| EMP_NAME_IX       | FIRST_NAME    | 1               | NONUNIQUE  |

### Índices Basados en Funciones

Un índice basado en funciones almacena los valores indexados y
utiliza el índice basado en una sentencia `SELECT` para recuperar
los datos. Un índice basado en funciones se basa en expresiones. **Las
expresiones de índice se generan a partir de columnas de tablas, restricciones,
funciones SQL y funciones definidas por el usuario.**

**Estos índices son útiles cuando no se sabe en qué caso los datos se han almacenado los datos en la BD.**

Por ejemplo, puede crear un índice basado en funciones que se pueda utilizar con una sentencia `SELECT` utilizando `UPPER` en la cláusula `WHERE`:

```sql
CREATE INDEX upper_last_name_idx
ON employees (UPPER(last_name));
```

El índice anterior se utilizará en esta búsqueda:

```sql
SELECT *
FROM employees WHERE UPPER(last_name) = 'KING';
```

Los índices basados en funciones definidos con las palabras claves
`UPPER(column_name` y `LOWER(column_name` permiten realizar búsquedas no
sensibles a mayúsuclas y minúsculas. Esto es útil, por ejemplo, si no sabe en
qué forma se han introducido los apellidos de la BD de empleados. **Este tipo de índices no se utilizan a menos que se modifique la consulta para que la expresión en la cláusula `WHERE` coincida con el utilizado al crear el índice, o en su defecto cree uno que sí coincida con la consulta.**

Otro ejemplo de consulta que podría utilizar el índice
anteriormente creado es:

```sql
SELECT *
FROM employees WHERE UPPER(last_name) LIKE 'KIN%';
```

**PARA GARANTIZAR QUE ORACLE SERVER UTILICE EL ÍNDICE EN LUGAR
DE REALIZAR UNA EXPLORACIÓN DE TABLA COMPLETA, ASEGÚRESE DE QUE EL VALOR DE LA FUNCIÓN NO SEA NULO EN CONSULTAS POSTERIORES.**

Siguiendo con el ejemplo de índice anterior, esta sería una forma
de garantizar esta restricción:

```sql
-- usando IS NOT NULL se garantiza que Oracle utilice el índice
-- para esta consulta específica, PERO si omite la cláusula WHERE
-- Oracle podría realizar una exploración de tabla completa.
SELECT *
FROM employees WHERE UPPER(last_name) IS NOT NULL
ORDER BY UPPER(last_name);
```

**El servidor de Oracle trata los índices con columnas marcadas
como `DESC` como índices basados en funciones.**

Cabe recordar que las columnas marcadas como `DESC` se ordenan
descendentemente.

Además de `UPPER` y `LOWER`, los índices basados en funciones también soportan todas las funciones incorporadas por Oracle, como `TO_CHAR`.

Por ejemplo, la siguiente consulta:

```sql
SELECT first_name, last_name, hire_date
FROM employees
WHERE TO_CHAR(hire_date, 'yyyy') = '1987';
```

Puede llegar a ser muy costosa si el tamaño de la tabla es
suficientemente grande, incluso si hubiera un índice a la colummna `hire_date`
previamente creado, porque Oracle no va a utilizarlo debido a
la expresión `TO_CHAR`.

Para optimizar esta consulta, habría que crear un índice así:

```sql
CREATE INDEX emp_hire_year_idx
ON employees (TO_CHAR(hire_date, 'yyyy'));
```

El índice anterior sí podría ser usado en esa consulta
específica, la cual devolvería:

| FIRST_NAME | LAST_NAME | HIRE_DATE   |
| ---------- | --------- | ----------- |
| Steven     | King      | 17-Jun-1987 |
| Jennifer   | Whalen    | 17-Sep-1987 |

### Eliminación de índices

**NO SE PUEDE MODIFICAR UN ÍNDICE, LA ÚNICA OPCIÓN ES ELIMINARLO
Y VOLVERLO A CREAR**. Se puede eliminar la definición de un
índice del diccionario de datos mediante `DROP INDEX`. Debe ser
el propietario del índice o tener el privilegio `DROP ANY INDEX`.
**Si borra una tabla, todas sus restricciones e índices se borran
con ella, pero permanecen las vistas y las secuencias:**

```sql
DROP INDEX upper_last_name_idx;
DROP INDEX emps_name_idx;
DROP INDEX emp_hire_year_idx;
```

### SYNONYM

En el lenguaje SQL, un sinónimo es una palabra o expresión que es
un substituto aceptado de otra palabra. Estos se utilizan para
simplificar el acceso a objetos creando otro nombre para el
objeto. Con ellos se puede hacer referencia a una tabla
propiedad de otro usuario de forma más sencilla y reducir los
nombres de objetos.

Por ejemplo, para hacer referencia a la tabla `amy_copy_employees` del esquema de otro usuario, puede poner
como prefijo el nombre de la tabla con el nombre del usuario que
la creó seguido de un punto, y a continuación, el nombre de la
tabla, como en `USMA_SBHS_SQL01_S04.amy_copy_employees`.
Con la creación de un sinónimo se elimina la necesidad de
cualificar el nombre del objeto con el esquema y se ofrecen
nombres alternativos para objetos de la BD, esto es útil para
objetos con nombres muy largos.

El DBA puede crear sinónimos públicos para el uso de todos los
usuarios, o puede otorgar el privilegio `CREATE PUBLIC SYNONYM`
para que cada usuario cree sus propios sinónimos públicos.

La sintáxis para crearlos es la siguiente:

```sql
-- PUBLIC: crea un sinónimo accesible para todos los usuarios
-- synonym: nombre del sinónimo
-- object: identifica el objeto para el que se crea el sinónimo
CREATE [PUBLIC] SYNONYM synonym
FOR object;
```

#### Directrices de SYNONYM

- El objeto no puede estar un paquete.
- Un nombre de sinónimo privado debe ser distinto de todos los
  demás objetos propiedad del mimo usuario.

#### Eliminar un Sinónimo

~~~sql
DROP [PUBLIC] SYNONYM name_of_synonym;

DROP SYNONYM amy_emps;
~~~

#### Confirmación de SYNONYM

La existencia de sinónimos se puede confirmar mediante la vista del diccionario de datos `USER_SYNONYMS`, la cual contiene:

- `synonymm_name`: nombre del sinónimo
- `table_name`: propietario del objeto al que hace referencia.
- `table_owner`: nombre del objeto al que hace referencia.
- `db_link`: enlace de base de datos al que se hace referencia en un sinónimo remoto.


