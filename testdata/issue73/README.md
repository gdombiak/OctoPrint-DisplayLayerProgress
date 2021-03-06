# DisplayLayerProgress

[![Version](https://img.shields.io/badge/dynamic/json.svg?color=brightgreen&label=version&url=https://api.github.com/repos/OllisGit/OctoPrint-DisplayLayerProgress/releases&query=$[0].name)]()
[![Released](https://img.shields.io/badge/dynamic/json.svg?color=brightgreen&label=released&url=https://api.github.com/repos/OllisGit/OctoPrint-DisplayLayerProgress/releases&query=$[0].published_at)]()
![GitHub Releases (by Release)](https://img.shields.io/github/downloads/OllisGit/OctoPrint-DisplayLayerProgress/latest/total.svg)

A OctoPrint-Plugin that sends the current progress of a print via M117 command to the printer-display and also to the top navigation bar.

A new feature is the "Desktop Printer-Display", which shows all M117 messages in a Desktop PopUp.

It shows the **current layer, total layer count, last/average layer duration, current height, total height, percentage, printTimeLeft, feedrate and fanspeed**:

- Printer Display: 50% L=60/120 H=23mm/47mm
- NavBar: Layer: 60 / 120 Height: 23mm of 47mm

*Output pattern is adjustable!*

**ATTENTION:** 
- The layer information works only when the slicer adds "layer-indicator" to the g-code (CURA-Example as comments like ```;LAYER:10```). Then these indicators are parsed via a regular-expression.
- Currently supported slicers: 
  - **CURA, Simplify3D, ideaMaker, KISSlicer, Slic3r** 
  - You can add your own layer-expressions in Plugin-Settings
- If you are not able find a layer-expression, try to add a "post processiong script in your slicer" E.g. for "slic3r", see [Enhancement #8](https://github.com/OllisGit/OctoPrint-DisplayLayerProgress/issues/8)
- Sometimes there is a "post processing script" that deletes all comments (e.g. see [Issue #33](https://github.com/OllisGit/OctoPrint-DisplayLayerProgress/issues/33))
- You need to upload your G-Code after installation of the plugin again (if you want to reuse already stored models in OctoPrint), because while uploading the G-Code is modfied
- The total height "calculation" can be done in two ways: 1)the max Z-Value in the G-Code, 2) max Z-Value with extrusion in this height
- The height/layer information is sometimes not matching with G-Code Viewer, because the viewer did a lot of "magic" (e.g. add extrusion diameter to height)

**Comment Format Examples:**

CURA: ```;LAYER:10```

Simplify3D: ```; layer 10, Z = 1.640```

The implementation is based on four steps:

1. PreProcessing the uploaded G-Code (replace layer-comment with M117) 
2. Read total layer count from G-Code before start (used last layer-comment)
3. G-Code-Hook to collect the current layer information (M117-command from step 1)
4. Progress-Hook to write all information to the printer/navbar


![navbar](screenshots/navbar.jpg "Progress in NavBar")
![statebar](screenshots/statebar.jpg "Progress in StateBar")
![desktopPrinterdisplay](screenshots/printerDisplay_popup.jpg "Desktop Printer-Display")
![desktopPrinterdisplayWithPrintTimeLeft](screenshots/desktop-printer-display_printTimeLeft.jpg "Desktop Printer-Display with printTimeLeft")
![printerdisplay](screenshots/example-printer-display.jpg "Progress in Printer-Display")

 
## Setup

Install via the bundled [Plugin Manager](https://github.com/foosel/OctoPrint/wiki/Plugin:-Plugin-Manager)
or manually using this URL:

    https://github.com/OllisGit/OctoPrint-DisplayLayerProgress/releases/latest/download/master.zip


## Configuration

It is possible to change the Output. See Plugin-Settings:


## Versions
see [Release-Overview](https://github.com/OllisGit/OctoPrint-DisplayLayerProgress/releases/)

---
# Developer - Section
Plugin sends the following custom events to the eventbus like this: 

    eventManager().fire(eventKey, eventPayload)

| EventKey                             |
| ------------------------------------ |
| DisplayLayerProgress_progressChanged |
| DisplayLayerProgress_layerChanged    |
| DisplayLayerProgress_feedrateChanged |
| DisplayLayerProgress_fanspeedChanged |

**Payload**
```javascript
 { 
   'totalLayer':'66',
   'currentLayer':'22',
   'currentHeight':'6.80',
   'totalHeightWithExtrusion':'20.0',
   'feedrate':'2700',
   'feedrateG0':'7200',
   'feedrateG1':'2700',
   'fanspeed':'100%',
   'progress':'28',
   'lastLayerDuration':'0h:00m:03s',
   'averageLayerDuration':'0h:00m:02s',
   'printTimeLeft':'2m3s',
   'printTimeLeftInSeconds':123
 }
```
Other Plugins could listen to this events like this:

    eventmanager.subscribe("DisplayLayerProgress_layerChanged", self._myEventListener)

or use `octoprint.plugin.EventHandlerPlugin` with something like this:

    def on_event(self, event, payload):
        if event == "DisplayLayerProgress_layerChanged":
            ## do something usefull
