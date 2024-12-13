# Blade Component Documentation

## Overview

The Blade component provides a sliding panel that appears from the right side of the screen, similar to Azure Portal's blade system. It's useful for displaying forms, details, or any other content without leaving the current page.

## Installation

1. Copy the blade component files to your project:

```shell
shared/
  components/
    blade/
      blade.service.ts
      blade.component.ts
      blade.component.html
      blade.component.scss
```

2. Add the BladeComponent to your page-base component:

```html
<am-blade />
```

## Usage

### Basic Usage

```typescript
readonly bladeService = inject(BladeService)

openBlade() {
  this.bladeService.show(YourComponent);
}
```

### With Custom Width

```typescript
this.bladeService.show(YourComponent, { width: '800px' });
```

## API

### BladeService

#### Methods

- `show(component: Type<unknown>, options?: BladeOptions): void`
  - Opens the blade with the specified component
  - Optional options object to customize the blade appearance

- `hide(): void`
  - Closes the blade

#### Interfaces

```typescript
interface BladeOptions {
  width?: string;  // Default: '480px'
}
```

### Creating Components for the Blade

Components that will be displayed in the blade should follow this structure:

```typescript
@Component({
  selector: 'am-your-component',
  template: `
    <div class="blade-content-wrapper">
      <h2 class="blade-title">Your Title</h2>
      <!-- Your content here -->
    </div>
  `
})
export class YourComponent {
  // Your component logic
}
```

## Styling

The blade comes with predefined styles including:

- Smooth Material Design animations
- Responsive layout (max-width: 90vw)
- Customizable width through options
- Styled scrollbar
- Proper z-indexing

## Example

```typescript
@Component({
  selector: 'am-example',
  template: `
    <button (click)="openDetailsBlade()">Open Details</button>
  `
})
export class ExampleComponent {
  constructor(private bladeService: BladeService) {}

  openDetailsBlade() {
    this.bladeService.show(DetailsComponent, { width: '500px' });
  }
}
```

## Features

- 🚀 Dynamic component loading
- 📱 Responsive design
- 🎨 Material Design styling
- 🔄 Smooth animations
- 🛠️ Customizable width
- 🎯 Simple API

## Best Practices

1. Let components manage their own content structure
2. Use appropriate widths based on content type:
   - Forms: 480px - 800px
   - Details: 320px - 480px
   - Complex views: Consider using percentages (e.g., '50vw')
3. Remember the blade is meant for supplementary content - keep the main content in the primary view

## Dependencies

- @angular/core
- @angular/common
- @angular/material (for styling and icons)
