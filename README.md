

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
knitr::opts_knit$set(root.dir = "")
getwd()
load("acs2017_ny_data.RData")
acs2017_ny[1:10,1:7]
attach(acs2017_ny)
```

<p style="color:rgb(182,18,27);font-family:corbel">Mónica Martínez-Raga</p>
<p style="color:rgb(182,18,27);font-family:corbel">HW4- Fall 2020</p>
<p style="color:rgb(182,18,27);font-family:corbel">OLS</p>
<p style="color:rgb(182,18,27);font-family:corbel">Collaborators: Isabela Vieira</p>


A lot of undergraduate students are already thinking about their next academic degree before finishing their bachelor's, for the most part to access high paying careers. But many are contesting whether a higher degree is even worth the time and financial costs. One way for many to mitigate the risk of college debt is by embarking in higher education that guarantees higher pay afterwards.

This is the question I want to answer today, does higher educational degree (EDUCD) equate higher pay (INCWAGE)?

Null Hypothesis - Ho : No relation, Higher educational degree achievement does not lead to increase in wage.
Alternative Hypothesis - Ha :  Relation, Higher educational degree achievement leads to increase in wage. 



We have to make some assumptions first: 
1) I did not find data in terms of what field people are actually working in. So I will assume most people work and earn income in the field of study of their highest achieved degree. 
2) At the moment were are not taking into account other possible variables that contribute to wage indifferences, like gender, race and school.
2) Below we have filtered out the population into ages assumed to be in the laborforce (LABFORCE), fulltime (WKSWORK2 & UHRSWORK) and working for wage rather than self-employed (CLASSWKR).

```{r}
use_varb <- (AGE >= 22) & (AGE <= 55) & (LABFORCE == 2) & (WKSWORK2 > 4) & (UHRSWORK >= 35) & (CLASSWKR == 2)
dat_use <- subset(acs2017_ny,use_varb) 
detach(acs2017_ny)
attach(dat_use)
```


As we can see below, our variables of interest are:
[23] "12th grade, no diploma" (high school drop-out)
[25] "Regular high school diploma"
[29] "1 or more years of college credit, no degree" (college drop-out)
[31] "Associate's degree, type not specified"
[36] "Bachelor's degree" 
[41] "Master's degree"
[42] "Professional degree beyond a bachelor's degree"
[43] "Doctoral degree"
*Some of these levels are unclear or seem repetitive, therefore I chose to work with the most populated one.

As we can observe, most people in our data set have bachelor's degrees, followed by just a highschool diploma, and then a master's degree. 

```{r}
levels(dat_use$EDUCD)
```

```{r}
plot(dat_use$EDUCD)
```


I attempted the following commands to filter out unpopulated levels with no success. However, I don't think they will cause too much trouble since their estimates are negligible due to their miniscule or non-exitent sample sizes. 
```{r}
#install.packages(c("dplyr"))
#as.integer(EDUCD) 
#dplyr::filter( EDUCD =< 23)
#outLevels<-c(36)
# dplyr :: filter(!as.integer(EDUCD) %in% outLevels)
```




Moving on: I included a function that normalizes any variable in the dat_use data set. This is necessary in order to equalize the lengths of each vector.
```{r}
norm_varb <- function(X_in) {
  (X_in - min(X_in, na.rm = TRUE))/( max(X_in, na.rm = TRUE) - min(X_in, na.rm = TRUE) )
}
```

We now normalize education and INCWAGE. 
```{r}
EDUCDx <- factor(dat_use$EDUCD)
EDUCDx <- as.numeric(EDUCDx)
is.na(EDUCDx) <- which(EDUCDx == 0)
norm_EDUCD <- norm_varb(EDUCDx)
is.na(INCWAGE) <- which(INCWAGE == 0)
norm_INCWAGE <- norm_varb(INCWAGE)
```


Below we run a quick correlation matrix, just to verify if our hypothesis is on the right track. Education attainment and wage are possitively correlated. 
```{r}
DF <- data.frame(norm_INCWAGE,
                 norm_EDUCD)
