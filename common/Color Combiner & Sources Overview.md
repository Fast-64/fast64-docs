# Color Combiner & Sources

Color and alpha values for pixels are determined using the color combiner. The color combiner takes various source values such as textures, solid colors, lights and for each pixel applies a formula of {math}`Color = (A-B)*C + D` to get a resultant value out for both color and alpha. This value is sent to the blender for combining the pixel and alpha with existing pixel colors or additional inputs such as fog.

## Color Combiner
In the F3D material inspector, the Combiner tab offers you a selection of 8 inputs in one cycle mode, and 16 in two cycle mode. The inputs in the left represent the RGB color that will be used for combining color, while the ones on the left aptly labeled "Alpha" will be for alpha combining.

Each color channel independently performs math operations following the color combiner formula. When given an RGB input, the color will use each channel as its own value, but when given a constant value, such as 0, 1 or a grayscale texture, then all channels will be given that value as input.

When in two cycle mode, two sets of combiner inputs appear. Values from the output of the first cycle are available to the second cycle under the variable called [combined]{.key_text}.

:::{admonition} Note
The value of `combined` is not valid in 1 cycle, and using it will result in noise patterns and unmapped values being used.
:::

:::{admonition} Note
When in 2 cycle mode and in the second cycle, `Texel0` and `Texel1` are swapped, and `Texel1` maps to the next pixel (in tile 0).
:::

## Sources

Each input available to edit will be one of the inputs selected in the combiner tab. Inputs available can be one of following types:

### Textures

Textures map the source image to pixels using UVs. There are several texture types and different bitsizes available to use.
:::{card} Allowed Texture Formats

- 4-bit intensity (I)
- 4-bit intensity w/alpha (I/A) (3/1)
- 4-bit color index (CI)
- 8-bit I
- 8-bit IA (4/4)
- 8-bit CI
- 16-bit red, green, blue, alpha (RGBA) (5/5/5/1)
- 16-bit IA (8/8)
- 16-bit YUV (Luminance, Blue-Y, Red-Y)
- 32-bit RGBA (8/8/8/8)
:::
% exact properties covered in their own article: (add this when it exists)%.

The combiner supports two textures for use, named [Texture 0]{.key_text} and [Texture 1]{.key_text}. You can load up to 4KiB into texture memory (TMEM) at once using any type but Color Indexed (CI), which only supports 2KiB, the other 2KiB is reserved for palettes.

Fast64 will calculate TMEM usage when selecting a texture. If your texture is too large, you will have to resize it, or use **Large Texture Mode**. When using large textures, you will have to subdivide the mesh so that there are an equal number of faces as TMEM size divisions; this can be done automatically using the **Create Large Texture Mesh** tool which is located in the Fast64 Tools panel in the 3D viewport.

### Geometry Influenced

Some sources vary their color based on the geometry in the viewport. Only the `Shade` and `LOD Fraction` inputs in the combiner do this.

[Shade]{.key_text} changes its color based on the geometry mode you have selected. If you are using lighting, then the shade color is based on the `Light` and `Ambient` color. The color will vary between the two depending on the direction of your lighting. If you are not using lighting, then the shade color will be based on vertex colors.

Shade alpha has its own quirks. Shade alpha will be from vertex alpha unless the `fog` geo mode is enabled, then shade alpha will become the fog value. Fog will change from 0 to 1 depending on how far away the mesh triangles are from the camera. The fog intensity depends on the fog factors chosen.

[LOD Fraction]{.key_text} is a variable that determines how far between LOD levels the current texel will be. This value shifts between 0 and 1 as the camera moves farther away from the target texel. Once reaching one, the value will wrap back to zero until the max LOD level is reached. This value is only enabled while using LOD calculations.

### Color Registers

Color registers are RGBA values stored in the RDP and used for combiner operations. These registers hold a discrete value until they are changed by a new material. The RGB values selected will be used under the alias `<X> Color` while the alpha component will be under `<X> Color Alpha`, where <X> represents the color registers name. All colors are constant rgba colors, though some registers have alternative usages or ad hoc purposes beyond being a free use color.

* Environment Color: Free use register.

* Primitive Color: Free use register. Buffered so that it can be changed between without requiring syncs.

* Chroma Key Center: Made to support chroma key usage. While not using chroma key, simply exists as a free use RGB register.

### Discrete Values

Discrete values are adjustable float values between 0 and 1 that you can use for combining or interpolating colors. These are simply factors that you can use to fine tune the contribution of each input to the color combiner. Some registers are created with specific graphical purpose, while others are just for user flexibility.

* Primitive LOD Fraction: This register exists simply for free use

* Chroma Key Scale: Made to support chroma key usage. While not using chroma key, simply exists as a free use RGB register.

* YUV Convert K4, YUV Convert K5: Both registers are made to support YUV to RGB conversion. If not converting, these registers are free to use.

You can also use the alpha component of color registers primitive and environment for interpolation. They are set at the same time as the colors are.