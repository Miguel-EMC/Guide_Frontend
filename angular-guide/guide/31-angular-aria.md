# Angular Aria (v21 - Developer Preview)

Angular v21 introduces **Angular Aria** in Developer Preview - a new modern library of accessible, unstyled components that prioritize accessibility and can be customized with your own styles.

## Overview

Angular Aria provides headless components built with accessibility as the primary concern:

| Feature | Description |
|---------|-------------|
| **Headless Components** | Unstyled, fully customizable |
| **Accessibility First** | Built with ARIA patterns |
| **Signal-Based** | Uses modern Angular reactivity |
| **Standalone Directives** | No NgModules required |
| **TypeScript Native** | Full type safety |

## Installation

```bash
# Install Angular Aria
npm install @angular/aria
```

## Available Components

Angular Aria v21 includes 13 components across 8 UI patterns:

| Pattern | Components |
|---------|------------|
| **Accordion** | `Accordion`, `AccordionItem`, `AccordionHeader`, `AccordionContent` |
| **Combobox** | `Combobox` |
| **Grid** | `Grid`, `GridRow`, `GridCell` |
| **Listbox** | `Listbox`, `ListboxOption` |
| **Menu** | `Menu`, `MenuGroup`, `MenuItem`, `MenuItemCheckbox`, `MenuItemRadio` |
| **Tabs** | `Tabs`, `TabList`, `Tab`, `TabPanel` |
| **Toolbar** | `Toolbar`, `ToolbarItem` |
| **Tree** | `Tree`, `TreeItem` |

## Basic Setup

```typescript
import { Component } from '@angular/core';
import { 
  Accordion, 
  AccordionItem, 
  AccordionHeader, 
  AccordionContent 
} from '@angular/aria';

@Component({
  standalone: true,
  imports: [
    Accordion, 
    AccordionItem, 
    AccordionHeader, 
    AccordionContent
  ],
  template: `...`
})
export class AccordionComponent {}
```

## Accordion Component

### Basic Accordion

