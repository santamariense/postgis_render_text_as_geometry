# PostGIS: Render text as geometry

This PostgreSQL/PostGIS function renders text along a `LINESTRING` geometry using pre-defined vector font glyphs. It scales, positions, and rotates the text to follow the shape and direction of the line geometry.

> Currently works **only for `LINESTRING` geometries**. Support for other geometry types is yet to be implemented.

---

## Requirements

This function depends on:

- **PostgreSQL**
- **PostGIS extension** (must be enabled in your database)

---

## Installation

### Prerequisites

1. **Ensure that PostGIS is installed** in your target database.

4. **Install the function**
 Run the SQL file "render_text_as_geometry.sql" in the database you want to use it.
 ```

---

## Function Signature

```sql
render_text_as_geometry(
input_text TEXT,
scale_factor DOUBLE PRECISION DEFAULT 1.0,
font_type TEXT DEFAULT 'LINES',
base_geom GEOMETRY DEFAULT NULL,
text_position INTEGER DEFAULT NULL,
max_height DOUBLE PRECISION DEFAULT NULL
) RETURNS GEOMETRY
```

### Parameters

| Name | Type| Description |
|----------------|-------------------|-------------|
| `input_text` | `TEXT`| Text to render |
| `scale_factor` | `DOUBLE PRECISION`| Optional scaling multiplier for width |
| `font_type`| `TEXT`| Currently unused, reserved for future use |
| `base_geom`| `GEOMETRY`| Line geometry along which the text will be rendered |
| `text_position`| `INTEGER` | Optional vertical offset in meters |
| `max_height` | `DOUBLE PRECISION`| Optional maximum height (in meters) for the rendered text |

---

## What It Does

1. **Preprocess Input Text**
 Splits text into characters and inserts invisible separator characters (`chr(0x2063)`) to create spacing.

2. **Load Glyphs**
 Loads the vector geometries for each character.

3. **Calculate Scaling**
 Computes scaling factors to fit text to the length of `base_geom` and/or to obey `max_height`.

4. **Place and Combine Glyphs**
 Applies offsets, scales, and combines all characters into a single `GEOMETRY`.

5. **Align Along Line**
 Centers the rendered text along the `LINESTRING` and rotates it to align with the geometry direction.

6. **Vertical Positioning**
 Applies vertical offset (`text_position`) if provided, using metric SRIDs for consistent scaling.

7. **Return Geometry**
 Outputs the final geometry, aligned and scaled, in the same SRID as `base_geom`.

---

## Notes

- If `input_text` contains only unsupported characters, or `base_geom` is `NULL`, the function returns `NULL`.
- The font table must be populated with appropriate vector representations for each character to render.
- The function assumes the SRID of `base_geom` is set correctly for rendering scale.
- Alignment rotation logic adjusts orientation to follow the `LINESTRING`.

---

## Example Usage

```sql
SELECT render_text_as_geometry(
'HELLO WORLD',
1.0,
'LINES',
ST_GeomFromText('LINESTRING(0 0, 100 0)', 4326),
5, -- vertical offset
20 -- max height
);
```

---

## TODO

- Support for other geometry types (`POLYGON`, `POINT`, etc.)
- Multi-line or wrapped text rendering
- Dynamic font loading from external sources



