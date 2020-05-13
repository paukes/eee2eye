README
================
PJKA
13/05/2020

# Calculating E:I Values from δ<sup>18</sup>O-H<sub>2</sub>O (‰)

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
\- evap (m/yr) = 0.397, annual evap. rate of area (from Gibson & Reid
2010)  
\- k = 0.7, estimated for our area

``` r
eee2eye <- function(df, dL, dI, dP, temp, humid, evap, k){
  
  #Calculate isotope fractionation and separation factors:
  alpha_plus <- exp((-7.685/(10^3)) + (6.7123/(273.15 + df[[temp]])) - (1666.4/((273.15 + df[[temp]])^2)) + (350410/((273.15 + df[[temp]])^3)))
  e_plus_permille <- (alpha_plus - 1) * 1000
 
  theta <- 1
  Ck_permille <- 14.2 
  ek_permille <- theta * Ck_permille * (1 - df[[humid]])
  
  #Calculate dA
  dA_permille <- (df[[dP]] - (df[[k]] * e_plus_permille)) / (1 + ((10^-3) * df[[k]] * e_plus_permille))

  #Calculate temporal enrichment slope (m)
  m <- (df[[humid]] - (10^-3) * (ek_permille + (e_plus_permille / alpha_plus))) / (1 - df[[humid]] + ((10^-3) * ek_permille))

  #Calculate limiting isotopic composition
  dstar_permille <- ((df[[humid]] * dA_permille) + ek_permille + (e_plus_permille/alpha_plus)) / (df[[humid]] - ((10^-3) * (ek_permille + (e_plus_permille/alpha_plus))))
  
  #Calculate our E:I based on lake sig
  df$E.I <- (df[[dL]] - df[[dI]]) / (m * (dstar_permille - df[[dL]]))

  return(df)
  
}

ei_input <- read.csv("F:/PostDoc/Publications/Manuscript 1 - Changes Due to Precip/SAMMS DOM and Precip/ei_input.csv")

ei_input <- eee2eye(ei_input, 'd18O_H2O_permille', 'dI_permille', 'dP_permille', 'temp_C', 'h_dec', 'evap_m.yr', 'k')
```
