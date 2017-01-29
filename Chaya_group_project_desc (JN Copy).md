---
title: "Lending Club Loans"
output: html_document
author: "Chaya Maheshwari, Pedro Henriques, Jada Neumann, Stanislaw Ostoja-Starzewski"
---



##Problem Statement:

The project aims to segment Lending Club's customers base so that P2P investors are better able to understand their expected returns given their lenders characetrisitics. For that purpose we will use two variables: PD (Probability of Default) and LGD (Loss Given Default). We will then have a model that allows us to estimate expected returns for each investment.


##Process:

### 1) Define Business Problem
Lending Club allows people with weak financial knowledge to invest in what can be highly risky assets. Our goal is to provide potential investors with inteligence that allows them to make better investment decisions. For that purpose we analize 500k entries of past data past investment data to build a predictive model. Our key risk parameter will be the Probability of Default (PD), i.e. the probability of a lender not servicing his debt on time

### 2) Collect and Clean Up Data

Before beginning the analysis of the data, ensure that the raw data is complete and organized in a way that is conducive to the analysis.  The actions to be taken are as follows:

- Download data as a .csv file from lendingclub.com or kaggle.com
- Load the data and make a working copy (so that none of the raw data was lost in case we wanted to recover it later)



```r
# Please ENTER the name of the file with the data used. The file should be a
# .csv with one row per observation (e.g. person) and one column per
# attribute. Do not add .csv at the end, make sure the data are numeric.
datafile_name = "../bdagroupproject/Data/loancopy.csv"

# Please enter the minimum number below which you would like not to print -
# this makes the readability of the tables easier. Default values are either
# 10e6 (to print everything) or 0.5. Try both to see the difference.
MIN_VALUE = 0.5

# Please enter the maximum number of observations to show in the report and
# slides.  DEFAULT is 10. If the number is large the report may be slow.
max_data_report = 10
```


```r
ProjectData <- read.csv(datafile_name)
ProjectData <- data.matrix(ProjectData) 
ProjectData_INITIAL <- ProjectData
```

- Test unique identifier for each entry/loan by seeing if there are any double entries under the <U+201C>id<U+201D> and <U+201C>member_id<U+201D> columns
- Eliminate variables that are out of scope or are too lengthy to parse/process for the benefit of analysis:
    + Remove active loans (i.e. loans that haven<U+2019>t had the opportunity to default or not because they are still ongoing) and loans with a blank status
    + Remove columns deemed unnecessary to analyze as they wouldn<U+2019>t provide useful information (e.g. <U+201C>url<U+201D>: URL for the Lending Club page with listing data)
    +	Remove columns that are too difficult to standardize (e.g. <U+201C>desc<U+201D>: loan description provided by the borrower; or <U+201C>emp_title<U+201D>: employee title)
    +	Remove columns that represent similar information to other columns (e.g. <U+201C>desc<U+201D> is largely covered by the more standardized <U+201C>purpose<U+201D> field)
    +	Remove columns containing information that would only be obtained AFTER somebody became a client (i.e. couldn<U+2019>t be used to make the initial lending decision) (e.g. <U+201C>tot_coll_amt<U+201D>: total collection amounts ever owed; or <U+201C>last_pymnt_d<U+201D>: last month payment was received)
- Exclude entries with missing information
    + Remove columns where there is mostly missing information, even if that column would have otherwise been informative
    + Remove rows where there is any missing information
- Combine non-numeric descriptions when appropriate, for example:
    + The Charge-Off and Default classifications could be combined into one Default category under the <U+201C>loan_status<U+201D> field because default occurs before charge-off (after 121 days vs. 150 days)
    + The <U+201C>purpose<U+201D> column had a number of non-numeric values such as <U+201C>debt consolidation<U+201D> or <U+201C>home improvement<U+201D>; however, over 80% of entries were debt-related, so it seemed reasonable to split the data into just two categories: debt-related and other
- Correct errors
    + The <U+201C>issue_d<U+201D> column had the dates formatted backwards, so, for example, January 2014 was showing as January 14, 2017; reformat to make the correction (note that due to lack of better information, assumptions can be made such as all issue dates occur on the first of the month)


