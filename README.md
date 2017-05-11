# Variances-from-Random-Regression

# Description
Calculates covariate specific variances and their SE on the basis of random regression estimates
Approach described by Fischer et al. 2004, Genet. Sel. Evol. 36:363–369
This function plots variances and approximate 95% CI based on double the SE. The function currently works only with RR of first-order polynomials (a + bX). 
Functions accepts three different inputs, but different additional information is required for each (see Arguments)
1. asreml-R object
2. name of the .vvp file generated by stand-alone Asreml
3. user specified information

# Usage
```
calc.varRR(vvp.name, phi)
calc.varRR(vvp.name, phi, K)
calc.varRR(vpp.name, phi, K, positions)
```

# Arguments 

vvp.name - either (1) asreml-r object of class "asreml" as produced by running the function asreml with random = ~us(pol(X,1)):SUBJECT **and** with the residual variances modelled explicitly, e.g. rcov=~idv(units) or equivalent. When the residual variances are not modelled explicitly, the average information will be on the scale of the "gammas". By specifying residual variances either as homogeneous (above) or heterogeneous the AI will be on the variance component scale, as assumedby calc.varRR (2) the name (and possibly path) of the .vvp file generated by stand-alone AsReml in which case needed is also a vector "positions" indicating the position of the relevant vvp elements. (3) The matrix containing the variances (in diagonal) of the REML-estimated variances, and the covariances associated with these variances, as estimated for each of the Random Regression covariance elements (slope, covariance between slope and elevation and elevation) provided as a 3x3 matrix. 

phi - the covariate along for which to calculate the variances. When asreml input is provided (options (1) and (2) for vvp.name, then phi is the *number* of node points. AsReml will have the minimal value of the covariate as -1 and the mean of the set of observed covariate values as 0. Thus, if your covariate has values 0, 1, 2 (each of which may occur different times in your data); AsReml will scale these to -1, 0, 1. For this reason, the variances and uncertainty are for asreml input always calculated over this interval. Other softare will not scale or scale  differently. lme in R, for example, does not change the scaling. In those cases, phi can be provided as vector containing the node points, which which *must* be scaled in the same way as in the analysis. Note that your software may internally scale this. To check internal scaling, you can multiply your covariate X by some factor which - in case the software does not scale - will produce a different estimate of the variance in slope (if X2 = X+X, the slope variance will be four times larger 

K - the covariance matrix of random effect for elevation (intercept) and slope. These random effects are here assumed to be polynomials (a + bX) leading to an estimate of the variance in intercept / elevation (a) and variance in slope (b) and a covariance between these which are provided in matrix form as K. Not needed if vvp.name is an asreml-r object. the elemants of K are in stand-alone AsReml obtained from the ".asr" file

positions - AsReml will generate a ".vvp" file. AsReml counts all estimated covariances in the order as reported in the ".asr" file. For example, the Random Regression variance in intercept, covariance between intercept and slope and variance in slope are the 3rd, 4th and 5th covariances reported, respectively. In that case, the relevant part of the vvp matrix concerns these positions. You can either extract these manually from the vvp file (example 1 below), or pass the whole file to calc.varRR and specify the positions (example 2 below).

# Use
Copy the script directly into your own script.
Alternatively, save it as a file to your working directory or other directory (in which case the path to that directory needs to be included). In the example below assumed to be saved in the work directory as a file called "calc_varRR.R"
```
calc.varRR<-dget(“calc_varRR.R”)
```

# example 1 (information specified by user - produces increasing variance as function of covariate) 
```
K=matrix(c(5,1,1,1),2,2)
phi=seq(-1,1,by=(1/3))
vvp=matrix(c(
0.160930E-01, -0.353929E-02, 0.778355E-02,
-0.353929E-02,  0.779459E-03, -0.171652E-02  ,
0.778355E-02, -0.171652E-02,  0.378527E-02),3,3)
calc.varRR(vvp,phi,K)
```

# example 2 (as above, but using a vvp file as input with the positions indicating where the RR elements are)
```
## Not run:
vvp.filename="C:\\vvp_example.txt"
K=matrix(c(5,1,1,1),2,2)
phi=30  #use of asreml requires giving  the number of nodes between -1 and 0 
calc.varRR(vvp.filename, phi, K, positions=c(3,5))
```

# example 3: asreml-R
```
## Not run:
require(asreml)
#assume a data.frame called "df" with repeated measures of Y taken on SUBJECT at various values of X
m.RR<-asreml(fixed= ~ 1 + X, random=~ us(pol(X,1)):SUBJECT, rcov=~idv(units), data=df)
calc.varRR(m.RR,30)
#how did asreml scale? 
scaling<-min(unique(df$X))-mean(unique(df$X))
#scaling times the "cov.value" produced by calc.varRR will provide you the covariate on the data scale
```
