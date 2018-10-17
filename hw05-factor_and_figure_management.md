---
title: "Hw05-factor and figure management"
output: 
   html_document:
      keep_md: yes
---

## Goals

 * Factor management 
     * Drop Oceania
     * Reorder the levels of country or continent
  
 * File I/O

 * Visualization design

 * Writing figures to file

Firstly, I need to load packages I am going to use for this assignment.


```r
library(gapminder)
library(tidyverse)
library(knitr)
library(plotly)
library(gridExtra)
```

## Part 1: factor management

### Drop Oceania

**Before Drop Oceania**

Before Dropp Oceania, let's look at levels of continent variable and see how many entries correspond to each continent.


```r
p1<-gapminder$continent 
```


```r
levels(p1) #levels of continent variable
```

```
## [1] "Africa"   "Americas" "Asia"     "Europe"   "Oceania"
```

```r
kable(fct_count(p1),col.names = c('continent','number of entry')) #number of entries for each continent
```



continent    number of entry
----------  ----------------
Africa                   624
Americas                 300
Asia                     396
Europe                   360
Oceania                   24

```r
a<-nlevels(p1) #number of levels
b<-nrow(gapminder) #number of entries after removing oceania
kable(data.frame(number_of_levels = a, number_of_entries = b))
```



 number_of_levels   number_of_entries
-----------------  ------------------
                5                1704

**Drop Oceania**

We need to remove observations associated with the continent of Oceania. Additionally, remove unused factor levels.


```r
p2<-gapminder%>%
  filter(continent != 'Oceania')%>%
  droplevels()
```


**After Drop Oceania**

Now we can look at the level of continent variable and see how many entries correspond to each continent.


```r
p3<-p2$continent
levels(p3) #levels of continent variable
```

```
## [1] "Africa"   "Americas" "Asia"     "Europe"
```

```r
kable(fct_count(p3),col.names = c('continent','number of entry')) #number of entries for each continent
```



continent    number of entry
----------  ----------------
Africa                   624
Americas                 300
Asia                     396
Europe                   360

```r
a<-nlevels(p3) #number of levels
b<-nrow(p2) #number of entries after removing oceania
kable(data.frame(number_of_levels = a, number_of_entries = b))
```



 number_of_levels   number_of_entries
-----------------  ------------------
                4                1680

As expected, the number of rows has dropped by 24, corresponding to the number of observations for Oceania. The number of levels for continent has dropped by 1(Oceania level is dropped).

### Reorder the levels of conitnent

Use the forcats package to change the order of the factor levels, based on a principled summary of one of the quantitative variables. Consider experimenting with a summary statistic beyond the most basic choice of the median.

In this question, I will reorder the data based on gdp range, which is difference between maximum gdpPercap and minimum gdpPercap of each continent.

Firstly, I need to use `group_by()` and `summarize()` functions to calculate gdp range for each continent. 


```r
p4<-group_by(gapminder,continent)%>%
  summarize(gdp_range = max(gdpPercap)-min(gdpPercap))
```


```r
levels(p4$continent)
```

```
## [1] "Africa"   "Americas" "Asia"     "Europe"   "Oceania"
```

```r
kable(p4)
```



continent    gdp_range
----------  ----------
Africa        21710.05
Americas      41750.02
Asia         113192.13
Europe        48383.66
Oceania       24395.77

```r
ggplot(p4,aes(x=gdp_range,y=continent))+
  geom_point(color='red')
```

