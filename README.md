---
title: 'Data Mining for Text: Topic Models & Semantic Networks'
subtitle: 'INFO 523 Final Presentation Examples'
author:
- name: Tara K. Prendergast
  affiliation: School of Sociology
output: html_document

---
-------------------

### Objectives and Attribution
This file includes code and outputs for the examples of text mining applications
included in my final presentation for 523. Both examples - of topic modeling (1) and
text networks (2) - come from Chris Bail. See links below.

1: https://cbail.github.io/SICSS_Topic_Modeling.html
2: https://cbail.github.io/textasdata/text-networks/rmarkdown/Text_Networks.html

--------------------

## Example 1: Topic Model

Install packages and load data

```{r}
# install.packages("topicmodels")
# install.packages("tidyverse")
# install.packages("tidytext")


data("AssociatedPress")

```

Apply LDA algorithm and set k 

```{r}

library(topicmodels)
AP_topic_model <- LDA(AssociatedPress, k=10, control = list(seed = 321))

```

Produce bar graphs showing top 10 terms for each topic (10 topics b/c k=10)

```{r}

library(dplyr)
library(ggplot2)
library(tidytext)

AP_topics <- tidy(AP_topic_model, matrix = "beta")

ap_top_terms <- 
  AP_topics %>%
  group_by(topic) %>%
  top_n(10, beta) %>%
  ungroup() %>%
  arrange(topic, -beta)


ap_top_terms %>%
  mutate(term = reorder(term, beta)) %>%
  ggplot(aes(term, beta, fill = factor(topic))) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free") +
  coord_flip()

```
Rerun but set k=5, show top 10 words for 5 topics

```{r}

library(topicmodels)
AP_topic_model <- LDA(AssociatedPress, k=5, control = list(seed = 321))

library(dplyr)
library(ggplot2)
library(tidytext)
AP_topics <- tidy(AP_topic_model, matrix = "beta")

ap_top_terms <- 
  AP_topics %>%
  group_by(topic) %>%
  top_n(10, beta) %>%
  ungroup() %>%
  arrange(topic, -beta)


(ap_top_terms %>%
  mutate(term = reorder(term, beta)) %>%
  ggplot(aes(term, beta, fill = factor(topic))) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free") +
  coord_flip())

```


## Example 2: Text Network

Install and load Textnets package

```{r}
Sys.unsetenv("GITHUB_PAT") #included because getting error message when not included

# install.packages("devtools")
library(devtools)
# install.packages("remotes")
remotes::install_github("cbail/textnets")
library(textnets)
```

Load and prepare text data using part-of-speech tagging - select first State of Union for each president, nodes as groups of presidents, edges as overlap of words used in speeches. Remove stop words and return noun compounds.

```{r}
library(textnets)
data("sotu")

sotu_first_speeches <- sotu %>% group_by(president) %>% slice(1L)

prepped_sotu <- PrepText(sotu_first_speeches, groupvar = "president", textvar = "sotu_text", node_type = "groups", tokenizer = "words", pos = "nouns", remove_stop_words = TRUE, compound_nouns = TRUE)

```


Create text networks using CreateTextnet function

```{r}

sotu_text_network <- CreateTextnet(prepped_sotu)

```

Visualize text networks using VisualizeText function, where nodes are colored by cluster

```{r}

(VisTextNet(sotu_text_network, label_degree_cut = 0))

```

Visualize text networks using VisTextNetD3 function to create interactive output. Hover over nodes to see labels. Nodes are colored by cluster.

```{r}

(VisTextNetD3(sotu_text_network))

```
