Parte 11

* [Volver al inicio](index.html)

## 11.1 Garantía de Reultados de Consultas de Calidad

Conocer las reglas de sintaxis para generar una consulta SQL
no le garantiza que pueda elaborar una que produzca los datos
que desea sin más. Lo mejor es visualizar la salida que se desea, y averiguar
las consulta necesaria para generar dicha salida, de esa forma
gana confianza en que los resultados de la consultas son los
esperados.

### Escriba la consulta

Cree una lista de todas las tablas cuyos dos primeros caracteres en el nombre
de la tabla sean JO. Las tablas deben ser propiedad del usuario de Oracle
actual. La tabla utilizada es `User_tables`. La tabla de salida es la
siguiente:

|TABLE_NAME|
|---|
|JOBS|
|JOB_GRADES|
|JOB_HISTORY|

Cree una lista que incluya la inicial del nombre de cada empleado, un espacio y el apellido del empleado. Utilice la
tabla `employees`. La salida es:

|Employee Names|
|---|
|E Abel|
|C Davies|
|L De Haan|
|B Ernst|
|P Fay|
|W Gietz|
|K Grant|
|M Hartstein|
|S Higgins|
|A Hunold|
|...|

Cree una lista de los nombres de todos los empleados concatenados a un espacio y el apellido del empleado y el
correo electrónico de todos los empleados donde la dirección
de correo electrónico contenga la cadena `IN`. Utilice la
tabla `employees`.

|Employee Name|Email|
|---|---|
|Shelley Higgins|SHIGGINS|
|Steven King|SKING|

Cree una lista del apellido "más pequeño" y el apellido "más grande" de la tabla `employees`.

|First Last Name|Last Last Name|
|---|---|
|Abel|Zlotkey|

Cree una lista de salarios semanales a partir de la tabla `employees` donde el salario semanal esté entre 700 y 3000.
Los salarios deben estar formateados para incluir un signo $ y tener dos decimales como `$99999,99`. Utilice `employees`

|Weekly Salary|
|---|
|$1015.38|
|$2769.23|
|$1915.38|
|$2423.08|
|$2538.46|
|$1984.62|
|$1615.38|
|$1338.46|
|$807.69|
|...|

Cree una lista de todos los empleados y su cargo relacionado ordenada por
`job_title`. Utilice a `employees`.

|Emplyoee Name|Job|
|---|---|
|S Higgins|Accounting Manager|
|J Whalen|Administration Assistan|
|L De Haan|Administration Vice President|
|N Kochhar|Administration Vice President|
|M Hartstein|Marketing Manager|
|P Fay|Marketing Representative| 
|S King|President|
|A Hunold|Programmer|
|B Ernst|Programmer|
|D Lorentz|Programmer|
|W Gietz|Public Accountant|
|E Zlotkey|Sales Manager|
|J Taylor|Sales Representative|
|K Grant|Sales Representative|
|E Abel|Sales Representative|
|T Rajs|Stock Clerk|
|C Davies|Stock Clerk|
|P Vargas|Stock Clerk|
|R Matos|Stock Clerk|
|K Mourgos|Stock Manager|

Cree una lista de cargos de todos los empleados, rangos de salario del cargo y
salario del empleado. Enumere el rango de salarios menor y mayor de cada rango
con un guión para separar los salarios de la siguiente forma: 100-200. Utilice
las tablas `employees` y `jobs`.

|Emplyoee Name|Job|Salary Range|Employee's Salary|||
|---|---|---|---|||
|S Higgins|Accounting Manager|8200 - 16000|12000|
|J Whalen|Administration Assistan|3000 - 6000|4400|
|L De Haan|Administration Vice President|15000 - 30000|17000|
|N Kochhar|Administration Vice President|15000 - 30000|17000|
|M Hartstein|Marketing Manager|4000|9000|
|P Fay|Marketing Representative|4000 - 10000|9000|
|S King|President|20000 - 40000|24000|
|A Hunold|Programmer|4000 - 10000|9000|
|B Ernst|Programmer|4000 - 10000|6000|
|D Lorentz|Programmer|4000 - 10000|4200|
|...|...|...|...|

