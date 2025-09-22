# Style Guide Document

**Document Version:** 1.0  
**Date Created:** September 15, 2025  
**Author(s):** AI-Generated  
**Project Name:** Guest Link - A WhatsApp-based Hospitality App

---

## 1. Introduction

### Purpose
This Style Guide Document establishes the visual and stylistic standards for Guest Link, a WhatsApp-integrated hospitality management platform. It defines the design system that ensures a cohesive, professional, and user-friendly experience across all interfaces, from welcome pages to staff dashboards. The guide supports Vibe Coding by providing clear specifications for AI-generated UI components while maintaining brand consistency and accessibility standards.

### Scope
- Visual identity (themes, colors, typography)
- Layout principles and responsive design
- Component library specifications
- Animation and interaction patterns
- Accessibility compliance (WCAG 2.1 AA)

### Assumptions
- Next.js 13+ with App Router as the framework
- Tailwind CSS for styling
- Mobile-first responsive design
- Light and dark theme support
- WhatsApp-inspired messaging components

---

## 2. Visual Identity

### 2.1 Themes

#### Light Theme (Default)
- Clean, minimalist design emphasizing readability
- White backgrounds with subtle shadows
- High contrast for text and interactive elements
- Accent colors for key actions and status indicators

#### Dark Theme
- Optimized for low-light environments
- Deep backgrounds with reduced eye strain
- Maintained readability and contrast ratios
- Consistent branding across themes

### 2.2 Color Palette

| Color Role     | Light Theme | Dark Theme  | Usage                  | Accessibility |
|----------------|-------------|-------------|------------------------|---------------|
| Primary        | #2563EB     | #3B82F6    | CTAs, links, focus    | WCAG AA+ |
| Secondary      | #4B5563     | #9CA3AF    | Supporting elements    | WCAG AA  |
| Background     | #FFFFFF     | #111827    | Page backgrounds      | Base     |
| Surface        | #F3F4F6     | #1F2937    | Cards, modals         | Base     |
| Text Primary   | #111827     | #F9FAFB    | Main content          | WCAG AAA |
| Text Secondary | #4B5563     | #D1D5DB    | Supporting text       | WCAG AA  |
| Border         | #E5E7EB     | #374151    | Dividers, inputs      | N/A      |
| Success        | #059669     | #10B981    | Confirmation states   | WCAG AA  |
| Warning        | #D97706     | #F59E0B    | Alerts, warnings      | WCAG AA  |
| Error          | #DC2626     | #EF4444    | Error states          | WCAG AA  |

**Implementation:**
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: '#2563EB',
          dark: '#3B82F6'
        },
        // ... other color definitions
      }
    }
  }
}
```

### 2.3 Typography

#### Font Families
- Primary: Inter (Sans-serif)
- System Fallback: -apple-system, BlinkMacSystemFont, system-ui

#### Type Scale

| Element          | Size           | Weight | Line Height | Use Case            |
|-----------------|----------------|---------|-------------|-------------------|
| Display         | 48px / 3rem    | 700     | 1.2         | Hero sections      |
| H1              | 36px / 2.25rem | 700     | 1.2         | Page titles        |
| H2              | 30px / 1.875rem| 600     | 1.3         | Section headers    |
| H3              | 24px / 1.5rem  | 600     | 1.4         | Card titles        |
| Body Large      | 18px / 1.125rem| 400     | 1.5         | Featured content   |
| Body            | 16px / 1rem    | 400     | 1.6         | Main content       |
| Small           | 14px / 0.875rem| 400     | 1.5         | Supporting text    |
| Caption         | 12px / 0.75rem | 400     | 1.5         | Meta information   |

**Implementation:**
```typescript
// Tailwind utility classes
const typography = {
  'display': 'text-5xl font-bold leading-tight',
  'h1': 'text-4xl font-bold leading-tight',
  'h2': 'text-3xl font-semibold leading-snug',
  'body': 'text-base leading-relaxed',
  // ... other definitions
}
```

---

## 3. Layout and Grid System

### 3.1 Grid Structure
- 12-column grid system
- Container max-width: 1280px
- Gutters: 16px (small), 24px (medium), 32px (large)

### 3.2 Breakpoints

| Name    | Size (px) | Tailwind    | Use Case                    |
|---------|-----------|-------------|----------------------------|
| Mobile  | < 640     | Default     | Single column layouts      |
| Tablet  | ≥ 640     | sm:         | Two column layouts         |
| Laptop  | ≥ 1024    | lg:         | Multi-column dashboard     |
| Desktop | ≥ 1280    | xl:         | Expanded features         |

### 3.3 Spacing Scale

| Size    | Pixels | Tailwind | Use Case                    |
|---------|--------|----------|----------------------------|
| xs      | 4px    | p-1      | Tight internal spacing     |
| sm      | 8px    | p-2      | Button padding             |
| md      | 16px   | p-4      | Card padding              |
| lg      | 24px   | p-6      | Section spacing           |
| xl      | 32px   | p-8      | Page section gaps         |
| 2xl     | 48px   | p-12     | Hero sections             |

---

## 4. Component Examples

### 4.1 Buttons

#### Primary Button
```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'outline';
  size: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  isLoading?: boolean;
}

