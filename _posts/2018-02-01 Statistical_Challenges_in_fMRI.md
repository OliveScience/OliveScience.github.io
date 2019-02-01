I recently read an journal article by Alex Huth on semantic maps in the
cerebral cortex. Huth applies Voxel-wise statistical modeling to predict
blood-oxygen-level-dependent (BOLD) to new natural stimuli generated
from reading specific text from The Moth Radio Hour. To get a better
idea of what voxel-wise statistical modeling entailed I dug deeper and
wound up finding Martin Lindquist's article on Statistical Analysis of
fMRI written back in 2008. Lindquist's does an excellent job
highlighting some of the challenges that fMRI faced in 2008, and, I
found it quite useful to put into context some of Huth's methodology. In
this post I will introduce a statistical distributin used to model
signal to noise in voxel data.

The super Basics of fMRI:
-------------------------

We start by inserting our subject within the space our instrument will
be generating a magnetic field. Imaging devices generate a uniform
magnetic field somewhere between 1.5 - 7.0 Tesla. The field aligns the
magnetization of H atoms in the brain via interaction between the
nuclear spin and the magnetic field. Following nuclear alignment, a
radio frequency pulse tips the vector of the magnetic moment in the H
atoms. The aggitated atoms then precess around the external magnetic
field direction. This induces a flux in a nearby receiver coil. We thus
can retrieve a current representing population of tipped precessing H
atoms. We have our magnetic resonance (MR) signal!

Our signal arriving at time *t* is coming over two channels, the real
and imaginary channels:
*s*(*t*)=*s*<sub>*r*</sub>(*t*)+*i**s*<sub>*i*</sub>(*t*) The error in
the signal at this point is normally distributed. The sources of error
are generally due to coil resistance and inductive losses in the sample.
[FFT](https://en.wikipedia.org/wiki/Fast_Fourier_transform) is a linear
operator, and after applying it to our signal our error remains normally
distributed. However, upon final reconstruction of our signal, we apply
a nonlinear transformation. This changes our error distribution
associated with the magnitude of the image, which in turn coerces our
error distribution to follow a Rician Distribution

Brief overview of the Rice Distribution
---------------------------------------

The [Rician
Distribution](https://en.wikipedia.org/wiki/Rice_distribution) is a two
parameter distribution, *σ* and *v**e**e*. I will be taking a dive into
the history and theory of this distribution in a future post. In the
equation below, *v**e**e* is represent by *A*.
$$P\_M\\left(M\\right)=\\frac M{\\sigma^2}exp(-{\\textstyle\\frac{(M^2+A^2)}{2\\sigma^2}})\\cdot I\_0^{(\\frac{A\\cdot M}{\\sigma^2})\\;}$$
 In the context of MRI our values are:  
*M*= our support. These are the values our distribution is defined for.
Where *M* ∈ \[0, ∞) Such that *M* encapsulates the magnitude of the
voxel in the presence of noise, aka normal voxel data.  
*A*= parameter of the Rician distribution that represents the magnitude
of the voxel in the absence of noise.  
*σ*=the standard deviation of the normal noise  
*I*<sub>0</sub>= a modified zeroth order [Bessel
function](https://en.wikipedia.org/wiki/Bessel_function)

We are particularly interested in the ratio between *A* and *σ*. This
ratio is the signal to noise ratio, SNR. Improving our SNR improves
approximations further in analysis. To increase the amount of signal to
noise recieved, we can fine tune our instrument such to improve the
quality of the signal recieved in the absence of noise, *A*. I will be
taking a plunged into those methodologies as well, but for now I will
end this post with a visualization of improving SNR. As we improve it,
our distribution of the magnitude of voxel becomes normal and less
varied. See below.

<iframe src="https://josepholiveira.shinyapps.io/Rician_distribution/" " width="100%" height="1000" frameborder="0" allowfullscreen="allowfullscreen">
</iframe>