correlation_1 <- cor(DF)
print(correlation_1)
```

Furthermore, in the graph below we see how there is a huge jump from obtaining a bachelor's and anything below. As we saw before, a lot of the levels that appear conglomerated in this graph have either none or a neglible sized population.  The fact that they lie close to the wage of our lower bound (highschool dropout or no degree) means that we can expect highschool drop out to be a good representation of the mean wage for lower categories.

```{r}
Table1 <- tapply(INCWAGE, list(EDUCD), mean )
print(Table1)
plot(Table1, main="Relationship between Income and Degree Achievement",
   xlab="Degree", ylab="Income", pch=18, col="blue")
text(Table1, row.names(Table1), cex=0.6, pos=4, col="red")
```

```{r}
install.packages(c("AER", "stargazer"))
```


After running a first regression, I see some annomalies. The biggest one is that our intercept seems to be our level 3, or "no schooling completed." The reason I think this is because the estimate is positive, yet below what a person with up to 10th grade of education would make. Given our plot from before, most people in our data set are actually at or above having graduated highschool. Therefore, I think the levels are disproportionately pulling our incepcept down, especially since most of the residuals are negative for the overall model but that's to be expected when we are only evaluating one variable and not accounting for others that affect wage. In addition, our residual standard error is huge, meaning our data points are quite dispersed, the standard errors for the negative numbers are very large relative to the estimate and the t value are also closer to 0 than with degree holders. 

However, we see some positive patterns in the levels we were specifically evaluating. First that most of the distiguished milestones below highshool diploma, 6th grade and 10th grade (many turn 16 legally able to work) have positive estimates. I can assume this means more people complete up to these levels and enter the workforce than dropping out mid-way. I also appreciated seeing that exactly at "12th grade, no diploma" our standard errors become smaller than the estimates, meaning those particular levels are more accurate. The other coefficients are also following higher and lower t and p values, respectively.
```{r}
m2 <- lm(INCWAGE ~ EDUCD)
summary(m2)
plot(m2)
require(stargazer)
stargazer(m2, type = "text")
```

I'm going to try to run the model with level 23 "12th grade, no diploma" as the leading intercept. This should show my model relative to what we've defined a non-degree. Our t-values have increase for our variables of interest, showing even more of a relationship between at least reaching 12th grade and getting higher pay.
```{r}
DF1 <- data.frame(INCWAGE, 
                  
                  EDUCD)
DF1$EDUCD = relevel(DF1$EDUCD, ref=23)
m2 <- lm(INCWAGE ~ EDUCD, data = DF1)
summary(m2)
plot(m2)
require(stargazer)
stargazer(m2, type = "text")
```


For the life of me I could not get this to work. The values I'm evaluating are numeric yet I am prevented from running it because " 'x' must be numeric."

```{r}
NNobs <- length(INCWAGE)
set.seed(12345) 
graph_obs <- (runif(NNobs) < 0.1)
dat_graph <-subset(dat_use,graph_obs)  
EDUCD <- as.numeric(EDUCD)
plot(INCWAGE ~ jitter(EDUCD, factor = 2), pch = 16, col = rgb(0.5, 0.5, 0.5, alpha = 0.2), data = dat_graph)
plot(INCWAGE ~ jitter(EDUCD, factor = 2), pch = 16, col = rgb(0.5, 0.5, 0.5, alpha = 0.2), ylim = c(0,150000), data = dat_graph)
to_be_predicted2 <- data.frame(INCWAGE, EDUCD)
to_be_predicted2$yhat <- predict(m2, newdata = to_be_predicted2)
lines(yhat ~ norm_EDUCD, data = to_be_predicted2)
```

In conclusion, we can reject the null hypothesis given our discoveries that obtaining higher degrees is positively correlated with wage. As the discrepancies between each additional level of education are not equal, if I were to go further with this porject I would analyze within differenct fields. Does wage increase for anyone that achives higher education? Is it more dependable in certain careers that others? Will wage increase a signifant enough amount that would make it worth it to pay for the degree it, incorporating college debt and national degree market prices across universities? 

```{r}
detach()
```
