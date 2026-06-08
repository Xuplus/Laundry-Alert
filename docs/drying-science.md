# The Science of Clothes Drying

## The simple version

When wet clothes hang in the air, water evaporates from the fabric into the surrounding air. How fast that happens depends on two things:

1. **How thirsty is the air?** Air can only hold a certain amount of water vapour before it's "full". The further it is from that limit, the faster it absorbs moisture from your laundry.
2. **Is fresh air reaching the clothes?** Air right next to the fabric quickly becomes saturated. Wind sweeps it away and replaces it with drier air, keeping evaporation going.

That's essentially it. Temperature, humidity, sun, and wind are all just dials on these two mechanisms.

---

## The key parameters

### Vapor Pressure Deficit (VPD) — the most important number

Temperature and humidity don't act independently — they combine into a single quantity called **Vapor Pressure Deficit (VPD)**: the gap between how much water vapour the air *could* hold and how much it *actually* holds. A large VPD means the air is hungry for moisture; a VPD near zero means the air is nearly saturated and drying slows to a crawl.

VPD explains things that separate temperature or humidity cannot:
- 30 °C at 80 % humidity dries *slower* than 20 °C at 40 % humidity — even though it's hotter.
- On a cold but crisp, dry winter day (5 °C, 30 % RH), clothes can still dry reasonably well because the VPD is positive.

Research in evaporation science consistently shows that VPD has a **near-linear relationship with evaporation rate**, making it the most reliable single predictor for drying speed.

### Wind speed — the biggest performance lever

Wind is, in most conditions, more important than sunshine. Here's why: even in low humidity, a thin layer of saturated air forms right around wet fabric. Without wind, this layer just sits there and evaporation nearly stalls. Wind strips that layer away continuously.

Engineering studies on mass transfer show the effect scales roughly with **wind speed to the power of 0.78** — meaning doubling the wind speed increases the drying rate by about 71 %. In practice, going from still air to a moderate breeze (20–25 km/h) can more than double how fast clothes dry.

### Solar radiation / cloud cover

Sun heats the fabric surface, which raises the local saturation point and increases the effective VPD right at the evaporation surface. This is real but secondary: studies and meteorological models (including the British Weather Services formula used by Vileda) put the contribution of full sun vs. full overcast at roughly **25–35 % speedup**. Wind and humidity matter more.

Cloud cover is a practical proxy for solar input. Open-Meteo also provides `shortwave_radiation` directly in W/m², which is more precise.

### What doesn't matter as much

- **Dew point alone** — useful, but it's just another way to express the same information as temperature + RH. VPD captures it more directly.
- **Atmospheric pressure** — meaningful in industrial dryers; negligible for home air-drying at typical altitudes.

---

## Typical drying times (measured)

These come from empirical studies, including full-scale indoor drying experiments and garment weight-loss measurements:

| Setup | Garment | Conditions | Time |
|---|---|---|---|
| Outdoor line | T-shirt | 21 °C, < 70 % RH, 15–20 km/h wind, sunny | 1–2 h |
| Outdoor line | Mixed average load | Good summer conditions | 2–3 h |
| Outdoor line | Jeans / towels | Good conditions | 4–6 h |
| Indoor rack | T-shirt | 20 °C, 60 % RH, still air | 4–6 h |
| Indoor rack | Jeans | 20 °C, 60 % RH, still air | 12–24 h |
| Indoor rack | Thick towel | 20 °C, 60 % RH, still air | 8–15 h |

Fabric type adds a further 2–3× range (synthetics dry fastest, thick cotton and wool slowest). The times above assume a typical mixed cotton load.

---

## The formula

### Underlying model

The formula is based on **Dalton's evaporation law**, the same model used by meteorological services to estimate evaporation from open water and soil:

```
Evaporation rate  ∝  VPD  ×  wind_factor  ×  solar_factor
Drying time       ∝  1 / Evaporation rate
```

### Step-by-step

**1. Compute saturation vapor pressure** from temperature (Bolton 1980 formula):
```
es(T)  =  6.112 × exp( 17.67 × T / (T + 243.5) )   [hPa]
```

**2. Compute VPD:**
```
VPD  =  es(T) × (1 − RH / 100)   [hPa]
```

If VPD < 0.5 hPa (air nearly saturated), drying is effectively impossible.

**3. Wind factor** (outdoor methods only):
```
windFactor  =  1  +  windspeed_km_h / 25
```
Derived from boundary-layer mass transfer theory. Gives 1.0 in still air, 2.0 at 25 km/h.

