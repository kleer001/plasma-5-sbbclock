# Plasma 5.27 LTS Migration Roadmap for SBB Clock
 
This document provides a complete step-by-step guide to convert the SBB Clock applet from Plasma 6 to Plasma 5.27 LTS.
 
## Overview
 
The SBB Clock was originally written for Plasma 6 and needs to be converted to work with Plasma 5.27 LTS. This involves:
- Updating metadata to remove Plasma 6 API requirements
- Converting all QML imports from versionless/Plasma 6 to Plasma 5 versions
- Replacing Plasma 6-specific APIs with Plasma 5 equivalents
- Removing features that depend on unavailable libraries (audio/QtMultimedia)
- Converting modern JavaScript syntax to Qt 5.15 compatible code
 
## Files to Modify
 
Total: 11 files need changes
 
1. `metadata.json` - Metadata update
2. `contents/ui/main.qml` - Core applet file
3. `contents/ui/AnalogClock.qml` - Analog clock component
4. `contents/ui/DigitalClock.qml` - Digital clock component
5. `contents/ui/CalendarView.qml` - Calendar popup
6. `contents/ui/Hand.qml` - Clock hand component
7. `contents/ui/Tooltip.qml` - Tooltip component
8. `contents/config/config.qml` - Configuration model
9. `contents/ui/configGeneral.qml` - General settings (minor/verify)
10. `contents/ui/configCalendar.qml` - Calendar settings
11. `contents/ui/configTimeZones.qml` - Timezone settings
12. `contents/ui/ConfigDialog.qml` - Config dialog
13. `contents/ui/NoTimezoneWarning.qml` - Warning component
 
---
 
## 1. Update metadata.json
 
**Purpose**: Remove Plasma 6 API requirements and update version
 
**Changes**:
 
```json
{
    "KPackageStructure": "Plasma/Applet",
    "KPlugin": {
        "Authors": [
            {
                "Email": "tomas.bautista1@gmail.com",
                "Name": "T. Bautista"
            }
        ],
        "BugReportUrl": "",
        "Category": "Date and Time",
        "Description": "An analog/digital clock",
        "EnabledByDefault": true,
        "FormFactors": [
            "tablet",
            "handset",
            "desktop"
        ],
        "Icon": "preferences-system-time",
        "Id": "sbbclock",
        "License": "BSD-2-Clause",
        "Name": "SBB Clock",
        "Website": "https://userbase.kde.org/Plasma/Clocks",
        "Version": "2.0"
    },
    "X-Plasma-API": "declarativeappletscript",
    "X-Plasma-MainScript": "ui/main.qml",
    "X-Plasma-Provides": [
        "org.kde.plasma.time",
        "org.kde.plasma.date"
    ]
}
```
 
**What to remove**:
- Line: `"X-Plasma-MainConfiguration": "contents/config/config.qml"`
- Line: `"X-Plasma-API-Minimum-Version": "6.0"`
 
**What to change**:
- `"Version": "1.0"` → `"Version": "2.0"`
 
---
 
## 2. Update contents/ui/main.qml
 
**Purpose**: Convert from PlasmoidItem to Item-based structure with Plasma 5 APIs
 
### Import Changes
 
**REPLACE** the entire import section:
 
```qml
import QtQuick 2.15
import QtQuick.Layouts 1.1
 
import org.kde.plasma.plasmoid 2.0
import org.kde.plasma.core 2.0 as PlasmaCore
import org.kde.kquickcontrolsaddons 2.0 as KQuickControlsAddons
import org.kde.plasma.components 3.0 as PlasmaComponents
import org.kde.kirigami 2.20 as Kirigami
import org.kde.plasma.calendar 2.0 as PlasmaCalendar
 
import "." as Local
```
 
### Root Element Change
 
**REPLACE**:
```qml
PlasmoidItem {
    id: root
```
 
**WITH**:
```qml
Item {
    id: root
```
 
### Property Changes
 
**FIND** all instances of:
- `Plasmoid.configuration` → Keep as `plasmoid.configuration` (lowercase)
- `Plasmoid.expanded` → Keep as `plasmoid.expanded` (lowercase)
- `Plasmoid.formFactor` → Keep as `plasmoid.formFactor` (lowercase)
 
### Remove Audio/Sound Features
 
**REMOVE** these entire blocks:
 
