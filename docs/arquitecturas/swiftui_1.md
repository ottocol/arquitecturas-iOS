# SwiftUI: Introducción y Cambio de Paradigma

## 1. Contexto: los orígenes y evolución de SwiftUI

SwiftUI fue presentado en la **WWDC 2019,** junto con iOS 13. Surgió como respuesta de Apple a la tendencia de la industria hacia los frameworks de UI declarativos como React o Flutter. 

!!! info "React y Flutter"
    React es un framework para el desarrollo web *frontend* y Flutter pretende cubrir el espectro *completo* de apps (web, escritorio, móviles). No son comparables son SwiftUI en el sentido de que no son *frameworks* nativos iOS pero sí inspiraron bastantes ideas en las que se basa SwiftUI (sobre todo la idea de vista como una función del estado de la app).
</aside>

SwiftUI no es únicamente una biblioteca de componentes de usuario, la filosofía de programación es muy distinta a la de UIKit y mucho más parecida por ejemplo a la de Jetpack Compose en Android.

### Estado Actual

Tras varios años de iteración agresiva, SwiftUI ha dejado de ser una "beta técnica":

- **2019-2021 (iOS 13-14):** Prometedor pero incompleto (faltaban componentes críticos como `CollectionView` o una navegación estable).
- **2022-2023 (iOS 15-16):** Madurez. Se introducen `NavigationStack` y protocolos de layout avanzados.
- **Actualidad:** Es la opción **por defecto** recomendada por Apple para nuevas Apps.  UIKit sigue siendo necesario para algunas cuestiones como mantener aplicaciones “legacy” o usar APIs muy específicas de bajo nivel.

### El alcance de este curso: iOS 17 / Xcode 15.2

En estas sesiones trabajaremos sobre el SDK de **iOS 17** (Swift 5.9). Esto es importante porque en esta versión hubo un cambio de arquitectura relevante:

1. **Framework `Observation`:** Usaremos las nuevas macros (`@Observable`) en lugar del antiguo protocolo `ObservableObject` y `@Published` (que veréis en muchos tutoriales antiguos).
2. **`NavigationStack`:** Usaremos la navegación basada en datos, desechando el obsoleto `NavigationView`.
3. **Macros:** Uso extensivo de la nueva sintaxis de previsualización `#Preview`.

> Si buscáis documentación online, filtrad resultados posteriores a Junio de 2023. Las soluciones de SwiftUI de 2019-2022 suelen ser sintácticamente diferentes u obsoletas.
> 

## 2. UIKit vs SwiftUI

- En UIKit (**Imperativo**), el programador es responsable de modificar las vistas manualmente cuando cambian los datos. Esto a menudo lleva a estados inconsistentes.
- En SwiftUI (**Declarativo**), la vista es una función pura del estado.

!!! Warning "Importante"  
    En SwiftUI no actualizamos las vistas, actualizamos el estado (`@State`). El framework se encarga de repintar la UI automáticamente. Es similar a Jetpack Compose, o si tienes experiencia en desarrollo web, puedes compararlo a lo que sucede con *frameworks* de *frontend* como React o Vue
> 


| **Concepto** | **UIKit** | **SwiftUI** |
| --- | --- | --- |
| **Componente Base** | `UIViewController` (Class) | `View` (Struct) |
| **Diseño UI** | Storyboard / XIB / AutoLayout | Código Declarativo (Stacks) |
| **Comunicación** | Delegates / Target-Action | Bindings / Closures |
| **Gestión de Memoria** | Referencias persistentes (Heap) | Tipos de valor desechables (Stack) |

---

## 3. Estructura del Proyecto: El Ciclo de Vida


En UIKit teníamos `AppDelegate` y `SceneDelegate`. En SwiftUI puro, el punto de entrada se simplifica drásticamente gracias al protocolo **`App`**.

```swift
@main // 1. Punto de entrada (equivalente al main.m o @UIApplicationMain)
struct MiApp: App { // 2. Protocolo App
    var body: some Scene {
        WindowGroup { // 3. Escena
            ContentView() // 4. Vista Raíz
        }
    }
}
```

