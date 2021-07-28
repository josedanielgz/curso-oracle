- [Volver al inicio](index.html)

## 20.1 Garantía de Resultados de Consultas de Calidad - Técnicas Avanzadas

Además de conocer las reglas de sintáxis para generar una consulta SQL, también
hay que verificar que produzcan los datos deseados. Los siguientes ejercicios
prácticos de observar la salida y averiguar una consulta para generar dicha
salida le ayudará a ganar confianza en los resultados de sus propias consultas.

Utilizaremos las siguientes tablas:

~~~sql
CREATE TABLE emp AS select * from employees;
CREATE TABLE dept AS select * from departments;
~~~

* Cree un informe que muestre el nombre de restricción, el tipo, el nombre de columna y la posición de columna de todas las restricciones de la tabla `JOB_HISTORY`, además de las restricciones NO NULAS
  
Utilice las tablas `user_constraints`, `user_cons_columns`

|CONSTRAINT_NAME|CONSTRAINT_TYPE|COLUMN_NAME|POSITION|
|---|---|---|---|
|JHIST_EMP_ID_ST_DATE_PK|P|EMPLOYEE_ID|1|
|JHIST_EMP_ID_ST_DATE_PK|P|START_DATE|2|
|JHIST_JOB_FK|R|JOB_ID|1|
|JHIST_EMP_FK|R|EMPLOYEE_ID|1|
|JHIST_DEPT_FK|R|DEPARTMENT_ID|1|

* Cree una restricción de clave primaria en la columna `employee_id` de la tabla `emp`.

