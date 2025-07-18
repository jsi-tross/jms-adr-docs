# ADR-0012: Wireframe Layout Architecture

## Status
Accepted

## Context
The JMS system needed a consistent and maintainable approach to implementing wireframe designs across all application pages. The initial implementation had several issues:

- Inconsistent spacing and layout patterns across different pages
- Hardcoded color values that didn't integrate with the theme system
- Mixed use of Material-UI DataGrid and standard tables without clear guidelines
- Lack of reusable layout components leading to code duplication
- No centralized approach to managing layout constants

These issues were causing:
- Visual inconsistencies across the application
- Difficulty in maintaining and updating the UI
- Poor theme integration preventing effective light/dark mode support
- Increased development time due to recreating similar layouts
- Challenges in ensuring responsive design consistency

## Decision
We have implemented a comprehensive wireframe layout architecture with the following key decisions:

1. **Reusable Layout Components**:
   - Created `PageContainer` component for consistent page-level layout with proper spacing
   - Implemented `PageHeader` component for standardized page titles and descriptions
   - Developed `CardLayout` component for consistent card-based content sections

2. **Centralized Layout Constants**:
   - Created `layoutConstants.js` to define all spacing, sizing, and layout values
   - Established consistent spacing units (xs: 8px, sm: 16px, md: 24px, lg: 32px, xl: 48px)
   - Defined standard component heights and spacing patterns
   - Centralized responsive breakpoint configurations

3. **Theme Integration**:
   - Removed all hardcoded color values (backgroundColor, borderColor, etc.)
   - Utilized Material-UI theme system for all colors
   - Properly integrated with light/dark mode theming
   - Used semantic color tokens (e.g., `theme.palette.background.paper`)

4. **Table Design Standardization**:
   - Replaced Material-UI DataGrid with standard HTML tables for simpler use cases
   - Used consistent table styling patterns across all pages
   - Implemented proper theme-aware table borders and backgrounds
   - Standardized table cell padding and alignment

5. **Consistent Component Structure**:
   - Established pattern: PageContainer > PageHeader > Content Cards
   - Used Material-UI Grid system for responsive layouts
   - Maintained consistent card elevation and spacing
   - Implemented uniform button placement and styling

## Consequences

### Positive
- **Consistency**: All pages now follow the same layout patterns and spacing
- **Maintainability**: Changes to layout constants automatically apply across all pages
- **Theme Support**: Full light/dark mode support without hardcoded colors
- **Developer Experience**: Reusable components reduce development time
- **Code Quality**: Eliminated code duplication across page implementations
- **Responsive Design**: Centralized breakpoints ensure consistent mobile experience
- **Visual Hierarchy**: Clear and consistent information architecture

### Negative
- **Migration Effort**: Existing pages need to be refactored to use new components
- **Learning Curve**: Developers need to understand the new component system
- **Initial Overhead**: Setting up the layout system required upfront investment

### Neutral
- **Component Library**: Created internal component library that needs documentation
- **Design System**: Moves toward a more formal design system approach
- **Testing Updates**: Component tests need to be updated for new structure

## Alternatives Considered

1. **Page-Specific Layouts**:
   - Rejected: Would lead to inconsistency and code duplication
   - Each page would need to reimplement common patterns

2. **CSS Framework (Bootstrap, Tailwind)**:
   - Rejected: Would conflict with Material-UI's styling approach
   - Adds unnecessary dependency and complexity

3. **Styled Components for Everything**:
   - Rejected: Material-UI's sx prop provides sufficient styling capability
   - Would increase bundle size and complexity

4. **Keep DataGrid for All Tables**:
   - Rejected: DataGrid is overkill for simple, static tables
   - Adds unnecessary complexity and performance overhead
   - Standard tables are more accessible and easier to customize

5. **Inline Styles and Values**:
   - Rejected: Makes maintenance difficult and prevents theming
   - Creates inconsistencies across the application

## References
- [Material-UI Theming Documentation](https://mui.com/material-ui/customization/theming/)
- [Material-UI Layout Components](https://mui.com/material-ui/react-box/)
- [ADR-0011: Admin Navigation Architecture](ADR-0011-admin-navigation-architecture.md) - Uses the same layout patterns
- [React Composition Patterns](https://reactjs.org/docs/composition-vs-inheritance.html)

## Notes
- The layout system is designed to be extensible for future requirements
- All spacing values are based on an 8px grid system for visual harmony
- Components should be documented with prop types and usage examples
- Consider creating a Storybook instance to showcase layout components
- Future enhancement: Add layout variants for different page types (forms, dashboards, etc.)
- Performance: Layout components use React.memo where appropriate for optimization
- Accessibility: All layout components maintain proper heading hierarchy and ARIA landmarks

### Implementation Details
The key files in the layout system:
- `src/components/layout/PageContainer.js` - Main page wrapper
- `src/components/layout/PageHeader.js` - Consistent page headers
- `src/components/layout/CardLayout.js` - Card-based content sections
- `src/constants/layoutConstants.js` - All layout-related constants

### Migration Guide
When updating existing pages:
1. Wrap the page content in `PageContainer`
2. Replace custom headers with `PageHeader`
3. Use `CardLayout` for content sections
4. Replace hardcoded values with `layoutConstants`
5. Remove all hardcoded colors
6. Update tables to use standard HTML table structure

---
**Date**: 2025-07-18  
**Author**: AI Assistant  
**Reviewers**: Pending