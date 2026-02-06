# Document Ingestion Prompt Management - UI Design

## Overview

This document describes the user interface design for managing prompt text that can be assigned to specific scopes within the Document Ingestion system. The interface allows users to configure custom prompt text overrides at different organizational levels (Team, Segment, Product) while viewing global defaults.

The design follows a phased approach, with an **MVP (Minimum Viable Product)** providing core functionality, followed by **Future Enhancements** that add advanced features.

## Background

The system currently stores prompt text configurations in the `distribution.DocumentIngestionScopedPromptContext` table. This new interface will provide a user-friendly way to manage these configurations without requiring direct database access.

---

# MVP Design (Phase 1)

The MVP focuses on delivering the essential functionality with a clean, intuitive interface that emphasizes:
- Visual hierarchy navigation over dropdown filters
- Context-aware display showing where users are working
- Simplified workflow with minimal UI clutter
- Clear separation of read-only defaults and editable overrides

## User Interface Structure

The MVP interface consists of two tabs:

1. **Edit Prompts** (Scope-Based View): Core editing interface with tree navigation
2. **Review and Publish**: Review pending changes before publishing

---

## Tab 1: Edit Prompts (Scope-Based View)

### Purpose
Allow users to select a specific scope using a visual tree and manage prompt text for all elements within that scope. This is the primary working area for prompt management.

### Design Decisions

**Why Tree Navigation Instead of Dropdowns?**
- **Visual Hierarchy**: Tree view makes organizational structure immediately visible
- **Reduced Cognitive Load**: Users can see all available scopes at once without opening multiple dropdowns
- **Clearer Context**: Current selection is always visible with visual highlighting
- **Easier Navigation**: Single click to select scope vs. multiple dropdown interactions
- **Expandable/Collapsible**: Users can focus on relevant portions of the hierarchy

**Why Streamlined Inline Editing?**
- **Direct Workflow**: Click directly into cells to edit without selection step
- **Cleaner Interface**: No selection controls cluttering the grid
- **Immediate Feedback**: Visual indicators show edited cells instantly
- **Focused Experience**: Each edit is intentional and visible

**Why Context-Aware Headers?**
- **Orientation**: Users always know which scope they're editing
- **Reduced Errors**: Clear headers prevent editing wrong scope by mistake
- **Scope Path Display**: Full path (e.g., "North America > Middle Market") reinforces hierarchy

### Layout

**Left Panel: Scope Tree Navigation**
- Hierarchical tree displaying organizational structure
- Three top-level nodes (not nested under each other):
  - **Global**: Top-level global scope (no children)
  - **North America**: Contains Middle Market and L&C segments
  - **CRB**: Contains GB segment
- Expandable/collapsible nodes with arrow icons (▼ expanded, ▶ collapsed)
- Click on node text to select scope
- Click on arrow to expand/collapse (separate from selection)
- Active selection visually highlighted
- Hover state for visual feedback

**Why This Scope Structure?**
- Reflects actual organizational hierarchy at WTW
- Global is peer-level with regions (not parent) because it represents baseline, not organizational parent
- Segments nested under their respective regions for logical grouping
- Extensible: More regions/segments can be added to tree easily

**Right Panel: Element Grid**
- Displays hierarchical table of elements relevant to the selected scope
- Each row represents an element type that can have prompt text configured
- Grid organized hierarchically with expand/collapse for grouped elements
- Element list is contextual: only shows elements applicable to the current scope selection

**Top Bar: Product Selector (Single Dropdown)**
- **Product dropdown**: Select a product (Cyber, Property, Casualty, D&O, E&O) to add product-specific prompt overrides
- Selecting a product shows the Product Prompt column and updates its header with scope context
- Selecting "[None]" hides Product Prompt column, showing only Global and Team-level prompts
- Product selection enables the most specific level of prompt override (Team + Product scope)

**Prompt Override Hierarchy**:
- **Global**: Baseline prompts that apply everywhere unless overridden
- **Team Scope**: Overrides global prompts for the selected team/segment
- **Team + Product Scope**: Most specific override; takes precedence over both Global and Team prompts
- Each level overrides the previous, with Team + Product being the final override

**Why Product Selector?**
- **Scope selection via tree**: Team/Segment selection handled by tree navigation (more intuitive)
- **Enables granular overrides**: Product selector allows creation of highly specific prompts within a scope
- **Clear hierarchy**: Product is the most specific override level, naturally fits as selector rather than navigation
- **Clean interface**: Single dropdown keeps UI uncluttered while enabling powerful customization

### Grid Columns

The grid uses **6 columns** (no checkbox column):

**Column 1: Element**
- Lists all element types in the system that support prompt configuration
- Displayed in hierarchical tree structure with indentation
- Examples: Response Type, Market Securities, Premium/Cost Component (with children)
- Expand/collapse icons (▼/▶) for parent elements

