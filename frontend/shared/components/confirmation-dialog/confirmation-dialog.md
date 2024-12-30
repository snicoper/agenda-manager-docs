# Confirmation Dialog Component

Un componente reutilizable para mostrar diálogos de confirmación usando Angular Material.

## Instalación

1. Asegúrate de tener instalado Angular Material
2. Copia los archivos del componente en tu proyecto:
   - `shared/components/confirmation-dialog/*`

## Uso Básico

```typescript
export class MyComponent {
  private readonly confirmationDialogService = inject(ConfirmationDialogService);

  deleteItem(): void {
    this.confirmationDialogService
      .confirm({
        title: 'Eliminar tipo de recurso',
        message: '¿Estás seguro de que quieres eliminar este tipo de recurso?',
        confirmText: 'Eliminar',
        cancelText: 'Cancelar',
      })
      .pipe(
        filter(Boolean),
        switchMap(() => this.apiService.delete().pipe(take(1))) // Llama a la API para eliminar
      )
      .subscribe({
        next: () => {
          // Manejar éxito
        },
        error: (error) => {
          // Manejar error
        }
      });
  }
}
```

## Configuración

La interfaz `ConfirmationDialogData` acepta las siguientes propiedades:

```typescript
interface ConfirmationDialogData {
  title?: string;      // Título del diálogo (opcional)
  message: string;     // Mensaje principal (requerido)
  confirmText?: string; // Texto del botón confirmar (opcional)
  cancelText?: string;  // Texto del botón cancelar (opcional)
}
```

## Ejemplos

### Confirmación Simple

```typescript
this.confirmationDialogService
  .confirm({
    message: 'Do you want to proceed?'
  })
  .subscribe(result => {
    if (result) {
      // Usuario confirmó
    }
  });
```

### Confirmación con Acción

```typescript
this.confirmationDialogService
  .confirm({
    title: 'Delete Resource',
    message: 'Are you sure you want to delete this resource?',
    confirmText: 'Delete',
    cancelText: 'Cancel'
  })
  .pipe(
    filter(Boolean),
    switchMap(() => this.apiService.deleteResource(id))
  )
  .subscribe({
    next: () => {
      this.snackBar.success('Resource deleted successfully');
      this.router.navigate(['/resources']);
    },
    error: (error: HttpErrorResponse) => {
      this.snackBar.error('Error deleting resource');
    }
  });
```

### Manejo de Errores Específicos

```typescript
this.confirmationDialogService
  .confirm({
    title: 'Delete Resource Type',
    message: 'Are you sure you want to delete this resource type?'
  })
  .pipe(
    filter(Boolean),
    switchMap(() => this.apiService.deleteResourceType(id))
  )
  .subscribe({
    next: () => {
      this.snackBar.success('Resource type deleted successfully');
      this.router.navigate(['/resource-types']);
    },
    error: (error: HttpErrorResponse) => {
      if (error.error === 'CANNOT_DELETE_RESOURCE_TYPE') {
        this.snackBar.error('Cannot delete resource type with associated resources');
        return;
      }
      this.snackBar.error('Error deleting resource type');
    }
  });
```

## Notas

- El diálogo retorna `true` cuando el usuario confirma y `false` cuando cancela
- `filter(Boolean)` se usa para proceder solo cuando el usuario confirma
- El diálogo no se puede cerrar haciendo clic fuera de él (disableClose: true)
- Usa `switchMap` para cancelar operaciones pendientes si se inicia una nueva
