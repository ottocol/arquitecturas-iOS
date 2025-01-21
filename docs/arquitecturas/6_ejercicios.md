

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

### Imprimir el tiempo en la consola

En este apartado conseguiremos que al pulsar en el botón "consultar tiempo" la descripción en modo texto (p.ej. "sol") aparezca en la pantalla del dispositivo.

Primero empezaremos por hacer que la información aparezca en la consola del desarrollador con un `print`.

El modelo (clase `TiempoModelo`) tiene un método `consultarTiempoActual(localidad:)` al que si le pasamos la localidad nos devolverá una tupla con un `estado` ("cielo claro", "nubes", "lluvioso",...) y una URL de un icono que representa gráficamente el tiempo (un sol, una pequeña nube, ...). Estos datos se obtienen de un API externo de un sitio llamado `OpenWeatherMap`.

**En el *viewmodel* (clase `TiempoViewModel`) crea el siguiente método**, que llamará al modelo para consultar el tiempo actual

```swift
func consultarTiempoActual(localidad : String) async {
    let result = try? await modelo.consultarTiempoActual(localidad: localidad)
    if let result = result {
        print(result)
    }
}
```

la llamada al API web que hace el modelo es una operación **asíncrona**, motivo por el cual en Swift se llama precedida de **await** (que significa algo así como "llama a esta operación y espera a que devuelva el resultado"). 

Cuando en una función llamamos a una operación asíncrona, automáticamente se convierte también en asíncrona y el compilador nos exige que la marquemos como tal con **async** (que es lo que pasa en la cabecera de nuestra función). Las funciones que son **async** siempre se deben llamar precedidas de **await**.

> Anteriormente vimos cómo llamar a operaciones asíncronas con colas de operaciones, `async` y `await` son la forma de trabajar con código asíncrono que está integrada en el propio lenguaje Swift desde la versión 5.5, mientras que las colas de operaciones son una biblioteca adicional no incluída en el "núcleo" de Swift.

Falta que al pulsar el botón de "Ver el tiempo" se llame a `consultarTiempoActual` de `TiempoViewModel`. En la vista (clase `MainView`) al pulsar el botón se llama al IBAction `botonPulsado` (¡sorpresa!). Verás que la función contiene `Task {}`. Esto sirve para marcar secciones de código asíncrono, coloca dentro la siguiente línea:

```swift
await viewModel.consultarTiempoActual(localidad:campoTexto.text ?? "")
```

que llama de forma asíncrona al `consultarTiempoActual` del `viewModel`.

De momento, prueba el código tal cual, debería imprimir en la consola el tiempo actual en la ciudad cuyo nombre escribas en el campo de texto, y vacío en caso de que no haya nada escrito o la ciudad no se encuentre.


### Mostrar el tiempo en la interfaz gráfica


En MVVM tenemos que vincular propiedades del viewmodel con componentes de UI de la vista.

En el viewmodel (la clase `TiempoViewModel`)

- Añade al comienzo un `import Combine`
- Crea la propiedad que representa el estado actual del tiempo, `estado` de tipo String, asignándole como valor inicial la cadena vacía
- Anota la propiedad con `@Published` para que Combine publique los cambios
- En la función `consultarTiempoActual`, en lugar de imprimir los datos con `print`, simplemente debes asignarle a la propiedad `estado` el valor obtenido (`self.estado = result.estado`).

Hay una cuestión importante:  vamos a vincular el cambio de la propiedad `self.estado` con un componente de UI. Recordemos de otras veces que la actualización de la interfaz se debe hacer en el hilo principal de la aplicación. Por tanto la línea `self.estado = result.estado` se debe ejecutar en el hilo principal. Hay dos formas de asegurarlo, la primera es con colas de operaciones:

```swift
OperationQueue.main.addOperation {
    self.estado = result.estado
}
```

La segunda es "más moderna", usando el soporte de concurrencia integrado en Swift. El hilo principal es `MainActor`:

```swift
await MainActor.run {
    self.estado = result.estado
}
```

Elige la forma que prefieras, ambas deberían ser equivalentes.

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



