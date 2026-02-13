# DaVinci Resolve Scripting API Documentation

**Last Updated:** October 7, 2025

*I just generated this lil documentation based off of the .txt file that acts as the only documentation for DaVinci Resolve's scripting stuff that I can find available.*

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Environment Setup](#environment-setup)
4. [Getting Started](#getting-started)
5. [Running Scripts](#running-scripts)
6. [API Reference](#api-reference)
7. [Key Concepts](#key-concepts)
8. [Common Properties](#common-properties)

---

## Introduction

The DaVinci Resolve Scripting API allows you to automate tasks in DaVinci Resolve Studio using Python or Lua. Scripts can be invoked from the Console window, command line, or through the Workspace menu.

### Supported Languages
- **Lua** 5.1
- **Python** ≥ 3.6 (64-bit)
- **Python** 2.7 (64-bit)

### Key Points
- DaVinci Resolve must be running for scripts to execute
- Starting from v16.2.0, `nodeIndex` parameters are 1-based (not 0-based)
- Security implications exist when allowing scripting from external networks

---

## Prerequisites

Install one of the following for all users:

- Lua 5.1
- Python ≥ 3.6 (64-bit)
- Python 2.7 (64-bit)

---

## Environment Setup

### macOS

```bash
export RESOLVE_SCRIPT_API="/Library/Application Support/Blackmagic Design/DaVinci Resolve/Developer/Scripting"
export RESOLVE_SCRIPT_LIB="/Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Libraries/Fusion/fusionscript.so"
export PYTHONPATH="$PYTHONPATH:$RESOLVE_SCRIPT_API/Modules/"
```

### Windows

```batch
set RESOLVE_SCRIPT_API=%PROGRAMDATA%\Blackmagic Design\DaVinci Resolve\Support\Developer\Scripting
set RESOLVE_SCRIPT_LIB=C:\Program Files\Blackmagic Design\DaVinci Resolve\fusionscript.dll
set PYTHONPATH=%PYTHONPATH%;%RESOLVE_SCRIPT_API%\Modules\
```

### Linux

```bash
export RESOLVE_SCRIPT_API="/opt/resolve/Developer/Scripting"
export RESOLVE_SCRIPT_LIB="/opt/resolve/libs/Fusion/fusionscript.so"
export PYTHONPATH="$PYTHONPATH:$RESOLVE_SCRIPT_API/Modules/"
```

> **Note:** For standard ISO Linux installations, replace `/opt/resolve` with `/home/resolve`

---

## Getting Started

### Basic Python Example

```python
#!/usr/bin/env python
import DaVinciResolveScript as dvr_script

# Get the resolve object
resolve = dvr_script.scriptapp("Resolve")

# Access project manager
projectManager = resolve.GetProjectManager()

# Create a new project
project = projectManager.CreateProject("Hello World")
```

### Script Placement

On startup, DaVinci Resolve scans specific folders for scripts:

**macOS:**
- All users: `/Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Scripts`
- Specific user: `/Users/<UserName>/Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Scripts`

**Windows:**
- All users: `%PROGRAMDATA%\Blackmagic Design\DaVinci Resolve\Support\Fusion\Scripts`
- Specific user: `%APPDATA%\Roaming\Blackmagic Design\DaVinci Resolve\Support\Fusion\Scripts`

**Linux:**
- All users: `/opt/resolve/Fusion/Scripts`
- Specific user: `/home/<UserName>/.config/Blackmagic Design/DaVinci Resolve/Fusion/Scripts`

**Script Categories:**
- `Utility/` - Available in all pages
- `Comp/` or `Tool/` - Fusion page only
- Page-specific: `Edit/`, `Color/`, `Deliver/`, etc.
- Scripts in `Deliver/` also appear under render jobs

---

## Running Scripts

### From the Console
1. Open DaVinci Resolve
2. Navigate to Fusion page
3. Open Console window
4. Enter Python or Lua commands directly

### From Command Line
```bash
# Execute external script (requires environment variables set)
python /path/to/script.py
```

### From Workspace Menu
1. Place script in appropriate Scripts folder
2. Restart DaVinci Resolve
3. Go to Workspace menu > Scripts > [Category] > [Script Name]

### Headless Mode
```bash
# Run DaVinci Resolve without UI
/path/to/resolve -nogui
```

---

## API Reference

### Core Object Hierarchy

```
Resolve (root)
├── Fusion
├── ProjectManager
│   └── Project
│       ├── MediaPool
│       │   └── Folder
│       │       └── MediaPoolItem
│       ├── Timeline
│       │   └── TimelineItem
│       └── Gallery
├── MediaStorage
└── [other utilities]
```

---

## Resolve

The fundamental starting point for scripting. Access all other objects through this.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `Fusion()` | Fusion | Returns the Fusion object |
| `GetMediaStorage()` | MediaStorage | Returns media storage object |
| `GetProjectManager()` | ProjectManager | Returns project manager for current database |
| `OpenPage(pageName)` | Bool | Switch to page: "media", "cut", "edit", "fusion", "color", "fairlight", "deliver" |
| `GetCurrentPage()` | String | Returns currently displayed page |
| `GetProductName()` | String | Returns product name |
| `GetVersion()` | List | Returns [major, minor, patch, build, suffix] |
| `GetVersionString()` | String | Returns "major.minor.patch[suffix].build" format |
| `SaveLayoutPreset(presetName)` | Bool | Saves current UI layout as preset |
| `LoadLayoutPreset(presetName)` | Bool | Loads UI layout from preset |
| `ImportRenderPreset(presetPath)` | Bool | Import render preset from path |
| `ExportRenderPreset(presetName, exportPath)` | Bool | Export render preset to path |
| `GetKeyframeMode()` | Int | Returns current keyframe mode |
| `SetKeyframeMode(keyframeMode)` | Bool | Sets keyframe mode |
| `Quit()` | None | Quits Resolve |

---

## ProjectManager

Manages project creation, loading, deletion, and database connections.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `CreateProject(projectName, [mediaLocationPath])` | Project | Creates new project |
| `DeleteProject(projectName)` | Bool | Deletes project if not currently loaded |
| `LoadProject(projectName)` | Project | Loads and returns project |
| `GetCurrentProject()` | Project | Returns currently loaded project |
| `SaveProject()` | Bool | Saves current project with its own name |
| `CloseProject(project)` | Bool | Closes specified project without saving |
| `GetProjectListInCurrentFolder()` | [String] | Lists projects in current folder |
| `GetFolderListInCurrentFolder()` | [String] | Lists subfolders in current folder |
| `CreateFolder(folderName)` | Bool | Creates folder if name is unique |
| `DeleteFolder(folderName)` | Bool | Deletes specified folder |
| `OpenFolder(folderName)` | Bool | Opens folder |
| `GotoRootFolder()` | Bool | Opens root folder |
| `GotoParentFolder()` | Bool | Opens parent folder |
| `GetCurrentFolder()` | String | Returns current folder name |
| `ArchiveProject(projectName, filePath, ...)` | Bool | Archives project with configuration |
| `ImportProject(filePath, [projectName])` | Bool | Imports project from file |
| `ExportProject(projectName, filePath, [withStillsAndLUTs])` | Bool | Exports project to file |
| `RestoreProject(filePath, [projectName])` | Bool | Restores project from file |
| `GetCurrentDatabase()` | Dict | Returns current database connection info |
| `GetDatabaseList()` | [Dict] | Lists all databases added to Resolve |
| `SetCurrentDatabase({dbInfo})` | Bool | Switches current database |

---

## Project

Represents a DaVinci Resolve project.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetMediaPool()` | MediaPool | Returns Media Pool object |
| `GetTimelineCount()` | Int | Returns number of timelines |
| `GetTimelineByIndex(idx)` | Timeline | Returns timeline at index (1-based) |
| `GetCurrentTimeline()` | Timeline | Returns currently loaded timeline |
| `SetCurrentTimeline(timeline)` | Bool | Sets timeline as current |
| `GetGallery()` | Gallery | Returns Gallery object |
| `GetName()` | String | Returns project name |
| `SetName(projectName)` | Bool | Sets project name if unique |
| `GetPresetList()` | [String] | Returns available presets |
| `SetPreset(presetName)` | Bool | Sets preset by name |
| `AddRenderJob()` | String | Adds render job, returns unique ID |
| `DeleteRenderJob(jobId)` | Bool | Deletes render job by ID |
| `DeleteAllRenderJobs()` | Bool | Deletes all render jobs |
| `GetRenderJobList()` | [Dict] | Returns list of render jobs |
| `GetRenderJobStatus(jobId)` | Dict | Returns job status and completion % |
| `GetRenderPresetList()` | [String] | Lists render presets |
| `StartRendering([jobIds], [isInteractiveMode])` | Bool | Starts rendering jobs |
| `StopRendering()` | None | Stops render processes |
| `IsRenderingInProgress()` | Bool | Returns True if rendering active |
| `LoadRenderPreset(presetName)` | Bool | Sets render preset as current |
| `SaveAsNewRenderPreset(presetName)` | Bool | Creates new render preset |
| `DeleteRenderPreset(presetName)` | Bool | Deletes render preset |
| `SetRenderSettings({settings})` | Bool | Sets render settings |
| `GetRenderFormats()` | Dict | Returns available render formats |
| `GetRenderCodecs(renderFormat)` | Dict | Returns codecs for format |
| `GetCurrentRenderFormatAndCodec()` | Dict | Returns current format and codec |
| `SetCurrentRenderFormatAndCodec(format, codec)` | Bool | Sets render format and codec |
| `GetRenderResolutions([format, codec])` | [Dict] | Returns available resolutions |
| `GetSetting(settingName)` | String | Returns project setting value |
| `SetSetting(settingName, settingValue)` | Bool | Sets project setting |
| `GetUniqueId()` | String | Returns unique project ID |

---

## MediaPool

Manages media, clips, folders, and timeline creation.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetRootFolder()` | Folder | Returns root Folder |
| `AddSubFolder(folder, name)` | Folder | Adds subfolder under Folder |
| `RefreshFolders()` | Bool | Updates folders in collaboration mode |
| `CreateEmptyTimeline(name)` | Timeline | Creates new empty timeline |
| `AppendToTimeline([clips])` | [TimelineItem] | Appends clips to current timeline |
| `AppendToTimeline([{clipInfo}])` | [TimelineItem] | Appends clips with metadata |
| `CreateTimelineFromClips(name, [clips])` | Timeline | Creates timeline with clips |
| `DeleteTimelines([timelines])` | Bool | Deletes specified timelines |
| `GetCurrentFolder()` | Folder | Returns currently selected Folder |
| `SetCurrentFolder(Folder)` | Bool | Sets current folder |
| `DeleteClips([clips])` | Bool | Deletes clips from media pool |
| `ImportMedia([items])` | [MediaPoolItems] | Imports files into media pool |
| `MoveClips([clips], targetFolder)` | Bool | Moves clips to target folder |
| `MoveFolders([folders], targetFolder)` | Bool | Moves folders to target folder |
| `RelinkClips([clips], folderPath)` | Bool | Updates folder location of clips |
| `UnlinkClips([clips])` | Bool | Unlinks media pool clips |
| `ExportMetadata(fileName, [clips])` | Bool | Exports metadata to CSV |
| `GetSelectedClips()` | [MediaPoolItems] | Returns selected clips |
| `SetSelectedClip(MediaPoolItem)` | Bool | Sets selected clip |

---

## Timeline

Represents a sequence/timeline.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetName()` | String | Returns timeline name |
| `SetName(timelineName)` | Bool | Sets timeline name if unique |
| `GetStartFrame()` | Int | Returns start frame number |
| `GetEndFrame()` | Int | Returns end frame number |
| `SetStartTimecode(timecode)` | Bool | Sets start timecode |
| `GetStartTimecode()` | String | Returns start timecode |
| `GetTrackCount(trackType)` | Int | Returns number of tracks ("video", "audio", "subtitle") |
| `AddTrack(trackType, [subTrackType])` | Bool | Adds track (subTrackType: "mono", "stereo", etc.) |
| `DeleteTrack(trackType, trackIndex)` | Bool | Deletes track |
| `GetTrackSubType(trackType, trackIndex)` | String | Returns audio track format |
| `SetTrackEnable(trackType, trackIndex, Bool)` | Bool | Enables/disables track |
| `GetIsTrackEnabled(trackType, trackIndex)` | Bool | Returns track enabled state |
| `SetTrackLock(trackType, trackIndex, Bool)` | Bool | Locks/unlocks track |
| `GetIsTrackLocked(trackType, trackIndex)` | Bool | Returns track locked state |
| `GetItemListInTrack(trackType, index)` | [TimelineItem] | Returns items on track |
| `DeleteClips([timelineItems], [Bool])` | Bool | Deletes items (ripple delete if True) |
| `SetClipsLinked([timelineItems], Bool)` | Bool | Links/unlinks items |
| `GetCurrentTimecode()` | String | Returns playhead position timecode |
| `SetCurrentTimecode(timecode)` | Bool | Sets playhead position |
| `GetCurrentVideoItem()` | TimelineItem | Returns current video item |
| `GetCurrentClipThumbnailImage()` | Dict | Returns thumbnail data (base64 RGB) |
| `GetTrackName(trackType, trackIndex)` | String | Returns track name |
| `SetTrackName(trackType, trackIndex, name)` | Bool | Sets track name |
| `DuplicateTimeline([timelineName])` | Timeline | Duplicates timeline |
| `CreateCompoundClip([timelineItems], [{clipInfo}])` | TimelineItem | Creates compound clip |
| `CreateFusionClip([timelineItems])` | TimelineItem | Creates Fusion clip |
| `InsertGeneratorIntoTimeline(generatorName)` | TimelineItem | Inserts generator |
| `InsertTitleIntoTimeline(titleName)` | TimelineItem | Inserts title |
| `GrabStill()` | GalleryStill | Grabs still from current clip |
| `GrabAllStills(stillFrameSource)` | [GalleryStill] | Grabs stills (1=first, 2=middle) |
| `Export(fileName, exportType, exportSubtype)` | Bool | Exports timeline |
| `GetSetting(settingName)` | String | Returns timeline setting |
| `SetSetting(settingName, settingValue)` | Bool | Sets timeline setting |
| `GetMarkers()` | Dict | Returns markers as {frameId: {info}} |
| `AddMarker(frameId, color, name, note, duration, [customData])` | Bool | Adds marker |
| `DeleteMarkersByColor(color)` | Bool | Deletes markers by color |
| `GetUniqueId()` | String | Returns unique timeline ID |

---

## TimelineItem

Represents a clip on the timeline.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetName()` | String | Returns clip name |
| `SetName(name)` | Bool | Sets clip name |
| `GetDuration([subframe_precision])` | Int/Float | Returns duration (fractional if True) |
| `GetEnd([subframe_precision])` | Int/Float | Returns end frame position |
| `GetStart([subframe_precision])` | Int/Float | Returns start frame position |
| `GetSourceStartFrame()` | Int | Returns media source start frame |
| `GetSourceEndFrame()` | Int | Returns media source end frame |
| `GetLeftOffset([subframe_precision])` | Int/Float | Returns maximum left extension |
| `GetRightOffset([subframe_precision])` | Int/Float | Returns maximum right extension |
| `GetFusionCompCount()` | Int | Returns number of Fusion comps |
| `GetFusionCompByIndex(compIndex)` | FusionComp | Returns Fusion comp by index |
| `GetFusionCompNameList()` | [String] | Lists Fusion comp names |
| `GetFusionCompByName(compName)` | FusionComp | Returns Fusion comp by name |

---

## MediaPoolItem

Represents a clip in the media pool.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetName()` | String | Returns clip name |
| `SetName(name)` | Bool | Sets clip name |
| `GetMediaId()` | String | Returns unique media ID |
| `GetMetadata([metadataType])` | String/Dict | Returns metadata value or all metadata |
| `SetMetadata(metadataType, metadataValue)` | Bool | Sets metadata |
| `SetMetadata({metadata})` | Bool | Sets multiple metadata |
| `GetThirdPartyMetadata([metadataType])` | String/Dict | Returns third-party metadata |
| `SetThirdPartyMetadata(metadataType, metadataValue)` | Bool | Sets third-party metadata |
| `AddMarker(frameId, color, name, note, duration, [customData])` | Bool | Adds marker |
| `GetMarkers()` | Dict | Returns markers |
| `GetMarkerByCustomData(customData)` | Dict | Returns marker by custom data |
| `DeleteMarkersByColor(color)` | Bool | Deletes markers by color |
| `DeleteMarkerAtFrame(frameNum)` | Bool | Deletes marker at frame |
| `GetClipProperty(propertyName)` | String/Dict | Returns clip property |
| `SetClipProperty(propertyName, propertyValue)` | Bool | Sets clip property |
| `AddFlag(color)` | Bool | Adds flag |
| `GetFlagList()` | [String] | Returns flag colors |
| `ClearFlags(color)` | Bool | Clears flags (supports "All") |
| `GetClipColor()` | String | Returns clip color |
| `SetClipColor(colorName)` | Bool | Sets clip color |
| `ClearClipColor()` | Bool | Clears clip color |
| `LinkProxyMedia(proxyMediaFilePath)` | Bool | Links proxy media |
| `UnlinkProxyMedia()` | Bool | Unlinks proxy media |
| `ReplaceClip(filePath)` | Bool | Replaces clip asset |
| `GetUniqueId()` | String | Returns unique ID |

---

## Folder

Represents a folder in the media pool.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetClipList()` | [MediaPoolItem] | Returns clips in folder |
| `GetSubFolderList()` | [Folder] | Returns subfolders |
| `GetName()` | String | Returns folder name |
| `GetIsFolderStale()` | Bool | Returns stale status in collaboration mode |
| `GetUniqueId()` | String | Returns unique folder ID |
| `Export(filePath)` | Bool | Exports folder to DRB file |

---

## MediaStorage

Provides access to file system and media locations.

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetMountedVolumeList()` | [String] | Lists mounted volumes |
| `GetSubFolderList(folderPath)` | [String] | Lists subfolders in path |
| `GetFileList(folderPath)` | [String] | Lists files/media in path |
| `RevealInStorage(path)` | Bool | Expands and displays path in Media Storage |
| `AddItemListToMediaPool([items])` | [MediaPoolItem] | Adds files to media pool |
| `AddClipMattesToMediaPool(MediaPoolItem, [paths], [stereoEye])` | Bool | Adds mattes to clip |

---

## Key Concepts

### Markers

Markers are metadata points on clips or timelines with color coding and custom data.

```python
# Add marker to media pool item
clip.AddMarker(frameId=96, color="Green", name="Marker 1", 
               note="Important point", duration=1.0)

# Get all markers
markers = clip.GetMarkers()
# Returns: {96.0: {'color': 'Green', 'duration': 1.0, 'note': '', 'name': 'Marker 1'}, ...}

# Delete markers by color
clip.DeleteMarkersByColor("Red")
```

### Flags

Quick visual indicators on media pool items.

```python
# Add flag
clip.AddFlag("Red")

# Get flags
flags = clip.GetFlagList()  # Returns: ["Red", "Blue"]

# Clear specific flag
clip.ClearFlags("Red")

# Clear all flags
clip.ClearFlags("All")
```

### Properties & Settings

Access clip, timeline, and project properties using key-value pairs.

```python
# Get clip property
super_scale = clip.GetClipProperty("Super Scale")

# Set clip property
clip.SetClipProperty("Super Scale", 2)

# Get all properties at once
all_props = clip.GetClipProperty()

# Project settings
frame_rate = project.GetSetting("timelineFrameRate")
project.SetSetting("timelineFrameRate", "25")
```

### Data Structures

- **Lists**: Denoted by `[ ... ]` - arrays of values
- **Dicts**: Denoted by `{ ... }` - key-value pairs

```python
# Example dict
job_status = {
    'status': 'Rendering',
    'percentage': 45
}

# Example list
timelines = [timeline1, timeline2, timeline3]
```

---

## Common Properties

### Super Scale Property
- **Values**: 1 (no scaling), 2-4 (2x, 3x, 4x multipliers)
- **For 2x Enhanced**: 4 arguments with sharpness and noise reduction
  ```python
  clip.SetClipProperty('Super Scale', 2, 0.8, 0.5)  # sharpness=0.8, noiseReduction=0.5
  ```

### Timeline Frame Rate
- **Format**: "25", "29.97", "24", etc.
- **Drop Frame**: Append "DF" to enable (e.g., "29.97 DF")
  ```python
  project.SetSetting('timelineFrameRate', "29.97 DF")
  ```

### Cloud Sync Status
```python
resolve.CLOUD_SYNC_DOWNLOAD_IN_QUEUE      # 0
resolve.CLOUD_SYNC_DOWNLOAD_IN_PROGRESS   # 1
resolve.CLOUD_SYNC_DOWNLOAD_SUCCESS       # 2
resolve.CLOUD_SYNC_DOWNLOAD_FAIL          # 3
resolve.CLOUD_SYNC_UPLOAD_IN_QUEUE        # 5
resolve.CLOUD_SYNC_UPLOAD_SUCCESS         # 7
```

---

## Examples

### Create a Project and Timeline

```python
import DaVinciResolveScript as dvr_script

resolve = dvr_script.scriptapp("Resolve")
projectManager = resolve.GetProjectManager()

# Create project
project = projectManager.CreateProject("My Project")

# Get media pool
mediaPool = project.GetMediaPool()

# Create empty timeline
timeline = mediaPool.CreateEmptyTimeline("Main Sequence")

# Set as current timeline
project.SetCurrentTimeline(timeline)
```

### Import Media and Create Timeline

```python
# Import media files
media_items = mediaPool.ImportMedia(["/path/to/video.mov", "/path/to/audio.mp3"])

# Create timeline from imported clips
timeline = mediaPool.CreateTimelineFromClips("Edited Timeline", media_items)
```

### Add Markers

```python
# Add marker to current timeline
timeline.AddMarker(frameId=100, color="Red", name="Scene Start", 
                   note="Important moment", duration=5.0)

# Add marker to media pool item
clip.AddMarker(frameId=50, color="Green", name="Good Take", 
               note="Use this take", duration=1.0)
```

### Render Project

```python
# Add render job
job_id = project.AddRenderJob()

# Set render settings
project.SetRenderSettings({
    "TargetDir": "/output/path",
    "CustomName": "MyVideo",
    "VideoQuality": "High"
})

# Start rendering
project.StartRendering([job_id])

# Check status
while project.IsRenderingInProgress():
    status = project.GetRenderJobStatus(job_id)
    print(f"Rendering: {status['percentage']}%")
```

---

## Tips & Best Practices

1. **Always check return values** when setting properties
2. **Use unique project/timeline names** to avoid conflicts
3. **Index is 1-based** for all collections (not 0-based)
4. **Set environment variables** before running external scripts
5. **Inspect objects** using Python's `dir()` or Lua's introspection
6. **Test in Console first** before deploying scripts
7. **Handle errors gracefully** with try-catch blocks

---

## References

- [Official Examples](./Examples/)
- [Python Module](./Modules/DaVinciResolveScript.py)
- Consult DaVinci Resolve User Manual for UI feature details

