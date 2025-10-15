# Mobile Responsive Design Implementation

This document describes the mobile responsive design improvements made to the OpenArchiver Svelte frontend.

## Overview

The frontend has been updated to provide an excellent mobile experience with proper responsive breakpoints, collapsible navigation, and optimized layouts for small screens.

## Key Features

### 1. Mobile Navigation Sheet

A new Sheet component has been added for mobile drawer/sidebar functionality:

- **Component Location**: `packages/frontend/src/lib/components/ui/sheet/`
- **Based on**: bits-ui Dialog primitive
- **Features**:
  - Slide-in animations from left/right/top/bottom
  - Backdrop overlay
  - Close button
  - Fully accessible

### 2. Dashboard Navigation

The main dashboard navigation has been optimized for mobile:

- **Desktop (lg+ screens)**: Traditional horizontal navigation menu
- **Mobile (<lg screens)**: 
  - Hamburger menu button in header
  - Full-screen navigation drawer
  - Collapsible sub-menus
  - Logout button integrated into mobile menu

### 3. Archived Emails Page

The archived emails page with folder sidebar is now fully responsive:

#### Desktop View (md+ screens)
- Sidebar visible on the left (1/4 width)
- Main content area on the right (3/4 width)
- All table columns visible

#### Mobile View (<md screens)
- Sidebar hidden by default
- Hamburger menu button to open folder drawer
- Table with horizontal scroll
- Responsive column visibility:
  - Date column: Shows short date format
  - Subject: Always visible with sender as subtitle
  - Sender: Hidden on mobile (shown under subject)
  - Inbox: Hidden on small screens (visible from md+)
  - Path: Hidden on mobile (visible from lg+)
- Compact action buttons

### 4. Responsive Breakpoints

The application uses Tailwind CSS responsive breakpoints:

- `sm`: 640px - Small devices (large phones)
- `md`: 768px - Medium devices (tablets)
- `lg`: 1024px - Large devices (desktops)
- `xl`: 1280px - Extra large screens

## Implementation Details

### Sheet Component Structure

```svelte
<Sheet.Root bind:open={isOpen}>
  <Sheet.Content side="left">
    <Sheet.Header>
      <Sheet.Title>Menu</Sheet.Title>
    </Sheet.Header>
    <!-- Content here -->
  </Sheet.Content>
</Sheet.Root>
```

### Responsive Grid Example

```svelte
<!-- Container that adapts to screen size -->
<div class="md:grid md:grid-cols-4 md:gap-6">
  <!-- Sidebar: hidden on mobile, visible from md+ -->
  <aside class="hidden md:block md:col-span-1">
    <!-- Sidebar content -->
  </aside>
  
  <!-- Main content: full width on mobile, 3/4 on desktop -->
  <main class="md:col-span-3">
    <!-- Main content -->
  </main>
</div>
```

### Responsive Table Columns

```svelte
<Table.Head class="hidden sm:table-cell">
  Sender
</Table.Head>
```

## Mobile User Experience

### Navigation Flow
1. User opens the app on mobile
2. Hamburger menu button is visible in the header
3. Tapping the hamburger opens a full-screen navigation drawer
4. User can navigate to different sections
5. Selecting a menu item closes the drawer automatically

### Folder Navigation (Archived Emails)
1. User navigates to Archived Emails page
2. Folder menu button is visible next to the page title
3. Tapping the button opens the folder tree drawer
4. User can select a folder
5. Drawer closes and emails are filtered by the selected folder

### Table Interaction
1. Tables are optimized for readability on small screens
2. Less important columns are hidden on mobile
3. Critical information is always visible
4. Horizontal scroll available for overflow content
5. Action buttons are sized appropriately for touch

## Testing Recommendations

To test the responsive design:

1. **Browser DevTools**:
   - Open Chrome/Firefox DevTools
   - Toggle device toolbar (Cmd/Ctrl + Shift + M)
   - Test various device sizes (iPhone, iPad, etc.)

2. **Key Screens to Test**:
   - Dashboard home page
   - Archived Emails page (with folders)
   - Email detail view
   - Settings pages

3. **Critical Breakpoints**:
   - 375px (iPhone SE)
   - 640px (small tablet)
   - 768px (iPad)
   - 1024px (desktop)

4. **Interactions to Verify**:
   - Hamburger menu opens/closes smoothly
   - Folder drawer appears/disappears correctly
   - Table columns show/hide at proper breakpoints
   - Touch targets are appropriately sized (minimum 44px)
   - Text remains readable at all sizes

## Future Improvements

Potential enhancements for mobile experience:

1. **Swipe Gestures**: Add swipe-to-close for drawers
2. **Pull-to-Refresh**: Implement pull-to-refresh on lists
3. **Bottom Navigation**: Consider bottom nav for frequent actions on mobile
4. **Improved Touch Targets**: Ensure all interactive elements meet accessibility guidelines
5. **Performance**: Optimize for slower mobile connections with lazy loading

## Browser Support

The responsive design works on:
- Modern mobile browsers (iOS Safari, Chrome, Firefox)
- Tablet browsers
- Desktop browsers with responsive testing tools

Minimum supported versions:
- iOS Safari 12+
- Chrome/Edge 88+
- Firefox 85+
- Samsung Internet 13+
