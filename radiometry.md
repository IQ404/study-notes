# Radiometry

### Intensive quantity and Extensive quantity:

  Intensive quantity (e.g. pressure) is a density function for an extensive quantity (e.g. force) with respect to another extensive quantity (e.g. area).

  ❓ What about intensive quantity over extensive quantity? (e.g. Spectral Power)

### Spectral Energy

- Unit: $J(nm)^{-1}$ i.e. Energy per wavelength

### Power

- Unit: $J(s)^{-1}$ or $W$ i.e. Energy per second

### Spectral Power
- Unit: $W(nm)^{-1}$ i.e. Power per wavelength
- Often denote as $\Phi$

### Irradiance

- See Spectral Irradiance, because we probably always want to (mostly implicitly) specify the wavelength.

### Spectral Irradiance
  
- (Spectral) Power per unit area of incident light.
- Unit: $J(s)^{-1}(nm)^{-1}(m)^{-2}$
- Often denote as $H$

### Radiant Exitance
  
- The same quantity as (spectral) irradiance, but for exitant light (light leaving the surface. E.g. reflected light).
- Often denote as $E$

### Radiance
  
- See Spectral Radiance, because we probably always want to (maybe implicitly) specify the wavelength.

### Spectral Radiance

Spectral radiance is a really essential and perhaps the most frequently used concept in graphic program. 

Admittedly, it caused me a lot of headache trying to understand its "physical" meaning when I first learned it. And I can't say I really understand it now. But here is my current understanding: (<ins>MAY NEED IMPROVEMENTS!!!</ins>)

Firstly, our definition of spectral irradiance $H$ can be conceptually interpreted as follows: let there be light constrained to some certain wavelength, think of a point of interest in the space as a hole where photons can enter when the hole is open, and the hole will only open for exactly 1 second. Now, count all the photons that enter the hole during this one second, and denote the total energy of those photons as $q$. Now, think of the point representing the hole as a point on a plane of unit area, and let the total number of points in this unit area to be $N$ (this is an abuse of notation and concept since N is not a finite number). The spectral irradiance at the point of our original interest can, to some extent, be thought (or should I say, represent) as $qN$.

Long story short, spectral irradiance represents the extent of energy entering a place.

Up to here, there is no requirement on what the direction of the energy entered must be.

Now, once we agree on "specific direction can be represented by specific differential solid angle $ds$", let's do the derivative:

$$\frac{dH}{ds}$$

Let me interpret this as follows: the amount of spectral irradiance contributed from the photons coming through the solid angle $ds$ is $dH$. And now, let there be many "solid angles" each exactly as large as $ds$, and each carries photons flow that contributes exactly $dH$ amount of spectral irradiance (think of those "solid angles" as "zones"), and those zones are adjacent, parallel to each other such that those microscopic zones form a macroscopic zone with its cross-sectional area equals 1. $\frac{dH}{ds}$ equals the amount of spectral irradiance contributed by this macro zone (we can think of it as a light pillar from a spotlight).

Now, I will tell you more: assume this light pillar is an incident light to the point of our interest (recall that there's one such point defined with spectral irradiance, now we assume this point is on a flat surface) such that it forms an angle $\theta$ with the normal of the flat surface. Then:

$$\frac{dH}{ds\cos{\theta}}$$

is the amount of spectral irradiance this light pillar would contributes to the flat surface if the direction of the light pillar was perpendicular to the flat surface.

Let's give this amount a name right away: it is the radiance at our point of interest. Let's denote it as $L$.

Radiance does not vary with distance because, when propagates through space, the light pillar does not lose energy until it hits anything.

Now, assume this light pillar is emitting out from our point of interest, and hits some other flat surface $F$ at some other point $B$ at an angle $\phi$ with the normal at $B$. Then $L\cos{\phi}$ gives the spectral irradiance this light pillar contributes to $F$.
