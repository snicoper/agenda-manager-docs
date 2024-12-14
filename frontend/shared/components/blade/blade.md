# Documentación del Componente Blade

## Índice

1. [Instalación](#instalación)
2. [Uso Básico](#uso-básico)
3. [API](#api)
4. [Ejemplos](#ejemplos)
5. [Mejores Prácticas](#mejores-prácticas)

## Instalación

### Estructura de Archivos

```shell
shared/
  components/
    interfaces/
      blade-options.interface.ts
    services/
      blade.service.ts
    blade/
      blade.component.ts
      blade.component.html
      blade.component.scss
```

## Uso Básico

### Crear un Componente Blade

```typescript
@Component({
  selector: 'am-create-role-blade',
  template: `
    <div class="blade-content-container">
      <h2 class="blade-title">Título</h2>
      <div class="blade-body">
        <!-- Contenido -->
      </div>
      <div class="blade-actions">
        <!-- Acciones -->
      </div>
      <div class="blade-loading">
        <!-- Loading -->
      </div>
    </div>
  `
})
export class CreateRoleBladeComponent {
  private readonly bladeService = inject(BladeService);

  handleCloseBlade(): void {
    this.bladeService.hide();
  }
}
```

### Mostrar el Blade

```typescript
private readonly bladeService = inject(BladeService);

openBlade() {
  this.bladeService.show(CreateRoleBladeComponent, {
    width: '500px'
  });
}
```

### Manejar Resultados

```typescript
constructor() {
  this.bladeService.result$
    .pipe(takeUntilDestroyed())
    .subscribe(result => {
      if (result) {
        this.loadData();
      }
    });
}
```

## API

### BladeService

#### Métodos

- `show<TData>(component: Type<unknown>, options?: BladeOptions<TData>)`: Muestra el blade
- `hide()`: Oculta el blade
- `emitResult(result: unknown)`: Emite un resultado (llama a `hide()`)

#### Propiedades

- `result`: Observable que emite cuando hay un resultado del blade

### Clases CSS Útiles

Ver: `src\styles\components\_blade.scss`

```scss
.blade-content-container {
  // Contenedor principal del contenido.
}

.blade-title {
  // Título del blade.
}

.blade-body {
  // Contenido principal.
}

.blade-actions {
  // Contenedor de acciones (botones).
}

.blade-loading {
  // Cuando esta en loading los posibles llamadas a la API.
}
```

## Ejemplos

### Formulario Simple

```typescript
@Component({
  selector: 'am-create-role-blade',
  template: `
    <div class="blade-content-container">
      <h2 class="blade-title">Crear Rol</h2>

      <div class="blade-body">
        <form [formGroup]="form">
          <mat-form-field>
            <input matInput placeholder="Nombre" formControlName="name">
          </mat-form-field>
        </form>
      </div>

      <div class="blade-actions">
        <button mat-button (click)="handleCloseBlade()">Cancelar</button>
        <button mat-raised-button (click)="handleSubmit()">
          Guardar
        </button>
      </div>
    </div>
  `
})
export class CreateRoleBladeComponent {
  private readonly bladeService = inject(BladeService);

  handleSubmit() {
    // Lógica de guardado.
    this.bladeService.emitResult(true);
  }

  handleCloseBlade() {
    this.bladeService.hide();
  }
}
```

### Pasar y Recibir Datos

Interfaz:

```typescript
interface BladeOptions<TData = unknown> {
  width?: string;    // Ancho del blade (default: '480px')
  data?: TData;      // Datos que se quieren pasar al blade
}
```

```typescript
// Al abrir el blade.
this.bladeService.show<EditRoleDataBlade>(EditRoleBladeComponent, {
  data: { id: '1', name: 'Admin' }
});

// En el componente blade.
export interface EditRoleDataBlade {
  id: string;
  name: string;
}

@Component({...})
export class EditRoleBladeComponent {
  // Acceder a la data:
  this.bladeService.bladeState().options.data?.id;
}
```

## Mejores Prácticas

### Convenciones de Nombres

- Usar el sufijo `-blade` en los nombres de componentes
- Ejemplo: `create-role-blade.component.ts`

### Estructura del Contenido

- Usar siempre `blade-content-container` como contenedor principal
- Dividir el contenido en título, cuerpo y acciones
- Mantener las acciones al final del blade

### Tips

1. No sobrecargar el blade con demasiado contenido
2. Usar el ancho apropiado según el contenido
3. Mantener la jerarquía visual con títulos claros
4. Proporcionar siempre una forma de cerrar/cancelar
5. Limpiar las suscripciones usando `takeUntilDestroyed()`

### Consideraciones de Rendimiento

- No cargar datos innecesarios
- Usar `OnPush` change detection cuando sea posible
- Limpiar recursos al cerrar el blade

### Cuándo Usar un Blade

- Formularios de creación/edición
- Paneles de detalles
- Configuraciones
- Cualquier contenido que no requiera navegación completa
