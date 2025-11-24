# Comprehensive Catalyst UI Components Guide

## Preamble: The Catalyst Design Philosophy

### Core Design Ethos

Catalyst UI represents a paradigm shift in component library architecture, embodying the principle of **"Your Components, Not Ours"**. Built by Tailwind Labs, it's a "disappearing UI kit" - components you download and copy directly into your codebase rather than installing as dependencies. After six months of use, the components become so integrated that you forget they weren't originally yours.

### Design Principles

**1. Competitive Excellence**
- Designed to compete with the most sophisticated interfaces on the web
- Meticulous attention to details like backdrop blurs, shadow blending, and form control refinement
- Thoughtful animation implementation for meaningful interactions

**2. Timeless Design Balance**
- Strikes equilibrium between flat and skeuomorphic design
- Avoids trend-chasing with unopinionated blue focus rings
- Provides just enough depth cues to remain relevant regardless of design trends

**3. Productivity-First Development**
- Generous whitespace balanced with information density
- Limited, fast animations that never block user actions
- Comprehensive dark mode with context-specific adjustments
- Responsive design built into every component

### Architectural Philosophy: HTML-Mirrored APIs

Catalyst deliberately mirrors HTML's compositional patterns rather than monolithic prop-heavy APIs:

```jsx
// ❌ Traditional monolithic approach (avoided)
<TextField 
  name="product_name" 
  label="Product name" 
  description="Use the name you'd like people to see in their cart." 
/>

// ✅ Catalyst HTML-like composition
<Field>
  <Label>Product name</Label>
  <Description>Use the name you'd like people to see in their cart.</Description>
  <Input name="product_name" className="max-w-sm" />
</Field>
```

## Technical Foundation

### Core Dependencies

```bash
# Required dependencies
npm install @headlessui/react@^2.0.0 framer-motion clsx
npm install tailwindcss@^4.0  # Requires v4.0+

# Optional but recommended
npm install @heroicons/react  # Icon system
```

### TypeScript Configuration

Catalyst is TypeScript-first with comprehensive type definitions. Both TypeScript and JavaScript versions are provided.

```typescript
// All components have full TypeScript support
interface ComponentProps extends React.ComponentPropsWithoutRef<'element'> {
  // Properly typed props with inference
}
```

### Framework Integration

Catalyst works with any React framework. Update the Link component for your router:

```typescript
// Next.js integration example
import * as Headless from '@headlessui/react'
import NextLink, { type LinkProps } from 'next/link'
import React, { forwardRef } from 'react'

export const Link = forwardRef(function Link(
  props: LinkProps & React.ComponentPropsWithoutRef<'a'>,
  ref: React.ForwardedRef<HTMLAnchorElement>
) {
  return (
    <Headless.DataInteractive>
      <NextLink {...props} ref={ref} />
    </Headless.DataInteractive>
  )
})
```

## Complete Component Documentation

### 1. Button Components

#### Button
**Purpose**: Primary interactive element for all user actions

**Props**:
- `type`: `'button' | 'submit' | 'reset'` (default: `'button'`)
- `color`: Extensive palette including adaptive and 20+ solid colors
- `outline`: `boolean` - Secondary style with no background
- `plain`: `boolean` - Minimal style with no borders/shadows
- `disabled`: `boolean` - Disabled state (incompatible with `href`)
- `href`: `string` - Renders as Link component for navigation

**Implementation**:
```tsx
import { Button } from '@/components/ui/button'
import { PlusIcon } from '@heroicons/react/16/solid'

// Basic button
<Button>Save changes</Button>

// With color
<Button color="cyan">Save changes</Button>

// Outline variant
<Button outline>Save draft</Button>

// With icon (use 16×16 icons)
<Button>
  <PlusIcon />
  Add item
</Button>

// As link
<Button href="/get-started">Get started</Button>

// Disabled state
<Button disabled={isLoading}>
  {isLoading ? 'Saving...' : 'Save changes'}
</Button>
```

**Accessibility**: Full keyboard navigation, focus management, ARIA attributes
**Responsive**: Adaptive colors adjust between light/dark modes
**Common Mistakes**: Using wrong icon size (must be 16×16), using `disabled` with `href`

### 2. Form Input Components

#### Input
**Purpose**: Text input for various data types

