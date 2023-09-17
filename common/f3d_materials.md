## F3D Materials

F3D materials offer an accurate way of recreating N64 graphics within blender. The materials contain all the inputs and outputs that N64 graphics using Nintendo's standard "Fast 3D" material system uses, with almost no concessions with regards to visual accuracy.

F3D materials are can be modified in either a simple or full UI. Using the simplified UI will allow you to select from preset options, and change the applicable input variables as needed, while the full UI will unlock all F3D options available. Each section within the full UI represent an important chunk of the F3D material system, and should be understood to make full use of the N64s graphical capabilities.

## Combiner

Colors on F3D materials are determined using the color combiner. In the F3D material inspector, the Combiner tab offers you a selection of 8 inputs in one cycle mode, and 16 in two cycle mode; by default one cycle mode is selected. Each grouping of four inputs follows the format of `Color = (A-B)*C + D)`. The inputs in the left represent the RGB color that will be mapped to rasterized triangles, while the ones on the left aptly labeled "Alpha" will map to alpha output.

Each color channel independently performs math operations following the color combiner formula. When given an RGB input, the color will use each channel as its own value, but when given a constant value, such as 0, 1 or a grayscale texture, then all channels will be given that value as input.

When in two cycle mode, two sets of combiner inputs appear. Values from the output of the first cycle are output to the second cycle under the variable called "Combined".

## Sources

Each input available to edit will be one of the inputs selected in the combiner tab. Inputs available can be one of following types:

#### Textures

Textures map the output of the source image to pixels using UVs. There are several texture types available to use, and many properties which is covered in their own article: (link I guess).

The combiner supports two textures for use, aptly named "Texture 0" and "Texture 1". You can load up to 4kb into texture memory (TMEM) at once using any type but Color Indexed (CI), which only support 2kb. Fast64 will calculate TMEM usage when selecting a texture. If you're texture is too large, you will have to resize it, or use "Large Texture Mode". When using large textures, you will have to subdivide the mesh so that there are an equal number of faces as TMEM size divisions; this can be done automatically using the "Create Large Texture Mesh" tool which is located in the "Fast64 Tools" panel in the 3D viewport.

#### Geometry Influenced

Some registers vary their color based on the geometry in the viewport. Only the "shade" and "LOD Fraction" input in the combiner do this.

Shade changes it's color based on the geometry mode you have selected. If you are using lighting, then the shade color is based on the Light and Ambient color. The color will vary between the two depending on the direction of your lighting. If you are not using lighting, then the shade color will be based on vertex colors.

Shade alpha has its own quirks. Shade alpha will be vertex colors unless fog is enabled, then shade alpha will become the fog value. Fog will change from 0 to 1 depending on how far away the mesh triangles are from the camera. The range of travel depends on the fog intensity chosen.

LOD fraction is a variable that determines how far between LOD levels the current texel will be. This value shifts between 0 and 1 as the camera moves farther away from the target texel. Once reaching one, the value will wrap back to zero until the max LOD level is reached. This value is only enabled while using LOD calculations.

#### Color Registers

Color registers are RGBA values that are stored in Display Memory (DMEM) and used for combiner operations. These registers hold a discrete value until they are changed by a new material. The RGB values selected will be used under the alias "X Color" while the alpha component will be under "X Color Alpha", which X representing the color registers name. All colors act as simply colors, though some registers have alternative usages or ad hoc purposes beyond being a free use color.

* Environment Color: Free use register

* Primitive Color: Free use register. Encoded at the same time as LOD variables but those have no impact on the color.

* Chroma Key Center: Made to support chroma key usage. While not using chroma key, simply exists as a free use RGB register.

#### Discrete Values

Discrete values are adjustable float values between 0 and 1 that you can use for blending or interpolating colors. These are simply factors that you can use to fine tune the contribution of each input to the color combiner. Some registers are created with specific graphical purpose, while others are just for user flexibility.

* Primitive LOD Fraction: This register exists simply for free use

* Chroma Key Scale: Made to support chroma key usage. While not using chroma key, simply exists as a free use RGB register.

* YUV Convert K4, K5: Both registers are made to support YUV to RGB conversion. If not converting, these registers are free to use.

## Geo

The geo tab refers to geometry mode selection, which affects how triangles are processed while loading or drawing.

#### Z Buffer

Enabling Z buffer will make it so that Z buffer calculations are carried out on drawn triangles. This makes it so that triangles can utilize depth checking per pixel if enabled inside the RDP. Disabling Z buffer will change the drawing algorithm of the triangles sent to the RDP to ignore depth. Depth values are never written to triangles in DMEM and when rasterizing depth is never considered for the entire primitive.

#### Shading

Enabling shade will make it so shade coefficients are calculated and stored while loading the triangle. These values are used for the "Shade" input to the combiner. If disabled, coefficients will not be calculated, and instead the previous values left in DMEM will be used.

