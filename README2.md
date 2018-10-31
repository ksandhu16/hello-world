
<!-- README.md is generated from README.Rmd. Please edit that file -->

# <img src="https://i.imgur.com/vTLlhbp.png" align="right" height=88 /> Calculate Abundance and Basal Area

[![lifecycle](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![Travis build
status](https://travis-ci.org/forestgeo/fgeo.abundance.svg?branch=master)](https://travis-ci.org/forestgeo/fgeo.abundance)
[![Coverage
status](https://codecov.io/gh/forestgeo/fgeo.abundance/branch/master/graph/badge.svg)](https://codecov.io/github/forestgeo/fgeo.abundance?branch=master)
[![CRAN
status](https://www.r-pkg.org/badges/version/fgeo.abundance)](https://cran.r-project.org/package=fgeo.abundance)

## Installation

Installation will require the pre-release version of **fgeo.abundance**:

```R
# install.packages("devtools")
devtools::install_github("forestgeo/fgeo.abundance@pre-release")
```
Or simply install the development version of **fgeo.abundance**:

```R
# install.packages("devtools")
devtools::install_github("forestgeo/fgeo.abundance")
```

Or install all **fgeo** packages in one step by using the following link:
(https://forestgeo.github.io/fgeo/index.html#installation)

For details on how to install packages from GitHub, visit the following website to review their article: (https://goo.gl/dQKEeg)

## Example

```{r}
library(dplyr)
library(fgeo.tool)
library(fgeo.abundance)
```

### Abundance

If the trees HAVE buttresses your data may have multiple stems per treeid and even multiple measures per stemid.

```{r}
# Trees with buttresses may have multiple measurements of a single stem. 
# Main stems will always have highest `HOM`, then largest `DBH`.

vft <- tribble(
  ~CensusID, ~TreeID, ~StemID, ~DBH, ~HOM,
          1,     "1",   "1.1",   88,  130,
          1,     "1",   "1.1",   10,  160,  # Main stem
          1,     "2",   "2.1",   20,  130,
          1,     "2",   "2.2",   30,  130,  # Main stem
)
```

Fundamentally, `abundance()` counts the number of rows. All of these results are the same:

```{r}
nrow(vft)
dplyr::count(vft)
dplyr::summarize(vft, n = n())
abundance(vft)
```
The result isn’t always likely what you expect, instead it would look more like:

```{r}
summarize(vft, n = n_distinct(TreeID))
```

As shown above, you can get a correct result by combining `summarize()` and `n_distinct()` (from the __dplyr__ package). The function `abundance()` on the other hand includes some useful additional features (see `?abundance()`). This code more clearly conveys your intention, i.e. to calculate tree abundance by counting the number of main stems:

```{r}
(main_stems <- pick_main_stem(vft))
abundance(main_stems)
```

If you have data from multiple censuses, you can compute by census (or any other group).

```{r}
vft2 <- tibble::tribble(
  ~CensusID, ~TreeID, ~StemID, ~DBH, ~HOM,
          1,     "1",   "1.1",   10,  130,
          1,     "1",   "1.2",   20,  130,  # Main stem
          2,     "1",   "1.1",   12,  130,
          2,     "1",   "1.2",   22,  130   # Main stem
)

by_census <- group_by(vft2, CensusID)
(main_stems_by_census <- pick_main_stem(by_census))
abundance(main_stems_by_census)
```

Often you will need to first subset data (e.g. by `status` or `DBH`) and then count.

```{r}
over20 <- filter(main_stems_by_census, DBH > 20)
abundance(over20)
```

### Basal area

If trees have buttresses, you may need to pick the main stemid of each stem, so you don't count the same stem more than once.

```{r}
vft3 <- tribble(
  ~CensusID, ~TreeID, ~StemID, ~DBH, ~HOM,
          1,     "1",   "1.1",   88,  130,
          1,     "1",   "1.1",   10,  160,  # Main stem
          1,     "2",   "2.1",   20,  130,
          1,     "2",   "2.2",   30,  130,  # Main stem

          2,     "1",   "1.1",   98,  130,
          2,     "1",   "1.1",   20,  160,  # Main stem
          2,     "2",   "2.1",   30,  130,
          2,     "2",   "2.2",   40,  130,  # Main stem
)

(main_stemids <- pick_main_stemid(vft3))
main_stemids

basal_area(main_stemids)

```

`basal_area()` also allows you to compute by groups.

```{r}
by_census <- group_by(main_stemids, CensusID)
basal_area(by_census)
```

However if you want to compute on a subset of data, you need to first pick the data.

```{r}
ten_to_twenty <- filter(by_census, DBH >= 10, DBH <= 20)
ba <- basal_area(ten_to_twenty)
ba
```

And if you need to convert units, you can do so after the fact with `fgeo.tool::convert_unit_at()`.

```{r}
convert_unit_at(ba, .at = "basal_area", from = "mm2", to = "hectare")
```

### Summaries Aggregated by Year

Example data.

```{r}
vft <- example_byyr
vft
```

Abundance by year.

```{r}
abundance_byyr(vft, DBH >= 10, DBH < 20)

abundance_byyr(vft, DBH >= 10)
```

Basal area by year.

```{r}
basal <- basal_area_byyr(vft, DBH >= 10)
basal

# Convert units and standardize by plot size in hectares
years <- c("yr_2001", "yr_2002")
in_he <- convert_unit_at(basal, .at = years, from = "mm2", to = "hectare")
standardize_at(in_he, .at = years, denominator = 50)
```

To get started visit the following link:
(https://forestgeo.github.io/fgeo/articles/fgeo.html)

## Information

* [Getting help](SUPPORT.md).
* [Contributing](CONTRIBUTING.md).
* [Contributor Code of Conduct](CODE_OF_CONDUCT.md).
