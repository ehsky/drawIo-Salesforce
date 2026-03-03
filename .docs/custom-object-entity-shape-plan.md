# Plan: Custom Object Entity Shape for DataModelNotation.xml

## Context

The current `DataModelNotation.xml` contains basic entity shapes and relationship cardinality edges. The goal is to **completely replace** it with a new set of entity shapes that:
- Have connectable attribute rows (for linking fields between entities)
- Support optional header icon, description footer, and icon sub-footer
- Minimize in size when optional sections are absent (separate library entries per variation)
- Use **nested containers** so only the attributes section accepts drag-and-drop field additions
- Auto-expand when new fields are dropped in

## Target Design (from user's mock images)

4 library entry variations + 1 standalone field:

| # | Title | Header Icon | Attributes | Description Footer | Icon Sub-Footer |
|---|-------|:-----------:|:----------:|:-----------------:|:--------------:|
| 1 | Data Entity (Full) | Yes | 4 | Yes | Yes |
| 2 | Data Entity with description | Yes | 3 | Yes | No |
| 3 | Data Entity (no icon) | No | 3 | Yes | No |
| 4 | Simple | No | 0 | No | No |
| 5 | Entity Field (standalone) | N/A | N/A | N/A | N/A |

## Architecture: Nested Containers

All entity variations use the **same nested container pattern**:

```
Outer swimlane
  container=1; dropTarget=0; childLayout=stackLayout; resizeParent=1
  (Has stackLayout so footer moves when inner grows. dropTarget=0 rejects drops.)
  startSize=50: Header area (title text + optional icon as HTML in value)

  ├── Inner attributes swimlane (startSize=0, strokeColor=none, fillColor=none)
  │     container=1; dropTarget=1; childLayout=stackLayout; resizeParent=1
  │     (Accepts drops. Auto-grows when fields are added.)
  │     ├── Separator spacer (partialRectangle, connectable=0, h=10)
  │     ├── Attribute 1 (partialRectangle, connectable=1, h=20)
  │     ├── Attribute 2 (partialRectangle, connectable=1, h=20)
  │     ├── Attribute 3 (partialRectangle, connectable=1, h=20)
  │     └── tableRow closer (h=10)
  │
  ├── [Optional] Description footer (grey rect, connectable=0, h=~37)
  ├── [Optional] Icon sub-footer container (connectable=0, h=~35)
  │     └── Icon image(s) (shape=image, 18.5x18.5)
  └── tableRow closer (h=~7)
```

### How the resize chain works:
1. User drops "Entity Field" onto the attributes area (inner swimlane highlights with purple halo)
2. Inner swimlane's stackLayout positions the new field
3. Inner swimlane's `resizeParent=1` grows the inner swimlane to fit
4. Outer swimlane's stackLayout detects inner swimlane changed size
5. Outer swimlane re-stacks: pushes footer sections down
6. Outer swimlane's `resizeParent=1` grows the outer to fit everything

### Why nested containers:
- Drops over the **attributes area** → inner swimlane accepts (dropTarget=1)
- Drops over the **footer/icon area** → outer swimlane rejects (dropTarget=0)
- This prevents accidental modification of footer/icon sections

## Cell Styles

### Outer swimlane (all entries)
```
swimlane;childLayout=stackLayout;horizontal=1;startSize=50;horizontalStack=0;
rounded=1;fontSize=14;fontStyle=0;strokeWidth=2;resizeParent=1;resizeLast=0;
shadow=0;dashed=0;align=left;arcSize=10;whiteSpace=wrap;html=1;
strokeColor=#969492;movableLabel=0;absoluteArcSize=1;fillColor=#FFFFFF;
swimlaneFillColor=#FFFFFF;container=1;dropTarget=0;collapsible=0;
```

### Inner attributes swimlane
```
swimlane;childLayout=stackLayout;horizontal=1;startSize=0;horizontalStack=0;
rounded=0;fontSize=14;fontStyle=0;strokeWidth=0;resizeParent=1;resizeLast=0;
shadow=0;dashed=0;align=left;arcSize=0;whiteSpace=wrap;html=1;
strokeColor=none;fillColor=none;swimlaneFillColor=none;
container=1;dropTarget=1;collapsible=0;
```

### Connectable attribute row
```
shape=partialRectangle;connectable=1;fillColor=none;top=0;left=0;bottom=0;
right=0;align=left;spacingLeft=10;overflow=hidden;strokeColor=#514F4D;
strokeWidth=2;fontColor=#001639;spacing=2;spacingTop=5;
```
Width: 190, Height: 20

### Separator spacer
```
shape=partialRectangle;connectable=0;fillColor=none;top=0;left=0;bottom=0;
right=0;overflow=hidden;strokeColor=#514F4D;strokeWidth=2;
allowArrows=0;editable=0;movable=0;resizable=0;rotatable=0;deletable=0;
```
Width: 190, Height: 10

