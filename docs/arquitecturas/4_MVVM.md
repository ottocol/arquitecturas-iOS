
## MVVM vs. MVP

El patrón de diseño Model/View/ViewModel es muy similar al MVP que vimos en el apartado anterior. 

![](img/mvvm.png)

De hecho, el *ViewModel* tiene más o menos la misma funcionalidad que el *presenter*, implementar la lógica de presentación y aislarla de la tecnología concreta usada para la presentación.

¿Dónde está la diferencia entonces?. En que MVVM soluciona uno de los principales problemas que tiene MVP, el acoplamiento entre vista y *presenter*. Como estuvimos discutiendo, la *vista* y el *presenter* deben "conocerse" mutuamente, ya que la vista debe comunicarle a este las acciones del usuario, y el *presenter* enviarle a la vista los datos a mostrar. Ya hemos visto en el código de ejemplo de MVP que en la vista hay una referencia al *presenter* y viceversa. Eso quiere decir que no es sencillo cambiar la implementación de uno de los dos sin afectar al otro.

En MVVM no hay acoplamiento entre vista y *viewmodel*, y lo vamos a evitar usando *bindings*, es decir, vinculación automática entre los datos del *viewmodel* y la vista, de manera que cuando cambie alguno de ellos se modifique automáticamente la otra. Esto permite que el código quede mucho más "limpio", ya que no hay que actualizar el otro componente de modo explícito.

En iOS podemos usar el *framework* **Combine** para hacer esta vinculación. Combine es un *framework* encuadrado en lo que se conoce comúnmente como "programación funcional reactiva". Si no has oído nunca este término no te preocupes, aquí solo vamos a usar algunas funcionalidades limitadas, y la idea básica no es difícil de entender.

