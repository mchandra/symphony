#*symphony*

##Overview

*symphony* calculates synchrotron emissivities and absorptivities, polarized in the Stokes basis {I, Q, U, V}, for any arbitrary gyrotropic momentum space distribution function.  Three distribution functions are built in: a relativistic thermal (Maxwell-Juettner) distribution, a nonthermal power-law distribution, and a kappa distribution.  

##Features

* `C` code to calculate synchrotron emissivities via the `j_nu()` function and absorptivities via the `alpha_nu()` function.
* `C` code to evaluate approximate fitting function values for the emissivity and absorptivity, via the `j_nu_fit()` and `alpha_nu_fit()` functions, respectively.
* CMake configure system, which helps during the build process to find all necessary libraries and files.
* `Python` interface for `j_nu()`, `alpha_nu()`, `j_nu_fit()`, and `alpha_nu_fit()`.
  * This combines the speed of `C` when evaluating emissivities and absorptivities with `Python`'s user-friendly syntax.  It also allows for interfacing with larger `Python` codes.

##How to use *symphony*

###To download and build *symphony*:
 1. Clone *symphony* from github.  Navigate into the *symphony* folder, and note that it contains a folder named "src/"; create a folder named "build" and navigate into it.
 2. Type "cmake" followed by the location of the "src/" folder.  Altogether, this line should look something like: "cmake /location/to/symphony/src"
 3. Type "make"

###To use *symphony*'s `Python` interface:
 1. Navigate to the "build/" folder created in step 1., above.  Open `Python` in the command line or by writing a ".py" file.
 2. Import *symphony* by typing `import symphonyPy`.  
  * This allows one to call 4 functions: `j_nu_py()`, `alpha_nu_py()`, `j_nu_fit_py()`, and `alpha_nu_fit_py()`.  
  * The first two provide calculated values of the emissivity and absorptivity for the input parameters, and the latter two provide the corresponding approximate fitting formula results.
 3. The arguments of these functions can be found by accessing the associated docstrings.  This can be done in the `Python` command line using the following: 
```
import symphonyPy
symphonyPy.j_nu_py?
```

###Arguments for Emissivity and Absorptivity Functions
* Arguments for all emissivity and absorptivity functions in both `C` and `Python` take the same arguments and ouput a double.  The arguments are: 
```
j_nu(nu, magnetic_field, electron_density, observer_angle, distribution, polarization, 
     theta_e, power_law_p, gamma_min, gamma_max, gamma_cutoff, kappa, kappa_width)
```
Sample values:
```
j_nu_py(): j_nu_py(230e9, 30, 1, 1.047, symphonyPy.MAXWELL_JUETTNER, symphonyPy.STOKES_I,
                   10, 2.5, 1, 1000, 1e10, 3.5, 10)
```
* Note: All parameters with units are in CGS.
* Note: In `C`, the keys `symphonyPy.MAXWELL_JUETTNER` and `symphonyPy.STOKES_I` are members of a struct called `params`.  They can be used with: `params->MAXWELL_JUETTNER` and `params->STOKES_I`.

###To add a new gyrotropic distribution function:
1. Navigate into the "src/" folder within *symphony*.  Enter "params.c" and go to the comment that says "Keys for the distributions"; add a new key equal to an integer that is not used for the existing three distribution functions (for example, 4).  Altogether, this should look like: `params->NEW_DIST = 4`.
2. Back in the "src/" folder, create a folder with the name of the new distribution function.  Enter that folder.
  * Note: there are three existing examples to follow for this procedure: "maxwell_juettner/", "power_law/", and "kappa/".
3. Make a file named new_dist.c (where new_dist is the name of the new distribution function).
4. Inside, #include "new_dist.h", and below write the function for the new distribution function.  It should return a `double` and take input parameters `double gamma` (the electron Lorentz factor) and `struct parameters * params`.  This struct contains all parameters for the calculation; additional parameters can be added as necessary in the files "params.c" and "params.h" in the src/ folder.
  * If this distribution function needs to be normalized, follow the pattern in "power_law.c" in the "power_law/" folder.  There is an unnormalized function `power_law_to_be_normalized` with arguments `double gamma` and `void * paramsInput`, where the latter is a void pointer to the previously mentioned struct of parameters.  It has to be a void pointer because this function will be integrated over by GSL's QAGIU integrator, which requires a function of this form.
  * Write another function `new_dist()`, which will be the normalized distribution function.  This should be equal to the unnormalized distribution multiplied by `1/normalize_f()`.
  * Go back into the "src/" folder and enter "integrate.c".  In the function `normalize_f()`, there is an `if` statement which chooses the function to be normalized.  Following the pattern set for the other two distributions, add new_dist using the key set in 1.
5. Back in the folder "new_dist/", add a file named "new_dist.h" (where "new_dist" is the name of the distribution function).  Inside this file, copy the format of "power_law.h" or "kappa.h".
6. Return to the folder "new_dist/" and make a file called "CMakeLists.txt".  Inside this file, write one line: `add_library(new_dist new_dist.c new_dist.h)`; if there are additional files (for instance, if you provide fitting formulae in a file named "new_dist_fits.c") then include that as well.
7. Navigate back to "src/" and enter "CMakelists.txt".  Add `add_subdirectory(new_dist)` below `add_subdirectory(kappa)`.  Find the two instances of `target_link_libraries()` and add `new_dist` as an argument to both of them.
8. Add "#include "new_dist/new_dist.h"" to the file calc.h
9. Add a line for new_dist in the function `distribution_function()` in the file calc.c following the pattern set for the other three distribution functions.