**Props**:
- `type`: `'email' | 'number' | 'password' | 'search' | 'tel' | 'text' | 'url' | 'date' | 'datetime-local' | 'month' | 'time' | 'week'`
- `disabled`: `boolean`
- `invalid`: `boolean` - Error state indication
- `name`: `string` - Form field name
- `defaultValue`: `string` - Uncontrolled initial value
- `value`: `string` - Controlled value
- `onChange`: `(e: ChangeEvent) => void`

**Implementation**:
```tsx
import { Field, Label, Description, ErrorMessage } from '@/components/ui/fieldset'
import { Input, InputGroup } from '@/components/ui/input'
import { MagnifyingGlassIcon } from '@heroicons/react/16/solid'

// Basic with label
<Field>
  <Label>Full name</Label>
  <Input name="full_name" />
</Field>

// With description
<Field>
  <Label>Product name</Label>
  <Description>Use the name you'd like people to see in their cart.</Description>
  <Input name="product_name" />
</Field>

// With icon
<InputGroup>
  <MagnifyingGlassIcon />
  <Input name="search" placeholder="Search…" />
</InputGroup>

// Validation error state
<Field>
  <Label>Email</Label>
  <Input name="email" type="email" invalid={errors.has('email')} />
  {errors.has('email') && <ErrorMessage>{errors.get('email')}</ErrorMessage>}
</Field>

// Controlled component
const [name, setName] = useState('')
<Input value={name} onChange={(e) => setName(e.target.value)} />
```

#### Textarea
**Purpose**: Multi-line text input

**Props**:
- `disabled`: `boolean`
- `invalid`: `boolean`
- `resizable`: `boolean` (default: `true`)
- `rows`: `number`
- Standard textarea props

**Implementation**:
```tsx
import { Textarea } from '@/components/ui/textarea'

<Field>
  <Label>Description</Label>
  <Textarea name="description" rows={4} />
</Field>
```

#### Select
**Purpose**: Dropdown selection from options

**Props**:
- `disabled`: `boolean`
- `invalid`: `boolean`
- Standard select props

**Implementation**:
```tsx
import { Select } from '@/components/ui/select'

<Field>
  <Label>Project status</Label>
  <Select name="status" defaultValue="">
    <option value="" disabled>Select a status…</option>
    <option value="active">Active</option>
    <option value="paused">Paused</option>
    <option value="delayed">Delayed</option>
  </Select>
</Field>
```

#### Combobox
**Purpose**: Searchable dropdown with filtering

**Props**:
- `options`: `array` - Data source
- `displayValue`: `(item: T) => string` - Display formatter
- `filter`: `(item: T, query: string) => boolean` - Custom filter
- `anchor`: `string` - Position (default: `'bottom'`)
- Standard input props

**Implementation**:
```tsx
import { Combobox, ComboboxOption, ComboboxLabel, ComboboxDescription } from '@/components/ui/combobox'
import { Avatar } from '@/components/avatar'

<Field>
  <Label>Assigned to</Label>
  <Combobox 
    name="user" 
    options={users} 
    displayValue={(user) => user?.name} 
    defaultValue={currentUser}
  >
    {(user) => (
      <ComboboxOption value={user}>
        <Avatar src={user.avatarUrl} initials={user.initials} />
        <ComboboxLabel>{user.name}</ComboboxLabel>
        <ComboboxDescription>{user.role}</ComboboxDescription>
      </ComboboxOption>
    )}
  </Combobox>
</Field>
```

#### Listbox
**Purpose**: Custom styled select alternative

**Props**:
- Similar to Select but with custom styling
- `placeholder`: `string`

**Implementation**:
```tsx
import { Listbox, ListboxOption, ListboxLabel, ListboxDescription } from '@/components/ui/listbox'

<Field>
  <Label>Priority</Label>
  <Listbox name="priority" defaultValue="normal">
    <ListboxOption value="low">
      <ListboxLabel>Low</ListboxLabel>
      <ListboxDescription>For minor issues</ListboxDescription>
    </ListboxOption>
    <ListboxOption value="normal">
      <ListboxLabel>Normal</ListboxLabel>
    </ListboxOption>
    <ListboxOption value="high">
      <ListboxLabel>High</ListboxLabel>
    </ListboxOption>
  </Listbox>
</Field>
```

### 3. Form Control Components

#### Checkbox
**Purpose**: Boolean selection, multiple choice

**Props**:
- `color`: Color palette options
- `disabled`: `boolean`
- `indeterminate`: `boolean` - Partial selection state
- `defaultChecked`: `boolean`
- `checked`: `boolean` - Controlled
- `onChange`: `(checked: boolean) => void`

