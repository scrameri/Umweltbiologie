## Praktikum Umweltbiologie - Ecological Genetics

### Tutorial on Data Exploration

#### Install packages if needed

```R
need.pckg <- c("GGally","ggplot2","EnvStats") # needed for this script
if (any(!need.pckg %in% installed.packages())) {
  for (i in need.pckg[!need.pckg %in% installed.packages()]) {
    cat("installing", i, "...\n")
    install.packages(i)
  }
}
```

#### Load libraries

```R
library("GGally") # for ggpairs()
library("ggplot2") # for ggplot()
library("EnvStats") # for qqPlot()
```

#### Read data
```R
morph <- read.csv("Puzzle_ANOVA_LM.csv", header = TRUE)
```

#### Get an overview over the dataset

```R
str(morph) # lists all variables in the data.frame and their classes
summary(morph) # gives a summary for each variable 
summary(morph$Staengel_hoehe) # gives a summary for one variable
```

This dataset contains morphological measurements of eight populations of *Dianthus carthusianorum* taken in Summer 2015 in Wallis (Switzerland). Populations are located in two classes of elevation (4 high, 4 low elevation).

A few facts about *D. carthusianorum*
- Karthäuser-Nelke, family Caryophyllaceae like Silene
- native to Middle Europe, in dry habitats from colline to alpine (introduced to N America) 
- gynodioecious, meaning that there are female and hermaphroditic individuals
- perennial (or biennial), meaning that plants survive winter as rosettes or seeds.
- insect-pollinated
- parasite: fungus Microbothryum dianthorum (anther smut fungus) that causes sterility and is transmitted by pollinators

A few facts about the dataset
- Individuum: individual ID
- Population: population ID
- Hoehe: elevation class (hoch = subalpine, tief = colline)
- Datum: measurement date
- Infektion: whether a plant was infected (infiziert) or healthy (gesund)
- Geschlecht: herm(aphroditic) versus female (weiblich)
- Staengel_hoehe: mean stalk height (mm)
- Staengel_anz: number of stalks per plant
- Knospen_anz: number of buds per plant
- Blueten_anz: number of open flowers per plant
- Blueten_durchm: flower (corolla) diameter (mm)
- Kronblatt_laenge: petal length (mm)
- Kronblatt_breite: petal width (mm)
- Kelch_laenge: sepal length (mm)
- Kelch_breite: sepal width (mm)
- Rosetten_durchm1: diameter of first rosette (mm) - only recorded at high elevation
- Rosetten_durchm2: diameter of second rosette (mm) - only recorded at high elevation

The experimental design and measurements have been done by Ursina Walther (a PhD student in our group). She was interested in studying the evolution of floral traits in this species, especially in relation to the interaction between the plants and their Microbothryum parasite.

#### Question 1: 
Based on what you now know about the study and the data
- what are the response variable(s)?
- what are the explanatory variable(s)?

#### Question 2:
There are factors and numerical variables in the data. If you want to plot one or two of them, which plots do you use for which (combination of) variables?

Use *Scatterplots* to plot two numeric variables against each other
```R
# base
plot(Kelch_laenge ~ Kronblatt_laenge, data = morph)

# ggplot2
ggplot(data = morph, aes(x = Kronblatt_laenge, y = Kelch_laenge)) + geom_point()
```
NOTE: scatterplots can be more informative if different sizes and symbols are used.

Use *Boxplots* to plot a numeric variable against a factor variable
```R
# base
boxplot(Kelch_laenge ~ Hoehe, data = morph)

# ggplot2
ggplot(data = morph, aes(x = Hoehe, y = Kelch_laenge)) + geom_boxplot()
```
NOTE: boxplots are summarizing the data and should not be used if you have less than eight data points per factor level.


Use *Histograms* to plot a variable's distribution
```R
# base
hist(morph$Kelch_laenge, breaks = 20)

# ggplot2
ggplot(data = morph, aes(x = Kelch_laenge)) + geom_histogram(bins = 20) 
```
NOTE: the argument ```breaks``` can be used to fine-tune the binning of values into histogram categories.

Use *Barplots* to plot all values of a single variable or a table of counts
```R
# base
barplot(table(morph$Population))

# ggplot2
ggplot(data = morph, aes(x = Population)) + geom_bar(stat = "count")
```

Use *Mosaic Plots* or *Stacked Barplots* to plot contingency tables of two factor variables against each other
```R
# base
mosaicplot(table(morph$Hoehe,morph$Infektion), main = "Mosaicplot")

# ggplot2
ggplot(morph, aes(x = Hoehe, fill = Infektion)) + 
  geom_bar(position = "fill") + 
  labs(y = "Proportion") +
  ggtitle("Stacked Barplot")

```

Since we have 17 variables, plotting all of them against each other would be tedious. For datasets with a moderate number of variables, you can use the ggpairs() function to get a graphical overview over many or all variables at once with a single line of code. With 17 variables, plotting all against all would lead to too many (289) plots, so let us subset the variables.

You can use ```grep``` and **regular expressions** to find the indices of certain variable names, this often saves code.

```R
# this returns the index of variable names *starting* with 'Kron' (^ specifies the *start*)
grep(pattern = "^Blueten_", x = names(morph))

# this returns the index of variable names *ending* with 'breite' ($ specifies the *end*)
grep(pattern = "breite$", x = names(morph)) 

# if you put a line in (), the returned value will be printed to the console (saves a print() call)
(vars.fertile <- names(morph)[grep(pattern = "^Blueten|^Knospen|^Kron|^Kelch", x = names(morph))])
(vars.sterile <- names(morph)[grep(pattern = "^Staengel|^Rosetten", x = names(morph))])
```

This prepares the **Pairs Plots** (creates gg class objects that can be printed)
```R
pairs.fertile <- ggpairs(data = morph, columns = c("Population","Hoehe","Infektion","Geschlecht", vars.fertile))
pairs.sterile <- ggpairs(data = morph, columns = c("Population","Hoehe","Infektion","Geschlecht", vars.sterile))
```

This saves a .pdf file of the specified size
```R
pdf("Pairsplots.pdf", height = 15, width = 15)
pairs.sterile # or print(pairs.sterile)
pairs.fertile # or print(pairs.fertile)
graphics.off()
```

#### Check the distribution of your variables
Knowing the distribution of your variables is important, especially if you want to create models of them that make certain assumptions on these distributions. You have already seen that the **histogram** is a good visualization of a variable's distribution because deviations from a normal distribution become apparent.

But which statistical distribution is best at describing your data if not a normal distribution? Simulations can help here, and the ```R``` package *EnvStats* allows for simulation of different probability distributions, and comparison of these against your data in so-called **QQ Plots** (Quantile-Quantile plots).

Let us explore the distribution of the variable ```Staengel_anz``` (stalk count):
```R
par(mfrow=c(2,2))
hist(morph$Staengel_anz, breaks = 20, col = "tomato")
qqPlot(morph$Staengel_anz, distribution = "pois", estimate.params = TRUE, add.line = TRUE, points.col = "chocolate")
qqPlot(morph$Staengel_anz, distribution = "gamma", estimate.params = TRUE, add.line = TRUE, points.col = "cadetblue")
qqPlot(morph$Staengel_anz, distribution = "norm", estimate.params = TRUE, add.line = TRUE, points.col = "purple")
par(mfrow=c(1,1))
```

#### Question 3:
Which probability distribution is best at describing stalk count? What does this mean for a model ```Staengel_anz ~ Hoehe``` fitted to this data?
