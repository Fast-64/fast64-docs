# Color Combiner

In one and two cycle mode, the color combiner takes input from the texturing unit. The color combiner is fed various inputs, and combines them via user set inputs. Pixel output from the color combiner is fed into the chroma key & alpha fix up unit and then into the alpha compare unit.

## Inputs

The color combiner has inputs a, b, c, d for color and alpha separately.

:::::{tab-set}
::::{tab-item} A Color Inputs
* Combined Color - 0
* Texel 0 Color - 1
* Texel 1 Color - 2
* Primitive Color - 3
* Shade Color - 4
* Environment Color - 5
* 1.0 - 6
* Noise - 7
* 0.0 - 8-15
::::
::::{tab-item} B Color Inputs
* Combined Color - 0
* Texel 0 Color - 1
* Texel 1 Color - 2
* Primitive Color - 3
* Shade Color - 4
* Environment Color - 5
* Key Center - 6
* Convert K4 - 7
* 0.0 - 8-15
::::
::::{tab-item} C Color Inputs
* Combined Color - 0
* Texel 0 Color - 1
* Texel 1 Color - 2
* Primitive Color - 3
* Shade Color - 4
* Environment Color - 5
* Key Scale - 6
* Combined Alpha - 7
* Texel 0 Alpha - 8
* Texel 1 Alpha - 9
* Primitive Alpha - 10
* Shade Alpha - 11
* Environment Alpha - 12
* LOD Fraction - 13
* Primitive LOD Fraction - 14
* Convert K5 - 15
* 0.0 - 16-31
::::
::::{tab-item} D Color Inputs
* Combined Color - 0
* Texel 0 Color - 1
* Texel 1 Color - 2
* Primitive Color - 3
* Shade Color - 4
* Environment Color - 5
* 1.0 - 6
* 0.0 - 7
::::
::::{tab-item} A Alpha Inputs
* Combined Alpha - 0
* Texel 0 Alpha - 1
* Texel 1 Alpha - 2
* Primitive Alpha - 3
* Shade Alpha - 4
* Environment Alpha - 5
* 1.0 - 6
* 0.0 - 7
::::
::::{tab-item} B Alpha Inputs
* Combined Alpha - 0
* Texel 0 Alpha - 1
* Texel 1 Alpha - 2
* Primitive Alpha - 3
* Shade Alpha - 4
* Environment Alpha - 5
* 1.0 - 6
* 0.0 - 7
::::
::::{tab-item} C Alpha Inputs
* Texel 0 Alpha - 1
* Texel 1 Alpha - 2
* Primitive Alpha - 3
* Shade Alpha - 4
* Environment Alpha - 5
* 1.0 - 6
* 0.0 - 7
::::
::::{tab-item} D Alpha Inputs
* Combined Alpha - 0
* Texel 0 Alpha - 1
* Texel 1 Alpha - 2
* Primitive Alpha - 3
* Shade Alpha - 4
* Environment Alpha - 5
* 1.0 - 6
* 0.0 - 7
::::
:::::

Inputs values are [q1.8  {math}`[0-511]`]{.math-literal} where [{math}`256 == 1.0`]{.math-literal}.
Input variables `A`, `B`, and `D` use a special sign extension [{math}`[-0.5 - 1.5)`]{.math-literal}, while `C` uses uses two's compliment [{math}`[-1.0 - 1.0)`]{.math-literal}.

:::{admonition} Note
1.0 is the only value set to 256 specifically, while others are set to a max of 255. This can allow for overflow via combined input on `C` if passing 1.0.
:::
% (check YUV K5 s1.7 as c input)

There are pipeline bugs when using Texel 0 and Texel 1 in the combiner. The pixel accessed for source colors changes depending on which cycle you are in, and which pipeline mode you are in. Texel 0 becomes Texel 1 while in the second cycle. If you are in one cycle mode, Texel 1 is undefined behavior, but generally it will be equal to the next pixel value of Texel 0.
::::{card}
:::{table} Texel Pixel Usage 1-Cycle Mode
:align: center
| --    | TEXEL0  | TEXEL1    |
| --    | ---     | ---       |
| cycle | tex0[n] | tex0[n+1] |
:::
:::{table} Texel Pixel Usage 2-Cycle Mode
:align: center
| ---       | TEXEL0  | TEXEL1    |
| ---       | ---     | ---       |
| 1st cycle | tex0[n] | tex1[n]   |
| 2nd cycle | tex1[n] | tex0[n+1] |
:::
+++
tex0 represents the current render tile<br>
tex1 represents the current render tile + 1<br>
Indexing represents individual pixel access
::::

## Outputs

The combiner equation {math}`Color = (A-B)*C + D` is performed on each cycle. The output of the first cycle is fed to the 2nd cycle under the [combined_X]{.key_text} input, where X is color or alpha. The final combined output is subject to the special sign extension [{math}`[-0.5 - 1.5)`]{.math-literal}.
Output of the color combiner value goes to Alpha Fix Up & Chroma Key Units.

## Chroma Key

Chroma key applies alpha based on the distance from the pixel color from the key value. Chroma key will decrease alpha from a max of [key_width]{.key_text} based on the distance of [key_center]{.key_text} from a chosen source color. The scale of that alpha change is based on [key_scale]{.key_text} where [{math}`(key\_scale == 1.0/edge\_size)`]{.math-literal}. Higher scales result in faster changes in alpha. A scale of 255 is the equivalent of hard edge keying.

The color combiner expects the last used cycle to have a setup of {eq}`chroma_key_combiner`. After which, the chroma key unit will perform {eq}`key_alpha_calc`, with the min being taken over the r, g, b channels, to get the [key_alpha]{.key_text}. Chroma key calculations are done before special sign extension and downshifting on combined_color, which means combined_color will be a [s1.16 - [0-0x1ffff]]{.math-literal} value.

`key_center`, and `key_scale` are [q0.7  {math}`[0-255]`]{.math-literal} values, while `key_width` is a [s7.4  {math}`[0-4096]`]{.math-literal} value. A width over 1.0 disables chroma key calculations, upon which key_alpha is set to 0.
:::{card}
$$
combined\_color = (A - key\_center)*key\_scale + 0
$$(chroma_key_combiner)
$$
key\_alpha = clamp(0, key\_width - abs(combined\_color), 255)
$$(key_alpha_calc)
:::
When `key_en` is enabled, the combined color is replaced with the `A_Color` combiner input.
If `alpha_cvg_sel` is disabled, the combined alpha is replaced with key_alpha.

## Alpha Fix Up

The alpha fix up applies changes to coverage and combined alpha based on rdp flag selection:

* Shade alpha is clamped to 0xff (since it is blender input).
* cvg_x_alpha - multiplies combined alpha and cvg and uses as cvg (not key alpha), alpha values of 0xff are set to 0x100 for this calc (not that important tbh)
* alpha_cvg_sel - uses cvg value for alpha out (overwrites key alpha)

## Alpha Compare

Alpha compare is performed after alpha fix up unit. The alpha compare unit takes the cycle 1 combined alpha and compares it to a test value. Alpha compare can be enabled in either threshold or dither modes.
* Threshold will disable writing if combined_alpha < threshold
* Dither will disable writing if combined_alpha < random value

:::{admonition} Note
In 2 cycle mode, alpha compare uses the combined alpha from cycle 1. It will also use the next pixel coverage if `alpha_cvg_sel` is enabled.
:::
