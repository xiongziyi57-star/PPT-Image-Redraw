# Arrows And Icons Checklist

Use this reference when redrawing PPT screenshots that contain flow arrows or semantic icons.

## Arrow Direction

Before building the PPTX, create an arrow inventory:

```json
{
  "id": "arrow_step_1_to_2",
  "from": "step_1_card",
  "to": "step_2_card",
  "direction": "right",
  "head_at": "step_2_card",
  "bbox": [115, 397, 132, 407],
  "style": "solid",
  "purpose": "flow"
}
```

Rules:

- Determine direction from the reference image, not from assumed reading order.
- For left-pointing arrows, use an explicit left arrow or horizontal flip with a recorded reason.
- For right-pointing arrows, ensure the arrowhead is on the right edge.
- Distinguish arrows from separators. A vertical dashed divider is not an arrow.
- Re-check direction after any coordinate scaling, mirroring, or reuse of a helper function.

Validation ideas:

- Count arrows by direction and compare with the inventory.
- Inspect shape preset names such as `rightArrow` / `leftArrow`.
- If using line arrows, check begin/end arrowhead attributes.
- For each arrow, verify its x/y coordinates place the head nearer the intended target.

## Icon Assets

Treat semantic icons as one object:

- Target / bullseye icon
- Diamond / relationship icon
- Shield / separation or protection icon
- Eye / visibility or stealth icon
- Badge, seal, device, UI symbol, or any decorative pictogram

Allowed final representations:

- User-provided icon asset inserted once
- Licensed or built-in PowerPoint icon inserted once
- Generated transparent PNG inserted once with a source record
- Simple SVG/EMF inserted once when the user accepts that PowerPoint may not expose all internals as separate editable geometry

Avoid:

- Multiple circles, lines, squares, and slashes layered together as the final icon
- Raw crops from the reference image with surrounding card borders or text fragments
- Programmatic shape stacks presented as production-grade icon assets

If a fully editable shape-stack version is useful, label it as `editable_approximation` and still keep a single-object icon version for the production slide.

## Inventory Fields

For each icon, record:

```json
{
  "semantic_id": "expected_target_confidence_icon",
  "meaning": "降低目标类别过度置信",
  "bbox": [34, 490, 63, 521],
  "asset_source": "provided_asset | built_in_icon | imagegen_asset | licensed_asset | editable_approximation",
  "inserted_as_single_object": true,
  "notes": "red line target icon"
}
```

## Acceptance Checks

- Every expected icon has exactly one final inserted object or a documented exception.
- The icon can be moved/replaced as a unit.
- No icon contains stray text, card border fragments, or cropped background.
- The final report lists icon source type and any approximation/downgrade.
