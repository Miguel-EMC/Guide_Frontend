# Angular Aria (Developer Preview)

Angular Aria is a headless, accessibility-first UI library. It provides unstyled components and directives so you can build accessible interfaces with your own design system.

> Angular Aria is in developer preview. APIs may change between minor versions.

## Installation

```bash
npm install @angular/aria
```

## Available Patterns

Angular Aria provides accessible primitives for:

- Autocomplete
- Listbox
- Select
- Multiselect
- Combobox
- Menu and Menubar
- Toolbar
- Accordion
- Tabs
- Tree
- Grid

## Accordion Example

```typescript
import { Component } from '@angular/core';
import {
  AccordionGroup,
  AccordionTrigger,
  AccordionPanel,
  AccordionContent
} from '@angular/aria/accordion';

@Component({
  standalone: true,
  imports: [AccordionGroup, AccordionTrigger, AccordionPanel, AccordionContent],
  template: `
    <section class="accordion" ngAccordionGroup>
      <button class="accordion-trigger" ngAccordionTrigger>
        What is Angular Aria?
      </button>
      <div class="accordion-panel" ngAccordionPanel>
        <p ngAccordionContent>
          Headless, accessible UI primitives designed to be styled by you.
        </p>
      </div>
    </section>
  `
})
export class AccordionExampleComponent {}
```

## Toolbar Example

```typescript
import { Component } from '@angular/core';
import {
  Toolbar,
  ToolbarWidgetGroup,
  ToolbarWidget
} from '@angular/aria/toolbar';

@Component({
  standalone: true,
  imports: [Toolbar, ToolbarWidgetGroup, ToolbarWidget],
  template: `
    <div class="toolbar" ngToolbar>
      <div ngToolbarWidgetGroup>
        <button ngToolbarWidget>Back</button>
        <button ngToolbarWidget>Forward</button>
      </div>
      <div ngToolbarWidgetGroup>
        <button ngToolbarWidget>Bold</button>
        <button ngToolbarWidget>Italic</button>
      </div>
    </div>
  `
})
export class ToolbarExampleComponent {}
```

## Styling Tips

- Keep focus states visible and consistent
- Preserve native button semantics
- Test with keyboard-only navigation
- Validate with a screen reader

## When to Use Angular Aria

- You want full control over styling
- You need accessible components without a full UI kit
- You are building a design system

## Next Steps

- [Angular Material](./17-angular-material.md) - Fully styled components
- [Testing](./22-testing.md) - Validate accessibility behavior

---

[Previous: Project Task Manager](./30-project-task-manager.md) | [Back to Index](./README.md) | [Next: Zoneless Change Detection](./32-zoneless-change-detection.md)
