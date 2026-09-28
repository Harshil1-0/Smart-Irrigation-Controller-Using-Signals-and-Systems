# Smart Irrigation Controller Using Signals and Systems

A discrete-time controller for smart irrigation, designed and analysed with core Signals and Systems tools: difference equations, convolution, the Z-transform, pole-zero analysis and frequency response. Soil moisture and temperature are modelled as sampled signals and fed into a first-order IIR system that produces a smooth irrigation command.

**Course:** Signals and Systems (S&S), Department of Mathematics, Mahindra University
**Instructor:** Prof. Rakesh Kumar
**Academic year:** 2025–2026

## Team

| Member | Roll No. |
|--------|----------|
| Sohum Sharma | SE24UCAM063 |
| Hasini Reddy | SE24UCAM019 |
| Amaan Rahman | SE24UCAM059 |
| Pearl Mendapara | SE24UCAM043 |
| Abhipsha Pujahari | SE24UCAM058 |
| Shivam Sharma | SE23UCAM022 |
| Harshil Pansala | SE24UCAM051 |

## Motivation

Agriculture uses roughly 70% of global freshwater, and fixed-schedule timers waste water when soil is already wet and stress crops during unexpected dry spells. Sensor-driven control can adapt to actual conditions, and because sensor readings are sampled over time, signal processing theory applies directly.

## System Model

**Inputs** (sampled hourly, n = 0…23, period 24 h):

| Signal | Model | Range |
|--------|-------|-------|
| Soil moisture `x_m[n]` | `60 − 10·sin(πn/12)` | 50% – 70% |
| Temperature `x_t[n]` | `25 + 5·sin(πn/12)` | 20 °C – 30 °C |

**Controller** (causal, time-invariant, multi-input single-output, first-order IIR):

```
y[n] = 0.7·y[n−1] + 0.15·x_m[n] + 0.05·x_m[n−1] + 0.05·x_t[n] + 0.02·x_t[n−1]
```

| Term | Coefficient | Role |
|------|-------------|------|
| `y[n−1]` | 0.7 | Feedback for smoothing and memory |
| `x_m[n]`, `x_m[n−1]` | 0.15, 0.05 | Primary (moisture) path |
| `x_t[n]`, `x_t[n−1]` | 0.05, 0.02 | Secondary (temperature) path |

Total moisture weight is 0.20 versus 0.07 for temperature, about a 3:1 ratio.

## Analysis Summary

| Topic | Result |
|-------|--------|
| Transfer functions | `H_m(z) = (0.15 + 0.05z⁻¹) / (1 − 0.7z⁻¹)`, `H_t(z) = (0.05 + 0.02z⁻¹) / (1 − 0.7z⁻¹)` |
| Pole | `z = 0.7` (shared by both paths), inside the unit circle |
| Zeros | `z = −0.333` (moisture), `z = −0.4` (temperature) |
| ROC | `\|z\| > 0.7`, which includes the unit circle |
| Stability | BIBO stable |
| Phase type | Minimum-phase (all poles and zeros inside the unit circle) |
| DC gain | `H_m(1) ≈ 0.667`, `H_t(1) ≈ 0.233` (ratio ≈ 2.86) |
| Frequency behaviour | Low-pass: passes slow daily trends, attenuates high-frequency sensor noise |

The impulse responses decay geometrically with the factor `(0.7)^n`, so recent inputs matter more than old ones.

## Simulation Results

24-hour run with initial conditions `y[−1] = 0`, `x_m[−1] = x_m[0]`, `x_t[−1] = x_t[0]`:

- Output stays bounded between **13.75** and **50.22**
- No oscillation or abrupt jumps; the output is smoother than the raw inputs
- Sample values: `y[0] = 13.75`, `y[6] = 38.07`, `y[12] = 42.96`, `y[20] = 50.22` (peak), `y[23] = 49.10`

The report includes the full hourly table, plots of both inputs, the output, a moisture-vs-output comparison and a 3D output plot.

## Repository Contents

```
.
├── README.md
├── Report.pdf                 # Full report (44 pages): theory, derivations, simulation, appendices
└── Smart-Irrigation-Controller-Using-Signals-and-Systems.pptx   # Presentation slides
```

Report chapters: Introduction, Problem Formulation, Input Signal Modelling, System Design, Convolution Analysis, Z-Transform and Transfer Functions, Pole-Zero Analysis and Stability, Frequency Response, Complete System Simulation, Applications and Future Scope, Conclusion. Appendices cover the MATLAB code, detailed calculations, contributions, glossary and notation.

## Running the Simulation (MATLAB)

Requires MATLAB with the Signal Processing Toolbox (for `freqz`). The full code is in Appendix A of the report. The core simulation is:

```matlab
n  = 0:23;
xm = 60 - 10*sin(pi*n/12);   % soil moisture
xt = 25 +  5*sin(pi*n/12);   % temperature

a1 = 0.7; b0 = 0.15; b1 = 0.05; c0 = 0.05; c1 = 0.02;

y = zeros(1, length(n));
for i = 1:length(n)
    if i == 1
        xm_prev = xm(1); xt_prev = xt(1); y_prev = 0;
    else
        xm_prev = xm(i-1); xt_prev = xt(i-1); y_prev = y(i-1);
    end
    y(i) = a1*y_prev + b0*xm(i) + b1*xm_prev + c0*xt(i) + c1*xt_prev;
end

stem(n, y, 'filled'); title('System Output y[n]');
xlabel('n (time index)'); ylabel('Output (control signal)'); grid on;
```

Frequency response:

```matlab
[H_m, w] = freqz([0.15 0.05], [1 -0.7], 1024);
[H_t, ~] = freqz([0.05 0.02], [1 -0.7], w);
plot(w/pi, abs(H_m), w/pi, abs(H_t));
legend('H_m', 'H_t'); xlabel('Normalized frequency (\times\pi rad/sample)');
```

## Applications

Smart drip irrigation, precision agriculture, greenhouse climate control, urban and vertical farming, hydroponics, residential smart gardens, large-scale farm automation, water conservation systems, sensor-based IoT farming, and academic simulation.

## Future Scope

Higher-order or adaptive controllers, additional sensors, real hardware and IoT deployment, and testing against noisy real-world sensor data (see Section 10.2 of the report).

## References

1. Adams, M. B. (2016). *Signals and Systems.* Oxford University Press.
2. Franklin, G. F., Powell, J. D., & Workman, M. L. (1998). *Digital Control of Dynamic Systems* (3rd ed.). Addison-Wesley.
3. FAO (2023). *Modern Irrigation Technologies and Water Optimization Techniques.*
4. USDA Agricultural Research Service (2024). *Soil Moisture and Temperature Monitoring Methods.*
5. MathWorks Documentation. *Signal Processing Toolbox User's Guide.*
6. MIT OpenCourseWare. *6.003 Signals and Systems.*