**Implementation**:
```tsx
import { Checkbox, CheckboxField, CheckboxGroup } from '@/components/ui/checkbox'

// Single checkbox
<CheckboxField>
  <Checkbox name="allow_embedding" />
  <Label>Allow embedding</Label>
  <Description>Allow others to embed your event details.</Description>
</CheckboxField>

// Multiple checkboxes
<CheckboxGroup>
  <CheckboxField>
    <Checkbox name="show_on_events_page" defaultChecked />
    <Label>Show on events page</Label>
  </CheckboxField>
  <CheckboxField>
    <Checkbox name="allow_embedding" />
    <Label>Allow embedding</Label>
  </CheckboxField>
</CheckboxGroup>

// Indeterminate state
<Checkbox
  checked={selected.length > 0}
  indeterminate={selected.length > 0 && selected.length < options.length}
  onChange={(checked) => setSelected(checked ? options : [])}
/>
```

#### Radio
**Purpose**: Single selection from options

**RadioGroup Props**:
- `name`: `string`
- `defaultValue`: `string`
- `value`: `string` - Controlled
- `onChange`: `(value: string) => void`

**Radio Props**:
- `value`: `string` - Option value
- `color`: Color options
- `disabled`: `boolean`

**Implementation**:
```tsx
import { Radio, RadioField, RadioGroup } from '@/components/ui/radio'

<RadioGroup name="resale" defaultValue="permit">
  <RadioField>
    <Radio value="permit" />
    <Label>Allow tickets to be resold</Label>
    <Description>Customers can transfer tickets.</Description>
  </RadioField>
  <RadioField>
    <Radio value="forbid" />
    <Label>Don't allow tickets to be resold</Label>
  </RadioField>
</RadioGroup>
```

#### Switch
**Purpose**: Toggle for binary settings

**Props**:
- `color`: Color options
- `disabled`: `boolean`
- `defaultChecked`: `boolean`
- `checked`: `boolean` - Controlled
- `onChange`: `(checked: boolean) => void`

**Implementation**:
```tsx
import { Switch, SwitchField, SwitchGroup } from '@/components/ui/switch'

<SwitchField>
  <Label>Allow notifications</Label>
  <Description>Receive email updates about your events.</Description>
  <Switch name="notifications" defaultChecked />
</SwitchField>

// Multiple switches
<SwitchGroup>
  <SwitchField>
    <Label>Show on events page</Label>
    <Switch name="show_on_events_page" defaultChecked />
  </SwitchField>
  <SwitchField>
    <Label>Allow embedding</Label>
    <Switch name="allow_embedding" />
  </SwitchField>
</SwitchGroup>
```

### 4. Form Organization Components

#### Field, Fieldset, Legend
**Purpose**: Semantic form organization and accessibility

**Components**:
- `Field`: Associates labels, inputs, descriptions
- `Fieldset`: Groups related fields
- `Legend`: Fieldset title
- `FieldGroup`: Groups fields within fieldset
- `Label`: Input labels
- `Description`: Helper text
- `ErrorMessage`: Validation errors

**Implementation**:
```tsx
import { Field, FieldGroup, Fieldset, Label, Legend, Description, ErrorMessage } from '@/components/ui/fieldset'

<Fieldset>
  <Legend>Shipping details</Legend>
  <Text>Enter your shipping information.</Text>
  <FieldGroup>
    <Field>
      <Label>Street address</Label>
      <Input name="street_address" />
    </Field>
    <Field>
      <Label>Country</Label>
      <Select name="country">
        <option>United States</option>
        <option>Canada</option>
      </Select>
      <Description>We currently only ship to North America.</Description>
    </Field>
  </FieldGroup>
</Fieldset>
```

### 5. Navigation Components

#### Navbar
**Purpose**: Horizontal navigation bar

**Components**:
- `Navbar`: Container
- `NavbarItem`: Navigation items
- `NavbarSection`: Groups items
- `NavbarSpacer`: Creates space
- `NavbarDivider`: Visual separator
- `NavbarLabel`: Text labels

**Props**:
- `NavbarItem`: `current` (boolean), `href` (string)

