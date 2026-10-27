# The physics behind the animation

This page lists what the animation computes and how, so it can be checked or extended. Everything is in [`index.html`](../index.html); the function names below point to the relevant code.

## Units and notation

Angles on the sky are measured in units of the Einstein radius $`\theta_E`$ of the lens. $`\boldsymbol{x}`$ is a position on the image plane and $`\boldsymbol{y}`$ the position of the source; the **Source position** slider sets $`y = |\boldsymbol{y}|`$ and dragging the source in the caustics panel sets its direction.

## Lens equation, magnification and arrival time

```math
\boldsymbol{y} = \boldsymbol{x} - \boldsymbol{\alpha}(\boldsymbol{x}), \qquad
\mu = \frac{1}{\det A}, \quad A_{ij} = \frac{\partial y_i}{\partial x_j}, \qquad
\tau(\boldsymbol{x}) = \tfrac{1}{2}\,|\boldsymbol{x} - \boldsymbol{y}|^2 - \psi(\boldsymbol{x})
```

$`\boldsymbol{\alpha} = \nabla\psi`$ is the deflection and $`\psi`$ the lensing potential. Images are the stationary points of the arrival time $`\tau`$:

| Type | Stationary point | Condition | Morse phase |
|---|---|---|---|
| I | minimum | $`\det A > 0`$, $`\mathrm{tr}\,A > 0`$ | $`0`$ |
| II | saddle point | $`\det A < 0`$ | $`-\pi/2`$ |
| III | maximum | $`\det A > 0`$, $`\mathrm{tr}\,A < 0`$ | $`-\pi`$ |

To find them (`solveImages`), the image plane is sampled on a 281 × 281 grid; every grid triangle whose image on the source plane contains the source gives a starting point, which Newton's method refines. Images are sorted by arrival time, so image 1 always arrives first.

## Lens models

Both lenses are softened isothermal ellipsoids (Keeton & Kochanek 1998; Keeton 2001; `makeLensModel`) with Einstein radius 1, axis ratio $`q = 1 - \varepsilon`$ (major axis along $`x_1`$) and core radius $`s`$:

```math
\psi_r = \sqrt{q^2 (s^2 + x_1^2) + x_2^2}, \qquad e = \sqrt{1 - q^2},
```

```math
\alpha_1 = \frac{q}{e} \arctan\frac{e\, x_1}{\psi_r + s}, \qquad
\alpha_2 = \frac{q}{e} \mathrm{artanh}\,\frac{e\, x_2}{\psi_r + q^2 s},
```

```math
\psi(\boldsymbol{x}) = \boldsymbol{x}\cdot\boldsymbol{\alpha} - \frac{q\, s}{2} \ln\!\left[(\psi_r + s)^2 + (1 - q^2)\, x_1^2\right] \quad \text{(up to a constant)}.
```

- **Galaxy:** $`s = 0`$, a singular isothermal sphere (SIS) when $`\varepsilon = 0`$ and a singular isothermal ellipsoid (SIE) otherwise.
- **Cluster:** $`s = 0.1`$. The core adds a faint central image (type III).

For the SIS everything is analytic: for $`y < 1`$ there are two images at $`x = y + 1`$ and $`x = y - 1`$, with $`|\mu_\pm| = 1/y \pm 1`$ and $`\Delta\tau = 2y`$; for $`y > 1`$ only the first one is left, with $`\mu = 1 + 1/y`$.

## Caustics and critical curves

The critical curves are where $`\det A = 0`$, that is, where one eigenvalue of $`A`$ vanishes: $`1 - \kappa - \gamma = 0`$ (tangential) or $`1 - \kappa + \gamma = 0`$ (radial). The caustics are their images on the source plane. Each time the source crosses a caustic, two images appear or merge, with a magnification that diverges on the caustic. A singular lens ($`s = 0`$) has no radial caustic but a *cut*: when the source crosses it, the innermost image disappears into the centre of the lens.

In the animation (`updateLensModel`, `zeroContour`) the zero contours of the two eigenvalues are traced on the grid with marching squares and mapped to the source plane.

## The lensed signal

In geometric optics each image $`j`$ multiplies the signal, in the frequency domain, by

```math
F_j(f) = \sqrt{|\mu_j|}\; \exp\!\left[\,2\pi i f\, t_j - i \pi n_j\, \mathrm{sign}(f)\right], \qquad n_j = 0, \tfrac{1}{2}, 1 \ \text{for types I, II, III},
```

and the detector records the sum over images. For a signal dominated by one harmonic, as in the animation, the Morse phase of a type II image is a constant phase shift of $`-\pi/2`$; with higher harmonics, precession or eccentricity it also distorts the waveform (Dai & Venumadhav 2017; Ezquiaga et al. 2021).

In the waveform panel the amplitudes are normalised to image 1, and the dashed curve under each copy is what the detector would receive without the lens: same arrival time, amplitude $`1/\sqrt{\mu_1}`$ in those units, no Morse phase.

## Time delays in physical units

```math
\Delta t = \frac{1 + z_d}{c}\, \frac{D_d D_s}{D_{ds}}\, \theta_E^2\, \Delta\tau,
```