1. **`@main`**: Indica que este struct contiene la función estática principal que arranca el ejecutable.
2. **`App` Protocol**: Sustituye al `UIApplicationDelegate`. Gestiona el ciclo de vida de la aplicación (active, background, inactive).
3. **`WindowGroup`**: Es una **Escena**. En iOS gestiona la ventana principal. En iPadOS o macOS, gestiona automáticamente múltiples ventanas si el usuario las crea.
4. **`ContentView`**: Es la vista raíz de tu jerarquía.

> Nota: Si necesitas el antiguo AppDelegate (por ejemplo, para notificaciones Push con Firebase), puedes inyectarlo usando el adaptador @UIApplicationDelegateAdaptor.
> 

## 4. Anatomía de una Vista

En SwiftUI, las vistas son estructuras (`structs`), no clases. Son extremadamente ligeras y se crean/destruyen constantemente, lo veremos con más detalle cuando hablemos del estado de una app

Una vista mínima en SwiftUI podría ser algo así (la plantilla de proyecto de Xcode es algo más complicada porque incluye no solo un texto sino una imagen y eso, como veremos, obliga a agruparlos)

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello, world!")
    }
}
```

1. **`struct`**: Las vistas son tipos por valor, lo que los hace ligeros y desechables. No son objetos vivos en memoria como un `UIView`.
2. **`View`**: Es un protocolo.
3. **`body`**: Es una propiedad calculada. SwiftUI la llama cuando necesita pintar la vista. En un momento explicaremos el uso de `some`.

SwiftUI usa más "magia" de la que parece a primera vista si no te fijas mucho en el código, por ejemplo:

```swift
struct ContentView: View {
    var cargando = false
    var body: some View {
        Text("Ejemplo")
        Text("SwiftUI es mágico")
        Button("Saludar", action: {print("hola")})
    }
}
```

Intuitivamente parece claro que queremos que nuestra vista sean dos campos de texto estáticos y un botón, pero si miramos el código desde el punto de vista de swift "puro", no tiene sentido: ¿una propiedad calculada que devuelve 3 valores uno al lado del otro?. Implícitamente SwiftUI está añadiendo código que combina los 3 objetos en una vista (si quieres más detalles busca información sobre la anotación `@ViewBuilder`, que es la que hace esta tarea y que `body` lleva implícita).

### ¿Qué es `some`?

Verás que el `body` siempre se declara como: `var body: some View`.
¿Por qué no devolvemos `Text` explícitamente? **`some View`** define un **Tipo de Resultado Opaco** (Opaque Result Type). Eso quiere decir que el compilador conoce el tipo exacto, pero el desarrollador no.

Swift es de tipado estático fuerte. En nuestro ejemplo el tipo de la vista es sencillo, pero una vista real en SwiftUI tiene un tipo, digamos, “complicadillo”:

```swift
// El tipo real de tu vista podría ser algo así:
VStack<TupleView<(Text, Button<Text>, Image)>>
```

Escribir eso es tedioso y frágil. Si cambiáramos el `Text` por un `Label`, tendríamos que cambiar la firma de la función. Al usar `some View`, le decimos al compilador:

> "Te prometo que voy a devolver UN tipo concreto que conforma al protocolo View, pero no te voy a decir cuál es en la firma pública. Tú calcúlalo por dentro".

### La *preview*

Por defecto en la parte derecha de Xcode aparece una previsualización de la vista sin necesidad de ejecutar el código en el simulador. Podemos ver que se actualiza en tiempo real conforme vamos tecleando y cambiando cosas. Esta previsualización aparece gracias al código que hay después de la definición de la vista:

```swift
#Preview {
    ContentView()
}
```

Ese `#preview` es una macro de Swift, lo que quiere decir que se expande a un código real algo más complicado pero que no nos hace falta conocer con detalle.

!!! info "Como curiosidad"
    La preview no forma parte del "ejecutable" generado por el compilador, solo le sirve a Xcode

