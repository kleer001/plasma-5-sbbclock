# Plasma 5 Migration Quick Reference Checklist
 
Use this as a quick reference while working through the migration.
 
## Global Search & Replace Operations
 
These can be done with your editor's find/replace across all files:
 
### 1. Import Statements
```
FIND: import QtQuick$
REPLACE: import QtQuick 2.15
 
FIND: import QtQuick.Layouts$
REPLACE: import QtQuick.Layouts 1.1
 
FIND: import org.kde.plasma.plasmoid$
REPLACE: import org.kde.plasma.plasmoid 2.0
 
FIND: import org.kde.plasma.core as PlasmaCore$
REPLACE: import org.kde.plasma.core 2.0 as PlasmaCore
 
FIND: import org.kde.ksvg 1.0 as KSvg
REPLACE: import org.kde.plasma.core 2.0 as PlasmaCore
 
FIND: import org.kde.plasma.plasma5support
REPLACE: (remove entirely - use PlasmaCore.DataSource)
 
FIND: import org.kde.plasma.extras as PlasmaExtras
REPLACE: import org.kde.plasma.extras 2.0 as PlasmaExtras
 
FIND: import org.kde.plasma.calendar as PlasmaCalendar
REPLACE: import org.kde.plasma.calendar 2.0 as PlasmaCalendar
```
 
### 2. Component Type Changes
```
FIND: PlasmoidItem {
REPLACE: Item {
 
FIND: KSvg.Svg {
REPLACE: PlasmaCore.Svg {
 
FIND: KSvg.SvgItem {
REPLACE: PlasmaCore.SvgItem {
 
FIND: KSvg.FrameSvgItem {
REPLACE: PlasmaCore.FrameSvgItem {
 
FIND: P5Support.DataSource {
REPLACE: PlasmaCore.DataSource {
 
FIND: PlasmaExtras.Representation {
REPLACE: Item {
```
 
### 3. Property References
```
FIND: Plasmoid\.
REPLACE: plasmoid.
(Note: Keep Plasmoid.compactRepresentation, Plasmoid.fullRepresentation,
 Plasmoid.toolTipItem, Plasmoid.preferredRepresentation as-is)
 
FIND: \.naturalSize\.width
REPLACE: .nativeWidth
 
FIND: \.naturalSize\.height
REPLACE: .nativeHeight
 
FIND: naturalSize\.
REPLACE: (context dependent - usually nativeWidth or nativeHeight)
```
 
### 4. JavaScript Syntax
```
FIND: \?\?
REPLACE: (convert to ternary or || operator - see context)
 
FIND: required property
REPLACE: property
 
FIND: \.forEach\(.*=>
REPLACE: (convert to for loop - manual)
 
FIND: => {
REPLACE: (convert to function() { or for loop - manual)
```
 
### 5. Pragma Directives
```
FIND: pragma ComponentBehavior: Bound
REPLACE: (remove line entirely)
```
 
---
 
## File-by-File Quick Checklist
 
### ✓ metadata.json
- [ ] Remove `"X-Plasma-MainConfiguration"` line
- [ ] Remove `"X-Plasma-API-Minimum-Version": "6.0"` line
- [ ] Change version to `"2.0"`
 
### ✓ contents/ui/main.qml
- [ ] Update all imports (add versions)
- [ ] Change `PlasmoidItem` → `Item`
- [ ] Remove all `import QtMultimedia` and QtMultimedia code
- [ ] Remove MediaPlayer, AudioOutput blocks
- [ ] Remove sound/gong properties and functions
- [ ] Change `P5Support.DataSource` → `PlasmaCore.DataSource`
- [ ] Add `Plasmoid.` prefix to: preferredRepresentation, compactRepresentation, fullRepresentation, toolTipItem
- [ ] Change all other `Plasmoid.` → `plasmoid.` (except above)
- [ ] Add `property PlasmaCalendar.MonthView monthView: null`
- [ ] Add `onMonthViewChanged` signal handler to CalendarView
 
