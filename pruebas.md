Este es el caso tipico de formulario en un componente

```ts
export class RoleUpdateBladeComponent {
  private readonly apiService = inject(AuthorizationApiService);
  private readonly formBuilder = inject(FormBuilder);
  private readonly snackBarService = inject(SnackBarService);

  readonly formState = {
    form: this.formBuilder.group({}),
    badRequest: undefined,
    isSubmitted: false,
    isLoading: false,
  } as FormState;

  handleSubmit(): void {
    this.formState.isSubmitted = true;

    if (this.formState.form.invalid) {
      return;
    }

    const request = this.formState.form.value as RoleUpdateRequest;
    this.updateRole(request);
  }

  private updateRole(request: RoleUpdateRequest): void {
    this.apiService
      .updateRole(this.role.id, request)
      .pipe(finalize(() => (this.formState.isLoading = false)))
      .subscribe({
        next: () => {
          this.snackBarService.success('Role actualizado correctamente.');
          this.bladeService.emitResult<string>(this.role.id);
        },
        error: (error) => {
          if (error.status === HttpStatusCode.BadRequest || error.status === HttpStatusCode.Conflict) {
            this.formState.badRequest = error.error;

            return;
          }

          this.snackBarService.error('Ha ocurrido un error al actualizar el role.');
        },
      });
  }

  private buildForm(): void {
    const formContract = {
      name: {
        ...RoleFormConfig.name,
        initialValue: this.role.name,
      },
      description: {
        ...RoleFormConfig.description,
        initialValue: this.role.description,
      },
    } as RoleFormContract;

    this.formState.form = this.formBuilder.group({
      name: [formContract.name.initialValue, formContract.name.validators],
      description: [formContract.description.initialValue, formContract.description.validators],
    });
  }
}
```

En este caso, el `RoleFormContract` es para compartir validaciones con el create y el update
