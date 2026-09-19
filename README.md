# MS-Vibration-Code

Code and data for *Carbon-Efficient Design Strategies for Mitigating Footfall Vibration in Steel Framed Floor Systems* (Farrow, Brown and Solnosky; *Engineering Structures*, under review), and for the MS thesis it grew out of.

**AI Usage Disclosure:** All code for the composite and non-composite sizing engines was handwritten over the course of the thesis. Claude Opus 5 was used to help identify bugs, streamline plotting code, and run agents to assist with sampling. All AI-assisted outputs were rigorously validated against the original handwritten thesis results to confirm consistency.

| File | Purpose |
|---|---|
| `Composite MS Code.ipynb` | The composite floor engine: deck and slab sizer, composite steel sizer, AISC DG 11 vibration check, embodied carbon take-off, and the two mitigation paths (upsize steel / add concrete topping). |
| `Non-Composite MS Code.ipynb` | Same for the non-composite system. |
| `AISC STEEL MANUAL V16 (Modified) Sorted.csv`, `DeckData.csv` | The two database tables the notebooks read. Keep them in the same folder as the notebooks. |
| `data/imperial/`, `data/SI/` | The published datasets (40,344 grid designs and the 10,000-run damping study), in imperial and SI units. `data/README.md` is the column dictionary. |
| `Hand Calcs.pdf` | Hand verification of the vibration calculation. |

## Running the code

1. Download the two `.csv` database files.
2. Download `Composite MS Code.ipynb` and/or `Non-Composite MS Code.ipynb` to the same location the `.csv` files were saved.
3. Install the dependencies in your Python environment: `numpy`, `pandas` (keep `pandas<3`), `matplotlib`. Developed on Python 3.13 with numpy 2.1 and pandas 2.2.
4. Set the input variables in the last cell (`sizer_and_calc(...)`): spans in ft, `Conc_Type_num` 0 = NWC / 1 = LWC, `Deck_Type` 1.5 / 2 / 3 in, and for composite `percent_comp` 25 / 50 / 75.
5. Run all cells.
6. The printout gives the initial design, each mitigated design, and the objective values (peak acceleration in %g, scaled embodied carbon) for each. The printed `Embodied Carbon Scaled` is in kg CO2e/ft2; divide by 0.45359237 for the lb CO2e/ft2 value in `data/imperial/`, or multiply by 10.763910417 for the kg CO2e/m2 value in `data/SI/`.

## Units and embodied carbon

Every column header in `data/` carries its unit. `data/imperial/`: spans in ft, deck depth and toppings in inches, embodied carbon in **lb CO2e/ft2** and lb CO2e per bay. `data/SI/`: spans in m, deck depth and toppings in mm, embodied carbon in **kg CO2e/m2** and kg CO2e per bay. The Table 3 carbon factors (steel 1.22, deck 2.32, studs 1.27, NWC 0.133, LWC 0.307) are mass ratios -- kg CO2e per kg is the same number as lb CO2e per lb -- so the same factors apply in both systems; SI is imperial x 0.45359237 (mass) x 10.763910417 (area) for carbon per area, and x 0.45359237 for carbon per bay.

Each bay is a repeating unit of the floor plate: all interior infills, one of the two edge infills and one of the two girders are counted, so members shared with adjacent bays are not counted twice. Normal-weight concrete is 145 pcf, lightweight 110 pcf.

## Citation and licence

See `CITATION.cff`. Released under the MIT licence (`LICENSE`).