1. Properties (around lines 190-197):
```qml
property bool playHourGong: plasmoid.configuration.playHourGong
property real volumeInput: plasmoid.configuration.volumeSlider / 100
property string hourSignalSound: plasmoid.configuration.hourSignalSound
property int hourSignalStartTime: plasmoid.configuration.hourSignalStartTime
property int hourSignalEndTime: plasmoid.configuration.hourSignalEndTime
property int hourSignalAdvance: plasmoid.configuration.hourSignalAdvance
```
 
2. MediaPlayer block (around lines 202-208):
```qml
MediaPlayer {
    id: soundPlayer
    source: hourSignalSound
    audioOutput: AudioOutput {
        volume: volumeInput
    }
}
```
 
3. Function `checkAndPlaySignal()` (around lines 234-244):
```qml
function checkAndPlaySignal() {
    var date = new Date(dataSource.data["Local"]["DateTime"])
    var currentMinute = date.getMinutes()
    var currentSecond = date.getSeconds()
 
    if (minutes === 59 &&
        (hourSignalAdvance === 0 ? currentSecond === 59 : currentSecond === 60 - hourSignalAdvance) &&
        playHourGong && hours >= hourSignalStartTime && hours < hourSignalEndTime) {
        soundPlayer.play()
    }
}
```
 
4. Call to checkAndPlaySignal in dataSource.onDataChanged (around line 137):
```qml
// REMOVE this line:
checkAndPlaySignal()
```
 
### DataSource Changes
 
**REPLACE**:
```qml
import org.kde.plasma.plasma5support 2.0 as P5Support
 
P5Support.DataSource {
```
 
**WITH**:
```qml
PlasmaCore.DataSource {
```
 
### Plasmoid Representations
 
**FIND**:
```qml
preferredRepresentation: compactRepresentation
 
compactRepresentation: Item {
```
 
**REPLACE WITH**:
```qml
Plasmoid.preferredRepresentation: Plasmoid.compactRepresentation
 
Plasmoid.compactRepresentation: Item {
```
 
**FIND**:
```qml
fullRepresentation: CalendarView {
```
 
**REPLACE WITH**:
```qml
Plasmoid.fullRepresentation: CalendarView {
```
 
**FIND**:
```qml
toolTipItem: Loader {
```
 
**REPLACE WITH**:
```qml
Plasmoid.toolTipItem: Loader {
```
 
### Add Signal Handler for MonthView
 
**ADD** after the CalendarView instantiation:
```qml
Plasmoid.fullRepresentation: CalendarView {
    Layout.minimumWidth: Kirigami.Units.gridUnit * 40
    Layout.minimumHeight: Kirigami.Units.gridUnit * 25
 
    onMonthViewChanged: {
        root.monthView = monthView;
    }
}
```
 
### Property for MonthView
 
**ADD** this property:
```qml
property PlasmaCalendar.MonthView monthView: null
```
 
**UPDATE** the `updateHasEventsToday()` function:
```qml
function updateHasEventsToday() {
    if (plasmoid.expanded && monthView && monthView.daysModel) {
        var today = new Date();
        today.setHours(0, 0, 0, 0);
        hasEventsToday = monthView.daysModel.eventsForDate(today).length > 0;
    } else {
        hasEventsToday = false;
    }
}
```
 
---
 
## 3. Update contents/ui/AnalogClock.qml
 
**Purpose**: Convert KSvg to PlasmaCore.Svg and update property references
 
### Import Changes
 
**REPLACE** entire import section:
```qml
import QtQuick 2.15
import QtQuick.Layouts 1.1
 
import org.kde.plasma.plasmoid 2.0
import org.kde.plasma.core 2.0 as PlasmaCore
import org.kde.plasma.components 3.0 as PlasmaComponents
import org.kde.kirigami 2.20 as Kirigami
```
 
### Property Changes
 
**FIND AND REPLACE**:
- `face.naturalSize` → `face.nativeWidth` or `face.nativeHeight`
- `Plasmoid.configuration` → `plasmoid.configuration`
- `Plasmoid.formFactor` → `plasmoid.formFactor`
 
### Svg Changes
 
**REPLACE**:
```qml
KSvg.Svg {
    id: clockSvg
```
 
**WITH**:
```qml
PlasmaCore.Svg {
    id: clockSvg
```
 
**REPLACE**:
```qml
KSvg.SvgItem {
```
 