### 3) Ensure Data Is Metric

In order to begin analysing the data (generating descriptive statistics, etc.), data must be metric (i.e. numbers, and specifically numbers that have meaningful hierarchical values).

- Remove text from otherwise numeric fields.  For example:
    + The <U+201C>term<U+201D> column had values of <U+201C>36 months<U+201D> or <U+201C>60 months<U+201D>; change to simply <U+201C>36<U+201D> or <U+201C>60<U+201D>, respectively
    + The <U+201C>emp_length<U+201D> column had values of <U+201C>[x] years<U+201D>; change to simply <U+201C>[x]<U+201D>
    + The <U+201C>emp_length<U+201D> column also had values of <U+201C><1<U+201D>, <U+201C>n/a<U+201D> and <U+201C>10+<U+201D>; change to <U+201C>0<U+201D>, <U+201C>0<U+201D> and <U+201C>10<U+201D>, respectively
- Create dummy variables for non-numeric values.  For example:
    + Add a separate column <U+201C>emp_length_known<U+201D> to separate which customers have <U+201C>n/a<U+201D> values for <U+201C>emp_length<U+201D> (indicated by a 0 here)
    + The <U+201C>home_ownership<U+201D> column had values of <U+201C>Own<U+201D>, <U+201C>Mortgage<U+201D> or <U+201C>Rent<U+201D>; separate into three columns: <U+201C>home_renter<U+201D>, <U+201C>home_mortgager<U+201D> and <U+201C>home_owner<U+201D>; a <U+201C>1<U+201D> in these columns indicates membership in that category
    + The <U+201C>loan_status<U+201D> column had values of <U+201C>Fully Paid<U+201D> or <U+201C>Default<U+201D>; change to <U+201C>1<U+201D> to indicate fully paid and <U+201C>0<U+201D> to indicate default
- Convert non-numeric but hierarchical data into numbers, for example:
    + The <U+201C>grade<U+201D> column had ratings of A to G; change to ratings of 1 to 7
    + The <U+201C>sub_grade<U+201D> column had ratings of A1 <U+2013> G5; change to 1.0 to 7.8 (each increment adds 0.2, so that, for example, B2 becomes 2.2 or D4 becomes 4.6)
- Convert physical addresses into a format in which distances can be measured
    + Two columns, <U+201C>zip_code<U+201D> and <U+201C>addr_state<U+201D>, provided information about customer addresses; however, <U+201C>zip_code<U+201D> was in the form <U+201C>###xx<U+201D>, showing only the first three numbers of a zipcode; therefore, it was not metric and this field was excluded
    + In order to make <U+201C>addr_state<U+201D> metric, the mid-point latitude and longitude of each state was added as two new columns: <U+201C>addr_lat<U+201D> and <U+201C>addr_lon<U+201D>, respectively; the original "addr_state" field was then deleted 


### 4) Scale the Data

<>


### 5) Dimentionality Reduction

- Step 1: analysing correlations and identifying variables which are linear combinations of one another


```r
# Please ENTER then original raw attributes to use.  Please use numbers, not
# column names, e.g. c(1:5, 7, 8) uses columns 1,2,3,4,5,7,8
factor_attributes_used = c(3:23)

# Please ENTER the selection criterions for the factors to use.  Choices:
# 'eigenvalue', 'variance', 'manual'
factor_selectionciterion = "eigenvalue"

# Please ENTER the desired minumum variance explained (Only used in case
# 'variance' is the factor selection criterion used).
minimum_variance_explained = 65  # between 1 and 100

# Please ENTER the number of factors to use (Only used in case 'manual' is
# the factor selection criterion used).
manual_numb_factors_used = 15

# Please ENTER the rotation eventually used (e.g. 'none', 'varimax',
# 'quatimax', 'promax', 'oblimin', 'simplimax', and 'cluster' - see
# help(principal)). Default is 'varimax'
rotation_used = "varimax"
```




## Check Correlations

Analysing correlations and identifying variables which are linear combinations of one another. This is the correlation matrix of all the different attributes/variables for the unique customers we have. 



