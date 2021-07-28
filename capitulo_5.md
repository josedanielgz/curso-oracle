Oracle Academy: Database Programing with SQL, Parte 5

* [Volver al inicio](index.html)

- [5.1 Funciones de Conversión](#51-funciones-de-conversión)
  - [Conversión de Tipo](#conversión-de-tipo)
  - [De datos de Fecha a Datos de Caracteres](#de-datos-de-fecha-a-datos-de-caracteres)
  - [Conversión de Datos Numéricos en Datos de Caracteres (VARCHAR2)](#conversión-de-datos-numéricos-en-datos-de-caracteres-varchar2)
  - [Conversión de Caracteres en Números](#conversión-de-caracteres-en-números)
  - [Conversión de Caracteres en Fechas](#conversión-de-caracteres-en-fechas)
  - [Reglas del modificador fx](#reglas-del-modificador-fx)
  - [Formato de Fecha RR y Formato de Fecha YY](#formato-de-fecha-rr-y-formato-de-fecha-yy)
- [5.2 Funciones NULL](#52-funciones-null)
  - [Método de Evaluación de las Funciones](#método-de-evaluación-de-las-funciones)
  - [Funciones Relacionadas con los Valores Nulos](#funciones-relacionadas-con-los-valores-nulos)
    - [NVL](#nvl)
    - [NVL2](#nvl2)
    - [NULLIF](#nullif)
    - [COALESCE](#coalesce)
- [5.3 Expresiones Condicionales](#53-expresiones-condicionales)
  - [Expresión CASE](#expresión-case)
  - [Expresión DECODE](#expresión-decode)

## 5.1 Funciones de Conversión

Los RDBMS cuentan con funciones de conversión pensadas para realizar cambios en la forma en que se visualiza la salida de la información. Con ellas se puede por ejemplo, mostrar los números en una moneda local, aplicar formatos a una fecha, mostrar la hora (incluídos los segundos) y hacer un seguimiento del siglo al que hace referencia dicha fecha, entre otros.

Al momento de crear una tabla para una BBDD, se define los tipos de datos que esta almacenará. SQL cuenta con distintos tipos de dato. Estos definen el dominio de valores que puede incluir cada columna. Las funciones de conversión interactúan principalmente con los siguientes tipos de dato: `VARCHAR2`, `CHAR`, `NUMBER` y `DATE`.

`VARCHAR2` se utiliza para datos de caracteres de longitud variable, incluidos los numeros, guiones y caracteres especiales.

`CHAR` se utiliza para datos de texto y de caracteres de longitud fija, incluídos números, guiones y caracteres especiales.

### Conversión de Tipo

Oracle Server puede convertir automáticamente datos `VARCHAR2` y `CHAR` en tipos de dato `NUMBER` Y `DATE`, pero se puede realizar el proceso opuesto para convertirlos de vuelta a cadenas de caracteres. Lo primero se conoce como conversión implícita, y lo segundo como conversión explícita. Lo mejor es establecer de forma explícita las conversiones a utilizar para hacer más flexibles las sentencias SQL.

|FROM|TO|
|---|---|
|VARCHAR2/CHAR|NUMBER|
|VARCHAR2/CHAR|CHAR|
|NUMBER|VARCHAR2|
|DATE|VARCHAR2|

_Descripción breve de las conversiones de dato disponibles en Oracle Server._

### De datos de Fecha a Datos de Caracteres

Generalmente una fecha se convierte de su formato por defecto `DD-Mes-YYYY` en un formato especificado por el usuario. Esto se puede lograr con la ayuda de la función `TO_CHAR`.

~~~sql
TO_CHAR(date|column, 'format-model-you-specify');
~~~

Se puede incluir cualquier elemento de formato de formato de fecha válido, los cuales están enumerados en la siguiente tabla:

|||
|---|---|
|YYYY|Año completo en números|
|YEAR|Año en letra|
|MM|Valor de dos dígitos del mes|
|MONTH|Nombre completo del mes|
|MON|Abreviatura de tres letras del mes|
|DY|Abreviatura de tres letras del día de la semana|
|DAY|Nombre completo del día de la semana|
|DD|Dia numérico del mes|
|DDspth|FOURTHEEN|
|Ddspth|Fourtheen|
|ddspth|fourtheen|
|DDD o DD o D|Dia del año, mes o semana|
|HH24:MI:SS AM|15:45:32 PM|
|DD "of" MONTH|12 de octubre|

Cabe aclarar que:
* Se puede incluir cualquier elemento de formato de fecha válido.
* `sp` sirve para deletrear un número
* `th` hace que el número aparezca como un ordinal (1°, 2°, 3°, etc...)
* `fm` elimina los espacios en blanco o ceros iniciales de la salida.

~~~sql
SELECT TO_CHAR(hire_date, 'fmDay ddth Mon, YYYY') FROM employees;
-- Devuelve 'Martes 7 de jun de 1994'
~~~

~~~sql
SELECT TO_CHAR(hire_date, 'fmDay ddthsp Mon, YYYY') FROM employees;
-- Devuelve 'Martes, siete de junio de 1994'
~~~

~~~sql
SELECT TO_CHAR(hire_date, 'fmDay, ddthsp "of" Month, Year') FROM employees;
-- Devuelve 'Martes, siete de junio, mil novecientos noventa y cuatro'
~~~

~~~sql
SELECT TO_CHAR(SYSDATE,'hh:mm');
-- Devuelve 2:07 AM
~~~

~~~sql
SELECT TO_CHAR(SYSDATE,'hh:mm pm');
-- Devuelve 2:07 AM
~~~

~~~sql
SELECT TO_CHAR(SYSDATE,'hh:mm:ss pm');
-- Devuelve 2:07:23 AM
~~~

### Conversión de Datos Numéricos en Datos de Caracteres (VARCHAR2)

Los números almacenados en la base de datos no tienen formato, con lo que no tienen ingún símbolo/signo de moneda, comas, decimales, etc...
Para agregar formato, en primer lugar se debe convertir el número en un formato de caracteres. 

~~~sql
TO_CHAR(number, 'format-model')
~~~

Algunos de los elementos de formato que se pueden utilizar en
`TO_CHAR` son:

|ELEMENT|DESCRIPTION|EXAMPLE|RESULT|
|---|---|---|---|    
|9|Numeric position (# of 9's determinte width)|999999|1234|    
|0|Display leading zeros|099999|001234|    
|$|Floating dollar sing|$999999|$1234|    
|L|Floating local currency symbol|L999999|FF1234|    
|.|Decimal point in position specified|999999.99|1234.00|    
|,|Comma in position specified|999,999|1,234|    
|MI|Minus signs to right (negative values)|9999999MI|1234-|    
|PR|Parenthesize negative numbers|999999PR|<1234>|    
|EEEE|Scientific notation (must have four EEEE)|99.9999EEEE|1,23E+03|    
|V|Multiply by 10 n times (n = number of 9's after V)|9999V99|9999V9|    
|B|Display zero values at blank, not 0|B9999.99|1234.00|

Por ejemplo

~~~sql
SELECT TO_CHAR(3000,'$999999.99') FROM DUAL;
-- Devuelve 3000,00$
~~~

~~~sql
SELECT TO_CHAR(4500,'99,999') FROM DUAL;
-- Devuelve 4.500
~~~

~~~sql
SELECT TO_CHAR(9000,'99,9999.99') FROM DUAL;
-- Devuelve 9.000,00
~~~

~~~sql
SELECT TO_CHAR(4442,'0,009,999') FROM DUAL;
-- Devuelve 0004422
~~~

### Conversión de Caracteres en Números

Por lo general, si hay caracteres numéricos involucrados, es mejor convertir una cadena de caracteres en un número, porque no se puede realizar cálculos de forma fiable con datos de caracteres. La función necesaria para realizar esta conversión es:

~~~sql
TO_NUMBER(character string,'format model')
~~~

El modelo de formato, mientras que es opcional, evita inconvenientes en caso de que la cadena a transformar contenga
caracteres que no sean números.

Por ejemplo:

~~~sql
SELECT TO_NUMBER('5,320','9,999') AS Number FROM dual;
-- Devuelve 5320
~~~

Hay que prestar atención a la naturaleza de los datos almacenados en una tabla
a la hora de establecer un formato en las funciones de conversión,
especialmente pasando de caracteres a números.

~~~sql
--- Es
SELECT last_name, TO_NUMBER(bonus, '999') AS "Bonus" FROM employees WHERE department_id=80;
-- La siguiente sentencia fallará porque la columna bonus posee números de más
-- de tres cifras.
~~~

### Conversión de Caracteres en Fechas

Para convertir una cadena de caracteres un formato de fecha, utilice:

~~~sql
TO_DATE('character string','format model')
~~~

Esta conversión toma una cadena de caracteres que represente una fecha (como '3 de noviembre de 2001') y la convierte de nuevo en una fecha. Especificando un formato se le indica al servidor de qué manera debe procesar la fecha. Por ejemplo:

~~~sql
TO_DATE('November 3, 2001', 'Month dd, yyyy');
-- Devuelve 03-Nov-2001
~~~

### Reglas del modificador fx

Cuando se convierten caracteres a una fecha, el modificador `fx` (formato exacto) especifica la coincidencia completa que hay entre la cadena de entrada y el formato especificado en el segundo argumento. Este sigue las siguientes reglas:

* La puntuación y el texto entre comillas en el argumento de caracteres deben coincidir con las partes correspondientes del modelo de formato exactamente (excepto mayúsculas/minúsculas).

* No puede tener caracteres blancos adicionales (estos son ignorados en caso de no usar `fx`).

* Los datos numéricos de la cadena de caracteres debe tener el mismo número de dígitos que el elemento correspondiente en el modelo de formato (sin `fx`, los números no pueden omitir los ceros iniciales).

Por ejemplo:

~~~sql
SELECT TO_DATE('May10,1989','fxMonDD,YYYY') FROM DUAL;
-- Retorna 10-May-1989
~~~

~~~sql
SELECT TO_DATE('Sep 07,1965','fxMon dd,YYYY') FROM DUAL;
-- Retorna 07-Sep-1965
~~~

~~~sql
SELECT TO_DATE('July312004','fxMonthDDYYYY') FROM DUAL;
-- Retorna 31-Jul-2004
~~~

~~~sql
SELECT TO_DATE('June 19, 1990','fxMonth dd, YYYY') FROM DUAL;
-- Retorna 19-Jun-1990
~~~

### Formato de Fecha RR y Formato de Fecha YY

Existe la posibilidad de que algunas bases de datos antiguas aún utilicen la notación de dos dígitos (YY) para referirse a los años en las fechas, en contraste con el estándar moderno que utiliza cuatro (YYYY). Esto puede generar confusiones a la hora de lidiar con fechas como `02-Jan-98`. Aunque originalmente esta se interpretaba como 2 de enero de 1998, ahora es posible la desambigüación de interpretarla como 2 de enero de 2098.

Para alivianar estos problemas de incongruencia, Oracle tiene una forma de interpretar estas fechas con el siglo correcto, mediante el símbolo `RR`.

~~~sql
SELECT TO_DATE('27-Oct-95','DD-Mon-RR') FROM DUAL;
-- Devuelve 27-Oct-1995
~~~

Intentar realizar esta consulta con la sintaxis más moderna podría resultar en un problema:

~~~sql
SELECT TO_DATE('27-Oct-95','DD-Mon-YY') FROM DUAL;
-- Devuelve 27-Oct-2095
~~~

Debido a que se asume que un formato con `YY` se está usando en una fecha en el siglo actual del servidor.

RR tiene algunas reglas sencillas:

**Suponiendo que EL AÑO ACTUAL DEL SERVIDOR SE ENCUENTRE ENTRE 00-49:**
* Si el año de la cadena está entre 00-49, la fecha de devolución se calcula como si estuviera en **el siglo actual**.

* Si **el año de la cadena se encuentra entre 50-99** se asume que **es una fecha del siglo pasado (en este caso, el siglo XX)**.

**Suponiendo que EL AÑO ACTUAL DEL SERVIDOR SE ENCUENTRE ENTRE 50-99:**

* Si **el año de la cadena está entre 00-49**, la fecha de devolución se calcula como si estuviera en **el siglo siguiente**.

* Si **el año de la cadena se encuentra entre 50-99** se asume que es una fecha del **siglo actual (en este caso, el siglo XXI)**.

Algunos ejemplos prácticos:

|Año actual|Fecha especificada|Formato RR|Formato YY|    
|---|---|---|---|    
|1995|27-Oct-95|1995|1995|    
|1995|27-Oct-17|2017|1917|    
|2015|27-Oct-17|2017|2017|    
|2015|27-Oct-95|1995|2095|    

## 5.2 Funciones NULL

Debido al significado especial que tienen los nulos en una base de datos (los cuales se habían mencionado antes con la existencia de palabras reservadas como `IS [NOT] NULL`, `NULL FIRST` y `NULL LAST`) existen medidas especiales  tanto en el propio estándar SQL como implementadas por Oracle Server, para el tratamiento de dichos valores nulos. Puede que un valor nulo no signifique "nada", pero puede afectar la forma en que se evalúe una expresión. ¿Cómo se calcula una medida y dónde aparece un valor en una lista ordenada?.

### Método de Evaluación de las Funciones

Las funciones pueden anidarse un número indefinido de veces (incluir una dentro de otra, de forma que el resultado de la función más interna sea usado en la función externa que la contiene). Siempre se empieza a evaluar desde la función más profunda hasta la más externa. Por ejemplo:

~~~sql
SELECT TO_CHAR(NEXT_DAY(ADD_MONTHS(hire_date, 6)),'FRIDAY'), 'fmDay, Month ddth, YYYY') AS "Next Evaluation" FROM employees WHERE employee_id=100;
-- Devuelve Friday, December 18th, 1987
~~~

En el ejemplo anterior:

* Se le agregan 6 meses a la fecha de contratación.
* Se identifica el viernes más cercano que le siga al día siguiente al devuelto en el primer paso.
* A la fecha devuelta en el paso anterior se le aplicará formato para que se muestre en un formato similar a "Viernes, 18 de diciembre de 1987", y aparecerá en la salida bajo el nombre de "Next Evaluation".

### Funciones Relacionadas con los Valores Nulos

Imagine la siguiente pregunta: ¿Es cierto que X=Y?. No puede responder esa pregunta hasta que sepa los valores de X y Y, pero de momento esos valores no los conoce, por lo tanto están indeterminados para esa expresión. Ese sería un ejemplo de una operación que la BBDD tendría que realizar donde los nulos pueden jugar un inconveniente. Oracle Server cuenta con cuatro funciones para manejar este tipo de situaciones:

#### NVL

Convierte un valor nulo en un valor conocido de un tipo de dato fijo (fecha, caractér o numérico).
Si se usa una columna, el tipo de dato que va a sustituir al valor nulo debe coincidir con esta. Su sintaxis es:

~~~sql
NVL (valor que podría ser nulo, valor que reemplaza un valor nulo);
~~~

Por ejemplo:

~~~sql
SELECT country_name, NVL(internet_extension,'None') AS "Internet extn" FROM wf_countries WHERE location='Southern Africa' ORDER BY internet_extension DESC;
~~~
 La sentencia anterior da como resultado:

|COUNTRY_NAME|Internet extn|
|---|---|  
|Juan de Nova Island|None|  
|Europa Island|None|  
|Republic of Zimbabwe|.zw|  
|Republic of Zambia|.zm|  
|Republic of South Africa|.za|

Una de las tareas más importantes en las que esta función se utiliza es en la de realizar cálculos aritméticos. **Dado que  una operación con valores nulos siempre da un resultado nulo, siempre que conviene CAMBIAR EL VALOR NULO A UN MÓDULO (0 O 1) ANTES DE OPERAR.**

Por ejemplo:

~~~sql
SELECT last_name, NVL(comission_pct,0)*250 AS "Comission" FROM employees WHERE department_id IN(80,90);
~~~

Esto devuelve como resultado:

|LAST_NAME|Commission|  
|---|---|  
|Zlotkey|50|  
|Abel|70|  
|Taylor|50|  
|King|0|  
|Kochhar|0|  
|De Haan|0|  

#### NVL2

Esta función evalúa tres valores, chequea si el primero es nulo, si lo es, utiliza el tercer valor para sustituirlo, si no lo es utiliza el segundo. El valor de la expresión 1 puede ser cualquier tipo de dato, las otras dos pueden ser de cualquier tipo, mientras que no sea un valor `LONG`. Esta función siempre devuelve un tipo de dato igual al del valor 2 (implícitamente esto hace que las expresiones 2 y 3 deban ser del mismo tipo) a menos que sean caracteres (en cuyo caso siempre devuelve `VARCHAR2`). 

La sintaxis es la siguiente:

~~~sql
NVL2(si esta expresion contiene un nulo, devuelve esto si NO es nulo, devuelve esto si ES NULO)
~~~

Por ejemplo:

~~~sql
SELECT last_name, salary, NVL2 (commission_pct,salary+(salary*comission_pct),salary) AS "income" FROM employees WHERE department_id IN(80,90);
~~~

Esa sentencia retorna:

|LAST_NAME|SALARY|INCOME|   
|---|---|---|   
|Zlotkey|10500|12600|   
|Abel|11000|143000|   
|Taylor|8600|10320|   
|King|24000|24000|   
|Kochhar|17000|17000|   
|De Haan|17000|17000|   

#### NULLIF

Esta función compara dos expresiones, y si son iguales, devuelve un valor nulo; en caso contrario, devuelve lo que haya en la primera expresión. Se usa así:

~~~sql
NULLIF(expresion1,expresion2);
~~~


Por ejemplo:

~~~sql
SELECT first_name, LENGTH(first_name) AS "Length FN", last_name, LENGTH(last_name) AS "Length LN", NULLIF(LENGTH(first_name),LENGTH(last_name)) AS "Compare Them" FROM employees;
~~~

La sentencia anterior devuelve:

|FIRST_NAME|Lenght FN|last_name|Length LN|Compare Them|     
|---|---|---|---|---|     
|Ellen|5|Abel|4|5|     
|Curtis|6|Davies|6|-|     
|Lex|3|De Haan|7|3|     

Es decir, esta sentencia compara la longitud del nombre y apellido de los empleados, si ambos son iguales retorna `NULL` (como en el caso de Curtis Davies) si no, retorna la longitud del nombre.

#### COALESCE

Esta función funciona como una extensión de la función `NVL`, pero puede contener más valores. Esta función "une" una lista de expresiones tratando de encontrar la primera de ellas que no sea nula, es en ese punto cuando se detiene.

~~~sql
COALESCE (expresion_1,expresion_2...,expresion_n);
~~~

Por ejemplo, la siguiente sentencia:

~~~sql
SELECT last_name, COALESCE(commission_pct,salary,10) AS "Comm" FROM employees ORDER BY commission_pct;
~~~

Devuelve:

|LAST_NAME|Comm|
|---|---| 
|Grant|.15| 
|Zlotkey|.2| 
|Taylor|.2| 
|Abel|.3| 
|Higgins|12000| 
|Gietz|8300| 

* Si un empleado tiene un valor no `NULL` para `commission_pct`, se devuelve; caso contrario, si el salario tiene algún valor, se devuelve ese salario.
  
* En caso de que un empleado no tenga ni `commission_pct` ni salario, se devuelve el número 10.

## 5.3 Expresiones Condicionales

La toma de decisiones es fundamental a la hora de modelar datos, se debe decidir qué funciones de negocio se deben modelar y cuáles no. Es necesario que los diseñadores analicen la información para identificar las entidades, resolver las relaciones y seleccionar los atributos, entre otros.

Este proceso de toma de decisiones no es muy distinto en programación, donde se pueden plantear requerimientos de la forma `IF-THEN-ELSE` (si algo pasa, reaccionamos de una forma, si no reaccionamos de otra). En SQL esto se conoce como métodos de procesamiento condicionales.

Las dos expresiones condicionales son `CASE` y `DECODE`. La expresión `CASE` en particular tiene similitud con la expresión `NULLIF` (que recapitulando, compara dos expresiones y retorna `NULL` si ambas son iguales, o el primer argumento si no lo son).

**NOTA:** Hay dos juegos de comandos o sintaxis que se pueden utilizar para escribir sentencias SQL, las que son compatibles con el estándar ANSI/ISO SQL y las que son propias de Oracle Server. Se intenta que ambas sean muy similares, pero son diferentes. `CASE` y `DECODE` son ejemplos de esto: la primera es propia del estándar ANSI/ISO, y la segunda es exclusiva de Oracle. Ambas devuelven lo mismo, pero su sintaxis es distinta.

### Expresión CASE

`CASE` básicamente realiza la función de una sentencia `IF-THEN_ELSE`, su sintaxis es la siguiente:

~~~sql
CASE expr WHEN comparacion_expr1 THEN return_expr1 [
          WHEN comparacion_expr2 THEN return_expr2 ...
          WHEN comparacion_exprn THEN return_exprn ]
          ELSE else_expr
END
~~~

Por ejemplo, la siguiente sentencia:

~~~sql
SELECT last_name,
CASE department_id
  WHEN 90 THEN 'Management'
  WHEN 80 THEN 'Sales'
  WHEN 60 THEN 'IT'
  ELSE 'Other dept.'
END AS "Department"
FROM employees;
~~~

Retorna:

|LAST_NAME|Department|
|---|---| 
|King|Management|
|Kochhar|Management|
|De Haan|Management|
|Whalen|Other dept.|
|Higgins|Other dept.|
|Gietz|Other dept.|
|Zlotkey|Sales|
|Abel|Sales|
|Taylor|Sales|
|Grant|Other dept.|
|Mourgos|Other dept.|
|Rajs|Other dept.|
|Davies|Other dept.|
|Matos|Other dept.|
|Vargas|Other dept.|
|Hunold|IT|
|Ernst|IT|
|Lorentz|IT|
|Hartstein|Other dept.|
|Fay|Other dept.|

La sentecia es autoexplicativa, devuelve una cadena de caracteres diferente dependiendo del valor del `department_id` (si es 90, 80 o 60, o si no es ninguno de esos tres).

### Expresión DECODE
La función `DECODE` evalúa una expresión de forma similar a como lo hase `CASE`.

~~~sql
DECODE(column1|expresion, search_1, result_1)
  [, search_2, result_2, ...]
  [, search_n, result_n, ...]
  [, default]
~~~

Las diferencias aunque sutiles comienzan a apreciarse. `DECODE` puede incluir un valor `DEFAULT` que sería el equivalente al `ELSE` de un `CASE`, con la diferencia de que **`DECODE` retorna un valor nulo en caso de que no se haya establecido un `DEFAULT`**.

Por ejemplo:

~~~sql
SELECT last_name,
DECODE(department_id,
  90,'Management',
  80,'Sales',
  60,'IT',
  'Other dept.')
AS "Department" FROM employees;
~~~

La sentencia anterior retorna:

|LAST_NAME|Department|
|---|---| 
|King|Management|
|Kochhar|Management|
|De Haan|Management|
|Whalen|Other dept.|
|Higgins|Other dept.|
|Gietz|Other dept.|
|Zlotkey|Sales|
|Abel|Sales|
|Taylor|Sales|
|Grant|Other dept.|
|Mourgos|Other dept.|
|Rajs|Other dept.|
|Davies|Other dept.|
|Matos|Other dept.|
|Vargas|Other dept.|
|Hunold|IT|
|Ernst|IT|
|Lorentz|IT|
|Hartstein|Other dept.|
|Fay|Other dept.|

Esta consulta devuelve exactamente lo mismo que su contraparte escrita usando `CASE`, pero su sintaxis es diferente.