* Cree una restricción de clave primaria en la columna``department_id` de la tabla `dept`.

* Agregue una restricción ajena entre `DEPT` y `EMP` para que en la tabla `EMP` sólo puedan introducirse departamentos válidos, pero asegúrese de que se puede suprimir cualquier fila de la tabla `DEPT`

~~~sql
ALTER TABLE emp CREATE CONSTRAINT FOREIGN KEY (dept_id) REFERENCES dept(deptid) ON DELETE CASCADE;
~~~

* Pruebe la restricción de clave ajena que acaba de crear siguiendo los ejemplos de esta diapositiva:

* Examine el número de filas de la tabla `emp`

~~~sql
SELECT COUNT(*) AS "Num emps"
FROM emp;
-- esto debe devolver 20
~~~

* Elimine los detalles del departamento 10 de la tabla `dept`

~~~sql
DELETE * FROM dept WHERE department_id = 10;
~~~

* Ahora compruebe de nuevo el número de empleados, debe haber menos

~~~sql
SELECT COUNT(*) AS "Num emps"
FROM emp;
-- esto debe devolver 19
~~~

* Genere un informe que devuelva el apellido, el salario, el número de departamento y el salario medio de todos los empleados en los que el salario es mayor que el salario medio. Utilice las tablas `employees` y `departments`

|LAST_NAME|SALARY|DEPARTMENT_ID|SALAVG|
|---|---|---|---|
|Hartstein|13000|20|9500|
|Mourgos|5800|50|3500|
|Hunold|9000|60|6400|
|Zlotkey|10500|80|10033|
|Abel|11000|80|10033|
|King|24000|90|19333|
|Higgins|12000|110|10150|

* Cree uan vista denominada v2 que devuelve el salario más alto, el salario más bajo, el salario medio y el nombre de cada departamento. Utilice las tablas `emp` y `dept`.

|Departament|Lowest Salary|Highest Salary|Average Salary|
|---|---|---|---|
|Accounting|8300|1200|10150|
|IT|4200|9000|6400|
|Executive|17000|24000|19333|
|Shipping|2500|5800|3500|
|Sales|8600|11000|10033|
|Marketing|6000|13000|9500|

* Cree una vista denominada `Dept_Managers_view` que devuelva una
lista de nombres de departamento junto con las iniciales y el apellido del jefe para dicho departamento. **Pruebe la vista devolviendo todas sos filas y asegúrese de que no se pueda actualizar ninguna fila a través de la vista.** Utilice las tablas `employees` y `departments`.

|DEPT_NAME|MGR_NAME|
|---|---|
|Executive|S.King|
|IT|A.Hunold|
|Shipping|K.Mourgos|
|Administration|J.Whalen|
|Marketing|M.Hartstein|
|Accountinging|S.Higgins|

* Cree una sentencia denominada `ct_seq` con todos los valores por defecto. Para eso, complete la siguiente sentencia y corrija el error:

~~~sql
-- corrija esta sentencia
CREATE SEQUENCE ct_seq;

SELECT ct_seq.currval FROM dual;
-- esta sentencia da error
~~~

* Basándose en la sentencia anterior, observe la siguiente sentencia `INSERT` y corrija el error:

~~~sql
-- corriga esta sentencia
INSERT emp
(
  employee_id, first_name, last_name, email, phone_number, hire_date, 
  job_id, salary, commission_pct, manager_id, department_id
)
VALUES
(
  currval.ct_seq, 'Kaare', 'Hansen', 'KHANSEN', '4496583212',
  sysdate, 'Manager', 6500, null, 100, 10
);
~~~

* Corrija el errror en la sentencia SQL para crear el índice como se muestra en la siguiente tabla:

~~~sql
-- corrija la siguiente sentencia
CREATE INX emp indx
FOR TABLE emp(
  employee_id DESC,
    UPPR(
      SUBSTR(first_name,1.1||" "||astname)
    )
)
~~~

Los datos del índice deben ser como estos:

|TABLE_NAME|INDEX_NAME|INDEX_TYPE|COLUMN_EXPRESSION|COLUMN_POSITION|
|---|---|---|---|---|
|EMP|EMP_INDX|FUNCTION-BASED NORMAL|"EMPLOYEE_ID"|1|
|EMP|EMP_INDX|FUNCTION-BASED NORMAL|UPPER(SUBSTR("FIRST_NAME",1,1)||' '||"LAST_NAME")|2|

* Escriba una sentencia SQL para mostrar todas las tablas de usuario que contienen el nombre `PRIV`. Utilice las vistas del diccionario de datos.

|TABLE_NAME|
|---|
|USER_AQ_AGENTS_PRIVS|
|USER_COL_PRIVS|
|USER_COL_PRIVS_MADE|
|USER_COL_PRIVS_RECD|
|USER_GOLDENGATE_PRIVILEGES|
|USER_NETWORK_ACL_PRIVILEGES|
|USER_REPGROUP_PRIVILEGES|
|USER_ROLE_PRIVS|
|USER_RSRC_CONSUMER_GROUP_PRIVS|
|USER_RSRC_MANAGER_SYSTEM_PRIVS|
|...|

* Conceda acceso de selección a público en la tabla EMP y verifique que se ha otorgado mediante la ejecución esta consulta. La consulta contiene errores que se deben corregir antes de poder ejecutar.

~~~sql
GRANT SELECT ON emp TO PUBLIC;

-- corregir esta sentencia
SELECT * FROM usr_tab_privs WHERE tablename = "emp"
~~~

Esto debe retornar:

|GRANTEE|OWNER|TABLE_NAME|GRANTOR|PRIVILEGE|GRANTABLE|HIERARCHY|
|---|---|---|---|---|---|---|
|PUBLIC|US_A009EMEA815_PLSQL_T01|EMP|US_A009EMEA815_PLSQL_T01|SELECT|NO|NO|

* Mediante las uniones propiedad de Oracle, construya una instrucción que devuelva todos los `employee_id` unidos a todos los `department_names`. Utilice las tablas `employees` y `departments`.

La salida debe ser parecida a esta:

|||
|---|---|
|...|...|
|104|Contracting|
|107|Contracting|
|124|Contracting|
|141|Contracting|
|142|Contracting|
|143|Contracting|
|144|Contracting|
|149|Contracting|
|174|Contracting|
|176|Contracting|
|178|Contracting|
|200|Contracting|
|201|Contracting|
|202|Contracting|
|205|Contracting|
|206|Contracting|

* Vuelva a utilizar las Uniones Oracle para corregir la sentencia anterior de modo que devuelva **sólo el nombre del departamento en que el empleado está trabajando actualmente.** Utilice las tablas `employees` y `departments`.

|EMPLOYEE_ID|DEPARTMENT_NAME|
|---|---|
|200|Administration|
|201|Marketing|
|202|Marketing|
|124|Shipping|
|144|Shipping|
|143|Shipping|
|142|Shipping|
|141|Shipping|
|107|IT|
|104|IT|
|103|IT|
|174|Sales|
|149|Sales|
|176|Sales|
|102|Executives|
|100|Executives|
|101|Executives|
|205|Accounting|
|206|Accounting|

* Vuelva a utilizar las uniones Oracle para crear una consulta que muestre el apellido de los empleados, el nombre de departamento, el salario y el nombre del país de todos los empleados. Utilice las tablas `employees`, `departments`, `locations` y `countries`.

|LAST_NAME|DEPARTMENT_NAME|SALARY|COUNTRY_NAME|
|---|---|---|---|
|King|Executive|24000|United States Of America|
|Kochhar|Executive|17000|United States Of America|
|De Haan|Executive|17000|United States Of America|
|Whalen|Administration|4400|United States Of America|
|Higgins|Accounting|12000|United States Of America|
|Gietz|Accounting|8300|United States Of America|
|Zlotkey|Sales|10500|United Kingdom|
|Abel|Sales|11000|United Kingdom|
|Taylor|Sales|8600|United Kingdom|
|Mourgos|Shipping|5800|United States Of America|
|Rajs|Shipping|3500|United States Of America|
|Davies|Shipping|3100|United States Of America|
|Matos|Shipping|2600|United States Of America|
|Vargas|Shipping|2500|United States Of America|
|Hunold|IT|9000|United States Of America|
|Ernst|IT|6000|United States Of America|
|Lorentz|IT|4200|United States Of America|
|Hartsteing|Marketing|13000|Canada|
|Fay|Marketing|6000|Canada|

* Vuelva a utilizar la sintaxis de unión de Oracle para modificar la consulta anterior de modo que incluya también el registro de empleado del empleado sin `department_id`, Grant. Utilice las tablas `employees`, `departments`, `locations` y `countries`.

|LAST_NAME|DEPARTMENT_NAME|SALARY|COUNTRY_NAME|
|---|---|---|---|
|Hartsteing|Marketing|13000|Canada|
|Fay|Marketing|6000|Canada|
|Zlotkey|Sales|10500|United Kingdom|
|Abel|Sales|11000|United Kingdom|
|Taylor|Sales|8600|United Kingdom|
|Hunold|IT|9000|United States Of America|
|Ernst|IT|6000|United States Of America|
|Lorentz|IT|4200|United States Of America|
|Mourgos|Shipping|5800|United States Of America|
|Rajs|Shipping|3500|United States Of America|
|Davies|Shipping|3100|United States Of America|
|Matos|Shipping|2600|United States Of America|
|Vargas|Shipping|2500|United States Of America|
|Higgins|Accounting|12000|United States Of America|
|Gietz|Accounting|8300|United States Of America|
|King|Executive|24000|United States Of America|
|Kochhar|Executive|17000|United States Of America|
|De Haan|Executive|17000|United States Of America|
|Whalen|Administration|4400|United States Of America|
|Grant|-|7000|-|
