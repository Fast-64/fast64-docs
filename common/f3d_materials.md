# F3D Material Overview

F3D materials offer an accurate way of recreating N64 graphics within blender. The materials contain all the inputs and outputs that N64 graphics using Nintendo's standard "Fast 3D" material system uses, with almost no concessions with regards to visual accuracy.

F3D materials are can be modified in either a simple or full UI. Using the simplified UI will allow you to select from preset options, and change the applicable input variables as needed, while the full UI will unlock all F3D options available. Each section within the full UI represent an important chunk of the F3D material system, and should be understood to make full use of the N64s graphical capabilities.

:::::{tab-set}

::::{tab-item} Color Combiner & Sources

Color and alpha values for pixels are determined using the color combiner. The color combiner takes various source value such as textures, solid colors, lights and for each pixel applies a formula of {math}`Color = (A-B)*C + D` to get a resultant value out for both color and alpha. This value is sent to the blender for combining the pixel and alpha with existing pixel colors or additional inputs such as fog.

## Color Combiner
In the F3D material inspector, the Combiner tab offers you a selection of 8 inputs in one cycle mode, and 16 in two cycle mode; by default one cycle mode is selected. The inputs in the left represent the RGB color that will be used for blending color, while the ones on the left aptly labeled "Alpha" will be for alpha blending.

Each color channel independently performs math operations following the color combiner formula. When given an RGB input, the color will use each channel as its own value, but when given a constant value, such as 0, 1 or a grayscale texture, then all channels will be given that value as input.

When in two cycle mode, two sets of combiner inputs appear. Values from the output of the first cycle are output to the second cycle under the variable called [combined]{.key_text}.

:::{admonition} Note
The value of `combined` is not valid in 1 cycle, and using it will result in noise patterns and unmapped values being used.
:::

:::{admonition} Note
When in 2 cycle mode, `Texel0` and `Texel1` are swapped, and `Texel1` maps to the next pixel in memory.
:::

## Sources

Each input available to edit will be one of the inputs selected in the combiner tab. Inputs available can be one of following types:

## Textures

Textures map the output of the source image to pixels using UVs. There are several texture types available to use, and many properties which is covered in their own article: (link I guess).

The combiner supports two textures for use, named [Texture 0]{.key_text} and [Texture 1]{.key_text}. You can load up to 4kb into texture memory (TMEM) at once using any type but Color Indexed (CI), which only supports 2kb, the other 2kb is reserved for palettes.

Fast64 will calculate TMEM usage when selecting a texture. If you're texture is too large, you will have to resize it, or use **Large Texture Mode**. When using large textures, you will have to subdivide the mesh so that there are an equal number of faces as TMEM size divisions; this can be done automatically using the **Create Large Texture Mesh** tool which is located in the Fast64 Tools panel in the 3D viewport.

## Geometry Influenced

Some registers vary their color based on the geometry in the viewport. Only the `Shade` and `LOD Fraction` inputs in the combiner do this.

[Shade]{.key_text} changes it's color based on the geometry mode you have selected. If you are using lighting, then the shade color is based on the `Light` and `Ambient` color. The color will vary between the two depending on the direction of your lighting. If you are not using lighting, then the shade color will be based on vertex colors.

Shade alpha has its own quirks. Shade alpha will be vertex colors unless the `fog` geo mode is enabled, then shade alpha will become the fog value. Fog will change from 0 to 1 depending on how far away the mesh triangles are from the camera. The range of travel depends on the fog intensity chosen.

[LOD Fraction]{.key_text} is a variable that determines how far between LOD levels the current texel will be. This value shifts between 0 and 1 as the camera moves farther away from the target texel. Once reaching one, the value will wrap back to zero until the max LOD level is reached. This value is only enabled while using LOD calculations.

## Color Registers

Color registers are RGBA values that are stored in Display Memory (DMEM) and used for combiner operations. These registers hold a discrete value until they are changed by a new material. The RGB values selected will be used under the alias `<X> Color` while the alpha component will be under `<X> Color Alpha`, where <X> representing the color registers name. All colors are constant rgba colors, though some registers have alternative usages or ad hoc purposes beyond being a free use color.

