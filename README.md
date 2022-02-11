# Proyecto N°2 - BD1
### Leonardo Roney Martínez Maldonado - 201780044

En este proyecto se nos bridó un archivo que contenía toda la información de las elecciones políticas que se deseaba migrar a una base de datos relacional. Se planteó un modelo relacional que almacenara de la mejor manera los datos. También se detalla el proceso de normalización que se llevó a cabo partiendo de una tabla temporal generada por la carga masiva de un archivo [ICE-Fuente.xls](https://tinyurl.com/yykzknzh), hasta la realización del modelo relacional propuesto. En pocas palabras, realizar una ingeniería inversa aplicando consultas a la tabla temporal.

![Modelo Entidad Relación Propuesto](https://github.com/leonardo0martinez/bases1/tree/master/p2/vainas/er_logico.png)

# Normalización de la Base de Datos
## 1ra Forma Normal
Para aplicar la primera forma normal a la tabla temporal se le removieron las columnas repetidas del archivo de entrada [enlace]:

- NOMBRE_ELECCION
- AÑO_ELECCION
- PAIS
- REGION
- DEPTO
- MUNICIPIO
- PARTIDO
- NOMBRE_PARTIDO
- SEXO
- RAZA
- ANALFABETOS
- ALFABETOS
- ~~SEXO~~
- ~~RAZA~~
- PRIMARIA
- NIVEL MEDIO
- UNIVERSITARIOS

Estas columnas repetidas se eliminan directamente en la carga masiva simplemente asignándolas a una variable de usuario y no haciendo uso de ella, para este proyecto sería de esta manera:
```sql
load data local infile 'RUTA ARCHIVO :)'
into table Temporal columns terminated by ','
lines terminated by '\r\n'
ignore 1 lines (
    nombre_eleccion,
    año_eleccion,
    pais,
    region,
    depto,
    municipio,
    partido,
    nombre_partido,
    sexo,
    raza,
    analfabetos,
    alfabetos,
    @nosirvexd,
    @nosirvexd,
    primaria,
    nivel_medio,
    universitarios
);
```
Una vez creada la tabla temporal, se pueden apreciar los campos repetidos y empezar a separar los atributos para generar las entidades, a continuación se muestran las primeras 3 filas de datos:
| nombre_eleccion        | año_eleccion | pais        | region   | depto   | municipio     | partido | nombre_partido    | sexo    | raza      | analfabetos | alfabetos | primaria | nivel_medio | universitarios |
|------------------------|--------------|-------------|----------|---------|---------------|---------|-------------------|---------|-----------|-------------|-----------|----------|-------------|----------------|
| Elecciones Municipales | 2005         | EL SALVADOR | REGION 1 | Cabañas | Sensuntepeque | ARENA   | Alianza Nac. Rep. | hombres | INDIGENAS | 2298        | 4800      | 1471     | 450         | 2879           |
| Elecciones Municipales | 2005         | EL SALVADOR | REGION 1 | Cabañas | Ilobasco      | ARENA   | Alianza Nac. Rep. | hombres | INDIGENAS | 2448        | 5536      | 1807     | 2966        | 763            |
| Elecciones Municipales | 2005         | EL SALVADOR | REGION 1 | Cabañas | Victoria      | ARENA   | Alianza Nac. Rep. | hombres | INDIGENAS | 1724        | 3638      | 1421     | 1183        | 1034           |

En primera instancia se debe de separar las columnas o atributos que por sí solas podrías ser una entidad de donde partirán todas las primeras relaciones, es decir, se obtienen son un simple `select distinct` a la tabla temporal ya que estas tablas serán las primeras en crearse por que no dependerán  de ninguna otra para que estas existan. A demás, como la tabla temporal no posee ningún tipo de identificador para cada una de las tuplas, se le creará uno en cada entidad que convenga realizarla:
>**Nota:** las consultas para la carga de datos están en el de CargaDatos.sql, la carga debe realizarse en el orden en el que aparecen los queries del archivo.
>
### Tabla Elección
```sql
insert into Eleccion(nombre, año)
select distinct nombre_eleccion, año_eleccion
from Temporal t;
```
Obteniendo como resultado todas las elecciones que se llevaron a cabo identificadas con un id:
| id_eleccion | nombre                 | año |
|-------------|------------------------|--------------|
| 1           | Elecciones Municipales | 2005         |
| 2           | Elecciones Municipales | 2001         |

El mismo procedimiento fue también es realizado para la tabla *Partido*, *País* y *Región* con las respectivas columnas de la tabla temporal. Después, se obtienen las tablas que mantienen una relación con las ya creadas, las relaciones más simples con las tablas ya creadas son la tabla *Departamento* y *Municipio*. Un país y una región ambos puede tener múltiples departamentos  y a su vez el departamento puede tener múltiples municipios por lo que al traer los datos de la tabla temporal se deben de comparar con las creadas con anterioridad.
| pais        | region   | depto        | municipio     |
|-------------|----------|--------------|---------------|
| EL SALVADOR | REGION 1 | Cabañas      | Sensuntepeque |
| EL SALVADOR | REGION 1 | Cabañas      | Ilobasco      |
| EL SALVADOR | REGION 1 | Cabañas      | Victoria      |
| EL SALVADOR | REGION 1 | Cabañas      | San Isidro    |
| EL SALVADOR | REGION 1 | Cabañas      | Jutiapa       |
| EL SALVADOR | REGION 1 | Cabañas      | Tejutepeque   |
| EL SALVADOR | REGION 1 | Cabañas      | Dolores       |
| EL SALVADOR | REGION 1 | Cabañas      | Cinquera      |
| EL SALVADOR | REGION 1 | Cabañas      | Guacotecti    |
| EL SALVADOR | REGION 2 | Chalatenango | Chalatenango  |

### Tabla Departamento
```sql
insert into Departamento(depto, id_region, id_pais)
select distinct depto, r.id_region, p.id_pais
from Temporal t, Pais p, Region r
where t.pais = p.pais and r.nombre = t.region;
```
Obteniendo como resultado todos los departamentos por país y por región identificadas con un id:
| id_depto | depto        | id_region | id_pais |
|----------|--------------|-----------|---------|
| 1        | Cabañas      | 1         | 1       |
| 2        | Chalatenango | 2         | 1       |
| 3        | Cuscatlán    | 3         | 1       |
| 4        | La Libertad  | 3         | 1       |
| 5        | La Paz       | 3         | 1       |
| 6        | La Unión     | 4         | 1       |
| 7        | Morazán      | 4         | 1       |

Y una vez creada la tabla departamento fue solo comparar el departamento con el municipio para generar la tabla de *Municipio*. Por último, se deja la tabla que posee más relaciones que todas, para el proyecto fue la tabla de *Votación* cantidad de votos, el partido y municipio donde se llevo acabo la votación, esta entidad sería prácticamente una copia de la tabla temporal ya que aquí los campos varían según la cantidad de votos por lo que en esencia se podría decir que son únicos. Por lo que para ello basta con obtener el id del municipio, le id de la elección que se está llevando a cabo y el id del partido  de las tablas previas para poder completar la consulta.

| municipio     | partido | nombre_partido    | sexo    | raza      | analfabetos | alfabetos | primaria | nivel_medio | universitarios |
|---------------|---------|-------------------|---------|-----------|-------------|-----------|----------|-------------|----------------|
| Sensuntepeque | ARENA   | Alianza Nac. Rep. | hombres | INDIGENAS | 2298        | 4800      | 1471     | 450         | 2879           |
| Ilobasco      | ARENA   | Alianza Nac. Rep. | hombres | INDIGENAS | 2448        | 5536      | 1807     | 2966        | 763            |
| Victoria      | ARENA   | Alianza Nac. Rep. | hombres | INDIGENAS | 1724        | 3638      | 1421     | 1183        | 1034           |
| San Isidro    | ARENA   | Alianza Nac. Rep. | hombres | INDIGENAS | 583         | 2772      | 1747     | 895         | 130            |

### Tabla Votación
```sql
select e.id_eleccion, q1.id_muni,
    p.id_partido, t.sexo, t.raza, 
    t.analfabetos, t.alfabetos, 
    t.primaria, t.nivel_medio, t.universitarios
from Temporal t, Partido p, Eleccion e,
(
    select m.id_muni, d.depto, m.municipio
    from Departamento d
    inner join Municipio m
    on d.id_depto = m.id_depto
) q1
where q1.depto = t.depto and 
    q1.municipio = t.municipio and
    p.nombre_partido = t.nombre_partido and 
    e.año = t.año_eleccion;
```
Obteniendo como resultado todas las votaciones por la elección que se llevó a cabo, el municipio, el partido y la cantidad de votos de cada sección de la población:

| id_eleccion | id_muni | id_partido | sexo    | raza      | analfabetos | alfabetos | primaria | nivel_medio | universitarios |
|-------------|---------|------------|---------|-----------|-------------|-----------|----------|-------------|----------------|
| 1           | 9       | 7          | hombres | INDIGENAS | 669         | 4624      | 2864     | 824         | 936            |
| 1           | 8       | 7          | hombres | INDIGENAS | 2971        | 6173      | 2605     | 2000        | 1568           |
| 1           | 7       | 7          | hombres | INDIGENAS | 2160        | 3539      | 179      | 831         | 2529           |
| 1           | 6       | 7          | hombres | INDIGENAS | 2330        | 2745      | 612      | 306         | 1827           |
| 1           | 5       | 7          | hombres | INDIGENAS | 2162        | 3431      | 1264     | 1261        | 906            |


## 2da Forma Normal

Para la segunda forma normal nos centraremos en la tabla Votación ya que esta todavía se puede dividir en más tablas. Para comenzar se puede notar que los atributos *id_elección*, *id_muni *, *id_partido* son los que se repiten a lo largo de la tabla y no dependen funcionalmente de la cantidad de votos en ese municipio por lo que se le puede asignar un id a la votación y separar la cantidad de votos en otra tabla *Datos_Votantes*, por lo que la tabla de *Votación* quedaría de la siguiente forma:
>**Nota:** El campo *total_votos* es un campo extra que muestra el total de votos del partido en ese municipio, para la facilidad de las consultas requeridas.
>
| id_votacion | id_eleccion | id_muni | id_partido | total_votos |
|-------------|-------------|---------|------------|-------------|
| 1           | 1           | 9       | 13         | 34878       |
| 2           | 1           | 8       | 13         | 37867       |
| 3           | 1           | 7       | 13         | 39899       |
| 4           | 1           | 6       | 13         | 42041       |

Ahora las votaciones quedarán con un id que permite identificar los municipios y el partido donde se realizaron las votaciones por lo que la consulta variará un poco quedando de la siguiente manera:
```sql
insert into Votacion(id_eleccion, id_muni, id_partido, total_votos)
select e.id_eleccion, q1.id_muni, p.id_partido, (sum(alfabetos) + sum(analfabetos)) suma
from Temporal t, Partido p, Eleccion e,
(
    select m.id_muni, d.depto, m.municipio
    from Departamento d
    inner join Municipio m
    on d.id_depto = m.id_depto
) q1
where q1.depto = t.depto and 
    q1.municipio = t.municipio and
    p.nombre_partido = t.nombre_partido and 
    e.año = t.año_eleccion
group by e.id_eleccion, q1.id_muni, p.id_partido;
```

La última entidad, *Datos_Votantes* sería la encargada de manejar los votos por tipo de población encontrados en la tabla temporal, por lo que la tabla quedaría de la siguiente manera:

| id_votacion | sexo    | raza      | analfabetos | alfabetos | primaria | secundaria | universitaria |
|-------------|---------|-----------|-------------|-----------|----------|------------|---------------|
| 10          | hombres | INDIGENAS | 669         | 4624      | 2864     | 824        | 936           |
| 11          | hombres | INDIGENAS | 2971        | 6173      | 2605     | 2000       | 1568          |
| 12          | hombres | INDIGENAS | 2160        | 3539      | 179      | 831        | 2529          |
| 13          | hombres | INDIGENAS | 2330        | 2745      | 612      | 306        | 1827          |

Para la consulta de *Datos_Votantes* se explica en la siguiente sección ya que este aún tiene un campo reducible.

## 3ra Forma Normal
A este punto, la base de datos estaría prácticamente normalizada con los entidades con datos atómicos y sin repeticiones. Pero en la tabla *Datos_Votantes* hay dos campos que poseen texto que se pueden separar en otras dos entidades ya que estas no dependen funcionalmente de la cantidad de votos únicamente de *id_votacion*, es decir, que cuando el cambio de una columna que no es clave primaria puede hacer que cambie cualquier otra columna que no sea clave. Para este caso son el campo *raza* y *sexo*, por lo que se crearía una nueva entidad *Raza* que posea una relación con *Datos_Poblacion*. La tabla *Raza* quedaría de la siguiente forma: 
>**Nota:** a mi criterio la única que separé fue la de raza, porque el sexo solo poseía dos valores y no se puede aumentar la cantidad de sexos, en cambio, las razas si pueden aumentar,  por lo que se dejó como atributo *sexo*.
>
### Tabla Raza
```sql
insert into Raza(raza)
select distinct raza
from Temporal t;
```
Obteniendo como resultado todas las razas que tomaron en cuenta para las votaciones identificadas con un id:
| id_raza | raza      |
|---------|-----------|
| 1       | INDIGENAS |
| 2       | LADINOS   |
| 3       | GARIFUNAS |

La consulta para *Datos_Población*, tomando los datos de la tabla temporal y de las entidades anteriormente creadas sería así:
```sql
insert into Datos_Votantes(id_votacion, sexo, id_raza, analfabetos, alfabetos, primaria, secundaria, universitaria)
select v.id_votacion, q2.sexo, q2.id_raza, q2.analfabetos, q2.alfabetos, q2.primaria, q2.nivel_medio, q2.universitarios
from Votacion v, (
select e.id_eleccion, q1.id_muni, p.id_partido, sexo, r.id_raza, analfabetos, alfabetos, primaria, nivel_medio, universitarios
from Temporal t, Partido p, Eleccion e, Raza r,
(
    select m.id_muni, d.depto, m.municipio
    from Departamento d
    inner join Municipio m
    on d.id_depto = m.id_depto
) q1
where q1.depto = t.depto and 
    q1.municipio = t.municipio and
    p.nombre_partido = t.nombre_partido and 
    e.año = t.año_eleccion and
    r.raza = t.raza#;
)q2
where v.id_eleccion = q2.id_eleccion and v.id_muni = q2.id_muni and v.id_partido = q2.id_partido;
```
Por lo que la tabla de *Datos_Población* normalizada quedaría de la siguiente manera:
| id_votacion | sexo    | id_raza | analfabetos | alfabetos | primaria | secundaria | universitaria |
|-------------|---------|---------|-------------|-----------|----------|------------|---------------|
| 10          | hombres | 1       | 669         | 4624      | 2864     | 824        | 936           |
| 11          | hombres | 1       | 2971        | 6173      | 2605     | 2000       | 1568          |
| 12          | hombres | 1       | 2160        | 3539      | 179      | 831        | 2529          |
| 13          | hombres | 1       | 2330        | 2745      | 612      | 306        | 1827          |


## Modelo Físico de BD

![](https://github.com/leonardo0martinez/bases1/tree/master/p2/vainas/er_fisico.png)
