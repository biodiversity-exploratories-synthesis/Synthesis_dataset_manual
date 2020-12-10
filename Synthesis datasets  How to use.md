# Synthesis datasets : How to use?

The aim of this manual is to help researchers using the synthesis datasets of the Biodiversity Exploratories. It gives tips on how to handle the datasets and outlines the common issues when analysing the data.



We strongly recommend reading the metadata of the original datasets and contact their authors to understand their limitations. The Bexis identifier (DataID) and version (Dataversion) of each dataset are provided in the synthesis dataset for reproducibility.



For any suggestion on this manual or on the synthesis datasets please contact us by email or open an issue on [GitHub](https://github.com/biodiversity-exploratories-synthesis). 



## Organisation

### Diversity datasets

The diversity synthesis datasets are organised in two separate files:

1. Assembled RAW diversity (~350Mb): contain information on plots, species, value, type (either abundance, presence-absence, OTU_number, ASV_number or cover), year of measurement, Bexis DataID and data version.

   Currently these files are:

   - Grasslands :  [27707 “Assembled RAW diversity from grassland EPs (2008-2020) for multidiversity synthesis - November 2020”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27707)
   - Forests : [24607 “Assembled RAW diversity from forest EPs (2007-2015) for multidiversity synthesis”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=24607)

2. Assembled species information (~20Mb): contain information on Trophic levels, functional and taxonomic grouping

   These two files can be merged using the common column “Species”.

   Currently these files are:

   - Grasslands : [27706 “Assembled species information from grassland EPs (2008-2020) for multidiversity synthesis - November 2020”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27706)
   - Forests : [24608 “Assembled species information from forest EPs (2007-2015) for multidiversity synthesis”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=24608)

### Function datasets

The functions synthesis datasets are organised in three separate files:

1. Assembled RAW functions: contains entries for each function for each year. As not every function has been measured every year, many entries are missing (these are the unmeasured year-function combinations). The metadata of this file is minimal and is thought to be used with the separately stored metadata (see point 3 below).

   Currently this dataset is: [27087 “Assembled ecosystem measures from grassland EPs (2008-2018) for multifunctionality synthesis - June 2020”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27087)

2. A script: this is a script to reformat the previous file to the wide format (plots x functions) that is easily usable for analysis. It also removes the unmeasured year-function combinations. 

   Currently this script is: [27626 “R Script to Reformat Dataset 27087 "Assembled ecosystem measures from grassland EPs" to easily usable wide format”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27626)

3. An additional metadata file : contains a table with detailed metadata for all functions.

   Currently this file is: [27088 “Additional metadata of dataset 27087: Assembled ecosystem measures from grassland EPs (2008-2018) for multifunctionality synthesis - June 2020”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27088)

Note that the forest functions dataset has not been updated to this format yet and is stored in a Plot X Functions format. Currently this dataset is: [24367 “Raw data of forest attributes of forest EPs of the Exploratories project used in "Multiple forest attributes underpin the supply of multiple ecosystem services"](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=24367).



## Datasets size and memory issues

The diversity and function synthesis datasets can be very large and this might create memory problems. We recommend the use of the package `data.table` in `R` to handle the dataset.

```R
#Example code to read the raw diversity and species information files and merge:
library(data.table)
grl <- fread("~/27707.txt")
tr<-fread("~/27706.txt")
grassland.dataset<-merge(grl,tr,by="Species")
```



## Wide and long format

The synthesis datasets are stored in the long format; however, most analyses are done in the wide format (plots x species, plots x groups, plots x functions), see Fig.1. To transform from long to wide format you can e.g. use the `dcast` (`reshape2`), `dcast.data.table` (`data.table`) or `pivot_wider` (`tidyr`) functions in `R`. Be careful, some functions insert NAs (or other values) when creating a wide-format table and this should be carefully checked. Some functions allow choosing the values to fill (e.g. argument `“fill”` in `dcast.data.table`).

```R
#Example code to reshape to the wide format and fill with zeros:

library(data.table)

dat.wide<-dcast.data.table(dat,Species~Plot,value.var="value",fill=0)
```

![Figure 1](https://lh5.googleusercontent.com/jGNmDUC_klYpHjIpPURtILnwqn3vGbxOXkok9qT-dyiwgdRyNhNRXL69I1oOUgaEPAfr5MG2pa3MTA4Ilhxtz7a5N56_7UZdNM1gkuloK4r7lSvh1Uo6z2itXvfPAoBvbg "test caption")

