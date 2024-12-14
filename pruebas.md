Hola, estoy creado un componente blade, para añadir el contenido de componentes dentro (similar en funcionacionamiento a Azure)

Este es el componente (omito el scss, ya que no es problemático)

```ts
// src\app\shared\components\blade\interfaces\blade-options.interface.ts
export interface BladeOptions<TData = unknown> {
  width?: string;
  data?: TData;
}

// src\app\shared\components\blade\services\blade.service.ts
@Injectable({ providedIn: 'root' })
export class BladeService {
  private readonly isVisible$ = signal(false);
  private readonly component$ = signal<Type<unknown> | null>(null);
  private readonly options$ = signal<BladeOptions>({ width: '480px' });
  private readonly outputResult$ = signal<unknown>(null);

  readonly bladeState = computed(() => ({
    isVisible: this.isVisible$(),
    component: this.component$(),
    options: this.options$(),
    outputResult: this.outputResult$(),
  }));

  show<TData = unknown>(component: Type<unknown>, options?: BladeOptions<TData>): void {
    this.component$.set(component);

    if (options) {
      this.options$.set({ ...this.options$(), ...options });
    }

    this.isVisible$.set(true);
  }

  handleOutputResult<TData = unknown>(result: TData): void {
    this.outputResult$.set(result);
  }

  hide(): void {
    this.isVisible$.set(false);
    this.component$.set(null);
    this.options$.set({ width: '480px' });
  }
}

// src\app\shared\components\blade\blade.component.ts
@Component({
  selector: 'am-blade',
  standalone: true,
  imports: [CommonModule, MatIconModule, MatButtonModule],
  templateUrl: './blade.component.html',
  styleUrl: './blade.component.scss',
})
export class BladeComponent {
  bladeService = inject(BladeService);

  data = input<unknown>();

  handleOutput(result: unknown): void {
    this.bladeService.handleOutputResult(result);
  }
}
```

```html
<!-- src\app\shared\components\blade\blade.component.html -->
<div class="blade-container" [class.visible]="bladeService.bladeState().isVisible">
  <div class="blade" [style.width]="bladeService.bladeState().options.width">
    <div class="blade-header">
      <button mat-icon-button (click)="bladeService.hide()">
        <mat-icon>close</mat-icon>
      </button>
    </div>

    <div class="blade-content">
      <ng-container
        *ngComponentOutlet="
          bladeService.bladeState().component;
          inputs: { data: bladeService.bladeState().options.data };
          outputs: { outputResult: handleOutput }
        "
      />
    </div>
  </div>
</div>
```

Como funciona? creo un componente, por ejemplo para este caso es para la creación de un role nuevo.

```ts
// src\app\features\authorization\models\create-role-data.blade.ts
export interface CreateRoleDataBlade {
  roleId: string;
}

// src\app\features\authorization\components\role-create-blade\role-create-blade.component.ts
export class RoleCreateBladeComponent {
  private readonly bladeService = inject(BladeService);

  outputResult = output<boolean>();

   private create(request: CreateRoleRequest): void {
    this.formState.isLoading = true;

    this.apiService
      .createRole(request)
      .pipe(finalize(() => (this.formState.isSubmitted = false)))
      .subscribe({
        next: (response) => {
          if (response) {
            this.snackBarService.success('Rol creado correctamente.');
            this.bladeService.hide();
            this.outputResult.emit(true);
          }
        },
        error: (error) => (this.formState.badRequest = error.error),
      });
  }

  // Omito partes de código no relevantes.
}
```

Ahora muestro desde donde se llama, igual, solo añado partes de código relevantes del componente.

```ts
// src\app\features\authorization\pages\role-list\role-list.component.ts
export class RoleListComponent implements AfterViewInit {
  private readonly bladeService = inject(BladeService);

  constructor() {
    effect(() => {
      const result = this.bladeService.bladeState().outputResult;

      if (result) {
        this.loadRoles();
      }
    });
  }

  handleOpenCreateRoleBlade(): void {
    this.bladeService.show(RoleCreateBladeComponent);
  }

  // Omito partes de código no relevantes.
}
```

El problema es que no emite o no recarga los roles cuando hace `this.outputResult.emit(true);` en `RoleCreateBladeComponent`
