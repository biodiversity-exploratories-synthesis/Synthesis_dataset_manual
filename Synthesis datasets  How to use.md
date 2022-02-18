# Synthesis datasets : How to use?

The aim of this manual is to help researchers using the synthesis datasets of the Biodiversity Exploratories. It provides information and tips on how to handle the datasets and outlines the common issues when analysing the data.



We strongly recommend reading the metadata of the original datasets and contact their authors to understand their limitations. The Bexis identifier (DataID) and version (Dataversion) of each contributing dataset are provided in the synthesis dataset for reproducibility.

For specific information on the treatment and operations made on the single datasets included in the synthesis datasets, please refer to the scripts available in our [GitHub repository](https://github.com/biodiversity-exploratories-synthesis) (e.g. [Grassland diversity datasets](https://github.com/biodiversity-exploratories-synthesis/Synthesis_dataset_diversity_grassland)) or contact us.




For any suggestion on this manual or on the synthesis datasets please contact us by email or open an issue on [GitHub](https://github.com/biodiversity-exploratories-synthesis/Synthesis-dataset-manual/issues). 


<sup><sub>*Table of Contents* : the below `[toc]` generates a Table of Content. On GitHub, the table of contents can be found at the upper left corner of this file.</sup></sub>

[toc]

## Organisation

### Diversity datasets

The diversity synthesis datasets are organised in two separate files:

1. Assembled RAW diversity (~350Mb): contain information on plots, species, value, type (either abundance, presence-absence, OTU_number, ASV_number or cover), year of measurement, Bexis DataID and data version.

   Currently these files are:

   - Grasslands :  [27707 “Assembled RAW diversity from grassland EPs (2008-2020) for multidiversity synthesis - November 2020”](https://www.bexis.uni-jena.de/ddm/data/Showdata/27707)
   - Forests : [31206 “Assembled RAW diversity from forest EPs (2007-2020) for multidiversity synthesis -  January 2022”](https://www.bexis.uni-jena.de/ddm/data/Showdata/31206)

2. Assembled species information (~20Mb): contain information on Trophic levels, functional and taxonomic grouping

   These two files can be merged using the common column “Species”.

   Currently these files are:

   - Grasslands : [27706 “Assembled species information from grassland EPs (2008-2020) for multidiversity synthesis - November 2020”](https://www.bexis.uni-jena.de/ddm/data/Showdata/27706)
   - Forests : [31207 “Assembled species information from forest EPs (2007-2020) for multidiversity synthesis - January 2022”](https://www.bexis.uni-jena.de/ddm/data/Showdata/31207)

### Function datasets

The functions synthesis datasets are organised in three separate files, all part of the same DatasetID 27087:

1. The dataset itself (e.g. `27087_16_data.csv`) Assembled RAW functions: contains entries for each function for each year. Not every function has been measured every year, therefore many entries are missing (these are the unmeasured year-function combinations, encoded with `NM`).

   Currently this dataset is: [27087 “Assembled ecosystem measures from grassland EPs (2008-2018) for multifunctionality synthesis - June 2020”](https://www.bexis.uni-jena.de/ddm/data/Showdata/27087) 

2. A script `bexis_to_wide_format.R`: this is a script to reformat the data file from its long format to a wide format (plots x functions). Wide format is more comfortable to handle for analysis. The script removes the unmeasured year-function combinations. 

   Currently this script is attached to dataset 27087.

3. A table with detailed metadata for all functions `synthesis_grassland_function_metadata_ID27087`. 

   *Note : as indicated in 27087's metadata, please do not rely on the bexis metadata which is displayed when browsing the dataset, but the table described here, which is attached to the dataset.*
   
   Currently this file attached to dataset 27087, available as .xlsx and .csv

Note that the forest functions dataset has not been updated to this format yet and is stored in a Plot X Functions format. Currently this dataset is: [24367 “Raw data of forest attributes of forest EPs of the Exploratories project used in "Multiple forest attributes underpin the supply of multiple ecosystem services"](https://www.bexis.uni-jena.de/ddm/data/Showdata/24367).


### Interactions dataset
This dataset includes a list of all observed biotic interactions between pairs of species in the Biodiversity Exploratories (forests and grasslands).
You will find the dataset here: [List of biotic interactions between pairs of species in Biodiversity Exploratories](https://www.bexis.uni-jena.de/ddm/data/Showdata/31209)
*NOte: the dataset will be uploaded soon, we are currently making all the final checks*

## Datasets size and memory issues

The diversity and function synthesis datasets can be very large and this might create memory problems. We recommend the use of the package [`data.table`](https://cran.r-project.org/web/packages/data.table/index.html) in `R` to handle the dataset.

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
#Example code to reshape to the wide format and fill with zeros, useful e.g. to add back "missing zeros" (see section below):

library(data.table)

dat.wide<-dcast.data.table(dat,Species~Plot,value.var="value",fill=0)
```

![img](https://github.com/biodiversity-exploratories-synthesis/Synthesis-dataset-manual/blob/main/images/Synthesis%20datasets%20%20How%20to%20use-Fig1?raw=true)

**Fig1**: *illustration of the long format (mostly used to store and assemble data from different sources) and wide format (mostly used to type, visualise data or for data analysis)*



## Grouping (e.g. into trophic level)

### Diversity datasets

Not all groups were measured in all plots. When grouping several taxonomic groups into a given trophic level (or any other grouping, e.g. grouping multiple years), we recommend to always check if all groups/ species were measured in the same plots and only use the plots where all groups/ species were measured. Otherwise the calculated richness or other diversity estimates per group will be biased. We also recommend checking again the number of available plots before analysing data. The number of available plots can be found in the metadata of each dataset or can be retrieved from the synthesis dataset using the DataID column.

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

Not all groups were measured with the same methods. When grouping taxa that have several hundreds of species (e.g. Operational Taxonomic Unit (OTU) data) with taxa with very few species, the information about taxa with lower richness will likely be “hidden” by the other one. When species numbers in different groups are of similar range, grouping to calculate richness is mostly fine. Grouping to calculate abundances (e.g. total number of individuals in a given plot) can be more sensitive to the methods of collection and should be done very carefully.



### Functions datasets

The grouping information described above also applies to the functions datasets, when combining multiple functions. Additionally, it is worth mentioning that besides raw data, some functions are already grouped from several years, and some functions are combined. Such cases are indicated as “assembled data” in the column “DataID” of the metadata table (see 3. of the [Organisation](#Organisation) paragraph). The method of assembling (e.g. mean over years) is described in the “calculation” column.



## "Missing" zeros

To avoid increasing the size of the synthesis dataset, the two largest datasets (bacteria and soil fungi) do not include all combinations of plots x species when a given species was not found in a given plot i.e. they do not contain zeros (see Fig.2). These “missing” combinations are true zeros and should not be confused with NAs. The information in the soil fungi datasets is complete (i.e. all combinations of plots x species were measured, so if the value is not in the dataset it should be replaced by a zero). 

In grasslands, the bacteria dataset is not complete, two plots miss information for all species (AEG33 and AEG34). For these two plots, the values for all species should be replaced by NAs and any information missing in the rest of the plots are true zeros (and should be replaced by zeros).

In forests, the bacteria dataset is not complete, one plot misses information for all species (HEW04, see https://www.bexis.uni-jena.de/ddm/data/Showdata/24868). For this plot, the values for all species should be replaced by NAs and any information missing in the rest of the plots are true zeros (and should be replaced by zeros).

*TODO : We will provide information on missing plots for forest after the release of the updated forest dataset.*



![img](https://github.com/biodiversity-exploratories-synthesis/Synthesis-dataset-manual/blob/main/images/Synthesis%20datasets%20%20How%20to%20use-Fig2?raw=true)

**Fig2**: *example illustration of the values that are not present in the bacteria and soil fungi datasets due to dataset size limitations. The red values are the ones that are not reported in the synthesis dataset (shown on the right). Note that the soil fungi dataset does not have NAs, the bacteria dataset has NAs for all species for plots AEG33 and AEG34.*

Real NAs can happen when a full plot is missing from the dataset (e.g. AEG33 in Fig2) or when a combination of plot X species is not available. This second case only happens with arthropod datasets (e.g. pollinators in grasslands) and we recommend to only use plots with information on all species (i.e. without NAs, using e.g. the command na.omit(dat)). We highlight this topic because it might cause issues when using the data to e.g. calculate richness of a given group. We recommend to always check how many plots were measured per DataID and per year.

```R
#Example code to add zeros back for the bacteria and fungi datasets. Also adds NAs for the plots with missing data. Note: this will heavily  increase the datasets size and might cause memory issues!:
#create all combinations of Plot x Species (value will be set to NA by default)
bac_dat<-setDT(bac_dat)[CJ(Species=Species,Plot=Plot,unique=T), on=.(Species, Plot)]
#replace NAs by zeros
bac_dat[is.na(value), value := 0 ]
#add back NAs for non measured plots (this can be done in the first line of code but is showed separately for clarity)
bac_dat[Plot %in% c(“AEG33”,”AEG34”), value:=NA]
```

**Special case: plots do not contain any species** : The code above adds all missing plots x species combinations. However in some cases a plot does not contain any species, i.e. it was sampled but no species were found, so it has zeros for all species (e.g. HEG07 symbiont.soilfungi in the year 2011). In this case, the plot is missing completely from the dataset (or from the subset of the dataset). The complete absence of species from a plot is, however, an important one and we recommend not to exclude these plots. To identify such cases, please check if the number of plots before adding the missing plots x species combinations is the expected one (148 plots for bacteria, 150 plots for soilfungi). In order to add back plots that do not contain species, we recommend to use the function `add_back_missing_plots_species_combinations` from our github folder [useful_functions](https://github.com/biodiversity-exploratories-synthesis/useful_functions). 



## Temporal information

Several taxa have information for more than one year. The use of a single year or multiple years highly depends on the research question. When using several years to calculate richness, the values could be either averaged or summed (if methods are compatible, see “grouping” paragraph). For abundance it is less straightforward: summing the number of individuals might be meaningful for short lived species (e.g. some arthropods) but not long-lived organisms (e.g. perennial plants, for which information is given on cover).

Note also that some plots might not have been surveyed in a given year but were surveyed other years (e.g. SEG33 in 2013 in the plant dataset ID27386). Keeping or removing those plots from the dataset will depend on the research question and analyses.


## Specific notes for the diversity datasets

**Richness and abundance.** The column “type” provides information about the type of data: abundance, presence absence, cover, number of OTUs (operational taxonomic units), number of ASVs (exact sequence variants).

1. Do not use the number of OTUs (or ASVs) as a proxy for abundance! PFLA measures are a more appropriate way to describe the abundance of microorganisms (e.g. datasets 20250 for grasslands and 20926 for forests, please check for any updates on these dataset IDs). Number of OTUs/ASVs is highly dependent on methods and is not biologically meaningful. However, the number of OTUs can be used to calculate relative abundances (see original datasets metadata).



2. Rarefaction is needed to calculate the richness of bacteria and soil fungi (see above). We recommend contacting the data owners of these datasets to ask for the most up-to-date techniques of rarefaction for this type of data. The `R` package `phyloseq`, functions `phyloseq` and `rarefy_even_depth` are currently (November 2020) a good method to rarefy these datasets.

```R
# Example code to rarefy a microbial dataset (thanks to Kezia Goldmann and Johannes Sikorski for guidance on this topic)
dat <- otu_table(fungi.dat, taxa_are_rows = TRUE) # get a phyloseq object step1
pylodat <- phyloseq(dat) # get a phyloseq object step2

set.seed(1) #set a seed to get reproducible results
rarefied.dat <- rarefy_even_depth(phylodat) #sample size should be the smallest number of sequences per sample, alternatively, you can use (0.9*smallest number) to also shuffle the sample.
```

**Duplicates**. In very few cases, the same species were identified by two different datasets (e.g. some species were common in the ants and in the pollinator datasets). We removed all duplicate species occurrences and kept only the information from the datasets having more plots. If the number of sampled plots in the two datasets was similar, we prioritised the most extensive sampling for these species (e.g. some myriapods were collected together with other arthropods but we prioritised the information coming from the myriapod-specific sampling).

**Original datasets**. Original datasets might contain more information. For instance:
- birds 2018 have information on birds detected at different scales
- soil fungi have probabilities of species assignments ("unassigned", "Probable", "Highly Probable", "Possible") and could be filtered accordingly. In the datasets, we added all species.
- ants have abundance information
- earthworms have biomass information (Dataset 21686)
Please always check the metadata of the original datasets. Some of this information can be useful for some research questions.

**Taxonomic information** Taxonomic information is contained in the columns “Group_broad” and “Group_fine”. The degree of resolution in the classification can vary between different groups and was chosen to optimise multidiversity analyses. Please be aware that different types of analyses might require other taxonomic/trophic classifications. Complete taxonomic classification can usually be obtained from the owners of the single datasets.

**Trophic and functional groups information**
The column `Fun_group_fine` is quite heterogeneous and should be used in combination with the column `Fun_group_broad`.



## Specific notes for the functions datasets

**Correlated functions.** Some functions are very similar because they describe the same process in a different way or because one is a part of the other. For instance “NRI” and “N_leaching_risk” describe the same process although measured with different methods, or “Biomass.2009” is part of the aggregated measure “Biomass” over all years. These should be carefully evaluated before using them together in an analysis.

**Pollinators**. See “Important notes on arthropods” above.

**Choice of functions.** The choice of functions to be included in an analysis and their categorisation depends on the research question. Previous synthesis analyses can provide some guidance as well as some categories (e.g. stock and fluxes) provided in the metadata file (see [Organisation](#Organisation) paragraph above).



## Specific notes for the forest datasets

### Forest diversity datset
The diversity data in the forest datasets is mostly about understorey groups.
Important notes:
- Be aware that **deadwood fungi come from two different datasets** (Group_broad: "fungi.deadw"), these datasets were collected with different methods on different years and on a different total nummber of plots. Dataset 17186 is from 2011, has abundance information but only data on 120 plots. Dataset 18547 is from 2010, has presence-absencce information and data on 150 plots. We included both datasets to allow analyses in different years and because the number of species in the two datasets only partially overlap. However we recommend to 1) choose only one dataset for analyses, 2) not compare years and 3) if both years are used, subset the dataset to the common 120 plots.

```R
# Example code to check for this potential issue
forest_dataset[Group_broad == "fungi.deadw"]
unique(frs2[Group_broad == "fungi.deadw"]$DataID)
length(unique(frs2[DataID==17186]$Plot)) #how many plots in dataset 17186?
```
- In the 2021/2022 update we removed all soil fungi datasets and replaced them by the datasets created with the latest sequencing techniques. However, datasets specifically collected on fine roots (and not on soils) could be of interest for specific projects:
  - 22968: Fungal communities from fine roots collected from 150 forest plots in 2014 by Illumina sequencing  - normalized to 8400 reads per plot
  - 30973: Abundant of root-associated fungi from the organic layer and mineral soil across 150 forest experimental plots (soil sampling campaign, 2017) - OTU taxonomic look-up table
  - 30974: Abundant of root-associated fungi from organic layer and mineral soil across 150 forest experimental plots (soil sampling campaign, 2017)

- The soil fungi datasets included in the synthesis dataset contain general abundant soil fungi (also including AMF). However, for specific analyses and more precidse data on AMF, these datasets should also be considered:
    - 27688: Arbuscular mycorrhizal fungi on all 150 forest EPs (from Soil Sampling Campain 2011; Illumina MiSeq) - ASV abundances
  - 27690: Arbuscular mycorrhizal fungi on all 150 forest EPs (from Soil Sampling Campain 2014; Illumina MiSeq) - ASV abundances
  - 27692: Arbuscular mycorrhizal fungi on all 150 forest EPs (from Soil Sampling Campain 2017; Illumina MiSeq) - ASV abundances
  - 27686: Arbuscular mycorrhizal fungi on all 300 EPs (from Soil Sampling Campains 2011, 2014 and 2017; Illumina MiSeq) - ASV taxonomic look-up table

- Protists information. Cercozoa are mainly bacterivores (omnivores and eukarivores could be removed from analyses). Oomicota are mainly plant parasites, hemibiotrophs (alternate parasites) and saprotrophs (less specialised): this informain is in the Fun_group_fine column.


### Clear-cuts in Hainich

#### HEW51 and HEW02
In 2016, the forest plot HEW02 was heavily damaged by a storm and had to be harvested (clear-cut) between the field seasons of 2016 and 2017. The plot was transformed to a grassland, therefore a new plot, **HEW51** was established in 2017. 

In the forest datasets, both plots are included, but HEW02 only contains data for the years until 2016 (including 2016), and NAs from 2017 on (including 2017). Accordingly, the earliest data for HEW51 is in 2017, all years before 2017 (2016 and earlier) should be set to NA.

If using data from only either before and including 2016 or after 2016, 50 HEW plots can be used. Otherwise, if data from before AND after 2016 are used together in an analysis, we suggest removing both HEW02 and HEW51.


#### Forest damage due to drought in Hainich - The Spruce Case

Spruce in the Hainich forest are suffering from beetle infestations favoured by the drought. The timeline and most drastic cases of damage are summarised below.

There was a drought in 2018 and 2019. As a result, in **2019**, the Hainich forest was in severe trouble. The foresters were harvesting as many trees as never before, cutting bark-beetle-infected spruce trees as well as beech trees suffering from drought damages.
- **HEW03** suffered from such an extensive infestation with bark beetles that it had to be deforested at the end of the year 2019.
- **HEW51** was also heavily damaged by bark beetles, but was not deforested.

- Spruce in **HEW13** had to be deforested in February 2020. However, some of the trees have been left on the plot, as they are intact deciduous trees. 

- **HEW01** : Like all the other cases before, HEW01 was affected by drought and subsequent beetle infestation. Unlike the other plots, the dead spruces on the plot itself were left, but everything around was cut, in order to establish an experiment with standing deadwood.


TODO [Check back if we officially recommend to remove both plots in case of dataset spanning over 2017. Check what we recommend for the dry years and the deforested plots.]

## Important notes on arthropods

1. **Temporal data**. The synthesis dataset does not contain the temporal arthropods dataset. This is because more groups were sampled in the 2008 arthropod dataset, therefore we focussed on the most complete year of sampling. However, for questions related to temporal dynamics or for more extensive surveys of some arthropod groups (Coleoptera, Aranaea, Hemiptera and Orthoptera), we strongly recommend to use the temporal datasets from the Arthropod Core Team (ID21969 ["Sweep net samples from grasslands since 2008: Araneae, Coleoptera, Hemiptera, Orthoptera"](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=21969) and 26008 [“List of plots without complete sampling for sweepnetting of arthropods on grassland EPs 2008 to 2017”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=26008))

2. **Larger taxonomic coverage**. Some insect groups were sampled in more detail with focussed methods. They cannot be easily merged to the synthesis dataset because of different sampling methods. However we strongly recommend to consider them.

Grasslands:
   - 21207: [Dungwebs Species List 2014 & 2015 (Invertebrates, Scarabaeoidea, Dung Beetles)](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=21207)
   - 19826: [Orthoptera Density 2014 - all Grassland EPs - using biocenometer sampling](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=19826)
   - 20526: [Auchenorrhyncha Density 2015 - all Grassland EPs - using biocenometer sampling](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=20526)
   - 26026: [Moth abundance from light trapping on all grassland and forest plots 2018](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=26026) This dataset is however included in the forest dataset
   
Forests:   
   - 20034: [Bark Beetle Antagonists sampled with PheromoneTraps in Forest EPs in 2010](https://www.bexis.uni-jena.de/ddm/data/Showdata/20034)
   - 20031: [Bark Beetles sampled with Pheromone Traps in Forest EPs in 2010](https://www.bexis.uni-jena.de/ddm/data/Showdata/20031) This dataset is included in the forests dataset
   - 20035: [Bark Beetles pest control based on samples with Pheromone Traps in Forest EPs in 2010](https://www.bexis.uni-jena.de/ddm/data/Showdata/20035)
   - 24106: [Ambrosia beetles and antagonists sampled by Pheromone traps on all EPs in 2010 and on a subset in 2011](https://www.bexis.uni-jena.de/ddm/data/Showdata/24106)
   - 22027: [Window traps in tree crowns on forest VIPs and EPs, 2008, Coleoptera, Hemiptera, Orthoptera](https://www.bexis.uni-jena.de/ddm/data/Showdata/22027)
   - 22066: [The soil macrofauna orderlevel from all forest EPs sampled in spring 2011](https://www.bexis.uni-jena.de/ddm/data/Showdata/22066)
   - 21906: [Pitfall traps on forest EPs in 2008 subset Formicidae Species Abundances](https://www.bexis.uni-jena.de/ddm/data/Showdata/21906) For this dataset, some issues in the data could not be solved, see details in [this script](https://github.com/biodiversity-exploratories-synthesis/Synthesis_dataset_diversity_forest/blob/main/210217_ForestDivDataUPDATE.R)
   
   
Please note that this list might not be exhaustive.

3. **Pollinators**. In the grassland diversity dataset, the trophic level “pollinators” comes from different datasets but the grouping can be used because it was checked and homogenised by specialists from the Arthropod Core Team. Not all taxonomic groups within pollinators were measured in the same plots, so we recommend to use only the plots where all groups were measured.

   The pollinators in the synthesis diversity dataset are derived from the same dataset included in the synthesis functions dataset. Therefore high correlations between these two are artificial and analyses using both measures would be circular (i.e. done with the same data).


## Varia

1. Please always use the scripts provided in Bexis with caution (e.g ID22046 [“R scripts usable for dataset: Assembled RAW diversity from grassland EPs (2010-2016) for multidiversity synthesis”](https://www.bexis.uni-jena.de/ddm/data/Showdata/22046) is mostly given as an example) and please report back any spotted errors or issues via email and/or via [Github](https://github.com/biodiversity-exploratories-synthesis).