La *preview* es bastante flexible, ya que podemos *personalizarla*. Por ejemplo podemos mostrar cómo quedaría la vista en modo claro y modo oscuro:

```swift
#Preview("Claro") {
    ContentView()
        .preferredColorScheme(.light)
}

#Preview("Oscuro") {
    ContentView()
        .preferredColorScheme(.dark)
}
```
si hacemos esto, en la parte derecha de la ventana de Xcode aparecerán dos previews con las etiquetas que hemos puesto y podremos seleccionar la que queramos.

Si tenemos una vista con propiedades (recordemos que no es más que un `struct` y por tanto puede tener campos), podemos pasarle fácilmente distintos valores a la *preview* para ver cómo quedaría la vista en cada caso, por ejemplo supongamos esta vista:

```swift
struct ContentView: View {
    let cargando: Bool

    var body: some View {
        if cargando {
            Text("Cargando…")
        } else {
            Image(systemName: "photo")
        }
    }
}
```

podemos generar distintas *previews* con distintos valores de la propiedad `cargando`

```swift
#Preview("Cargando") {
    ContentView(cargando: true)
}

#Preview("Contenido") {
    ContentView(cargando: false)
}
```

### Vistas y subvistas

SwiftUI se puede considerar un framework de UI orientado a componentes, donde estos son o bien primitivas de SwiftUI (como `Text` o `Image`) o bien componentes definidos por el desarrollador (como `ContentView`). Hasta ahora los componentes definidos por nosotros solamente contenían primitivas, pero nada nos impide crear nuestros propios subcomponentes (o subvistas, como las queramos llamar):



## 5. Layout Declarativo (Stacks)

No existen las *Constraints* de AutoLayout. Maquetamos usando pilas (**Stacks**, similares a `UIStackView` de `UIKit`) y espaciadores (**Spacers**). El contenedor padre propone un tamaño, y la vista hija decide cuánto ocupar.

### Componentes Principales

- **`VStack`**: Pila Vertical.
- **`HStack`**: Pila Horizontal.
- **`Spacer`**: Un "muelle" flexible que ocupa todo el espacio disponible, empujando el contenido.
- **`.padding()`**: Añade espacio alrededor de la vista.

```swift
HStack { 
    Image(systemName: "globe") // SF Symbol
    
    VStack(alignment: .leading) { 
        Text("Perfil de Usuario")
            .font(.headline)
        Text("iOS Developer")
            .foregroundStyle(.secondary)
    }
    
    Spacer() // Empuja todo el contenido hacia la izquierda
}
.padding() // Margen externo
```

!!! Note "Ver el espacio ocupado por los componentes"
    Al ser el fondo por defecto transparente no se suele apreciar bien dónde empieza y acaba cada componente. Como truco puedes añadir un `.background()` con un color como parámetro. Por ejemplo, para ver el espacio ocupado por el `HStack`:

    ```swift
    HStack { 
        //resto del código...
    }
    .background(.green)
    .padding()
    ``` 

### Más sobre el layout

En SwiftUI, los métodos como `.padding()` o `.background()` no modifican la propiedad de un objeto (como en UIKit). En su lugar, **envuelven** la vista original en una nueva vista modificada.

**⚠️ El orden importa:**

```swift
// Caso A: El fondo envuelve al padding (Texto con margen coloreado)
Text("Botón").padding().background(.blue)

// Caso B: El padding envuelve al fondo (Texto con fondo pegado y margen blanco fuera)
Text("Botón").background(.blue).padding()
```

Tienes imágenes predefinidas, listas para usar, al igual que en `UIkit`, los **SF Symbols:**. Se usan como `Image(systemName: "nombre")` y se tratan como texto (acepta `.font` y `.foregroundStyle`).


## 6. La vista como una función del estado

Una de las partes más interesantes de SwiftUI es que, como ya hemos dicho anteriormente, **la vista es una función del estado de la aplicación**, lo que elimina los problemas de inconsistencia que trae la actualización manual de la UI.

En SwiftUI la vista se repinta cada vez que cambia el estado. Esto pasa porque la propiedad `body` se reevalúa. Vamos a ver un ejemplo:

