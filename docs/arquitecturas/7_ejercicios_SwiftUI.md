

El ejercicio completo se puntúa con **1 punto**

El objetivo del ejercicio es implementar la *app* del tiempo pero ahora con SwiftUI. Para ahorrar algo de tiempo coge la plantilla de proyecto de la sesión.

Consejos:

- En general, intenta que primero se vea el texto con el estado del tiempo y una vez esté funcionando ocúpate del icono con la imagen, recuerda que este último hay que cargarlo desde una url y la cosa se complica un poco con él.
- Fíjate en la transparencia con el [ejemplo de formulario](https://ottocol.github.io/pddm-traspas/traspas/swiftui/#/19)
	+ Puedes por ejemplo hacer dos secciones, una para el campo de texto donde se escribe la localidad y el botón y otra para mostrar los resultados en forma de texto y en forma gráfica
	+ Los resultados solo los debes mostrar si no están vacíos, mira en la traspa cómo se hace para mostrar componentes de manera condicional
- Para comunicarnos con el servidor usaremos el mismo `TiempoModelo` que en el ejercicio de MVVM. El método para consultar el tiempo es `consultarTiempoActual(localidad:)`. Recuerda que al ser un método asíncrono hay que llamarlo con await.