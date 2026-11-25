# Probing the Photonuclear Cross Section with the Cygnus Air-Shower Excess

Nickolay&nbsp;S.&nbsp;Martynenko<sup>1,2,*</sup>, Grigory&nbsp;I.&nbsp;Rubtsov<sup>1,2</sup>, Sergey&nbsp;V.&nbsp;Troitsky<sup>1,2</sup>

<sup>1</sup> Lomonosov Moscow State University, 1-2 Leninskie Gory, Moscow 119991, Russia<br>
<sup>2</sup> Institute for Nuclear Research of the Russian Academy of Sciences, 60th October Anniversary Prospect 7a, Moscow 117312, Russia<br>
<sup>*</sup> Corresponding author: [martynenko@inr.ac.ru](mailto:martynenko@inr.ac.ru)

Preprocessed CONEX shower samples, photon-selection efficiency scans, and CORSIKA muon counts used to fit the lateral distribution in *Probing the photonuclear cross section with the Carpet-2 air-shower excess from the Cygnus region*.

## Download and read the data

Install [Git LFS](https://git-lfs.com), run `git lfs install`, and clone this repository. If you already have a clone, run `git lfs pull` from its root. Use [NumPy](https://numpy.org/) to read the shower arrays and [SciPy](https://scipy.org/) to read the NetCDF scans.

## Files

| Path | Content |
| --- | --- |
| `showers/diffuse_gamma/` | Isotropic photon showers used to calibrate the photon-selection criterion |
| `showers/cygnus_gamma/` | Photon showers from the direction of the Cygnus Cocoon used in both scans |
| `showers/diffuse_proton/` | Isotropic proton showers used for comparison in the muon-to-electron ratio distribution |
| `scans/cygnus_tpr_scan.nc` | Efficiency scan over the cross-section scale, spectral index, and cutoff energy |
| `scans/cygnus_tpr_scan_bykov.nc` | Efficiency scan over the cross-section scale at a fixed flare spectrum |
| `ldf/corsika-300tev-photon-shower-muon-radial-counts.npz` | Per-shower radial and total muon counts used to fit the photon-shower muon lateral distribution |
| `ldf/metadata.json` | CORSIKA settings, array descriptions, and fit conventions |

### Shower samples

Each directory under `showers/` contains `arrays.npz` and `metadata.json` for 65 536 showers. The JSON file records the simulation and sampling settings and the array units.

| Keys in `arrays.npz` | Content |
| --- | --- |
| `events/` | Base-10 logarithm of primary energy in eV, zenith and azimuth angles, and exposure weights |
| `profiles/` | Slant depth, altitude, and longitudinal electron and muon counts |
| `ground/` | Slant depth and electron and muon counts at the profile point nearest the observatory altitude |

```python
import numpy as np

with np.load("showers/cygnus_gamma/arrays.npz") as showers:
    log10_energy_eV = showers["events/log10_primary_energy_eV"]
    muon_profiles = showers["profiles/muon_count"]
```

### True positive rate scans

In both NetCDF files, `TPR` (true positive rate) is the weighted fraction of simulated photon showers that pass the photon-median cut among those that pass the quality and reconstructed-energy cuts. The coordinate `scales` is the multiplier applied to the EGS4 photonuclear cross section on air at photon energies of 10 GeV and above in the target-nucleus rest frame. Below 10 GeV the multiplier is unity.

- `cygnus_tpr_scan.nc` stores `TPR` on a grid of shape `(61, 41, 41)`. The axes are `scales` (0.1 to 100), `spectrum_negative_slope` (spectral index α in dN/dE ~ E^−α exp(−E/E_cutoff), from −1 to 3), and `log10_energy_cutoff_eV` (base-10 logarithm of the cutoff energy E_cutoff in eV, corresponding to 0.3–30 PeV).
- `cygnus_tpr_scan_bykov.nc` stores `TPR` at 93 cross-section scales from 0.3 to 20. The fixed spectral parameters, α = 0.13 and `log10(E_cutoff / eV) = 15.45`, are stored as global attributes. They come from a fit to the flare spectrum in [Bykov _et al._ (2021), Fig. 3](https://doi.org/10.3847/2041-8213/ac2f3d).

```python
from scipy.io import netcdf_file

with netcdf_file("scans/cygnus_tpr_scan.nc", "r", mmap=False) as scan:
    scales = scan.variables["scales"][:]
    tpr = scan.variables["TPR"][:]
```

### CORSIKA muon lateral distribution

The CORSIKA sample contains 8 192 vertical photon showers at 300 TeV. Muons with kinetic energy of at least 1 GeV were counted at an observation altitude of 1 700 m. The simulation settings are listed in `ldf/metadata.json`.

- `counts` has shape `(8192, 30)`. Each row gives the muon counts in 1 m radial bins from 30 to 60 m, measured from the shower axis. The first bin is [30, 31) m and the last is [59, 60) m.
- `totals` has shape `(8192,)`. Each entry is the muon count over all radii for the corresponding shower.

The fit uses counts summed over showers in each radial bin and one additional count for all muons outside 30–60 m. The lateral distribution is normalized over the full plane, with the radial scale fixed at 320 m.

## Citation

If you use these data, please cite N. S. Martynenko, G. I. Rubtsov, and S. V. Troitsky, *Probing the photonuclear cross section with the Carpet-2 air-shower excess from the Cygnus region* (2026).

## License

The files in this repository are distributed under the MIT License. See [LICENSE](LICENSE).