**Column 2: Currently Applied Prompt (Read-Only)**
- Shows the actual prompt text that is currently in effect for this element in the selected scope
- Displays the resolved prompt after applying the cascade logic (Team+Product → Team → Global)
- Read-only display to help users understand what prompt is currently being used
- Updates dynamically based on scope and product selection

**Column 3: Source**
- Displays a badge indicating which level is providing the currently applied prompt
- Badge values: "Global", "Team", "Product"
- **Dynamic behavior**: Updates based on which override level is active
- Helps users quickly identify where the current prompt originates
- Read-only display

**Column 4: Global Prompt (Read-Only)**
- Displays the default prompt text from `distribution.DocumentIngestionPrompt`
- Read-only column
- Shows the base configuration that applies when no overrides exist
- Helps users understand what they are overriding

**Column 5: Team Prompt (Editable)**
- **Dynamic header**: Shows selected scope path + "Prompt (Editable)"
  - Example: "North America > Middle Market Prompt (Editable)"
- Editable field for the currently selected scope
- Allows users to override the global prompt text for this team/segment
- If empty, the global prompt applies
- **Conditional visibility**: Hidden when Global scope is selected (no team overrides possible at global level)
- Data stored in `distribution.DocumentIngestionScopedPromptContext` with `ScopeId` and `ElementTypeId`

**Why Hide Team Column at Global Scope?**
- **Logical consistency**: Can't have team-specific overrides at global level
- **Reduces confusion**: Hiding impossible options prevents user errors
- **Cleaner interface**: Only shows relevant columns for current context

**Column 6: Product Prompt (Editable)**
- **Dynamic header**: Shows selected scope path + "Product Prompt (Editable)"
  - Example: "North America > L&C Product Prompt (Editable)"
- Editable field for product-specific overrides within the selected scope
- Initially hidden; visible when product selected from dropdown
- If empty, falls back to Team Scoped prompt, then Global prompt
- Data stored in `distribution.DocumentIngestionScopedPromptContext` with `ScopeId`, `ElementTypeId`, and `ProductId`

**Why Dynamic Product Header?**
- **Scope awareness**: Products exist within scopes, not independently
- **Prevents confusion**: Users know they're editing "North America Cyber", not just "Cyber"
- **Consistency**: Matches Team Prompt header pattern

### Changes Banner

**Unsaved Changes Indicator**
- Banner appears at top when any edits are made
- Shows count of unpublished changes
- Provides "Discard" and "Review & Publish" buttons
- Tab badge shows change count: "Review and Publish (3)"

**Why This Approach?**
- **Clear feedback**: Users always know if they have unsaved work
- **Prevents data loss**: Prominent banner prevents accidental navigation away
- **Quick actions**: Buttons provide immediate path to review or discard

### Key Features

**Hierarchy Display**
- Elements displayed in tree structure matching their relationships
- Users can expand/collapse parent elements (e.g., Premium/Cost Component)
- Visual indentation shows parent-child relationships
- Expand icons (▼/▶) toggle visibility of children

**Inheritance Model**
- Prompts follow cascade: Global → Team → Product
- More specific scopes override less specific ones
- Empty values inherit from next level up
- Currently Applied Scopes column shows which level is active

**Inline Editing**
- Click into any editable cell to begin typing
- Changes tracked immediately
- Edited cells marked to indicate pending changes

**Scope Context Awareness**
- Tree shows current selection visually highlighted
- Headers update to show scope path
- Team column hides at Global scope
- Product column header shows scope + product context

---

## Tab 2: Review and Publish

### Purpose
Review all pending changes in a consolidated view before publishing them to become active in the document ingestion system.

**MVP Status**: Displays pending changes in hierarchical format with Publish/Discard actions

**Why Separate Review Tab?**
- **Staged workflow**: Allows review before changes go live
- **Batch publishing**: See all changes at once before committing
- **Audit clarity**: Clear separation between editing and publishing
- **Error prevention**: Chance to review and catch mistakes before affecting production

---

## MVP User Workflows

### Scenario 1: Add Team-Specific Prompt
1. User clicks "North America > Middle Market" in tree navigation
2. Tree node highlights in blue, headers update to show "North America > Middle Market Prompt (Editable)"
3. User scrolls to "Quote Expiry Date" element
4. Clicks into Team Prompt column cell
5. Types: "For North America, use MM/DD/YYYY format"
6. Clicks outside cell to save
7. Cell gets yellow-orange border indicating pending change
8. Changes banner appears: "You have 1 unpublished change"
9. User clicks "Review & Publish" button
10. Navigates to Review and Publish tab
11. Reviews change and clicks "Publish All"
12. Prompt now applies to Middle Market team for that element

### Scenario 2: Add Product-Specific Prompt Within Scope
1. User selects "North America > L&C" from tree
2. Selects "Cyber" from Product dropdown
3. Product Prompt column appears with header "North America > L&C Product Prompt (Editable)"
4. User navigates to "Response Type" element
5. Enters prompt in Product Prompt column: "For cyber risks, identify..."
6. Change tracked in banner
7. Reviews and publishes
8. Prompt applies only for Cyber products within North America L&C

