# Pimp my plot: An intro to plotting in R using ggplot

# Outline

1. Selecting and planning how to plot your data
2. Setting up R studio for plotting
3. Creating a basic plot using gapminder data in ggplot
4. Refining and tweaking your plot

# 1. Selecting and planning your plot

* What type of data do you have?
* What will communicate the trends and meaning you want to highlight in your data?

These two sites provide some useful guidance on which types of plots (or visualisations) are suitable for different data types and different numbers of variables:

* [data-to-viz](https://www.data-to-viz.com/)

* [data viz catalogue](https://datavizcatalogue.com/search.html)

# 2. Setting up R studio for plotting

## Open rStudio desktop or rstudio.cloud

Launch rstudio on your desktop.

If you don't have rstudio installed then go to [rstudio.cloud](https://rstudio.cloud) and login with your Burnet credentials

## Install packages

In rStudio, open and run `libraries.R`. This script will install the packages needed for working with ggplot.

This only needs to be done once for each installation of rStudio. You can skip this step if you already have these packages installed.

## Load the packages and the data

In rStudio, open and run `setup.R`.

# 3. Creating a basic plot using gapminder data in ggplot

## Quick overview of ggplot

The `gg` in `ggplot` stands for grammar of graphics. Think of ggplot as a language for telling the computer how to display your data.

`ggplot` works best when our data is [tidy](https://vita.had.co.nz/papers/tidy-data.pdf). It's good to make it routine to get your data into a tidy format before plotting

Compare the gapminder data (mostly tidy)...

```
head(gapminder)
```

With a "messy" version

```
gapminderMessy <- gapminder %>% select(-pop, -gdpPercap) %>% spread(year, lifeExp)
head(gapminderMessy)
```

## Coding a basic plot

Begin by telling `ggplot` what data to use and define the **aesthetic mappings**. **Aesthetic mappings** describe how variables in the data are mapped to visual properties of plots (eg x values, y values).

Then we can specify the type of plot we want to generate, in this case a **scatter plot** for which we use `geom_point` in `ggplot`.

```
ggplot(df1, aes(x = year, y = lifeExp)) + 
    geom_point()
```

## Changing the plot type

We can keep the code where we define our data and the aesthetic mappings and simply change the type of plot by replacing `geom_point()` with `geom_line()`

```
ggplot(df1, aes(x = year, y = lifeExp)) + 
    geom_line()
```

But the output looks strange because it's drawing a line between all data points, rather than by country. So when we change the plot type we sometimes need to change or add aesthetic mappings

```
ggplot(df1, aes(x = year, y = lifeExp)) + 
    geom_line(aes(group = country, color = continent))
```

# 4. Refining and tweaking your plot

## Manipulating plot elements

`ggplot` allows you to specify how many different elements of the plot should be displayed - colors, size, position

There's a [cheatsheet](https://rstudio.com/wp-content/uploads/2015/03/ggplot2-cheatsheet.pdf) that summarises many of these.

## Using themes
However, rather than trying to customise everything we can also start by using themes to format our plot and then adjust once we find a theme that is close to what we want.

To make our life easier let's assign the basic plot syntax so we don't need to keep repeating it

```
gmPlot <- ggplot(df1, aes(x = year, y = lifeExp)) + 
    geom_line(aes(group = country, color = continent))
```
BW theme
```
gmPlot +
    theme_bw()
```

Mimimal theme

```
gmPlot +
    theme_minimal()
```

The `ggthemes` package includes more themes. First we need to load the package

```
library(ggthemes)
```

Tufte theme
```
gmPlot +
    theme_tufte()
```

Economist theme

```
gmPlot +
    theme_economist()
```

Or make it look like you did your plot in Stata

```
gmPlot +
    theme_stata()
```

## Faceting

Faceting allows you to generate a panel of plots over a variable (usually a categorical). Let's break our gapminder data into a series of plots by continent using `facet_wrap`

```
ggplot(df1, aes(x = year, y = lifeExp)) + 
    geom_line(aes(group = country)) + 
    facet_wrap(~continent)
```

Let's try doing this by World Bank Income Classification. This following section is a bit fiddly but what it does it gets the world bank classification(current) and joins it to our gapminder data in a new dataframe `df2`

```
# We use `gdata` to import excel data
install.packages("gdata")
library(gdata)

# Import the excel data
wbclass <- read.xls("http://databank.worldbank.org/data/download/site-content/CLASS.xls", skip = 2, blank.lines.skip = TRUE)

# And then we drop the blank first row using dplyr
library(dplyr)
wbclass <- wbclass %>%
    filter(Economy != "x") %>%
    # Select only the vars we need and rename Economy to country to make the join easier
    select(country = Economy, Income.group) 

# Join the classification to the gapminder data
df2 <- left_join(df1, wbclass, by = "country" )
```

Now we can do a facet grid where we split plots into panels over two vars
```
ggplot(df2, aes(x = year, y = lifeExp)) + 
    geom_line(aes(group = country)) + 
    facet_grid(Income.group~continent)
```

A further improvement would be make `Income.group` an ordered factor ie High Income > Upper middle income > Lower middle income > Low income. Then the grid would display income in the correct order.

# Resources

## Guides to how to choose the right visualisation

* [data-to-viz](https://www.data-to-viz.com/)

* [data viz catalogue](https://datavizcatalogue.com/search.html)

## Principles of good data visualisation

* [Essentials of data visualization](http://mkweb.bcgsc.ca/essentials.of.data.visualization/) is an 8-part mini series covering the basics

* *The visual display of quantitative information* by Edward Tufte is a good intro

* *The functional art* and *The truthful art* by Albert Cairo. Available through Safari Books - [The functional art]( https://learning.oreilly.com/library/view/the-functional-art/9780133041187/) and [The truthful art](https://learning.oreilly.com/library/view/the-truthful-art/9780133440492/). Monash staff have access to safari books. I suspect it's the same for University of Melbourne

## Importing stata data sets

You can use `R` for plotting while continuing to use `Stata` for data cleaning and analysis. The easiest way to do this is to save your `Stata` data. You can use the `haven` package to read `Stata` data sets.

```
# Run this line if you don't already have `haven` or `tidyverse` installed
install.packages("haven")

# Load the library
library(haven)

# Example import syntax
mPlotData <- read_dta("/Users/Myname/Mydata.dta")

# Plot the data
ggplot(myPlotData, aes(x = var1, y = var2)) + geom(point)
```

The [documentation on read_dta](https://haven.tidyverse.org/reference/read_dta.html) lists the other arguments/options for importing a `.dta` file.