Disabling shade will not affect the use of shade alpha as a fog variable if fog is enabled.

#### Cull Front/Back

Culling will disable the drawing of triangles whose normals face (front) and don't face (back) the camera. Frontface culling is unfortunately not displayed in fast64 as a limitation of blender.

#### Fog

The fog geo mode enables the calculation of fog coefficients, which is stored in shade alpha. Fog is blended on top of the combiner output by setting a render mode to an appropriate one with fog.

#### Lighting

Enabling lighting will turn on light calculations of shading is enabled. Light colors will be the combination of ambient, plus the affect of directional lights on the surface of triangles.

#### Texture UV Generation/Linear

UV generation will create UVs for the triangle based on the triangles normals. Lighting must be enabled to use either of these options.

Default Texture UV Generate will create UV values equal to the component of the screen space normal on an interval of [0, 1]. Texture UV Generate Linear will take the trig decomposition those normals instead. Consider the following conversions from normals to UV coordinates.

$$
Gen(-1, 0) \rightarrow (0, 0.5) \hspace{1cm} GenLinear(-1, 0)\rightarrow (0, 0.5)\\
Gen(-\sqrt{2}, \sqrt{2} \rightarrow (-\sqrt{2}, \sqrt{2}) \hspace{1cm} GenLinear(-\sqrt{2}, \sqrt{2})\rightarrow (0.25, 0.75)\\
Gen(0, 1) \rightarrow (0.5, 1) \hspace{1cm} GenLinear(0, 1)\rightarrow (0.5, 1)\\
$$

#### Smooth Shading

If shading is enabled, then smooth shading will interpolate between the vertices of each triangle to create a smooth color. This is typically known as Gouraud Shading.

## Upper/Lower

The upper and lower tabs feature a slew of various options for unique features the RCP is capable of performing. These features offer core control over how rendering occurs on the N64, and some options have considerable depth.

#### RGB/Alpha Dither

Dithering affects the final color that is output to the framebuffer. The affect is used to reduce artifacts and banding when converting higher precision source colors from varoius sources into the lower precision framebuffer. 8 bits are used per RGB component in the RDP pipeline, and it is written to a 5 bit per channel framebuffer. Try changing the types of dithering if banding occurs.

#### Chroma Key

Chroma keying allows you to specify colors as tests for alpha comparison. A combiner of `(input - KeyCenter)*KeyScale + 0` is required in the second cycle and you must have alpha compare enabled.

#### Texture Convert/Filter

Texture Convert allows for the enabling of YUV conversion, and texture filtering. Filtering type is controlled by the Texture Filter option. For filtering, billinear is standard, and point is for no filtering. Average is a special type of filter only for pixel/texel aligned rendering.

#### Texture LOD/Detail

Texture LOD enables the use of Level of Detail (LOD) tiles. Texture Detail controls the type of LOD used. LOD calculations start with texture 0 being tile 0, and increase based on the texel/pixel ratio up to a max based on the number of mipmaps setting. The Min LOD Ratio is used when in LOD mode clamp, and governs how low the LOD can get while under magnification (texel/pixel < 1). For other LOD modes, this value is locked at 0.5.

#### Texture Perspective Correction

This enables the correction of rasterized triangles with skew. Without correction, triangle colors would distort when triangles are moved in relation to the camera.

#### Cycle Type

1 and 2 Cycle enable the full rendering pipeline and are the default options, with 2 cycle enabling additional options for combiner and render mode selection. 2 Cycle is required for loading multiple textures or doing certain types of blending (like with fog).

Copy mode will copy loaded textures to the framebuffer directly with no blending or perspective correction.

Fill mode will fill the space defined by a fill rectangle with the fill color. This is typically used to clear the screen.

#### Pipeline Span Buffer Coherency

1 Primitive basically adds in extra pauses during rendering for debugging. Unless you are using vanilla SM64 this is not necessary. If you have a graphics crash you may consider using this, otherwise use N primitive, which executes graphics cmds as written in display lists.

#### Alpha Compare

Alpha compare will determine if pixels are written based on passed alpha values from the color combiner. In threshold mode, the pixel is written if the value is >= Blend alpha. In dither, the pixel is written if the alpha is >= a random value.

#### Z Source Selection

This option determines where the Z values used for depth comparison are retrieved from. Pixel will use the triangle values to calculate a depth per pixel, while primitive will use a constant chosen value for the entire primitive.

#### Render Mode

The render mode determines how rendering is performed. In general games use a system of preset options for each type of primitive. If those values are not satisfactory, you can choose from a set of options in a drop down for different types of surfaces, or manually choose individual options.

If you are using 2 cycle mode, then the render mode in the first cycle must be `G_RM_PASS` (pass in dropdown) or a special mode such as fog.