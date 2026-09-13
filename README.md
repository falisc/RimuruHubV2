# Rimuru Hub UI Documentation

This document describes the finished RimuruUIInternal Roblox/Luau UI library and the current Example.lua integration.

## Source layout

| File | Purpose |
| --- | --- |
| UI.lua | Complete UI library: loader, window, navigation, widgets, popups, localization, Premium gates, world controls, overlays, assets, and config persistence. |
| Example.lua | Example integration with game tabs and Premium section, widget, and dropdown-option locks. |
| RimuruAssets/ | Bundled image assets written to the executor workspace when a window is created. |

The library returns a table named UI.

## Loading

~~~lua
local UI = loadstring(game:HttpGet("https://fal.lol/raw/oZ2uew0lNL"))()
~~~

The loader is native to UI.lua. Example.lua does not call a separate loader.

When a window is created, the library writes bundled icons and theme assets through the available executor filesystem/asset functions. The normal asset path uses writefile, makefolder, and either getcustomasset or getsynasset. Procedural fallbacks keep non-image icons usable when a particular image asset function is unavailable.

## Quick start

~~~lua
local UI = loadstring(game:HttpGet("https://fal.lol/raw/oZ2uew0lNL"))()

local window = UI:CreateWindow({
    Title = "Rimuru.temp",
    Width = 700,
    Height = 470,
    ConfigFolder = "RimuruConfigs",
    ConfigName = "ViolenceDistrict",
})

local tab = window:CreateTab("Visuals", "visual", {Group = "Utility"})
local section = tab:CreateSection("Targeting", {Column = "left"})

section:CreateCheckbox({
    Name = "Enabled",
    Default = true,
    Callback = function(value)
        print("enabled:", value)
    end,
})
~~~

## Finished UI defaults

- The only built-in theme is Rimuru Hub.
- English is the initial language.
- Supported languages are English, Indo, Ukrainian, Russian, and Thai.
- The hardcoded Languages section is the first section in Settings.
- Native labels, dropdown contents, keybind popups, Premium overlays, and entitlement text use the selected language where a translation is registered.
- The Rimuru Hub title gradient runs continuously instead of jumping between static colors.
- Darkening, Rain Effect, Watermark, and Keybind Overlay start disabled.
- The old Floating Island control is exposed as Watermark.
- The bottom-right UI resize handle is not part of the finished layout.
- The main window keeps fixed aspect-ratio behavior while resizing from viewport changes.
- Minimize shrinks the window into the draggable logo. Restore uses the same scale and position animation family.
- The loading panel is hardcoded and uses the Rimuru Hub surface, transparency, rounding, divider, and accent treatment.

## Library API

### UI:CreateWindow(config)

Creates and returns a Window. An existing RimuruUI_Internal instance is removed before the new window is built.

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| Title | string | "Rimuru.temp" | Window title shown in the rail and loader. |
| Brand | string | "Rimuru Hub" | Main header brand text. |
| BrandIcon | string | "logo" | Header brand icon name. |
| Discord | string | "discord.gg/rimuruhub" | Header Discord label; clicking copies the invite when clipboard support exists. |
| GameTitle | string | detected game name | Optional explicit game title. |
| Width | number | 700 | Normal window width in pixels. |
| Height | number | 470 | Normal window height in pixels. |
| RailWidth | number | 20% of width | Left navigation rail width. |
| Theme | string | "Rimuru Hub" | Initial theme. Unknown names fall back to Rimuru Hub. |
| Layout | string | "stacked" | Stacked content layout. "columns" and "two-column" enable two-column sections. |
| ConfigFolder | string | "RimuruConfigs" | Folder used for JSON configs. |
| ConfigName | string | empty | Initial config name and deferred autoload target. |
| AutoSave | boolean | false | Enables automatic config writes after state changes. |
| UIKeybind | string | "RightAlt" | Keyboard key used to show or hide the UI. |
| LoaderTitle | string | Title | Native loader title override. |
| LoaderDuration | number | 1.5 | Native loader duration in seconds. |
| IconData | table | bundled assets | Optional icon-kind to asset-data/path overrides. |
| OnDestroy | function | nil | Called before the window is destroyed. |

The window header shows the detected entitlement:

- More than 8 hours: Premium (time left)
- 8 hours or less: Free (time left)
- math.huge: Premium (Lifetime)
- no readable runtime time: Free (nil)

The runtime value used for this is LRM_SecondsLeft when present. The entitlement is evaluated when the window is created.

### UI:ShowLoader(config)

