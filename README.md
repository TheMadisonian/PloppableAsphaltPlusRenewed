# Ploppable Asphalt Renewed

## Version Update - Game Compatibility Fix
- Updated for Race Day. 

#### Fixed NullReferenceException in ApplyProperties() method
- **Issue**: Mod was crashing with `System.NullReferenceException: Object reference not set to an instance of an object` when applying properties to ploppable asphalt decals during game load.
- **Root Cause**: The mod was using `UnityEngine.Object.FindObjectOfType<NetProperties>()` to access the `NetProperties` singleton, which could return null if the `NetProperties` object hadn't been instantiated yet in the scene.
- **Solution**: Replaced scene-based lookup with the reliable Singleton pattern used by the game itself:
  - Changed from: `UnityEngine.Object.FindObjectOfType<NetProperties>().m_upwardDiffuse`
  - Changed to: `Singleton<NetManager>.instance.m_properties.m_upwardDiffuse`
- **Impact**: The `NetManager` singleton is guaranteed to be initialized before mods run during gameplay, making this approach much more reliable.

#### Added comprehensive null checks
- Added validation for `Singleton<NetManager>.instance` before accessing properties
- Added validation for `netManager.m_properties` to ensure it exists
- Added validation for `netManager.m_properties.m_upwardDiffuse` before setting texture
- Added null checks for `prefab.m_material` and `prefab.m_lodMaterial` before texture assignment
- **Impact**: Prevents crashes and allows graceful degradation if any property is unavailable

#### Added missing namespace import
- Added `using ColossalFramework;` to enable access to the Singleton class
- **Impact**: Ensures the mod compiles correctly with the proper Singleton pattern

### Compatibility

- ✅ Tested against new game version
- ✅ Verified that `NetProperties` structure is consistent across versions
- ✅ Confirmed compatibility with the game's standard access patterns
- ✅ Mod compiles without warnings or errors

### Surface Rendering Fixes

- Added logic to populate **`_MainTex`** and **`_XYSMap`** on props when switching to road shaders, using textures from `NetProperties`.
- Added fallback for `_APRMap` (uses original `_ACIMap` or net diffuse if missing).
- Used `HasProperty()` guards just like the game does to avoid shader errors.
- Ensured the size/prop modifications don't crash when `m_generatedInfo` or `m_mesh` are null, which previously prevented surfaces from being processed at all.

### Additional Null Reference Protections

- Added guards in `SetRenderProperties`, `SetRenderPropertiesAll`, `ApplyProperties`, and `ApplyColors` loops to skip props with `m_mesh == null` or missing generated info.
- Fixed `DesaturationControl` handling with null checks.

### Files Modified

- `PloppableAsphalt.cs`
  - Line 1: Added `using ColossalFramework;` import
  - Lines 433-447: Updated `ApplyProperties()` method to use Singleton pattern with proper null checks

### Testing Notes

The fix ensures:
1. No null reference exceptions when `NetProperties` isn't immediately available
2. Graceful handling if any intermediate property is null
3. Compatibility with new game version
4. Alignment with the game's standard singleton access patterns used throughout Assembly-CSharp
