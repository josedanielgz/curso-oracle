Parte 14

- [14.1 Introducción a las Restricciones; Restricciones NOT NULL y UNIQUE](#141-introducción-a-las-restricciones-restricciones-not-null-y-unique)
  - [Restricciones en General](#restricciones-en-general)
  - [Restricciones de Nomenclatura](#restricciones-de-nomenclatura)
    - [A nivel de Columna](#a-nivel-de-columna)
    - [A nivel de Tabla](#a-nivel-de-tabla)
  - [Reglas Básicas para Restricciones](#reglas-básicas-para-restricciones)
    - [Ejemplos de Violaciones de las Reglas](#ejemplos-de-violaciones-de-las-reglas)
  - [Cinco Tipos de Restricciones](#cinco-tipos-de-restricciones)
    - [NOT NULL](#not-null)
    - [UNIQUE](#unique)
  - [Definición de restricciones UNIQUE](#definición-de-restricciones-unique)
  - [Restricciones creadas en la creación de la tabla](#restricciones-creadas-en-la-creación-de-la-tabla)
- [14.2 Restricciones PRIMARY KEY, FOREIGN KEY y CHECK](#142-restricciones-primary-key-foreign-key-y-check)
  - [Restricciones PRIMARY KEY](#restricciones-primary-key)
  - [FOREIGN KEY](#foreign-key)
    - [Visualización de una Clave Ajena](#visualización-de-una-clave-ajena)
    - [Restricción de Integridad Referencial](#restricción-de-integridad-referencial)
  - [Sintaxis de Restricción FOREIGN KEY](#sintaxis-de-restricción-foreign-key)
  - [Mantenimiento de Integridad Referencial](#mantenimiento-de-integridad-referencial)
    - [ON DELETE CASCADE](#on-delete-cascade)
      - [Sintaxis de ON DELETE CASCADE](#sintaxis-de-on-delete-cascade)
    - [ON DELETE SET NULL](#on-delete-set-null)
    - [Restricciones CHECK](#restricciones-check)
- [14.3 Gestión de Restricciones](#143-gestión-de-restricciones)
  - [Gestión de Restricciones](#gestión-de-restricciones)
  - [Sentencia ALTER](#sentencia-alter)
    - [Adición De Restricciones](#adición-de-restricciones)
    - [Ejemplos de Adición de Restricciones](#ejemplos-de-adición-de-restricciones)
    - [Condiciones para la Adición de Restricciones](#condiciones-para-la-adición-de-restricciones)
  - [¿Por qué activar y desactivar restricciones?](#por-qué-activar-y-desactivar-restricciones)
  - [Borrado de Restricciones](#borrado-de-restricciones)
  - [Desactivación de Restricciones](#desactivación-de-restricciones)
    - [Uso de la Cláusula DISABLE](#uso-de-la-cláusula-disable)
    - [Uso de la Cláusula CASCADE](#uso-de-la-cláusula-cascade)
  - [Activación de Restricciones](#activación-de-restricciones)
  - [Consideraciones sobre la Activación de Restricciones](#consideraciones-sobre-la-activación-de-restricciones)
  - [Restricciones en Cascada](#restricciones-en-cascada)
    - [Cuando No Es Necesario CASCADE](#cuando-no-es-necesario-cascade)
  - [Visualización de Restricciones](#visualización-de-restricciones)
    - [Consulta USER_CONSTRAINTS](#consulta-user_constraints)

- [Volver al inicio](index.html)

## 14.1 Introducción a las Restricciones; Restricciones NOT NULL y UNIQUE

Las BBDD sólo son tan fiables como los datos que contienen, para garantizar su
fiabilidad existen una serie de reglas que se aplican (tanto a conveniencia del
usuario como automáticamente) para proteger la integridad de estos datos. Las
restricciones son reglas que sirven para evitar la entrada de datos no válidos
en las tablas.

Por ejemplo, tendría poco sentido que los salarios de los empleados pudieran ser números negativos, o que más de un alumno
tuviera el mismo identificador.

### Restricciones en General

¿Qué es exactamente una restricción? son reglas de la base de datos. Todas las
restricciones se almacenan en el diccionario de datos. Las restricciones, entre
otras cosas, evitan que una tabla pueda ser borrada si otra tabla la está
referenciando, y se aplican a los datos cuando se inserta/actualiza/suprime una
fila de la tabla.

Asignar nombres descriptivos a las restricciones es importante, porque si no
será difícil destinguirlas entre sí, complicando el trabajo.

A la hora de crear una tabla, en la sentencia `CREATE TABLE` (como en el
siguiente ejemplo) se define cada columna y su tipo de dato, pero también puede
establecer restricciones para cada columna de la tabla.

Existen dos formas de agregar restricciones a una tabla:

- A nivel de la columna junto al tipo de dato, en el área de
  la sentencia `CREATE TABLE`.
- A nivel de la tabla una vez se hayan declarado todos los nombres
  de columna.

Por ejemplo, para la siguiente tabla:

```sql
CREATE TABLE clients
(
  client_number NUMBER(4),
  first_name VARCHAR2(14),
  last_name VARCHAR2(13)
);
```

UNa forma de agregar una restricción de columna es:

```sql
CREATE TABLE clients
(
  client_number NUMBER(4) CONSTRAINT clients_client_num_pk PRIMARY KEY,
  first_name VARCHAR2(14),
  last_name VARCHAR2(13)
);
```

Explicando:

- la restricción se llama `clients_client_num_pk`
- La regla que se aplica es que la columna `client_number` es la
  clave primaria.

### Restricciones de Nomenclatura

Todas las restricciones de la BD tienen un nombre. Puede asignarlo el que escribe la sentencia (como en el ejemplo anterior) o puede dejar que el sistema le asigne un nombre automáticamente, de la forma `SYS_C(n. entero único)`, aunque estos nombres suelen ser poco legibles, como `SYS_C00585417`.

Es mejor adoptar una convención y nombrarlas manualmente, como por ejemplo `nombre-tabla_nombre-columna_tipo-restricción`.
**Usar la palabra `CONSTRAINT` en una definición `CREATE TABLE`
fuerza obligatoriamente a escribir un nombre de restricción. EL NOMBRE DE UNA RESTRICCIÓN NO PUEDE SER MÁS LARGO QUE 30 CARACTERES.**

#### A nivel de Columna

Algunos ejemplos de la convención descrita anteriormente para
restricciones a nivel de columna son:

```sql
CREATE TABLE clients
(
  -- una restricción de clave primaria en client_number
  client_number NUMBER(4) CONSTRAINT clients_client_num_pk PRIMARY KEY,

  -- una restricción NOT NULL en last_name
  last_name VARCHAR2(14) CONSTRAINT clients_last_name_nn NOT NULL,

  -- una restricción única en una dirección de correo electrónico
  last_name VARCHAR2(13) CONSTRAINT clients_email_uk UNIQUE
);
```

**El ejemplo anterior, pero dejando que el sistema le asigne los
nombres a algunas de las restricciones:**

```sql
CREATE TABLE clients
(
  -- una restricción de clave primaria en client_number
  client_number NUMBER(4) CONSTRAINT clients_client_num_pk PRIMARY KEY,

  -- una restricción NOT NULL en last_name con nombre autoasignado
  last_name VARCHAR2(14) NOT NULL,

  -- una restricción única en una dirección de correo electrónico con nombre autoasignado
  last_name VARCHAR2(13) UNIQUE
);
```

#### A nivel de Tabla

Estas restricciones se muestran por separado de las definiciones
de columna en la sentencia `CREATE TABLE`. Estas se muestran
después de definir todas las columnas de la tabla.

Por ejemplo:

```sql
CREATE TABLE clients
(
  client_number NUMBER(6),
  first_name VARCHAR2(20),
  last_name VARCHAR2(20),
  phone VARCHAR2(20),
  email VARCHAR2(10)

  -- restricción de combinación única de email y teléfono
  CONSTRAINT clients_phone_email_uk UNIQUE (email,phone);
);
```

### Reglas Básicas para Restricciones

- **Las restricciones que hacen referencia a más de una columna se
  deben especificar en el nivel de tabla.**

- **`NOT NULL` sólo puede definirse a nivel de columna, no de tabla**

- Las restricciones `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY` y `CHECK` pueden
  usarse tanto a nivel de columna como de tabla.

- Por obligación, si se utiliza la palabra `CONSTRAINT` en la creación de una
  tabla, se está obligado a declarar un nombre y una restricción de columna.

#### Ejemplos de Violaciones de las Reglas

```sql
CREATE TABLE clients
(
  client_number NUMBER(6),
  first_name VARCHAR2(20),
  last_name VARCHAR2(20),

  -- LA SIGUIENTE RESTRICCIÓN ESTÁ MAL, NO SE PUEDEN DEFINIR
  -- UNA RESTRICCIÓN DE CLAVE COMPUESTA A NIVEL DE COLUMNA
  phone VARCHAR2(20) CONSTRAINT phone_email_ul UNIQUE (email,phone),

  -- LA SIGUIENTE RESTRICCIÓN ESTÁ MAL PORQUE EL TÉRMINO CONSTRAINT DEBE IR SEGUIDO DE UN NOMBRE DE RESTRICCIÓN
  email VARCHAR2(10) CONSTRAINT NOT NULL,

  -- LAS RESTRICCIONES DE NOT NUL SOLO SE PUEDEN DEFINIR A NIVEL
  -- DE COLUMNA
  CONSTRAINT email_clients_email NOT NULL,
  CONSTRAINT clients_client_num_pk PRIMARY KEY (client_number)
);

```

### Cinco Tipos de Restricciones

Existen cinco tipos de restricciones en una BD de Oracle: `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY` y `CHECK`.

#### NOT NULL

Una columna definida como `NOT NULL` neccesita que para cada fila
introducida en la tabla debe existir un valor para esa columna.

Por ejemplo:

```sql
CREATE TABLE clients
(
  client_number NUMBER(4) CONSTRAINT clients_client_num_pk PRIMARY KEY,

  -- Creando una restricción no nula para el apellido del empleado
  last_name VARCHAR2(14) NOT NULL,

  last_name VARCHAR2(13) UNIQUE
);
```

#### UNIQUE

Una restricción `UNIQUE` necesita que todos los valores de una
columna o juego de columnas (clave compuesta) sean únicos; o sea,
que todos los valores de la columna tienen que ser diferentes.

**A un cojunto de una o más columnas que se definen como `UNIQUE` se denomina clave única.**

**Si la combinación de dos o más columnas debe ser única para
cada entrada, se dice que la restricción es una clave única
compuesta.**

**LA PALABRA CLAVE HACE REFERENCIA A LAS COLUMNAS, NO A LOS NOMBRES
DE RESTRICCIONES.**

UN ejemplo de restricción UNIQUE es el siguiente: una tabla `clients` tiene los
siguientes datos:

| CLIENT_NUMBER | FIRST_NAME | LAST_NAME | PHONE      | EMAIL                  |
| ------------- | ---------- | --------- | ---------- | ---------------------- |
| 5922          | Hiram      | Peters    | 3715832249 | hpeters@yahoo.com      |
| 5857          | Serena     | Jones     | 7035335900 | serena.jones@jones.com |
| 6133          | Lauren     | Vigil     | 4072220090 | lbv@lvb.net            |

Y tiene una restricción llamada `client_email_uk` que establece
que dos clientes no pueden compartir un correo electrónico.

Si intentaramos insertar el siguiente cliente:

```sql
INSERT INTO clients
  (client_number, first_name, last_name, phone, email)
VALUES (7234,'Lonny','Vigil',4072220090,'lvb@lvb.net')
```

Esto no se podría realizar, se arrojaría el siguiente error:

```sql
ORA-00001: unique constraint (USWA_SKHS_SQL01_T01.CLIENT_EMAIL_UK) violated
```

### Definición de restricciones UNIQUE

Es habitual usar el sufijo `uk` a la hora de realizar restricciones de clave única. Para definir una clave única
compuesta, esta práctica no es diferente, pero tiene una diferencia crucial: estas sólo pueden declararse al final de
la definición de las tablas:

```sql
CONSTRAINT clients_phone_email_uk UNIQUE(email,phone)
```

**`UNIQUE` permite la entrada de valores nulos por sí solo,
porque un valor nulo en una columna (o en todas las columnas
únicas de la clave compuesta) NO SE CONSIDERA IGUAL A NADA.**

Respecto al ejemplo anterior, considere que cambiamos tanto la restricción
como el valor que ibamos a introducir inicialmente, y ahora tiene la siguiente
salida:

| CLIENT_NUMBER | FIRST_NAME | LAST_NAME | PHONE          | EMAIL                  |
| ------------- | ---------- | --------- | -------------- | ---------------------- |
| 5922          | Hiram      | Peters    | 3715832249     | hpeters@yahoo.com      |
| 5857          | Serena     | Jones     | 7035335900     | serena.jones@jones.com |
| 6133          | Lauren     | Vigil     | **4072220090** | **lbv@lbv.net**        |
| 6133          | Lonny      | Vigil     | **4072220091** | **lbv@lbv.net**        |

Considere los valores resaltados en las últimas dos filas. Note
cómo `EMAIL` tiene el mismo valor en ambos empleados, pero el
número de teléfono es diferente. Eso se debe a la nueva restricción
`client_phone_email_uk` reemplazando a `client_email_uk`.
**Esta nueva restricción compuesta establece que ninguna combinación de los
valores `PHONE` e `EMAIL` debe ser igual a otra.**

### Restricciones creadas en la creación de la tabla

Al agregar una restricción `NOT NULL` como parte de la
sentencia de creación de una tabla, Oracle creará una
restricción Check en la BD para aplicar un valor en la
columna `NOT NULL`. Este proceso es invisible para el usuario
final, quien solo ve la confirmación de la tabla creada.

## 14.2 Restricciones PRIMARY KEY, FOREIGN KEY y CHECK

Las restricciones sirven para garantizar la integridad de los datos en una BD, que es posiblemente la operación más importante que debe realizar un RDBMS; una restricción resalta sobre las
demás, la de clave primaria, pues las claves primarias garantizan que no existan inconsistencias a la hora de identificar una fila específica.

### Restricciones PRIMARY KEY

UNa restricción `PRIMARY KEY` es una regla que establece que los valores de un conjunto de una o más columnas debe identificar
de forma única cada fila de una tabla.
Una clave primaria sigue las siguientes reglas:

- **NINGUNA COLUMNA QUE FORME PARTE DE LA CLAVE PRIMARIA PUEDE
  CONTENER UN VALOR NULO.**

- **SÓLO PUEDE HABER UNA CLAVE PRIMARIA POR TABLA.**

- **NO PUEDE APARECER NINGÚN VALOR DE CLAVE PRIMARIA EN MÁS DE
  UNA FILA DE LA TABLA.**

Las restricciones de clave primaria pueden definirse a nivel de
columna o de tabla, **excepto si es una clave compuesta, en cuyo caso debe
definirse a nivel de tabla.**

Por ejemplo:

```sql
CREATE TABLE clients
(
  client_number NUMBER(4) CONSTRAINT clients_client_num_pk PRIMARY KEY,
  first_name VARCHAR2(14),
  last_name VARCHAR2(13)
);
-- en la definición de tabla anterior se definió una restricción
-- de clave primaria a nivel de columna, con una sola columna,
-- client_number
```

En el siguiente ejemplo, se define una clave primaria compuesta:

```sql
CREATE TABLE copy_job_history
(
  employee_id number(6,0),
  start_date DATE,
  job_id VARCHAR2(10),
  department_id NUMBER(4,0),
  CONSTRAINT copy_jhist_id_st_date_pk PRIMARY KEY(employee_id,start_date)
);
-- la anterior fue la definición de una clave primaria compuesta
-- a nivel de tabla, usando las columnas employee_id y start_date
```

### FOREIGN KEY

**Las restricciones `FOREIGN KEY` también se conocen como restricciones de
integridad referencial.** En este tipo de restricciones se designa un conjunto de una o más columnas como una clave ajena,
esta enlaza la **clave primaria (o única)** de una tabla en otra,
dicho enlace establece la relación entre ambas tablas.

#### Visualización de una Clave Ajena

La table que contiene la clave ajena se denomina **tabla secundaria**, y la que contiene la clave referenciada se
denomina **tabla principal**.

Por ejemplo, considere la relación entre las siguientes tablas:

La siguiente sería la tabla principal (departamentos):

| **DEPARTMENT_ID** | DEPT_NAME   | MANAGER_ID | LOCATION_ID |
| ----------------- | ----------- | ---------- | ----------- |
| 90                | Executive   | 100        | 1700        |
| 110               | Accounting  | 205        | 1700        |
| 190               | Contracting | -          | 1700        |

Esta sería la tabla secundaria (empleados):

| EMPLOYEE_ID | FIRST_NAME | LAST_NAME | **DEPARTMENT_ID** |
| ----------- | ---------- | --------- | ----------------- |
| 100         | Steven     | King      | 90                |
| 101         | Neena      | Kochhar   | 90                |
| 102         | Lex        | De Haan   | 90                |

La clave que conecta a ambas tablas es `department_id`. Nótese el hecho de que las claves ajenas pueden repetirse en
la tabla secundaria sin que esto afecte la restricción que debe
cumplir la clave primaria en la tabla principal.

#### Restricción de Integridad Referencial

Para cumplir una restricción de integridad referencial, la clave
ajena en la tabla secundaria debe coincidir con un valor existente de la tabla
principal o ser un valor `NULL`.

**Puede existir un valor de clave primaria sin un valor de clave
ajena correspondiente; SIN EMBARGO, UNA CLAVE AJENA DEBE TENER
UNA CLAVE PRIMARIA CORRESPONDIENTE**

La regla es: antes de definir una restricción de integridad referencial en la tabla secundaria, ya debe estar definida la
restricción `UNIQUE` o `PRIMARY` a la que se hace referencia.
**ES DECIR, PRIMERO DEBE HABER UNA CLAVE PRINCIPAL O ÚNICA EN LA
TABLA PRIMARIA PARA QUE PUEDA HABER UNA CLAVE AJENA EN LA TABLA
SECUNDARIA.**

### Sintaxis de Restricción FOREIGN KEY

La sintaxis para definir una `FOREIGN KEY` necesita una referencia a la tabla y
columna de la tabla principal. Estas pueden definirse tanto a
nivel de columna como a nivel de tabla, por ejemplo:

```sql
CREATE TABLE copy_employees
(
  employee_id NUMBER(6,0) CONSTRAINT copy_emp_pk PRIMARY KEY,
  first_name VARCHAR2(20),
  last_name VARCHAR2(25),
  -- la siguiente es una restricción de clave ajena a nivel
  -- de columna usando la columna department_id de la tabla
  -- departments
  department_id NUMBER(4,0) CONSTRAINT c_emps_dept_id_fk REFERENCES departments(department_id),
  email VARCHAR(25)
);
```

Al igual que con las restricciones de clave primaria, también
se puede declarar restricciones de clave ajena a nivel de tabla,
lo cual es necesario cuando se intenta crear una clave ajena compuesta.

```sql
CREATE TABLE copy_employees
(
  employee_id NUMBER(6,0) CONSTRAINT copy_emp_pk PRIMARY KEY,
  first_name VARCHAR2(20),
  last_name VARCHAR2(25),
  department_id NUMBER(4,0),
  email VARCHAR(25)
  -- la siguiente es una restricción de clave ajena a nivel
  -- de tabla equivalente al ejemplo anterior
  CONSTRAINT c_emps_dept_id_fk FOREIGN KEY (department_id)
    REFERENCES departments(department_id);
);
```

### Mantenimiento de Integridad Referencial

#### ON DELETE CASCADE

El uso de la opción `ON DELETE CASCADE` al definir una clave
ajena permite la supresión de filas dependientes en la tabla
secundaria cuando se suprime una fila de la tabla principal. Esto
se hace agregando el permiso `ON DELETE CASCADE` en la cláusula
`FOREIGN KEY` al momento de crear la restricción en la tabla
secundaria.

**Sin la clave ajena no tiene una opción ON DELETE, las
filas a las que hace referencia de la tabla principal no se
pueden suprimir.** Por ejemplo, retomando el ejemplo de
las tablas:

La tabla de copy_departments:

| **DEPARTMENT_ID** | DEPT_NAME   | MANAGER_ID | LOCATION_ID |
| ----------------- | ----------- | ---------- | ----------- |
| 90                | Executive   | 100        | 1700        |
| 110               | Accounting  | 205        | 1700        |
| 190               | Contracting | -          | 1700        |

La tabla copy_employees:

| EMPLOYEE_ID | FIRST_NAME | LAST_NAME | **DEPARTMENT_ID** |
| ----------- | ---------- | --------- | ----------------- |
| 100         | Steven     | King      | 90                |
| 101         | Neena      | Kochhar   | 90                |
| 102         | Lex        | De Haan   | 90                |

Si a la columna `department_id` de la tabla empleados se le creó con el permiso
`ON DELETE CASCADE` al crear una `FOREIGN KEY`, eliminar un `department_id` de
la tabla de departamentos eliminará todos los empleados asociados a ese
departamento, si no no se podrá eliminar ningún departamento que tenga
empleados referenciados.

##### Sintaxis de ON DELETE CASCADE

La siguiente es la sentencia que se usó para crear la tabla
`copy_employees` sin el permiso `ON DELETE CASCADE`:

```sql
CREATE TABLE copy_employees
(
  employee_id NUMBER(6,0) CONSTRAINT copy_emp_pk PRIMARY KEY,
  first_name VARCHAR2(20),
  last_name VARCHAR2(25),
  department_id NUMBER(4,0),
  email VARCHAR2(25),
  CONSTRAINT cdept_dept_id_fk FOREIGN KEY (department_id)
    REFERENCES copy_departments(departments)
);
```

Basándose en el contenido de las tablas anteriores, si se intenta eliminar el
departamento con id 90, se produciría un error de violación de restricción de
integridad.

La siguiente es la sentencia que se necesitaría para crear la
tabla con el permiso `ON DELETE CASCADE` correcto:

```sql
CREATE TABLE copy_employees
(
  employee_id NUMBER(6,0) CONSTRAINT copy_emp_pk PRIMARY KEY,
  first_name VARCHAR2(20),
  last_name VARCHAR2(25),
  department_id NUMBER(4,0),
  email VARCHAR2(25),
  -- ejemplo de cómo crear una constancia
  -- de integridad con ON DELETE CASCADE
  CONSTRAINT cdept_dept_id_fk FOREIGN KEY (department_id)
    REFERENCES copy_departments(departments) ON DELETE CASCADE
);
```

Si la tabla se hubiera creado de esta forma, se podría suprimir
la fila correctamente, lo que eliminaría también a todos los
empleados que dependieran de ese departamento.

#### ON DELETE SET NULL

Una opción alternativa a eliminar las filas dependientes con
`ON DELETE CASCADE`, es establecer el valor de las claves ajenas
con valores nulos mediante la restricción `ON DELETE SET NULL`.
Esta se utiliza de la misma forma que su contraparte:.
Esta se utiliza de la misma forma que su contraparte:

```sql
CREATE TABLE copy_employees
(
  employee_id NUMBER(6,0) CONSTRAINT copy_emp_pk PRIMARY KEY,
  first_name VARCHAR2(20),
  last_name VARCHAR2(25),
  department_id NUMBER(4,0),
  email VARCHAR2(25),
  -- ejemplo de cómo crear una constancia
  -- de integridad con ON DELETE SET NULL
  CONSTRAINT cdept_dept_id_fk FOREIGN KEY (department_id)
    REFERENCES copy_departments(departments) ON DELETE SET NULL
);
```

Esto podría ser de utilidad cuando el valor de la tabla principal
se va a cambiar a un nuevo valor, y cuando no desea suprimir las
filas en la tabla secundaria. Actualizar valores en las filas de
esta forma acelera el proceso, pues no es necesario introducir
de vuelta todos los datos de los registros de la tabla secundaria.

#### Restricciones CHECK

Las restricciones `CHECK` definen explícitamente una condición
que debe cumplirse. Para que se cumpla la restricción, cada una
de las filas de la tabla debe hacer que una condición sea `TRUE`
o `NULL` (debido a que este representa un valor desconocido).
**La condición de una restricción `CHECK` puede hacer referencia a cualquier columna de la tabla especificada, PERO NO A COLUMNAS DE OTRAS TABLAS.**

Las restricciones `CHECK` pueden definirse tanto a nivel de tabla como de columna, la sintáxis para utilizarlas es la siguiente:

```sql
-- restricción CHECK a nivel de columna
salary NUMBER(8,2) CONSTRAINT employees_min_sal_ck CHECK (salary>0)

-- restricción CHECK a nivel de tabla
CONSTRAINT employees_min_sal_ck CHECK (salary>0)
```

Por ejemplo, la siguiente restricción garantiza que un valor
introducido para `end_date` sea posterior a `start_date`:

```sql
CREATE TABLE copy_job_history
(
  employee_id NUMBER(6,0),
  start_date DATE,
  end_date DATE,
  job_id VARCHAR2(10),
  department_id NUMBER(4,0),
  CONSTRAINT cjhist_emp_id_st_date_pk
    PRIMARY KEY(employee_id,start_date),
 -- la siguiente es una restricción para comprobar que
 -- un empleado no haya terminado un trabajo algun día antes
 -- que el día que supuestamente se contrató
  CONSTRAINT cjhist_end_ck CHECK
    (end_date > start_date)
);
```

**Esta restricción CHECK, y en general cualquiera que referencie
más de una colummna de la tabla, debe definirse en el nivel de
tabla.**

- Una restricción `CHECK` sólo debe estar en la fila en que se
  define la restricción.

- **Una restricción `CHECK` NO SE PUEDE UTILIZAR EN CONSULTAS QUE
  HACEN REFERENCIAS A VALORES DE OTRAS FILAS.**

- **Las restricciones `CHECK` NO PUEDEN CONTENER LLAMADAS A LAS FUNCIONES `SYSDATE`, `UID`, `USER` o `USERENV`.**

  - Por ejemplo, la sentencia `CHECK(SYSDATE>'05-May-1999')` no
    está permitida.

- **La restricción `CHECK` tampoco puede utilizar las pseudocolumnas `CURRVAL`, `NEXTVAL`, `LEVEL` o `ROWNUM`.**

  - Por ejemplo, la sentencia `CHECK(NEXTVAL>0)` no está
    permitida.

- NO existe un límite en cuanto al número de restricciones
  `CHECK` que puede definirse en una columna.

## 14.3 Gestión de Restricciones

UN sistema de BBDD tiene que poder aplicar reglas de negocio y, al mismo
tiempo, evitar la adición, modificación o supresión de datos que pueda dar como
resultado una violación de la integridad de los datos. Sería catastrófico que, por ejemplo, un banco
pudiera asignarle el mismo número de tarjeta de crédito a más de una persona, o
que más de un estudiante se identificara con el mismo código en una
institución.

### Gestión de Restricciones

La sentencia `ALTER TABLE` se utiliza para realizar cambios en
las restricciones de las tablas existentes. Estos cambios pueden
incluir: agregar o borrar restricciones, activarlas o desactivarlas, y agregar restricciones `NOT NULL`.
Las directrices para realizar cambios en las restricciones son:

- Puede agregar, borrar, activar o desactivar su restricción, **pero no puede modificar su estructura**.

- Puede agregar una restricción `NOT NULL` a una columna existente mediante la
  cláusula `MODIFY` de la sentencia `ALTER TABLE`.

- `MODIFY` se utiliza porque `NOT NULL` es un cambi de nivel de
  columna.

- **Sólo puede definir una restricción `NOT NULL` si la tabla está
  vacía o si la columna tiene un valor para cada fila.**

### Sentencia ALTER

La sentencia `ALTER` necesita lo siguiente:

- nombre de la tabla
- nombre de la restricción
- tipo de restricción
- nombre de la columna a la que afecta la restricción

En el ejemplo de código se muestra a continuación, con la tabla de empleados,
la restricción de clave principal se puede haber agregado después de que se
creara originalmente la tabla:

```sql
ALTER TABLE employees ADD CONSTRAINT emp_id_pk PRIMARY KEY (employee_id);
```

#### Adición De Restricciones

Para agregar una restricción a una tabla existente, utilice la
siguiente sintaxis:

```sql
ALTER TABLE table_name ADD [CONSTRAINT constraint_name] type_of_constraint (column_name);
```

Si la restricción es para agregrar una clave ajena, se debe
incluir la palabra `REFERENCES` de la siguiente forma:

```sql
ALTER TABLE table_name
  ADD CONSTRAINT constraint_name
    FOREIGN KEY (column_name)
    REFERENCES (column_name);
```

#### Ejemplos de Adición de Restricciones

Considere la BD de empleados. La clave primaria de la tabla
`departments` se introduce en la tabla `employees` como una
clave ajena:

| **DEPARTMENT_ID** | DEPT_NAME   | MANAGER_ID | LOCATION_ID |
| ----------------- | ----------- | ---------- | ----------- |
| 90                | Executive   | 100        | 1700        |
| 110               | Accounting  | 205        | 1700        |
| 190               | Contracting | -          | 1700        |

Esta sería la tabla secundaria (empleados):

| EMPLOYEE_ID | FIRST_NAME | LAST_NAME | **DEPARTMENT_ID** |
| ----------- | ---------- | --------- | ----------------- |
| 100         | Steven     | King      | 90                |
| 101         | Neena      | Kochhar   | 90                |
| 102         | Lex        | De Haan   | 90                |

La siguiente es la sentencia que agrega esta clave ajena a
la tabla `employees`

```sql
ALTER TABLE employees ADD CONSTRAINT emp_dept_fk
  FOREIGN KEY (department_id)
  REFERENCES departments(department_id)
ON DELETE CASCADE;
```

#### Condiciones para la Adición de Restricciones

- Si la restricción es una restricción `NOT NULL`, la sentencia
  `ALTER TABLE` utiliza `MODIFY` en lugar de `ADD`

- Las restricciones `NOT NULL` sólo pueden agregarse si la tabla está vacía o
  si la columna tiene un valor para cada fila.

Ejemplo de la restricción `NOT NULL`:

```sql
ALTER TABLE employees
  MODIFY (email CONSTRAINT emp_email_nn NOT NULL);
```

### ¿Por qué activar y desactivar restricciones?

**Para aplicar las reglas definidas por restricciones de integridad, las
restricciones deben estar siempre activadas.**

En determinadas situaciones, se recomienda desactivar temporalmente las restricciones de integridad de una tabla
por motivos de rendimiento, por ejemplo:

- Al cargar grandes cantidades de datos en una tabla.

- Al ejecutar operaciones por lotes que realizan cambios masivos en una tabla (como puede ser cambiar el número de empleado de todas las personas agregando 1000 al número existente).

### Borrado de Restricciones

Para borrar una restricción, debe saber su nombre. Si no lo sabe, puede buscar el nombre de lar restricción en `USER_CONSTRAINTS` y `USER_CONS_COLUMNS` en el diccionario de datos.

**Tenga en cuenta que al borrr una restricción de integridad, Oracle Server ya
no aplica esa restricción y deja de estar disponible para el diccionario de
datos.**

**No se borra ninguna fila ni dato en ninguna de las tablas afectadas al borrar
una restricción.**

```sql
-- sintaxis para eliminar una restricción
ALTER TABLE table_name DROP CONSTRAINT name [CASCADE]

-- ejemplo de borrado de restricción
ALTER TABLE copy_departments
DROP CONSTRAINT c_dept_dept_id_pk CASCADE;
```

### Desactivación de Restricciones

Por defecto, siempre que una restricción de integridad esté definida en una
sentencia `CREATE` o `ALTER TABLE`, Oracle activa (aplica) automáticamente la
restricción, a menos que se cree específicamente con un estado desactivado con
la cláusula `DISABLE`.

- Puede desactivar una restricción sin borrarla o volverla a crear mediante
  `DISABLE` en la opción `ALTER TABLE`.

- `DISABLE` permite datos entrantes, tanto si se ajustan a la restricción como si
  no.

- Esta función permite agregar datos en una tabla secundaria
  sin tener los valores correspondientes en la tabla principal,
  `DISABLE` simplemente desactiva la restricción.

#### Uso de la Cláusula DISABLE

Puede utilizar `DISABLE` tanto en la sentencia `ALTER TABLE` como
en la sentencia `CREATE TABLE`.

```sql
CREATE TABLE copy_employees
(
  employee_id NUMBER(6,0) PRIMARY KEY DISABLE,
  ...,
  ...,
);

ALTER TABLE copy_employees
DISABLE CONSTRAINT c_emp_dept_id_fk;
```

**La desactivación de una restricción `UNIQUE` o `PRIMARY KEY` elimina el
índice único.**

#### Uso de la Cláusula CASCADE

La cláusula `CASCADE` desactiva las restricciones de integridad
dependientes. **Si la restricción se activa posteriormente, LAS
RESTRICCIONES DEPENDIENTES NO SE ACTIVAN AUTOMÁTICAMENTE.**

Sintáxis y ejemplo:

```sql
-- sintáxis de CASCADE
ALTER TABLE table_name
DISABLE CONSTRAINT constraint_name [CASCADE];

-- ejemplo de CASCADE
ALTER TABLE copy_departments
DISABLE CONSTRAINT c_dept_dept_id_pk CASCADE;
```

### Activación de Restricciones

Para activar una restricción de integridad actualmente desactivada, utilice la cláusula `ENABLE` en la sentencia
`ALTER TABLE`. **CON `ENABLE` SE GARANTIZA QUE TODOS LOS DATOS ENTRANTES SE AJUSTEN A LA RESTRICCIÓN.**

Sintáxis y ejemplo:

```sql
-- sintaxis de ENABLE
ALTER TABLE table_name ENABLE CONSTRAINT constraint_name;

-- ejemplo de ENABLE
ALTER TABLE copy_departments
ENABLE CONSTRAINT c_dept_dept_id_pk ;
```

Puede utilizar la cláusula `ENABLE` tanto en la sentencia `CREATE TABLE` como en la sentencia `ALTER TABLE`.

### Consideraciones sobre la Activación de Restricciones

- Si activa una restricción, si aplica a todos los datos de la tabla.

- Todos los datos de la tabla deben cumplir la restricción.

- Si activa una clave `UNIQUE` o una restricción `PRIMARY KEY`, se crea un
  índice `UNIQUE` o `PRIMARY KEY` automáticamente.

- **La activación de una restricción `PRIMARY KEY` desactivada con la opción
  `CASCADE` no activa ninguna clave ajena dependiente de la clave primaria**

- `ENABLE` vuelve a activar la restricción después de desactivarla.

### Restricciones en Cascada

- Las restricciones de integridad referencial en cascada definen las acciones que
  lleva a cabo el servidor de BD cuando un usuario intenta suprimir o actualizar
  una clave a las que apuntan las claves ajenas existentes.

- La cláusula `CASCADE CONSTRAINTS` se utiliza junto con la cláusula `DROP COLUMN`.

- Borra todas las restricciones de integridad referencial que hacen referencia a las claves primarias y únicas definidas en
  las columnas borradas.

- Borra también todas las restricciones de varias columnas definidas en las
  columnas borradas.

- **Si una sentencia `ALTER TABLE` no incluye la opción de `CASCADE CONSTRAINTS`, cualquier intento de borrar una restricción de clave primaria o
  varias columnas fallará.**

- **Recuerde que no puede suprimir un valor principal si existen valores
  secundarios en otras tablas.**

#### Cuando No Es Necesario CASCADE

Si las columnas a las que hacen referencia las restricciones
definidas en las columnas borradas también se borran, `CASCADE CONSTRAINTS` no es necesario.

- Por ejemplo, si ninguna restricción referencial de otras tablas
  hace referencia a la columna `PK`, es válido ejecutar
  la siguiente sentencia sin la cláusula `CASCADE CONSTRAINTS`:

```sql
ALTER TABLE tablename DROP (pk_column_name)
```

- **Sin embargo, si las columnas de otras tablas o las columnas que
  quedan en la tabla de destino hacen referencia a las restricciones, se debe especificar `CASCADE CONSTRAINTS`.**

### Visualización de Restricciones

Después de crear una tabla, puede confirmar su existencia emitiendo un comando `DESCRIBE`. La única restricción que se
puede verificar con un comando `DESCRIBE` es la restricción `NOT NULL`.
**La restricción `NOT NULL` aparecce en el diccionario de datos como una restricción `CHECK`**

Para ver todas las restricciones de tabla, consulte la tabla
`USER_CONSTRAINTS`:

~~~sql
SELECT constraint_name, tabe_name, constraint_type, status
FROM USER_CONSTRAINTS
WHERE table_name='COPY_EMPLOYEES';
~~~

|CONSTRAINT_NAME|TABLE_NAME|CONSTRAINT_TYPE|STATUS|
|---|---|---|
|COPY_EMP_PK|COPY_EMPLOYEES|P|ENABLED|
|CDEPT_DEPT_ID_FK|COPY_EMPLOYEES|R|ENABLED|

#### Consulta USER_CONSTRAINTS

Los tipos de restricción que aparecen en el diccionario de datos son:
- P: PRIMARY KEY
- R: REFERENCES (clave ajena)
- C: restricción CHECK (incluída NOT NULL)
- U: UNIQUE