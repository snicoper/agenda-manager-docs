# NavToolbar - Guía de Uso

El componente NavToolbar proporciona una manera sencilla de crear navegación por pestañas con contenido dinámico.

## Convención de Nomenclatura

Por convención, todos los componentes destinados a ser utilizados como pestañas deben tener el sufijo `-tab`:

- `UserInfoTabComponent`
- `UserRolesTabComponent`
- `SettingsTabComponent`

Esta convención ayuda a:

- Identificar claramente componentes diseñados para ser tabs
- Mantener una estructura de código consistente
- Facilitar la navegación y búsqueda en el proyecto

## Uso Básico

```typescript
@Component({
  template: `<am-nav-toolbar [data]="navData" />`
})
class MyComponent {
  navData: NavToolbarData = {
    tabs: [
      {
        label: 'Información',
        icon: 'person',
        component: UserInfoTabComponent
      },
      {
        label: 'Configuración',
        icon: 'settings',
        component: SettingsTabComponent
      }
    ]
  };
}
```

## Pasando Datos a los Componentes

Puedes pasar datos a los componentes de cada pestaña usando la propiedad `inputs`:

```typescript
navData: NavToolbarData = {
  tabs: [
    {
      label: 'Detalles Usuario',
      icon: 'person',
      component: UserDetailsTabComponent,
      inputs: {
        userId: '123',
        isAdmin: true
      }
    }
  ]
};
```

## Badges y Notificaciones

Añade badges para mostrar notificaciones o contadores:

```typescript
{
  label: 'Mensajes',
  icon: 'mail',
  component: MessagesTabComponent,
  badge: {
    value: '4',
    color: 'warn'  // 'primary' | 'accent' | 'warn'
  }
}
```

## Control de Pestañas

### Pestañas Deshabilitadas

```typescript
{
  label: 'Administración',
  icon: 'admin_panel_settings',
  component: AdminTabComponent,
  disabled: true
}
```

### Control del Índice Seleccionado

```typescript
<am-nav-toolbar
  [data]="navData"
  [(selectedIndex)]="currentTabIndex"
/>
```

## Casos de Uso Comunes

### Detalles de Usuario

```typescript
navData: NavToolbarData = {
  tabs: [
    {
      label: 'Información',
      icon: 'person',
      component: UserInfoTabComponent,
      inputs: { userId: this.userId }
    },
    {
      label: 'Roles',
      icon: 'security',
      component: UserRolesTabComponent,
      inputs: { userId: this.userId }
    }
  ]
};
```

### Panel de Administración

```typescript
navData: NavToolbarData = {
  tabs: [
    {
      label: 'Dashboard',
      icon: 'dashboard',
      component: DashboardTabComponent
    },
    {
      label: 'Usuarios',
      icon: 'group',
      component: UsersTabComponent,
      badge: {
        value: '2',
        color: 'accent'
      }
    },
    {
      label: 'Configuración',
      icon: 'settings',
      component: SettingsTabComponent
    }
  ]
};
```

### Gestión de Recursos

```typescript
navData: NavToolbarData = {
  tabs: [
    {
      label: 'Lista',
      icon: 'list',
      component: ResourceListTabComponent
    },
    {
      label: 'Calendario',
      icon: 'calendar_today',
      component: ResourceCalendarTabComponent
    },
    {
      label: 'Estadísticas',
      icon: 'bar_chart',
      component: ResourceStatsTabComponent
    }
  ]
};
```

## Mejores Prácticas

1. **Nomenclatura**:
   - Usa el sufijo `-tab` para todos los componentes de pestañas
   - Los selectores deben seguir el patrón `am-[nombre]-tab`
2. **Nombrado de Pestañas**: Usa nombres cortos y descriptivos
3. **Iconos**: Utiliza iconos que representen claramente la función de la pestaña
4. **Orden**: Coloca las pestañas más utilizadas al principio
5. **Badges**: Úsalos solo para información importante y actual
6. **Componentes**: Cada componente debe ser independiente y manejar su propio estado

## Consideraciones de Rendimiento

- Los componentes se cargan de forma perezosa (lazy loading) gracias a `@defer`
- Solo se renderiza el contenido de la pestaña activa
- Los inputs se actualizan automáticamente cuando cambian
