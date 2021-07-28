- [Volver al inicio](index.html)

## 19.1 Pruebas

La mayoría de usuarios compran un vehículo deseando saber que es
fiable y no se va a romper. Lo mismo ocurre con las BBDD. Antes
de venderla, se debe probar para verificar que cumple con los
requisitos del negocio.

### Pruebas de Unidades

Si se prueban dos cosas a la vez y falla la prueba, es difícil o
imposible averiguar qué ha provocado el fallo, por lo que es
importante probar sólo una cosa a la vez. **Esto se conoce como
prueba de unidades.**

Al probar una BD, se deben probar una gran cantidad de elementos:

- Que las columnas contengan el tipo de dato correcto.
- Que las columnas alberguen la mayor cantidad de datos que se puedan
  introducir.
- Que las restricciones restringan sólo los datos que deben restringir, ni más
  ni menos.

### ¿Qué se debe probar?

Con frecuencia resulta poco realista probar todas las columnas y 
todas las restricciones de cada tabla de una BD, especialmente si
se trata de una de gran tamaño. Deben entonces realizarse una serie de pruebas aleatorias, que comprueben algunas columnas y algunas restricciones.

### Diseño de las Pruebas

Antes de realizar una prueba, se debe tener una buena idea de lo
que se espera ver como resultado si la BD funciona correctamente.
Esta debe documentarse en forma similar a la descrita en esta 
tabla:

|Número de  Prueba|Fecha|Descripción|Entrada|Salida Esperada|Resultado/Discrepancia|Acción|
|---|---|---|---|---|---|---|
|22|19-Aug-2006|Confirmar restricción NOT NULL en JOB_TITLE en JOBS|INSERT INTO jobs (job_id, job_title, min_salary, max_salary) VALUES (222,NULL,100,200)|No se puede Introducir NULL|||

### Ejecución de las Pruebas

Una vez que haya diseñado la prueba, debe ejecutarla y registrar
sus resultados:

|Número de  Prueba|Fecha|Descripción|Entrada|Salida Esperada|Resultado/Discrepancia|Acción|
|---|---|---|---|---|---|---|
|22|19-Aug-2006|Confirmar restricción NOT NULL en JOB_TITLE en JOBS|INSERT INTO jobs (job_id, job_title, min_salary, max_salary) VALUES (222,NULL,100,200)|No se puede Introducir NULL|No se puede introducir NULL|Ninguna|