Shows the native themed loading panel. CreateWindow calls it automatically once per library instance, so a normal script should not call it separately.

~~~lua
UI:ShowLoader({
    Title = "Rimuru.temp",
    Status = "Loading interface...",
    Duration = 1.5,
    IconData = customIcons,
})
~~~

Supported fields are Title, Status, Duration, Theme, and IconData.

### UI:Notify(title, message, duration)

Creates a notification for the most recently created window.

~~~lua
UI:Notify("Rimuru.temp", "Action pressed", 5)
~~~

### UI:SetTheme(values)

Applies direct palette overrides to existing windows. Supported palette keys are:

~~~lua
{
    Background, Surface, Card, Hover, Border, Text, Muted,
    Disabled, Divider, Accent, Selected, Sidebar,
    SidebarTransparency, WindowTransparency, SurfaceTransparency,
    CardTransparency, SectionTransparency, BackgroundImage,
    BackgroundImageTransparency,
}
~~~

The only named palette is Rimuru Hub. Use window:SetTheme("Rimuru Hub") to reapply it.

## Window API

### Navigation and visibility

- window:CreateTab(name, icon, config) creates and returns a Tab.
- window:SelectTab(tab) selects a tab and displays its sections.
- window:SetVisible(value) shows or hides the full UI with the visibility animation.
- window:SetMinimized(value) shrinks into or restores from the draggable logo.
- window:SetUIToggleKeybind(key) changes the UI visibility key.
- window:ClosePopup() closes the active dropdown, keybind menu, or color picker.
- window:Destroy() disconnects runtime connections, restores Lighting, cancels tweens, and removes the UI.

### Appearance and overlays

- window:SetTheme(name) selects the named built-in theme.
- window:SetLanguage(name) selects English, Indo, Ukrainian, Russian, or Thai. Invalid values use English.
- window:SetWatermarkVisible(value) shows or hides the draggable Watermark.
- window:SetKeybindOverlayVisible(value) shows or hides the keybind list.
- window:SetDarkeningEnabled(value) toggles the darkening layer.
- window:SetRainEnabled(value) toggles the rain effect.
- window:SetScreensaverEnabled(value) toggles the black full-screen screensaver cover.

The screensaver is a black low-layer cover intended to reduce visible rendering work. Its layer remains below the main UI and popup layers.

### World controls

These methods snapshot the current game values when enabled and restore them when disabled or when the window is destroyed.

- window:SetRemoveFog(value) sets a very large FogEnd while active, then restores FogStart, FogEnd, and FogColor.
- window:SetFullbright(value) adjusts Brightness, ClockTime, Ambient, OutdoorAmbient, ColorShift_Bottom, ColorShift_Top, and GlobalShadows, then restores saved Lighting values.
- window:SetAmbientLightColor(color) sets the target Ambient and OutdoorAmbient color.
- window:SetAmbientLightEnabled(value) applies the target ambient color and restores the game's original ambient values when turned off.

### Search

~~~lua
window:Filter("pallet")
~~~

The search field uses end truncation so long text does not paint over adjacent controls. Results appear in a preview panel under the field and show the matching widget plus its tab and section.

Clicking a result selects the tab, expands the section, scrolls the widget into view, and briefly flashes its outline. Search includes native names and registered translated names; it does not remove widgets from the UI.

### Config persistence

~~~lua
window:SaveConfig("combat")
window:LoadConfig("combat")
window:CreateConfig("practice")
window:RefreshConfigs()
window:DeleteConfig("practice")
~~~

Config files are JSON format version 2 and store the selected theme, selected language, Auto Save state, checkbox values, slider values, single and multi dropdown values, keybind key/mode/active state, input text, Color3 RGB values, and color transparency.

Language is loaded after widget values, so a saved language cannot be replaced by a theme name or a boolean. Assign ConfigIgnore = true to a widget that should not be persisted.

## Tabs and sections

### window:CreateTab(name, icon, config)

~~~lua
local tab = window:CreateTab("Visuals", "visual", {
    Group = "Utility",
})
~~~

Arguments:

- name: visible tab label
- icon: registered icon name
- Group: optional navigation group label
- Bottom: optional bottom-navigation placement
- Hardcoded: internal flag used by Home Page and Settings

### tab:CreateSection(title, config)

~~~lua
local section = tab:CreateSection("Targeting", {
    Column = "left",
})
~~~

Column accepts left or right. Mode is an alias for Column.

Sections calculate their content height, keep a minimum height for Premium lock cards, and clip their card contents during collapse/expand animation. The returned section exposes:

~~~lua
section.SetCollapsed(true)
section.SetCollapsed(false)
~~~

