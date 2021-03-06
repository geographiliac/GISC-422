#### GISC 422 T1 2020
First just make sure we have all the data and libraries we need set up.
```{r}
library(sf)
library(ggplot2)
library(tidyr)
library(dplyr)

sfd <- st_read('sf_demo.geojson')
sfd <- drop_na(sfd)
sfd.d <- st_drop_geometry(sfd)
```
# Introducing the `tidyverse`
When it comes to dealing with complicated datasets, it is really helpful to have a more systematic way of dealing with such datasets. Base *R* provides this OK but gets rather messy very quickly. An alternative approach is provided by the [`tidyverse`](https://www.tidyverse.org/)
 a large collection of packages for handling data in a 'tidy' way, and with an associated powerful set of plotting libraries. This document can only look at these very quickly, but will hopefully begin to give you a flavour of what is possible, and encourage you to explore further if you need to.
 
Like *R* itself the `tidyverse` is largely inspired by the work of another New Zealander, [Hadley Wickham](http://hadley.nz/)... Aotearoa represent!

We can't really get into the philosophy of it all here. Instead we focus on some key functionality provided by functions in the `dplyr` package. We will also look quickly at processing pipelines using the `>%>` or 'pipe' operator. We'll round things off with a quick look at `ggplot2`. 

## `select` 
A common requirement in data analysis is selecting only the data attributes you want, and getting rid of all the other junk. The `sfd` dataset has a lot going on. A nice tidy tool for looking at data is `as_tibble()`
```{r}
as_tibble(sfd)
```

This shows us that we have 25 columns in our dataset (one of them is the geometry). We can get a list of the names with `names()`
```{r}
names(sfd)
```

Selecting only columns of interest is easy, using the `select` function, we simply list them
```{r}
select(sfd, density, PCdoctorate, perCapitaIncome)
```

This hasn't changed the data, we've just looked at a selection from it. But we can easily assign the result of the selection to a new variable
```{r}
sfd.3 <- select(sfd, density, PCdoctorate, perCapitaIncome)
```

What is nice about `select` is that it provides lots of different ways to make selections. We can list names, or column numbers, or use colons to include all the columns between two names or two numbers, or even use a minus sign to drop a column. And we can use these (mostly) in all kinds of combinations. For example
```{r}
select(sfd, 1:2, PClessHighSchool, PCraceBlackAlone:PCforeignBorn)
```

or
```{r}
select(sfd, -(1:10))
```

Note that here I need to put `1:10` in parentheses so it knows to remove all the columns 1 to 10, and doesn't start by trying to remove a (non-existent) column number -1.

### Selecting rows
We look at filtering based on data in the next section. If you just want rows, then use `slice()`
```{r}
slice(sfd, 2:10, 15:25)
```

## `filter`
Another common data tidying operation is selection based on the attributes of the data. This is called `filter`ing in the `tidyverse`. We provide a filter specification, usually data based to perform such operations
```{r}
filter(sfd, density > 0.3)
```

If we want data that satisfy more than one filter, we simply include combine the filters with **and** `&` and **or** `|` operators
```{r}
filter(sfd, (density > 0.1 & perCapitaIncome > 0.1) | PClessHighSchool > 0.5)
```

Using select and filter in combination, we can usually quickly and easily reduce large complicated datasets down to the parts we really want to look at. We'll see a lit bit later how to chain operations together into processing pipelines. First, one more tool is really useful, `mutate`.

## `mutate`
Selecting and filtering data leaves things unchanged. Often we want to combine columns, in various ways. This option is provide by the `mutate` function
```{r}
mutate(sfd, x = density + PCwithKids)
```

This has added a new column to the data by adding together the values of two other columns (in this case, it was a meaningless calculation, but you should easily be able to imagine other examples that would make sense!)

## Combining operations into pipelines
Something that can easily become tedious is this kind of thing (not executable code, but you get the idea)

    a <- select(y, ...)
    b <- filter(a, ...)
    c <- mutate(b, ...)
    
and so on. Normally to combine these operations into a single line you would do something like this

    c <- mutate(filter(select(y, ...), ...), ... )
    
but this can get very confusing very quickly, because the order of operations is opposite to the order they are written, and keeping track of all those opening and closing parentheses is error-prone. The tidyverse introduces a 'pipe' operator `%>%` which (once you get used to it) simplifies things greatly. Instead of the above, we have

    c <- y %>% select(...) %>% filter(...) %>% mutate(...)
    
This reads "assign to c the result of passing y into select, then into filter, then into mutate". Here is a nonsensical example with the `sfd` dataset, combining operations from each of the previous three sections
```{r}
sfd %>%
  select(1:10) %>%
  slice(10:50) %>%
  filter(density > 0.1) %>%
  mutate(x = density + PCcommutingNotCar)
```

## Tidying up plotting with `ggplot2`
Another aspect to the tidyverse is a more consistent approach to plotting, particularly if you are making complicated figures. We've already seen an example of this in the previous document. Here it is again
```{r}
ggplot(sfd) +
  geom_point(aes(x=density,
                 y=medianYearBuilt,
                 colour=PConeUnit,
                 size=PCownerOccUnits), alpha=0.5)
```

What's going on here?! The idea behind `ggplot2` functions is that there should be an *aesthetic mapping* between each data attribute and some graphical aspect. This idea is discussed in [this paper](http://vita.had.co.nz/papers/layered-grammar.pdf). We've already seen a version of it in `tmap` when we specify `col=` for a map variable. The above example is more complete implementation of the idea. The `ggplot` function specifies the dataset, an additional layer is specified by a geometry function, in this case `geom_point`, for which we must specify the aesthetic mapping using `aes()` telling which graphical parameters, x and y location, colour and size are linked to which data attributes.

It is worth knowing that `ggplot` knows about `sf` data, and so can be used as an alternative to `tmap`. This is a big topic, and I only touch on it here so I can used `ggplot` functions from time to time without freaking everybody out! Happy to discuss further if this is a topic that interests or excites you.
```{r}
ggplot(sfd) +
  geom_sf(aes(fill=density)) + 
  scale_fill_distiller(palette='Reds', direction=1)
```

Now let's get back to multivariate data. Go to [this document](03-dimensional-reduction.md).