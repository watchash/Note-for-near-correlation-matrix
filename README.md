### Introduction

This document briefly describes how to use
[sdpt3r](https://cran.r-project.org/web/packages/sdpt3r/index.html)
package in R to construct a nearest correlation matrix of a given
approximate correlation matrix for sample size simulation.  

A valid correlation matrix must conform the following properties:  

-   **Diagonal elements all equal 1**  
-   **Non-diagonal elements in the closed interval \[-1, 1\]**  
-   **Symmetric**  
-   **Positive semi-definite**  

Generally, an empirical correlation matrix from previous studies can be
used in simulation, but manual intervention to the matrix may also be
performed for scenario analysis. In these cases, the fourth property may
be hard to satisfy through visual inspection.  

This problem occurs in several important areas of finance and risk
management, and can be solved using standard non-linear optimization
procedure. The *sdpt3r* package is a semi-definite quadratic linear
programming solver in R, which incorporate a special function
`nearcorr()` to deal with this problem.

### Optimization framework in `nearcorr()`

The solver tries to find the nearest valid correlation matrix to the
input matrix. The Forbenius form is used as the error function for
optimization process: the solution matrix will have the smallest sum of
squares of element difference compared with the input matrix.

### Example

``` r
library(sdpt3r)
```

Here is an empirical correlation matrix generated from historical data.
The Vx consists of 21 serotypes, resulting in a 21x21 correlation
matrix. The matrix is stored in a R object called `empirical`.  

    ##       S1   S2   S3   S4   S5   S6   S7   S8   S9  S10  S11  S12  S13  S14  S15  S16  S17  S18  S19  S20  S21
    ## S1  1.00 0.55 0.55 0.82 0.69 0.55 0.56 0.69 0.43 0.55 0.45 0.51 0.57 0.65 0.65 0.56 0.74 0.66 0.63 0.76 0.62
    ## S2  0.55 1.00 0.56 0.50 0.51 0.30 0.65 0.46 0.48 0.56 0.34 0.47 0.46 0.75 0.59 0.44 0.52 0.60 0.44 0.66 0.56
    ## S3  0.55 0.56 1.00 0.41 0.63 0.41 0.70 0.62 0.46 0.63 0.53 0.64 0.68 0.66 0.66 0.49 0.66 0.65 0.36 0.51 0.42
    ## S4  0.82 0.50 0.41 1.00 0.58 0.52 0.53 0.62 0.29 0.43 0.37 0.40 0.43 0.58 0.54 0.52 0.58 0.64 0.69 0.81 0.64
    ## S5  0.69 0.51 0.63 0.58 1.00 0.66 0.66 0.72 0.41 0.60 0.52 0.66 0.67 0.58 0.68 0.54 0.74 0.63 0.44 0.60 0.53
    ## S6  0.55 0.30 0.41 0.52 0.66 1.00 0.54 0.56 0.39 0.33 0.59 0.48 0.52 0.32 0.55 0.51 0.69 0.53 0.40 0.52 0.57
    ## S7  0.56 0.65 0.70 0.53 0.66 0.54 1.00 0.56 0.50 0.52 0.55 0.55 0.61 0.69 0.64 0.51 0.68 0.69 0.52 0.72 0.53
    ## S8  0.69 0.46 0.62 0.62 0.72 0.56 0.56 1.00 0.38 0.56 0.50 0.64 0.59 0.60 0.64 0.52 0.73 0.61 0.41 0.56 0.56
    ## S9  0.43 0.48 0.46 0.29 0.41 0.39 0.50 0.38 1.00 0.27 0.43 0.43 0.46 0.45 0.39 0.23 0.41 0.46 0.37 0.41 0.40
    ## S10 0.55 0.56 0.63 0.43 0.60 0.33 0.52 0.56 0.27 1.00 0.31 0.51 0.55 0.70 0.61 0.52 0.56 0.57 0.46 0.51 0.42
    ## S11 0.45 0.34 0.53 0.37 0.52 0.59 0.55 0.50 0.43 0.31 1.00 0.48 0.46 0.41 0.53 0.35 0.59 0.41 0.40 0.47 0.41
    ## S12 0.51 0.47 0.64 0.40 0.66 0.48 0.55 0.64 0.43 0.51 0.48 1.00 0.61 0.49 0.47 0.30 0.65 0.49 0.33 0.46 0.47
    ## S13 0.57 0.46 0.68 0.43 0.67 0.52 0.61 0.59 0.46 0.55 0.46 0.61 1.00 0.51 0.64 0.45 0.65 0.67 0.39 0.52 0.45
    ## S14 0.65 0.75 0.66 0.58 0.58 0.32 0.69 0.60 0.45 0.70 0.41 0.49 0.51 1.00 0.69 0.54 0.63 0.62 0.55 0.70 0.57
    ## S15 0.65 0.59 0.66 0.54 0.68 0.55 0.64 0.64 0.39 0.61 0.53 0.47 0.64 0.69 1.00 0.53 0.71 0.67 0.50 0.66 0.54
    ## S16 0.56 0.44 0.49 0.52 0.54 0.51 0.51 0.52 0.23 0.52 0.35 0.30 0.45 0.54 0.53 1.00 0.57 0.54 0.40 0.55 0.59
    ## S17 0.74 0.52 0.66 0.58 0.74 0.69 0.68 0.73 0.41 0.56 0.59 0.65 0.65 0.63 0.71 0.57 1.00 0.65 0.50 0.64 0.56
    ## S18 0.66 0.60 0.65 0.64 0.63 0.53 0.69 0.61 0.46 0.57 0.41 0.49 0.67 0.62 0.67 0.54 0.65 1.00 0.59 0.70 0.61
    ## S19 0.63 0.44 0.36 0.69 0.44 0.40 0.52 0.41 0.37 0.46 0.40 0.33 0.39 0.55 0.50 0.40 0.50 0.59 1.00 0.72 0.47
    ## S20 0.76 0.66 0.51 0.81 0.60 0.52 0.72 0.56 0.41 0.51 0.47 0.46 0.52 0.70 0.66 0.55 0.64 0.70 0.72 1.00 0.61
    ## S21 0.62 0.56 0.42 0.64 0.53 0.57 0.53 0.56 0.40 0.42 0.41 0.47 0.45 0.57 0.54 0.59 0.56 0.61 0.47 0.61 1.00

We may want to make some adjustment to the empirical matrix to perform
different scenario analysis. For example, we want to decrease the
correlation between serotypes to represent a conservative scenario.  

-   For correlation coefficients \>= 0.6, we would make them equal to
    0.6  
-   For correlation coefficients between 0.3 and 0.6, we would make them
    equal to 0.3  

The target adjusted matrix is stored in `scenario` as shown below:  

    ##      S1  S2  S3   S4  S5  S6  S7  S8   S9  S10 S11 S12 S13 S14 S15  S16 S17 S18 S19 S20 S21
    ## S1  1.0 0.3 0.3 0.60 0.6 0.3 0.3 0.6 0.30 0.30 0.3 0.3 0.3 0.6 0.6 0.30 0.6 0.6 0.6 0.6 0.6
    ## S2  0.3 1.0 0.3 0.30 0.3 0.3 0.6 0.3 0.30 0.30 0.3 0.3 0.3 0.6 0.3 0.30 0.3 0.6 0.3 0.6 0.3
    ## S3  0.3 0.3 1.0 0.30 0.6 0.3 0.6 0.6 0.30 0.60 0.3 0.6 0.6 0.6 0.6 0.30 0.6 0.6 0.3 0.3 0.3
    ## S4  0.6 0.3 0.3 1.00 0.3 0.3 0.3 0.6 0.29 0.30 0.3 0.3 0.3 0.3 0.3 0.30 0.3 0.6 0.6 0.6 0.6
    ## S5  0.6 0.3 0.6 0.30 1.0 0.6 0.6 0.6 0.30 0.60 0.3 0.6 0.6 0.3 0.6 0.30 0.6 0.6 0.3 0.6 0.3
    ## S6  0.3 0.3 0.3 0.30 0.6 1.0 0.3 0.3 0.30 0.30 0.3 0.3 0.3 0.3 0.3 0.30 0.6 0.3 0.3 0.3 0.3
    ## S7  0.3 0.6 0.6 0.30 0.6 0.3 1.0 0.3 0.30 0.30 0.3 0.3 0.6 0.6 0.6 0.30 0.6 0.6 0.3 0.6 0.3
    ## S8  0.6 0.3 0.6 0.60 0.6 0.3 0.3 1.0 0.30 0.30 0.3 0.6 0.3 0.6 0.6 0.30 0.6 0.6 0.3 0.3 0.3
    ## S9  0.3 0.3 0.3 0.29 0.3 0.3 0.3 0.3 1.00 0.27 0.3 0.3 0.3 0.3 0.3 0.23 0.3 0.3 0.3 0.3 0.3
    ## S10 0.3 0.3 0.6 0.30 0.6 0.3 0.3 0.3 0.27 1.00 0.3 0.3 0.3 0.6 0.6 0.30 0.3 0.3 0.3 0.3 0.3
    ## S11 0.3 0.3 0.3 0.30 0.3 0.3 0.3 0.3 0.30 0.30 1.0 0.3 0.3 0.3 0.3 0.30 0.3 0.3 0.3 0.3 0.3
    ## S12 0.3 0.3 0.6 0.30 0.6 0.3 0.3 0.6 0.30 0.30 0.3 1.0 0.6 0.3 0.3 0.30 0.6 0.3 0.3 0.3 0.3
    ## S13 0.3 0.3 0.6 0.30 0.6 0.3 0.6 0.3 0.30 0.30 0.3 0.6 1.0 0.3 0.6 0.30 0.6 0.6 0.3 0.3 0.3
    ## S14 0.6 0.6 0.6 0.30 0.3 0.3 0.6 0.6 0.30 0.60 0.3 0.3 0.3 1.0 0.6 0.30 0.6 0.6 0.3 0.6 0.3
    ## S15 0.6 0.3 0.6 0.30 0.6 0.3 0.6 0.6 0.30 0.60 0.3 0.3 0.6 0.6 1.0 0.30 0.6 0.6 0.3 0.6 0.3
    ## S16 0.3 0.3 0.3 0.30 0.3 0.3 0.3 0.3 0.23 0.30 0.3 0.3 0.3 0.3 0.3 1.00 0.3 0.3 0.3 0.3 0.3
    ## S17 0.6 0.3 0.6 0.30 0.6 0.6 0.6 0.6 0.30 0.30 0.3 0.6 0.6 0.6 0.6 0.30 1.0 0.6 0.3 0.6 0.3
    ## S18 0.6 0.6 0.6 0.60 0.6 0.3 0.6 0.6 0.30 0.30 0.3 0.3 0.6 0.6 0.6 0.30 0.6 1.0 0.3 0.6 0.6
    ## S19 0.6 0.3 0.3 0.60 0.3 0.3 0.3 0.3 0.30 0.30 0.3 0.3 0.3 0.3 0.3 0.30 0.3 0.3 1.0 0.6 0.3
    ## S20 0.6 0.6 0.3 0.60 0.6 0.3 0.6 0.3 0.30 0.30 0.3 0.3 0.3 0.6 0.6 0.30 0.6 0.6 0.6 1.0 0.6
    ## S21 0.6 0.3 0.3 0.60 0.3 0.3 0.3 0.3 0.30 0.30 0.3 0.3 0.3 0.3 0.3 0.30 0.3 0.6 0.3 0.6 1.0

Generally, `eigen()` function from base R can be used to check if the
adjusted matrix is positive semi-definite, which is equivalent to say
that all eigenvalues are non-negative.

``` r
eigen(scenario,only.values = T)$values
```

    ##  [1]  9.22467990  1.76906325  1.32584718  1.14157531  1.05709670  0.93040190  0.84997330  0.77311107  0.76784523  0.68708272  0.66300119
    ## [12]  0.53760507  0.49866969  0.37034589  0.29724907  0.19239673  0.18859825  0.05150263  0.01428275 -0.04097909 -0.29934875

The result shows that the new matrix has two negative eigenvalues, thus
it is not positive semi-definite and can not be considered a valid
correlation matrix.  

To get the nearest correlation matrix, simply use `nearcorr()` function.
The calculation may take some time if the number of serotypes are large.

``` r
new_correlation <- sdpt3r::nearcorr(scenario)
```

The output of this function is a list project, you can get the matrix
and the error value in `$X[[1]]` and `$pobj` elements, respectively.

``` r
new_correlation$X[[1]]
```

    ## 21 x 21 Matrix of class "dgeMatrix"
    ##            [,1]      [,2]      [,3]      [,4]      [,5]      [,6]      [,7]      [,8]      [,9]     [,10]     [,11]     [,12]     [,13]
    ##  [1,] 1.0000000 0.2880133 0.3030486 0.5795289 0.5534098 0.3186271 0.3180102 0.6222118 0.2995108 0.3257202 0.2990307 0.3089816 0.3100159
    ##  [2,] 0.2880133 1.0000000 0.2997304 0.3079442 0.3206432 0.2912866 0.5910936 0.2936333 0.3001657 0.2892048 0.3003790 0.2926036 0.2986170
    ##  [3,] 0.3030486 0.2997304 1.0000000 0.2967487 0.5948824 0.3016302 0.6010632 0.6056593 0.2999005 0.6033460 0.2998491 0.5977528 0.6040046
    ##  [4,] 0.5795289 0.3079442 0.2967487 1.0000000 0.3364896 0.2863139 0.2870431 0.5764701 0.2904866 0.2784879 0.3008685 0.2984154 0.2874001
    ##  [5,] 0.5534098 0.3206432 0.5948824 0.3364896 1.0000000 0.5678636 0.5687640 0.5595078 0.3008882 0.5546709 0.3017290 0.5857153 0.5814266
    ##  [6,] 0.3186271 0.2912866 0.3016302 0.2863139 0.5678636 1.0000000 0.3128113 0.3139381 0.2996823 0.3175143 0.2993507 0.3075786 0.3056671
    ##  [7,] 0.3180102 0.5910936 0.6010632 0.2870431 0.5687640 0.3128113 1.0000000 0.3123331 0.2997073 0.3168558 0.2993838 0.3086404 0.6043532
    ##  [8,] 0.6222118 0.2936333 0.6056593 0.5764701 0.5595078 0.3139381 0.3123331 1.0000000 0.2993192 0.3256363 0.2988919 0.5933626 0.3214062
    ##  [9,] 0.2995108 0.3001657 0.2999005 0.2904866 0.3008882 0.2996823 0.2997073 0.2993192 1.0000000 0.2694538 0.3000230 0.3000603 0.2996064
    ## [10,] 0.3257202 0.2892048 0.6033460 0.2784879 0.5546709 0.3175143 0.3168558 0.3256363 0.2694538 1.0000000 0.2989816 0.3052657 0.3127522
    ## [11,] 0.2990307 0.3003790 0.2998491 0.3008685 0.3017290 0.2993507 0.2993838 0.2988919 0.3000230 0.2989816 1.0000000 0.2999177 0.2994096
    ## [12,] 0.3089816 0.2926036 0.5977528 0.2984154 0.5857153 0.3075786 0.3086404 0.5933626 0.3000603 0.3052657 0.2999177 1.0000000 0.5918595
    ## [13,] 0.3100159 0.2986170 0.6040046 0.2874001 0.5814266 0.3056671 0.6043532 0.3214062 0.2996064 0.3127522 0.2994096 0.5918595 1.0000000
    ## [14,] 0.5707717 0.6120186 0.5959080 0.3243820 0.3512774 0.2802164 0.5811176 0.5706510 0.3006206 0.5706125 0.3011534 0.2944966 0.2851472
    ## [15,] 0.5822491 0.3054721 0.5957991 0.3178361 0.6319504 0.2887202 0.5899123 0.5742452 0.3005049 0.5801998 0.3008402 0.3035399 0.5845900
    ## [16,] 0.2989990 0.3003887 0.2998418 0.3009024 0.3017875 0.2993305 0.2993655 0.2988421 0.2300240 0.2989445 0.3000427 0.2999261 0.2993798
    ## [17,] 0.5803851 0.3092938 0.5983777 0.3140725 0.6336847 0.5862395 0.5864452 0.5860962 0.3003209 0.2818031 0.3006678 0.5914362 0.5945993
    ## [18,] 0.6139823 0.5908201 0.5986507 0.5932886 0.5764584 0.3108740 0.6117536 0.6004800 0.2999164 0.3110052 0.2996752 0.3148505 0.5958922
    ## [19,] 0.5871063 0.3053056 0.2981245 0.6101147 0.3221603 0.2912970 0.2917797 0.2882201 0.3002500 0.2875703 0.3004778 0.2969493 0.2941390
    ## [20,] 0.6340920 0.5871465 0.3059107 0.5700044 0.5399447 0.3225918 0.6210782 0.3386105 0.2992085 0.3352099 0.2985838 0.3023902 0.3209987
    ## [21,] 0.5826594 0.3085032 0.2988234 0.6118625 0.3295779 0.2877238 0.2878185 0.2891398 0.3002598 0.2843054 0.3005634 0.2912501 0.2963385
    ##           [,14]     [,15]     [,16]     [,17]     [,18]     [,19]     [,20]     [,21]
    ##  [1,] 0.5707717 0.5822491 0.2989990 0.5803851 0.6139823 0.5871063 0.6340920 0.5826594
    ##  [2,] 0.6120186 0.3054721 0.3003887 0.3092938 0.5908201 0.3053056 0.5871465 0.3085032
    ##  [3,] 0.5959080 0.5957991 0.2998418 0.5983777 0.5986507 0.2981245 0.3059107 0.2988234
    ##  [4,] 0.3243820 0.3178361 0.3009024 0.3140725 0.5932886 0.6101147 0.5700044 0.6118625
    ##  [5,] 0.3512774 0.6319504 0.3017875 0.6336847 0.5764584 0.3221603 0.5399447 0.3295779
    ##  [6,] 0.2802164 0.2887202 0.2993305 0.5862395 0.3108740 0.2912970 0.3225918 0.2877238
    ##  [7,] 0.5811176 0.5899123 0.2993655 0.5864452 0.6117536 0.2917797 0.6210782 0.2878185
    ##  [8,] 0.5706510 0.5742452 0.2988421 0.5860962 0.6004800 0.2882201 0.3386105 0.2891398
    ##  [9,] 0.3006206 0.3005049 0.2300240 0.3003209 0.2999164 0.3002500 0.2992085 0.3002598
    ## [10,] 0.5706125 0.5801998 0.2989445 0.2818031 0.3110052 0.2875703 0.3352099 0.2843054
    ## [11,] 0.3011534 0.3008402 0.3000427 0.3006678 0.2996752 0.3004778 0.2985838 0.3005634
    ## [12,] 0.2944966 0.3035399 0.2999261 0.5914362 0.3148505 0.2969493 0.3023902 0.2912501
    ## [13,] 0.2851472 0.5845900 0.2993798 0.5945993 0.5958922 0.2941390 0.3209987 0.2963385
    ## [14,] 1.0000000 0.6226937 0.3011954 0.6205673 0.5880453 0.3142351 0.5598144 0.3177235
    ## [15,] 0.6226937 1.0000000 0.3008767 0.6113742 0.5983033 0.3092931 0.5705590 0.3091067
    ## [16,] 0.3011954 0.3008767 1.0000000 0.3006879 0.2996732 0.3004943 0.2985290 0.3005792
    ## [17,] 0.6205673 0.6113742 0.3006879 1.0000000 0.5881269 0.3091291 0.5766906 0.3130688
    ## [18,] 0.5880453 0.5983033 0.2996732 0.5881269 1.0000000 0.2944398 0.6105586 0.5885854
    ## [19,] 0.3142351 0.3092931 0.3004943 0.3091291 0.2944398 1.0000000 0.5829659 0.3079735
    ## [20,] 0.5598144 0.5705590 0.2985290 0.5766906 0.6105586 0.5829659 1.0000000 0.5802943
    ## [21,] 0.3177235 0.3091067 0.3005792 0.3130688 0.5885854 0.3079735 0.5802943 1.0000000

``` r
new_correlation$pobj
```

    ## [1] 0.3219091

``` r
#The error term to be minimized is the Forbenius norm (type="F") of the subtraction of two matrices
norm(scenario - new_correlation$X[[1]],"F")
```

    ## [1] 0.3219091

We can see that the summation of squared changes in all 420 non-diagonal
elements is `new_correlation$pobj` = 0.3219091, which may seem
acceptable to approximate our target correlation matrix.  

The maximum absolute change for a single matrix element is 0.0600553,
the value might increase for large matrices, and the Stats can decide if
it’s acceptable to use the resulting matrix as a valid approximation for
scenario analysis.

Before exporting the result, please note that the diagonal elements in
the matrix do not exactly equal 1, we may round them to certain number
of decimal places and re-check if the matrix is positive
semi-definite.  

Here we round the result to 7 decimal places, which should have adequate
precision.

``` r
output_matrix <- new_correlation$X[[1]]
output_matrix <- round(output_matrix,7)
eigen(output_matrix,only.values = T)$value
```

    ##  [1] 9.202332e+00 1.749470e+00 1.306335e+00 1.128897e+00 1.036125e+00 9.089967e-01 8.282201e-01 7.699814e-01 7.590444e-01 6.814078e-01
    ## [11] 6.542532e-01 5.179706e-01 4.640698e-01 3.506477e-01 2.736835e-01 1.735651e-01 1.686135e-01 2.638774e-02 9.971970e-08 7.836802e-08
    ## [21] 1.057650e-09

The eigenvalues are all non-negative. We are fine to export the
result.  

A convenient approach is to write the result to system clipboard and
directly paste it to SAS file. The `write_clip()` function from *clipr*
package will be helpful. Here we add a new column of “CORR” for
convenience.

``` r
clipr::write_clip(cbind("CORR",as.data.frame(as.matrix(output_matrix))))
```

Just delete the header row, and the data should fit the datalines
statement in SAS data step.  
