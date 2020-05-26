---
title: "Unemployment Claims"
date: 2020-05-26
tags: [blogposts, R, Data vis]
header: 
  image: "/images/placeholder.jpg"
excerpt: "Placeholder"
---



```R



rm(list=ls())

library(data.table)
library(usmap)
library(ggplot2)
library(rvest)
library(pdftools)
library(tidyverse)
library(plotly)

unemp_data <- fread("/Users/charlieahlstrom/Documents/GitHub/New-Ideas/covid/data/unemp_claim_may24.csv")

sub <- unemp_data[, c("Reflecting Week Ended", "State",
                      "Initial Claims", "Continued Claims","Covered Employment", "Insured Unemployment Rate")]

names(sub) <- c("week_end","state","claims_initial","claims_cont","covered_emp","unemp_rate")
sub[, ':=' (week_end = as.Date(week_end, "%m/%d/%y"),
  claims_initial=as.numeric(gsub(",","", claims_initial)),
            claims_cont=as.numeric(gsub(",","", claims_cont)),
            covered_emp=as.numeric(gsub(",","", covered_emp)))]


il <- sub[state=="Illinois"]
ca <- sub[state=="California"]
fl <- sub[state=="Florida"]

#plot(x=ca$week_end, y=ca$claims_initial, type="l")
#plot(x=ca$week_end, y=ca$claims_initial_chg, type="l")
#plot(x=ca$week_end, y=ca$claims_cont, type="l")
#plot(x=ca$week_end, y=ca$claims_cont_chg, type="l")


#plot(x=fl$week_end, y=fl$claims_initial, type="l")
#plot(x=fl$week_end, y=fl$claims_initial_chg, type="l")
#plot(x=fl$week_end, y=fl$claims_cont, type="l")
#plot(x=fl$week_end, y=fl$claims_cont_chg, type="l")

sub[, claims_initial_chg := claims_initial - shift(claims_initial,n=1, type= "lag", fill=NA), by = .(state)]
sub[, claims_cont_chg := claims_cont - shift(claims_cont,n=1, type= "lag", fill=NA), by = .(state)]

may <- sub[week_end=="2020-05-02"]


#plot_usmap(data = may, values = "claims_cont", color = "red") +
#  scale_fill_continuous(
#    low = "white", high = "red", name = "Intial Claims", label = scales::comma
#  ) + theme(legend.position = "right")




#plot_usmap(data = may, values = "claims_initial_chg", color = "red") +
#  scale_fill_continuous(
#    low = "white", high = "red", name = "Intial Claims", label = scales::comma
#  ) + theme(legend.position = "right")



#plot_usmap(data = may, values = "claims_cont_chg", color = "red") +
#  scale_fill_continuous(
#    low = "white", high = "red", name = "Intial Claims", label = scales::comma
#  ) + theme(legend.position = "right")




pdf <- "/Users/charlieahlstrom/Documents/GitHub/New-Ideas/covid/data/data.pdf"
webpage <- pdf_text(pdf)  %>% strsplit(split = "\n")




all_text <- pdf_text(pdf)  %>%
  read_lines

state_comments_raw <- all_text[420:481] %>%
                        str_squish() %>%
                        str_replace_all(",", "") %>%
                        strsplit(split = " ")


cur_comment=state_comments_raw[[4]]


state_link <- data.table(abbre = state.abb, state=state.name)

clean_comments <- function(cur_comment){
  # keep only rows where there is a state comment/no comment
  if(cur_comment[1] %in% state.abb){

    cur_state_abbre <- cur_comment[1]
    comment <- paste(cur_comment[-c(1,2)], collapse=" ")

    cur_state <- state_link[abbre==cur_state_abbre]$state
    ret_table <- data.table(state=cur_state, abbre=cur_state_abbre, comment)
    return(ret_table)
  }
}

state_comments_clean <- lapply(FUN=clean_comments, state_comments_raw)
state_comments_clean <- do.call(rbind,state_comments_clean)


# keep states with a comment
#state_comments <- state_comments_clean[comment != "No comment."]
tt=merge(state_comments_clean, may, by = c("state"), all.y=TRUE)




#may[, comments := "No comment."]
tt <- tt[state != "District of Columbia"]
tt <- tt[state != "Puerto Rico"]
tt <- tt[state != "Virgin Islands"]
tt[, code := abbre]

tt$hover <- with(tt, paste(state, '<br>', tt$comment))


# give state boundaries a white border
l <- list(color = toRGB("white"), width = 2)
# specify some map projection/options
g <- list(
  scope = 'usa',
  projection = list(type = 'albers usa'),
  showlakes = TRUE,
  lakecolor = toRGB('white')
)

fig <- plot_geo(tt, locationmode = 'USA-states')
fig <- fig %>% add_trace(
    z = ~claims_initial, text = ~hover, locations = ~code,
    color = ~claims_initial, colors = 'Reds'
  )
fig <- fig %>% colorbar(title = "Legend units")
fig <- fig %>% layout(
    title = 'Figure title',
    geo = g
  )

fig




```


<iframe width='100%' height='300' src='https://rdrr.io/snippets/embed/' frameborder='0'>
	fig
</iframe>