```swift
struct ContentView: View {
    @Environment(\.colorScheme) var esquema

    var body: some View {
        Text("Modo actual: \(esquema == .dark ? "Oscuro" : "Claro")")
    }
}
```

En este ejemplo introducimos en la vista una especie de "estado o variable externa" que es el esquema de color activo en el dispositivo actualmente (modo oscuro o claro, representado con los enumerados `.dark` o `.light`). Si vamos al simulador (no vale la *preview*) y cambiamos el modo (`Features` > `Toggle Appearance`) veremos cómo cambia el mensaje del componente `Text` porque la vista se está repintando.

### Definir el estado con `@State`

En lugar de que el estado venga del entorno lo más habitual es definirlo en la propia vista anotando una variable con la anotación `@State`. Aquí tenemos el clásico ejemplo del contador, que viene a ser el "hola mundo" de todos los frameworks reactivos como SwiftUI (es de los primeros ejemplos que se suele poner en React, Angular, Vue, ...)

```swift
struct ContentView: View {
    @State private var valor = 0 // <--- La "magia"

    var body: some View {
        VStack {
            Text("Contador: \(valor)")
            Button("Incrementar") {
                valor += 1
            }
        }
    }
}
```
Cada vez que pulses el botón, se incrementará la variable `valor`. Como la variable pertenece al estado de la vista el cambio forzará a que se repinte.

!!! warning "A ver a ver, un momento..."

    ¿Pero en Swift los `struct` no son por defecto inmutables? ¿Qué está pasando aquí, como es posible que la variable dentro del `struct` esté cambiando su valor? En efecto, como los structs son inmutables , necesitamos un lugar externo para guardar los datos que cambian, y eso es lo que hace la anotación `@State`, le dice a SwiftUI: *"Guarda esta variable en una memoria persistente gestionada por ti, fuera del struct"*. Como puedes comprobar, si quitas la anotación `@State` de la variable, verás un error del compilador indicando que el struct es inmutable.

### El ciclo de repintado

Al igual que en otros frameworks declarativos, SiwftUI no repinta todos los componentes de la vista, solo aquellos que deberían cambiar. Por ejemplo:

```swift
struct ContentView: View {
    @State private var contador = 0

    var body: some View {
        VStack(spacing: 20) {
            TextoFijo()
            TextoContador(contador: contador)

            Button("Incrementar") {
                contador += 1
            }
        }
    }
}

struct TextoFijo: View {
    var body: some View {
        let _ = Self._printChanges()
        Text("Yo no dependo del contador")
    }
}

struct TextoContador: View {
    let contador: Int

    var body: some View {
        let _ =  Self._printChanges()
        Text("Contador: \(contador)")
    }
}
```

Aquí hemos separado los dos componentes en sus propias mini-vistas simplemente para poder llevar la pista de qué cambia y qué no, eso lo hacen las líneas `let _ =  Self._printChanges()` (en realidad solo haría falta el `Self._printChanges`, pero no es una instrucción que podamos meter dentro del `body` así que nos vemos obligados a asignarla a una variable cualquiera).

Si ejecutas la aplicación en el simulador (NO en la *preview*) verás que al inicio cambian los dos componentes pero cada vez que pulsamos el vorón solo cambia el componente `TextoContador`. Swift solo repinta lo que cambia para ser lo más eficiente posible.

### Creación y destrucción de vistas

Fíjate en el siguiente ejemplo en el que una vista "madre" contiene una vista hija de tipo Contador, con estado, pero que no existe siempre, sino solo si un *toggle* en la vista madre está activo:

```swift
struct ContadorView: View {

    @State private var contador = 0

    init() {
        print("ContadorView init")
    }

    var body: some View {
        let _ = print("ContadorView body")

        VStack(spacing: 8) {
            Text("Contador: \(contador)")
            Button("Incrementar") {
                contador += 1
            }
        }
        .padding()
        .border(.blue)
    }
}

struct MadreView: View {

    @State private var mostrarContador = false

    var body: some View {
        let _ = print("MadreView body")

        VStack(spacing: 16) {

            Toggle("Mostrar contador", isOn: $mostrarContador)

            if mostrarContador {
                ContadorView()
            }
        }
        .padding()
    }
}
```
Si ejecutas el código verás varias cosas:

