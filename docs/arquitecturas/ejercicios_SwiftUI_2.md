## App de "mis lugares favoritos" (1 punto)

Implementa una mini-app en la que el usuario puede ver una lista de sitios turísticos o interesantes, al pulsar en el nombre de uno de ellos entrará en una página de detalle, donde además del nombre se ve una breve descripción y se puede marcar el sitio como favorito.

Tendrás que:

1. Implementar el modelo. 
   Como base, puedes definir un struct `Lugar` con las propiedades:
    - `id` (recuerda que SwiftUI lo necesita para listar, y que puedes usar la función `UUID()`)
    - `nombre`
    - `descripcion`
    - `favorito` (booleano)

   El modelo será una clase `LugaresModel` que contendrá un array de lugares. Recuerda hacerlo `@Observable`

2. Como en la app no se pueden añadir lugares, para simplificar introduce unos cuantos al definir `LugarModel`:

```swift
@Observable
class LugaresModel {
    var lista: [Lugar] = [
        Lugar(id: UUID(), nombre: "Castillo de Santa Bárbara", descripcion: "Fortaleza de Alicante con vistas al mar."),
        //define unos pocos más...
    ]
}
```

3. La Lista (Pantalla 1):

 - Usa un NavigationStack que contenga una List.
 - Cada fila debe mostrar el nombre del lugar y un corazón rojo relleno si el sitio es favorito. El corazón lo puedes mostrar con un SF Symbol (`Image(systemName:)`) que sea "heart.fill" ("heart" es el mismo pero sin rellenar)
 - En el NavigationLink puedes enviar el `id` y en el NavigationDestination buscar el `Lugar` correspondiente a ese `id` y mostrar la vista detalle pasándole un *binding* al objeto, como en el ejemplo de la lista de la compra.

4. El Detalle (Pantalla 2):

  - Crea una vista que reciba el lugar usando `@Bindable`
  - Muestra la descripción y un Button para conmutar el estado de favorito.
