# tryColors
Ever wanted to generate harmonious color in any language? Look no further than Try Colors. Try and make good colors!

- # Obviously AI-generated spec, but it looks good and the implementations were done by me. Used on [my website](unatried.com), the [the specs site](trycolors.unatried.com), Lunarity and other places.

# Generation Spec
The Color System (zero to five input colors)

Input

The system takes between zero and five arbitrary colors, each given as a hex code. Duplicates are removed, and if more than five are supplied, only the first five are kept. Three-digit hex codes are expanded to six digits (ABC becomes AABBCC), and everything is stored in uppercase. Whatever is missing to reach five colors is generated, as described below. Every color is tagged as either source (supplied) or generated.

Internal values for each color

Each hex color is converted into a set of numbers used only for internal decisions:

1. RGB: the hex code split into red, green, and blue values from 0 to 255.
2. HSL: hue (0 to 360 degrees around the color wheel), saturation (0 to 100), and lightness (0 to 100), calculated from the RGB values.
3. Chroma: an approximate measure of how vivid the color looks. It is the saturation multiplied by a lightness penalty, where the penalty is one minus the distance of lightness from 50 divided by 100. A color at lightness 50 keeps its full saturation. A color near black or white is scaled toward zero, because very dark or very light colors look less vivid regardless of their saturation number. Chroma, therefore, runs from 0 to 100.
4. Perceived brightness: a weighted mix of red (0.299), green (0.587), and blue (0.114), divided by 255. This is used only to decide text color.

Color generation spec

If fewer than five colors are supplied, the missing ones are generated from the colors that were given. The strategy depends on how many were supplied.

Zero inputs, five generated:
First, a random hex color is generated. It is tagged as generated, not source, since the user did not supply it. Then the one-input rule below is applied to that random color, producing four more generated colors. The result is five generated colors in total. To avoid a weak starting point, the random color should be re-rolled if it is nearly gray or nearly black or white (for example, saturation below 20 or lightness below 15 or above 85), because the one-input rule scales its outputs from the starting saturation, and a dull start gives a dull palette.

One input, four generated:
Generated color A: hue rotated +32 degrees, saturation 85 percent of the source (capped at 85), and lightness 48.
Generated color B: hue rotated +150 degrees, saturation 80 percent of the source (capped at 80), and lightness 48.
Generated color C: hue rotated +210 degrees, saturation 82 percent of the source (capped at 82), and lightness 45.
Generated color D: same hue as the source, saturation 22 percent of the source (capped at 25), and lightness 55. This is a muted, nearly gray relative of the source and will usually become the neutral.

Two inputs, three generated:
Midpoint hue: the average of the two hues, taken the short way around the wheel so hues near 350 and 10 degrees average to near 0.
Generated color A: the midpoint hue, with the average saturation and average lightness of the two inputs. It blends the pair.
Generated color B: midpoint hue plus 90 degrees, saturation 80 percent of the average (minimum 25), and lightness 50.
Generated color C: midpoint hue plus 180 degrees, saturation 65 percent of the average (minimum 18), and lightness 48. This is a muted complement.

Three inputs, one generated:
Sort the three hues around the wheel and measure the gap between each neighbor, including the wrap from the last back to the first. Find the largest gap and place the new color in the middle of it. Saturation is the average saturation of the inputs, clamped between 25 and 85. Lightness is 50. This fills the biggest hole in the palette's hue coverage.

Four inputs, one generated:
A near-neutral. Hue is the average hue of the four inputs, saturation is fixed at 8, and lightness is the average lightness clamped between 42 and 65. This gives the palette a quiet supporting color that fits the others.

Five inputs, none generated.

Generated colors are converted back to hex and then run through the same analysis as source colors, so they take part in role sorting on equal terms.

Color distance

To compare two colors, the system blends three differences, each normalized to a 0 to 1 range:

Hue difference, measured the short way around the wheel and divided by 180, weighted 55 percent.
Saturation difference, divided by 100, weighted 25 percent.
Lightness difference, divided by 100, weighted 20 percent.

A smaller number means the colors look more alike. Hue counts most because it is the most noticeable difference.

Sorting the five colors into roles

The five roles are primary, secondary, tertiary, accent, and neutral. Colors are picked one at a time, and each pick leaves the pool:

1. Neutral: the color with the lowest chroma, meaning the grayest one.
2. Primary: the color with the highest primary score, which is its chroma minus 0.12 times its lightness distance from 50. This favors strong colors that are also mid-lightness and avoids picking something washed out or nearly black.
3. Accent: of the remaining three, the one with the highest chroma. It is the most vivid leftover color.
4. Secondary and Tertiary: The last two colors are ranked by color distance from the primary. The closer one becomes secondary and the farther one becomes tertiary. This makes Secondary a supporting relative of the Primary and Tertiary the more distinct additional color.

The order of the picks matters. Because Neutral is removed first, the Primary can never be the grayest color. Because accent is chosen before secondary and tertiary, accent means the most vivid leftover, not the one contrasting most with primary. Roles are assigned purely by these scores, so a generated color can end up as primary, and a source color can end up as neutral.

Final role order for display: Primary, Secondary, Tertiary, Accent, Neutral.

Shades for each role

Every role gets four tones, all keeping that color's original hue:

Shade 1: very light. Lightness is fixed at 94. Saturation is 35 percent of the original, with a floor of 4.
Shade 2: light. Lightness is fixed at 78. Saturation is 65 percent of the original, with a floor of 6.
Base: the exact source or generated color, unchanged.
Shade 3: deep. Lightness is fixed at 24. Saturation is 85 percent of the original, capped at 100.

Reducing saturation on the light shades keeps them from looking like neon pastels. The dark shade keeps most of its saturation so it stays rich instead of muddy.

Text color on a swatch

For legible labels on any color, if its perceived brightness is above 0.62, use near-black text (111111). Otherwise, use white text (FFFFFF).

Known limitations

HSL is not perceptually even. Saturated yellows look far brighter than saturated blues at the same lightness, so chroma is only a rough guide. A more accurate system would do the same steps in a perceptual space such as OKLCH.

In the four-input case, the average hue is a plain average and does not account for wrapping around the wheel, so hues near 350 and 10 degrees would average to about 180 instead of near 0. The two-input case handles wraparound correctly.

Shades use fixed lightness values, so a very light base color (above roughly 80) will sit out of order relative to Shade 2. The fix is to place the base by its actual lightness or to make the light shades relative to the base.

The zero-input re-roll condition is an addition to the original behavior, which had no random path at all. Without it, a random gray or near-black start would make the generated palette flat.
