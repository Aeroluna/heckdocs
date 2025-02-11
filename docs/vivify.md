# Vivify

Bring your map to life.

If you use any of these features, you MUST add "Vivify" as a requirement for your map for them to function, you can
go [Here](https://github.com/Kylemc1413/SongCore/blob/master/README.md) to see how adding suggestions/requirements to
the info.dat works. Also, Vivify will only load on v3 maps.

This documentation assumes basic understanding of custom events and tracks.

### Event Types

- [`SetMaterialProperty`](#setmaterialproperty)
- [`SetGlobalProperty`](#setglobalproperty)
- [`Blit`](#blit)
- [`CreateCamera`](#createcamera)
- [`CreateScreenTexture`](#createscreentexture)
- [`InstantiatePrefab`](#instantiateprefab)
- [`DestroyObject`](#destroyobject)
- [`SetAnimatorProperty`](#setanimatorproperty)
- [`SetCameraProperty`](#setcameraproperty)
- [`AssignObjectPrefab`](#assignobjectprefab)
- [`SetRenderSettings`](#setrendersettings)

## Setting up Unity

First, you should download the Unity Hub at https://unity3d.com/get-unity/download. Beat Saber 1.29.1 uses version
2019.4.28f and Beat Saber 1.30.0+ uses 2021.3.16f1. For maximum compatibility, you should use 2019.4.28f found in
the [archive](https://unity3d.com/get-unity/download/archive) and use Swifter's VivifyTemplate to build your
bundles https://github.com/Swifter1243/VivifyTemplate. 2019 bundles built without using that script will have
non-functioning shader keywords in 2021.

Make sure you have `Virtual Reality Supported` enabled in your project and your stereo rendering mode is set
to `Single Pass`. (Edit > Project Settings > Player > XR Settings > Deprecated Settings > Virtual Reality Supported).

## Writing VR shaders

Beat Saber v1.29.1 uses Single Pass Stereo rendering (See https://docs.unity3d.com/2019.4/Documentation/Manual/SinglePassStereoRendering.html for more info). Use the unity
built-in function `UnityStereoTransformScreenSpaceTex` to fix your shaders in vr.

```csharp
sampler2D _MainTex;

fixed4 frag (v2f i) : SV_Target
{
  return tex2D(_MainTex, UnityStereoTransformScreenSpaceTex(i.uv));
}
```

Beat Saber v1.30.0+ uses Single Pass Instanced rendering. Any incompatible shaders will only appear in the left eye. To
make your shader compatible with this vr rendering method, add instancing support to your shader.
See https://docs.unity3d.com/Manual/SinglePassInstancing.html for how to add instancing support. Look under "
Post-Processing shaders" to see how to sample a screen space texture.

A tip for writing shaders, there are many commonly used structs/functions in UnityCG.cginc. As a few
examples, `appdata_base`, `appdata_img`, `appdata_full`, and `v2f_img` can usually be used instead of writing your own
structs and since most image effect shaders use the same vertex function, the include file has a `vert_img` that can be
used with `#pragma vertex vert_img`.

### Creating an asset bundle

Use https://github.com/Swifter1243/VivifyTemplate to create the bundle. (TODO: more instructions here)

(Optional) See https://docs.unity3d.com/Manual/AssetBundles-Browser.html. this tool allows you to browse the contents of
a built asset bundle.

Bundles should be placed in your map folder and called either `bundleWindows2019.vivify`, `bundleWindows2021.vivify`. Although Quest
support does not exist yet, bundles should still be built for them and be called `bundleAndroid2021.vivify`.

```
Map Folder
├── bundleWindows2019.vivify
├── bundleWindows2021.vivify
├── bundleAndroid2021.vivify
├── song.ogg
├── cover.jpg
├── ExpertPlusStandard.dat
└── info.dat
```

When referencing an asset's file path in an event, remember to write in all lower case. You can use the above Asset
Bundle Browser tool to see the path of specific assets.

By default, when Vivify will check against a checksum when loading an asset bundle, but this checksum check can be
disabled by enabling debug mode using the `-aerolunaisthebestmodder` launch parameter. You can add the checksum to the
map by using the `"_assetBundle"` field in the info.dat.

```json5
  ...
  "_environmentName": "DefaultEnvironment",
  "_allDirectionsEnvironmentName": "GlassDesertEnvironment",
  "_customData": {
    "_assetBundle": {
      "_windows2019": 1414251160,
      "_windows2021": 6436275894,
      "_android2021": 4262884586,
    }
  },
  "_difficultyBeatmapSets": [
    {
  ...
```

## SetMaterialProperty

```json5
{
  "b": float, // Time in beats.
  "t": "SetMaterialProperty",
  "d": {
    "asset": string, // File path to the desired material.
    "duration": float, // The length of the event in beats. Defaults to 0.
    "easing": string, // An easing for the animation to follow. Defaults to "easeLinear".
    "properties": [{
      "id": string, // Name of the property on the material.
      "type": string, // Type of the property (Texture, Float, Color, Vector, Keyword).
      "value": ? // What to set the property to, type varies depending on property type.
    }]
  }
}
```

Allows setting material properties, e.g. Texture, Float, Color, Vector, Keyword.

## SetGlobalProperty

```json5
{
  "b": float, // Time in beats.
  "t": "SetGlobalProperty",
  "d": {
    "duration": float, // The length of the event in beats. Defaults to 0.
    "easing": string, // An easing for the animation to follow. Defaults to "easeLinear".
    "properties": [{
      "id": string, // Name of the property.
      "type": string, // Type of the property (Texture, Float, Color, Vector, Keyword).
      "value": ? // What to set the property to, type varies depending on property type.
    }]
  }
}
```

Allows setting global properties, e.g. Texture, Float, Color, Vector, Keyword. These will persist even after the map ends, do not rely on
their default value.

### Property types

- Texture: Must be a string that is a direct path file to a texture.
- Float: May either be a direct value (`"value": 10.4`) or a point definition (`"value": [[0,0], [10, 1]]`).
- Color: May either be an RGBA array (`"value": [0, 1, 0]`) or a point
  definition (`"value": [1, 0, 0, 0, 0.2], [0, 0, 1, 0, 0.6]`)
- Vector: May either be an array (`"value": [0, 1, 0]`) or a point
  definition (`"value": [1, 0, 0, 0, 0.2], [0, 0, 1, 0, 0.6]`)
- Keyword: May either be a direct value (`"value": true`) or a point definition (`"value": [[0,0], [1, 1]]`) where values equal to or greater than 1 will enable the keyword.

```json5
// Example
{
  "b": 3.0,
  "t": "SetMaterialProperty",
  "d": {
    "asset": "assets/screens/glitchmat.mat",
    "duration": 8,
    "properties": [{
      "id": "_Juice",
      "type": "Float",
      "value": [
        [0.02, 0],
        [0.04, 0.1875, "easeStep"],
        [0.06, 0.375, "easeStep"],
        [0.08, 0.5, "easeStep"],
        [0.1, 0.625, "easeStep"],
        [0.12, 0.75, "easeStep"]
      ]
    }]
  }
}
```

## Blit

```json5
{
  "b": float, // Time in beats.
  "t": "Blit",
  "d": {
    "asset": string, // (Optional) File path to the desired material. If missing, will just copy from source to destination without anything special.
    "priority": int, // (Optional) Which order to run current active post processing effects. Higher priority will run first. Default = 0
    "pass": int, // (Optional) Which pass in the shader to use. Will use all passes if not defined.
    "order": string, // (Optional) BeforeMainEffect, AfterMainEffect. Whether to activate before the main bloom effect or after. Defaults fo AfterMainEffect
    "source": string, // (Optional) Which texture to pass to the shader as "_MainTex". "_Main" is reserved for the camera. Default = "_Main"
    "destination": string, // (Optional) Which render texture to save to. Can be an array. "_Main" is reserved for the camera. Default = "_Main"
    "duration": float, // (Optional) How long will this material be applied. Defaults to 0
    "easing": string, // (Optional) See SetMaterialProperty.
    "properties": ? // (Optional) See SetMaterialProperty.
  }
}
```

Assigns a material to the camera. A duration of 0 will run for exactly one frame. If a destination is the same as
source, a temporary render texture will be created as a buffer.

This event allows you to call a [SetMaterialProperty](#SetMaterialProperty) from within.

```json5
// Example
{
  "b": 73.0,
  "t": "Blit",
  "d": {
    "asset": "assets/shaders/tvdistortmat.mat",
    "duration": 32,
    "properties": [
      {
        "id": "_Juice",
        "type": "Float",
        "value": 0.2
      }
    ]
  }
}
```

## CreateCamera

```json5
{
  "b": float, // Time in beats.
  "t": "CreateCamera",
  "d": {
    "id": string, // Id of the camera.
    "texture": string, // (Optional) Will render to a new texture set to this key.
    "depthTexture": string // (Optional) Renders just the depth to this texture.
    "properties": ? // (Optional) See SetCameraProperty
  }
}
```

Creates an additional camera that will render to the desired texture. Useful for creating a secondary texture where a certain track is culled.

```json5
// Example
{
  "b": 0.0,
  "t": "CreateCamera",
  "d": {
    "id": "NotesCam",
    "texture": "_Notes",
    "depthTexture": "_Notes_Depth",
    "properties": {
      "culling": {
        "track": "allnotes",
        "whitelist": true,
      },
      "depthTextureMode": ["Depth"]
    }
  }
}
```

```csharp
//Example where notes are not rendered on the right side of the screen
UNITY_DECLARE_SCREENSPACE_TEXTURE(_Notes);

fixed4 frag(v2f i) : SV_Target
{
  UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(i);
  if (i.uv.x > 0.5)
  {
    return UNITY_SAMPLE_SCREENSPACE_TEXTURE(_Notes, UnityStereoTransformScreenSpaceTex(i.uv));
  }
  else {
    return UNITY_SAMPLE_SCREENSPACE_TEXTURE(_MainTex, UnityStereoTransformScreenSpaceTex(i.uv));
  }
}
```

## CreateScreenTexture

```json5
{
  "b": float, // Time in beats.
  "t": "CreateScreenTexture",
  "d": {
    "id": string, // Name of the texture
    "xRatio": float, // (Optional) Number to divide width by, i.e. on a 1920x1080 screen, an xRatio of 2 will give you a 960x1080 texture.
    "yRatio": float, // (Optional) Number to divide height by.
    "width": int, // (Optional) Exact width for the texture.
    "height": int, // (Optional) Exact height for the texture.
    "colorFormat": string, // (Optional) https://docs.unity3d.com/ScriptReference/RenderTextureFormat.html
    "filterMode": string // (Optional) https://docs.unity3d.com/ScriptReference/FilterMode.html
  }
}
```

Declares a RenderTexture to be used anywhere. They are set as a global variable and can be accessed by declaring a
sampler named what you put in "id".

```json5
// Example
// Here we declare a texture called "snapshot", capture a single frame at 78.0, then store it in our new render texture.
// Lastly we destroy the texture (See below) after we are done with it to free up any memory it was taking.
// (Realistically, won't provide noticable boost to performance, but it can't hurt.)
{
  "b": 70.0,
  "t": "DeclareRenderTexture",
  "d": {
    "id": "snapshot"
  }
},
{
  "b": 78.0,
  "t": "Blit",
  "d": {
    "destination": "snapshot"
  }
},
{
  "b": 120.0,
  "t": "DestroyTexture",
  "d": {
    "id": "snapshot"
  }
}
```

## InstantiatePrefab

```json5
{
  "b": float, // Time in beats.
  "t": "InstantiatePrefab",
  "d": {
    "asset": string, // File path to the desired prefab.
    "id": string, // (Optional) Unique id for referencing prefab later. Random id will be given by default.
    "track": string, // (Optional) Track to animate prefab transform.
    "position": vector3, // (Optional) Set position.
    "localPosition": vector3, // (Optional) Set localPosition.
    "rotation": vector3, // (Optional) Set rotation (in euler angles).
    "localRotation": vector3. // (Optional) Set localRotation (in euler angles).
    "scale": vector3 //(Optional) Set scale.
  }
}
```

Instantiates a prefab in the scene. If left-handed option is enabled, then the position, rotation, and scale will be
mirrored.

## DestroyObject

```json5
{
  "b": float, // Time in beats.
  "t": "DestroyObject",
  "d": {
    "id": string or string[], // Id(s) of object to destroy.
  }
}
```

Destroys an object in the scene. Can be a prefab, camera, or texture id.

It is important to destroy any cameras created through `CreateCamera` because the scene
will have to be rendered again for each active camera. This can also be used for textures created
through `DeclareRenderTexture` to free up memory.

## SetAnimatorProperty

```json5
{
  "b": float, // Time in beats.
  "t": "SetAnimatorProperty",
  "d": {
    "id": string, // Id assigned to prefab.
    "duration": float, // (Optional) The length of the event in beats. Defaults to 0.
    "easing": string, // (Optional) An easing for the animation to follow. Defaults to "easeLinear".
    "properties": [{
      "id": string, // Name of the property.
      "type": string, // Type of the property (Bool, Float, Trigger).
      "value": ? // What to set the property to, type varies depending on property type.
    }]
  }
}
```

Allows setting animator properties. This will search the prefab for all Animator components.

### Property types

- Bool: May either be a direct value (`"value": true`) or a point definition (`"value": [[0,0], [1, 1]]`). Any value
  greater than or equal to 1 is true.
- Float: May either be a direct value (`"value": 10.4`) or a point definition (`"value": [[0,0], [10, 1]]`).
- Integer: May either be a direct value (`"value": 10`) or a point definition (`"value": [[0,0], [10, 1]]`). Value will
  be rounded.
- Trigger: Must be `true` to set trigger or `false` to reset trigger.

## SetCameraProperty

```json5
{
  "b": float, // Time in beats.
  "t": "SetCameraProperty",
  "d": {
    "id": string, // (Optional) Id of camera to affect. Default to "_Main".
    "properties": { 
      "depthTextureMode": [], // (Optional) Sets the depth texture mode on the camera. Can be [Depth, DepthNormals, MotionVectors].
      "clearFlags": string, // (Optional) Can be [Skybox, SolidColor, Depth, Nothing]. See https://docs.unity3d.com/ScriptReference/CameraClearFlags.html
      "backgroundColor": [], // (Optional) [R, G, B, (Optional) A] Color to clear screen with. Only used with SolidColor clear flag.
      "culling": { // (Optional) Sets a culling mask where the selected tracks are culled
          "track": string/string[], // Name(s) of your track(s). Everything on the track(s) will be added to this mask.
          "whitelist": bool // (Optional) When true, will cull everything but the selected tracks. Defaults to false.
      },
      "bloomPrePass": bool, // (Optional) Enable or disable the bloom pre pass effect.
      "mainEffect": bool // (Optional) Enable or disable the main bloom effect.
    }
  }
}
```

Setting any field to `null` will return it to its default. Remember to clear the `depthTextureMode` to `null` after you are done using it as rendering a depth texture can impact
performance. See https://docs.unity3d.com/Manual/SL-CameraDepthTexture.html for more info. Note: if the player has the
Smoke option enabled, the `depthTextureMode` will always have `Depth`.

## AssignObjectPrefab

```json5
{
  "b": float, // Time in beats.
  "t": "AssignObjectPrefab",
  "d": {
    "loadMode": string, // (Optional) How to load the asset (Single, Additive).
    "object": {} // See below
  }
}
```

Assigns prefabs to a specific object. Setting any asset to `null` is equivalent to resetting to the default model. Most
objects will have their per-instance properties set automatically. (See section "Adding per-instance properties to GPU
instancing shaders" at https://docs.unity3d.com/Manual/gpu-instancing-shader.html)

- `loadMode`: `Single, Additive`
    - `Single`: Clears all loaded prefabs on the object and adds a prefab
    - `Additive`: Adds a prefab to the currently loaded prefabs.
- `colorNotes`:
    - `track`: `string` Only notes on this track(s) will be affected.
    - `asset`: `string` (Optional) File path to the desired prefab. Only applies to directional notes. Sets
      properties `_Color`, `_Cutout`, and `_CutoutTexOffset`.
    - `anyDirectionAsset`: `string` (Optional) Only applies to dot notes. Sets same properties as directional notes.
    - `debrisAsset`: `string` (Optional) Applies to cut debris. Sets properties `_Color`, `_Cutout`, `_CutPlane`,
      and `_CutoutTexOffset`.
- `burstSliders`:
    - `track`: `string` See above.
    - `asset`: `string` (Optional) See above.
    - `debrisAsset`: `string` (Optional) See above.
- `burstSliderElemeents`:
    - `track`: `string` See above.
    - `asset`: `string` (Optional) See above.
    - `debrisAsset`: `string` (Optional) See above.
- `saber`:
    - `type`: `string` Which saber to affect. `Left`, `Right` or `Both`.
    - `asset`: `string` (Optional) File path to the desired prefab. Sets property `_Color`.
    - `trailAsset`: `string` (Optional) File path to the material to replace the trail. Sets property `_Color` and sets
      vertex colors for a gradient.
    - `trailTopPos`: `vector3` (Optional) Vector3 position of the top of the trail. Defaults to [0, 0, 1]
    - `trailBottomPos`: `vector3` (Optional) Vector3 position of the top of the trail. Defaults to [0, 0, 0]
    - `trailDuration`: `float` (Optional) Age of most distant segment of trail. Defaults to 0.4
    - `trailSamplingFrequency`: `int` (Optional) Saber position snapshots taken per second. Defaults to 50
    - `trailGranularity`: `int` (Optional) Segments count in final trail mesh. Defaults to 60

```json5
// Example
// Adds a cool particle system to your sabers!
{
  "b": 70.0,
  "t": "AssignObjectPrefab",
  "d": {
    "loadMode": "Additive",
    "saber": {
      "type": "Both",
      "asset": "assets/path/to/cool/particlesystem.prefab"
    }
  }
}
```

## SetRenderingSettings

```json5
{
  "b": float, // Time in beats.
  "t": "SetRenderingSettings",
  "d": {
    "duration": float, // (Optional) The length of the event in beats. Defaults to 0.
    "easing": string, // (Optional) An easing for the animation to follow. Defaults to "easeLinear".
    "category": {
        "property": value or point definition // The setting to set
    }
  }
}
```

Property does not have to be a point definition. When enabling a render setting with a performance cost, remember to disable it after you no longer need it to gain performance back.

Current provided settings:

`"renderSettings"`: https://docs.unity3d.com/ScriptReference/RenderSettings.html

- `"ambientEquatorColor"`: (color)
- `"ambientGroundColor"`: (color)
- `"ambientIntensity"`: (float)
- `"ambientLight"`: (color)
- `"ambientMode"`: (0, 1, 3, 4) Skybox, Trilight, Flat, Custom
- `"ambientSkyColor"`: (color)
- `"defaultReflectionMode"`: (0, 1) Skybox, Custom
- `"defaultReflectionResolution"`: (int)
- `"flareFadeSpeed"`: (float)
- `"flareStrength"`: (float)
- `"fog"`: (0, 1) Bool
- `"fogColor"`: (color)
- `"fogDensity"`: (float)
- `"fogEndDistance"`: (float)
- `"fogMode"`: (1, 2, 3) Linear, Exponential, ExponentialSquared
- `"fogEndDistance"`: (float)
- `"haloStrength"`: (float)
- `"reflectionBounces"`: (int)
- `"reflectionIntensity"`: (float)
- `"skybox"`: (string) File path to a material
- `"subtractiveShadowColor"`: (color)
- `"sun"`: (string) Id from InstantiatePrefab event, will find the first directional light on the top level GameObject

`"qualitySettings"`: https://docs.unity3d.com/ScriptReference/QualitySettings.html

- `"anisotropicFiltering"`: (0 - 2) Disable, Enable, ForceEnable.
- `"antiAliasing"`: (0, 2, 4, 8)
- `"pixelLightCount"`: (int)
- `"realtimeReflectionProbes"`: (0, 1) Bool
- `"shadowCascades"`: (0, 2, 4)
- `"shadowDistance"`: (float)
- `"shadowmaskMode"`: (0, 1) Shadowmask, DistanceShadowmask
- `"shadowNearPlaneOffset"`: (float)
- `"shadowProjection"`: (0, 1) CloseFit, StableFit
- `"shadowResolution"`: (0, 1, 2, 3) Low, Medium, High, VeryHigh.
- `"shadows"`: (0, 1, 2) Disable, HardOnly, All. WARNING: May cause random crashes, needs more investigation.
- `"softParticles"`: (0, 1) Bool

`"xrSettings"`: https://docs.unity3d.com/ScriptReference/XR.XRSettings.html

- `"useOcclusionMesh"`: (0, 1) Bool WARNING: Only works on 2019 versions.

```json5
// Example
{
  "b": 70.0,
  "t": "SetRenderingSettings",
  "d": {
    "qualitySettings": {
      "ambientLight": [0, 0, 0, 0]
    }
  }
}
```
