# PRUEBA DATA SCIENTIST R5

El siguiente proyecto es creado para darle solución al reto de R5 para el puesto de  Data Scientist Junior


## Introducción
El reto se basa en desarrollar lo siguiente:
 - Una base de datos en el gestor postgresql
    - Dentro de esta base de datos se debe crear una consulta para ver el porcentaje de fraudes y siniestros 
- A traves de Python conectarse a la base de datos creada.
    - Se debe traer el conjunto de datos de la manera más limpia posible.
    - Se debe analizar y buscar al menos 3 insigths y realizar algunas recomendaciones para reducir fraudes.
    - **De manera opcional** se puede crear un modelo de Machine Learning que ayude a evitar el fraude.

- Se debe crear un tablero interactivo que permita visualizar los principales hallazgos encontrados en el punto anterior.

- Se finaliza el reto al realizar recomendaciones y unas acciones posibles a tomar para evitar fraude.

## Solución Paso a Paso
Para la solución se realiza una explicación paso a paso para poder replicar el experimento y los resultados obtenidos.

**Desarrollo de la base de datos.**
Para este paso se realizo el despliegue en un contenedor de Docker, para esto se realizo lo siguiente:

Hacemos un pull a la imagen de Docker
```bash
  docker pull postgres
```

Creamos nuestro contenedor

**Nota:** para este caso el contenedor usa el usuario **postgres** y la contraseña **mysecretpassword**, además de usar el puerto **5432**
```bash
   docker container run `
   --name some-postgres `
   -e POSTGRES_PASSWORD=mysecretpassword `
   -dp 5432:5432 postgres
```

Despues de tener desplegado el servidor en docker se uso [Tableplus](https://tableplus.com) ya que permite la integración de diversas bases de datos sin un mayor problema.

Cabe aclarar que la creación de la tabla se hizo usando el archivo que esta en ./data/create_table.txt

Para el proceso de inserción de datos se utilizo el siguiente contenedor dentro de Docker para compartir el archivo.

_(Para la ruta_en_el_computador/ se recomienda apuntar toda la ruta y no la ruta relativa y para el container_id es el identificador del contenedor en Docker.)_

```bash
   docker cp ruta_en el_computador/r5-ds-challenge/data/fraud.csv id_container:/var/lib/postgresql/data/archivo.csv
