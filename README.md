# 2.4 GHz Hairpin Bandpass Filter — FR4 Microstrip

A 5-pole Chebyshev hairpin bandpass filter designed for the 2.4 GHz ISM band on standard FR4 substrate, following the methodology of Brady (2002). The full design flow from prototype theory through ADS circuit optimization to Sonnet planar EM simulation is documented here.

## Specifications

| Parameter | Target | Simulated |
|---|---|---|
| Centre frequency | 2.4 GHz | 2.455 GHz |
| Filter type | Chebyshev | Chebyshev |
| Filter order | 5 poles | 5 poles |
| In-band ripple | 0.5 dB | |
| Fractional BW | 10% (240 MHz) | ~8.2% (~200 MHz) |
| Peak insertion loss | < 6 dB | 5.9 dB |
| Return loss at f0 | > 15 dB | 27.6 dB |
| Stopband rejection at 2.7 GHz | > 30 dB | > 40 dB |
| Substrate | FR4 | FR4 |
| Substrate thickness | 1.57 mm | 1.57 mm |
| εr | 4.4 | 4.4 |
| tan δ | ~0.02 | ~0.02 |
| Minimum feature size | 0.2 mm | 0.1 mm (outer gaps) |

## Design Flow

The design replicates Brady's three-stage methodology adapted to 2.4 GHz on FR4.

**Stage 1: Substrate characterization and prototype synthesis.** LineCalc in ADS was used to determine the 50 Ω microstrip width (w = 2.93 mm) and effective permittivity (K_Eff = 3.286) on FR4 1.57 mm at 2.4 GHz. A 5-pole 0.5 dB Chebyshev prototype was synthesized using standard g-values, giving the initial resonator arm lengths and coupling gap starting values.

**Stage 2: ADS circuit optimization.** A half-filter schematic was built using MLIN, MBEND, MTEE, MLEF, and MCLIN elements referencing a single MSub block. Two instances of the half-filter were placed back-to-back in a top-level schematic to form the full 5-pole filter. Three GOAL blocks drove a Gradient optimizer over 200 iterations: S11 < -16 dB across 2.28 to 2.52 GHz, S21 < -28 dB below 2.0 GHz, and S21 < -28 dB above 2.8 GHz.

**Stage 3: Sonnet planar EM simulation.** The optimized layout was exported to Sonnet Lite for full-wave 2.5D EM verification. The substrate stackup uses Metal1 on top (35 µm copper), FR4 dielectric (1.57 mm), and an implicit perfect ground at the box bottom. An ABS frequency sweep from 1.8 to 3.2 GHz was used. The EM result confirmed the bandpass response with a 55 MHz upward shift from the circuit model, consistent with Brady's reported discrepancy between ADS and Sonnet results.

## Key Design Decisions

Tapped input coupling via MTEE and MLEF stub was used at both ports rather than direct end-coupled feed, following Brady's original topology. This gives an extra degree of freedom to control external Q independently of the resonator coupling gaps.

The outer coupling gaps (between the feed resonators and their neighbors) are 0.6 mm and the inner coupling gaps are 0.2 mm. This asymmetric gap choice balances inter-resonator coupling strength against individual resonator unloaded Q degradation, which is the primary loss mechanism on FR4.

Lumped fringing capacitances were intentionally omitted from the final layout since the MLEF element in ADS already models open-end fringing internally. Including explicit capacitors on top of MLEF double-counts the effect and introduces layout elements with no EM simulation support in Sonnet and Momentum.

## Results vs Brady

| Parameter | Brady (Rogers, 3.85 GHz) | This work (FR4, 2.4 GHz) |
|---|---|---|
| Substrate | Rogers RO4003 | FR4 |
| tan δ | 0.0027 | 0.020 |
| Insertion loss | ~1 dB | 5.9 dB |
| Return loss | > 16 dB | 27.6 dB |
| Stopband rejection | > 28 dB | > 40 dB |

The 5x difference in insertion loss directly demonstrates the substrate loss penalty of FR4 versus low-loss microwave laminate. The higher tan δ of FR4 (approximately 7x higher than Rogers) limits the unloaded Q of each resonator to roughly 80 to 120, versus 300 to 500 on Rogers, which sets a fundamental floor on achievable insertion loss for a given bandwidth and pole count.

## Repository Structure

```
/
├── ADS/
│   ├── half_filter_v2/          # Half-filter schematic cell
│   ├── full_filter_optim/       # Back-to-back optimization schematic
│   └── MyFirstWorkspace_lib/    # ADS library
├── Sonnet/
│   └── BPF.son                  # Sonnet EM project file
├── docs/
│   └── ALL_xx-TIE-xxxx.pdf      # Project report (IEEE format)
└── README.md
```

## Tools

ADS (Keysight Advanced Design System), university license, for schematic entry, LineCalc substrate calculation, and circuit-level optimization. Sonnet Lite 17.56 for planar EM simulation and verification.

## Status

EM simulation complete. Layout cleared for PCB fabrication with minimum feature size 0.1 mm on outer coupling gaps. Post-fabrication VNA measurement (SOLT calibration, S11 and S21) will confirm simulation accuracy. A second layout iteration may be needed to correct the 55 MHz center frequency offset introduced by junction discontinuities not fully captured in the circuit model.

## References

G. L. Matthaei, L. Young, and E. M. T. Jones, Microwave Filters, Impedance-Matching Networks, and Coupling Structures. Artech House, 1980.

D. M. Pozar, Microwave Engineering, 4th ed. Wiley, 2011.

J.-S. Hong and M. J. Lancaster, Microstrip Filters for RF/Microwave Applications. Wiley, 2001.

D. Brady, "The Design, Fabrication and Measurement of Microstrip Filter and Coupler Circuits," High Frequency Electronics, July 2002.
