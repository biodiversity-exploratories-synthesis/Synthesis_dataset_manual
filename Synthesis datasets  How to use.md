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

![img](./images/Synthesis datasets  How to use-Fig1)

**Fig1**: *illustration of the long format (mostly used to store and assemble data from different sources) and wide format (mostly used to type, visualise data or for data analysis)*



## Grouping (e.g. into trophic level)

### Diversity datasets

Not all groups were measured in all plots. When grouping several taxonomic groups into a given trophic level (or any other grouping, e.g. grouping multiple years), we recommend to always check if all groups/species were measured in the same plots and only use the plots where all groups/species were measured. Otherwise the calculated richness or other diversity estimates per group will be biased. We also recommend checking again the number of available plots before analysing data. The number of available plots can be found in the metadata of each dataset or can be retrieved from the synthesis dataset using the DataID column.

```R
#Example code to check which datasets have information on less than 150 plots, using the Bexis dataset ID:
dat<-dat[!is.na(value)] #remove all NAs
for (i in sort(unique(dat$DataID))){
tt<-dat[DataID==i]
if(length(unique(tt$Plot))!=150)

print(paste(i,":",length(unique(tt$Plot)),unique(tt$Trophic_level),",",unique(tt$Group_broad)))
}

#Another example using Trophic level and checking the number of plots X species combinations:
for (i in unique(dat$Trophic_level)){
tt<-dat[Group_broad==i]
if(length(unique(tt$Plot))*length(unique(tt$Species))*length(unique(tt$Year))!=nrow(tt))

print(paste(i,":",length(unique(tt$Plot))*length(unique(tt$Species)),"/",nrow(tt)))
}

```

Not all groups were measured with the same methods. When grouping taxa that have several hundreds of species (e.g. OTU data) with taxa with very few species, the information about taxa with lower richness will likely be “hidden” by the other one. When species numbers in different groups are of similar range, grouping to calculate richness is mostly fine. Grouping to calculate abundances (e.g. total number of individuals in a given plot) can be more sensitive to the methods of collection and should be done very carefully.



### Functions datasets

This also applies to the functions datasets when combining multiple functions. Additionally, it is worth mentioning that besides raw data, some functions are already assembled from several years and some functions are combined. This is indicated as “assembled data” in the column “DataID” of the metadata table (see 3. of the [Organisation](#Organisation) paragraph). The method of assembling (e.g. mean over years) is described in the “calculation” column.