![](hw05-factor_and_figure_management_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Now I want to use `fct_reorder()` function to reorder the factor levels of each continent by their gdp_range variable.


```r
p5<-mutate(p4,continent=fct_reorder(continent,gdp_range))
```


```r
levels(p5$continent)
```

```
## [1] "Africa"   "Oceania"  "Americas" "Europe"   "Asia"
```

```r
kable(p5)
```



continent    gdp_range
----------  ----------
Africa        21710.05
Americas      41750.02
Asia         113192.13
Europe        48383.66
Oceania       24395.77

```r
ggplot(p5,aes(x=gdp_range,y=continent))+
  geom_point(color='red')
```

![](hw05-factor_and_figure_management_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

According to table, we can find `fct_reorder()` function reorder factor levels of continent and the plot is ordered in the desired way, but it does not rearrange data in the dataframe.  

Next, I will use `arrange()` function to arrange the order of data by their gdp_range variable.


```r
p6<-p4%>%
  arrange(gdp_range)
```


```r
levels(p6$continent)
```

```
## [1] "Africa"   "Americas" "Asia"     "Europe"   "Oceania"
```

```r
kable(p6)
```



continent    gdp_range
----------  ----------
Africa        21710.05
Oceania       24395.77
Americas      41750.02
Europe        48383.66
Asia         113192.13

```r
ggplot(p6,aes(x=gdp_range,y=continent))+
  geom_point(color='red')
```

![](hw05-factor_and_figure_management_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

Obviously, `arrange()` function arranges the data in an increading order of gdp_range variable, but it does not change the factor levels of contiennt. 

Now we can combine `arrange()` and `fct_reorder` to see what happens. 


```r
p7<-p4%>%
  arrange(gdp_range)%>%
  mutate(continent=fct_reorder(continent,gdp_range))
```


```r
levels(p7$continent)
```

```
## [1] "Africa"   "Oceania"  "Americas" "Europe"   "Asia"
```

```r
kable(p7)
```



continent    gdp_range
----------  ----------
Africa        21710.05
Oceania       24395.77
Americas      41750.02
Europe        48383.66
Asia         113192.13

```r
ggplot(p7,aes(x=gdp_range,y=continent))+
  geom_point(color='red')
```

![](hw05-factor_and_figure_management_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

We can find that both table and plot are ordered in the desired way. 

Conclusion:

 * `fct_reorder()` function changes factor levels of continent but it does not rearrange data in the dataframe. It has an effect on how variables are organized on plots.  
 * `arrange()` function arranges the data in an increading order of gdp_range variable, but it does not change the factor levels of contiennt. 
So the how variables are organized on plots are not affected by `arrange()` function. 
* combining `fct_reorder()` and `arrange()` can both changes factor levels of continent and arranges the data in an increading order of gdp_range variable. 

## Part 2: File I/O

#### Explore `write_csv()` and `read_csv` function

I'm going to try writing the re-ordered dataframe to a csv file and then 
reload it. 


```r
re_order <- group_by(gapminder,continent)%>%
            summarize(gdp_range = max(gdpPercap)-min(gdpPercap))%>%
            mutate(continent=fct_reorder(continent,gdp_range))
str(re_order) # look at the structure of re-ordered dataframe
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	5 obs. of  2 variables:
##  $ continent: Factor w/ 5 levels "Africa","Oceania",..: 1 3 5 4 2
##  $ gdp_range: num  21710 41750 113192 48384 24396
```

```r
levels(re_order$continent) # look at the levels of continent variable
```

```
## [1] "Africa"   "Oceania"  "Americas" "Europe"   "Asia"
```

Write and reload .csv files. 

```r
write_csv(re_order,"a1.csv") 
data<-read_csv("a1.csv")
```

```
## Parsed with column specification:
## cols(
##   continent = col_character(),
##   gdp_range = col_double()
## )
```

Look at the structure of reloaded dataframe. 

```r
str(data)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	5 obs. of  2 variables:
##  $ continent: chr  "Africa" "Americas" "Asia" "Europe" ...
##  $ gdp_range: num  21710 41750 113192 48384 24396
##  - attr(*, "spec")=List of 2
##   ..$ cols   :List of 2
##   .. ..$ continent: list()
##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
##   .. ..$ gdp_range: list()
##   .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
##   ..$ default: list()
##   .. ..- attr(*, "class")= chr  "collector_guess" "collector"
##   ..- attr(*, "class")= chr "col_spec"
```

Obviously, the continent variable is not a factor but a list. So `write_csv()\read_csv()` changes the the attributes of factor variable in dataframe. 

#### Explore `saveRDS()` and `readRDS()`

Now let's save re-ordered dataframe to a file and reopen it again, this time using saveRDS()/readRDS().


```r
str(re_order)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	5 obs. of  2 variables:
##  $ continent: Factor w/ 5 levels "Africa","Oceania",..: 1 3 5 4 2
##  $ gdp_range: num  21710 41750 113192 48384 24396
```

```r
levels(re_order$continent)
```

```
## [1] "Africa"   "Oceania"  "Americas" "Europe"   "Asia"
```

Write and reload .rds files.


```r
saveRDS(re_order,"a2.rds")
data<-readRDS("a2.rds")
```

Look at the structure of reloaded dataframe. 


```r
str(data)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	5 obs. of  2 variables:
##  $ continent: Factor w/ 5 levels "Africa","Oceania",..: 1 3 5 4 2
##  $ gdp_range: num  21710 41750 113192 48384 24396
```

```r
levels(data$continent)
```

```
## [1] "Africa"   "Oceania"  "Americas" "Europe"   "Asia"
```

The dataframes are completely identical. Functions `saveRDS()/readRDS()` preserved the order of the factor continent, unlike `write_csv()\read_csv()`.

#### Explore `dput()` and `dget()`

Look at the structure of re-ordered dataframe and levels of continent variables. 


```r
str(re_order)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	5 obs. of  2 variables:
##  $ continent: Factor w/ 5 levels "Africa","Oceania",..: 1 3 5 4 2
##  $ gdp_range: num  21710 41750 113192 48384 24396
```

```r
levels(re_order$continent)
```

```
## [1] "Africa"   "Oceania"  "Americas" "Europe"   "Asia"
```

Write and reload .R files.


```r
dput(re_order,"a3.R")
data<-dget("a3.R")
```

Look at the structure of reloaded dataframe. 


```r
str(data)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	5 obs. of  2 variables:
##  $ continent: Factor w/ 5 levels "Africa","Oceania",..: 1 3 5 4 2
##  $ gdp_range: num  21710 41750 113192 48384 24396
```

```r
levels(data$continent)
```

```
## [1] "Africa"   "Oceania"  "Americas" "Europe"   "Asia"
```

We can see that dataframe before saving is the same as that after loading, so `dput()\dget()` does not destroy any information in dataframe. 

## Part 3: Visualization design

In part I, I use scatter plots to show difference of `arrange()` and `fct_reorder` function. In this part, I will use bar plots to visulize gdp range (difference between maximum gdp per capita and minimum gdp per capita) in different continents. 


```r
# group and summarize gdp range data in different continents
p8 <- group_by(gapminder,continent)%>%
      summarize(gdp_range = max(gdpPercap)-min(gdpPercap))

# bar plot for original data
pl1<-ggplot(p8,aes(x=continent,y=gdp_range))+
     geom_bar(aes(fill=continent),stat="identity",position="dodge")+
     theme(plot.title = element_text(size=14,hjust=0.5),
           axis.text.x = element_text(angle = 45, hjust = 1))+
     labs(x="continent",
          y="gdp_range",
          title="before process")

# arrange gap_range in an increasing order
p9 <- arrange(p8,gdp_range)
# bar plot for arranged data
pl2<-ggplot(p9,aes(x=continent,y=gdp_range))+
     geom_bar(aes(fill=continent),stat="identity",position="dodge")+
     theme(plot.title = element_text(size=14,hjust=0.5),
           axis.text.x = element_text(angle = 45, hjust = 1))+
     labs(x="continent",
          y="gdp_range",
          title="arrange")

# re-order factor levels of continent variable
p10 <- mutate(p8,continent = fct_reorder(continent, gdp_range))
# bar plot for re-ordered data
pl3<-ggplot(p10,aes(x=continent,y=gdp_range))+
     geom_bar(aes(fill=continent),stat="identity",position="dodge")+
     theme(plot.title = element_text(size=14,hjust=0.5),
           axis.text.x = element_text(angle = 45, hjust = 1))+
     labs(x="continent",
          y="gdp_range",
          title="reorder factor levels")

# combine arrange and re-order 
p11<-arrange(p8,gdp_range)%>%
     mutate(continent = fct_reorder(continent, gdp_range))
# bar plot for arrange and fct_reorder
pl4<-ggplot(p11,aes(x=continent,y=gdp_range))+
     geom_bar(aes(fill=continent),stat="identity",position="dodge")+
     theme(plot.title = element_text(size=14,hjust=0.5),
           axis.text.x = element_text(angle = 45, hjust = 1))+
     labs(x="continent",
          y="gdp_range",
          title="arrange and reorder")

# show 4 plots in one page
grid.arrange(pl1,pl2,pl3,pl4,nrow=2)
```

![](hw05-factor_and_figure_management_files/figure-html/unnamed-chunk-23-1.png)<!-- -->

Next, I will compare the difference of ggplot2 graph and plotly graph.

First, I will make a scatterplot to compare life expectanct of China, Canada and Germany using ggplot2.


```r
p12<-filter(gapminder, country %in% c('China','Germany','Canada'))%>%
  select(year,country,lifeExp)
(pl5<-ggplot(p12,aes(x=year,y=lifeExp,color=country))+
      geom_point()+
      geom_line()+
      scale_x_continuous(limits=c(1952,2007),breaks=seq(1952,2007,5))+
      theme(axis.text.x = element_text(angle = 45, hjust = 1),
           plot.title = element_text(size=14,hjust=0.5))+
      labs(x="country",
           y="life expectancy",
           title="Life expectancy of China, Canada, Germany from 1952 to 2007"))
```

![](hw05-factor_and_figure_management_files/figure-html/unnamed-chunk-24-1.png)<!-- -->

Now I want to convert it to plotly graph.

`plotly` is amazing! It allows people to quickly create beautiful, reactive D3 plots that are particularly powerful in websites and dashboards. we can also hover our mouse over the plots and see the data values, zoom in and out of specific regions, and capture stills.

## Part 4: Writing figures to file 