### ✓ contents/ui/AnalogClock.qml
- [ ] Update imports (add versions, remove KSvg)
- [ ] Change `KSvg.*` → `PlasmaCore.*`
- [ ] Change `.naturalSize.width` → `.nativeWidth`
- [ ] Change `.naturalSize.height` → `.nativeHeight`
- [ ] Change `Plasmoid.` → `plasmoid.`
 
### ✓ contents/ui/DigitalClock.qml
- [ ] Update imports (add versions)
- [ ] Change all `Plasmoid.configuration` → `plasmoid.configuration`
- [ ] Replace all `??` with `? :` or `||`
- [ ] Convert `forEach(part => ...)` to traditional for loop
- [ ] Remove arrow functions
 
### ✓ contents/ui/CalendarView.qml
- [ ] Remove `pragma ComponentBehavior: Bound`
- [ ] Update imports (add versions)
- [ ] Change `PlasmaExtras.Representation` → `Item`
- [ ] Add `signal monthViewChanged(var monthView)`
- [ ] Change `.naturalSize.width` → `.nativeWidth`
- [ ] Change `.naturalSize.height` → `.nativeHeight`
- [ ] Change `required property` → `property`
- [ ] Add model. prefix to repeater delegate properties
- [ ] Remove `viewHeader.height`, `showDigitalClockHeader`, `digitalClock`, `eventButton` from MonthView
- [ ] Add `Component.onCompleted` with `monthViewChanged` signal to MonthView
- [ ] Change `Plasmoid.` → `plasmoid.`
 
### ✓ contents/ui/Hand.qml
- [ ] Update imports (remove KSvg, add versions)
- [ ] Change `KSvg.SvgItem` → `PlasmaCore.SvgItem`
- [ ] Change `naturalSize` → `nativeWidth/nativeHeight`
 
### ✓ contents/ui/Tooltip.qml
- [ ] Remove `pragma ComponentBehavior: Bound`
- [ ] Update imports (add versions)
- [ ] Convert repeater model `.reduce()` to for loop
- [ ] Change `required property` → `property`
- [ ] Add itemIndex/timeZone property names
- [ ] Change `??` to ternary/||
- [ ] Change optional chaining `?.` to explicit null checks
- [ ] Change `Plasmoid.` → `plasmoid.`
 
### ✓ contents/config/config.qml
- [ ] Remove `pragma ComponentBehavior: Bound`
- [ ] Update imports (add versions)
- [ ] Change Instantiator delegate `required property` → `property`
- [ ] Add `model.` prefix to delegate properties
- [ ] Remove arrow functions from event handlers
- [ ] Change `Plasmoid.` → `plasmoid.`
 
### ✓ contents/ui/configCalendar.qml
- [ ] Remove `pragma ComponentBehavior: Bound`
- [ ] Update imports (add versions)
- [ ] Convert ComboBox `.map()` to manual for loop
- [ ] Remove arrow function from onActivated
- [ ] Add Component.onCompleted for currentIndex
- [ ] Change repeater delegate `required property` → `property`
- [ ] Change `Plasmoid.` → `plasmoid.`
 
### ✓ contents/ui/configTimeZones.qml
- [ ] Remove `pragma ComponentBehavior: Bound`
- [ ] Update imports (add versions)
- [ ] Remove `org.kde.plasma.private.digitalclock` import
- [ ] Keep placeholder message
 
### ✓ contents/ui/ConfigDialog.qml
- [ ] Update imports (add versions)
 
### ✓ contents/ui/NoTimezoneWarning.qml
- [ ] Update imports (add versions)
- [ ] Change `Plasmoid.` → `plasmoid.`
 
---
 
## Common Patterns to Watch For
 
### Pattern 1: Nullish Coalescing
```javascript
// WRONG (Plasma 6)
property bool foo: config.foo ?? false
 
// RIGHT (Plasma 5)
property bool foo: config.foo !== undefined ? config.foo : false
// OR if it's a string/object:
property string bar: config.bar || "default"
```
 