> Combine fue presentado por Apple en la WWDC en junio del 2019. Antes de esto, para usar funcionalidades similares en una aplicación se podían emplear *frameworks* de terceros como [RXSwift](https://github.com/ReactiveX/RxSwift) o [Bond](https://github.com/DeclarativeHub/Bond). Sin entrar en si éstos son mejores o peores que Combine, aquí usaremos este último para evitarnos incluir dependencias de terceros, ya que no necesitamos funcionalidades demasiado complejas. 


## MVVM con Combine

Vamos a verlo con un ejemplo, ya que así se entenderán mejor los conceptos. Implementaremos  una versión MVVM de la aplicación `UAdivino`, al estilo de la que hicimos en el apartado anterior, es decir, mostrando cada tipo de respuesta de un color distinto. La diferencia fundamental va a estar en que vincularemos de modo automático tanto el texto de la respuesta como el color para que no haga falta fijarlos de forma explícita en la vista. Esta vinculación la haremos gracias al *framework* Combine. El código fuente está disponible en el [repositorio de Github](https://github.com/ottocol/UAdivino_MVVM_Combine).


### Vinculación entre *viewmodel* y vista

Al vincular un *origen* con un *destino* lo que hacemos es que cuando el primero cambia, el segundo también lo hace automáticamente. Esto simplifica mucho el código ya que podemos cambiar una propiedad del *viewmodel* y que se refleje automáticamente en la vista, o viceversa.

En nuestro ejemplo del `UAdivino` queremos vincular dos propiedades del *viewmodel* con la vista: el texto de la respuesta y el color de la misma. En Combine los *bindings* se basan en la idea de **Publishers** y **Subscribers**. Esta es una idea similar a la de los eventos. Podemos indicar que queremos suscribirnos a un *publisher* de modo que se nos "avisará" de cuándo éste "emita" su valor. Lo más interesante es que los *publisher* pueden ser simplemente variables, y simplemente cambiar su valor hace que éste se "publique".

Una idea adicional que como veremos es muy potente es que además la información que emiten los *publisher* se puede manipular, transformar y combinar con diversos operadores, permitiéndonos realizar tareas complejas de modo relativamente sencillo. 

> Estas ideas no son exclusivas de Combine, sino que como hemos dicho son las que están tras el paradigma de **programación funcional reactiva**, aunque en este paradigma se suele usar una terminología distinta, hablando a veces de *observables* en lugar de *publishers* o  *streams* en lugar de *events*. Si tenéis interés en el tema podéis consultar [esta fantástica introducción](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754), que aunque no está explicada en el contexto de iOS ni en Swift introduce perfectamente las ideas de la programación funcional reactiva, que son perfectamente aplicables a cualquier lenguaje de desarrollo y cualquier tipo de aplicación.

#### En el *viewmodel*

Así, en el *viewmodel* definiremos el texto de la respuesta y su color como *publishers* , para poder luego vincularlos a la vista. Esto se hace con la anotación `@Published`

```swift
import Combine
...
@Published var textoResp = ""
@Published var colorResp = ColorRespuesta.verde 
```

En el ejemplo anterior hemos dado valores iniciales a los *publishers* para evitar problemas con los valores `nil`.

#### En la vista

En la vista, para vincular el *publisher* con una propiedad de un elemento de `UIKit` usamos el método `assign(to:on:)`, donde con el `to:` especificamos cuál es la propiedad, y con el `on:` cuál es el elemento. 

Por ejemplo, queremos vincular el *publisher* `textoRespuesta` con la propiedad `text` de la etiqueta (`UILabel`) `respuestaLabel` que está en la vista. Eso lo haremos con la siguiente línea:

```swift
viewModel.$textoResp.assign(to: \.text!, on: respuestaLabel),
```

A partir de este momento, cada vez que cambie la variable `textoRespuesta` en el *viewmodel* se actualizará automáticamente el texto de la etiqueta `respuestaLabel`.

Fíjate en que:

- Usamos la notación `$` para indicar que queremos usar el valor "publicado" de `$textoResp`, es decir, que cada vez que cambie la variable original queremos recibir el nuevo valor
- Usamos la notación llamada *keypath* en Swift para expresar la propiedad de `respuestaLabel` a vincular. En esta notación se usa la sintaxis `\.nombre_de_propiedad` y si hubiera "subpropiedades" se seguirían concatenendo con puntos. Esta notación permite comprobar en tiempo de compilación que la propiedad es correcta, en lugar de poner una cadena que habría que evaluar en tiempo de ejecución. Finalmente hemos tenido que poner el `!` para desempaquetar el opcional, ya que el `text` de un `UILabel` es un opcional.

Finalmente nótese que en debemos almacenar la vinculación o *binding* en una variable miembro de la clase `UAdivinoView`. Esto lo hacemos para que las vinculaciones no se pierdan mientras esta variable esté definida (como la vista está "viva" durante todo el ciclo de vida de la *app* pasará lo mismo con la variable miembro).

```swift
//En el viewmodel
private var bindingRespuesta = AnyCancellable!
...
override func viewDidLoad() {
   super.viewDidLoad()
   bindingRespuesta = viewModel.$textoResp.assign(to: \.text!, on: respuestaLabel)
}


```

#### Transformar los valores de un *publisher*

Nos queda por vincular el *publisher* `colorResp` con la propiedad `textColor` de la etiqueta `respuestaLabel`. Sin embargo aquí tenemos un problema: `colorResp` es de un tipo definido por nosotros, `ColorRespuesta`, mientras que `textColor` es un tipo propio de `UIKit`. Por lo tanto no podemos aplicar directamente el `assign(to:on)`.

Podríamos representar los colores en el *viewmodel* con `UIColor` para evitarnos esta discordancia de tipos, pero recordemos que el *viewmodel* no debería contener código de `UIKit`.

Una idea bastante poderosa de **los publishers** es que los eventos que emiten **se pueden transformar** y además de modo encadenado. Para ello se pueden usar las primitivas típicas de programación funcional (`map`, `filter`,...) y además algunas adicionales (de ahí viene precisamente el apelativo de "funcional" de la programación funcional reactiva). Por tanto lo que haremos será transformar el evento en uno de tipo `UIColor` y vincular el color del *label* con este. En la vista haríamos algo como:

```swift
viewModel.$colorResp.map {
    color in
    color == ColorRespuesta.verde ? UIColor.green : UIColor.red
}
.assign(to: \.textColor!, on: respuestaLabel)
```

Recordemos que la primitiva `map` transforma un valor en otro.

#### Vinculación de vista a *viewmodel*

En el ejemplo de `UAdivino` no hay ningún caso de uso de vinculación en la dirección contraria, es decir, de la vista hacia el *viewmodel*, así que pondremos otro ejemplo. 

Supongamos una *app* con un campo de texto en que el usuario escribe un texto de búsqueda. En la vista tendremos definido un *outlet* que referencie el campo de texto. Nuestro objetivo es detectar cuándo cambia el texto del campo y asignarlo a alguna propiedad del *viewmodel*. Podemos aprovechar que cada vez que cambia el texto se emite una notificación del tipo `textDidChangeNotification`


```swift
//En la vista
let binding : AnyCancellable!

...
func viewDidLoad() {
   binding = NotificationCenter.default.publisher(for: UITextField.textDidChangeNotification, object: textoField)
            //obtenemos el UITextField que ha cambiado pero nos interesa el texto
            .compactMap { ($0.object as? UITextField)?.text }
            //Cambiamos el tipo de evento a "genérico"
            .eraseToAnyPublisher()
            //asignamos el valor observado a una variable que nos interese
            .assign(to: \.texto, on: self.viewModel)
}
```

En el ejemplo estamos suponiendo que podemos acceder al *viewmodel* desde la vista con una variable `viewModel` (ver el apartado siguiente). Entonces asignamos el valor "notificado" a la variable `texto` del *viewmodel*.

> Teóricamente esto se debería poder hacer de forma mucho más fácil simplemente observando los cambios en la propiedad `text` del campo de texto. Por desgracia aunque observar propiedades se puede hacer con muchos objetos, en la versión actual de iOS esta funcionalidad no se implementa con los `UIControl` (y `UITextField`lo es). Hay que decir que en *frameworks* de terceros como RxSwift esto sí es posible.


### "Ensamblaje" de modelo, *viewmodel* y vista

Al igual que en el caso de MVP, hay que conectar las "piezas": modelo, *viewmodel* y vista. Como según el esquema del patrón la vista "posee" al *viewmodel*, definiremos este como una propiedad de la vista:

```swift
class UAdivinoView : UIViewController {
   let viewModel = UAdivinoViewModel()
   ...
} 
```

Y como el *viewmodel* "posee" al modelo definiremos este como una propiedad del primero:

```swift
class UAdivinoViewModel {
   let model = UAdivinoModel(nombre: "Rappel")
   ...
}
```


En MVVM, una parte importante del ensamblaje es **inicializar las vinculaciones o *bindings***. Una forma sencilla de hacer esto es en el `viewDidLoad` de la vista:

```swift
override func viewDidLoad() {
   self.bindViewModel()
}

func bindViewModel() {
    //conectamos viewModel.textoResp -> texto del label
    viewModel.$textoResp.assign(to: \.text!, on: respuestaLabel)
    ...
}
```