**Implementation**:
```tsx
import { Navbar, NavbarItem, NavbarSection, NavbarSpacer } from '@/components/ui/navbar'
import { MagnifyingGlassIcon } from '@heroicons/react/20/solid' // Note: 20×20 for nav

<Navbar>
  <NavbarSection>
    <NavbarItem href="/" current>Home</NavbarItem>
    <NavbarItem href="/events">Events</NavbarItem>
  </NavbarSection>
  <NavbarSpacer />
  <NavbarSection>
    <NavbarItem href="/search" aria-label="Search">
      <MagnifyingGlassIcon />
    </NavbarItem>
  </NavbarSection>
</Navbar>
```

#### Sidebar
**Purpose**: Vertical navigation sidebar

**Components**:
- `Sidebar`: Container
- `SidebarHeader`: Sticky header
- `SidebarBody`: Scrollable content
- `SidebarFooter`: Sticky footer
- `SidebarItem`: Navigation items
- `SidebarSection`: Groups items
- `SidebarHeading`: Section titles
- `SidebarLabel`: Item labels

**Props**:
- `SidebarItem`: `current` (boolean), `href` (string)

**Implementation**:
```tsx
import { Sidebar, SidebarBody, SidebarItem, SidebarLabel, SidebarSection } from '@/components/ui/sidebar'
import { HomeIcon } from '@heroicons/react/20/solid' // 20×20 for sidebar

<Sidebar>
  <SidebarBody>
    <SidebarSection>
      <SidebarItem href="/" current>
        <HomeIcon />
        <SidebarLabel>Home</SidebarLabel>
      </SidebarItem>
      <SidebarItem href="/events">
        <Square2StackIcon />
        <SidebarLabel>Events</SidebarLabel>
      </SidebarItem>
    </SidebarSection>
  </SidebarBody>
</Sidebar>
```

#### Application Layouts
**Purpose**: Complete layout systems

**SidebarLayout**: Desktop sidebar with mobile navbar
```tsx
import { SidebarLayout } from '@/components/ui/sidebar-layout'

<SidebarLayout 
  sidebar={<Sidebar>{/* sidebar content */}</Sidebar>}
  navbar={<Navbar>{/* mobile navigation */}</Navbar>}
>
  {/* page content */}
</SidebarLayout>
```

**StackedLayout**: Horizontal navbar with optional sidebar
```tsx
import { StackedLayout } from '@/components/ui/stacked-layout'

<StackedLayout navbar={<Navbar>{/* navigation */}</Navbar>}>
  {/* page content */}
</StackedLayout>
```

#### Pagination
**Purpose**: Navigate between data pages

**Components**:
- `Pagination`: Container (nav element)
- `PaginationPrevious`: Previous link
- `PaginationNext`: Next link
- `PaginationList`: Page number container
- `PaginationPage`: Individual page links
- `PaginationGap`: Ellipsis indicator

**Props**:
- `PaginationPage`: `current` (boolean), `href` (string)
- `PaginationPrevious/Next`: `href` (omit to disable)

**Implementation**:
```tsx
import { Pagination, PaginationGap, PaginationList, PaginationNext, PaginationPage, PaginationPrevious } from '@/components/ui/pagination'

<Pagination>
  <PaginationPrevious href="?page=2" />
  <PaginationList>
    <PaginationPage href="?page=1">1</PaginationPage>
    <PaginationPage href="?page=2">2</PaginationPage>
    <PaginationPage href="?page=3" current>3</PaginationPage>
    <PaginationGap />
    <PaginationPage href="?page=10">10</PaginationPage>
  </PaginationList>
  <PaginationNext href="?page=4" />
</Pagination>
```

### 6. Data Display Components

#### Table
**Purpose**: Display tabular data

**Components**:
- `Table`: Container
- `TableHead`: Header section
- `TableBody`: Body section
- `TableRow`: Rows
- `TableHeader`: Header cells
- `TableCell`: Data cells

**Props**:
- `Table`: `bleed` (full-width), `dense` (compact), `grid` (vertical lines), `striped` (alternating rows)
- `TableRow`: `href` (clickable rows), `target`, `title`

**Implementation**:
```tsx
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table'

<Table className="[--gutter:theme(spacing.6)] sm:[--gutter:theme(spacing.8)]">
  <TableHead>
    <TableRow>
      <TableHeader>Name</TableHeader>
      <TableHeader>Email</TableHeader>
      <TableHeader>Role</TableHeader>
    </TableRow>
  </TableHead>
  <TableBody>
    {users.map((user) => (
      <TableRow key={user.id} href={`/users/${user.id}`}>
        <TableCell className="font-medium">{user.name}</TableCell>
        <TableCell>{user.email}</TableCell>
        <TableCell className="text-zinc-500">{user.role}</TableCell>
      </TableRow>
    ))}
  </TableBody>
</Table>
```