with $`D_d`$, $`D_s`$ and $`D_{ds}`$ the angular-diameter distances to the lens, to the source and between them. The animation uses a lens at $`z_d = 0.5`$ and a source at $`z_s = 2`$ in flat ΛCDM ($`H_0 = 70`$ km s⁻¹ Mpc⁻¹, $`\Omega_m = 0.3`$):

- **Galaxy:** velocity dispersion 250 km/s, so $`\theta_E = 4\pi (\sigma/c)^2 D_{ds}/D_s \approx 1.15''`$ and $`\Delta t \approx 109\ \text{days} \times \Delta\tau`$.
- **Cluster:** $`\theta_E = 10''`$, so $`\Delta t \approx 8320\ \text{days} \times \Delta\tau`$.

These delays are written above the waveform panel (`DAYS_PER_TAU`). The time axis of the panel is compressed: consecutive copies are drawn at least 1.5 animation seconds apart so each chirp can be seen on its own.

## Microlensing by a point mass, in wave optics

A star of mass $`M`$ near image 1, with the source at $`y`$ in units of the star's Einstein radius, multiplies that image by the amplification factor of a point mass (Nakamura 1998; Takahashi & Nakamura 2003; `microF`):

```math
F(w, y) = \exp\!\left[\frac{\pi w}{4} + \frac{i w}{2}\left(\ln\frac{w}{2} - 2\phi_m(y)\right)\right]
\Gamma\!\left(1 - \frac{i w}{2}\right)\, {}_1F_1\!\left(\frac{i w}{2};\, 1;\, \frac{i w y^2}{2}\right),
```

```math
w = \frac{8\pi G M (1 + z_d)\, f}{c^3}, \qquad x_m = \frac{y + \sqrt{y^2 + 4}}{2}, \qquad \phi_m = \frac{(x_m - y)^2}{2} - \ln x_m.
```

- For $`w \ll 1`$ the wavelength is much larger than the Schwarzschild radius of the star, the wave diffracts around it and $`|F| \to 1`$.
- For $`w \gg 1`$ geometric optics returns: two micro-images, the second delayed and with phase $`-\pi/2`$, which interfere and make the amplification oscillate with frequency.

At $`f = 100`$ Hz, $`w = 1`$ for $`M(1 + z_d) \approx 80\, M_\odot`$, so wave effects matter for lenses of roughly 10 to $`10^4`$ solar masses in ground-based detectors. The animation uses $`y = 0.7`$ and $`4GM/c^3 = 0.05`$ animation seconds, so $`w \approx 4`$ at the merger frequency: the effect is exaggerated so it can be seen. Image 1's chirp is Fourier transformed (1024 points), multiplied by $`F`$ and transformed back (`buildMicroWave`); $`\Gamma`$ uses the Lanczos approximation and $`{}_1F_1`$ its power series.

## What is a cartoon

- **The chirp.** The frequency grows as $`(t_c - t)^{-3/8}`$ and the amplitude as $`f^{2/3}`$, with a smooth peak at the merger and a ringdown whose frequency rises to 1.4 times the peak frequency while the amplitude decays. Frequencies are in "animation hertz" (14 Hz at the merger) so the chirp can be followed by eye.
- **The binary.** The separation shrinks as $`a_0(1 - p^2)`$ and the angular velocity follows Kepler's law, $`\omega \propto a^{-3/2}`$, capped near the plunge.
- **The spacetime sheet.** The lens dents it with a smooth well; the paths are smooth curves through the image positions, and each image spends its extra delay where its path runs deep in the well (a cartoon of the Shapiro delay). Distant sources are pulled in so they stay on the sheet.

## References

- P. Schneider, J. Ehlers and E. E. Falco, *Gravitational Lenses* (Springer, 1992).
- Y. Wang, A. Stebbins and E. L. Turner, Phys. Rev. Lett. 77, 2875 (1996).
- T. T. Nakamura, Phys. Rev. Lett. 80, 1138 (1998).
- C. R. Keeton and C. S. Kochanek, Astrophys. J. 495, 157 (1998).
- C. R. Keeton, *A catalog of mass models for gravitational lensing*, arXiv:astro-ph/0102341 (2001).
- R. Takahashi and T. Nakamura, Astrophys. J. 595, 1039 (2003).
- C. M. Hirata, D. E. Holz and C. Cutler, Phys. Rev. D 81, 124046 (2010).
- L. Dai and T. Venumadhav, *On the waveforms of gravitationally lensed gravitational waves*, arXiv:1702.04724 (2017).
- K. Haris, A. K. Mehta, S. Kumar, T. Venumadhav and P. Ajith, *Identifying strongly lensed gravitational wave signals from binary black hole mergers*, arXiv:1807.07062 (2018).
- J. M. Ezquiaga, D. E. Holz, W. Hu, M. Lagos and R. M. Wald, Phys. Rev. D 103, 064047 (2021).
- H. Phurailatpam et al., *ler: LVK (LIGO-Virgo-KAGRA collaboration) event (compact-binary mergers) rate calculator and simulator*, J. Open Source Softw. 11, 7420, doi:10.21105/joss.07420.
