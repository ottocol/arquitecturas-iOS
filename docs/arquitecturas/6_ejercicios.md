

Queremos hacer una aplicación con arquitectura MVVM para consultar el tiempo meteorológico. El proyecto con la plantilla está en moodle. Ya tiene creadas las "carpetas" y los esqueletos de código para la vista, el modelo y el *viewmodel*. 

De hecho el modelo ya está terminado y no es necesario que modifiques nada de `TiempoModel.swift`, aunque sí tendrás que completar la vista (`MainView.swift`) y el *viewmodel* (`ViewModel.swift`)


## Ensamblaje de vista, modelo y *viewmodel*

En la clase de la vista (`MainView`) añadir una propiedad que represente al *viewmodel*

```swift
let viewModel = TiempoViewModel()
```

En el `TiempoViewModel` añadir una propiedad que represente al modelo

```swift
let modelo = TiempoModelo()
```

## Mostrar la descripción del tiempo (0,5 puntos)

En este apartado conseguiremos que al pulsar en el botón "consultar tiempo" la descripción en modo texto (p.ej. "sol") aparezca en la pantalla del dispositivo.

En el *viewmodel*: `TiempoViewModel`

- Añade al comienzo un `import Combine`
- Crea la propiedad que representa el estado actual del tiempo, `estado` de tipo String, asignándole como valor inicial la cadena vacía
- Anota la propiedad con `@Published` para que se publiquen los cambios
- crea el siguiente método, que llamará al modelo para consultar el tiempo actual

```swift
func consultarTiempoActual(localidad : String) {
  modelo.consultarTiempoActual(localidad: localidad) {
    estado, urlIcono in
    OperationQueue.main.addOperation {
        //como al tocar estas propiedades se va a modificar la interfaz
        //debemos hacerlo desde el hilo principal
        self.estado = estado
        //de momento no hacemos nada con `urlIcono`
    }
  }
}
```

Fíjate que esta función le "pasa la pelota al modelo". Al ser una operación asíncrona, tras obtener la información, el modelo ejecuta la clausura que se le pasa como parámetro. Aquí lo que hacemos es copiar el texto del estado en la propiedad del *viewmodel* del mismo nombre.

En la vista: `MainView`

- Añade al comienzo un `import Combine`
- En el `viewDidLoad()` vincula la propiedad `estado` del *viewmodel* con la propiedad `text` del label `estadoLabel`
- Ten en cuenta que debes guardar esta vinculación en alguna variable que sea una propiedad del *viewModel* para que no se "pierda" y se mantenga "activa"
- Haz que en el método `botonPulsado` se llame a `consultarTiempoActual(localidad:)` del *viewmodel*, pasándole como parámetro lo que haya escrito el el campo de texto `campoTexto`.

Si todo es correcto, al probar la *app*, escribir una localidad en el campo de texto y pulsar sobre el botón de "Ver tiempo" debería aparecer la descripción con el estado actual del tiempo en la pantalla del dispositivo.

## Mostrar el icono del tiempo (0,5 puntos)

Haz lo mismo para el icono del tiempo. Intenta hacer todos los pasos por tí mismo, serán similares al apartado anterior, solo que será más complicado ya que queremos vincular lo que en origen es un String con la URL a la que hay que acceder para obtener el icono del tiempo. 

Tendrás que hacer las siguientes transformaciones:

- Filtrar que la URL del icono sea distinta a la cadena vacía. (operador `filter`)
- Con el operador `map` convertir el String con la URL del icono a una `UIImage`. Tienes que hacerlo en varios pasos: String -> objeto de la clase URL -> datos binarios -> imagen

```swift
let url = URL(string: lo_que_sea_la_cadena_con_la_URL)
let datos = try! Data(contentsOf: url!)
return UIImage(data: datos)
```