**WITH**:
```qml
PlasmaCore.SvgItem {
```
 
**REPLACE**:
```qml
KSvg.FrameSvgItem {
```
 
**WITH**:
```qml
PlasmaCore.FrameSvgItem {
```
 
### Specific Property Updates
 
**FIND** (around line 28):
```qml
property real handScale: Math.min(width, height) / Math.max(face.naturalSize.width, face.naturalSize.height)
```
 
**REPLACE WITH**:
```qml
property real handScale: Math.min(width, height) / Math.max(face.nativeWidth, face.nativeHeight)
```
 
**FIND** (around lines 113-114):
```qml
width: naturalSize.width * (face.width / face.naturalSize.width)
height: naturalSize.height * (face.width / face.naturalSize.width)
```
 
**REPLACE WITH**:
```qml
width: nativeWidth * (face.width / face.nativeWidth)
height: nativeHeight * (face.width / face.nativeWidth)
```
 
---
 
## 4. Update contents/ui/DigitalClock.qml
 
**Purpose**: Fix Plasmoid references, remove unsupported operators, and fix imports
 
### Import Changes
 
**REPLACE** imports:
```qml
import QtQuick 2.15
import org.kde.plasma.plasmoid 2.0
import org.kde.plasma.core 2.0 as PlasmaCore
import org.kde.plasma.components 3.0 as PlasmaComponents
import org.kde.plasma.extras 2.0 as PlasmaExtras
import org.kde.kirigami 2.20 as Kirigami
```
 
### Configuration Properties
 
**FIND** (lines ~27-51) all properties using `Plasmoid.configuration` and `??` operator:
 
**REPLACE** with:
```qml
// Configuration properties
readonly property bool showSeconds: plasmoid.configuration.showSecondHand ? plasmoid.configuration.showSecondHand : false
readonly property bool showTimezone: plasmoid.configuration.showTimezoneString ? plasmoid.configuration.showTimezoneString : false
readonly property bool showDate: plasmoid.configuration.showDate !== undefined ? plasmoid.configuration.showDate : true
readonly property bool transparentBackground: plasmoid.configuration.transparentBackground ? plasmoid.configuration.transparentBackground : false
readonly property bool blinkingTimeSeparator: plasmoid.configuration.blinkingTimeSeparator ? plasmoid.configuration.blinkingTimeSeparator : false
readonly property bool useCustomColors: plasmoid.configuration.useCustomColors ? plasmoid.configuration.useCustomColors : false
readonly property color timeColor: plasmoid.configuration.timeColor || PlasmaCore.Theme.textColor
readonly property color dateColor: plasmoid.configuration.dateColor || PlasmaCore.Theme.textColor
 
// Time format handling
readonly property string timeFormat: plasmoid.configuration.timeFormat || "hh:mm"
 
// Use system locale for formatting
readonly property string effectiveTimeFormat: timeFormat
 
// Date format handling
readonly property string dateFormat: plasmoid.configuration.dateFormat || "ddd, MMM d"
 
// Font properties
readonly property int timeFontSize: plasmoid.configuration.timeFontSize !== undefined ? plasmoid.configuration.timeFontSize : 24
readonly property int dateFontSize: plasmoid.configuration.dateFontSize !== undefined ? plasmoid.configuration.dateFontSize : 18
readonly property bool timeIsBold: plasmoid.configuration.timeIsBold ? plasmoid.configuration.timeIsBold : false
readonly property bool dateIsBold: plasmoid.configuration.dateIsBold ? plasmoid.configuration.dateIsBold : false
readonly property string fontFamily: plasmoid.configuration.fontFamily || ""
```
 
### Arrow Function to For Loop
 
**FIND** (around line 88):
```qml
var parts = processedFormat.match(/"[^"]*"|[^"]+/g);
var result = "";
parts.forEach(part => {
    if (part.startsWith('"') && part.endsWith('"')) {
        result += part.slice(1, -1);
    } else {
        var formatted = date.toLocaleDateString(Qt.locale(), part);
        if (part.includes('dddd')) {
            if (hasAllCaps) {
                formatted = formatted.toUpperCase();
            } else if (hasFirstCap) {
                formatted = capitalize(formatted);
            }
        }
        result += formatted;
    }
});
return result;
```
 
