# Geometry Modes

The geo tab refers to geometry mode selection, which affects how triangles are processed while loading or drawing. Some options require other options to be enabled, or change what values certain variables will refer to. These nuances should be described with tooltips when hovering over the options in fast64.

## Z Buffer

Enabling [Z buffer]{.key_text} will make it so that Z calculations are carried out on loaded triangles. This makes it so that triangles can utilize depth checking per pixel if enabled inside the RDP. Disabling Z buffer will make it so Z values are never sent to the RDP. If you do depth testing you will have garbage values used for z calculations causing undefined behavior. If you disable, make sure Z is not used in the render mode.

## Shading

Enabling [Shade]{.key_text} will make it so shade coefficients are calculated and stored while loading the triangle. These values are used for the `Shade` and `Shade Alpha` input to the combiner. If disabled, coefficients will not be calculated, and instead garbage values from the RDP's command buffer will be used.

Disabling shade will also disable fog, as that uses the `Shade Alpha` variable to function.

## Cull Front/Back

Culling will disable the drawing of triangles whose normals face (front) and don't face (back) the camera. Frontface culling is unfortunately not displayed in fast64 as a limitation of Blender.

## Fog

The [Fog]{.key_text} geo mode enables the calculation of fog coefficients, which is stored in shade alpha. Fog is blended on top of the combiner output by setting a render mode to an appropriate one with fog.

## Lighting

Enabling [Lighting]{.key_text} will turn on light calculations of shading is enabled. Light colors will be the combination of ambient, plus the affect of directional lights on the surface of triangles.

## Texture UV Generation/Linear

UV generation will create UVs for the triangle based on the triangles normals. Lighting must be enabled to use either of these options.

[Texture UV Generate]{.key_text} will create UV values equal to the component of the screen space normal on an interval of `[0, 1]`. [Texture UV Generate Linear]{.key_text} will take the trig decomposition those normals instead. Consider the following conversions of normals to UV coordinates.

```{card} Texture Gen Conversions
:text-align: center
{math}`Gen(-1, 0) \rightarrow (0, 0.5) \hspace{1cm} GenLinear(-1, 0)\rightarrow (0, 0.5)`
{math}`Gen(-\sqrt{2}, \sqrt{2}) \rightarrow (-\sqrt{2}, \sqrt{2}) \hspace{1cm} GenLinear(-\sqrt{2}, \sqrt{2})\rightarrow (0.25, 0.75)`
{math}`Gen(0, 1) \rightarrow (0.5, 1) \hspace{1cm} GenLinear(0, 1)\rightarrow (0.5, 1)`
```

In an intuitive sense, UV generation's goal is to project one tile onto the visible portion of a mesh. Base UV generation will appear as if it is scrunching at the sides, giving a curved appearance, while UV generation linear will have the same density across the entire projection.

## Smooth Shading

With Shading is enabled, then [Smooth Shading]{.key_text} will interpolate between the vertices of each triangle to create a smooth color. This is typically known as Gouraud Shading.