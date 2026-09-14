# Datasets

Every dataset is published twice, with identical rows and columns in the same order:

| | `data/imperial/` | `data/SI/` |
|---|---|---|
| spans | ft | m |
| deck depth, toppings | in | mm |
| scaled embodied carbon (`EC_Scaled`, six columns) | **lb CO2e / ft2** | **kg CO2e / m2** |
| per-component embodied carbon (twenty `EC_*` columns) | lb CO2e per bay | kg CO2e per bay |
| peak acceleration, frequency, damping, counts, codes, member labels | identical | identical |

| file (imperial / SI) | rows | design space |
|---|---|---|
| `composite_results_*.csv` | 30,258 | girder and infill spans 10-50 ft in 1 ft steps (41 x 41) x NWC/LWC x 1.5/2/3 in deck x 25/50/75 % composite |
| `noncomposite_results_*.csv` | 10,086 | 41 x 41 spans x NWC/LWC x 3 decks |
| `damping_study_composite_*.csv` | 7,500 | 1,500 Latin-hypercube bays x damping 1, 2, 3, 4, 5 % |
| `damping_study_noncomposite_*.csv` | 2,500 | 500 bays x damping 1-5 % |

Every column header carries its unit or meaning in parentheses.

## Embodied carbon: read this before converting anything

The five carbon factors of Table 3 -- steel 1.22, deck 2.32, studs 1.27, NWC 0.133, LWC 0.307 -- are
mass ratios: **kg CO2e per kg of material, which is the same number as lb CO2e per lb**. So the
imperial files hold **lb CO2e per ft2** (and lb CO2e per bay) computed directly from the lb take-off
masses, and the SI files hold **kg CO2e per m2** (and kg CO2e per bay).

* imperial -> SI carbon per area: `kg CO2e/m2 = lb CO2e/ft2 x 0.45359237 x 10.763910417` (= x 4.882427636).
* imperial -> SI carbon per bay: `kg CO2e = lb CO2e x 0.45359237`.
* The factors themselves are never converted.

(The notebooks print `Embodied Carbon Scaled` in kg CO2e/ft2 -- they convert the lb masses to kg once,
with `LB_TO_KG = 0.45359237`, before applying the factors. Divide that printout by 0.45359237 to get the
imperial file's value, or multiply it by 10.763910417 to get the SI file's value.)

The take-off counts each bay as a repeating unit of the floor plate: all interior infills, **one** of the
two edge infills and **one** of the two girders, so members shared with the next bay are not counted
twice. The five component columns therefore satisfy, on every row,

    EC_Infill   [lb CO2e] = W_infill [lb/ft] x infill span [ft] x (interior infills + 1) x 1.22
    EC_Girder   [lb CO2e] = W_girder [lb/ft] x girder span [ft] x 1                     x 1.22
    EC_Deck     [lb CO2e] = deck weight [psf] x bay area [ft2]                          x 2.32
    EC_Concrete [lb CO2e] = (topping + flute fill) [in] / 12 x density [pcf] x area     x ECF_conc
    EC_Studs    [lb CO2e] = stud weight [lb] x (studs per infill x counted infills + studs on the girder) x 1.27
    EC_Scaled [lb CO2e/ft2] x bay area [ft2] = EC_Infill + EC_Girder + EC_Deck + EC_Concrete + EC_Studs

and the SI file carries the same five terms x 0.45359237 (kg CO2e per bay) with `EC_Scaled` in kg CO2e/m2.

with flute fill 0.532414 / 1.0 / 1.5 in for the 1.5 / 2 / 3 in decks, 145 pcf and 0.133 for NWC,
110 pcf and 0.307 for LWC, and stud weights 0.374 / 0.437 / 0.562 lb. These identities were checked on 100 random
designs per file, every design state, against the AISC and deck tables in this repository -- the lb values
against the imperial file and their kg conversion against the SI file -- and the two unit systems were
compared column by column: 400 / 400 pass.

## Columns

* `Girder Span`, `Infill Span`, `Conc_Type_num` (0 = NWC, 1 = LWC), `Deck_Type`, `Clear_Spans`,
  `Damping Ratio`, `Building Use` (3 = residence), `Percent Composite Connection` (composite only) --
  the inputs. Damping-study files add `Study`, `Bay ID` and `Damping Level (%)`.
* Five design states, each with member sizes, topping, `EC_Scaled`, `Peak Acceleration` and
  `Fundamental Frequency`: `Initial` (lightest-carbon design that meets strength and deflection),
  `Mitigated Infill` / `Mitigated Girder` (upsize one member family until 0.5 %g), `Mitigated Conc`
  (add topping in 0.25 in steps until 0.5 %g; `Additional Topping` is the increment), `Mitigated Steel`
  (upsize infills and girders together), and `Best` (the lower-carbon of the two strategies).
* Member labels: AISC designation; composite labels add `[studs] + camber"`, camber in inches in both files.
* `Best Method old`: the engine's code -- 1 pass, 2 upsize infills, 3 upsize girders, 4 add topping,
  5 none feasible, 6 upsize infills and girders. `Best Method`: 2 and 3 folded into 6 (they are the
  same steel-upsizing outcome; in every folded row the member set and carbon are identical).
* `Initial PA over limit` = max(initial peak acceleration - 0.5, 0) %g.
* The twenty `EC_*` per-component columns for the `Initial`, `Mitigated Steel`, `Mitigated Conc` and
  `Best` states.
* A state that does not apply (e.g. `Mitigated Conc` when no topping reaches 0.5 %g, or any mitigation on
  a design that passes as built) is written as all zeros. There are no blank cells in any file.

## How the two sets relate

`data/imperial/` is the engine's output with the carbon columns divided by 0.45359237 (the engine
works in kg), units added to the headers, and one documented repair. The sampler read `Initial Topping` from the concrete-mitigation
frame and `Best Topping` from the best-method frame, so it wrote `Initial Topping` = 0 on the 6,270
designs that pass 0.5 %g as built (no mitigation ran) and left `Best Topping` blank on the 42,868 rows
whose best design is the initial design or a steel upsizing (those frames carry no topping). Both
columns are now filled with the topping implied by that state's `EC_Concrete`, asserted on every
affected row to equal the deck table's 2.0 in (initial) or the initial topping (best design with no
added topping). No carbon value was changed.

`data/SI/` is produced from `data/imperial/` (x 0.3048, x 25.4,
x 0.45359237 x 10.763910417 for carbon per area, x 0.45359237 for carbon per bay); converted values are
rounded to 9 decimals so that 1.5 in reads 38.1 mm and can be matched with `==`.