**REPLACE WITH**:
```qml
var parts = processedFormat.match(/"[^"]*"|[^"]+/g);
var result = "";
for (var i = 0; i < parts.length; i++) {
    var part = parts[i];
    if (part.startsWith('"') && part.endsWith('"')) {
        result += part.slice(1, -1);
    } else {
        var formatted = date.toLocaleDateString(Qt.locale(), part);
        if (part.includes('dddd')) {
            if (hasAllCaps) {
                formatted = formatted.toUpperCase();
            } else if (hasFirstCap) {
                formatted = capitalize(formatted);
            }
        }
        result += formatted;
    }
}
return result;
```
 
### Time Label Text Property
 
**FIND** (around line 155):
```qml
var format = Plasmoid.configuration.timeFormat;
```
 
**REPLACE WITH**:
```qml
var format = plasmoid.configuration.timeFormat;
```
 
### Date Label Color Property
 
**FIND** (around line 180):
```qml
color: {
    if (hasEventsToday && Plasmoid.configuration.showEventColor) {
        return Plasmoid.configuration.eventColor;
```
 
**REPLACE WITH**:
```qml
color: {
    if (hasEventsToday && plasmoid.configuration.showEventColor) {
        return plasmoid.configuration.eventColor;
```
 
---
 
## 5. Update contents/ui/CalendarView.qml
 
**Purpose**: Remove Plasma 6 PlasmaExtras.Representation and fix property names
 
### Import Changes
 
**REPLACE** imports:
```qml
import QtQuick 2.15
import QtQuick.Layouts 1.1
 
import org.kde.plasma.plasmoid 2.0
import org.kde.plasma.core 2.0 as PlasmaCore
import org.kde.plasma.calendar 2.0 as PlasmaCalendar
import org.kde.plasma.components 3.0 as PlasmaComponents
import org.kde.plasma.extras 2.0 as PlasmaExtras
import org.kde.kirigami 2.20 as Kirigami
```
 
### Root Element
 
**REPLACE**:
```qml
pragma ComponentBehavior: Bound
 
import QtQuick
// ... imports
 
PlasmaExtras.Representation {
    id: calendar
```
 
**WITH**:
```qml
import QtQuick 2.15
// ... imports
 
Item {
    id: calendar
 
    signal monthViewChanged(var monthView)
```
 
### Remove Required Properties
 
**FIND** any `required property` declarations and change them to regular properties.
 
For example:
```qml
required property string modelData
```
 
**REPLACE WITH**:
```qml
property string timeZone: modelData
```
 
### naturalSize to nativeWidth/nativeHeight
 
**FIND** (around line 344):
```qml
Layout.preferredHeight: naturalSize.height
```
 
**REPLACE WITH**:
```qml
Layout.preferredHeight: nativeHeight
```
 
**FIND** (around line 455):
```qml
width: naturalSize.width
```
 
**REPLACE WITH**:
```qml
width: nativeWidth
```
 
### MonthView Changes
 
**FIND** (around line 577):
```qml
PlasmaCalendar.MonthView {
    id: monthView
    viewHeader.height: calendar.headerHeight
```
 
**REPLACE WITH**:
```qml
PlasmaCalendar.MonthView {
    id: monthView
```
 
**REMOVE** the line:
```qml
viewHeader.height: calendar.headerHeight
```
 
**ADD** property:
```qml
readonly property double headerHeight: Math.max(agendaHeader.implicitHeight, monthView.headerHeight)
```
 
**FIND** near the end:
```qml
showDigitalClockHeader: true
digitalClock: Plasmoid
eventButton: addEventButton
```
 
**REMOVE** these three lines as they're Plasma 6 specific.
 
**ADD** at the end of MonthView:
```qml
Component.onCompleted: {
    calendar.monthViewChanged(monthView);
}
```
 
### Plasmoid References
 
**FIND ALL** instances of `Plasmoid.` and change to `plasmoid.` (lowercase)
 
---
 
## 6. Update contents/ui/Hand.qml
 
**Purpose**: Convert KSvg to PlasmaCore
 
### Import Changes
 
**REPLACE**:
```qml
import QtQuick 2.15
import QtQuick.Layouts 1.1
import org.kde.plasma.core 2.0 as PlasmaCore
import org.kde.kirigami 2.20 as Kirigami
```
 
**REMOVE**:
```qml
import org.kde.ksvg 1.0 as KSvg
```
 
### Element Type
 
