# 📜 RecycleScrollView

This package provides a scroll view system that works similarly to `GridViewComponent`,
but offers a much smoother user experience by reducing flickering issues.

---

## Features

### Reusable Cell System

`RecycleScrollView` creates only a fixed number of cells.
As the user scrolls, these cells are repositioned and reused, and the `onUpdateCell` callback is triggered.

### Vertical / Horizontal Scrolling

You can set the scroll direction to match your layout.

### Scrollbars

Supports both horizontal and vertical scrollbars.

---

## Methods

| Method | Description |
|--------|-------------|
| `SetTotalCount(count, isResetPos)` | Sets the total number of items and optionally resets the scroll position. |
| `ScrollToIndex(index, scrollAlignmentType)` | Scrolls to a specific item index with alignment options: `Start` (beginning of view), `Center` (center of view), `End` (end of view). |
| `OnScroll(direction)` / `OnDrag(direction)` | Scrolls in the specified direction. |

---

## Custom Callbacks

A callback is provided to dynamically update data for each cell:

**`onUpdateCell(index, entity)`**

| Parameter | Description |
|-----------|-------------|
| `index` | The index of the data |
| `entity` | The reused cell object |

This callback is called whenever a cell is reused, allowing you to update the entity with data corresponding to the given index.

---

## Configurable Properties

### RecycleScrollView

| Property | Type | Description |
|----------|------|-------------|
| `Padding` | Vector4 | Sets the top, bottom, left, and right padding. |
| `Spacing` | Vector2 | Sets the spacing between scroll items. |
| `ScrollingType` | integer | Sets the scroll direction. (`0`: Horizontal, `1`: Vertical) |
| `TotalCount` | integer | Sets the total number of scroll items. |
| `CrossAxisItemCount` | integer | Sets the number of items placed along the cross axis. |
| `ChildAlignmentType` | integer | Sets the alignment method for child items within the layout. |
| `Item` | Entity | The prefab (Entity) used to create scroll items. |
| `ScrollSensitivity` | number | Sets the mouse wheel scroll sensitivity. |
| `HorizontalBar` | ScrollBar | Assigns the horizontal scrollbar. |
| `VerticalBar` | ScrollBar | Assigns the vertical scrollbar. |

### ScrollBar

| Property | Type | Description |
|----------|------|-------------|
| `ScrollingType` | integer | Sets the scroll direction. (`0`: Horizontal, `1`: Vertical) |
| `Handle` | UITransformComponent | Specifies the UITransform component used as the scroll handle (bar). |

---

## Installation Guide

1. Add the `RecycleScrollView` component to a UI entity.
   - It should have `SpriteGUIRendererComponent`, `UITouchReceiveComponent`, and `MaskComponent`.
2. Assign a template `Item` entity that will be reused.
3. Configure `TotalCount` and other layout properties.
4. For scrollbar integration:
   - Create a `ScrollBar` entity with a `Handle` child object.
   - Assign it to the `HorizontalBar` / `VerticalBar` property.
   - The `ScrollBar` entity should have a `UITouchReceiveComponent`.

---

## Sample Project Guide

You can test the functionality using the included sample models:
`Model_RecycleScrollView` and `Model_UIScrollViewBar`.

### Keyboard Shortcuts

| Key | Description |
|-----|-------------|
| **K** | Set total item count to 100 |
| **M** | Scroll to index 25 (aligned to end) |
| **L** | Test `OnScroll(Vector2(30000, 30000))` |
| **J** | Test `OnDrag(Vector2(-10000, -1000))` |

※ Refer to the `RecycleViewTest` script for sample usage.