**Responsive**: Use `--gutter` CSS variable for responsive scrolling
**Features**: Automatic horizontal scrolling, clickable rows, complex content support

#### DescriptionList
**Purpose**: Display key-value pairs

**Components**:
- `DescriptionList`: Container (dl)
- `DescriptionTerm`: Keys (dt)
- `DescriptionDetails`: Values (dd)

**Implementation**:
```tsx
import { DescriptionDetails, DescriptionList, DescriptionTerm } from '@/components/ui/description-list'

<DescriptionList>
  <DescriptionTerm>Customer</DescriptionTerm>
  <DescriptionDetails>Michael Foster</DescriptionDetails>
  <DescriptionTerm>Amount</DescriptionTerm>
  <DescriptionDetails>$150.00 USD</DescriptionDetails>
  <DescriptionTerm>Date</DescriptionTerm>
  <DescriptionDetails>December 9, 2024</DescriptionDetails>
</DescriptionList>
```

### 7. Overlay Components

#### Dialog/Modal
**Purpose**: Modal overlays for important interactions

**Components**:
- `Dialog`: Container
- `DialogTitle`: Modal title
- `DialogDescription`: Description text
- `DialogBody`: Content area
- `DialogActions`: Button area

**Props**:
- `open`: `boolean` - Controlled visibility
- `onClose`: `(open: boolean) => void`
- `size`: `'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl' | '3xl' | '4xl' | '5xl'`

**Implementation**:
```tsx
import { Dialog, DialogTitle, DialogDescription, DialogBody, DialogActions } from '@/components/ui/dialog'
import { Button } from '@/components/ui/button'

const [isOpen, setIsOpen] = useState(false)

<Dialog open={isOpen} onClose={setIsOpen} size="md">
  <DialogTitle>Confirm deletion</DialogTitle>
  <DialogDescription>
    Are you sure you want to delete this item? This action cannot be undone.
  </DialogDescription>
  <DialogBody>
    {/* Additional content */}
  </DialogBody>
  <DialogActions>
    <Button plain onClick={() => setIsOpen(false)}>
      Cancel
    </Button>
    <Button color="red" onClick={handleDelete}>
      Delete
    </Button>
  </DialogActions>
</Dialog>
```

**Accessibility**: Focus trapping, ESC key handling, backdrop click, screen reader support
**Z-index**: Handled automatically via portals

#### Dropdown
**Purpose**: Contextual menus and actions

**Components**:
- `Dropdown`: Container
- `DropdownButton`: Trigger button
- `DropdownMenu`: Menu container
- `DropdownItem`: Menu items
- `DropdownSection`: Section grouping
- `DropdownHeading`: Section titles
- `DropdownDivider`: Separators
- `DropdownLabel`: Item labels
- `DropdownDescription`: Item descriptions
- `DropdownShortcut`: Keyboard shortcuts

**Props**:
- `anchor`: Position (`'top' | 'right' | 'bottom' | 'left'` + start/end variants)
- `DropdownButton`: Inherits Button props

**Implementation**:
```tsx
import { Dropdown, DropdownButton, DropdownMenu, DropdownItem, DropdownSection, DropdownHeading, DropdownDivider } from '@/components/ui/dropdown'
import { EllipsisHorizontalIcon, PencilIcon, TrashIcon } from '@heroicons/react/16/solid'

<Dropdown>
  <DropdownButton plain>
    <EllipsisHorizontalIcon />
  </DropdownButton>
  <DropdownMenu anchor="bottom end">
    <DropdownSection>
      <DropdownHeading>Actions</DropdownHeading>
      <DropdownItem href={`/edit/${item.id}`}>
        <PencilIcon />
        <DropdownLabel>Edit</DropdownLabel>
      </DropdownItem>
      <DropdownItem onClick={() => handleDelete(item.id)}>
        <TrashIcon />
        <DropdownLabel>Delete</DropdownLabel>
        <DropdownShortcut>⌘D</DropdownShortcut>
      </DropdownItem>
    </DropdownSection>
  </DropdownMenu>
</Dropdown>
```

### 8. Feedback Components

#### Badge
**Purpose**: Status indicators and labels

**Components**:
- `Badge`: Static badge
- `BadgeButton`: Interactive badge

