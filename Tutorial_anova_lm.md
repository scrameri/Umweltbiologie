## Praktikum Umweltbiologie - Ecological Genetics

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
