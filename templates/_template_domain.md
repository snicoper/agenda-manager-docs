# Nombre de la Entidad (e.g., `Appointment`, `Calendar`)

## Descripción General
<!-- Explica brevemente qué representa esta entidad o aggregate root en el contexto del dominio.
     Describe el rol y propósito principal de la entidad en el negocio y su función en el sistema. -->

<!-- Aquí es útil pensar en el “¿por qué?” de la entidad. Pregúntate: ¿Por qué existe esta entidad en el sistema? ¿Qué rol juega en el negocio? Incluso una o dos frases concisas pueden ser efectivas, como: -->

> "La entidad `Appointment` gestiona la organización de citas para el sistema, representando las reuniones entre recursos y clientes."

## Responsabilidades
<!-- Lista las responsabilidades clave de la entidad, es decir, las tareas principales que maneja
     en el sistema, como validar ciertos datos, gestionar estados, o colaborar con otras entidades. -->

<!-- Piensa en las acciones clave que realiza la entidad para cumplir con su propósito. Si un método es esencial para que la entidad cumpla su rol, eso suele indicar una responsabilidad. Por ejemplo: -->

- Para una entidad `Calendar`, podrías decir que una responsabilidad es "gestionar las disponibilidades de los recursos".
- En `Appointment`, una responsabilidad podría ser "confirmar la disponibilidad de recursos antes de agendar una cita".

## Invariantes
<!-- Define las reglas que deben cumplirse siempre para que la entidad esté en un estado válido.
     Estas son condiciones que, si no se cumplen, indican un estado inválido del objeto. -->

<!-- Esta parte suele ser difícil, pero es básicamente cualquier regla que deba cumplirse para que la entidad esté en un estado válido. Un buen punto de partida es considerar preguntas como: -->

- En `Calendar`, una invariante podría ser "una fecha de inicio debe ser anterior a la fecha de fin".

## Reglas de Negocio
<!-- Lista las reglas de negocio específicas que gobiernan esta entidad, incluyendo lógica o
     restricciones adicionales que no califican como invariantes pero sí como normas de uso.
     Especifica también cómo se relaciona con otras entidades o recursos del dominio si aplica. -->

<!-- Si las invariantes te parecieron complejas, aquí tienes un poco más de libertad, ya que las reglas de negocio pueden ser normas adicionales que no son absolutas, pero que rigen el uso de la entidad en situaciones específicas. Piensa en reglas como "Condiciones que debe cumplir esta entidad en ciertos contextos". -->

- "Una cita solo puede ser confirmada si todos los recursos necesarios están disponibles en ese horario."

## Relaciones
<!-- Detalla las relaciones con otras entidades o agregados. Explica si son relaciones de tipo
     composición, agregación, asociación, etc. Incluye la cardinalidad y el propósito de cada relación. -->

<!-- Aquí, lo útil es indicar con quién interactúa esta entidad y por qué. No siempre es necesario ir al detalle de si es una relación de tipo *composición* o *asociación*, sino más bien el propósito general. -->

- `Appointment` se relaciona con `Resource` para gestionar la asignación de recursos a las citas.

## Propiedades

## Métodos
<!-- Enumera y describe los métodos públicos principales de la entidad, incluyendo su propósito.
     Esto ayuda a entender qué operaciones se pueden realizar en la entidad y cómo interactúa con otras partes del sistema. -->
- **Método `NombreDelMetodo1`**: Descripción del propósito del método.
- **Método `NombreDelMetodo2`**: Descripción del propósito del método.

## Eventos de Dominio
<!-- Si la entidad genera eventos de dominio, indícalos aquí con una breve explicación de cuándo
     y por qué se generan en el contexto del negocio. -->

- **Evento `NombreDelEvento1`**: Descripción de la situación que lo desencadena y el objetivo del evento.
- **Evento `NombreDelEvento2`**: Descripción de la situación que lo desencadena y el objetivo del evento.

## Interceptores EF Core

## Estado y Transiciones

## Dependencias

## Comentarios adicionales

## Ejemplos de Uso
<!-- Proporciona ejemplos prácticos de cómo interactuar con esta entidad, lo cual facilita
     entender su flujo de uso típico en el sistema. Esto podría incluir código en C# o pseudocódigo. -->
```csharp
var appointment = new Appointment(/* parámetros de inicialización */);
appointment.Schedule(/* parámetros de uso */);