**REPLACE**:
```qml
KSvg.SvgItem {
```
 
**WITH**:
```qml
PlasmaCore.SvgItem {
```
 
### Property Changes
 
**FIND**:
```qml
property real _fixedWidth: naturalSize.width * svgScale
property real _fixedHeight: naturalSize.height * svgScale
```
 
**REPLACE WITH**:
```qml
property real _fixedWidth: nativeWidth * svgScale
property real _fixedHeight: nativeHeight * svgScale
```
 
---
 
## 7. Update contents/ui/Tooltip.qml
 
**Purpose**: Remove Plasma 6 pragma and fix property access
 
### Remove Pragma
 
**REMOVE**:
```qml
pragma ComponentBehavior: Bound
```
 
### Import Changes
 
**REPLACE**:
```qml
import QtQuick 2.15
import QtQuick.Layouts 1.1
 
import org.kde.plasma.components 3.0 as PlasmaComponents
import org.kde.plasma.plasmoid 2.0
import org.kde.kirigami 2.20 as Kirigami
```
 
### Plasmoid References
 
Change all `Plasmoid.` to `plasmoid.`
 
### Repeater Model
 
**FIND** (around line 96):
```qml
model: root.selectedTimeZonesDeduplicatingExplicitLocalTimeZone()
    // Duplicate each entry...
    .reduce((array, item) => {
        array.push(item, item);
        return array;
    }, [])
```
 
**REPLACE WITH**:
```qml
model: {
    var zones = root.selectedTimeZonesDeduplicatingExplicitLocalTimeZone();
    var array = [];
    for (var i = 0; i < zones.length; i++) {
        array.push(zones[i]);
        array.push(zones[i]);
    }
    return array;
}
```
 
### Repeater Delegate
 
**FIND**:
```qml
PlasmaComponents.Label {
    required property int index
    required property string modelData
```
 
**REPLACE WITH**:
```qml
PlasmaComponents.Label {
    property int itemIndex: index
    property string timeZone: modelData
```
 
**THEN** update all references from `index` to `itemIndex` and `modelData` to `timeZone` within the delegate.
 
### MonthView Access
 
**FIND**:
```qml
text: (root.fullRepresentationItem as CalendarView)?.monthView.todayAuxilliaryText ?? ""
```
 
**REPLACE WITH**:
```qml
text: root.monthView && root.monthView.todayAuxilliaryText ? root.monthView.todayAuxilliaryText : ""
```
 
---
 
## 8. Update contents/config/config.qml
 
**Purpose**: Fix imports and model access
 
### Import Changes
 
**REPLACE**:
```qml
import QtQuick 2.15
 
import org.kde.plasma.plasmoid 2.0
import org.kde.plasma.configuration 2.0
import org.kde.plasma.calendar 2.0 as PlasmaCalendar
```
 
### Remove Pragma
 
**REMOVE**:
```qml
pragma ComponentBehavior: Bound
```
 
### Instantiator Delegate
 
**FIND**:
```qml
delegate: ConfigCategory {
    required property string display
    required property string decoration
    required property string configUi
    required property string pluginId
 
    name: display
    icon: decoration
    source: configUi
    visible: Plasmoid.configuration.enabledCalendarPlugins.indexOf(pluginId) > -1
}
```
 
**REPLACE WITH**:
```qml
delegate: ConfigCategory {
    property string pluginDisplay: model.display
    property string pluginDecoration: model.decoration
    property string pluginConfigUi: model.configUi
    property string pluginId: model.pluginId
 
    name: pluginDisplay
    icon: pluginDecoration
    source: pluginConfigUi
    visible: plasmoid.configuration.enabledCalendarPlugins.indexOf(pluginId) > -1
}
```
 
### Event Handlers
 
**FIND**:
```qml
onObjectAdded: (index, object) => configModel.appendCategory(object)
onObjectRemoved: (index, object) => configModel.removeCategory(object)
```
 
**REPLACE WITH**:
```qml
onObjectAdded: configModel.appendCategory(object)
onObjectRemoved: configModel.removeCategory(object)
```
 
### Plasmoid Configuration
 
Change `Plasmoid.configuration` to `plasmoid.configuration`
 
---
 
## 9. Update contents/ui/configCalendar.qml
 
**Purpose**: Fix imports and model access
 
### Remove Pragma
 
**REMOVE**:
```qml
pragma ComponentBehavior: Bound
```
 
