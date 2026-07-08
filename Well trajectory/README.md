# Understanding the Volve Well Trajectory Notebook

## 1. Why does a well need a "trajectory"?

Most modern wells aren't straight vertical holes — they're steered off vertical, sometimes
turning almost 90°, to reach a reservoir that isn't directly under the rig, avoid obstacles, or
maximize contact with the reservoir rock (more contact area = more oil/gas can flow in, similar to
increasing surface area for mass transfer in a packed column). Because the path curves, engineers
need a precise 3D map of where the borehole actually goes — that map is the **well trajectory**.

## 2. The core measurements

Sensors near the drill bit periodically record three numbers at each survey "station" (every
~30–100 m). To reconstruct the full 3D path, we need these 3 values:

| Term | What it means |
|---|---|
| **MD** — Measured Depth | Length of pipe/cable run into the ground, measured along the curved path (not a straight-line distance) |
| **Inclination** | Angle from vertical, in degrees. 0° = straight down, 90° = horizontal |
| **Azimuth** | Compass direction the well is heading, in degrees (0°/360° = North, 90° = East) |

From MD + Inclination + Azimuth at each station, an industry-standard technique called the
**minimum curvature method** (fits a smooth arc between stations rather than a jagged straight
line) gives two more useful quantities:

- **TVD (True Vertical Depth)** — straight-down depth, ignoring sideways drift. This is what
  drives hydrostatic pressure calculations.
- **N/S and E/W displacement** — how far the well has moved north/south and east/west of its
  surface starting point.

## 3. Where the coordinates come from

To place 2 wells on the same map, their coordinates need to share a reference point. Here, that shared reference is a **UTM
grid** (Northing/Easting in meters) anchored to each well's **surface datum**. Mismatched datums
would make two wells look wrong relative to each other even if each one's own data is fine.

## 4. Why the raw data is messy

The source data is tables buried inside scanned PDF drilling reports, in Norwegian formatting:

- Comma as the decimal point, space as the thousands separator (`"1 364,50"` = 1364.50)
- Non-numeric text mixed into numeric columns — bit sizes (`12 1/4"`), coordinates in
  degrees-minutes-seconds (`58° 26' 29,706" N`)
- The extraction tool sometimes misreads the first row of numbers as a header row

This is the same "real-world data is messy" problem you'd encounter pulling data from any lab
instrument export or scanned datasheet; thus we have to "clean" it.

## 5. What this notebook try to accomplish

1. **Extract** the survey table from each PDF report with `tabula`.
2. **Clean** every cell: convert Norwegian number formatting to floats, drop non-numeric cells.
3. **Reconstruct** the true table, station by station, per well.
4. **Calculate** 3D position at each station via minimum curvature (`welly` handles the math).
5. **Plot** it three ways: a **3D view**, a **plan view** (top-down, shows sideways drift), and a
   **vertical section** (MD vs. TVD — the view drilling engineers check most often).

## 6. Why this matters beyond drilling

The trajectory feeds directly into classic engineering questions: how much wellbore is inside the
productive rock (→ surface area for mass transfer), what's the pressure drop to surface (→ needs
TVD, not MD), and how close is this well to another one (→ clearance, like piping design). Get the
geometry wrong, and everything built on it — pressure profiles, flow assurance, reserves — inherits
the error.

---

*This document accompanies `Volve_Well_Trajectory.ipynb`, which implements the steps above using
real EOWR data from the public Volve field dataset.*
The open-source data set can be found at [Equinor Volve field dataset](https://www.equinor.com/energy/volve-data-sharing)