### Pattern 2: Arrow Functions
```javascript
// WRONG (Plasma 6)
items.forEach(item => {
    doSomething(item);
});
 
// RIGHT (Plasma 5)
for (var i = 0; i < items.length; i++) {
    var item = items[i];
    doSomething(item);
}
```
 
### Pattern 3: Required Properties
```javascript
// WRONG (Plasma 6)
required property string modelData
text: modelData
 
// RIGHT (Plasma 5)
property string itemData: modelData
text: itemData
```
 
### Pattern 4: Optional Chaining
```javascript
// WRONG (Plasma 6)
text: obj?.property ?? ""
 
// RIGHT (Plasma 5)
text: obj && obj.property ? obj.property : ""
```
 
### Pattern 5: SvgItem Properties
```javascript
// WRONG (Plasma 6)
width: naturalSize.width
height: naturalSize.height
 
// RIGHT (Plasma 5)
width: nativeWidth
height: nativeHeight
```
 
---
 
## Testing Commands
 
```bash
# Package
cd /path/to/plasma-5-sbbclock
zip -r sbbclock.plasmoid .
 
# Install
kpackagetool5 -i sbbclock.plasmoid
 
# Update
kpackagetool5 -u sbbclock.plasmoid
 
# Remove
kpackagetool5 -r sbbclock
 
# Restart plasmashell
plasmashell --replace &
 
# Check logs
journalctl -f | grep -E "(qml|plasma|sbbclock)"
```
 
---
 
## Validation Checks
 
Run these grep commands to ensure no Plasma 6 syntax remains:
 
```bash
cd /path/to/plasma-5-sbbclock
 
# Check for versionless imports
grep -r "^import QtQuick$" contents/
 
# Check for KSvg
grep -r "org.kde.ksvg" contents/
 
# Check for plasma5support
grep -r "plasma5support" contents/
 
# Check for PlasmoidItem
grep -r "PlasmoidItem" contents/
 
# Check for naturalSize (should return nothing or be in comments)
grep -r "naturalSize" contents/ui/
 
# Check for pragma
grep -r "pragma ComponentBehavior" contents/
 
# Check for ?? operator
grep -r " ?? " contents/
 
# Check for arrow functions in QML
grep -r " => " contents/ui/*.qml
 
# Check for required properties
grep -r "required property" contents/
 
# Check for uppercase Plasmoid (excluding specific properties)
grep -r "Plasmoid\." contents/ | grep -v "Plasmoid.compactRepresentation\|Plasmoid.fullRepresentation\|Plasmoid.toolTipItem\|Plasmoid.preferredRepresentation\|Plasmoid.backgroundHints"
```
 
If any of these return results (except where noted), you need to fix those instances.
 
---
 
## Priority Order
 
Work through files in this order:
 
1. **metadata.json** - Quick, foundational
2. **contents/ui/Hand.qml** - Simple component
3. **contents/ui/AnalogClock.qml** - Uses Hand
4. **contents/ui/DigitalClock.qml** - Standalone component
5. **contents/ui/Tooltip.qml** - Standalone component
6. **contents/ui/main.qml** - Main file, uses above components
7. **contents/ui/CalendarView.qml** - Complex, used by main
8. **contents/config/config.qml** - Configuration
9. **contents/ui/configGeneral.qml** - Verify only
10. **contents/ui/configCalendar.qml** - Configuration UI
11. **contents/ui/configTimeZones.qml** - Configuration UI
12. **contents/ui/ConfigDialog.qml** - Simple config
13. **contents/ui/NoTimezoneWarning.qml** - Simple warning
 
---
 
## Success Criteria
 
Your migration is complete when:
 
- [ ] All files follow Plasma 5 syntax
- [ ] No QML errors in plasmashell logs
- [ ] Applet loads in panel/desktop
- [ ] Clock displays and updates
- [ ] Configuration dialog works
- [ ] Calendar view opens and displays
- [ ] All validation checks pass (grep commands above)
 
---
 
End of Quick Reference
