[🇮🇹 Leggi in Italiano](README-it.md)

> **Note:** The application interface is currently available in Italian only, though the data consists mostly of universally understandable numbers and metrics.

# 🏃‍♂️ GPX Runner Analytics & Strava Race Predictor

A **100% client-side** web application (zero backend, zero tracking) to analyze running GPS tracks (`.gpx`) and predict race times for **5k, 10k, Half Marathon, Marathon**, and custom distances starting from a **Strava** export archive (`.zip` or `.tcx`/`.gpx` files)[cite: 6].

🌐 **Try it live:** [romeo0212.github.io/runner-analytics](https://romeo0212.github.io/runner-analytics/)[cite: 6]

> ⚡ **Project Philosophy (Single-File & Vibe Coded):**  
> Besides using the app directly via the link above, the entire application is contained within a **single `index.html` file** (which integrates HTML, Tailwind CSS, and JavaScript). This architectural choice guarantees **maximum portability**: anyone can download the file and run it offline in their browser, making it incredibly easy to inspect, customize, or freely expand the code without setting up any development environment. The tool was born through *vibe coding*, iterating the code in close contact with real sports physiology metrics[cite: 6].

---

## ✨ Main Features[cite: 6]

### 1. 📊 Single GPX Analysis (`Mode 1`)[cite: 6]
* **Instant Summary & GPS Track:** Calculation of distance, total time, average pace, smoothed positive elevation gain ($D+$), and interactive map visualization via Leaflet/OpenStreetMap[cite: 6].
* **Start/End Trimming:** Interactive sliders to crop out the initial warmup, traffic light stops, or final cooldown walks by distance or time[cite: 6].
* **Regular Splits & Fartlek Builder:** Analysis by fixed distances (`1 km`, `500 m`, `400 m`, `1 min`, etc.) or custom mixed workouts (e.g., *2 km Warmup + 6×(1' Fast + 1' Recovery) + Cooldown*)[cite: 6].
* **Internal Variability Index ($\pm\%$):** Measures pace consistency *within* each individual block, distinguishing a steady-paced interval from an uneven, surging run[cite: 6].

### 2. 🏆 Strava Archive Race Projections (`Mode 2`)[cite: 6]
* **Direct Strava `.zip` Export Reading:** In-memory browser extraction using `JSZip` and `pako` (automatic decompression of `activities.csv` and `.tcx.gz` / `.gpx.gz` tracks)[cite: 6].
* **Multi-Sport & Cross-Training Separation:** Automatically discards other sports from the race pace calculation, but credits the aerobic volume of **Soccer / Cross-Training (weighted at 65%)** and short warmups ($< 2\text{ km}$) when calculating muscular endurance for long distances[cite: 6].
* **Cyclic Fartlek & Continuous Segment Detector:** Scans tracks second-by-second to extract both the best continuous segments ($\ge 1\text{ km}$) and short interval workouts (via the median of distinct repetitions), preventing warmups or slow recoveries from lowering the estimated aerobic potential[cite: 6].

---

## 🧮 Mathematical and Physiological Model[cite: 6]

The predictive engine combines four classic endurance literature models to avoid both overly pessimistic estimates (caused by averaging with easy runs) and false positives (caused by isolated GPS spikes)[cite: 6].

### 1. Elevation Smoothing and GAP (Grade Adjusted Pace)[cite: 6]
Raw GPS elevation is filtered with a 7-point moving average and a $3.5\text{ m}$ hysteresis threshold to eliminate satellite noise on flat terrain[cite: 6]. The pace is then converted into **GAP** (Grade Adjusted Pace) according to Minetti's metabolic model (~$3.3\text{ s/km}$ per $10\text{ m}$ of $D+$ per km)[cite: 6]:

$$\text{GAP}_{\text{sec/km}} = \text{Raw\_Pace} - \left( \frac{\text{Total\_}D+}{\text{Distance}_{\text{km}} \cdot 10} \right) \times 3.3$$

### 2. Aerobic Engine Estimation ($\text{VDOT}$ by Daniels & Gilbert)[cite: 6]
For every performance of distance $D$ (in meters) and equivalent flat time $T$ (in minutes), with velocity $v = D / T$ (in $\text{m/min}$), the theoretical oxygen consumption ($\text{VO}_2$) and sustainable fraction ($\text{\%VO}_2\max$) are calculated using the Daniels & Gilbert equations[cite: 6]:

$$\text{VO}_2(v) = -4.60 + 0.182258 \cdot v + 0.000104 \cdot v^2$$

$$\%\text{VO}_2\max(T) = 0.8 + 0.1894393 \cdot e^{-0.012778 \cdot T} + 0.2989558 \cdot e^{-0.1932605 \cdot T}$$

$$\text{VDOT} = \frac{\text{VO}_2(v)}{\%\text{VO}_2\max(T)}$$

*(If average heart rate data $\text{HR}_{\text{avg}}$ compared to $\text{HR}_{\max}$ is present, sub-maximal efforts run between $65\%$ and $91\%$ of $\text{HR}_{\max}$ receive a proportional credit based on heart rate reserve)*[cite: 6].

### 3. Cyclic Fartlek Detection (Repetition Median) and Consistency Bonus[cite: 6]
To prevent interval workouts from being penalized by slow recovery phases[cite: 6]:
* The track is split into consecutive $30\text{ s}$ windows based on continuous geodetic distance (Haversine formula)[cite: 6].
* A session is recognized as a **Fartlek** only if it features $\ge 3$ distinct push/recovery cycles where fast blocks are at least $12\%$ faster than the run's median and adjacent to recovery blocks at least $30\%$ slower[cite: 6].
* The repetition pace is extracted via the **median** of the fast blocks (robust to GPS outliers) and converted into an equivalent $10\text{ km}$ pace[cite: 6].
* The final $\text{VDOT}_{\text{eff}}$ index is anchored to the **Certified Base Peak** (with time decay after 21 days), boosted by a **Consistency Bonus** (up to $+1.5\%$) derived from the best quality workouts and weekly volume[cite: 6].

### 4. Distance Projection with Dynamic Riegel Exponent[cite: 6]
From the base $10\text{ km}$ time ($T_{10\text{k}}$, obtained by numerically reversing the $\text{VDOT}$ function via bisection), the time $T_2$ for a target distance $D_2$ is calculated using Peter Riegel's power law[cite: 6]:

$$T_2 = T_{10\text{k}} \cdot \left( \frac{D_2}{10000} \right)^k$$

where the **fatigue exponent $k$** is not fixed at the standard $1.06$ value (which only applies to elite, high-mileage athletes), but dynamically adapts based on the **Equivalent Weekly Volume** ($V_{\text{eq}} = \text{run}_{\text{km}} + 0.65 \cdot \text{soccer/cross}_{\text{km}}$)[cite: 6]:

$$k = \text{clamp}\Big( 1.06 + \max\big(0,\; (45 - V_{\text{eq}}) \cdot 0.0005\big),\; 1.055,\; 1.090 \Big)$$

For distances over $15\text{ km}$ (Half Marathon and Marathon), if the recent longest run ($L_{\max}$) is less than $68\%$ of the race distance, an additional muscular endurance prudential coefficient is applied[cite: 6].

---

## 🔒 Privacy[cite: 6]

The application does not use any backend server: the entire parsing of `.zip`, `.csv`, `.gpx`, and `.tcx` files happens locally in the user's browser. No files are uploaded to the internet[cite: 6].