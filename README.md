# Scholarly Citations and Twitter: An Insight into the Case of Economists
Final data science project for Econ-5970 of the University of Oklahoma, Norman Campus. This project aims to scrape data from [RePec](https://ideas.repec.org/), which is a database containing information on registered economists, their publications, institution of affiliation, and contributions. I use R Studio Version 1.0.153 for all web scraping and coding practices. 

## Getting Started
In order to replicate the results, the following packages need to called and/or installed in R Studio:
```
library(rvest)
library(stringr)
library(moments)
library(stargazer)
library(ggplot2)
library(ggrepel)
```
All the information was extracted from [IDEAS](https://ideas.repec.org/), however, three main web pages within this website were used for this project: 1) [Top 25% Economists by Twitter Follwoers](https://ideas.repec.org/top/top.person.twitter.html), 2) [Top 5% Authors, Number of Citations, as of MArch 2018](https://ideas.repec.org/top/top.person.nbcites.html), and 3) [Top 5% Authors, h-index, as of March 2018](https://ideas.repec.org/top/top.person.hindex.html). We read each the html information of each of these links and denominate them website1, website2, and website3 respectively. 
```
website  <- read_html("https://ideas.repec.org/top/top.person.twitter.html")
website2 <- read_html("https://ideas.repec.org/top/top.person.nbcites.html")
website3 <- read_html("https://ideas.repec.org/top/top.person.hindex.html")
```

### Extracting data from website1
This data set will contain information on Twitter ranking, author name, the IDEAS id (which is the personal link to each author's profile within the IDEAS data base, link to the authors' Citec profile (which has more extensive information on on the academic activity of the author), number of twitter followers and twitter handles.

First, we will extract the rankings based on the number of twitter followers. However, there is a period at the end of each number, but the second line of code will help us clean it: 
```
rank.twitter <- website %>% html_nodes("td:nth-child(2)") %>% html_text(trim = TRUE)
rank.twitter <- str_sub(rank.twitter, start = 0, end = -2) # to trim last "period" character
```
We proceed to extract the authors/economists' names and last names. The data will include some empty cells, so we clean it again with the last line of code:
```
economist    <- website %>% html_nodes("#listing a")
author       <- economist %>% html_text(trim = TRUE)
author       <- author[author!=""] # to drop empty cells
```
To extract the IDEAS ID, we will run the following code and omit all NA values in order to clean our data: 
```
author.id    <- html_attr(economist, "name")
author.id    <- na.omit(author.id) # drop all NA values
```
And then we extract the number of Twitter followers, which fortunately does not need to be cleaned:
```
t.followers <- website %>% html_nodes("td~ td+ td") %>% html_text(trim = TRUE)
```
We finally bind all strings of information to build a data frame with the following code:
```
top25.twitter <- cbind(rank.twitter, author, author.id, ideas.link, t.followers)
top25.twitter <- as.data.frame(top25.twitter)
```
And we keep editing the format. The following line of code will re-order the 'Last Name, Name' format to the 'Name and Last Name' format, so it can be read more easily, but mainly to avoid futher troubles when merging the different data sets we are preparing. 
```
splits<- str_split_fixed(top25.twitter$author, ",", 2)
top25.twitter$author <- paste(splits[,2], splits[,1], sep = ' ')
```
Our column with information on the IDEAS' links to each authors' personal profiles is incomplete as well, so we paste the missing information to every observation in the column by running the following code:

```
top25.twitter$ideas.link<- paste("https://ideas.repec.org", top25.twitter$ideas.link, sep = "")
```
With this last piece of information, we will be able to extract the Twitter handles from every registered author with a Twitter account that appears in the list 'Top 25% Economists, by Twitter Followers.' We will run a loop for this purpose, so we need to create an empty vector to store the data our loop will gather: 
```
t.handle      <- 0 # create empty vector 
for (i in top25.twitter$ideas.link){
  t.handle[i] <- read_html(c(i)) %>% # run loop to go through each author profile in IDEAS
    html_node("tr:nth-child(10) td+ td a") %>%
    html_text(trim=TRUE)
}
```
We save it as a data frame and delete the first row with empty values: 

```
t.handle <- as.data.frame(t.handle)
t.handle <- t.handle[-1,] # drop first empty row 
```
We inorcporate the Twitter handle column into our data set: 
```
top25.twitter$t.handle<- t.handle
```
Again, with loops, we extract each authors' personal link in the Citec data base, drop the first empty row, and incoporate as a column to our first data set from website1. The citec.link variable will have a lot of missing values, but I haven't explored the reason behind this. The first data set will look like this:

```
citec.link<- 0
for (i in top25.twitter$ideas.link){
  citec.link[i]<- html_attr((read_html(c(i)) %>%
                               html_node("#cites a:nth-child(3)")), "href")
}
citec.link<- as.data.frame(citec.link)
citec.link<- citec.link[-1,]
# incorporate Citec column into dataset
top25.twitter$citec.link<- citec.link # though we only have 182 citec links, and we are missing 172
View(top25.twitter)

 rank.twitter                author     id                            ideas.link t.followers         t.handle citec.link
1            1       Paul R. Krugman  pkr10  https://ideas.repec.org/e/pkr10.html     4501778     @paulkrugman       <NA>
2            2  Xavier Sala-i-Martin psa510 https://ideas.repec.org/f/psa510.html      441591    @xsalaimartin       <NA>
3            3    Joseph E. Stiglitz  pst33  https://ideas.repec.org/e/pst33.html      219312 @JosephEStiglitz       <NA>
4            4     Alejandro Gaviria pga134 https://ideas.repec.org/e/pga134.html      190877       @agaviriau       <NA>
5            5     Erik Brynjolfsson  pbr87  https://ideas.repec.org/e/pbr87.html      165839        @erikbryn       <NA>
6            6        Justin Wolfers   pwo9   https://ideas.repec.org/e/pwo9.html      143943   @justinwolfers       <NA>

```


### Prerequisites

### Installing

## Running the tests

### Brek down into end to end tests

## and coding style tests

## Deployment 

## Built With

## Acknoledgements 