```

Dentro de postgresql se opto por hacer una insercción usando el siguiente comando:

```bash
   COPY "public"."fraudes" (
    Monthh,
    WeekOfMonth,
    DayOfWeek,
    Make,
    AccidentArea,
    DayOfWeekClaimed,
    MonthClaimed,
    WeekOfMonthClaimed,
    Sex,
    MaritalStatus,
    Age,
    Fault,
    PolicyType,
    VehicleCategory,
    VehiclePrice,
    FraudFound_P,
    PolicyNumber,
    RepNumber,
    Deductible,
    DriverRating,
    Days_Policy_Accident,
    Days_Policy_Claim,
    PastNumberOfClaims,
    AgeOfVehicle,
    AgeOfPolicyHolder,
    PoliceReportFiled,
    WitnessPresent,
    AgentType,
    NumberOfSuppliments,
    AddressChange_Claim,
    NumberOfCars,
    Yearr,
    BasePolicy
)
FROM '/var/lib/postgresql/data/archivo.csv' 
DELIMITER ',' CSV HEADER;
```


**Para el punto de: 'con su base de datos cargada escriba un query de SQL que replique la siguiente salida sin usar subconsultas.'**
Se uso la siguiente consulta:
```bash
SELECT
DISTINCT monthh,
weekofmonth,
dayofweek,
ROUND((SUM(fraudfound_p) OVER (PARTITION BY monthh) * 100) / NULLIF(COUNT(fraudfound_p) OVER (PARTITION BY monthh), 0.0),2) AS percentage_fraud_month,
ROUND((SUM(fraudfound_p) OVER (PARTITION BY monthh, weekofmonth) * 100) / NULLIF(COUNT(fraudfound_p) OVER (PARTITION BY monthh, weekofmonth), 0.0),2) AS percentage_fraud_month_week,
ROUND((SUM(fraudfound_p) OVER (PARTITION BY monthh, weekofmonth, dayofweek) * 100) / NULLIF(COUNT(fraudfound_p) OVER (PARTITION BY monthh, weekofmonth, dayofweek), 0.0),2) AS percentage_fraud_month_week_day
FROM fraudes
ORDER BY monthh, weekofmonth;
```

El resultado de esta consulta fue el siguiente:

<img src="/data/salida_obtenida.png" alt="Resultado de la consulta SQL">



- **Con relación al punto 3 y 4 se puede consultar estos puntos en la ruta _/notebooks/Análisis Descriptivo.ipynb_**
- Para el punto 4.1 se puede encontrar los datos del modelo [**Aquí**](./models/modelo_clasificador.pkl) y los datos del pipeline [**Aquí**](models/pipeline_data.pkl)


**Para la creación ei mplementación de un tablero, se opto por el uso de Power BI, en estas se realizaron algunos analisis en relación a las columnas más relevantes para la detección de Fraude**
Para consultar el tablero  puede hacerlo [**Aquí**](https://app.powerbi.com/view?r=eyJrIjoiMWQ0YjViOTUtODAzZC00MjM5LWFiMTEtYmQxZDhkMjExZWRjIiwidCI6IjA3ZGE2N2EwLTFmNDMtNGU4Yy05NzdmLTVmODhiNjQ3MGVlNiIsImMiOjR9) o si desea puede ejecutar este tablero a traves de la ejecución del archivo que esta en [**la siguiente carpeta**](./data/data_tablero_powerbi.pbix)


**Respecto al punto 4 y al punto 6, para las conclusiones y las acciones a tomar, se opta por analizar y recomendar lo siguiente:**

A traves del analisis respectivo se pudieron encontrar algunos elementos que fueron de interes porque en general pueden darnos una idea de a qué datos podemos darle un mayor foco para saber si las personas representan un riesgo respecto al fraude.

- Respecto a la categoría del vehículo (VehicleCatergory) se observa que los Sedan son los autos más susceptibles a ser utilizados en fraudes, pero esto puede deberse a aspectos como la generalización de estos vehiculos, ya que representa la mayoria de autos (9670 autos).

- Para el estado civil (MaritalStatus) se pudo observar que las personas casadas representan un mayor porcentaje de personas que cometen fraude y esto se puede deber a aspectos sociales y economicos, que pueden llevar a las personas casadas a cometer fraude, tambien es importante mencionar que en su mayoria los hombres son aquellos que cometen más fraude, de una manera mucho más generalizada que las mujeres.

- Con relación al tipo de seguro (BasePolicy) se encontro una relación respecto al fraude, ya que aquellas personas que tienen seguro 'Todo Riesgo' son más susceptibles a cometer Fraude, aun cuando estos no representan la mayoria de manera general.

- Respecto al costo del seguro se encontro que aquellas personas que tienen seguros en el rango de 400, representan el 92.7% de los fraudes, lo cual nos da una idea de que en estos seguros se debe prestar mayor atención ya que suelen ser más susceptibles a las estafas.

- Por ultimo es importante mencionar que la edad juega un papel demasiado importante, ya que aquellas personas que tienen entre 16 a 23 años y 65 o más años, son los que tienen mayor susceptibilidad a cometer fraude, esto puede deberse a diversos factores tanto sociales como economicos, ya que una situación financiera precaria puede llevar a las personas a participar de actividades fraudulentas para obtener algo de dinero.


**Respecto a las decisiones a tomar para evitar el fraude**

Se pueden llegar a varias acciones para evitar el fraude de manera generalizada, dentro de estos se pueden realizar:

- Implementar revisiones más robustas para personas menores de 23 años y mayores de 65.
- Realizar peritajes más intensivos cuando los reclamantes sean hombres y personas casadas.

- Evaluar la opción de dar paquetes más robustos que incluyan sensores que permitan monitorizar el carro cuando los seguros sean todo riesgo o caundo los carros sean Sedan.

- Evaluar la opción de dar paquetes más robustos que incluyan sensores que permitan monitorizar el carro cuando los seguros sean todo riesgo o caundo los carros sean Sedan.