**Props**:
- `color`: 18 color options with automatic dark mode
- `href`: Make badge a link (BadgeButton only)

**Implementation**:
```tsx
import { Badge, BadgeButton } from '@/components/ui/badge'

// Static badge
<Badge color="lime">Active</Badge>
<Badge color="red">Critical</Badge>

// Interactive badge
<BadgeButton href="/docs">Documentation</BadgeButton>
<BadgeButton onClick={handleClick}>Clickable</BadgeButton>
```

### 9. Typography and Content Components

#### Text
**Purpose**: Paragraph text with rich content

**Components**:
- `Text`: Paragraph container
- `TextLink`: In-text links
- `Strong`: Bold emphasis
- `Code`: Inline code

**Implementation**:
```tsx
import { Text, TextLink, Strong, Code } from '@/components/ui/text'

<Text>
  This feature requires the <Strong>Business Plan</Strong>. 
  View the <TextLink href="/docs">documentation</TextLink> or 
  use API key: <Code>sk_test_123abc</Code>
</Text>
```

#### Avatar
**Purpose**: User avatars with fallback

**Props**:
- `src`: `string` - Image URL
- `square`: `boolean` - Square shape (default: circular)
- `initials`: `string` - Fallback text
- `className`: Size utilities (`size-6`, `size-8`, `size-10`)

**Implementation**:
```tsx
import { Avatar, AvatarButton } from '@/components/ui/avatar'

// Basic avatar
<Avatar 
  src={user.avatarUrl} 
  initials={user.initials}
  className="size-10"
/>

// Avatar group (overlapping)
<div className="flex -space-x-2">
  {users.map((user) => (
    <Avatar 
      key={user.id}
      src={user.avatarUrl}
      className="size-8 ring-2 ring-white dark:ring-zinc-900"
    />
  ))}
</div>

// Interactive avatar
<AvatarButton 
  href={user.profileUrl}
  src={user.avatarUrl}
  square
/>
```

#### Divider
**Purpose**: Visual separation

**Implementation**:
```tsx
import { Divider } from '@/components/ui/divider'

<Divider />
```

## Color System

### Adaptive Colors (3 variants)
Automatically change between light/dark modes:
- `dark/zinc` (default)
- `light`
- `dark/white`

### Solid Colors (20+ options)
Consistent across modes with subtle adjustments:
- Grays: `dark`, `zinc`, `white`
- Warm: `red`, `orange`, `amber`, `yellow`
- Natural: `lime`, `green`, `emerald`, `teal`
- Cool: `cyan`, `sky`, `blue`, `indigo`
- Creative: `violet`, `purple`, `fuchsia`, `pink`, `rose`

## Icon Guidelines

### Size Requirements
- **16×16 pixels**: Buttons, form controls, dropdown items, table cells
- **20×20 pixels**: Navigation items (navbar, sidebar)

### Implementation
```tsx
// Correct icon sizing
import { PlusIcon } from '@heroicons/react/16/solid' // For buttons
import { HomeIcon } from '@heroicons/react/20/solid' // For navigation

// Custom icons need data-slot
<CustomIcon data-slot="icon" className="size-4" />
```

## Accessibility Considerations

### Universal Features
- Full keyboard navigation on all components
- Proper ARIA attributes and roles
- Focus management and trapping in overlays
- Screen reader optimized
- High contrast support
- Respects prefers-reduced-motion

### Component-Specific
- **Forms**: Automatic ID generation and label association
- **Tables**: Proper header associations
- **Dialogs**: Focus trapping and return
- **Navigation**: Active state announcements

## Responsive Behavior

### Mobile Patterns
- **SidebarLayout**: Automatic mobile drawer
- **Tables**: Horizontal scrolling with `--gutter`
- **Navigation**: Responsive utility classes
- **Forms**: Stack on small screens

### Breakpoint Usage
```tsx
// Hide on mobile
<NavbarItem className="max-lg:hidden">Desktop Only</NavbarItem>

// Responsive spacing
<Table className="[--gutter:theme(spacing.6)] sm:[--gutter:theme(spacing.8)]">
```

## State Management Patterns

### Controlled vs Uncontrolled
```tsx
// Uncontrolled (default values)
<Input name="email" defaultValue="user@example.com" />
<Switch name="notifications" defaultChecked />

// Controlled (state management)
const [email, setEmail] = useState('')
const [enabled, setEnabled] = useState(true)

<Input value={email} onChange={(e) => setEmail(e.target.value)} />
<Switch checked={enabled} onChange={setEnabled} />
```