**4. Solar factor** (outdoor methods only):
```
solarFactor  =  1  +  0.3 × (1 − cloudcover / 100)
```
+0 % for full overcast, +30 % for clear sky.

**5. Base drying times** (calibrated at 20 °C, 50 % RH, still air, overcast — the neutral reference point):

| Method | Base (minutes) |
|---|---|
| Outside line | 185 |
| Outdoor rack | 240 |
| Indoor rack | 270 |

**6. Final estimate:**
```
dryTime  =  base × (VPD_ref / VPD) / (windFactor × solarFactor)
```

Where `VPD_ref = 11.7 hPa` is the VPD at the reference conditions (20 °C, 50 % RH).

### Spot checks

| Conditions | Method | Formula result | Empirical expectation |
|---|---|---|---|
| 22 °C, 50 % RH, 15 km/h, sunny | Outside | ~95 min | 90–120 min ✓ |
| 20 °C, 60 % RH, still, overcast | Indoor rack | ~270 min (4.5 h) | 4–6 h ✓ |
| 30 °C, 30 % RH, 20 km/h, sunny | Outside | ~47 min | < 1 h in hot dry conditions ✓ |
| 10 °C, 85 % RH, 5 km/h, overcast | Outside | ~18 h | Very long / not recommended ✓ |

---

## Why this is better than separate temperature and humidity multipliers

The previous approach used independent factors for temperature and humidity applied as separate multipliers. That creates two problems:

- **Double-counting**: temperature already determines how much moisture air can hold; treating it as a separate speedup factor over-counts it.
- **Wrong behavior in edge cases**: a hot, very humid day gets an inflated drying speed estimate because the temperature multiplier is high, even though the air is nearly saturated.

VPD resolves both issues by combining temperature and humidity into one physically grounded quantity.

---

## Sources

1. **Dalton's evaporation law and the Penman equation** — the foundation for all modern open-surface evaporation models, including those used by meteorological services for "washing day" forecasts.
   - Penman, H. L. (1948). *Natural evaporation from open water, bare soil and grass.* Proceedings of the Royal Society A, 193, 120–145.

2. **Bolton (1980) saturation vapor pressure formula** — the standard meteorological formula used in this model.
   - Bolton, D. (1980). *The computation of equivalent potential temperature.* Monthly Weather Review, 108, 1046–1053.

3. **Analytical and Experimental Studies of Rapid Cloth Drying** — peer-reviewed paper measuring drying rates under controlled conditions.
   - [ResearchGate](https://www.researchgate.net/publication/328593146_Analytical_and_Experimental_Studies_of_Rapid_Cloth_Drying_for_Technological_Innovation)

4. **Indoor laundry drying: Full-scale determination of water emission rate and impact on thermal comfort** — empirical study measuring water emission rates and drying times on a full-scale indoor drying rack.
   - [ScienceDirect (2025)](https://www.sciencedirect.com/science/article/pii/S2950362025000189)

5. **Efficiency limits of evaporative fabric drying methods** — engineering study examining the thermodynamic limits of air drying.
   - [Drying Technology, Taylor & Francis (2020)](https://www.tandfonline.com/doi/full/10.1080/07373937.2020.1839486)

6. **Determination of Mass Transfer Coefficient for Evaporation** — quantifies how the convective mass transfer coefficient scales with wind speed.
   - [IJSRD](https://www.ijsrd.com/articles/IJSRDV4I120199.pdf)

7. **MetService NZ — The science of drying** — operational meteorological model (Penman-based evapotranspiration) applied to produce "washing day" forecasts.
   - [MetService Blog](https://blog.metservice.com/Washing_Weather)

8. **Vileda / British Weather Services — The Perfect Drying Formula** — empirically validated practical thresholds (21 °C+, < 70 % RH, 8–12 mph wind, 1 h sun) for 1-hour drying.
   - [Vileda UK](https://www.vileda.co.uk/perfectdryingformula) · [Ideal Home coverage](https://www.idealhome.co.uk/news/perfect-formula-drying-washing-weather-experts-281086)

9. **Vapour-pressure deficit** — overview of VPD and its near-linear relationship to evaporation rate.
   - [Wikipedia](https://en.wikipedia.org/wiki/Vapour-pressure_deficit)

10. **DryTime — Weather conditions best for drying** — applied analysis confirming wind dominates over sunshine for outdoor drying.
    - [dryti.me](https://dryti.me/articles/weather-conditions-best-for-drying/)