- Cada vez que SwiftUI tiene que repintar una vista se reevalúa el `body`
- Cuando dejas de mostrar la vista hija que hace de contador, la vista hija se destruye junto con con su estado asociado. Cuando la volvemos a mostrar se vuelve a crear (se ve el print del init) y el contador comienza otra vez de cero. Esto indica que en SwiftUI los structs que escribimos no son las vistas "vivas" sino *descripciones de cómo construir una vista*. SwiftUI decide cuándo crear/destruir las vistas "vivas".


### `@State` y los tipos por referencia

`@State` funciona directamente solo con los tipos por valor, como `Int`, `Bool`, ... o incluso `struct` pero no con las clases, el siguiente ejemplo, en el que encapsulamos el estado y la lógica de un contador en una clase propia no funcionará correctamente:

```swift
class Contador {
    var valor: Int
    var maximo: Int
    
    init(valor: Int, maximo: Int) {
        self.valor = valor
        self.maximo = maximo
    }
    
    func inc() {
        if (valor<maximo) {
            self.valor += 1
        }
    }
    
}

struct ContentView : View {
    @State private var contador = Contador(valor: 0, maximo: 10)
    var body: some View {
        Text("\(contador.valor)")
        Button("Incrementar", action:{
            contador.inc()
        })
    }
}
```

Si lo probáis veréis que el contador no se incrementa, porque `@State` está diseñado en principio para tipos por valor, SwiftUI solo "reacciona" al cambio si asigamos un nuevo valor a la variable marcada con `@State`, pero en este caso no estamos asignando un nuevo valor, sino llamando a un método que cambia una propiedad *dentro del objeto*.

A partir de iOS 17 podemos conseguir que el ejemplo anterior funcione simplemente precediendo la clase `Contador` de la macro `@Observable`. Podéis probarla y comprobar que el contador ya funciona correctamente, pero ya explicaremos su uso con más detalle en la siguiente sesión. En versiones anteriores, el mismo efecto se podía conseguir indicando que la clase implementa el protocolo `ObservedObject` y marcando la propiedad cuyos cambios queremos detectar (en nuestro caso `valor` con la anotación `@Published`).

## 7. Comunicación Padre-Hijo (`@Binding`)

Si necesitamos que una vista hija modifique el estado de su padre, no podemos pasarle una copia del valor. Necesitamos pasarle una referencia de escritura.

- **Sin `$`**: Pasas el valor (Lectura).
- **Con `$`**: Pasas el Binding (Lectura y Escritura).

```swift
// VISTA PADRE
struct ParentView: View {
    @State private var isOn = false
    
    var body: some View {
        let valor = isOn ? "activo" : "inactivo"
        Text(valor)
        // Pasamos el binding con $
        ChildView(isOn: $isOn)
    }
}

// VISTA HIJA
struct ChildView: View {
    @Binding var isOn: Bool // Recibe referencia
    
    var body: some View {
        Toggle("Activar", isOn: $isOn) // Modifica el estado del padre
    }
}
```

Tened en cuenta que en SwiftUI las vistas "del sistema" como Text o Toggle también son vistas hijas de la actual, de modo que el ejemplo anterior también requeriría un *binding* aunque solamente hubiera una vista propia:

```swift
struct ToggleView: View {
    @State private var isOn = false
    
    var body: some View {
        let valor = isOn ? "activo" : "inactivo"
        Text(valor)
        // Pasamos el binding con $
        Toggle("Activar", isOn: $isOn)
    }
}
```

El `Toggle` modifica un estado que no es suyo sino de `ToggleView` y por eso necesita que le pasemos el valor como un *binding*, pero no hace falta definir la variable `isOn` como `@Binding` porque sí pertenece a la vista en la que se define, `ToggleView`, no la recibimos de otra vista.