Mediante un método de unión ANSI, cree una lista de iniciales y apellidos de
todos los empleados y nombres de departamento, asegrúrese de que las tablas se
usen en todas claves ajenas declaradas entre las dos tablas. Utilice las tablas
`employees` y `departments`

|Employee Name|Department Name|
|---|---|
|N Kochhar|Executive|
|L De Haan|Executive|
|W Gietz|Accounting|
|E Abel|Sales|
|J Taylor|Sales|
|T Rajs|Shipping|
|C Davies|Shipping|
|R Matos|Shipping|
|P Vargas|Shipping|
|B Ernst|Programmer|
|D Lorentz|Programmer|
|P Fay|Marketing|

Cambie la consulta anterior  a unión solo en la colummna `department_id`

|Employee Name|Department Name|
|---|---|
|J Whalen|Administration|
|M Hartstein|Marketing|
|P Fay|Marketing|
|C Davies|Shipping|
|P Vargas|Shipping|
|T Rajs|Shipping|
|K Mourgos|Shipping|
|R Matos|Shipping|
|...|...|
|J Taylor|Sales|
|L Zlotkey|Sales|
|E Abel|Sales|
|L De Haan|Executive|
|S King|Executive|
|N Kochhar|Executive|
|S Higgins|Accounting|
|W Gietz|Accounting|
|...|...|

Cree una lista de apellidos de todos los empleados y la palabra `nobody` o
`somebody` en función de si el empleado tiene un jefe.  Utilice la función
`DECODE` de Oracle para crearla. Utilice la tabla `employees`.

|Works For|Last Name|
|---|---|
|nobody|King|
|somebody|Kochhar|
|somebody|De Haan|
|somebody|Whalen|
|somebody|Higgins|
|...|...|

Cree una lista de iniciales y apellidos de todos los empleados, salarios y un
sío no para mostrar si un empleado tiene una comisión. Básese en
este resultado y corríjalo:

~~~sql
SELECT SUBST(first_name,1,1)||' '||last_name,"Employee Name",salary "Salary", DEC(comission_pct NULL, 'No','Yes') 'Commission' FROM employees;
~~~

|Employee Name|Salary|Commission|
|---|---|---|
|S King|24000|No|
|N Kochhar|17000|No|
|L De Haan|17000|No|
|J Whalen|4400|No|
|S Higgins|12000|No|
|W Gietz|8300|No|
|E Zlotkey|10500|Yes|
|...|...|...|

Cree una lista de apellidos de todos los empleados, nombres de
departamento, ciudades y estados/provincias. Incluya los departamentos sin empleados. Es necesaria una unión externa. Utilice las tablas `employees`, `departments`, `locations`.

|LAST_NAME|DEPARTMENT_NAME|CITY|STATE_PROVINCE|
|---|---|---|---|
|Abel|Sales|Oxford|Oxford|
|Davies|Shipping|South San Francisco|California|
|De Haan|Executive|Seattle|Washington|
|Ernst|IT|Southlake|Texas|
|Fay|Marketing|Toronto|Ontario|
|...|...|...|...|
|-|Contracting|Seattle|Washington|

Cree una lista de nombres y apellidos de todos los empleados y
la primera incidenca de `commission_pct`, `manager_id` o -1.
Si un empleado tiene una comisión, muestre la columna `commission_pc`, si no
muestre su `manager_id`, si no tiene niguna de las dos, muestre -1. Utilice la tabla `employees`

|First Name|Last Name|Which function???|
|---|---|---|
|Steven|King|-1|
|Neena|Kochhar|100|
|Lex|De Haan|100|
|Jenniger|Whalen|101|
|Shelley|Higgins|101|
|William|Gietz|205|
|Eleni|Zlotkey|.2|
|Ellen|Abel|.3|
|...|...|...|

