# classe2-calculator

__classe2-calculator__ calculates [Class E Power Amplifier](https://en.wikipedia.org/wiki/Power_amplifier_classes#Class_E) circuit component values in order to assist with the practical design of Class E Power Amplifier based circuits for RF communication, plasma control, or any other application. It is pronounced "[classi](https://en.wiktionary.org/wiki/classy#Adjective)[*er*](https://en.wiktionary.org/wiki/%E4%BA%8C#Pronunciation)", as an English-Mandarin [*portmanteau*](https://en.wikipedia.org/wiki/Blend_word).

## Why?

The motivation for writing this software was increasing frustration attempting to understand and explore the solution space for the design of surface mount component based systems using existing tools, coupled with the intent to perform similar calculations in the future. What distinguishes it from other options?

 * Open source
   * Reliable tool for exploration of the solution space
   * Transparent algorithms
   * Well documented design
 * Portable (runs on all platforms)
 * SI / metric units (no cubit foot-yard waveform eighth-of-a-bodypart handspannery here)
 * SMD design as a primary use case, thus the following are in scope:
   * SRF
   * Current
   * Zener protection against voltage peaks
 * Produces explainable (well documented) designs which can be readily audited.

## Potential output / feature-set goals

 * __Primary schematic__
   * __Class E design__ ("infinite" `L1` inductor)
   * Variant class E designs ("finite" `L1` inductor, non-50% duty cycles)
   * Class E/F design (maybe later)
   * Class F design (maybe later)
 * __Antenna matching networks__
   * __L type__
   * __Pi type__
 * Antenna tuning unit (ATU) recommendations (maybe later)
 * SPICE simulation source (maybe later)
 * Coil core / winding instructions (maybe later)

## Class E Schematic

![Sokal Class Low Order Class E Power Amplifier Schematic](images/sokal-class-e.webp)

Taken from Sokal (1975).

 * `L1` = RF Choke inductor with high reactance at the fundamental frequency of operation.
 * `C1` = Shunt capacitor. Practically affected by switching MOSFET device output capacitance `C(OSS)`.
   * According to [classeradio](http://www.classeradio.com/rfvalues.htm): *The function of the shunt capacitance is to reduce the peak voltage appearing across the MOSFET when the device is in the off state, and to spread the width of the "off" pulse. The shunt capacitor is also part of the output matching network. This capacitor must be a high quality, high current component. Typically, multi-layer ceramic capacitors (MLCC) work best, however silver-mica capacitors can also be used in this function. The actual value of the shunt capacitor is important for a couple of reasons. First, if the capacitance is too small, you will see a very high RF peak voltage across your MOSFETs. If the value is too large, the efficiency can suffer. This is not an extremely critical value. [...] The shunt capacitor value is correct if the peak RF voltage across the MOSFETs during the "off" cycle is around 3.5 times the DC voltage applied to the stage. If the voltage is higher than this value, you may need to increase your shunt capacitor.*
 * `C2` / `L2` = Resonator
   * `C2` = "Tuning capacitor" (or "Series capacitor")
     * According to [classeradio](http://www.classeradio.com/rfvalues.htm): *The series tuning capacitor is subject to very high RF voltage [...] Using a lower inductance and higher capacitance in the resonant circuit will reduce the voltage across the tuning capacitor somewhat. A useful rule of thumb for figuring the tuning capacitor voltage rating is 3 to 5 times the peak to peak RF voltage fed to the resonant network plus a safety factor.*
 * `R` = Optimal load resistance
 * `Q` = Quality factor under load
   * Higher `Q` factor values mean higher peak voltages.
   * Certain bands require higher `Q` factors.
 * `VCC` = DC supply and nominal operating voltage
 * `Q1` = Active Device Switch (MOSFET)
  * According to [classeradio](http://www.classeradio.com/rfvalues.htm): *The peak voltage across the MOSFETs is going to be a little less than 4x the DC applied voltage for a proper class E transmitter. This will vary somewhat with tuning and your exact circuit. If you have very low, or NO shunt capacitor, the ratio of peak RF voltage to applied DC can be 6x or 8x the DC or MORE. For class E transmitters with proper shunt capacitors, figure 4x the DC, plus a safety factor.*
  * Key selection criteria include voltage (`VGS`; higher is better, >4x VCC and/or add low capacitance zener diodes which are typically those rated for higher voltages), capacitances (`C(RSS)`, `C(ISS)`, `C(OSS)`; lower is better), drain current (`I(D)`; higher is better).
 * `P` = target output power in watts

## Class E Design equations

The two most common types of Class E power amplifiers are classified based on the scale of `L1` DC feed inductance. When `L1` is large (to the point of approximating "infinite"), it is termed "infinite". Otherwise, it is termed "finite".

|                            | "Infinite"                         | "Finite"                         |
| -------------------------- | ---------------------------------- | -------------------------------- |
|               `L1` vs `L2` | 10-15× (or more)                   | <10×                             |
|                  Bandwidth | Broadband                          | Narrowband                       |
|                Selectivity | Poorly selective                   | Strongly selective               |
|       Harmonic suppression | Poor                               | Good                             |
|          Losses due to ESR | Higher                             | Lower                            |
|            Load resistance | Lower (harder matching network)    | Higher (easier matching network) | 
| Inter component dependence | Independent                        | Strongly interdependent          |
|             Ease of design | Relatively easy                    | Relatively difficult             |
|              Physical size | Large                              | Small                            |
|         Manufacturing cost | Expensive                          | Inexpensive                      |
|           Hand-wound coils | Suitable                           | Relatively unsuitable            |
|      Surface-mount devices | Relatively unsuitable              | Suitable                         |
|                 Popularity | Most amateur radio projects        | Relatively less popular          |
|        Load transformation | No                                 | Yes                              |

Generalities:
 * The sharper the resonance (higher `Q`), the sharper resonance peaks and greater sensitivity, and the more critical it is to have accurate `C1` and `C2`.
 * Some bands require higher `Q` than others
 * Most amateur radio projects seem to assume hand-wound coils which are relatively unsuitable for finite induction variants

### Design equations

Forenotes for non physics people because terminology can be confusing:
 * `f` is the frequency in Hz (cycles per second).
   * `2π` represents one full rotation (ie. 360°) in radians.
   * Multiplying by `2 ⋅ π` converts the frequency to radians per second.
   * Therefore `2πf` represents the angular frequency (`ω`) in radians per second.
 * `ω` is the angular frequency in radians per second.
   * To convert to angular frequency in radians per second from Hz, `ω = 2πf`
   * To convert angular frequency in radians per second to Hz, `f = ω ÷ 2π`
 * φ means phase.


The equations:

 * `R` — __Load resistance__
   * This is the optimal load resistance required to produce the maximum output power `P`.
   * __*VK1SV* variant__ (2001 or later)
     * Javascript: `var R=0.576801*((Vcc-Vo)*(Vcc-Vo)/P)*(1.0000086-(0.414396/Q)-(0.577501/(Q*Q))+(0.205967/(Q*Q*Q)));`
     * Unicode:    `R = 0.576801 ⋅ ((VCC-Vo)² ÷ P) ⋅ (1.0000086 - (0.414396/Q) - (0.577501/Q²) + (0.205967/Q³))`
     * Github:     $$R = 0.576801 \cdot \frac{(V_{CC}-V_o)^2}{P} \cdot [1.0000086 - \frac{0.414396}{Q} - \frac{0.577501}{Q^2} + \frac{0.205967}{Q^3}]$$
     * Judging from source code of the [VK1SV calculator](https://people.physics.anu.edu.au/~dxt103/calculators/class-e.php), this seems to be a variant implementation of *Sokal's improved formula* (2001), with the polynomial formula `f(Q)` substituted in to provide the complete set of terms. The only known difference is that `0.414396` is used in place of the original's `0.414395` — a functionally insignificant difference, yet worth noting.
   * __*Sokal*'s improved formula__ (2001)
     * Unicode: `R = K * ((VCC-Vo)² / P) * (1 + f(Q))`
     * Where:
       * `K` is a constant and `f(Q)` is a third-order polynomial function of `Q` to provide a closer fit to empirical data.
       * `f(Q)` is a polynomial approximation to account for the effect of the loaded quality factor (`Q`) on the load resistance calculation.
         * Unicode: `f(Q) = 1.0000086 - 0.414395/Q - 0.577501/Q² + 0.205967/Q³`
         * Github:  $$f(Q) = 1.0000086 - {0.414395}{Q} - \frac{0.577501}{Q^2} + \frac{0.205967}{Q^3}$$
     * Advancement over *Raab* (2001) by providing improved accuracy across a wider range of `Q` values (especially lower `Q` values).
     * It accounts for non-ideal switch behavior by including the `Vo` term (voltage drop across the switch when on).
     * Originally published as *[Class-E High Efficiency RF/Microwave Power Amplifiers: Principles of Operation, Design Procedures, and Experimental Verification](theory/2001-class-e-rf-power-amplifiers-sokal.pdf)* by Nathan O. Sokal in *[QEX](https://www.arrl.org/qex/)* (2001), since [republished with corrections](theory/2006-class-e-high-efficiency-rf-microwave-pas-updated-corrected-sokal.pdf) (2006).
   * __*Raab* formula__ (2001)
     * Unicode: `R = ( ((VCC - Vo)^2) / P) ⋅ k1 ⋅ (1 + (k2 / QL) + (k3 / QL^2))`
     * Github:  $$R = \frac{(V_{CC} - V_o)^2}{P}[k_1][1 + \frac{k_2}{Q_L} + \frac{k_3}{Q_L^2}]$$
     * Where `k1` is Raab's first curve-fitting constant of `0.576801`, `k2` is Raab's second curve-fitting constant of `-0.451759`, `k3` is Raab's third curve-fitting constant of `-0.402444`, `VCC` is the supply voltage in volts, `P` is the desired output power in watts, `QL` is the loaded quality factor, and `Vo` is the minimum voltage across the switch ("saturation voltage").
     * Improves accuracy by including the effects of `QL` on the relationship between supply voltage, output power, and load resistance.
     * The inclusion of `QL` allows the equation to be used for both high-Q and low-Q designs.
       * For very high `QL` values, the `QL` terms become negligible and the equation approaches the ideal Class E case.
       * For lower `QL` values, the additional terms provide the necessary corrections to maintain accuracy.
     * This equation is significant because it allows Class E amplifiers to be designed more accurately for a wider range of applications, including those where a lower `Q` is desirable for bandwidth or other reasons. It has been widely adopted in both academic and industrial Class E design processes.
   * __Basic (infinite) equation__
     * Unicode: `R = 0.5768 ⋅ VDD² ÷ P`
     * Github:  $$R = \frac{0.5768 V_{DD}^2}{P_{out}}$$
     * Assumes a 50% duty cycle.
     * Practical tolerance: ±10-15%
   * __Finite variant__
     * Unicode: `R = 0.5768 ⋅ VDD² ÷ P ⋅ f(QL)`
     * Github: $$R = \frac{0.5768 V_{DD}^2}{P_{out}} \cdot f(Q_L)$$  
     * Where `f(QL)` is a correction factor derived from numerical simulations and based on loaded `Q`.
     * Allows for more accurate designs with finite choke inductance.
     * Practical tolerance: ±5-10%

 * `C1` — __Shunt capacitance__
   * Optimal shunt capacitance is a function of the operating frequency and the optimal load resistance, and therefore in turn the supply voltage and the desired output power.
   * __*Raab* formula__ (2001)
     * Unicode: `C1 = (1 / (34.2219 ⋅ f ⋅ R)) ⋅ (0.99866 + (0.91424 / QL) - (1.03175 / QL²)) + (0.6 / (2πf)²) ⋅ L1`
     * Github:  $$C_1 = \frac{1}{34.2219fR}[0.99866 + \frac{0.91424}{Q_L} - \frac{1.03175}{Q_L^2}] + \frac{0.6}{(2\pi f)^2L_1}$$
     * Where `f` is the operating frequency in Hz, `R` is the output load resistance in ohms, `QL` is the loaded quality factor, and `L1` is the RF choke or DC feed inductor value in henries.
     * This equation improves accuracy by including second-order effects of `QL` and accounting for the finite inductance of `L1`. The constants were derived from curve-fitting to exact numerical solutions.
   * __Basic (infinite) equation__
     * Unicode: `C1 = 0.1836 ÷ 2πfR`
     * Github:  $$C1 = \frac{0.1836}{2\pi f R}$$ 
     * Calculates the minimum shunt capacitance value at 50% duty cycle and infinite choke inductance.
     * Practical tolerance: ±15-20%
   * __Duty cycle variant__
     * Unicode: `C1 = kC ÷ 2πfR`
     * Github:  $$C1 = \frac{k_C}{2\pi f R}$$ 
     * Where `kC` is a coefficient dependent upon duty cycle.
     * Allows for non-50% duty cycle designs.
     * Practical tolerance: ±10-15%
   * __Finite DC feed inductance variant__
     * Unicode: `C1 = kC ÷ 2πfR ⋅ g(QL)`
     * Github:  $$C1 = \frac{0.1836}{2\pi f R} \cdot g(Q_L)$$
     * Where `g(QL)` is a correction factor based on loaded `Q`.
     * Provides more accurate results for real-world choke inductances.
     * Practical tolerance: ±8-12%

 * `L` — __Series inductance__
   * Optimal series inductance is a function of frequency and of the optimal load resistance, and therefore in turn the supply voltage and the desired output power.
   * __Basic (infinite) equation__
     * Unicode: `L = 1.15R ÷ 2πf`
     * Github:  $$L = \frac{1.15R}{2\pi f}$$
     * Exact value for 50% duty cycle, infinite choke inductance
     * Practical tolerance: ±10-15%
   * __Duty cycle variant__
     * Unicode: `L = (kL ⋅ R) ÷ 2πf`
     * Github:  $$L = \frac{k_L R}{2\pi f}$$
     * Where `kL` is a coefficient dependent on duty cycle.
     * Enables designs with non-50% duty cycles.
     * Practical tolerance: ±8-12%
     * Inductance has an inverse relationship to duty cycle, ie. it increases as the duty cycle lowers, and decreases as the duty cycle increases.
   * __Finite DC feed inductance variant__
     * Unicode: `L = 1.15R ÷ 2πf ⋅ h(QL)`
     * Github:  $$L = \frac{1.15R}{2\pi f} \cdot h(Q_L)$$
     * Where `h(QL)` is a correction factor based on loaded `Q`. It is usually less than one, reducing the required inductance.
     * Provides more accurate results for real-world choke inductances.
     * Practical tolerance: ±5-10%

 * `L1` — __RF Choke or DC feed inductance__
   * __*Choi* formula__ (2001)
     * Unicode: `L1 = kC * V²/(f ⋅ P) ⋅ (1 + k₁/QL + k₂/QL²)`
     * Github:  $$L_1 = \frac{k_C \cdot V^2}{f \cdot P} \cdot \left[1 + \frac{k_1}{Q_L} + \frac{k_2}{Q_L^2}\right]$$
     * Where `kC` is Choi's primary constant `0.2085`, `k₁` is Choi's first `QL` adjustment constant `1.789`, `k₂` is Choi's second `QL` adjustment constant `-1.481`, `V` is the supply voltage in volts, `P` is the desired power output in watts, `f` is the frequency in Hz, and `QL` is the loaded quality factor.
     * This formula by Jaehyeong Choi better accounts for variations in `QL`.
     * Tolerance: Approximately ±2-5%
     * Increasingly used in modern designs where accuracy is required.
   * __*Raab* improved infinite formula (1990s)__
     * Unicode: `L1 ≥ 5R ÷ 2πf`
     * Github:  $$L_1 \geq \frac{5 \cdot R}{2\pi f}$$
     * Calculates the minimum value of the "infinite" choke inductance approximation.
     * Here RF choke infinite inductance minimum value is a function of frequency and of the optimal load resistance, and therefore in turn the supply voltage and the desired output power.
     * Practical tolerance: +20-30% (higher values generally acceptable)
   * __*Kazimierczuk-Puczko* simplified formula__ (1987) — __NRND__
     * Unicode: `L1 ≥ 10 ⋅ L2`
     * Github:  $$L_1 \geq 10 \cdot L_2$$
     * Simplified formula for approximating the mimimum `L1` inductance.
     * First ratio-based approach to simplify the design process.
     * Widely referenced in the amateur radio community.
   * __*Kazimierczuk-Puczko* formula__ (1987) — __NRND__
     * Unicode: `L1 = kKP * V²/(f * P)`
     * Github:  $$L_1 = \frac{K_{KP} \cdot V^2}{f \cdot P}$$
     * Where `kKP` is the Kazimierczuk-Puczko constant (0.2116) which is an optimized coefficient based on ideal Class E operation, `V` is the supply voltage in volts, `P` is the desired power output in watts and `f` is the frequency in Hz.
     * Antique formula from Kazimierczuk and Puczko's 1987 paper which is no longer recommended.
     * At the time of its publication, the formula provided an improvement over Sokal's formula by considering the loaded quality factor (`QL`).
   * __*Sokal* formula__ (1975) — __NRND__
     * Unicode: `L1 = kS ⋅ V² ÷ (2πf ⋅ P)` or `L1 = kS ⋅ V² ÷ (ω ⋅ P)`
     * Github:  $$L_1 = \frac{K_S \cdot V^2}{2\pi \cdot f \cdot P}$$ or $$L_1 = \frac{K_S \cdot V^2}{\omega \cdot P}$$
     * Where `kS` is "Sokal's constant" (generally a value between 1.8 and 2, but observed to sometimes be higher in later use), `V` is the supply voltage in volts, `P` is the desired power output in watts, `f` is the frequency in Hz, and `ω` is the angular frequency in radians per second.
     * Antique formula from Sokal's 1975 paper which is no longer recommended.
     * Doesn't account for many factors considered in modern class E design, such as loaded Q factor, duty cycle, and device characteristics.
   * __Finite DC feed inductance rule of thumb variant__
     * Unicode: `L1 = kLRF ⋅ R ÷ 2πf`
     * Github:  $$L_1 = k_{LRF} \cdot \frac{R}{2\pi f}$$
     * Where `kLRF` is a coefficient based on desired loaded `Q` value, typically smaller than 5, allowing for more compact designs at the cost of some efficiency.
     * Allows for intentional finite choke inductance designs.
     * Practical tolerance: ±10-15%
   * __Optimized efficiency rule of thumb variant__
     * Unicode: `L1 = (0.732 ⋅ R) ÷ 2πf ⋅ j(QL)`
     * Github:  $$L_1 = \frac{0.732R}{2\pi f} \cdot j(Q_L)$$
     * Where `j(QL)` is a correction factor based on loaded `Q`, typically being greater than 1.
     * Optimizes efficiency for finite choke inductance designs.
     * Provides a balance between size and performance.
     * Practical tolerance: ±5-10%

 * `L2` — __Resonator inductance__ (aka "output inductor", aka "tank inductor")
   * __*Raab* formula__ (2001)
     * Unicode: `L2 = (QL ⋅ R) ÷ 2πf`
     * Github:  $$L_2 = \frac{Q_L \cdot R}{2\pi f}$$
     * Where `QL` is the loaded quality factor, `R` is the load resistance, and `f` is the operating frequency in Hz.
     * This equation represents an improvement by explicitly including `QL`, allowing more accurate calculation of `L2` for different loaded `Q` values. Previously, `L2` was often calculated assuming an infinite `Q`.

 * `C2` — __Series capacitance__
   * Optimal series capacitance is a function of frequency and of the optimal load resistance, and therefore in turn the supply voltage and the desired output power.
   * __*Raab* formula__ (2001)
     * Unicode: `C2 = (1 ÷ (2πfR)) ⋅ (1 / (QL - 0.104823)) ⋅ (1.00121 + (1.01468 / (QL - 1.7879)) ) - (0.2 / (2πf)² ⋅ L1)`
     * Github: $$C_2 = \frac{1}{2\pi fR}\frac{1}{Q_L - 0.104823}[1.00121 + \frac{1.01468}{Q_L - 1.7879}] - \frac{0.2}{(2\pi f)^2L_1}$$
     * Where `QL` is the loaded quality factor, `R` is the load resistance, `f` is the operating frequency in Hz, and `L1` is the RF choke or DC feed inductor value in henries.
   * __Basic (infinite) equation__
     * Unicode: `C2 = 1 ÷ 2πf(5.4466R)`
     * Github:  $$C_2 = \frac{1}{2\pi f(5.4466R)}$$ 
     * Exact value for 50% duty cycle, infinite choke inductance design.
     * Practical tolerance: ±15-20%
   * __Duty cycle variant__
     * Unicode: `C2 = 1 ÷ 2πf(kC0 ⋅ R)`
     * Github:  $$C_2 = \frac{1}{2\pi f(k_{C0}R)}$$
     * Where `kC0` is a coefficient dependent on duty cycle.
     * Allows for non-50% duty cycle designs.
     * Practical tolerance: ±10-15%
   * __Finite DC feed inductance variant__ (subsequent to *Raab* (2001))
     * Unicode: `C2 = 1 ÷ 2πf(5.4466R) ⋅ i(QL)`
     * Github:  $$C_2 = \frac{1}{2\pi f(5.4466R)} \cdot i(Q_L)$$
     * Where `i(QL)` is a correction factor based on the loaded `Q` factor `QL`. It is usually less than 1, reducing the required capacitance.
     * Provides more accurate results for real-world choke inductances.
     * Practical tolerance: ±8-12%

 * `Q` — __Quality factor__
   * Typical values in Class E amplifiers range from 1.8 to 5 for practical implementations, with 3 to 5 being common for narrow-band applications.
   * Lower values (around 1.8 to 3) are often used for broadband applications or when component tolerances are a concern.
   * Higher values (3-5) are often used for narrowband applications or when precision components are available.
   * The choice affects other circuit parameters, such as the shunt capacitance and the overall power output capability of the amplifier.
   * Unicode: `QL = (ωL) ÷ R`
   * Github:  $$Q_L = \frac{\omega L}{R}$$
   * Where `ω` is the angular frequency in radians per second, `L` is the resonator inductance, and `R` is the load resistance (usually 50 ohms).


### Additional formulae

 * `Q(BAND)` — __Quality factor equation__
   * Used to calculate the nominal `Q` factor of a radio band.
   * A function of band center frequency and aggregate band bandwidth.
   * Unicode: `Q(BAND) = f₀ / Δf`
   * Github:  $$Q_{BAND} = f_0 / \Delta f$$
   * Where:
     * `Q` is the quality factor.
     * `f₀` is the center frequency of the band (in Hz).
     * `Δf` is the aggregate bandwidth of the band (in Hz), typically measured at -3dB points.
 * `C(DRAINMAX)` — __Maximum MOSFET Drain Capacitance__
   * Maximum MOSFET drain capacitance is a function of frequency and load resistance.
   * Unicode: `C(DRAINMAX) = k / (2πf ⋅ R(LOAD))`
   * Github:  $$C_{DRAINMAX} = \frac{1}{2\pi f \cdot R_{LOAD}}$$
   * Where:
     * `k` is a scaling factor (typically between 1 (possibly unworkable at high frequencies) and 4, with 3 as a suggested default at high frequencies)
     * `f` is the operating frequency in Hz
     * `R(LOAD)` is the load resistance in ohms, typically 50Ω
   * Note this formula is merely suggested as a design guideline and has no theoretical basis in the literature.
 * __Transistor Selection Formulae__
   * __Figure of Merit (FOM)__
     * Unicode: `FOM = RDS(ON) ⋅ Q(g)`
     * Github:  $$FOM = R_{DS(ON)} \cdot Q_g$$
     * Traditional metric for MOSFET selection - does not function well at high frequencies.
   * __High Frequency Figure of Merit (HFFOM)__
     * Unicode: `HFFOM = RDS(ON) C(OSS)`
     * Github:  $$HF-FOM = R_{DS(ON)} \cdot C_{OSS}$$
     * Revised formula to account for the increased significance of output gate capacitance at high switching frequencies.
   * __Baliga's Figure of Merit (BFOM)__
     * Unicode: `BFOM = 1 ÷ (RDS(ON) ⋅ A)`
     * Github:  $$BFOM = \frac{1}{R_{DS(ON)} \cdot A}$$
   * __Baliga's High Frequency Figure of Merit (BHFFOM)__
     * Unicode: `BHFFOM = 1 ÷ (RDS(ON) ⋅ C(ISS))`
     * Github:  $$BHFFOM = \frac{1}{R_{DS(ON)} \cdot C_{iss}}$$
   * __Combined Figure of Merit (CFOM)__
     * Unicode: `CFOM = (1 / RDS(ON)) ⋅ Qg ⋅ C(OSS)`
     * Github:  $$CFOM = \frac{1}{R_{DS(ON)} \cdot Q_g \cdot C_{OSS}}$$
   * __Effective Capacitance__
     * Unicode: `C(EFF) = Q(OSS) ÷ V(DS)`
     * Github:  $$C_{EFF} = \frac{Q_{OSS}}{V_{DS}}$$
     * Provides a more accurate representation of the MOSFET's behavior at high frequencies than the static `C(OSS)` value.
   * __Switching Loss Formula__
     * Unicode: `P(SW) ≈ 0.5 ⋅ V(DS)² ⋅ f ⋅ C(OSS_EFF)`
     * Github:  $$P_{sw} \approx 0.5 \cdot V_{DS}^2 \cdot f \cdot C_{oss\_eff}$$
     * Estimates power dissipation.
     
## Zener diode selection

Zener diodes act as shunt capacitors.

If zener diodes are selected to provide a safety for voltage peaks, note that their capacitance should be a concern, and is frequently not specified in the datasheet.

General rules of thumb:
 * __Zener diodes are less effective at voltage regulation at higher frequencies__.
   * It may be necessary to use multiple diodes or to physically distribute them around the circuit depending upon your application.
 * __Zener diodes degrade rapidly in function with thermal stress__.
   * For systems which may see use with high transmit duty cycles, especially high power applications with a single zener diode, consider thermals during layout and/or consider using multiple zener diodes.
 * __Select the physically smallest zener diodes possible__.
   * Physically larger zener diodes will have greater capacitance.
 * __Select the highest voltage rated zener diodes possible__.
   * Capacitance is inversely related to voltage rating, very broadly as follows:
     * Low voltage zeners (< 5V): 200-400 pF (generally to be avoided)
     * Medium voltage zeners (5-20V): 50-200 pF (potentially acceptable)
     * High voltage zeners (> 20V): 5-50 pF (significantly preferable)

For example:
 * `VCC` is 12V
 * Using a "~5x" rule of thumb we can expect peaks of >60V
 * However, certain components selected are only be available at 30V ratings
 * Consequently, we seek to add zener diode based protection.

In this case, zener based protection would be best selected with a voltage rating *as close to 30V as possible*. If we find there are multiple options available, we should *select the physically smallest zener diode* with that voltage rating.

In a different case where a negligible capacitance value zener diode is unavailable and you must use a higher capacitance component (eg. perhaps if the desired voltage rating is only available at larger physical size, or multiple capacitors will be in use creating a higher aggregate effective shunt capacitance), you might try to reduce the shunt capacitor `C1` in value somewhat to compensate.

## Architecture

 * python
 * Unix style command-line execution (for maximum compatibility, integration, and ease of automation)

It pulls in various information from external JSON files.

 * __components.json__: Contains common sizes for capacitors and inductors, and common zener diode voltage cutoffs used in generating proposed circuits.
 * __transistors.json__: Contains detailed properties of MOSFETs used as the Class E Power Amplifier switching device as follows:

| Name        | Unit | Property                      |
| ----------- | ---- | ----------------------------- |
| `lcsc`      | N/A  | Component ID at [LCSC](https://www.lcsc.com/products/) |
| `vdss_v`    | V    | Maximum drain-source voltage  |
| `id_a`      | A    | Maximum drain current         |
| `rds_on_o`  | Ω    | RDS(On) resistance            |
| `ciss_pf`   | pF   | Input capacitance             |
| `coss_pf`   | pF   | Output capacitance            |
| `crss_pf`   | pF   | Reverse transfer capacitance  |
| `vgs_th_v`  | V    | Gate threshold voltage        |

## Contributing

 * All contributions gladly accepted.
   * Bugfixes
   * Documentation
   * New features
   * New transistors
   * Translations
   * Unit tests
   * Use cases

Python is not my usual language, so I'm very happy to take recommendations on library selection, unit testing, etc.