### Hardcoded Settings sections

Settings is created by the library and begins with these native sections:

1. Languages: Language dropdown, default English, options English/Indo/Ukrainian/Russian/Thai.
2. Interface: Keybind Overlay default off; Watermark default off.
3. World: Remove Fog, Fullbright, Ambient Light, and Ambient Color.
4. Configuration: Config Name, Auto Save, Auto Load, Save Config, Load Config, Create Config, Delete Config, and Refresh Configs.
5. UI Keybind: UI Toggle and Unload Script.
6. Themes: Theme fixed to Rimuru Hub, Darkening default off, Rain Effect default off, and Screensaver default off.

## Widgets

Every widget returns a handle with at least Name, Root, and Label. Value widgets expose Value and a setter where applicable.

Every supported widget also accepts Premium = true. A Premium widget receives the lock veil and rejects input, setter, callback, popup, and keybind paths while the window is Free.

### section:CreateLabel(name)

Creates muted informational text.

~~~lua
section:CreateLabel("Full-size ESP controls")
~~~

### section:CreateButton(config)

~~~lua
local action = section:CreateButton({
    Name = "Run action",
    Icon = "click",
    Width = 132,
    Callback = function()
        print("clicked")
    end,
})
~~~

Fields are Name, Text (default Click), Icon, Width (minimum 114 pixels), and Callback. A string can be passed instead of a config table.

### section:CreateCheckbox(config)

~~~lua
local enabled = section:CreateCheckbox({
    Name = "Enabled",
    Default = false,
    Premium = true,
    Keybind = {
        Default = "F",
        Mode = "TOGGLE",
    },
    Callback = function(value)
        print(value)
    end,
})
~~~

Fields are Name, Default, Premium, Keybind, and Callback. Keybind can be a string or a table with Default/Key and Mode.

The handle exposes Set(value, silent), Value, Box, Row, and optional Keybind. section:CreateToggle(config) is an alias.

### section:CreateSlider(config)

~~~lua
local distance = section:CreateSlider({
    Name = "Distance",
    Min = 0,
    Max = 500,
    Default = 200,
    Step = 1,
    Premium = true,
    Callback = function(value)
        print(value)
    end,
})
~~~

Fields are Name, Min (default 0), Max (default 100), Default (default Min), Step, Increment (alias for Step), Premium, and Callback. The handle exposes Set(value, silent) and Value.

### section:CreateDropdown(config)

Dropdowns have a searchable popup. Multi dropdowns add Select All and Unselect All.

~~~lua
local mode = section:CreateDropdown({
    Name = "Mode",
    Default = "Nearest",
    Options = {"Nearest", "Visible", "Random"},
    Callback = function(value)
        print(value)
    end,
})
~~~

Fields are Name, Default (one value or an array with Multi), Options, Multi, Premium, PremiumOptions, OptionCallbacks, and Callback.

Options can be strings or records:

~~~lua
{
    Name = "Instant",
    Value = "instant",
    Premium = true,
    Callback = function(value, selected)
        print(value, selected)
    end,
}
~~~

For string options, PremiumOptions and OptionCallbacks provide the same behavior:

~~~lua
local dropdown = section:CreateDropdown({
    Name = "Mode",
    Default = "Safe",
    Options = {"Safe", "Instant", "Practice"},
    PremiumOptions = {"Instant"},
    OptionCallbacks = {
        Instant = function(value)
            print("premium option:", value)
        end,
    },
})
~~~

A locked option has its own active overlay, Premium gradient, and lock icon and cannot be selected. Protection covers popup clicks, defaults, Set, SetOptions, Select All, and option callback dispatch. A locked dropdown widget blocks the complete popup. The handle exposes Set(value), SetOptions(options), and Value.

### section:CreateKeybind(config)

~~~lua
local bind = section:CreateKeybind({
    Name = "Target Key",
    Default = "G",
    Mode = "TOGGLE",
    ShowInOverlay = true,
    Premium = true,
    Callback = function(key)
        print(key)
    end,
})
~~~

Fields are Name, Default, Mode (TOGGLE, HOLD, ALWAYS, or CLICK), ShowInOverlay (default true), Premium, and Callback.

The settings icon opens the mode/key capture popup. Locked widgets block that popup and the runtime key path. The handle exposes Button, Icon, and Keybind. The key entry exposes KeyName, Mode, Active, Capture, and SetActive(value).

### section:CreateColorPicker(config)

