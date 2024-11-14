- **Proceso de Desactivación/Activación**:
  - Un recurso puede ser desactivado en cualquier momento
  - Al desactivar, se debe proporcionar un motivo que quedará registrado
  - Al reactivar, se limpia el motivo de desactivación
  - La desactivación afecta a las citas existentes de la siguiente manera:
    - Citas `Completed`: No se ven afectadas
    - Citas `InProgress`: Pueden completarse normalmente
    - Citas `Pending`, `Accepted` y `Waiting`: Pasan a estado `RequiresRescheduling`
  - La desactivación debe generar notificaciones para:
    - Usuarios afectados por citas que requieren reprogramación
    - Administradores del sistema
  - Un recurso desactivado no puede aceptar nuevas citas
  - Un recurso puede ser reactivado si no ha sido eliminado

En la parte `- La desactivación debe generar notificaciones para:`, da un poco la sensación de que sea responsabilidad de que el agregado es el responsable de notificar a los usuarios y administradores.

No se si me explico, quizá su responsabilidad sea emitir el evento, aunque claro esto no se yo si deja cojo la regla (ya queda como ambiguo)
