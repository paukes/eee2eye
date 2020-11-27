
# eee2eye <img src="man/figures/eee2eye_logo.png" width="120" align="right" />

## Calculating E:I Values from δ<sup>18</sup>O-H<sub>2</sub>O (‰)

Please see ‘EI-Calculations.pdf’ in the ‘man’ folder for a step-by-step
description of the Isotope Mass Balance used to calculate E:I.

## E:I Function

Briefly, this function calculates E:I ratios, based on
δ18O-H<sub>2</sub>O. Environmental conditions (i.e. evaporation rate,
humidity, temperatures, etc.) are set for the sub-arctic around
Yellowknife, NT.

The function is based on a table with the following input parameters per
sample:  
\- dL (‰) = -11.77, steady-state lake isotope value (measured value from
field)  
\- dI (‰) = -20.7, source water, likely precipitation (value from Gibson
2001 and GNIP 1999)  
\- dP (‰) = -23, average value during evaporation season (signal of
rain)  
\- temp (C) = 14.3, average temp. on lake (from Gibson & Reid, 2010)  
\- humid (dec) = 0.68, relative humidity (from Gibson & Reid, 2010)  
\- k = 0.7, estimated for our area

### ‘eee2eye’ Package

``` r
eee2eye <- function(df, dL, dI, dP, temp, humid, k){

  #check if using values or specific columns for each parameter
  dL_input <- if(is.numeric(dL) == TRUE) {
    dL
  } else {
    df[[dL]]
  }

  dI_input <- if(is.numeric(dI) == TRUE) {
    dI
  } else {
    df[[dI]]
  }

  dP_input <- if(is.numeric(dP) == TRUE) {
    dP
  } else {
    df[[dP]]
  }

  temp_input <- if(is.numeric(temp) == TRUE) {
    temp
  } else {
    df[[temp]]
  }

  humid_input <- if(is.numeric(humid) == TRUE) {
    humid
  } else {
    df[[humid]]
  }

  k_input <- if(is.numeric(k) == TRUE) {
    k
  } else {
    df[[k]]
  }

  #Calculate isotope fractionation and separation factors:
  alpha_plus <- exp((-7.685/(10^3)) + (6.7123/(273.15 + temp_input)) - (1666.4/((273.15 + temp_input)^2)) + (350410/((273.15 + temp_input)^3)))
  e_plus_permille <- (alpha_plus - 1) * 1000

  theta <- 1
  Ck_permille <- 14.2
  ek_permille <- theta * Ck_permille * (1 - humid_input)


  #Calculate dA
  dA_permille <- (dP_input - (k_input * e_plus_permille)) / (1 + ((10^-3) * k_input * e_plus_permille))

  #Calculate temporal enrichment slope (m)
  m <- (humid_input - (10^-3) * (ek_permille + (e_plus_permille / alpha_plus))) / (1 - humid_input + ((10^-3) * ek_permille))

  #Calculate limiting isotopic composition
  dstar_permille <- ((humid_input * dA_permille) + ek_permille + (e_plus_permille/alpha_plus)) / (humid_input - ((10^-3) * (ek_permille + (e_plus_permille/alpha_plus))))

  #Calculate our E:I based on lake sig
  df$E.I <- (dL_input - dI_input) / (m * (dstar_permille - dL_input))

  return(df)

}
```

### Example

To add E:I ratios to your dataframe, essentially run:

``` r
#example database
ei_input <- data.frame(dL_permille = c(-11.77, -15.67, -18.23),
                       dI_permille = c(-20.7, -18.2, -20.2),
                       dP_permille = c(-23, -28, -32), 
                       temp_C = c(14.3, 12.1, 8.9), 
                       h_dec = c(0.68, 0.71, 0.58), 
                       k = c(0.7, 0.72, 0.65))

#adding calculated E:I
ei_input <- eee2eye(ei_input, 'dL_permille', 'dI_permille', 'dP_permille', 'temp_C', 'h_dec', 'k')
```

or with numerical inputs:

``` r
ei_input <- eee2eye(ei_input, 'dL_permille', -20.7, 'dP_permille', 14.3, 0.68, 0.7)
```