### Form Validation Pattern
```tsx
function FormWithValidation() {
  const [errors, setErrors] = useState(new Map())
  
  return (
    <form onSubmit={handleSubmit}>
      <Field>
        <Label>Email</Label>
        <Input 
          name="email" 
          type="email"
          invalid={errors.has('email')}
        />
        {errors.has('email') && (
          <ErrorMessage>{errors.get('email')}</ErrorMessage>
        )}
      </Field>
    </form>
  )
}
```

## Common Patterns and Best Practices

### Component Composition
```tsx
// Compose smaller components for complex UI
<Field>
  <Label>Payment method</Label>
  <RadioGroup name="payment" defaultValue="card">
    <RadioField>
      <Radio value="card" />
      <Label>Credit card</Label>
      <Description>Pay with credit or debit card</Description>
    </RadioField>
    <RadioField>
      <Radio value="paypal" />
      <Label>PayPal</Label>
      <Description>Pay with your PayPal account</Description>
    </RadioField>
  </RadioGroup>
</Field>
```

### Loading States
```tsx
const [isLoading, setIsLoading] = useState(false)

<Button disabled={isLoading}>
  {isLoading ? 'Processing...' : 'Submit'}
</Button>
```

### Data Tables with Actions
```tsx
<Table>
  <TableBody>
    {items.map((item) => (
      <TableRow key={item.id}>
        <TableCell>{item.name}</TableCell>
        <TableCell>
          <Dropdown>
            <DropdownButton plain>
              <EllipsisHorizontalIcon />
            </DropdownButton>
            <DropdownMenu>
              <DropdownItem href={`/edit/${item.id}`}>Edit</DropdownItem>
              <DropdownItem onClick={() => handleDelete(item.id)}>Delete</DropdownItem>
            </DropdownMenu>
          </Dropdown>
        </TableCell>
      </TableRow>
    ))}
  </TableBody>
</Table>
```

## Common Mistakes to Avoid

### ❌ Incorrect Patterns

1. **Wrong Icon Sizes**
```tsx
// ❌ Wrong
<Button><HomeIcon /></Button> // 20×20 icon in button

// ✅ Correct
<Button><PlusIcon /></Button> // 16×16 icon
```

2. **Missing Field Wrapper**
```tsx
// ❌ Wrong - no accessibility
<Label>Email</Label>
<Input name="email" />

// ✅ Correct - proper association
<Field>
  <Label>Email</Label>
  <Input name="email" />
</Field>
```

3. **Disabled with href**
```tsx
// ❌ Wrong - disabled doesn't work with href
<Button href="/page" disabled>Link</Button>

// ✅ Correct - use onClick or remove disabled
<Button href="/page">Link</Button>
```

4. **Theme Conflicts**
```tsx
// ❌ Wrong - heavily customized theme may break components
tailwind.config.js: {
  theme: {
    extend: {
      spacing: { /* completely different scale */ }
    }
  }
}

// ✅ Correct - extend carefully or adjust components
```

5. **Missing Router Integration**
```tsx
// ❌ Wrong - using plain <a> tags
<Button href="/page">Navigate</Button> // Won't use client routing

// ✅ Correct - update Link component for your framework
// See Framework Integration section
```

## TypeScript Support

### Type Patterns
```typescript
// Component prop types
interface ButtonProps extends React.ComponentPropsWithoutRef<'button'> {
  color?: ColorOption
  outline?: boolean
  plain?: boolean
  disabled?: boolean
}

// Using with TypeScript
const MyComponent: React.FC<{ user: User }> = ({ user }) => {
  const [isOpen, setIsOpen] = useState<boolean>(false)
  
  return (
    <Dialog open={isOpen} onClose={setIsOpen}>
      {/* content */}
    </Dialog>
  )
}

// Form handling with types
interface FormData {
  email: string
  name: string
}

const handleSubmit = (data: FormData) => {
  // type-safe form handling
}
```

## Theming and Customization

### Direct Modification Approach
```tsx
// Components are in your codebase - modify directly
// components/button.tsx
export function Button({ className, ...props }) {
  return (
    <button 
      className={clsx(
        'rounded-lg px-4 py-2', // Modify these classes directly
        className
      )}
      {...props}
    />
  )
}
```

