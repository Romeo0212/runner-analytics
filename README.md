[🇮🇹 Leggi in Italiano](README-it.md)

> **Note:** The application interface is currently available in Italian only, though the data consists mostly of universally understandable numbers and metrics.

# 🏃‍♂️ GPX Runner Analytics & Strava Race Predictor

A **100% client-side** web application (zero backend, zero tracking) to analyze running GPS tracks (`.gpx`) and predict race times for **5k, 10k, Half Marathon, Marathon**, and custom distances starting from a **Strava** export archive (`.zip` or `.tcx`/`.gpx` files).

🌐 **Try it live:** [romeo0212.github.io/runner-analytics](https://romeo0212.github.io/runner-analytics/)

> ⚡ **Project Philosophy:**  
> Besides using the app directly via the link above, the entire application is contained within a **single `index.html` file** (which integrates HTML, Tailwind CSS, and JavaScript). This architectural choice guarantees **maximum portability**: anyone can download the file and run it offline in their browser, making it incredibly easy to inspect, customize, or freely expand the code without setting up any development environment. The tool was born through *vibe coding*, iterating the code in close contact with real sports physiology metrics.

---

## ✨ Main Features

### 1. 📊 Single GPX Analysis (`Mode 1`)
* **Instant Summary & GPS Track:** Calculation of distance, total time, average pace, smoothed positive elevation gain ($D+$), and interactive map visualization via Leaflet/OpenStreetMap.
* **Start/End Trimming:** Interactive sliders to crop out the initial warmup, traffic light stops, or final cooldown walks by distance or time.
* **Regular Splits & Fartlek Builder:** Analysis by fixed distances (`1 km`, `500 m`, `400 m`, `1 min`, etc.) or custom mixed workouts (e.g., *2 km Warmup + 6×(1' Fast + 1' Recovery) + Cooldown*).
* **Internal Variability Index ($\pm\%$):** Measures pace consistency *within* each individual block, distinguishing a steady-paced interval from an uneven, surging run.

### 2. 🏆 Strava Archive Race Projections (`Mode 2`)
* **Direct Strava `.zip` Export Reading:** In-memory browser extraction using `JSZip` and `pako` (automatic decompression of `activities.csv` and `.tcx.gz` / `.gpx.gz` tracks).
* **Multi-Sport & Cross-Training Separation:** Automatically discards other sports from the race pace calculation, but credits the aerobic volume of **Soccer / Cross-Training (weighted at 65%)** and short warmups ($< 2\text{ km}$) when calculating muscular endurance for long distances.
* **Cyclic Fartlek & Continuous Segment Detector:** Scans tracks second-by-second to extract both the best continuous segments ($\ge 1\text{ km}$) and short interval workouts (via the median of distinct repetitions), preventing warmups or slow recoveries from lowering the estimated aerobic potential.

---

## 🧮 Mathematical and Physiological Model

The predictive engine combines four classic endurance literature models to avoid both overly pessimistic estimates (caused by averaging with easy runs) and false positives (caused by isolated GPS spikes).

### 1. Elevation Smoothing and GAP (Grade Adjusted Pace)
Raw GPS elevation is filtered with a 7-point moving average and a $3.5\text{ m}$ hysteresis threshold to eliminate satellite noise on flat terrain. The pace is then converted into **GAP** (Grade Adjusted Pace) according to Minetti's metabolic model (~$3.3\text{ s/km}$ per $10\text{ m}$ of $D+$ per km):

$$\text{GAP}_{\text{sec/km}} = \text{Raw Pace} - \left( \frac{\text{Total } D+}{\text{Distance}_{\text{km}} \cdot 10} \right) \times 3.3$$

### 2. Aerobic Engine Estimation ($\text{VDOT}$ by Daniels & Gilbert)
For every performance of distance $D$ (in meters) and equivalent flat time $T$ (in minutes), with velocity $v = D / T$ (in $\text{m/min}$), the theoretical oxygen consumption ($\text{VO}_2$) and sustainable fraction ($\text{\%VO}_2\max$) are calculated using the Daniels & Gilbert equations:

$$\text{VO}_2(v) = -4.60 + 0.182258 \cdot v + 0.000104 \cdot v^2$$

$$\%\text{VO}_2\max(T) = 0.8 + 0.1894393 \cdot e^{-0.012778 \cdot T} + 0.2989558 \cdot e^{-0.1932605 \cdot T}$$

$$\text{VDOT} = \frac{\text{VO}_2(v)}{\%\text{VO}_2\max(T)}$$

*(If average heart rate data $\text{HR}_{\text{avg}}$ compared to $\text{HR}_{\max}$ is present, sub-maximal efforts run between $65\%$ and $91\%$ of $\text{HR}_{\max}$ receive a proportional credit based on heart rate reserve).*

### 3. Cyclic Fartlek Detection (Repetition Median) and Consistency Bonus
To prevent interval workouts from being penalized by slow recovery phases:
* The track is split into consecutive $30\text{ s}$ windows based on continuous geodetic distance (Haversine formula).
* A session is recognized as a **Fartlek** only if it features $\ge 3$ distinct push/recovery cycles where fast blocks are at least $12\%$ faster than the run's median and adjacent to recovery blocks at least $30\%$ slower.
* The repetition pace is extracted via the **median** of the fast blocks (robust to GPS outliers) and converted into an equivalent $10\text{ km}$ pace.
* The final $\text{VDOT}_{\text{eff}}$ index is anchored to the **Certified Base Peak** (with time decay after 21 days), boosted by a **Consistency Bonus** (up to $+1.5\%$) derived from the best quality workouts and weekly volume.

### 4. Distance Projection with Dynamic Riegel Exponent
From the base $10\text{ km}$ time ($T_{10\text{k}}$, obtained by numerically reversing the $\text{VDOT}$ function via bisection), the time $T_2$ for a target distance $D_2$ is calculated using Peter Riegel's power law:

$$T_2 = T_{10\text{k}} \cdot \left( \frac{D_2}{10000} \right)^k$$

where the **fatigue exponent $k$** is not fixed at the standard $1.06$ value (which only applies to elite, high-mileage athletes), but dynamically adapts based on the **Equivalent Weekly Volume** ($V_{\text{eq}} = \text{run}_{\text{km}} + 0.65 \cdot \text{soccer/cross}_{\text{km}}$):

$$k = \text{clamp}\Big( 1.06 + \max\big(0,\; (45 - V_{\text{eq}}) \cdot 0.0005\big),\; 1.055,\; 1.090 \Big)$$

For distances over $15\text{ km}$ (Half Marathon and Marathon), if the recent longest run ($L_{\max}$) is less than $68\%$ of the race distance, an additional muscular endurance prudential coefficient is applied.

---

## 🔒 Privacy

The application does not use any backend server: the entire parsing of `.zip`, `.csv`, `.gpx`, and `.tcx` files happens locally in the user's browser. No files are uploaded to the internet.