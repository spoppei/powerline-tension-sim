# Powerline Tension Model — PIERS0028

Interactive simulation of static catenary tension on a simulated powerline used for ice and wet snow accretion research.

**Site:** UConn / GAIL — PIERS0028 station  
**Project:** NASA GPM Ground Validation  
**Wire:** 556.5-20/7 Type 16 ACSR/TW/GA2 Dove  

---

## What this is

The PIERS0028 station uses a dynamometer to measure tension on a simulated powerline. The goal is to detect ice and wet snow accreting on the wire — when ice builds up, the wire gets heavier and tension increases. When it sheds, tension drops suddenly.

The problem is that the dynamometer does not measure *true* tension. Every time the battery is replaced, the instrument resets to a new arbitrary zero. So the raw reading is:

```
dynamometer_reading = true_tension + bias
```

where `bias` is a different unknown constant after each battery reset.

This tool computes the **true physical tension** from the wire's geometry using catenary mechanics. Once you know the true tension, you can figure out the bias and eventually separate the accretion signal from other effects in the data (thermal expansion, diurnal cycles, etc.).

---

## What it does

- Takes three inputs: **span** (distance between poles), **sag** (how much the wire droops at the midpoint), and **wire weight** (lbs per foot)
- Computes the true tension at the support point using the catenary equations
- Draws the wire profile so you can visually check that the geometry makes sense
- Sliders and number inputs stay in sync — you can drag or type

---

## Inputs

| Parameter | Description | Unit | Default |
|---|---|---|---|
| Span | Horizontal distance between attachment points | ft | 164 ft (~50 m estimate) |
| Midspan sag | Vertical drop from chord to lowest point of wire | ft | 1.0 ft (~1 ft observed) |
| Wire weight | Linear weight of wire including LLDPE cover | lb/ft | 0.833 |

> **Note:** Wire weight is pre-filled from the manufacturer spec sheet (0.764 lb/ft bare + ~0.069 lb/ft LLDPE cover). Only change it if the cover has been removed or a different wire is used.

---

## Pseudocode

```
# 1. Get inputs
L = span in feet
d = sag in feet
w = wire weight in lbs/ft

# 2. Compute horizontal tension
#    This is the sideways pull the wire puts on each attachment point.
#    A tighter wire (less sag) requires much more horizontal force to hold its shape.
H = (w * L^2) / (8 * d)

# 3. Compute vertical component
#    At the support, the wire also pulls upward to hold up the weight
#    of the wire hanging on each side (half the span worth of wire).
V = w * (L / 2)

# 4. Compute true tension at the support point
#    The support sees both H and V at the same time.
#    Combined using Pythagorean theorem (they act at right angles).
T = sqrt(H^2 + V^2)

# 5. Output T — this is the true tension the dynamometer should read
#    in a perfectly unbiased instrument.
```

---

## Wire specification

| Property | Value |
|---|---|
| Type | ACSR/TW/GA2 |
| Name | Dove 556.5 |
| Stranding | 20/7 (20 Al wires, 7 steel core) |
| Bare outside diameter | 0.850 in |
| LLDPE cover thickness | ~0.060 in (verify vs spool documentation) |
| Bare linear weight | 0.764 lb/ft |
| Covered linear weight | ~0.833 lb/ft |
| Rated breaking strength | 22,600 lbs |

Source: Nassau National Cable / Southwire manufacturer data

---

## How to measure span and sag on site

**Span:** Measure the horizontal distance between the two wire attachment points (clamp to clamp). A tape measure along the ground works for a level span. A laser rangefinder is faster.

**Sag:** Stretch a string or laser level between the two attachment points. At the midpoint of the span, measure the vertical distance from the string down to the wire. That distance is the sag.

---

## Next steps

- [ ] Plug in real span and sag measurements from PIERS0028
- [ ] Add bias correction section (dynamometer reading → bias → corrected signal)
- [ ] Add ice accretion loading model (tension increase as a function of ice thickness)
- [ ] Connect to dynamometer time series for live decomposition

---

## Background reading

- Catenary mechanics: the shape a hanging wire makes under its own weight
- STL decomposition: used later to separate the accretion signal from thermal/diurnal cycles
- NASA GPM GV network: [gpm.nasa.gov](https://gpm.nasa.gov/data/ground-validation)
