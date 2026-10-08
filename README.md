# CharmmGuiAuto

CharmmGuiAuto is a tool designed to automate interactions with CHARMM-GUI, a web-based graphical user interface for CHARMM (Chemistry at HARvard Macromolecular Mechanics), a widely used software for molecular simulations. This tool simplifies the process of preparing input files for molecular dynamics simulations.

## About this fork

This is a maintained fork of [AmandaStange/CharmmGuiAuto](https://github.com/AmandaStange/CharmmGuiAuto) by **Prithvinath Gollakota** ([@CodingCodon](https://github.com/CodingCodon), IIT Gandhinagar). The original tool and its design are by Amanda D. Stange and co-authors; please cite their paper (see [Citation](#citation)). If this fork's fixes helped you, a mention of the fork is appreciated.

The fixes below were needed to run Solution Builder against CHARMM-GUI and current Selenium as of October 2026. They were found while building 40 peptide systems (wild-type and lysine-acetylated) and checked against systems built by hand in the browser.

| Fix | What went wrong before |
|---|---|
| Dropped the unused `FirefoxBinary` import | Script crashed at import on Selenium ≥ 4.10. |
| Optional `geckodriver` path (`system_info.geckodriver`, or `CHARMMGUIAUTO_GECKODRIVER`, or `geckodriver` on PATH, or `/snap/bin/geckodriver`) | Selenium Manager could not find the driver for Ubuntu's snap Firefox ("Unable to obtain driver for firefox"). |
| `path_out` no longer needs a trailing `/` | Without it, the temporary download folder was created beside the output folder. |
| `https://charmm-gui.org` instead of `https://www.charmm-gui.org` | The `www` host did not resolve. |
| **New: Lys/Arg PTMs** (`ptms:`, e.g. lysine acetylation `KAC`) | These patches are in CHARMM-GUI's separate "Lys / Arg PTMs" table, which the tool could not reach; `phosphorylations:` has no LYS entries. |
| Phosphorylation row index | Taken from the last character of the element id, so from the 10th phosphorylation on the wrong row was filled. |
| pH option hidden | CHARMM-GUI hides the pH box when its PROPKA run gives no result for the job (this can differ between submissions of the same structure). Setting a pH then crashed. It is now skipped with a message, and `self.ph_status` records `set`, `off` or `unavailable`. |
| Explicit water box (`waterbox: {size: explicit, ...}`) | Failed with "Invalid system volume": the form was clicked before the page finished loading, so the size fields were never created. The page is now waited for, the fields are cleared before typing, and the values are read back. |
| Error messages | Every failure printed only "A very specific bad thing happened."; the traceback is now printed first. |

New YAML keys, by example ([`Example_input_yaml/SolutionProteinPTM.yaml`](Example_input_yaml/SolutionProteinPTM.yaml)):

```yaml
system_info:
  geckodriver: /snap/bin/geckodriver   # optional
details:
  pH: 6.8
  ptms:                                # Lys / Arg PTMs
    - chain: PROA
      res_i: LYS
      rid: '16'
      res_p: KAC
  waterbox:                            # explicit cubic box, 95 Å edges
    size: explicit
    shape: rect
    X: 95
    Y: 95
    Z: 95
```

Not changed: the CHARMM-GUI terms of use still apply to automated use. Run jobs at a pace similar to a person with a few browser tabs.

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Example Input Files](#example-input-files)
- [Known Issues](#known-issues)
- [How to Contribute](#how-to-contribute)
- [Citation](#citation)
- [License](#license)

## Introduction

CHARMM-GUI is a powerful tool for preparing complex molecular systems for simulation, but its web interface can be cumbersome for repetitive tasks. CharmmGuiAuto aims to automate these interactions, streamlining the preparation process and reducing manual input errors.

## Features

- Automates the generation of MD input files using CHARMM-GUI.
- Simplifies the preparation of molecular dynamics simulations.
- Provides a set of example YAML files for easy customization.
- Integrates seamlessly with CHARMM-GUI web interface.

## Installation

To install CharmmGuiAuto, clone the repository and install the required dependencies:

```sh
git clone https://github.com/CodingCodon/CharmmGuiAuto.git   # this fork; the original is AmandaStange/CharmmGuiAuto
cd CharmmGuiAuto
micromamba create -n charmmauto -f requirements.txt -c conda-forge
micromamba activate charmmauto
```

## Usage

Here is a basic example of how to use CharmmGuiAuto:

1. Prepare your input YAML file. You can start with one of the examples provided in the `Example_input_yaml` directory.
2. Run the script with your YAML file as input.

```sh
python CharmmGuiAuto.py -i Example_input_yaml/MembraneProtein.yaml
```

## Example Input Files

The `Example_input_yaml` directory contains sample YAML files that demonstrate how to configure various simulations, and gives an overview of the different parameters that can be changed. You can customize these files to fit your specific needs.

## Known Issues

- Only works with Firefox.
- Cannot be used to continue retrieved jobs, but can download jobs that are finished but not yet downloaded if you have the job ID.
- ~~Firefox binary (`firefox_binary.cpython-37.pyc` ...) must be placed in the `bin` of your environment.~~ Not needed in this fork: the import was removed. If Selenium cannot find geckodriver, pass its path (see [About this fork](#about-this-fork)).
- The forms are matched by element ids and some absolute XPaths, so changes to CHARMM-GUI's pages can break individual steps. Tested in this fork (October 2026) only for Solution Builder from a local PDB with terminal patches, pH, Lys PTMs, an explicit box, NaCl, CHARMM36m and GROMACS output. The other workflows only received the shared fixes.

## How to Contribute

Contributions are welcome! 

If you would like to contribute:

1. **Fork** this repository.
2. **Clone** your fork locally.
3. **Create a branch** for your changes.
4. **Make your changes**, following PEP 8 style guidelines.
5. **Commit and push** your changes.
6. **Open a pull request**.

For detailed contribution guidelines, please see [CONTRIBUTING.md](CONTRIBUTING.md).

If you plan to contribute **a new feature** or **significant changes**, please open an issue first to discuss your idea.

## Citation

If you use CharmmGuiAuto in your research, please cite the following article:

Stange, A.D., Zuzic, L., Schiøtt, B., & Berglund, N.A. (2025). Exploring insulin-receptor dynamics: Stability and binding mechanisms. *Structure*, 33(8), 1–11. https://doi.org/10.1016/j.str.2025.04.022

BibTeX entry is provided in [citation.bib](citation.bib).

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.