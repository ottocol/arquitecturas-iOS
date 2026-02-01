

## Perfil en una red social (1 punto)

Crea una vista que represente una especie de "tarjeta" de perfil de un usuario en una red social. La pantalla debe incluir:

1. **Nombre del usuario**
    - Texto fijo (por ejemplo: “Alex”, “María”, etc.)
2. **Número de seguidores**
    - Debe mostrarse en pantalla
    - Debe actualizarse automáticamente al interactuar con el siguiente botón
3. **Botón “Seguir / Dejar de seguir”**
	- Debería estar en su propia vista
    - Al pulsarlo:
        - cambia el estado de seguimiento. Además si estamos siguiendo al usuario debe aparecer en gris, y si no en azul
        - cambia el texto del botón
        - actualiza el número de seguidores
4. **Gestión del estado**
    - Usar `@State` para representar:
        - si el usuario está seguido o no
        - el número de seguidores
5. **Layout**
    - Usar `VStack` y HStack
    - Usar al menos `padding`