### Dark Mode
All components include automatic dark mode support:
```tsx
// Automatic dark mode adaptation
<div className="bg-white dark:bg-zinc-900">
  <Button color="blue">Adapts automatically</Button>
</div>
```

### Custom Styling
```tsx
// Add custom classes
<Input className="max-w-xs border-2 border-blue-500" />
<Button className="rounded-full shadow-xl">Custom styled</Button>
```

## Component Evolution and Versioning

### Current Status
- Stable release with Headless UI v2.0
- Regular component additions
- Free updates with purchase

### Upcoming Components
- Command palettes
- Tooltips
- Date pickers
- Enhanced comboboxes

### Update Strategy
- Re-download latest version
- Compare with your customized components
- Merge desired updates

## Integration with React/Next.js

### Next.js App Router
```typescript
// app/components/link.tsx
'use client'

import * as Headless from '@headlessui/react'
import NextLink from 'next/link'
import { forwardRef } from 'react'

export const Link = forwardRef(function Link(props, ref) {
  return (
    <Headless.DataInteractive>
      <NextLink {...props} ref={ref} />
    </Headless.DataInteractive>
  )
})
```

### Remix Integration
```typescript
// app/components/link.tsx
import * as Headless from '@headlessui/react'
import { Link as RemixLink } from '@remix-run/react'
import { forwardRef } from 'react'

export const Link = forwardRef(function Link(props, ref) {
  return (
    <Headless.DataInteractive>
      <RemixLink {...props} ref={ref} />
    </Headless.DataInteractive>
  )
})
```

## How Components Work Together

### Complete Application Example
```tsx
function Application() {
  return (
    <SidebarLayout
      sidebar={
        <Sidebar>
          <SidebarBody>
            <SidebarSection>
              <SidebarItem href="/" current>
                <HomeIcon />
                <SidebarLabel>Dashboard</SidebarLabel>
              </SidebarItem>
            </SidebarSection>
          </SidebarBody>
        </Sidebar>
      }
      navbar={
        <Navbar>
          <NavbarSection>
            <NavbarItem href="/">Home</NavbarItem>
          </NavbarSection>
        </Navbar>
      }
    >
      <div className="p-8">
        <Heading>Users</Heading>
        <Table>
          <TableHead>
            <TableRow>
              <TableHeader>Name</TableHeader>
              <TableHeader>Status</TableHeader>
              <TableHeader>Actions</TableHeader>
            </TableRow>
          </TableHead>
          <TableBody>
            {users.map(user => (
              <TableRow key={user.id}>
                <TableCell>{user.name}</TableCell>
                <TableCell>
                  <Badge color={user.active ? 'green' : 'zinc'}>
                    {user.active ? 'Active' : 'Inactive'}
                  </Badge>
                </TableCell>
                <TableCell>
                  <Button size="sm" onClick={() => editUser(user)}>
                    Edit
                  </Button>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
        <Pagination className="mt-6">
          <PaginationPrevious href="?page=1" />
          <PaginationList>
            <PaginationPage href="?page=1">1</PaginationPage>
            <PaginationPage href="?page=2" current>2</PaginationPage>
          </PaginationList>
          <PaginationNext href="?page=3" />
        </Pagination>
      </div>
    </SidebarLayout>
  )
}
```

## Performance Optimization

### Best Practices
1. **Lazy Load Dialogs**: Import dialogs only when needed
2. **Virtualize Large Lists**: Combobox includes built-in virtualization
3. **Optimize Icons**: Use correct sizes to prevent layout shifts
4. **Minimize Re-renders**: Use controlled components judiciously

### Bundle Size Management
```typescript
// Only import what you need
import { Button } from '@/components/ui/button'
// Not: import * from '@/components'
```

## Conclusion

Catalyst UI represents a mature, production-ready component system that prioritizes developer ownership and customization. By following this guide, AI models can correctly implement Catalyst components with proper typing, accessibility, and responsive behavior. Remember that components are designed to be modified directly in your codebase, making them truly yours over time.

Key principles for AI implementation:
1. Always use correct icon sizes (16×16 vs 20×20)
2. Wrap form controls in Field components
3. Use semantic HTML patterns (Fieldset, Legend, etc.)
4. Implement proper TypeScript types
5. Follow the compositional API pattern
6. Respect the framework's routing requirements
7. Maintain accessibility standards
8. Apply responsive utilities appropriately

This guide provides the complete technical specification needed to accurately generate Catalyst UI component implementations while avoiding common pitfalls and maintaining best practices.