```typescript
import { Component } from '@angular/core';
import { 
  Accordion, 
  AccordionItem, 
  AccordionHeader, 
  AccordionContent 
} from '@angular/aria';

@Component({
  standalone: true,
  imports: [Accordion, AccordionItem, AccordionHeader, AccordionContent],
  template: `
    <div class="accordion">
      <AccordionItem>
        <AccordionHeader>Section 1</AccordionHeader>
        <AccordionContent>
          <p>Content for section 1</p>
        </AccordionContent>
      </AccordionItem>

      <AccordionItem>
        <AccordionHeader>Section 2</AccordionHeader>
        <AccordionContent>
          <p>Content for section 2</p>
        </AccordionContent>
      </AccordionItem>
    </div>
  `,
  styles: [`
    .accordion {
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    
    [aria-expanded="true"] + [role="region"] {
      display: block;
    }
    
    [aria-expanded="false"] + [role="region"] {
      display: none;
    }
  `]
})
export class BasicAccordionComponent {}
```

### Custom Styled Accordion

```typescript
@Component({
  standalone: true,
  imports: [Accordion, AccordionItem, AccordionHeader, AccordionContent],
  template: `
    <div class="custom-accordion">
      <AccordionItem>
        <AccordionHeader class="accordion-header">
          <span class="header-title">Features</span>
          <span class="header-icon">▼</span>
        </AccordionHeader>
        <AccordionContent class="accordion-content">
          <ul>
            <li>Accessible by default</li>
            <li>Keyboard navigation</li>
            <li>Screen reader support</li>
          </ul>
        </AccordionContent>
      </AccordionItem>
    </div>
  `,
  styles: [`
    .custom-accordion {
      border: 2px solid #4a90e2;
      border-radius: 8px;
      overflow: hidden;
    }

    .accordion-header {
      background: #4a90e2;
      color: white;
      padding: 1rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
      cursor: pointer;
      border: none;
      width: 100%;
      font-size: 1rem;
      font-weight: 600;
    }

    .accordion-header:hover {
      background: #357abd;
    }

    .accordion-header:focus {
      outline: 2px solid #ff6b6b;
      outline-offset: 2px;
    }

    .accordion-content {
      padding: 1rem;
      background: #f8f9fa;
    }
  `]
})
export class StyledAccordionComponent {}
```

## Listbox Component

### Single Selection Listbox

```typescript
import { Component, signal } from '@angular/core';
import { Listbox, ListboxOption } from '@angular/aria';

@Component({
  standalone: true,
  imports: [Listbox, ListboxOption],
  template: `
    <label for="frameworks">Choose a framework:</label>
    <Listbox 
      [selected]="selectedFramework"
      (selectedChange)="onSelectionChange($event)"
      class="framework-listbox">
      
      <ListboxOption value="angular">Angular</ListboxOption>
      <ListboxOption value="react">React</ListboxOption>
      <ListboxOption value="vue">Vue</ListboxOption>
      <ListboxOption value="svelte">Svelte</ListboxOption>
    </Listbox>
    
    <p>Selected: {{ selectedFramework() }}</p>
  `,
  styles: [`
    .framework-listbox {
      border: 2px solid #ddd;
      border-radius: 4px;
      padding: 0.5rem;
      max-height: 200px;
      overflow-y: auto;
    }
    
    [role="option"] {
      padding: 0.5rem;
      cursor: pointer;
      border-radius: 2px;
    }
    
    [role="option"]:hover,
    [role="option"][aria-selected="true"] {
      background: #e3f2fd;
    }
    
    [role="option"][aria-selected="true"] {
      font-weight: 600;
    }
  `]
})
export class ListboxComponent {
  selectedFramework = signal('angular');

  onSelectionChange(value: string) {
    this.selectedFramework.set(value);
  }
}
```

### Multi-Selection Listbox

```typescript
@Component({
  standalone: true,
  imports: [Listbox, ListboxOption],
  template: `
    <label for="skills">Select your skills:</label>
    <Listbox 
      [selectionMode]="'multiple'"
      [selected]="selectedSkills"
      (selectedChange)="onSkillsChange($event)"
      class="skills-listbox">
      
      <ListboxOption value="typescript">TypeScript</ListboxOption>
      <ListboxOption value="javascript">JavaScript</ListboxOption>
      <ListboxOption value="html">HTML</ListboxOption>
      <ListboxOption value="css">CSS</ListboxOption>
      <ListboxOption value="angular">Angular</ListboxOption>
    </Listbox>
    
    <p>Skills: {{ selectedSkills().join(', ') }}</p>
  `
})
export class MultiListboxComponent {
  selectedSkills = signal<string[]>(['typescript', 'angular']);

  onSkillsChange(values: string[]) {
    this.selectedSkills.set(values);
  }
}
```

## Menu Component

### Basic Menu

```typescript
import { Component, signal } from '@angular/core';
import { 
  Menu, 
  MenuItem, 
  MenuItemCheckbox 
} from '@angular/aria';

@Component({
  standalone: true,
  imports: [Menu, MenuItem, MenuItemCheckbox],
  template: `
    <div class="menu-container">
      <button (click)="toggleMenu()" class="menu-trigger">
        File ▼
      </button>
      
      <Menu 
        *ngIf="menuOpen()"
        (close)="menuOpen.set(false)"
        class="dropdown-menu">
        
        <MenuItem (select)="newFile()">New File</MenuItem>
        <MenuItem (select)="openFile()">Open File</MenuItem>
        <MenuItem (select)="saveFile()">Save File</MenuItem>
        
        <hr class="menu-divider" />
        
        <MenuItemCheckbox 
          [checked]="autoSave()"
          (checkedChange)="autoSave.set($event)">
          Auto Save
        </MenuItemCheckbox>
      </Menu>
    </div>
  `,
  styles: [`
    .menu-container {
      position: relative;
      display: inline-block;
    }
    
    .menu-trigger {
      background: #f0f0f0;
      border: 1px solid #ccc;
      padding: 0.5rem 1rem;
      cursor: pointer;
      border-radius: 4px;
    }
    
    .dropdown-menu {
      position: absolute;
      top: 100%;
      left: 0;
      background: white;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      min-width: 200px;
      z-index: 1000;
    }
    
    [role="menuitem"] {
      padding: 0.5rem 1rem;
      cursor: pointer;
      display: block;
      width: 100%;
      text-align: left;
      border: none;
      background: none;
    }
    
    [role="menuitem"]:hover,
    [role="menuitem"]:focus {
      background: #f0f0f0;
    }
    
    .menu-divider {
      margin: 0.25rem 0;
      border: none;
      border-top: 1px solid #eee;
    }
  `]
})
export class MenuComponent {
  menuOpen = signal(false);
  autoSave = signal(false);

  toggleMenu() {
    this.menuOpen.set(!this.menuOpen());
  }

  newFile() {
    console.log('New file created');
    this.menuOpen.set(false);
  }

  openFile() {
    console.log('File opened');
    this.menuOpen.set(false);
  }

  saveFile() {
    console.log('File saved');
    this.menuOpen.set(false);
  }
}
```

## Tabs Component

### Tab Navigation

```typescript
import { Component, signal } from '@angular/core';
import { 
  Tabs, 
  TabList, 
  Tab, 
  TabPanel 
} from '@angular/aria';

@Component({
  standalone: true,
  imports: [Tabs, TabList, Tab, TabPanel],
  template: `
    <Tabs [selected]="selectedTab()" (selectedChange)="selectTab($event)">
      <TabList class="tab-list">
        <Tab value="home">Home</Tab>
        <Tab value="profile">Profile</Tab>
        <Tab value="settings">Settings</Tab>
      </TabList>

      <TabPanel value="home" class="tab-panel">
        <h2>Home</h2>
        <p>Welcome to your dashboard</p>
      </TabPanel>

      <TabPanel value="profile" class="tab-panel">
        <h2>Profile</h2>
        <p>Manage your profile information</p>
      </TabPanel>

      <TabPanel value="settings" class="tab-panel">
        <h2>Settings</h2>
        <p>Configure your preferences</p>
      </TabPanel>
    </Tabs>
  `,
  styles: [`
    .tab-list {
      display: flex;
      border-bottom: 2px solid #e0e0e0;
      margin-bottom: 1rem;
    }
    
    [role="tab"] {
      padding: 0.75rem 1.5rem;
      background: none;
      border: none;
      cursor: pointer;
      border-bottom: 2px solid transparent;
      margin-bottom: -2px;
      font-weight: 500;
      color: #666;
      transition: all 0.2s;
    }
    
    [role="tab"]:hover {
      color: #333;
    }
    
    [role="tab"][aria-selected="true"] {
      color: #4a90e2;
      border-bottom-color: #4a90e2;
    }
    
    [role="tab"]:focus {
      outline: 2px solid #4a90e2;
      outline-offset: 2px;
    }
    
    .tab-panel {
      padding: 1rem 0;
    }
  `]
})
export class TabsComponent {
  selectedTab = signal('home');

  selectTab(value: string) {
    this.selectedTab.set(value);
  }
}
```

## Combobox Component

### Searchable Select

```typescript
import { Component, signal } from '@angular/core';
import { Combobox } from '@angular/aria';

@Component({
  standalone: true,
  imports: [Combobox],
  template: `
    <label for="country-select">Select a country:</label>
    <Combobox
      id="country-select"
      [items]="countries"
      [selected]="selectedCountry()"
      (selectedChange)="onCountrySelect($event)"
      [filterable]="true"
      placeholder="Search countries..."
      class="country-combobox">
    </Combobox>
    
    <p *ngIf="selectedCountry()">Selected: {{ selectedCountry() }}</p>
  `,
  styles: [`
    .country-combobox {
      width: 300px;
    }
    
    [role="combobox"] {
      width: 100%;
      padding: 0.5rem;
      border: 2px solid #ddd;
      border-radius: 4px;
      font-size: 1rem;
    }
    
    [role="combobox"]:focus {
      border-color: #4a90e2;
      outline: none;
    }
    
    [role="listbox"] {
      position: absolute;
      background: white;
      border: 1px solid #ddd;
      border-radius: 4px;
      max-height: 200px;
      overflow-y: auto;
      width: 100%;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    
    [role="option"] {
      padding: 0.5rem;
      cursor: pointer;
    }
    
    [role="option"]:hover,
    [role="option"][aria-selected="true"] {
      background: #f0f0f0;
    }
  `]
})
export class ComboboxComponent {
  countries = [
    'United States', 'Canada', 'Mexico', 'United Kingdom',
    'France', 'Germany', 'Spain', 'Italy', 'Japan', 'China',
    'India', 'Brazil', 'Argentina', 'Australia', 'New Zealand'
  ];
  
  selectedCountry = signal('');

  onCountrySelect(country: string) {
    this.selectedCountry.set(country);
  }
}
```

## Styling Angular Aria Components

### CSS Custom Properties

Angular Aria components use semantic HTML and ARIA attributes, making them highly styleable:

```css
/* Global Aria component styles */
[role="tab"] {
  transition: all 0.2s ease-in-out;
}

[role="tab"][aria-selected="true"] {
  background: var(--primary-color, #4a90e2);
  color: var(--primary-text-color, white);
}

[role="option"]:hover {
  background: var(--hover-bg, #f5f5f5);
}

/* Focus styles for accessibility */
[role="tab"]:focus,
[role="option"]:focus,
[role="menuitem"]:focus {
  outline: 2px solid var(--focus-color, #4a90e2);
  outline-offset: 2px;
}
```

### Theme Integration

```typescript
import { Component } from '@angular/core';
import { Tabs, TabList, Tab, TabPanel } from '@angular/aria';

@Component({
  standalone: true,
  imports: [Tabs, TabList, Tab, TabPanel],
  template: `
    <div class="theme-dark">
      <Tabs [selected]="selectedTab()" (selectedChange)="selectTab($event)">
        <TabList class="dark-tab-list">
          <Tab value="tab1">Dark Tab 1</Tab>
          <Tab value="tab2">Dark Tab 2</Tab>
        </TabList>
        <TabPanel value="tab1">Dark content 1</TabPanel>
        <TabPanel value="tab2">Dark content 2</TabPanel>
      </Tabs>
    </div>
  `,
  styles: [`
    .theme-dark {
      --bg-primary: #1a1a1a;
      --bg-secondary: #2d2d2d;
      --text-primary: #ffffff;
      --text-secondary: #cccccc;
      --accent: #4a9eff;
      --border: #404040;
      
      background: var(--bg-primary);
      color: var(--text-primary);
      padding: 1rem;
    }
    
    .dark-tab-list {
      background: var(--bg-secondary);
      border-bottom: 1px solid var(--border);
    }
    
    .dark-tab-list [role="tab"] {
      color: var(--text-secondary);
      border-bottom: 2px solid transparent;
    }
    
    .dark-tab-list [role="tab"][aria-selected="true"] {
      color: var(--accent);
      border-bottom-color: var(--accent);
    }
  `]
})
export class DarkThemeComponent {
  selectedTab = 'tab1';

  selectTab(value: string) {
    this.selectedTab = value;
  }
}
```

## Accessibility Features

### Built-in ARIA Support

- **Keyboard Navigation**: Arrow keys, Enter, Escape, Tab
- **Screen Reader Support**: Proper ARIA labels and roles
- **Focus Management**: Logical focus flow
- **High Contrast**: Works with high contrast modes

### Keyboard Interactions

| Component | Keys | Action |
|-----------|------|--------|
| **Accordion** | ↑/↓, Space, Enter | Navigate, toggle sections |
| **Listbox** | ↑/↓, Home, End, Type-ahead | Navigate, select options |
| **Menu** | ↑/↓, Enter, Escape | Navigate, activate items |
| **Tabs** | ←/→, Home, End | Navigate between tabs |
| **Combobox** | ↑/↓, Escape, Enter | Navigate options, select |

## Best Practices

### 1. Always Provide Labels

```typescript
// Good
<label for="country">Country:</label>
<Combobox id="country" [items]="countries" />

// Good - with aria-label
<Combobox aria-label="Select your country" [items]="countries" />

// Bad - no label
<Combobox [items]="countries" />
```

### 2. Maintain Contrast Ratios

```css
/* Ensure 4.5:1 contrast for normal text */
.aria-component {
  color: #333; /* Good contrast on white background */
}

/* Ensure 3:1 contrast for large text */
.aria-component h1 {
  color: #666; /* Acceptable for large text */
}
```

### 3. Test with Screen Readers

- Test with NVDA, JAWS, VoiceOver
- Verify ARIA labels are announced
- Check navigation order

### 4. Keyboard Only Testing

- Unplug mouse and test full workflow
- Ensure all interactive elements are reachable
- Verify logical tab order

## Comparison with Other Libraries

| Feature | Angular Aria | Angular Material | Radix UI | Headless UI |
|---------|--------------|------------------|----------|-------------|
| **Styling** | Unstyled | Styled Material | Unstyled | Unstyled |
| **Accessibility** | Built-in | Built-in | Built-in | Built-in |
| **Angular** | Native | Native | React | React |
| **Signals** | Yes | Limited | No | No |
| **API Surface** | Growing | Mature | Mature | Mature |
| **Status** | Developer Preview | Stable | Stable | Stable |

## Summary

| Feature | Description |
|---------|-------------|
| **13 Components** | Across 8 UI patterns |
| **Headless Design** | Full styling control |
| **Accessibility First** | WCAG compliant by default |
| **Signal-Based** | Modern Angular patterns |
| **Developer Preview** | Evolving API |
| **TypeScript Native** | Full type safety |
| **Keyboard Navigation** | Built-in keyboard support |

### Current Status

- **Developer Preview**: API may change
- **Core patterns stable**: Accordion, Tabs, Menu, etc.
- **Documentation**: Improving rapidly
- **Community feedback**: Welcomed for refinement

### When to Use Angular Aria

- ✅ **Custom designs** needing accessibility
- ✅ **Design systems** with specific styling needs  
- ✅ **Accessibility requirements** are critical
- ✅ **Modern Angular** applications with signals
- ❌ **Production apps** during developer preview (caution)

## Next Steps

- [Testing](./22-testing.md) - Test your accessible components
- [Angular Material](./17-angular-material.md) - Compare with styled alternatives

---

[Previous: Libraries and Integrations](./18-libraries-and-integrations.md) | [Back to Index](./README.md) | [Next: Animations](./21-animations.md) (Chapter not yet available)