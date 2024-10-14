# Upper/Lower modes

The upper and lower tabs feature a slew of various options for unique features the RCP is capable of performing. These features offer core control over how rendering occurs on the N64, and some options have considerable depth.

## RGB/Alpha Dither

Dithering affects the final color that is output to the framebuffer. It is used to reduce artifacts and banding when converting higher precision source colors from various sources into a lower precision framebuffer (typically RGBA16). 8 bits are used per RGB component in the RDP pipeline, and it is written to a 5 bit per channel framebuffer. Try changing the types of dithering if banding occurs.

* Pattern - Alpha only option. Follows the same dithering as rgb or uses Bayer
* NotPattern - Alpha only option. Dithers with the opposite pattern as "Pattern"
* Magic Square - RGB only option. Uses a magic square for dithering
* Bayer - RGB only option. Uses a Bayer matrix for dithering
* Noise - Random dithering
* Enable - Option just for backwards compatibility. Equivalent to "Noise"
* Disable - No dithering is used.

## Chroma Key

Chroma keying allows you to specify colors as pixel alpha. A combiner with equation {eq}`chroma_key_combiner_f3d_mat` is recommended for use and should be used in the last available cycle. The combiner will use the `A` value put in variable A as the color output, and the alpha out will be the min value from each color channel of {eq}`key_alpha_calc_f3d_mat`.
In order to use chroma key, `Key` must be selected in the chroma key dropdown and [alpha_cvg_sel]{.key_text} must be off in the chosen render mode.

:::{card}
$$
combined\_color = (A - key\_center)*key\_scale + 0
$$(chroma_key_combiner_f3d_mat)
$$
key\_alpha = clamp(0, key\_width - abs(combined\_color), 255)
$$(key_alpha_calc_f3d_mat)
:::

## Texture Convert/Filter

Texture Convert allows for the enabling of YUV conversion, and texture filtering. Filtering type is controlled by the Texture Filter option. For filtering, bilinear is standard, point filtering is also known as "nearest neighbor". Average is a special type of filter only for pixel/texel aligned rendering.

## Texture LOD/Detail

Texture LOD enables the use of Level of Detail (LOD) tiles. Texture Detail controls the type of LOD used. LOD calculations start with texture 0 being tile 0, and increase based on the texel/pixel ratio up to a max based on the number of mipmaps setting. The Min LOD Ratio is used when in LOD mode clamp, and governs how low the LOD can get while under magnification (texel/pixel < 1). For other LOD modes, this value is locked at 0.5.

## Texture Perspective Correction

This enables the correction of rasterized triangles with skew. Without correction, texture coordinates would distort when triangles are moved in relation to the camera. This is typically enabled for 3D geometry and disabled for 2D sprites (tex rects).

## Cycle Type

1 and 2 Cycle enable the full rendering pipeline and are the default options, with 2 cycle enabling additional options for combiner and render mode selection. 2 Cycle is required for loading multiple textures, LoD, or doing certain types of blending (like with fog).

Copy mode will copy the loaded texture to the framebuffer directly, skipping most of the rasterization pipeline except for alpha compare.
As such, you are limited to only one texture with the same type as the framebuffer (usually RGBA16), indexed formats of the same type count (CI8 rgba16 for example)

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

If you are using 2 cycle mode, then the render mode in the first cycle generally should be [G_RM_PASS]{.key_text} (Pass in dropdown) or a special mode such as fog.

If you are using copy or fill cycle modes, then the render mode should be [G_RM_NOOP]{.key_text} (No Op in dropdown)  for both cycles.