### Grey description footer
```
fillColor=#C9C7C5;strokeColor=#999999;align=center;spacing=2;fontSize=12;
strokeWidth=1;labelBackgroundColor=none;verticalAlign=middle;fontColor=#2E2E2E;
whiteSpace=wrap;connectable=0;movable=0;deletable=0;
```
Value: "Provide short description..."
Width: 190, Height: ~37

### Icon sub-footer container
```
fillColor=none;strokeColor=none;align=center;spacing=2;fontSize=12;
strokeWidth=1;verticalAlign=middle;connectable=0;movable=0;deletable=0;
container=0;dropTarget=0;
```
Width: 190, Height: ~35
Contains child `shape=image` cells (18.5x18.5 each)

### Bottom tableRow closer
```
shape=tableRow;horizontal=0;startSize=0;swimlaneHead=0;swimlaneBody=0;
fillColor=none;collapsible=0;dropTarget=0;points=[[0,0.5],[1,0.5]];
portConstraint=eastwest;top=0;left=0;right=0;bottom=0;
strokeColor=#514F4D;strokeWidth=2;
```

### Header icon (embedded in swimlane value as HTML)
For entries with header icons, the swimlane's `value` attribute contains:
```html
<img src="data:image/png,..." width="30" height="30" style="vertical-align:middle;margin-right:8px"/>
<b style="color:#001639;">Data Entity</b><br/>
<i style="font-size:11px;color:#001639;">EntityApiName__c</i>
```
For entries without icon:
```html
<b style="color:#001639;">Data Entity</b><br/>
<i style="font-size:11px;color:#001639;">EntityApiName__c</i>
```

## Detailed Entry Specifications

### Entry 1: "Data Entity" (Full — icon + 4 attrs + description + icon footer)
- Width: 190, Height: ~260
- Outer swimlane with header icon in HTML value (startSize=50)
- Inner attributes swimlane: separator + 4 attributes + tableRow closer
- Description footer: "Provide short description..."
- Icon sub-footer: 1 placeholder icon (users add more)
- Outer tableRow closer

### Entry 2: "Data Entity with description" (icon + 3 attrs + description)
- Width: 190, Height: ~200
- Same as Entry 1 but: 3 attribute rows, no icon sub-footer

### Entry 3: "Data Entity (no icon)" (no header icon + 3 attrs + description)
- Width: 190, Height: ~180
- Same as Entry 2 but: no icon in header HTML value

### Entry 4: "Simple" (header only, minimal)
- Width: 190, Height: ~60
- Outer swimlane with title only (no icon)
- Inner attributes swimlane: empty (startSize=0, very small)
- No footer, no icon sub-footer
- Just the outer tableRow closer

### Entry 5: "Entity Field" (standalone attribute row)
- Single `partialRectangle` cell, connectable=1
- Width: 190, Height: 20
- Value: "FieldApiName__c"
- Users drag this INTO an entity's attributes section to add fields

## Header Icon Source

Extract a Salesforce object icon (purple circle with grid/table pattern) from the existing icon libraries. Use base64 `data:image/png,...` format embedded in the swimlane's HTML value attribute.

## Files to Modify

| File | Action |
|------|--------|
| `shapes/DataModelNotation.xml` | **Replace entirely** with 5 new library entries |
| `configuration.json` | **Update** library title if needed |
| `README.md` | **Update** to describe new entity shapes |

## Implementation Steps

1. **Extract icon base64 data**: Get a suitable Salesforce icon from existing libraries (PlatformIcons.xml or ProductIcons.xml) and icon footer images from Cards.xml
2. **Build Entry 1 XML**: Construct the full Data Entity mxGraphModel with nested containers
3. **Build Entries 2-4**: Derive from Entry 1, removing optional sections
4. **Build Entry 5**: Standalone field row
5. **Assemble library**: HTML-entity encode all entries, wrap in `<mxlibrary>[...]</mxlibrary>` JSON array
6. **Write** to `shapes/DataModelNotation.xml`
7. **Update** `configuration.json` if library title changes
8. **Update** `README.md`

## Verification

1. Import `DataModelNotation.xml` in draw.io
2. Verify all 5 library entries appear
3. For each entity variation:
   - Header renders correctly (title, optional icon, underline)
   - Attribute rows are individually connectable
   - Description footer shows grey text when present
   - Icon sub-footer renders when present
4. **Dynamic field addition**: Drag "Entity Field" onto an entity's attributes area:
   - Only the attributes section highlights (purple halo) — NOT footer/header
   - Field is added and stacks with existing attributes
   - Entity auto-expands in height
   - New field is connectable
5. Draw edges between attribute rows of different entities
6. Validate JSON in configuration files