### Import Changes
 
**REPLACE**:
```qml
import QtQuick 2.15
import QtQuick.Layouts 1.1
 
import org.kde.plasma.plasmoid 2.0
import org.kde.plasma.calendar 2.0 as PlasmaCalendar
import org.kde.kirigami 2.20 as Kirigami
import org.kde.kcmutils as KCMUtils
import QtQuick.Controls 2.15 as QQC2
```
 
### ComboBox Model
 
**FIND**:
```qml
model: [-1, 0, 1, 5, 6].map(day => ({ day, text: day === -1 ? ... }))
onActivated: (index) => { cfg_firstDayOfWeek = model[index].day; }
currentIndex: model.findIndex(item => item.day === cfg_firstDayOfWeek)
```
 
**REPLACE WITH**:
```qml
model: {
    var days = [-1, 0, 1, 5, 6];
    var result = [];
    for (var i = 0; i < days.length; i++) {
        var day = days[i];
        result.push({
            day: day,
            text: day === -1 ? i18nc("@item:inlistbox first day of week option", "Use region defaults") : Qt.locale().dayName(day)
        });
    }
    return result;
}
onActivated: { cfg_firstDayOfWeek = model[index].day; }
Component.onCompleted: {
    for (var i = 0; i < model.length; i++) {
        if (model[i].day === cfg_firstDayOfWeek) {
            currentIndex = i;
            break;
        }
    }
}
```
 
### Repeater Delegate
 
**FIND**:
```qml
delegate: QQC2.CheckBox {
    required property var model
    text: model.display
    checked: model.checked
```
 
**REPLACE WITH**:
```qml
delegate: QQC2.CheckBox {
    property var itemModel: model
    text: itemModel.display
    checked: itemModel.checked
```
 
**UPDATE** the onClicked handler:
```qml
onClicked: {
    itemModel.checked = checked;
    calendarPage.saveCalendarPlugins();
}
```
 
### Plasmoid Configuration
 
Change `Plasmoid.configuration` to `plasmoid.configuration`
 
---
 
## 10. Update contents/ui/configTimeZones.qml
 
**Purpose**: Simplify for Plasma 5
 
### Remove Pragma
 
**REMOVE**:
```qml
pragma ComponentBehavior: Bound
```
 
### Import Changes
 
**REPLACE**:
```qml
import QtQuick 2.15
import QtQuick.Controls 2.15 as QQC2
import QtQuick.Layouts 1.1
 
import org.kde.kirigami 2.20 as Kirigami
import org.kde.kcmutils as KCMUtils
```
 
**REMOVE**:
```qml
import org.kde.plasma.private.digitalclock
import org.kde.config as KConfig
```
 
### Update Message
 
**FIND**:
```qml
text: i18n("Time Zone configuration is currently disabled for maintenance.")
```
 
**REPLACE WITH**:
```qml
text: i18n("Time Zone configuration is currently disabled for maintenance.")
```
 
