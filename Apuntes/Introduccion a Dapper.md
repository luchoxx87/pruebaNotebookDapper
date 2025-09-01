# Dapper

## Introducción a Dapper 👓

Sobre la librería de _Ado.Net_ la gente de [Stackoverflow](https://stackoverflow.com/) (foro muy popular de programación que inicio con preguntas sobre _.NET_) creó un _wrapper_ (abstracción) para facilitar el uso de la librería original.  
Dapper **extiende** la Interfaz de _Ado.Net_ `IDbConnection`, una interfaz que describe un acceso a una BD Relacional donde los proveedores de BD's la implementan y distribuyen. También se conoce a **Dapper** como un _MicroORM_ (termino que se refiere a su capacidad de obtener objetos instanciados a través de filas de tablas y viceversa).

## Definiendo la estructura que vamos a usar 📁

Para los siguientes ejemplos vamos a partir de que tenemos las siguientes estructuras

### Nuestra clase

```csharp
public class Persona
{
    public string Nombre {get; set;}
    public string Apellido {get; set;}
    public uint Dni {get; set;}
}
```
### Nuestra Tabla y Muestra

| Atributo de Tabla | Tipo de Dato |
| :---------------: | :----------: |
| nombre            | VARCHAR(45)  |
| apellido          | VARCHAR(45)  |
| dni               | INT UNSIGNED |

| Nombre | Apellido | Dni      |
| ------ | -------- | -------- |
| Ana    | García   | 12345678 |
| Luis   | Duran    | 87654321 |
| Luis   | López    | 44556677 |

## Principales métodos de Dapper

Vamos a ver un listado de los principales métodos que agregan al comportamiento de `IDbConnection`, es importante entender que cuando hablamos de métodos genéricos (`<T>`) hablamos de métodos donde le tenemos que explicitar el tipo de dato a trabajar (primitivos como `int`, `string`, `double`, etc y/o complejos como clases definidas por nosotros):

### Query\<T>

Es un método genérico que permite obtener una colección de objetos del tipo `T`. Puede ejecutar consultas o _Procedimientos Almacenados_ que devuelvan filas.

#### Ejemplo con Consulta

```csharp
string sql = "SELECT Nombre, Apellido, Dni FROM Persona";
IEnumerable<Persona> personas = db.Query<Persona>(sql);
personas.ForEach(p => Console.WriteLine($"{p.Nombre} {p.Apellido} - DNI: {p.Dni}"));
```

Y la salida es:

```shell
Ana García - DNI: 12345678
Luis Duran - DNI: 87654321
Luis López - DNI: 44556677
```

### Parámetros

Para las consultas que desarrollemos, va a ser común asignarle parámetros, veamos 2 formas que tiene _Dapper_ de implementarlos. Es importante tener en cuenta también que las implementaciones establecidas por _Dapper_ previenen ataques de _SQL Injection_.

#### Dapper.DynamicParameters

Es una clase desarrollada por _Dapper_ que sirve como una colección de parámetros para _inyectar_ en nuestra consulta parametrizada.

```csharp
DynamicParameters parametros = new DynamicParameters();
//El primer parámetro es el nombre, el segundo su valor
parametros.Add("unNombre", "Luis");

var sql = @"SELECT * FROM Persona WHERE nombre = @unNombre";
var personas = db.Query<Persona>(sql, parametros);
```

Como vemos en el ejemplo anterior, dentro de la cadena de nuestra consulta, en la sección del `WHERE` colocamos el parámetro `@unNombre`, el cual _Dapper_ va a inyectarle los valores que hayamos asignado en el objeto `parametros`. La salida sera:

```shell
Luis Duran - DNI: 87654321
Luis López - DNI: 44556677
```

Veamos ahora como asignarle más de un parámetro:

```csharp
var parametros = new DynamicParameters();
parametros.Add("unNombre", "Luis");
parametros.Add("unApellido", "Duran");

var sql = @"SELECT * FROM Persona WHERE nombre = @unNombre AND apellido = @unApellido";
var personas = db.Query<Persona>(sql, parametros);
```

Y nuestra salida:

```shell
Luis Duran - DNI: 87654321
```

Es muy importante notar, que tiene que haber correspondencia entre el nombre del parámetro dentro de nuestra consulta (`@variable`) con el que le pasamos nosotros dentro del método `DynamicParameters.Add()`; tambien se tienen que corresponder los tipos de datos.
### QueryFirstOrDefault\<T>

Es un método genérico que permite obtener un elemento y en caso de que devuelva nada, el valor por defecto para el tipo `T` (_null_ si es un tipo definido por nosotros).

### Execute

Es un método destinado a ejecutar instrucciones SQL que no devuelvan filas (_INSERT, UPDATE, DELETE, Stored Function y Procedures_). Devuelve un entero que representa la cantidad de filas involucradas. 

|       Método      | Descripción  |Devolución |
| :---------------: | :---------: | :----------: |
| `Query<T>`       | Es un método genérico que permite obtener una colección de objetos del tipo `T`. Puede ejecutar consultas o _Procedimientos Almacenados_ que devuelvan filas.  | `IEnumerable<T>` |
| `QueryFirstOrDefault<T>`       | Es un método genérico que permite obtener un elemento y en caso de que devuelva nada, el valor por defecto para el tipo `T`.  | `T` ó valor por defecto |
| `Execute`       | Es un método destinado a ejecutar instrucciones SQL que no devuelvan filas (_INSERT, UPDATE, DELETE, Stored Function y Procedures_). Devuelve un entero que representa la cantidad de filas involucradas.  | `int`|

### Parámetros en Dapper

Ya sea para las consultas SQL o ejecución de _SP/SFs_ vamos a necesitar pasar parámetros a los métodos Dapper que puede ejecutar la conexión.

Hay 2 formas de inyectar nuestros parámetros en Dapper.

#### Parámetros por objetos anónimos

En `C#` existe un concepto que se llama _Objetos anónimos_, objetos que se definen en el momento, sin la necesidad de definir previamente una clase. Veamos un poco de su implementación.

Vamos

```csharp
string queryPersonaPorDNI =
    @"SELECT *
    FROM    Persona
    WHERE   dni = @unDni";
```

[Volver al indice](../README.md)