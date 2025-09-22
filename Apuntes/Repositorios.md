# Patrón Repositorio

Una vez que tenemos desarrollada nuestra lógica de negocios, vamos a necesitar persistirla, es decir poder guardar la información de nuestras entidades en Memoria Secundaria para poder guardar su estado una vez que se cierre nuestra aplicación.

Si bien existen distintas metodologías desde el Paradigma Orientado a Objetos, vamos a encararlo desde la metodología del Patrón Repositorio.

Este patrón tiene como objetivo **desacoplar** la Lógica de Negocios del Acceso a Datos y básicamente consiste en por cada una de nuestras entidades relevantes, dedicar una _Interfaz_ para su persistencia y lectura en la Memoria Secundaria. Existen varias formas de persistir como Bases de Datos Relacionales, No Relacionales, Archivos Binarios, etc. pero nosotros nos vamos a dedicar a las BDs Relacionales (MySQL 🐬)

### Ejemplo supermercado

Partimos de nuestro modelo base

```mermaid
classDiagram
    direction LR
    class Categoria {
        - id: byte
        - nombre: string
    }

    class Producto {
        - id: short
        - cantidad: short
        - categoria: Categoria
        - historiales: List<HistorialPrecio>
        - nombre: string
        - precioUnitario: float
        + cambiarPrecioUnitario(float) void
        + decrementarCantidad(short) void
        + precioPromedioEntre(DateTime, DateTime) float
    }

    class HistorialPrecio {
        - idProducto: short
        - fechaHora: DateTime
        - precioUnitario: float
        - entre(DateTime, DateTime) bool
    }

    class Cajero {
        - dni: int
        - apellido: string
        - nombre: string
        - password: string
        + NombreCompleto() string
    }
    
    class Ticket {
        - idTicket: int
        - cajero: Cajero
        - confirmado: bool
        - fechaHora: DateTime
        - items: List~Item~
        + agregarItem(Item) void
        + confirmar() void
        + totalTicket() float
    }

    class Item {
        - cantidad: short
        - id: int
        - precioUnitario: float
        - producto: Producto
        - ticket: Ticket
        + decrementarProducto() void
        + totalItem() float
    }

    Producto o-- "1" Categoria
    Producto *-- "1..n" HistorialPrecio
    Ticket "1" *-- "*" Item
    Ticket "1" -- "1" Cajero
    Item  o-- "1" Producto
    Ticket *-- "1..n" Item
```

Podemos observar que va a ser importante para nuestra aplicación, manipular todo lo relacionado con el producto, ya sea para traer información detallada o crear productos nuevos. Para el caso de las Categorias por ejemplo, si bien vamos a poder crearlas y leerlas, vemos que su uso va a ser principalmente por y para los productos, por lo que podemos deducir que es una entidad **que solo va a ser utilizada en el contexto del Producto** por lo que podemos incluir todo lo referente a su lectura y carga, en el repositorio de Producto.

Nuestra interfaz de Producto, luciría asi:

```mermaid
    classDiagram
    direction LR
    class IRepoProducto{
    <<interface>>
    Alta(Producto) void
    Obtener() IEnumerable~Producto~
    Detalle(short idProducto) Producto?
    AltaCategoria(Categoria) void
    ObtenerCategorias() IEnumerable~Categoria~
    }
```

Ahora bien recordemos, que el objetivo de usar la interfaz, es desacoplar nuestra Logica de Negocios de nuestro Acceso a datos, por lo que nuestra clase que implemente la interfaz, tendría esta forma:

```mermaid
    classDiagram
    direction LR
    class RepoProducto{
    - conexion: IDBConnection
    RepoProducto(IDBConnection conexion)
    Alta(Producto) void
    Obtener() IEnumerable~Producto~
    Detalle(short idProducto) Producto?
    AltaCategoria(Categoria) void
    ObtenerCategorias() IEnumerable~Categoria~
    }
    class IRepoProducto{
    <<interface>>
    }
    RepoProducto ..|> IRepoProducto 
```

Podemos ver que la clase concreta, implementa los métodos de la interfaz y posee el estado interno según su implementación; en nuestro caso el atributo `conexion` para poder usar los métodos de Dapper.

[<<-- Indice 📖](../README.md#indice-)