(Keep as placeholder - the private API isn't available in Plasma 5)
 
---
 
## 11. Update contents/ui/ConfigDialog.qml
 
**Purpose**: Fix imports
 
### Import Changes
 
**REPLACE**:
```qml
import QtQuick 2.15
import org.kde.plasma.configuration 2.0
```
 
**REMOVE**:
```qml
pragma ComponentBehavior: Bound
```
 
---
 
## 12. Update contents/ui/NoTimezoneWarning.qml
 
**Purpose**: Fix imports and Plasmoid references
 
### Import Changes
 
**REPLACE**:
```qml
import QtQuick 2.15
import QtQuick.Layouts 1.1
 
import org.kde.plasma.core 2.0 as PlasmaCore
import org.kde.plasma.components 3.0 as PlasmaComponents
import org.kde.plasma.plasmoid 2.0
import org.kde.kirigami 2.20 as Kirigami
 
import org.kde.kcmutils as KCM
```
 
### Plasmoid References
 
**FIND**:
```qml
visible: Plasmoid.formFactor == PlasmaCore.Types.Horizontal
```
 
**REPLACE WITH**:
```qml
visible: plasmoid.formFactor == PlasmaCore.Types.Horizontal
```
 
---
 
## Testing Checklist
 
After making all changes, test the following:
 
### Installation
```bash
# Package the applet
cd /path/to/plasma-5-sbbclock
zip -r sbbclock.plasmoid .
 
# Install
kpackagetool5 -i sbbclock.plasmoid
# Or update if already installed
kpackagetool5 -u sbbclock.plasmoid
```
 
### Functionality Tests
 
- [ ] Applet loads without QML errors
- [ ] Analog clock displays correctly
- [ ] Clock hands move and update every second
- [ ] Second hand has smooth animation (if enabled)
- [ ] Digital mode toggle works
- [ ] Digital clock displays time correctly
- [ ] Configuration dialog opens
- [ ] General settings can be changed
- [ ] Calendar settings can be modified
- [ ] Calendar view opens when clicked
- [ ] Calendar displays current month
- [ ] Events are shown (if calendar plugins enabled)
- [ ] Multiple timezones display correctly
- [ ] Tooltip shows correct information
- [ ] Timezone string displays (if enabled)
 
### Check for Errors
 
```bash
# Monitor plasmashell logs
journalctl -f | grep -i "qml\|plasma"
 
# Or restart plasmashell with debug output
plasmashell --replace &
```
 
Common errors to watch for:
- Import errors (version mismatches)
- Property access errors (Plasmoid vs plasmoid)
- Undefined property errors
- Type errors
 
---
 
## Summary of Key Changes
 
### API Conversions
- `PlasmoidItem` → `Item` + `Plasmoid.*` properties
- `org.kde.ksvg` → `org.kde.plasma.core` (for Svg components)
- `org.kde.plasma.plasma5support` → `org.kde.plasma.core` (DataSource)
- `PlasmaExtras.Representation` → `Item`
- `Plasmoid.*` → `plasmoid.*` (lowercase)
 
### Property Name Changes
- `naturalSize.width/height` → `nativeWidth/nativeHeight`
- `required property` → `property` (with model. prefix)
 
### JavaScript Syntax
- Arrow functions `=>` → traditional functions
- `??` operator → `? :` or `||`
- `.map()`, `.reduce()` → `for` loops (where needed)
 
### Features Removed
- QtMultimedia 6.7 (audio/sound functionality)
- All hourly gong/signal features
- Digital clock header features (Plasma 6 specific)
 
### Version Numbers
- All imports must have explicit version numbers
- QtQuick: `2.15`
- Plasma components: `2.0` or `3.0`
- Kirigami: `2.20`
 
---
 
## Commit Messages (Suggested)
 
```
Commit 1: Rewrite SBB Clock for Plasma 5.27 LTS compatibility
 
Major changes:
- Updated metadata.json: Changed API version from 6.0 to Plasma 5 format
- Converted all QML imports from Plasma 6 to Plasma 5 syntax
- Replaced PlasmoidItem with Item + Plasmoid properties
- Updated PlasmaCore.Svg/SvgItem to use Plasma 5 API
- Converted PlasmaExtras.Representation to regular Item
- Removed QtMultimedia 6.7 and all sound playback features
- Updated all configuration files for Plasma 5 compatibility
- Maintained existing features: analog/digital clock, calendar view, events, timezones
 
Commit 2: Fix Plasma 5 compatibility issues
 
- Fixed CalendarView.qml: naturalSize.width/height → nativeWidth/Height
- Fixed DigitalClock.qml: Plasmoid → plasmoid (lowercase for Plasma 5)
- Fixed DigitalClock.qml: ?? operator → ternary/|| operator
- Fixed DigitalClock.qml: Arrow function → traditional for loop
- Fixed DigitalClock.qml: Added version numbers to import statements
```
 
---
 
## Additional Notes
 
1. **DigitalClock private API**: The `org.kde.plasma.private.digitalclock` is not available in Plasma 5. We've implemented a custom DigitalClock component instead.
 
2. **Audio features**: QtMultimedia 6.7 is not compatible with Plasma 5. All audio/sound features have been removed. If sound is needed, it would require implementing with QtMultimedia 5.x separately.
 
3. **Calendar plugins**: The calendar plugin system works in Plasma 5, but some plugins may have different interfaces. Test with common plugins like holidays and events.
 
4. **Type annotations**: Plasma 5 doesn't support the `type: returnType` syntax in function signatures, so all function signatures must use the older format.
 
5. **Component behavior pragma**: The `pragma ComponentBehavior: Bound` is Plasma 6 only and must be removed.
 
---
 
End of Roadmap