Cree una lista de apellidos de todos los empleados, salarios y `job_grade` para todos los empleados que trabajan en departamentos con `department_id` superior a 50. Utilice `employees` y `job_grades`.

|LAST_NAME|SALARY|GRADE_LEVEL|
|---|---|---|
|Lorentz|4200|B|
|Ernst|6000|C|
|Gietz|8300|C|
|Taylor|8600|C|
|Hunold|9000|C|
|Zlotkey|10500|C|
|Abel|11000|D|
|Higgins|12000|D|
|Kochhar|17000|E|
|De Haan|17000|E|
|King|24000|E|

Elabore una lista del apellido de todos los empleados y el nombre
de departamento. Incluya los empleados sin departamento
Y los departamentos sin empleados. Utilice las tablas `employees` y `departments`.

|LAST_NAME|GRADE_LEVEL|
|---|---|---|
|Whalen|Administration|
|Fay|Marketing|
|Hartstein|Marketing|
|Mourgos|Shipping|
|Vargas|Shipping|
|Davies|Shipping|
|Matos|Shipping|
|Rajs|Shipping|
|...|...|
|Zlotkey|Sales|
|Abel|Sales|
|De Haan|Executive|
|Kochhar|Executive|
|King|Executive|
|Gietz|Accounting|
|Higgins|Accounting|
|Grant|-|
|-|Contracting|

Cree una lista de recorrido de árboles de apellidos de todos los
empleados, apellido de su jefe y posición en la compañía.
El jefe de nivel superior tiene la posición 1, los subordinados
de este jefe tienen la posición 2, sus subordinados tienen la posición 3, y así
sucesivamente. Inicie la lista con el número de empleado 100. Utilice la tabla `employees`.

|POSITION|LAST_NAME|MANAGER_NAME|
|---|---|---|---|
|1|King|-|
|2|Kochhar|King|
|3|Whalen|Kochhar|
|3|Higgins|Kochhar|
|4|Gietz|Higgins|
|2|De Haan|King|
|3|A Hunold|De Haan|
|4|B Ernst|Hunold|
|4|D Lorentz|Hunold|
|2|Mourgos|King|
|3|Rajs|Mourgos|
|3|Davies|Mourgos|
|3|Matos|Mourgos|
|3|Vargas|Mourgos|
|2|Zlotkey|King|
|3|Abel|Zlotkey|
|3|J Taylor|Zlotkey|
|3|Grant|Zlotkey|
|2|Hartstein|King|
|3|Fay|Hartstein|

Cree una lista de la primera y última fecha de contratación, y el
número de empleados de la tabla `employees`.

|Lowest|Highest|No of Employees|
|---|---|---|
|17-Jun-1987|29-Jan-2000|20|

Cree una lista de nombres de departamento y los costos de departamento (sumados
los salarios). Incluya sólo los departamentos cuyos costos de salario sean de
entre 15000 y 31000, y ordene la lista por costo. Utilize las tablas `employees` y `departments`.

|DEPARTMENT_NAME|SALARIES|
|---|---|
|Shipping|17500|
|Marketing|19000|
|IT|19200|
|Accounting|20300|
|Sales|30100|

Crea una lista de nombres de departamaneto, el ID de jefe, el nombre del jefe (apellido del empleado) de dicho departamento y
el salario medio de cada departamento. Utilice las tablas `employees` y `departments`.

|DEPARTMENT_NAME|MANAGER_ID|MANAGER_NAME|AVG_DEPT_SALARY|
|---|---|---|---|
|Shipping|124|Mourgos|3500|
|Administration|200|Whalen|440|
|IT|103|Hunold|6400|
|Marketing|201|Hartstein|9500|
|Sales|140|Zlotkey|10033|
|Accounting|205|Higgins|10150|
|Executives|100|King|19333|

Muestre el mayor salario medio para los departamentos en la tabla
`employees`. Redondee el resultado al número entero más cercano.

|Highest Avg Sal for Depts|
|---|
|19333|

