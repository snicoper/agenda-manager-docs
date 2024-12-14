# Guía de uso: Form Facade y Patrones de Formularios

## ¿Qué es?

Una solución que simplifica la creación y gestión de formularios en Angular, combinando lo mejor de Reactive Forms con un manejo de estado unificado.

## ¿Cuándo usarlo?

- Para formularios que siguen patrones comunes (login, crear/editar recursos, etc.)
- Cuando quieres reducir código repetitivo
- Cuando necesitas un manejo consistente de estados y errores
- En formularios que requieren validaciones estándar

## ¿Cuándo NO usarlo?

- Formularios muy específicos o únicos
- Casos que requieren lógica muy personalizada
- Formularios simples donde añadiría complejidad innecesaria

## Ejemplo básico

```typescript
readonly loginForm = this.formFacade.createForm<LoginForm>({
  fields: {
    email: {
      value: '',
      validators: [Validators.required, CustomValidators.email],
      component: {
        type: 'email',
        label: 'Email',
        icon: 'mail'
      }
    }
  },
  onSuccess: () => {
    // Manejar éxito
  },
  onError: (error) => {
    // Manejar error
  }
});
```

## ¿Qué nos da?

### 1. Gestión de Estado

- Estado de carga (`isLoading`)
- Estado de validación (`isValid`)
- Errores del backend (`badRequest`)
- Estado del formulario (`isSubmitted`)

### 2. Métodos Útiles

- `submit()`: Manejo unificado de envío
- `reset()`: Reinicio del formulario
- `getFieldValue()`: Obtener valores
- `setFieldValue()`: Establecer valores
- `getFieldConfig()`: Obtener configuración

### 3. Integración con Componentes

```html
<am-form-input
  [formState]="loginForm.state()"
  [formInputType]="formInputType.Email"
  fieldName="email"
  formControlName="email"
  [label]="loginForm.getFieldConfig('email').label"
  [icon]="loginForm.getFieldConfig('email').icon"
/>
```

## Ventajas

1. Menos código repetitivo
2. Manejo consistente de errores
3. Tipado fuerte
4. Fácil de mantener y testear
5. API unificada y clara

## Consejos de Uso

1. Define bien tus interfaces de formulario
2. Utiliza los componentes wrapper existentes (`am-form-input`, etc.)
3. Aprovecha la configuración por campo para metadata
4. Mantén la lógica específica en los componentes
5. Usa los helpers proporcionados en lugar de acceder directamente al FormGroup

## Flexibilidad

Recuerda que siempre puedes volver al enfoque tradicional cuando sea necesario:

```typescript
// Enfoque tradicional cuando sea más apropiado
formState: FormState = {
  form: this.formBuilder.group({}),
  badRequest: undefined,
  isSubmitted: false,
  isLoading: false
};
```

## Migración

- Migra formularios gradualmente
- Empieza por los más simples y repetitivos
- Mantén la consistencia en cada feature
- Documenta las decisiones de cuándo usar cada enfoque

## Ejemplos de Patrones

### Wrapper Pattern (am-form-input)

Simplifica la interfaz de componentes complejos:

```html
<!-- Wrapper que simplifica el uso -->
<am-form-input
  [formState]="formState"
  [label]="label"
  [icon]="icon"
/>

<!-- vs la implementación compleja que oculta -->
<mat-form-field>
  <mat-label>{{label}}</mat-label>
  <input matInput>
  <mat-icon>{{icon}}</mat-icon>
  <!-- ... más configuración ... -->
</mat-form-field>
```

### Facade Pattern (FormFacade)

Proporciona una interfaz unificada para todo el sistema de formularios:

```typescript
// Una interfaz simple para un sistema complejo
this.formFacade.createForm({
  fields: { ... },
  onSuccess: () => { ... },
  onError: () => { ... }
});
```

## Conclusión

El FormFacade no es una solución mágica, sino una herramienta más en tu caja de herramientas. Úsalo cuando aporte valor y simplicidad a tu código, y no dudes en combinar diferentes enfoques según las necesidades específicas de cada caso.