~~~lua
local color = section:CreateColorPicker({
    Name = "Accent Color",
    Default = Color3.fromRGB(83, 178, 224),
    Transparency = 0,
    Premium = true,
    Callback = function(value, transparency)
        print(value, transparency)
    end,
})
~~~

Fields are Name, Default, Transparency from 0 to 1, Premium, and Callback receiving Color3 plus transparency.

The popup includes hue, saturation/brightness, hexadecimal input, copy, and transparency controls. The handle exposes Set(color, silent), SetTransparency(value, silent), Value, Swatch, Stroke, and Transparency.

### section:CreateInput(config)

~~~lua
local label = section:CreateInput({
    Name = "Label Text",
    Placeholder = "Enter text...",
    Default = "Ready",
    Premium = true,
    Callback = function(value)
        print(value)
    end,
})
~~~

Fields are Name, Placeholder (default Type here...), Default, Premium, and Callback. Callback runs when focus leaves the field. The handle exposes Set(value, silent), Value, and Input.

## Premium section gates

A section can gate all of its widgets with one call:

~~~lua
local premiumSection = tab:CreateSection("Advanced Visuals", {Column = "left"})
premiumSection:CreateCheckbox({Name = "Glow", Default = false})
premiumSection:CreateSlider({Name = "Intensity", Min = 0, Max = 10, Default = 5})
premiumSection:LockPremium()
~~~

The section lock applies the animated Premium gradient to the section title, places a centered lock card over the complete section body, keeps the header usable for collapse/expand, keeps a minimum body height, blocks clicks through the overlay, closes popups anchored inside the locked section, and leaves the underlying callbacks and controls inaccessible while locked.

The lock card uses the recolorable bundled lock asset. Its detail text is Buy Premium in Discord.

### Entitlement rule

The window entitlement is read at creation from runtime values such as LRM_SecondsLeft:

~~~text
LRM_SecondsLeft > 8 hours  -> Premium
LRM_SecondsLeft <= 8 hours -> Free
LRM_SecondsLeft == math.huge -> Premium / Lifetime
missing runtime time       -> Free (nil)
~~~

When the window is Free, Section:LockPremium(), Premium = true, and locked dropdown options remain unavailable. When the window is Premium, those declarations remain visually marked with the Premium gradient while controls and callbacks are available.

Premium = true declares what should be gated; it does not replace the runtime entitlement check.

## Complete Premium example

~~~lua
local pallet = survivor:CreateSection("Pallet ESP", {Column = "right"})

pallet:CreateDropdown({
    Name = "Tags (Multi)",
    Default = {"Players", "Objectives"},
    Multi = true,
    Options = {
        "Players",
        "Objectives",
        {Name = "World", Premium = true},
        "Distance",
    },
})

pallet:CreateColorPicker({
    Name = "Accent Color",
    Default = Color3.fromRGB(73, 187, 255),
    Premium = true,
})

pallet:CreateInput({
    Name = "Label Text",
    Default = "Pallet ESP",
    Premium = true,
})
~~~

The current Example.lua additionally demonstrates Premium sections on multiple tabs and a Premium Instant dropdown option.

## Icons and assets

Icon names are case-insensitive. They can be passed to CreateTab or to a button's Icon field.

Bundled icon kinds include:

~~~text
logo, home, settings, visual, eye, eyeoff, search, minimize, close,
combat, assist, keyboard, star, click, player, location, skull, target,
lightning, chevron, lock, discord, resizeTriangle
~~~

Current recolorable assets include:

- RimuruAssets/Logos/LockRecolorable.png
- RimuruAssets/Logos/TouchRecolorable.png
- RimuruAssets/Logos/TriangleRecolorable.png
- RimuruAssets/Logos/DiscordRecolorable.png

These are white-alpha images, so ImageColor3 can tint them to the active UI palette.

## Current Example tabs

Example.lua builds:

- ESP
- Auto Features
- Killer ESP
- The Hidden
- Generator
- Movement
- Visuals
- Misc

It demonstrates section placement, checkbox keybinds, dropdowns, multi dropdowns, sliders, color pickers, inputs, action buttons, keybinds, section locks, widget locks, and a locked dropdown option.

## Integration notes

- Use returned handles instead of reaching into Roblox instances directly.
- Keep feature logic in callbacks and UI construction in tab/section declarations.
- Use silent = true when restoring a value without firing its callback.
- Call window:Destroy() from an unload path so Lighting snapshots and input connections are released.
- Use a stable ConfigFolder and ConfigName for persistent per-game layouts.
- Keep custom widget names stable when relying on config compatibility with older files.

## Credits

Rimuru Hub UI credits: catscure on Discord.