### Scenario 3: Navigate Scope Hierarchy
1. User sees tree with "North America" expanded (▼ icon)
2. Clicks arrow next to "CRB" to expand
3. Tree expands showing "GB" child node
4. Clicks "GB" text to select it
5. Tree highlights "GB", headers update to "CRB > GB Prompt (Editable)"
6. Grid shows elements with any existing GB-specific prompts

### Scenario 4: Dynamic Scope Badge Behavior
1. User selects "North America" scope (no product selected)
2. "Market Securities" element has empty Team Prompt column
3. Currently Applied Scopes shows "Global" badge (inherited from Global Prompt column)
4. User types text into Team Prompt column for "Quote Expiry Date"
5. Currently Applied Scopes for that element updates to show "Team" badge
6. User selects "Property" from Product dropdown
7. Product Prompt column appears
8. Original scope badges restore (showing "Global", "Team", "Product" as configured)

### Scenario 5: Remove Override
1. User selects scope with existing team prompt
2. Clicks into Team Prompt cell and deletes all text
3. Presses outside cell
4. Change tracked (deletion is a change)
5. Currently Applied Scopes updates to show "Global" (fallback)
6. User publishes change
7. System removes row from database or marks as deprecated
8. Element falls back to Global prompt

---

# Future Enhancements (Post-MVP)

The following features can be added in subsequent phases to enhance the prompt management experience:

---

## Elements to Scopes View (Cross-Scope Element Management)

### Purpose
This future tab will provide a cross-scope view of all elements, allowing users to see and manage prompt configurations for any element across all their accessible scopes.

### Advanced Features

**Full Cross-Scope Editing**
- Edit prompts for any element across all user's scopes in single view
- Side-by-side comparison of prompt text across scopes
- Identify inconsistencies or duplications

**Grid Structure Enhancement**

**Group Rows: Element Names**
- Top-level expandable rows showing each element type
- Shows aggregate information (how many scopes have overrides)

**Child Rows: Scope Configurations**
- Each child row represents specific scope where element has prompt override
- Shows both Team-only scopes and Team+Product scopes
- Multiple rows per element if configured in multiple scopes

**Additional Columns for Enhanced View**
1. **Element/Scope Name**: Group rows show element; children show scope path
2. **Scope Type**: "Global", "Team", "Team + Product"
3. **Product**: Shows product name if Team+Product scope
4. **Prompt Text**: Editable field
5. **Source**: "Global Default", "Team Override", "Product Override"
6. **Last Modified**: Timestamp and user who made last change

**Advanced Filtering**
- Filter by specific elements
- Search for specific prompt text content
- Filter by scope or product
- Show only overridden vs. only inherited
- Filter by modification date or author

---

## Advanced Publishing Workflow

### Granular Publication Options

**Per Element Publishing**
- Publish changes for single element only across all scopes
- Use case: Targeted fix for one element

**Per Element Group Publishing**
- Publish all changes within expandable element group
- Example: Publish all "Premium/Cost Component" children
- Use case: Related changes that should go live together

**Multiselect Publishing**
- Re-introduce checkboxes for selection (not always visible, only in publish mode)
- Select multiple specific elements across different groups
- Publish selected subset
- Use case: Cherry-pick specific changes to publish

**Scope-Based Publishing**
- Publish all changes for currently selected scope
- Example: Publish all "North America > Middle Market" changes
- Use case: Rollout changes for one team at a time

**Scheduled Publishing**
- Set future date/time for changes to go live
- Useful for coordinated rollouts
- Can cancel scheduled publications before they execute

### Enhanced Review Interface

**Change Comparison View**
- Side-by-side diff showing before/after for each change
- Highlight additions/deletions within prompt text
- Group changes by element or scope for easier review

**Validation Before Publishing**
- Check for empty prompts that might cause fallback issues
- Warn if global prompt is being overridden with very similar text
- Validate prompt text format/length constraints

**Publication History**
- View history of all publications
- Filter by date, user, scope, element
- Ability to rollback to previous version

**Conflict Resolution**
- Detect if another user has edited same prompt
- Show both versions and allow user to choose or merge
- Prevent overwriting others' unpublished changes

---

## Bulk Operations

### Copy Prompts Across Scopes
- Copy prompt from one scope to another (or multiple others)
- Example: Copy all "North America" prompts to "APAC"
- Supports partial copying (selected elements only)

### Template Library
- Save common prompt patterns as templates
- Apply template to multiple elements at once
- Share templates across teams

### Batch Editing
- Apply same prompt text to multiple elements simultaneously
- Find and replace across prompts
- Append/prepend text to existing prompts

### Import/Export
- Export prompts to CSV or JSON for offline editing
- Import prompts from file for bulk updates
- Useful for large-scale prompt migrations
