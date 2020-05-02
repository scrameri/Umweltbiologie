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