Cree una lista de nombres de departamento y sus costos mensuales (sumados los
salarios)

|Department Name|Monthly Cost|
|---|---|
|Administration|4400|
|Accounting|20300|
|IT|103|19200|
|Executives|58000|
|Shipping|140|17500|
|Sales|30100|
|Marketing|19000|

Cree una lista de nombres de departamento y `jobs_id`. Calcule el
costo de salarios mensuales para cada `job_id` dentro de un departamento y para
todos los departamentos sumados juntos. Utilice las tablas
`employees` y `departments`.

|Department Name|Job Title|Monthly Cost|
|---|---|---|
|Accounting|AC_ACCOUNT|8300|
|Accounting|AC_MGR|12000|
|Accounting|-|20300|
|Administration|AD_ASST|440|
|Administration|-|4400|
|Executive|AD_PRES|24000|
|Executive|AD_VP|34000|
|Executive|-|58000|
|IT|IT_PROG|19200|
|IT|-|19200|
|Marketing|MK_MAN|13000|
|Marketing|MK_REP|6000|
|Marketing|-|19000|
|Sales|SA_MAN|10500|
|Sales|SA_REP|19600|
|Sales|-|30100|
|Shipping|ST_CLERK|11700|
|Shipping|ST_MAN|5800|
|Shipping|-|17500|
|-|-|168500|

Cree una lista de nombres de departamento y `jobs_id`. Calcule
el costo de salarios mensuales para cada `job_id` dentro de un
departamento para cada departamento, para cada grupo de `jobs_id` independientemente del departamento y para todos los
departamentos sumados juntos (indicación: un cubo). Utilice
las tablas `employees` y `departments`.

|Department Name|Job Title|Monthly Cost|
|---|---|---|
|Accounting|AC_ACCOUNT|8300|
|Accounting|AC_MGR|12000|
|Accounting|-|20300|
|Administration|AD_ASST|440|
|Administration|-|4400|
|Executive|AD_PRES|24000|
|Executive|AD_VP|34000|
|Executive|-|58000|
|IT|IT_PROG|19200|
|IT|-|19200|
|Marketing|MK_MAN|13000|
|Marketing|MK_REP|6000|
|Marketing|-|19000|
|Sales|SA_MAN|10500|
|Sales|SA_REP|19600|
|Sales|-|30100|
|Shipping|ST_CLERK|11700|
|Shipping|ST_MAN|5800|
|Shipping|-|17500|
|-|AC_ACCOUNT|8300|
|-|AC_MGR|12000|
|-|AD_ASST|440|
|-|AD_PRES|24000|
|-|AD_VP|34000|
|-|IT_PROG|19200|
|-|MK_MAN|13000|
|-|MK_REP|6000|
|-|SA_MAN|10500|
|-|SA_REP|19600|
|-|ST_CLERK|11700|
|-|ST_MAN|5800|
|-|-|168500|

Amplíe la lista anterir para mostrar también el valor de department_id o job_id que se ha utilizado para crear los
subtotales mostrados en la salida (indicación: cubo, agrupación).
Utilice las tablas `employees`, `departments`.

