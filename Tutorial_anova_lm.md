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

This dataset contains morphological measurements of eight populations of [*Dianthus carthusianorum*](https://www.infoflora.ch/de/flora/dianthus-carthusianorum.html) taken in Summer 2015 in Wallis (Switzerland). Populations were located in two classes of elevation (4 high, 4 low elevation).

A few facts about [*D. carthusianorum*](https://de.wikipedia.org/wiki/Kart%C3%A4usernelke)
- Karthäuser-Nelke in German, family Caryophyllaceae like *Silene*
- native to Middle Europe, in dry habitats from colline to alpine (introduced to N America) 
- [gynodioecious](https://en.wikipedia.org/wiki/Gynodioecy), meaning that there are female and hermaphroditic individuals
- perennial (or biennial), meaning that plants survive winter as rosettes or seeds
- insect-pollinated
- parasite: fungus [*Microbothryum*](https://fr.wikipedia.org/wiki/Microbotryum) *dianthorum* (anther smut fungus) that causes sterility and is transmitted by pollinators

A few facts about the dataset
- ID: individual ID
- Population: population ID
- Elevation: elevation class
- Date: measurement date
- Infection: whether a plant was [infected](https://upload.wikimedia.org/wikipedia/commons/e/ef/Microbotryum_dianthorum_Dianthus_sp._2019_07_18_05.jpg) or healthy
- Sex: whether a plant was hermaphroditic or female
- Stalk_height: mean stalk height per plant (mm)
- Stalk_count: number of stalks per plant
- Bud_count: number of buds per plant
- Flower_count: number of open flowers per plant
- Flower_diam: flower (corolla) diameter (mm)
- Petal and Sepal lengths and widths (mm)
- Rosette_diam1: diameter of first rosette (mm) - only recorded at high elevation
- Rosette_diam2: diameter of second rosette (mm) - only recorded at high elevation

The experimental design and measurements have been done by [Ursina Walther](https://peg.ethz.ch/people/person-detail.html?persid=158239) (a PhD student in our group). She was interested in studying the evolution of floral traits in this species, especially in relation to the interaction between the plants and their Microbothryum parasite.

![logo] Based on what you now know about the study and the data:
- what are the response variable(s)?
- what are the explanatory variable(s)?

***

#### Plot the data
Use **Scatterplots** to plot two numeric variables against each other.
```R
# base
plot(Kelch_laenge ~ Kronblatt_laenge, data = morph)

# ggplot2
ggplot(data = morph, aes(x = Kronblatt_laenge, y = Kelch_laenge)) + geom_point()
```
NOTE: scatterplots can be more informative if different symbols for an additional factor and sizes for an additional continuous variable are used.

Use **Boxplots** to plot a numeric variable against a factor variable.
```R
# base
boxplot(Sepal_length ~ Elevation, data = morph)

# ggplot2
ggplot(data = morph, aes(x = Elevation, y = Sepal_length)) + geom_boxplot()
```
NOTE: boxplots are summarizing the data and should not be used if you have less than eight data points per factor level.


Use **Histograms** to plot a variable's distribution.
```R
# base
hist(morph$Sepal_length, breaks = 20)

# ggplot2
ggplot(data = morph, aes(x = Sepal_length)) + geom_histogram(bins = 20)  
```
NOTE: the argument ```breaks``` can be used to fine-tune the binning of values into histogram categories.

Use *Barplots* to plot all values of a single variable or a table of counts.
```R
# base
barplot(table(morph$Population))

# ggplot2
ggplot(data = morph, aes(x = Population)) + geom_bar(stat = "count")
```

Use **Mosaic Plots** or **Stacked Barplots** to plot contingency tables of two factor variables against each other
```R
# base
mosaicplot(table(morph$Elevation,morph$Infection), main = "Mosaicplot")

# ggplot2
ggplot(morph, aes(x = Elevation, fill = Infection)) + 
  geom_bar(position = "fill") + 
  labs(y = "Proportion") +
  ggtitle("Stacked Barplot")

```

Given that we have 17 variables, plotting all of them against each other would be tedious. For datasets with a moderate number of variables, you can use the ```ggpairs``` function from the *GGally* package to get a graphical overview over many or all variables at once with a single line of code. The function takes care of factors and numerical variables automatically.

With 17 variables, plotting all against all would lead to too many (289) plots. Let us therefore subset the variables. You can use ```grep``` and [**regular expressions**](https://regex101.com/) to find the indices of certain variable names, this often saves code.

```R
# this returns the index of variable names *starting* with 'Kron' (^ specifies the *start*)
grep(pattern = "^Flower_", x = names(morph))

# this returns the index of variable names *ending* with 'breite' ($ specifies the *end*)
grep(pattern = "width$", x = names(morph)) 

# if you put a line in (), the returned value will be printed to the console (saves a print() call)
(vars.fertile <- names(morph)[grep(pattern = "^Flower|^Bud|^Petal|^Sepal", x = names(morph))])
(vars.sterile <- names(morph)[grep(pattern = "^Stalk|^Rosette", x = names(morph))])
```

This prepares the **Pairs Plots** (creates gg class objects that can be printed)
```R
pairs.fertile <- ggpairs(data = morph, columns = c("Population","Elevation","Infection","Sex", vars.fertile))
pairs.sterile <- ggpairs(data = morph, columns = c("Population","Elevation","Infection","Sex", vars.sterile))
```

This saves a .pdf file of the specified size
```R
pdf("Pairsplots.pdf", height = 15, width = 15)
pairs.sterile # or print(pairs.sterile)
pairs.fertile # or print(pairs.fertile)
graphics.off()
```

![logo]
- Can you produce a plot of ```Petal_length ~ Sepal_length``` with points color-coded for ```Infection```?

***

#### Check the distribution of your variables
Knowing the distribution of your variables is important, especially if you want to create models of them that make certain assumptions on distributions. You have already seen that the **histogram** is a good visualization of a variable's distribution because deviations from a normal distribution become intuitively apparent.

But which statistical distribution is best at describing your data if not a normal distribution? Simulations can help here, and the ```R``` package *EnvStats* allows for simulation of different probability distributions, and comparison of these against your data in so-called **QQ Plots** (Quantile-Quantile plots).

Let us explore the distribution of the variable ```Stalk_count```:
```R
par(mfrow=c(2,2))
hist(morph$Stalk_count, breaks = 20, col = "tomato")
qqPlot(morph$Stalk_count, distribution = "pois", estimate.params = TRUE, add.line = TRUE, points.col = "chocolate")
qqPlot(morph$Stalk_count, distribution = "gamma", estimate.params = TRUE, add.line = TRUE, points.col = "cadetblue")
qqPlot(morph$Stalk_count, distribution = "norm", estimate.params = TRUE, add.line = TRUE, points.col = "purple")
par(mfrow=c(1,1))
```

![logo]
- Which probability distribution is best at describing ```Stalk_count```? 
- What does this mean for an ANOVA model ```Stalk_count ~ Elevation``` fitted to this data?


[logo]: https://img.icons8.com/flat_round/64/000000/question-mark.png "Question"
