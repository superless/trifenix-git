# TRIFENIX CONNECT
Trifenix connect son los cimientos tanto en arquitectura como metodológicos para la construcción de programas en la [nube nativa](https://en.wikipedia.org/wiki/Cloud_native_computing), con una base [devops](https://es.wikipedia.org/wiki/DevOps) y [scrum](https://es.wikipedia.org/wiki/Scrum_(desarrollo_de_software)), con el fin de ofrecer aplicaciones de escalabilidad ilimitada e instantaniedad en la experiencia de usuario.

![genkidama](https://images.trifenix.io/genkidama.gif)

**Obviamente Open Source**


# Principios de la Arquitectura

# Asincronía de datos
Que las operaciones se envíen desde las aplicaciones de manera asincrónica y tengan un panel de notificaciones de exito de la operación, esto permitirá que no se dependa de la velocidad de la base de datos al insertar, eliminar u actualizar una entidad. 
![asincrono](https://images.trifenix.io/images/asincrono.gif)


# Modelo de Metadatos
Que todos los esfuerzos sean unificados en un solo modelo de desarrollo de aplicaciones basado en los metadatos de las entidades de una aplicación.

A continuación un ejemplo, pero lo abordaremos más adelante.
![metadata](ihttps://images.trifenix.io/mages/metadata.png)

# Estructura única de documentos
Modelo único de Documentos Entity Search.

# 



# Base tecnológica

## 1. Automatización

Todos las operaciones de automatización, Integración continua / Entrega continua son llevadas actualmente sobre [Azure Devops](https://dev.azure.com), los proyectos que iremos construyendo vendrán con los scripts de automatización y se acompañaran de la estructura de variables a ingresar en el azure devops library.

La automatización juega un rol clave en la velocidad de construcción y en el exito de los proyectos.

## 2. Gestión de proyectos
Nuestros proyectos están basados en metodologías ágiles, tales como scrum o kanban, actualmente llevamos esta gestión en [Azure Devops](https://dev.azure.com) que nos da la oportunidad de dar seguimiento en cada fase del ciclo de vida de una funcionalidad de aplicación.

## 3. Nube
Nuestros esfuerzos siempre tenderán al multicloud, sin embargo en la actualidad usamos azure como motor principal de trifenix connect. Todas nuestras operaciones de backend son con C#.

## 4. Interfaz de usuario (UX).
Actualmente nuestras interfaces son construidas en [React](https://es.wikipedia.org/wiki/React), sin embargo nuestro modelo es ampliable a otros frameworks Ux.




# 1. Velocidad de los datos (a buen costo).

Partamos de la base de que toda aplicación debe tener una base de datos. 

La mayor parte de los programas que conocemos usan una base de datos llamada [SQL (Structured Query Language)](https://es.wikipedia.org/wiki/SQL) la cual hoy se encuentra tanto on  Premise, como en la nube, el costo de este tipo de base de datos, si se desea velocidad es muy elevado.  

Las bases de datos SQL tienen un reinado de 30 años, pero como todo lo viejo tiene que cambiar, desde hace ya unos 10 años que las bases de datos documentales han demostrado ser más eficientes, exactas, con un universo mayor de opciones e infinitamente escalables.

Nuestro modelo es ampliable a base de datos de SQL, pero por ahora enfoquemonos en nuestras queridas bases de datos documentales, SQl ya es el pasado.

Bueno, las bases de datos documentales son libres, su base fundamental se basa en un key y un value, normalmente el key será el id del elemento y el value será una estructura json con la entidad. 

El lenguaje de consultas utilizado por la base de datos depende bastante de la base de datos escogida, pero la mayoría tiene lenguajes de consultas que permiten llevar todos los filtros que se podrían imaginar.

# 1 aplicación una base de datos

Nuestra primera experiencia con bases de datos documentales fué con [Azure CosmosDb](https://azure.microsoft.com/es-es/services/cosmos-db/), como programadores ratas, buscabamos lo más barato posible y el [año gratis de cosmosdb](https://azure.microsoft.com/en-us/free/) nos venia muy bien. 

Lo bueno de las bases de datos documentales es que son exactas, en el plan gratis me brindaban 400 RU por segundo, donde 1 RU equivale a una lectura de 1kb, 5 ru equivale a una escritura de un 1 kb. entonces 400 ru se vé bastante bueno, en teoría.

Bueno, la verdad no tanto, aunque no medimos en detalle la lectura de los datos,  pudimos percibir con pocos datos, que mas temprano que tarde tendríamos que subir la palanquita de los RU, que por cierto van de los 400RU al infinito y más allá. 

Mientras veíamos frustradamente la velocidad de lectura de nuestro ultimo nivel de velocidad de cosmosdb, veíamos como los 50mb gratis de azure search era realmente instantaneo y su lenguaje de consultas funcionaba maravillosamente bien. 

Para entender que es azure search, hay que partir primero por decir, que es una base de datos documental, tal como lo es cosmosdb, pero de velocidad instantanea, este tipo de base de datos se puede reconocer en cualquier buscador de alguna aplicación de redes sociales o delivery. La lectura de los datos siempre será más rápida, cuando exista mayor indexación, la indexación confía de una estructura definida con anticipación. 

Hablando de indexación, un indice es una estructura definida previamente, donde cada elemento insertado debe obedecer a esa estructura.

Por ejemplo
![estructura](https://images.trifenix.io/estructura.png)

De acuerdo a las variables indicadas, se indexara cada campo, mientras más indexado sea un campo, más espacio ocuparará en la base de datos y más demorará en la inserción. 

De ahora en adelante solo se podrán insertar elementos como este

```
"value": [
        {
            "@search.score": 1,
            "Nombre": "Bebida fantasía",
            "Updated": "2020-10-24T05:36:38.354Z",
            "Stock_Fisico": 24,
            "Precio": 800,
            "Super_Categoria": "ALIM",
            "Categoria": "BEBI",
            "Sub_Categoria": "BGAS",
            "Id": "BEBIDA"
        }
    ]
```

Bueno, como se podrán imaginar un modelo de una aplicación no tiene una entidad sino muchas más y la versión gratuita nos limitaba a solo 3 :(

Esto ya había pasado antes con cosmos db, tuve que ocupar cosmonaut para crear distintos tipos de documentos (entidad) y cosmonaut uso el tipo de dato como partición, de esa manera podía usar una sola colección.

Pero Azure search es diferente, debo pagar si pongo más de tres entidades, entonces debo elegir sabiamente como ocupar estos tres indices, no es algo muy comodo, siempre tendré que estar creando objetos para lecturas particulares, esto no es muy automatizable. 


Nuestro principio siempre ha sido que la excelencia técnica debe permitir obtener el menor costo posible y veíamos con buenos ojos, poder replicar nuestra base de datos de cosmosdb en Azure Search, de tal manera que las inserciones no fueran nuestra preocupación, porque son asíncronas  (se notificará cuando termine) y si se desea que sea más rápido, solo debe subir la palanquita de los RU, por otro lado si el modelo estaba replicado en el azure search, nuestras consultas volarían a un precio fijo.

Era un gran desafío que nos apremiaba, porque nuestras lecturas eran lentas y no queríamos cargar la mano en el precio, si sabíamos que había una manera mejor, mas costosa en terminos de tiempo pero mejor. 

Como somos porfiados seguimos con la manera de lograr realizar el modelo perfecto, inserción de base de datos para históricos y lectura en base de datos de alta velocidad, si es posible, sin pagar :D. 

Bueno, ahora después de harto tiempo podemos finalmente documentar el proceso, pero claramente no fué fácil, el desafío supuso la creación de un modelo llamado Trifenix IMes, el que lleva consigo un modelo basado en metadatos llamado Trifenix Mdm.

Partamos con IMes

![Imes](https://images.trifenix.io/IMes.PNG)

Bueno, el modelo IMes lleva consigo varios conceptos que están en su mismo nombre. A continuación se enumeran estos conceptos de manera breve.

1. Input
Se refiere a las entradas de un sistema, una entidad que determina cuales son los valores ingresados por un usuario o máquina, en esta caso es un input de una temporada agricola. Un input debe heredar de InputBase, después veremos en detalle los atributos de cada propiedad, por ahora se debe entender que son parte del metadata model y permiten conectar los input, el model y los entitySearch, además de otros metadatos útiles dentro de trifenix connect.

```
[ReferenceSearchHeader(EntityRelated.SEASON)]
public class SeasonInput : InputBase {

    [Required]
    [DateSearch(DateRelated.START_DATE_SEASON)]
    public DateTime  StartDate { get; set; }

    [Required]
    [DateSearch(DateRelated.END_DATE_SEASON)]
    public DateTime EndDate { get; set; }

    [BoolSearch(BoolRelated.CURRENT)]
    public bool? Current { get; set; }

    [Required, Reference(typeof(CostCenter))]
    [ReferenceSearch(EntityRelated.COSTCENTER)]
    public string IdCostCenter { get; set; }

}
```

2. Model

El modelo refiere a la entidad que será guardada en una base de datos, un input puede originar uno o mas entidades de base de datos. A continuación se puede ver la misma temporada, pero en este caso tiene un campo ClientId, que es un autonumérico, por tanto no se requiere en la entrada.

También se puede notar como algunos metadatos vinculan los mismos campos que el input, a diferencia del input, esta clase usa un atributo de Cosmonaut, que lo identifica como entidad de una base de datos.

```
[SharedCosmosCollection("agro", "Season")]
[ReferenceSearchHeader(EntityRelated.SEASON, PathName = "seasons", Kind = EntityKind.CUSTOM_ENTITY)]
public class Season : DocumentBase, ISharedCosmosEntity {

    /// <summary>
    /// identificador.
    /// </summary>
    public override string Id { get; set; }

    /// <summary>
    /// Autonumérico del identificador del cliente.
    /// </summary>
    [AutoNumericSearch(StringRelated.GENERIC_CORRELATIVE)]
    public override string ClientId { get; set; }


    /// <summary>
    /// fecha de inicio
    /// </summary>
    [DateSearch(DateRelated.START_DATE_SEASON)]
    public DateTime StartDate { get; set; }


    /// <summary>
    /// fecha fin
    /// </summary>
    [DateSearch(DateRelated.END_DATE_SEASON)]
    public DateTime EndDate { get; set; }

    /// <summary>
    /// Identifica si el agricola es el actual.
    /// </summary>
    [BoolSearch(BoolRelated.CURRENT)]
    public bool Current { get; set; }

    /// <summary>
    /// identificador del costcenter.
    /// </summary>
    [ReferenceSearch(EntityRelated.COSTCENTER)]
    public string IdCostCenter { get; set; }

}
```

3. EntitySearch

Un entitySearch es una estructura definida para todo tipo de entidades, como se puedo apreciar en los ejemplos de input y model, cada propiedad fué identificada con un valor en su atributo, por ejemplo Current, encima de esta propiedad se encuentra el atributo BoolSearch en el que se indica dentro de un diccionario cual es el índice que representa a ese campo, en este caso BoolRelated.CURRENT que es igual a 0 (es el primero en el diccionario). Asi también en StarDate, EndDate  e IdCostCenter, se podrá dar cuenta que el atributo determina el tipo de dato y el índice al que pertenece.

A través de estos índices otorgados por el metadata model se puede transformar un objeto a un entitySearch y un entitySearch en un objeto. 

Un entitySearch es una estructura que puede albergar cualquier modelo, para esto debe tener una estrucutura definida, con el fin de que cada objeto sea convertido a un entitySearch y almacenado en el índice.

El inconveniente que presenta este tipo de modelo, es que siempre existirá un tamaño mínimo, porque la estructura se mantiene, tenga datos o no, por eso se ha hecho el esfuerzo en hacerla lo más simplicado posible para que ocupe el menor espacio, el beneficio es que el entitySearch tiene previamente asignado los índices, por tanto no habrá trabajo manual posterior. 

Antes de pasar a la explicación de las propiedades de un entitySearch, debemos conocer que es un facet, dentro de las bases de datos de busqueda.

![facet](https://images.trifenix.io/facet.png)

Como lo muestra la imágen son agrupaciones asociadas a una busqueda en partícular, por ejemplo, los facets de una consulta de productos, podrían ser las el listado de marcas encontradas en la consulta, otro facet puede ser el tipo de frutas al que puede ser aplicado, etc.


A continuación se enumerarán todas las propiedades de un entitySearch.

## a. id
el identificador de un elemento, todo elemento en entitySearch tiene un identificador.

## b. index
Siempre una entidad de base de datos tendrá un índice asignado por el modelo de metadatos, este índice será asociado a un diccionario que determinará la clase vinculada.
Por ejemplo, la entidad producto podría tener el índice 22.

## c. Created
Este campo identifica la fecha de creación de un elemento.

## d. rel
Un entitySearch es un grafo que se relaciona con otras entidades a través de esta propiedad, esta propiedad es una colección por tanto, un elemento se puede relacionar con 0 o más.
Una relación se compone de los siguientes campos.


### id
identificador del elemento relacionado a esta entidad

### index
índice del elemento relacionado, recordemos que todas las entidades de la base de datos están asociadas con un índice.

### facet
el facet es un campo de tipo string que tiene el índice "," el id del elemento, generando un valor como este "1,9aebaf15-eb85-49d7-acca-643329d4078b" 
Cuando hagamos una consulta podremos usar esta propiedad como facet, se agruparán los resultados por tipos de valor de esta propiedad, por ejemplo, digamos que queremos las marcas
en una consulta de procutos y las marcas tienen el índice 12, y digamos que dentro de la busqueda, se encontraron 2 marcas, la primera con 3 productos y la segunda con 7, como lo que sigue:

```
[
    {"12,9aebaf15-eb85-49d7-acca-643329d4078b":3},
    {"12,7990893f-74e1-45d6-8f3d-af1c9896842c":7},
]
```
Al asignar este campo como facet, poniendo el índice y el id, se puede identificar las dos marcas y el número de registros encontrados.

















4. Mdm o Metadata Model





## 2. Software Factory