* Environment Color: Free use register

* Primitive Color: Free use register. Encoded at the same time as LOD variables but those have no impact on the color.

* Chroma Key Center: Made to support chroma key usage. While not using chroma key, simply exists as a free use RGB register.

## Discrete Values

Discrete values are adjustable float values between 0 and 1 that you can use for blending or interpolating colors. These are simply factors that you can use to fine tune the contribution of each input to the color combiner. Some registers are created with specific graphical purpose, while others are just for user flexibility.

* Primitive LOD Fraction This register exists simply for free use

* Chroma Key Scale: Made to support chroma key usage. While not using chroma key, simply exists as a free use RGB register.

* YUV Convert K4, YUV Convert K5: Both registers are made to support YUV to RGB conversion. If not converting, these registers are free to use.

::::
::::{tab-item} Geo

The geo tab refers to geometry mode selection, which affects how triangles are processed while loading or drawing. Some options require other options to be enabled, or change what values certain variables will refer to. These nuances should be described with tooltips when hovering over the options in fast64.

## Z Buffer

Enabling [Z buffer]{.key_text} will make it so that Z buffer calculations are carried out on drawn triangles. This makes it so that triangles can utilize depth checking per pixel if enabled inside the RDP. Disabling Z buffer will change the drawing algorithm of the triangles sent to the RDP to ignore depth. Depth values are never written to triangles in DMEM and when rasterizing depth is never considered for the entire primitive.

## Shading

Enabling [Shade]{.key_text} will make it so shade coefficients are calculated and stored while loading the triangle. These values are used for the `Shade` and `Shade Alpha` input to the combiner. If disabled, coefficients will not be calculated, and instead the previous values left in DMEM will be used.

Disabling shade will also disable fog, as that uses the `Shade Alpha` variable to function.

## Cull Front/Back

Culling will disable the drawing of triangles whose normals face (front) and don't face (back) the camera. Frontface culling is unfortunately not displayed in fast64 as a limitation of blender.

## Fog

The [Fog]{.key_text} geo mode enables the calculation of fog coefficients, which is stored in shade alpha. Fog is blended on top of the combiner output by setting a render mode to an appropriate one with fog.

## Lighting

Enabling [Lighting]{.key_text} will turn on light calculations of shading is enabled. Light colors will be the combination of ambient, plus the affect of directional lights on the surface of triangles.

## Texture UV Generation/Linear

UV generation will create UVs for the triangle based on the triangles normals. Lighting must be enabled to use either of these options.

[Texture UV Generate]{.key_text} will create UV values equal to the component of the screen space normal on an interval of `[0, 1]`. [Texture UV Generate Linear]{.key_text} will take the trig decomposition those normals instead. Consider the following conversions from normals to UV coordinates.

```{card} Texture Gen Conversions
:text-align: center
{math}`Gen(-1, 0) \rightarrow (0, 0.5) \hspace{1cm} GenLinear(-1, 0)\rightarrow (0, 0.5)`
{math}`Gen(-\sqrt{2}, \sqrt{2}) \rightarrow (-\sqrt{2}, \sqrt{2}) \hspace{1cm} GenLinear(-\sqrt{2}, \sqrt{2})\rightarrow (0.25, 0.75)`
{math}`Gen(0, 1) \rightarrow (0.5, 1) \hspace{1cm} GenLinear(0, 1)\rightarrow (0.5, 1)`
```

## Smooth Shading

With Shading is enabled, then [Smooth Shading]{.key_text} will interpolate between the vertices of each triangle to create a smooth color. This is typically known as Gouraud Shading.

::::
::::{tab-item} Upper/Lower

The upper and lower tabs feature a slew of various options for unique features the RCP is capable of performing. These features offer core control over how rendering occurs on the N64, and some options have considerable depth.

## RGB/Alpha Dither