```r
thecor = round(cor(ProjectDataFactor),2)
iprint.df(round(thecor,2), scale=TRUE)
```

```
## Error in eval(expr, envir, enclos): could not find function "iprint.df"
```

## Choose number of factors

Clearly the different column variables have several correlations between them, so we may be able to actually "group" these variables into only a few "key factors". This not only will simplify the data, but will also greatly facilitate our understanding of the lenders club members.


```r
# Here is how the `principal` function is used 
UnRotated_Results<-principal(ProjectDataFactor, nfactors=ncol(ProjectDataFactor), rotate="none",score=TRUE)
```

```
## Error in eval(expr, envir, enclos): could not find function "principal"
```

```r
UnRotated_Factors<-round(UnRotated_Results$loadings,2)
```

```
## Error in eval(expr, envir, enclos): object 'UnRotated_Results' not found
```

```r
UnRotated_Factors<-as.data.frame(unclass(UnRotated_Factors))
```

```
## Error in as.data.frame(unclass(UnRotated_Factors)): object 'UnRotated_Factors' not found
```

```r
colnames(UnRotated_Factors)<-paste("Comp",1:ncol(UnRotated_Factors),sep="")
```

```
## Error in ncol(UnRotated_Factors): object 'UnRotated_Factors' not found
```


```r
# Here is how we use the `PCA` function 
Variance_Explained_Table_results<-PCA(ProjectDataFactor, graph=FALSE)
```

```
## Error in eval(expr, envir, enclos): could not find function "PCA"
```

```r
Variance_Explained_Table<-Variance_Explained_Table_results$eig
```

```
## Error in eval(expr, envir, enclos): object 'Variance_Explained_Table_results' not found
```

```r
Variance_Explained_Table_copy<-Variance_Explained_Table
```

```
## Error in eval(expr, envir, enclos): object 'Variance_Explained_Table' not found
```

```r
rownames(Variance_Explained_Table) <- paste("Component", 1:nrow(Variance_Explained_Table), sep=" ")
```

```
## Error in nrow(Variance_Explained_Table): object 'Variance_Explained_Table' not found
```

```r
colnames(Variance_Explained_Table) <- c("Eigenvalue", "Pct of explained variance", "Cumulative pct of explained variance")
```

```
## Error in colnames(Variance_Explained_Table) <- c("Eigenvalue", "Pct of explained variance", : object 'Variance_Explained_Table' not found
```

Let's look at the **variance explained** as well as the **eigenvalues**


```r
iprint.df(round(Variance_Explained_Table, 2))
```

```
## Error in eval(expr, envir, enclos): could not find function "iprint.df"
```


```r
eigenvalues  <- Variance_Explained_Table[, "Eigenvalue"]
```

```
## Error in eval(expr, envir, enclos): object 'Variance_Explained_Table' not found
```

```r
df           <- cbind(as.data.frame(eigenvalues), c(1:length(eigenvalues)), rep(1, length(eigenvalues)))
```

```
## Error in as.data.frame(eigenvalues): object 'eigenvalues' not found
```

```r
colnames(df) <- c("eigenvalues", "components", "abline")
```

```
## Error in `colnames<-`(`*tmp*`, value = c("eigenvalues", "components", : attempt to set 'colnames' on an object with less than two dimensions
```

```r
iplot.df(melt(df, id="components"))
```

```
## Error in eval(expr, envir, enclos): could not find function "iplot.df"
```

## Interpret the factors

This is how the "top factors" look like. 


```r
if (factor_selectionciterion == "eigenvalue")
  factors_selected = sum(Variance_Explained_Table_copy[,1] >= 1)
```

```
## Error in eval(expr, envir, enclos): object 'Variance_Explained_Table_copy' not found
```

```r
if (factor_selectionciterion == "variance")
  factors_selected = 1:head(which(Variance_Explained_Table_copy[,"cumulative percentage of variance"]>= minimum_variance_explained),1)
if (factor_selectionciterion == "manual")
  factors_selected = manual_numb_factors_used
```