|Department Name|Job Title|Monthly Cost|Department ID Used|
|---|---|---|---|---|
|Accounting|AC_ACCOUNT|8300|Yes|Yes|
|Accounting|AC_MGR|12000|Yes|Yes|
|Accounting|-|20300|Yes|No|
|Administration|AD_ASST|440|Yes|Yes|
|Administration|-|4400|Yes|No|
|Executive|AD_PRES|24000|Yes|Yes|
|Executive|AD_VP|34000|Yes|Yes|
|Executive|-|58000|Yes|No|
|IT|IT_PROG|19200|Yes|Yes|
|IT|-|19200|Yes|No|
|Marketing|MK_MAN|13000|Yes|Yes|
|Marketing|MK_REP|6000|Yes|Yes|
|Marketing|-|19000|Yes|No|
|Sales|SA_MAN|10500|Yes|Yes|
|Sales|SA_REP|19600|Yes|Yes|
|Sales|-|30100|Yes|No|
|Shipping|ST_CLERK|11700|Yes|Yes|
|Shipping|ST_MAN|5800|Yes|Yes|
|Shipping|-|17500|Yes|No|
|-|AC_ACCOUNT|8300|No|Yes|
|-|AC_MGR|12000|No||Yes
|-|AD_ASST|440|No|Yes|
|-|AD_PRES|24000|No|Yes|
|-|AD_VP|34000|No|Yes|
|-|IT_PROG|19200|No|Yes|
|-|MK_MAN|13000|No|Yes|
|-|MK_REP|6000|No|Yes|
|-|SA_MAN|10500|No|Yes|
|-|SA_REP|19600|No|Yes|
|-|ST_CLERK|11700|No|Yes|
|-|ST_MAN|5800|No|Yes|
|-|-|168500|No|No|

Cree una lista que incluya los costos de salarios mensuales de cada cargo dentro de un departamento. En la misma lista, muestre
el costo de salarios mensuales por ciudad (indicación: juegos de agrupamiento). Utilice las tablas `employees`, `departments` y `locations`.

|DEPARTMENT_NAME|JOB_ID|CITY|SUM(SALARY)|
|---|---|---|---|
|Accounting|AC_MGR|-|12000|
|Accounting|AC_ACCOUNT|-|8300|
|Adminstration|AD_ASST|-|4400|
|Executive|AD_VP|-|34000|
|Executive|AD_PRES|-|24000|
|IT|IT_PROG|-|19200|
|Marketing|MK_REP|-|6000|
|Marketing|MK_MAN|-|13000|
|Sales|SA_REP|-|19600|
|Sales|SA_MAN|-|10500|
|Shipping|ST_MAN|-|5800|
|Shipping|ST_CLERK|-|11700|
|-|-|Oxford|30100|
|-|-|Seattle|82700|
|-|-|South San Francisco|17500|
|-|-|Southlake|19200|
|-|-|Toronto|19000|

Cree una lista de nombres de empleados como se muestran e ID de departamento. En el mismo informe, mistre los IDS y nombres de
departamento. Por último, enumere las ciudades. Las filas no deben estar unidas, solo aparecer en el mismo informe (indicación: unión). Utilice las tablas `employees`, `departments` y `locations`.

|A Hunold|60|-|-|
|B Ernst|60|-|-|
|C Davies|50|-|-|
|D Lorentz|60|-|-|
|E Abel|80|-|-|
|E Zlotkey|80|-|-|
|J Taylor|80|-|-|
|J Whalen|10|-|-|
|K Grant|-|-|-|
|K Mourgos|50|-|-|
|L De Haan|90|-|-|
|M Hartstein|20|-|-|
|N Kochhar|90|-|-|
|P Fay|20|-|-|
|P Vargas|50|-|-|
|R Matos|50|-|-|
|S Higgins|110|-|-|
|S King|90|-|-|
|T Rajs|50|-|-|
|W Gietz|110|-|-|
|-|10|Administration|-|
|-|20|Marketing|-|
|-|50|Shipping|-|
|-|60|IT|-|
|-|80|Sales|-|
|-|90|Executives|-|
|-|110|Accounting|-|
|-|190|Contracting|-|
|-|-|-|Oxford|
|-|-|-|Seattle|
|-|-|-|South San Francisco|
|-|-|-|Southlake|
|-|-|-|Toronto

Cree una lista de inicial y apellido de cada empleado, salario
y nombre de departamento de cada empleado que gane más de la medida de su departamento. Utilice las tablas `departments`, `employees`.

|Employee|Salary|Department Name|
|---|---|---|
|M Hartstein|13000|Marketing|
|K Mourgos|5800|Shipping|
|A Hunold|9000|IT|
|E Zlotkey|10500|Sales|
|E Abel|11000|Sales|
|S King|24000|Executives|
|S Higgins|12000|Accounting|