const buttonStyles = {
  primary: `
    bg-primary text-white
    hover:bg-primary-dark
    focus:ring-2 focus:ring-primary/50
    disabled:opacity-50
    rounded-lg font-medium
    transition-colors
  `,
  sizes: {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  }
}
```

### 4.2 Cards

#### Welcome Page Card
```typescript
const welcomeCardStyles = `
  bg-white dark:bg-gray-800
  shadow-lg rounded-xl
  p-6 md:p-8
  border border-gray-100 dark:border-gray-700
  transition-shadow hover:shadow-xl
`
```

#### Service Request Card
```typescript
const serviceCardStyles = `
  bg-white dark:bg-gray-800
  p-4 rounded-lg
  border-l-4 border-primary
  hover:bg-gray-50 dark:hover:bg-gray-700
  transition-colors
  cursor-pointer
`
```

### 4.3 Forms

#### Input Field
```typescript
const inputStyles = `
  w-full
  px-4 py-2
  border border-gray-200 dark:border-gray-700
  rounded-lg
  focus:ring-2 focus:ring-primary/50 focus:border-primary
  bg-white dark:bg-gray-800
  text-gray-900 dark:text-white
  placeholder-gray-400 dark:placeholder-gray-500
  transition-colors
`
```

#### Select Field
```typescript
const selectStyles = `
  w-full
  appearance-none
  bg-white dark:bg-gray-800
  border border-gray-200 dark:border-gray-700
  rounded-lg
  px-4 py-2
  pr-8
  focus:ring-2 focus:ring-primary/50
  focus:border-primary
`
```

### 4.4 Navigation

#### Top Navigation Bar
```typescript
const navStyles = `
  fixed top-0 w-full
  bg-white dark:bg-gray-800
  border-b border-gray-200 dark:border-gray-700
  px-4 md:px-6
  h-16
  flex items-center
  justify-between
  z-50
`
```

#### Sidebar Navigation
```typescript
const sidebarStyles = `
  fixed left-0 top-16
  w-64 h-[calc(100vh-4rem)]
  bg-white dark:bg-gray-800
  border-r border-gray-200 dark:border-gray-700
  overflow-y-auto
  transition-transform
`
```

### 4.5 Message Components

#### Chat Bubble
```typescript
const chatBubbleStyles = {
  guest: `
    bg-gray-100 dark:bg-gray-700
    rounded-t-lg rounded-br-lg
    ml-4 mr-12
    p-4
    relative
  `,
  host: `
    bg-primary text-white
    rounded-t-lg rounded-bl-lg
    mr-4 ml-12
    p-4
    relative
  `
}
```

---

## 5. Tools and Implementation

### 5.1 Development Tools
- Storybook for component documentation
- Tailwind CSS for styling
- CSS Modules for component-specific styles
- PostCSS for processing

### 5.2 Asset Management
- SVG icons with configurable colors
- Optimized images via next/image
- Font loading via next/font

### 5.3 Accessibility Tools
- ARIA attributes implementation
- Color contrast checker
- Screen reader testing
- Keyboard navigation support

### 5.4 Code Generation
```typescript
// Example AI prompt for component generation
"Generate a React component for a service request card using these Tailwind classes:
bg-white dark:bg-gray-800 p-4 rounded-lg border-l-4 border-primary
Include props for service name, status, and timestamp."
```

---

## 6. Animation Guidelines

### Micro-interactions
- Hover states: 150ms transition
- Button clicks: Scale transform
- Loading states: Subtle pulse
- Form feedback: Gentle shake for errors

### Page Transitions
- Route changes: Fade transition
- Modal: Scale and fade
- Sidebar: Slide from left
- Toast notifications: Slide up

```typescript
const transitions = {
  hover: 'transition-all duration-150 ease-in-out',
  page: 'transition-opacity duration-200 ease-in-out',
  modal: 'transition-all duration-300 ease-in-out'
}
```

---

## 7. Responsive Design Guidelines

### Mobile-First Approach
- Design for mobile screens first
- Progressive enhancement for larger screens
- Touch-friendly tap targets (min 44x44px)
- Simplified navigation patterns

### Responsive Patterns
- Stack layouts on mobile
- Side-by-side on tablet+
- Full dashboard on desktop
- Conditional rendering of complex features

---

## 8. Revision History
- September 15, 2025 - v1.0 - Initial style guide document
- Future updates will be logged here

---

**Document Purpose Recap:** This Style Guide Document establishes a comprehensive design system for Guest Link, ensuring visual consistency and maintainable code across all interfaces while supporting efficient development through Vibe Coding with clear component specifications and accessibility standards.