Dithering affects the final color that is output to the framebuffer. The affect is used to reduce artifacts and banding when converting higher precision source colors from varoius sources into the lower precision framebuffer. 8 bits are used per RGB component in the RDP pipeline, and it is written to a 5 bit per channel framebuffer. Try changing the types of dithering if banding occurs.

* Pattern - Alpha only option. Follows the same dithering as rgb or uses Bayer
* NotPattern - Alpha only option. Dithers with the opposite pattern as "Pattern"
* Magic Square - RGB only option. Uses a magic square for dithering
* Bayer - RGB only option. Uses a Bayer matrix for dithering
* Noise - Random dithering
* Enable - Option just for backwards compatibility. Equivalent to "Noise"
* Disable - No dithering is uses.

## Chroma Key

Chroma keying allows you to specify colors as pixel alpha. A combiner with equation {eq}`chroma_key_combiner` is recommended for use and should be used in the last available cycle. The combiner will use the `A` value put in variable A as the color output, and the alpha out will be the min value from each color channel of {eq}`key_alpha_calc`.
In order to use chroma key, `Key` must be selected in the chroma key dropdown and [alpha_cvg_sel]{.key_text} must be off in the chosen render mode.

:::{card}
$$
combined\_color = (A - key\_center)*key\_scale + 0
$$(chroma_key_combiner)
$$
key\_alpha = clamp(0, key\_width - abs(combined\_color), 255)
$$(key_alpha_calc)
:::

## Texture Convert/Filter

Texture Convert allows for the enabling of YUV conversion, and texture filtering. Filtering type is controlled by the Texture Filter option. For filtering, billinear is standard, and point is for no filtering. Average is a special type of filter only for pixel/texel aligned rendering.

## Texture LOD/Detail

Texture LOD enables the use of Level of Detail (LOD) tiles. Texture Detail controls the type of LOD used. LOD calculations start with texture 0 being tile 0, and increase based on the texel/pixel ratio up to a max based on the number of mipmaps setting. The Min LOD Ratio is used when in LOD mode clamp, and governs how low the LOD can get while under magnification (texel/pixel < 1). For other LOD modes, this value is locked at 0.5.

## Texture Perspective Correction

This enables the correction of rasterized triangles with skew. Without correction, triangle colors would distort when triangles are moved in relation to the camera.

## Cycle Type

1 and 2 Cycle enable the full rendering pipeline and are the default options, with 2 cycle enabling additional options for combiner and render mode selection. 2 Cycle is required for loading multiple textures or doing certain types of blending (like with fog).

Copy mode will copy loaded textures to the framebuffer directly with no blending or perspective correction.

Fill mode will fill the space defined by a fill rectangle with the fill color. This is typically used to clear the screen.

:::{admonition} Note
When using fill or copy, the render mode should be set to :G_RM_NOOP:.
:::

## Pipeline Span Buffer Coherency

1 Primitive basically adds in extra pauses during rendering for debugging. Unless you are using vanilla SM64 this is not necessary. If you have a graphics crash you may consider using this, otherwise use N primitive, which executes graphics cmds as written in display lists.

## Alpha Compare

Alpha compare will determine if pixels are written based on passed alpha values from the color combiner. In threshold mode, the pixel is written if combined alpha >= Blend alpha. In dither, the pixel is written if the alpha is >= a random value. Alpha compare can be used in 1 or 2 cycle and copy mode.

:::{admonition} Note
Due to a bug in the RDP, only the first cycle output will be used for alpha compare calculations.
:::

## Z Source Selection

This option determines where the Z values used for depth comparison are retrieved from. Pixel will use the triangle values to calculate a depth per pixel, while primitive will use a constant chosen value for the entire primitive.

## Render Mode

The render mode determines how rendering is performed. In general games use a system of preset options for each type of primitive. If those values are not satisfactory, you can choose from a set of options in a drop down for different types of surfaces, or manually choose individual options.

If you are using 2 cycle mode, then the render mode in the first cycle generally should be [G_RM_PASS]{.key_text} (pass in dropdown) or a special mode such as fog.

::::

:::::