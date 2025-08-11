# Dapper

## Introducción a Dapper 👓

Sobre la librería de _Ado.Net_ la gente de [Stackoverflow](https://stackoverflow.com/) (foro muy popular de programación que inicio con preguntas sobre _.NET_) creó un _wrapper_ (abstracción) para facilitar el uso de la librería original.  
Dapper **extiende** la Interfaz de _Ado.Net_ `IDbConnection`, una interfaz que describe un acceso a una BD Relacional donde los proveedores de BD's la implementan y distribuyen. También se conoce a **Dapper** como un _MicroORM_ (termino que se refiere a su capacidad de obtener objetos instanciados a través de filas de tablas y viceversa).

## Definiendo la estructura que vamos a usar 📁

Para los siguientes ejemplos vamos a partir de que tenemos las siguientes estructuras

```csharp
public class Persona
{
    public string Nombre {get; set;}
    public string Apellido {get; set;}
    public uint Dni {get; set;}
}
```

| Atributo de Tabla | Tipo de Dato |
| :---------------: | :----------: |
| nombre            | VARCHAR(45)  |
| apellido          | VARCHAR(45)  |
| dni               | INT UNSIGNED |

## Principales métodos de Dapper

Vamos a ver un listado de los principales métodos que agregan al comportamiento de `IDbConnection`, es importante entender que cuando hablamos de métodos genéricos (`<T>`) hablamos de métodos donde le tenemos que explicitar el tipo de dato a trabajar (primitivos como `int`, `string`, `double`, etc y/o complejos como clases definidas por nosotros):

### Query\<T>

Es un método genérico que permite obtener una colección de objetos del tipo `T`. Puede ejecutar consultas o _Procedimientos Almacenados_ que devuelvan filas.

### QueryFirstOrDefault\<T>

Es un método genérico que permite obtener un elemento y en caso de que devuelva nada, el valor por defecto para el tipo `T`.

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