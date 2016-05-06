# A Stargazer Cheatsheet
Updated `r format(Sys.time(), '%d %B, %Y')`  

## Dataset: dplyr and nycflights13

Setting up a dataset for this cheatsheet allows me to spotlight two recent
R packages created by [Hadley Wickham](http://had.co.nz/). The first, 
[dplyr](https://github.com/hadley/dplyr), is a set of new tools for data 
manipulation. Using `dplyr`, I will extract flights and weather data from 
another new package called 
[nycflights13](https://github.com/hadley/nycflights13). With this data I 
will show how to estimate a couple of regression models and nicely format the 
results into tables with `stargazer`.

Note: `stargazer` v. 5.1 does not play nicely with `dplyr`'s tbl_df class.
As a temporary work-around I pipe the merged dataset to `data.frame`.


```r
library("dplyr")
library("nycflights13")
library("AER") # Applied Econometrics with R
library("stargazer")

daily <- flights %>%
  filter(origin == "EWR") %>%
  group_by(year, month, day) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE))

daily_weather <- weather %>%
  filter(origin == "EWR") %>%
  group_by(year, month, day) %>%
  summarise(temp   = mean(temp, na.rm = TRUE),
            wind   = mean(wind_speed, na.rm = TRUE),
            precip = sum(precip, na.rm = TRUE))

# Merge flights with weather data frames
both <- inner_join(daily, daily_weather) %>% 
  data.frame()  # Temporary fix

# Create an indicator for quarter
both$quarter <- cut(both$month, breaks = c(0, 3, 6, 9, 12), 
                                labels = c("1", "2", "3", "4"))

# Create a vector of class logical
both$hot <- as.logical(both$temp > 85)

head(both)
```

```
##   year month day     delay    temp      wind precip quarter   hot
## 1 2013     1   1 17.483553 38.4800 12.758648      0       1 FALSE
## 2 2013     1   2 25.322674 28.8350 12.514732      0       1 FALSE
## 3 2013     1   3  8.450450 29.4575  7.863663      0       1 FALSE
## 4 2013     1   4 12.103858 33.4775 13.857309      0       1 FALSE
## 5 2013     1   5  5.696203 36.7325 10.836512      0       1 FALSE
## 6 2013     1   6 12.383333 37.9700  8.007511      0       1 FALSE
```

\
We can use the `both` data frame to estimate a couple of linear models 
predicting the average delay out of Newark controlling for the weather. The 
first model will use only the weather variables and in the second I'll add 
dummy variables indicating the quarter. I also estimate a third model, using 
using the `ivreg` command from package 
[AER](http://cran.r-project.org/web/packages/AER/index.html) to demonstrate 
output with mixed models. The raw R output:


```r
output  <- lm(delay ~ temp + wind + precip, data = both)
output2 <- lm(delay ~ temp + wind + precip + quarter, data = both)

# Instrumental variables model 
output3 <- ivreg(delay ~ temp + wind + precip | . - temp + hot, data = both)

summary(output)
```

```
## 
## Call:
## lm(formula = delay ~ temp + wind + precip, data = both)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -26.201  -8.497  -3.533   4.708  75.727 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  7.26298    3.09879   2.344   0.0196 *  
## temp         0.08808    0.04068   2.165   0.0310 *  
## wind         0.16648    0.16392   1.016   0.3105    
## precip      18.91805    3.24948   5.822 1.29e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 13.25 on 360 degrees of freedom
## Multiple R-squared:  0.09693,	Adjusted R-squared:  0.0894 
## F-statistic: 12.88 on 3 and 360 DF,  p-value: 5.19e-08
```

```r
summary(output2)
```

```
## 
## Call:
## lm(formula = delay ~ temp + wind + precip + quarter, data = both)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -26.927  -8.740  -3.937   5.181  74.631 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  6.14163    3.53599   1.737  0.08327 .  
## temp         0.18396    0.06856   2.683  0.00763 ** 
## wind         0.11445    0.16357   0.700  0.48459    
## precip      18.16739    3.22973   5.625 3.75e-08 ***
## quarter2    -2.26471    2.66038  -0.851  0.39519    
## quarter3    -7.52596    3.22585  -2.333  0.02020 *  
## quarter4    -4.75737    2.10380  -2.261  0.02434 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 13.12 on 357 degrees of freedom
## Multiple R-squared:  0.1218,	Adjusted R-squared:  0.107 
## F-statistic: 8.253 on 6 and 357 DF,  p-value: 2.221e-08
```

```r
summary(output3)
```

```
## 
## Call:
## ivreg(formula = delay ~ temp + wind + precip | . - temp + hot, 
##     data = both)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -25.739  -8.929  -3.834   5.015  74.767 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.7297     9.1384   1.174    0.241    
## temp          0.0342     0.1397   0.245    0.807    
## wind          0.1157     0.2070   0.559    0.577    
## precip       18.8775     3.2589   5.793 1.51e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 13.28 on 360 degrees of freedom
## Multiple R-Squared: 0.09253,	Adjusted R-squared: 0.08496 
## Wald test: 11.28 on 3 and 360 DF,  p-value: 4.326e-07
```

\
[Back to table of contents](#TOC)

## Quick notes

Since I'm using [knitr](http://cran.rstudio.com/web/packages/knitr/index.html) 
and [R markdown](http://rmarkdown.rstudio.com/) to create this webpage, in the 
code that follows I will include the `stargazer` option `type = "html"`. 
`stargazer` is set to produce **LaTeX** output by default. If you desire 
**LaTeX** output, just remove the type option from the code below.

\
Also, while I have added an example for many of the available `stargazer` 
options, I have not included all of them. So while you're likely to find a 
relevant example don't assume if it's not listed that `stargazer` can't do 
it. Check the documentation for additional features and updates to the package.
It is often the case that an omitted argument is specific for **LaTeX** output 
and I can't demonstrate it here.

### HTML formatting

It is possible to change the formatting of html tables generated with `stargazer` 
via an html style sheet. See the [R Markdown documentation](http://rmarkdown.rstudio.com/html_document_format.html#custom_css) 
about incorporating a custom CSS. 

\
[Back to table of contents](#TOC)

## The default summary statistics table

```r
stargazer(both, type = "html")
```


<table style="text-align:center"><tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Statistic</td><td>N</td><td>Mean</td><td>St. Dev.</td><td>Min</td><td>Max</td></tr>
<tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">year</td><td>364</td><td>2,013.000</td><td>0.000</td><td>2,013</td><td>2,013</td></tr>
<tr><td style="text-align:left">month</td><td>364</td><td>6.511</td><td>3.445</td><td>1</td><td>12</td></tr>
<tr><td style="text-align:left">day</td><td>364</td><td>15.679</td><td>8.784</td><td>1</td><td>31</td></tr>
<tr><td style="text-align:left">delay</td><td>364</td><td>15.080</td><td>13.883</td><td>-1.349</td><td>97.771</td></tr>
<tr><td style="text-align:left">temp</td><td>364</td><td>55.481</td><td>17.581</td><td>15.492</td><td>91.168</td></tr>
<tr><td style="text-align:left">wind</td><td>364</td><td>9.339</td><td>4.363</td><td>2.014</td><td>55.669</td></tr>
<tr><td style="text-align:left">precip</td><td>364</td><td>0.073</td><td>0.214</td><td>0.000</td><td>1.890</td></tr>
<tr><td style="text-align:left">hot</td><td>364</td><td>0.022</td><td>0.147</td><td>0</td><td>1</td></tr>
<tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr></table>

\
When supplied a data frame, by default `stargazer` creates a table with summary 
statistics. If the `summary` option is set to `FALSE` then stargazer 
will instead print the contents of the data frame.


```r
# Use only a few rows
both2 <- both %>% slice(1:6)

stargazer(both2, type = "html", summary = FALSE, rownames = FALSE)
```


<table style="text-align:center"><tr><td colspan="9" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">year</td><td>month</td><td>day</td><td>delay</td><td>temp</td><td>wind</td><td>precip</td><td>quarter</td><td>hot</td></tr>
<tr><td colspan="9" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">2,013</td><td>1</td><td>1</td><td>17.484</td><td>38.480</td><td>12.759</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>2</td><td>25.323</td><td>28.835</td><td>12.515</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>3</td><td>8.450</td><td>29.457</td><td>7.864</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>4</td><td>12.104</td><td>33.477</td><td>13.857</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>5</td><td>5.696</td><td>36.733</td><td>10.837</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>6</td><td>12.383</td><td>37.970</td><td>8.008</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td colspan="9" style="border-bottom: 1px solid black"></td></tr></table>

### Remove row and column names


```r
stargazer(both2, type = "html", summary = FALSE,
          rownames = FALSE,
          colnames = FALSE)
```


<table style="text-align:center"><tr><td colspan="9" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">2,013</td><td>1</td><td>1</td><td>17.484</td><td>38.480</td><td>12.759</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>2</td><td>25.323</td><td>28.835</td><td>12.515</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>3</td><td>8.450</td><td>29.457</td><td>7.864</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>4</td><td>12.104</td><td>33.477</td><td>13.857</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>5</td><td>5.696</td><td>36.733</td><td>10.837</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td style="text-align:left">2,013</td><td>1</td><td>6</td><td>12.383</td><td>37.970</td><td>8.008</td><td>0</td><td>1</td><td>FALSE</td></tr>
<tr><td colspan="9" style="border-bottom: 1px solid black"></td></tr></table>

### Change which statistics are displayed

In order to customize which summary statistics are displayed, change any of the
the following (logical) parameters, `nobs`, `mean.sd`, `min.max`, `median`, and
`iqr`. 


```r
stargazer(both, type = "html", nobs = FALSE, mean.sd = TRUE, median = TRUE,
          iqr = TRUE)
```


<table style="text-align:center"><tr><td colspan="8" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Statistic</td><td>Mean</td><td>St. Dev.</td><td>Min</td><td>Pctl(25)</td><td>Median</td><td>Pctl(75)</td><td>Max</td></tr>
<tr><td colspan="8" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">year</td><td>2,013.000</td><td>0.000</td><td>2,013</td><td>2,013</td><td>2,013</td><td>2,013</td><td>2,013</td></tr>
<tr><td style="text-align:left">month</td><td>6.511</td><td>3.445</td><td>1</td><td>4</td><td>7</td><td>9.2</td><td>12</td></tr>
<tr><td style="text-align:left">day</td><td>15.679</td><td>8.784</td><td>1</td><td>8</td><td>16</td><td>23</td><td>31</td></tr>
<tr><td style="text-align:left">delay</td><td>15.080</td><td>13.883</td><td>-1.349</td><td>5.446</td><td>10.501</td><td>20.007</td><td>97.771</td></tr>
<tr><td style="text-align:left">temp</td><td>55.481</td><td>17.581</td><td>15.492</td><td>39.873</td><td>56.960</td><td>71.570</td><td>91.168</td></tr>
<tr><td style="text-align:left">wind</td><td>9.339</td><td>4.363</td><td>2.014</td><td>6.557</td><td>8.847</td><td>11.556</td><td>55.669</td></tr>
<tr><td style="text-align:left">precip</td><td>0.073</td><td>0.214</td><td>0.000</td><td>0.000</td><td>0.000</td><td>0.020</td><td>1.890</td></tr>
<tr><td style="text-align:left">hot</td><td>0.022</td><td>0.147</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
<tr><td colspan="8" style="border-bottom: 1px solid black"></td></tr></table>

### Change which statistics are displayed (a second way)


```r
stargazer(both, type = "html", summary.stat = c("n", "p75", "sd"))
```


<table style="text-align:center"><tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Statistic</td><td>N</td><td>Pctl(75)</td><td>St. Dev.</td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">year</td><td>364</td><td>2,013</td><td>0.000</td></tr>
<tr><td style="text-align:left">month</td><td>364</td><td>9.2</td><td>3.445</td></tr>
<tr><td style="text-align:left">day</td><td>364</td><td>23</td><td>8.784</td></tr>
<tr><td style="text-align:left">delay</td><td>364</td><td>20.007</td><td>13.883</td></tr>
<tr><td style="text-align:left">temp</td><td>364</td><td>71.570</td><td>17.581</td></tr>
<tr><td style="text-align:left">wind</td><td>364</td><td>11.556</td><td>4.363</td></tr>
<tr><td style="text-align:left">precip</td><td>364</td><td>0.020</td><td>0.214</td></tr>
<tr><td style="text-align:left">hot</td><td>364</td><td>0</td><td>0.147</td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr></table>

### Remove logical variables in the summary statistics

`stargazer` reports summary statistics for logical variables by default 
(0 = FALSE and 1 = TRUE). To supress the reporting of logical vectors change 
`summary.logical` to `FALSE`. Note the stats for our vector `hot` are gone. 


```r
stargazer(both, type = "html", summary.logical = FALSE)
```


<table style="text-align:center"><tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Statistic</td><td>N</td><td>Mean</td><td>St. Dev.</td><td>Min</td><td>Max</td></tr>
<tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">year</td><td>364</td><td>2,013.000</td><td>0.000</td><td>2,013</td><td>2,013</td></tr>
<tr><td style="text-align:left">month</td><td>364</td><td>6.511</td><td>3.445</td><td>1</td><td>12</td></tr>
<tr><td style="text-align:left">day</td><td>364</td><td>15.679</td><td>8.784</td><td>1</td><td>31</td></tr>
<tr><td style="text-align:left">delay</td><td>364</td><td>15.080</td><td>13.883</td><td>-1.349</td><td>97.771</td></tr>
<tr><td style="text-align:left">temp</td><td>364</td><td>55.481</td><td>17.581</td><td>15.492</td><td>91.168</td></tr>
<tr><td style="text-align:left">wind</td><td>364</td><td>9.339</td><td>4.363</td><td>2.014</td><td>55.669</td></tr>
<tr><td style="text-align:left">precip</td><td>364</td><td>0.073</td><td>0.214</td><td>0.000</td><td>1.890</td></tr>
<tr><td colspan="6" style="border-bottom: 1px solid black"></td></tr></table>

### Flip the table axes


```r
stargazer(both, type = "html", flip = TRUE)
```


<table style="text-align:center"><tr><td colspan="9" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Statistic</td><td>year</td><td>month</td><td>day</td><td>delay</td><td>temp</td><td>wind</td><td>precip</td><td>hot</td></tr>
<tr><td colspan="9" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">N</td><td>364</td><td>364</td><td>364</td><td>364</td><td>364</td><td>364</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">Mean</td><td>2,013.000</td><td>6.511</td><td>15.679</td><td>15.080</td><td>55.481</td><td>9.339</td><td>0.073</td><td>0.022</td></tr>
<tr><td style="text-align:left">St. Dev.</td><td>0.000</td><td>3.445</td><td>8.784</td><td>13.883</td><td>17.581</td><td>4.363</td><td>0.214</td><td>0.147</td></tr>
<tr><td style="text-align:left">Min</td><td>2,013</td><td>1</td><td>1</td><td>-1.349</td><td>15.492</td><td>2.014</td><td>0.000</td><td>0</td></tr>
<tr><td style="text-align:left">Max</td><td>2,013</td><td>12</td><td>31</td><td>97.771</td><td>91.168</td><td>55.669</td><td>1.890</td><td>1</td></tr>
<tr><td colspan="9" style="border-bottom: 1px solid black"></td></tr></table>

\
[Back to table of contents](#TOC)

## The default regression table


```r
stargazer(output, output2, type = "html")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

\
[Back to table of contents](#TOC)

## Change the style

`stargazer` includes several pre-formatted styles that imitate popular
academic journals. Use the `style` argument.


```r
stargazer(output, output2, type = "html", style = "qje")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left"><em>N</em></td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Notes:</em></td><td colspan="2" style="text-align:right"><sup>***</sup>Significant at the 1 percent level.</td></tr>
<tr><td style="text-align:left"></td><td colspan="2" style="text-align:right"><sup>**</sup>Significant at the 5 percent level.</td></tr>
<tr><td style="text-align:left"></td><td colspan="2" style="text-align:right"><sup>*</sup>Significant at the 10 percent level.</td></tr>
</table>

\
[Back to table of contents](#TOC)

## Labelling the table

### Add a title; change the variable labels 


```r
stargazer(output, output2, type = "html", 
          title            = "These are awesome results!",
          covariate.labels = c("Temperature", "Wind speed", "Rain (inches)",
                               "2nd quarter", "3rd quarter", "Fourth quarter"),
          dep.var.caption  = "A better caption",
          dep.var.labels   = "Flight delay (in minutes)")
```


<table style="text-align:center"><caption><strong>These are awesome results!</strong></caption>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2">A better caption</td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">Flight delay (in minutes)</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Temperature</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Wind speed</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Rain (inches)</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">2nd quarter</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">3rd quarter</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Fourth quarter</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Exclude the dependent variable label or the model numbers

Note the dependent variable caption stays. To additionally remove the caption 
add `dep.var.caption = ""`.


```r
stargazer(output, output2, type = "html", 
          dep.var.labels.include = FALSE,
          model.numbers          = FALSE)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Change the column names

To change the column names just supply a character vector with the new labels, 
as shown below. 


```r
stargazer(output, output2, type = "html", column.labels = c("Good", "Better"))
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>Good</td><td>Better</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Apply a label to more than one column

The option `column.separate` allows for assigning a label to more than one 
column. In this example I told `stargazer` to report each regression twice, for a
total of four columns. Using `column.separate`, `stargazer` now applies the first
label to the first two columns and the second label to the next two columns.


```r
stargazer(output, output, output2, output2, type = "html", 
          column.labels   = c("Good", "Better"),
          column.separate = c(2, 2))
```


<table style="text-align:center"><tr><td colspan="5" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="4"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="4" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="4">delay</td></tr>
<tr><td style="text-align:left"></td><td colspan="2">Good</td><td colspan="2">Better</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td><td>(3)</td><td>(4)</td></tr>
<tr><td colspan="5" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.041)</td><td>(0.069)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.166</td><td>0.114</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.249)</td><td>(3.230)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td></td><td>-2.265</td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td>(2.660)</td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td></td><td>-7.526<sup>**</sup></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td>(3.226)</td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td></td><td>-4.757<sup>**</sup></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td>(2.104)</td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.099)</td><td>(3.536)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td><td></td></tr>
<tr><td colspan="5" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.097</td><td>0.122</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.089</td><td>0.107</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="5" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="4" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Model names

When the results from different types of regression models (e.g., "OLS", 
"probit") are displayed in the same table `stargazer` adds a row indicating 
model type. Remove these labels by including `model.names = FALSE` (not shown).


```r
stargazer(output, output2, output3, type = "html")
```


<table style="text-align:center"><tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="3"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="3" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="3">delay</td></tr>
<tr><td style="text-align:left"></td><td colspan="2"><em>OLS</em></td><td><em>instrumental</em></td></tr>
<tr><td style="text-align:left"></td><td colspan="2"><em></em></td><td><em>variable</em></td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td><td>(3)</td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td><td>0.034</td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td><td>(0.140)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td><td>0.116</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td><td>(0.207)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td><td>18.877<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td><td>(3.259)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td><td>10.730</td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td><td>(9.138)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td><td>0.093</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td><td>0.085</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td><td>13.280 (df = 360)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td><td></td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="3" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Model names (again)

The example above shows the default behavior of `stargazer` is to display only 
one model name (and dependent variable caption) for adjacent columns with the 
same model type. To repeat these labels for all of the columns, do the 
following:


```r
stargazer(output, output2, output3, type = "html",
          multicolumn = FALSE)
```


<table style="text-align:center"><tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="3"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="3" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td>delay</td><td>delay</td><td>delay</td></tr>
<tr><td style="text-align:left"></td><td><em>OLS</em></td><td><em>OLS</em></td><td><em>instrumental</em></td></tr>
<tr><td style="text-align:left"></td><td><em></em></td><td><em></em></td><td><em>variable</em></td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td><td>(3)</td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td><td>0.034</td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td><td>(0.140)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td><td>0.116</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td><td>(0.207)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td><td>18.877<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td><td>(3.259)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td><td></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td><td>10.730</td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td><td>(9.138)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td><td></td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td><td>0.093</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td><td>0.085</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td><td>13.280 (df = 360)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td><td></td></tr>
<tr><td colspan="4" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="3" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Add a custom row to the reported statistics

I use this example to show how to add a row(s), such as reporting fixed effects.


```r
stargazer(output, output2, type = "html",
          add.lines = list(c("Fixed effects?", "No", "No"),
                           c("Results believable?", "Maybe", "Try again later")))
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Fixed effects?</td><td>No</td><td>No</td></tr>
<tr><td style="text-align:left">Results believable?</td><td>Maybe</td><td>Try again later</td></tr>
<tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Include R object names


```r
stargazer(output, output2, type = "html", 
          object.names = TRUE)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td style="text-align:left"></td><td>output</td><td>output2</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

\
[Back to table of contents](#TOC)

## Change the default output

### Report t-statistics or p-values instead of standard errors

Standard errors are reported by default. To report the t-statistics or
p-values instead, see the `report` argument. Notice that I've used this option 
to move the star characters to the t-statistics instead of being next to the
coefficients.


```r
stargazer(output, output2, type = "html",
          report = "vct*")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088</td><td>0.184</td></tr>
<tr><td style="text-align:left"></td><td>t = 2.165<sup>**</sup></td><td>t = 2.683<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>t = 1.016</td><td>t = 0.700</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918</td><td>18.167</td></tr>
<tr><td style="text-align:left"></td><td>t = 5.822<sup>***</sup></td><td>t = 5.625<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>t = -0.851</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526</td></tr>
<tr><td style="text-align:left"></td><td></td><td>t = -2.333<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757</td></tr>
<tr><td style="text-align:left"></td><td></td><td>t = -2.261<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263</td><td>6.142</td></tr>
<tr><td style="text-align:left"></td><td>t = 2.344<sup>**</sup></td><td>t = 1.737<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Report confidence intervals


```r
stargazer(output, output2, type = "html",
          ci = TRUE)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.008, 0.168)</td><td>(0.050, 0.318)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(-0.155, 0.488)</td><td>(-0.206, 0.435)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(12.549, 25.287)</td><td>(11.837, 24.498)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(-7.479, 2.950)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(-13.849, -1.203)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(-8.881, -0.634)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(1.189, 13.337)</td><td>(-0.789, 13.072)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Adjust the confidence intervals
 
By default `ci.level = 0.95`. You may also change the character that separates 
the intervals with the `ci.separator` argument.


```r
stargazer(output, output2, type = "html",
          ci = TRUE, ci.level = 0.90, ci.separator = " @@ ")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.021 @@ 0.155)</td><td>(0.071 @@ 0.297)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(-0.103 @@ 0.436)</td><td>(-0.155 @@ 0.384)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(13.573 @@ 24.263)</td><td>(12.855 @@ 23.480)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(-6.641 @@ 2.111)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(-12.832 @@ -2.220)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(-8.218 @@ -1.297)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(2.166 @@ 12.360)</td><td>(0.325 @@ 11.958)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Robust standard errors (replicating Stata's robust option)

If you want to use robust standard errors (or clustered), `stargazer` allows 
for replacing the default output by supplying a new vector of values to the 
option `se`. For this example I will display the same model twice and adjust the 
standard errors in the second column with the `HC1` correction from the `sandwich`
package (i.e. the same correction Stata uses). 

I also need to adjust the F statistic with the corrected variance-covariance 
matrix (matching Stata's results). Currently, this must be done manually 
(via `add.lines`) as `stargazer` does not (yet) have an option for directly 
replacing the F statistic.

Similar options exist to supply adjusted values to the coefficients, 
t-statistics, confidence intervals, and p-values. See `coef`, `t`, `ci.custom`, 
or `p`.


```r
library(sandwich)
library(lmtest)   # waldtest; see also coeftest.

# Adjust standard errors
cov1         <- vcovHC(output, type = "HC1")
robust_se    <- sqrt(diag(cov1))

# Adjust F statistic 
wald_results <- waldtest(output, vcov = cov1)

stargazer(output, output, type = "html",
          se        = list(NULL, robust_se),
          omit.stat = "f",
          add.lines = list(c("F Statistic (df = 3; 360)", "12.879***", "7.73***")))
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.088<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.043)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.166</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.159)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.918<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(4.735)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>7.263<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.053)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">F Statistic (df = 3; 360)</td><td>12.879***</td><td>7.73***</td></tr>
<tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.097</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.089</td></tr>
<tr><td style="text-align:left">Residual Std. Error (df = 360)</td><td>13.248</td><td>13.248</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Move the intercept term to the top of the table


```r
stargazer(output, output2, type = "html",
          intercept.bottom = FALSE)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Compress the table output

We can condense the table output by placing all the the output on the same row.
When `single.row` is set to `TRUE`, the argument `no.space` is automatically 
set to `TRUE`.


```r
stargazer(output, output2, type = "html",
          single.row = TRUE)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup> (0.041)</td><td>0.184<sup>***</sup> (0.069)</td></tr>
<tr><td style="text-align:left">wind</td><td>0.166 (0.164)</td><td>0.114 (0.164)</td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup> (3.249)</td><td>18.167<sup>***</sup> (3.230)</td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265 (2.660)</td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup> (3.226)</td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup> (2.104)</td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup> (3.099)</td><td>6.142<sup>*</sup> (3.536)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

\
[Back to table of contents](#TOC)

## Omit parts of the default output

In fixed effect model specifications it is often undesirable to report the 
fixed effect coefficients. To omit any coefficient, supply a regular 
expression to `omit`.


```r
stargazer(output, output2, type = "html", omit = "quarter")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Reporting omitted variables

Add the `omit.labels` parameter to report which variables have been omitted. 
Must be the same length as the number of regular expressions supplied to `omit`.
By default `omit.labels` reports "Yes" or "No". To change this supply 
a new vector of length 2 to `omit.yes.no = c("Yes", "No")`.


```r
stargazer(output, output2, type = "html", 
          omit        = "quarter",
          omit.labels = "Quarter dummies?")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Quarter dummies?</td><td>No</td><td>No</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Omit summary statistics


```r
## Remove r-square and f-statistic
stargazer(output, output2, type = "html", 
          omit.stat = c("rsq", "f"))
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

\
See also `keep.stat` a related argument with the opposite behavior.

### Omit whole parts of the table

If you just want to remove parts of the table it is easier to use 
`omit.table.layout` to explicitly specify table elements. See 
`table layout chracters` for a list of codes.


```r
# Remove statistics and notes sections completely
stargazer(output, output2, type = "html", 
          omit.table.layout = "sn")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr></table>

### Omit whole parts of the table (a second way)

Another way to achieve the result above is through the argument `table.layout`. It 
also accepts a character string that tells `stargazer` which table elements to 
**include**.


```r
# Include everything except the statistics and notes sections
stargazer(output, output2, type = "html", 
          table.layout = "-ld#-t-")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr></table>

### Omit the degrees of freedom


```r
stargazer(output, output2, type = "html", 
          df = FALSE)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248</td><td>13.119</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup></td><td>8.253<sup>***</sup></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

\
[Back to table of contents](#TOC)

## Statistical significance options

By default `stargazer` uses \*\*\*, \*\*, and \* to denote statistical 
significance at the one, five, and ten percent levels. This behavior can be 
changed by altering the `star.char` option. 


```r
stargazer(output, output2, type = "html", 
          star.char = c("@", "@@", "@@@"))
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>@@</sup></td><td>0.184<sup>@@@</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>@@@</sup></td><td>18.167<sup>@@@</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>@@</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>@@</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>@@</sup></td><td>6.142<sup>@</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>@@@</sup> (df = 3; 360)</td><td>8.253<sup>@@@</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Change the cutoffs for significance

Notice that temperature, quarter3, and quarter4 have each lost a gold star 
because we made it tougher to earn them.


```r
stargazer(output, output2, type = "html", 
          star.cutoffs = c(0.05, 0.01, 0.001)) # star.cutoffs = NULL by default
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>*</sup></td><td>0.184<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>*</sup></td><td>6.142</td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.05; <sup>**</sup>p<0.01; <sup>***</sup>p<0.001</td></tr>
</table>

\
[Back to table of contents](#TOC)

## Modifying table notes

### Make an addition to the existing note section


```r
stargazer(output, output2, type = "html", 
          notes = "I make this look good!")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
<tr><td style="text-align:left"></td><td colspan="2" style="text-align:right">I make this look good!</td></tr>
</table>

### Replace the note section


```r
stargazer(output, output2, type = "html", 
          notes        = "Sometimes you just have to start over.", 
          notes.append = FALSE)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right">Sometimes you just have to start over.</td></tr>
</table>

### Change note alignment


```r
stargazer(output, output2, type = "html", 
          notes.align = "l")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:left"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Change the note section label


```r
stargazer(output, output2, type = "html", 
          notes.label = "New note label")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">New note label</td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

\
[Back to table of contents](#TOC)

## Table aesthetics

### Use html tags to modify table elements

```r
# For LaTeX output you can also wrap table text in commands like \textbf{...}, 
# just remember to escape the first backslash, e.g., "A \\textbf{better} caption"

stargazer(output, output2, type = "html", 
          title = "These are <em> awesome </em> results!",  # Italics
          dep.var.caption  = "A <b> better </b> caption")   # Bold
```


<table style="text-align:center"><caption><strong>These are <em> awesome </em> results!</strong></caption>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2">A <b> better </b> caption</td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Change decimal character


```r
stargazer(output, output2, type = "html", 
          decimal.mark = ",")
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0,088<sup>**</sup></td><td>0,184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0,041)</td><td>(0,069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0,166</td><td>0,114</td></tr>
<tr><td style="text-align:left"></td><td>(0,164)</td><td>(0,164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18,918<sup>***</sup></td><td>18,167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3,249)</td><td>(3,230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2,265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2,660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7,526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3,226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4,757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2,104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7,263<sup>**</sup></td><td>6,142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3,099)</td><td>(3,536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0,097</td><td>0,122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0,089</td><td>0,107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13,248 (df = 360)</td><td>13,119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12,879<sup>***</sup> (df = 3; 360)</td><td>8,253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0,1; <sup>**</sup>p<0,05; <sup>***</sup>p<0,01</td></tr>
</table>

### Control the number of decimal places


```r
stargazer(output, output2, type = "html", 
          digits = 1)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.1<sup>**</sup></td><td>0.2<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.04)</td><td>(0.1)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.2</td><td>0.1</td></tr>
<tr><td style="text-align:left"></td><td>(0.2)</td><td>(0.2)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.9<sup>***</sup></td><td>18.2<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.2)</td><td>(3.2)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.3</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.7)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.5<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.2)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.8<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.1)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.3<sup>**</sup></td><td>6.1<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.1)</td><td>(3.5)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.1</td><td>0.1</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.1</td><td>0.1</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.2 (df = 360)</td><td>13.1 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.9<sup>***</sup> (df = 3; 360)</td><td>8.3<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Additional decimal controls

You may also specify the number of additional decimal places to be used if a 
number, when rounded to `digits` decimal places, is equal to zero (Use argument 
`digits.extra`).

My example models do not have any numbers in the thousands, so I won't show 
them, but `digit.separate` and `digits.separator` are also available for 
customizing the output of those characters.


```r
stargazer(output, output2, type = "html", 
          digits       = 1,
          digits.extra = 1)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>0.1<sup>**</sup></td><td>0.2<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.04)</td><td>(0.1)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.2</td><td>0.1</td></tr>
<tr><td style="text-align:left"></td><td>(0.2)</td><td>(0.2)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.9<sup>***</sup></td><td>18.2<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.2)</td><td>(3.2)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.3</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.7)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.5<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.2)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.8<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.1)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.3<sup>**</sup></td><td>6.1<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.1)</td><td>(3.5)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.1</td><td>0.1</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.1</td><td>0.1</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.2 (df = 360)</td><td>13.1 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.9<sup>***</sup> (df = 3; 360)</td><td>8.3<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Drop leading zeros from decimals


```r
stargazer(output, output2, type = "html", 
          initial.zero = FALSE)
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">temp</td><td>.088<sup>**</sup></td><td>.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(.041)</td><td>(.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>.166</td><td>.114</td></tr>
<tr><td style="text-align:left"></td><td>(.164)</td><td>(.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>.097</td><td>.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>.089</td><td>.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Change the order of the variables

The `order` argument will also accept a vector of regular expressions.


```r
stargazer(output, output2, type = "html", 
          order = c(4, 5, 6, 3, 2, 1))
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">quarter2</td><td></td><td>-2.265</td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.660)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter3</td><td></td><td>-7.526<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(3.226)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">quarter4</td><td></td><td>-4.757<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td></td><td>(2.104)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">wind</td><td>0.166</td><td>0.114</td></tr>
<tr><td style="text-align:left"></td><td>(0.164)</td><td>(0.164)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">temp</td><td>0.088<sup>**</sup></td><td>0.184<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.041)</td><td>(0.069)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">Constant</td><td>7.263<sup>**</sup></td><td>6.142<sup>*</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.099)</td><td>(3.536)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

### Select which variables to keep in the table

By default `keep = NULL` meaning all variables are included. `keep` accepts a
vector of regular expressions.


```r
# Regex for keep "precip" but not "precipitation"
stargazer(output, output2, type = "html", 
          keep = c("\\bprecip\\b"))
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">delay</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">precip</td><td>18.918<sup>***</sup></td><td>18.167<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(3.249)</td><td>(3.230)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>364</td><td>364</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.097</td><td>0.122</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.089</td><td>0.107</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>13.248 (df = 360)</td><td>13.119 (df = 357)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>12.879<sup>***</sup> (df = 3; 360)</td><td>8.253<sup>***</sup> (df = 6; 357)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

\
[Back to table of contents](#TOC)

## Feedback

The .Rmd file for this cheatsheet [is on GitHub](https://github.com/JakeRuss/cheatsheets) 
and I welcome suggestions or pull requests.

\
[Back to table of contents](#TOC)
