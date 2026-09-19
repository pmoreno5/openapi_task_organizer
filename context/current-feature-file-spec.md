# Especificación del descriptor de feature actual

## Estructura

- El descriptor es @context/current-feature.md.
- El nombre de la feature es el encabezado de primer nivel.
- Contiene una sección `## Objetivos` que puede contener requisitos, requisitos técnicos, criterios de aceptación y otras subsecciones que se crean convenientes.
- Contiene una sección `## Notas` que puede contener notas adicionales.
- Contiene una sección `## Histórico` que contiene una lista de entradas ordenadas de la más reciente a la más antigua.

## Restricciones

El histórico no se eliminará nunca. La única modificación posible es añadir entradas.

## Acciones previstas

- Cuando el usuario pida que se actualice el contenido a partir de la información de alguna fuente con el fin de iniciar el desarrollo de una feature, se actualizará el nombre y las secciones `## Objetivos` y `## Notas`. No se añadirá una entrada a la sección `## Histórico` indicando la actualización de inicio del desarrollo de una feature.
- Cuando el usuario pida que se añada al histórico, se añadirá, preservando el orden previsto, una entrada con un resumen de una línea del trabajo realizado en la sesión.
- Cuando el usuario pida limpiar o reiniciar este descriptor, se cambiará el encabezado de primer nivel a `Feature actual` y se borrará el contenido de las secciones `## Objetivos` y `## Notas`, preservando los encabezados.
