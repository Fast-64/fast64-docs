# Blender

The blender is fed the output from the Color Combiner & Alpha Compare unit. It checks whether or not pixels should be written with the Z Compare unit then blends the chosen input colors with the framebuffer, fog & other values.

## Z Compare Unit

The Z Compare unit calculates and tests Z values for a given pixel out with a value from the z_buffer. This test is enabled with the `z_cmp` flag enabled. If this flag is off, pixels are always output. If the `z_upd` flag is written, values calculated in this unit will be written out to the z_buffer.

The values used for calculation are z_px which is the pixel's z value, and dz_max which is the max delta Z. z_px and dz_max are either the depth values of the current pixel, or user programmed ones. This selection is made with the `z_src_sel` option. If you chose `z_src_prim` then a constant value for both options is made. With the selected z variables, the following calculations are made:

```C
farther = (z_px+dz_max) >= mem_z; // pixel is farther than mem val

nearer = (z_px-dz_max) <= mem_z; // pixel is nearer than mem val

in front = z_px < mem_z; // pixel is in front of mem val

max = mem_z == z_far (0x3ffff); // mem val is the max z value

overflow =  (memcvg + curpx_cvg) & 8; // Basically edges of primitives but not always
```

These values then test whether or not a pixel is written depends on the `z_mode`. Each `z_mode` has its own test.
- opaque - [return (max || (overflow ? infront : nearer));]{.code}
- interpenetrating:
	- [return (max || (overflow ? infront : nearer))]{.code}
	- 1 if [(!infront || !farther || !overflow);]{.code} e.g. not edge and z~=mem_z aka intersecting
- xlu - [return (infront || max);]{.code}
- decal - [return (farther && nearer \&\& !max);]{.code}

## Inputs

The blender has 4 inputs, `p`, `m`, `a`, `b`. P and M are color inputs, while A, B are alpha inputs
P&M inputs:
- 1st cycle: combined color, 2nd cycle: 1st cycle numerator
- framebuffer color (aka mem_clr)
- blend color
- fog color

A inputs:
- combined alpha
- fog alpha
- shade alpha
- 0

B inputs:
- 1-A
- fog alpha
- 1
- 0

## Outputs

While blending, the output color is [{math}`p*m + b*a`]{.math-literal}. While doing Anti Aliasing, the output is [{math}`(p*m + b*a) / (a + b)`]{.math-literal}. When not blending, the output is either:
- M if clr_on_cvg is on and no cvg overflow occurs
- P otherwise

There is no clamping on the final output color, so overflow occurs at 0xFF
and wraps back to 0.

## Blending

The formula for determining whether or not blending occurs is:
```c
blend_en = wstate->other_modes.force_blend || (!overflow && wstate->other_modes.antialias_en && farther);
```
Which basically means blending occurs when the `force_bl` flag is on or when anti aliasing is occuring. Anti aliasing occurs when `aa_en` is enabled and there is no cvg overflow.


## Coverage

Coverage represents the amount of each pixel the tri covers. Generally speaking, this value is 1 except for tri edges, which
have partial values. Coverage is a 4 bit number, 1.0 = 8. Coverage stored in memory is 3 bits long making a max of 7; one is subtracted from the output coverage before storing.

Pixel coverage is calculated by taking a 4x4 subpixel mask of a pixel, and adding up the bits the primitive will write to. A checkerboard mask is
used on this 4x4 grid, and each black square covered adds up to equal total coverage.

```{figure} coverage_mask.png
:align: center
:alt: mask of coverage bit calculation
Bits used to calculate coverage per pixel. Each bit adds to total pixel coverage.
```

The coverage then used for blending is memory coverage plus the pixel coverage. Overflow occurs
at [overflow = (mem_cvg + curpixel_cvg) & 8]{.code}.
A coverage of 0 is not written. The framebuffer is implicitly set to have cvg of 1.0 each frame.
Coverage can be used/edited with the following flags:

* cvg_x_alpha - multiplies combiner alpha and cvg and uses as cvg
* alpha_cvg_sel - uses cvg value for alpha input to blender
* clr_on_cvg - normal out on cvg overflow, otherwise uses M input. This essentially triggers on shared tri edges.

The coverage stored to the fb is affected by the cvg_dst flag:

* clamp - if blending (e.g. aa_en/force_bl) clamp to 7, else store curpixel_cvg - 1
* wrap - store the calculated coverage by wrapping, e.g. (mem_cvg + curpixel_cvg) & 7.
* full - stores a value of 7
* save - stores mem_cvg

### Edge mismatch

Edge mismatches occur when one edge of a primitive renders a different output than the compliment edge of a continguous primitive. This occurs due to
differences in blending and coverage overflow. The first rendered edge on a pixel will always overflow and first edge's output will then be stored to `mem_clr` which may be used in compliment edge blending. Memory coverage starts at 7, so {math}`x + 7 > 7: x>0` guaranteeing overflow. The coverage stored is based on what `cvg_dst_*` flags you have enabled, for this example an average pixel coverage of 4 for both edges will be assumed.
 - clamp stores new; [4-1 = 3]{.code} or 7 if `force_bl`
 - wrap stores total;  [(7+4) & 7 = 3]{.code}
 - full & save store 7 ( not always but basically unless explicitly set to not that)
 
The compliment edge will not overflow if using `cvg_dst_wrap`, or if using `cvg_dst_clamp` & `!force_bl`, both of which will cause AA.
You can also miss overflow if `cvg_x_alpha` is as edge_cvg + compliment_cvg will be < 1. Consider the following cases:
 1. [7+4 & 8 == 8]{.code} (full&save); ovflw
 2. [7+4 & 8 == 8]{.code} (clamp & force bl); ovflw
 3. [3+4 & 8 == 0]{.code} (wrap & clamp !force_bl); !ovflw
 4. [(4\*0.125:= 0) + 7 < 8]{.code} (`cvg_x_alpha`) !ovflw

With these cases in mind, edge mismatch occurs when:
- e1 blends(w/ mem_clr input), e2 blends (w/ mem_clr input) -> blending is disproportionate against mem_clr
	- `force_bl` && !`clr_on_cvg` in case 3 or 4
	- `force_bl` && case 1 or 2
- e1 blends(w/ mem_clr input), e2 M out != mem_clr -> blending will not occur only on edges
	- `force_bl` && `clr_on_cvg` in case 3 or 4 w/ `M != mem_clr`
- e1 P out, e2 M out or blends (w/ no mem_clr input && P != M) -> output (blend||M) on edges will not match rest of primitive
	- `!force_bl` && `clr_on_cvg` in case 3 or 4 w/ `M != mem_clr`
	- `!force_bl` && `!clr_on_cvg` in case 3 or 4 w/ `M != P`
	- `!force_bl` && `!clr_on_cvg` && `!aa_en` in case 3 or 4 w/ `M != P`
	