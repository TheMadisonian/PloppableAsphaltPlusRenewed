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
- ✅ **Automatic settings migration** from old PloppableAsphaltFix.xml (legacy mod)

### Settings Migration

**On first load, the mod will automatically:**

1. Detect the old `PloppableAsphaltFix.xml` settings file (from the previous mod version)
2. Import your custom asphalt color settings
3. Write the settings to the new location (`Cities_Skylines/ModsSettings/PloppableAsphaltRenewed.xml`)

**Manual cleanup:** After migration, you can safely delete the old `PloppableAsphaltFix.xml` file from your main `Cities_Skylines` folder if you wish. Your settings are now stored in the new location.

### Surface Rendering Fixes

- Preserved original texture behavior: non-decal props (asphalt, cliffgrass, gravel) only set **`_APRMap`** from the original `_ACIMap` texture, matching the original mod exactly.
- The `ploppableasphalt-decal` mesh continues to set `_MainTex` to `m_upwardDiffuse` as in the original.
- Used `HasProperty()` guards and null checks on `_APRMap` assignment to avoid shader errors.
- Ensured the size/prop modifications don't crash when `m_generatedInfo` or `m_mesh` are null, which previously prevented surfaces from being processed at all.

### Additional Null Reference Protections

- Added guards in `SetRenderProperties`, `SetRenderPropertiesAll`, `ApplyProperties`, and `ApplyColors` loops to skip props with `m_mesh == null` or missing generated info.
- Fixed `DesaturationControl` handling with null checks.



### Testing Notes

The fix ensures:
1. No null reference exceptions when `NetProperties` isn't immediately available
2. Graceful handling if any intermediate property is null
3. Compatibility with new game version
4. Alignment with the game's standard singleton access patterns used throughout Assembly-CSharp
