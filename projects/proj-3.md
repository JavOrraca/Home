
**IBM Watson NLU: What is it? And in R?!?**
The following instructions provide a step-by-step guide for running IBM Watson's Natural Language Understanding (NLU) tool via R. Watson NLU is a powerful text analysis tool for extracting metadata from content such as concepts, entities, keywords, categories, sentiment, emotions, relations, and even semantic roles.

One of the projects that I'm working on with a team is to analyze consumer-created Facebook and Twitter content to infer brand perceptions on social media. In my case, I'm analyzing Southwest and jetBlue content on Facebook and Twitter. My data preparation consists of the following: social media post and metadata collection, cleaning / organizing / categorizing data, and structuring the data into a dataframe for R or Excel / CSV data set. For this test set, we pulled all the data manually (I realize there are ways to automate this but our data set was about 50 posts per airline and source).

The code utilizes R to call Watson's NLU API and analyze social media content for sentiment analysis and emotions. I did not perform a time-series based study, however, this is definitely something I'll consider in the future to see how consumer perceptions of brands change over time. 

**Prerequisites**
* [IBM Watson NLU Demo](https://natural-language-understanding-demo.ng.bluemix.net): If you've read this far and want to stop or don't feel you have the skills/tools to continue, this link lets you test drive Watson NLU via their demo, free for public use and no signup required
* [IBMid](https://myibm.ibm.com): If you don't have an IBMid, sign up for one here
* [IBM Cloud Lite Account](https://www.ibm.com/cloud/lite-account): You'll need an IBMid to sign up for a free Cloud Lite account, but this is a prerequisite for API access to Watson NLU
* R notebook: I typically use [RStudio (desktop)](https://www.rstudio.com/) if I'm solo or [Jupyter Labs (web-based)](https://blog.jupyter.org/jupyterlab-is-ready-for-users-5a6f039b8906) if I'm collaborating with others
* R packages: httr, jsonlite, xlsx (in my team's case, we collected our social data in Excel), reshape, dplyr, ggplot2

First, load the appropriate libraries, installing any of the packages you don't already have:


```R
library(httr)
library(jsonlite)
library(xlsx)
library(reshape)
library(dplyr)
library(ggplot2)
```

After loading the libraries, set your working directory followed by reading your Excel file, CSV file, or URL, and assign this to variable "df":


```R
setwd("{Enter your working directory here}")

## As I was processing an Excel file and not an URL, I relied on the xlsx package to read the first sheet of my Excel file, acknowledging that my top row has header names.
df <- read.xlsx("Sentiment Analysis SWA vs JB_Clean.xlsx", sheetIndex = 1, header = TRUE, colClasses = NA)
```

Next, assign your IBM Watson NLU API username and password details to variables in order to send data to/from the Watson NLU. Replace the username and password astericks with your API credentials. The function assignment that follows, watsonNLUtoDF, is the bread and butter of this R code.


```R
username <- "********-****-****-****-************"
password <- "************"

watsonNLUtoDF <- function(data, username, password, verbose = F, language = 'en') {
  
  ## Url for Watson NLU service on Bluemix
  base_url <- "https://gateway.watsonplatform.net/natural-language-understanding/api/v1/analyze?version=2018-03-16"
  
  ## Initialize Empty Dataframes
  conceptsDF <- data.frame()
  keywordsDF <- data.frame()
  emotionDF <- data.frame()
  sentimentDF <- data.frame()
  categoriesDF <- data.frame()
  analyzedTextDF <- data.frame()
  
  ## Loop over each id, identify the type and send the value to Watson
  for (i in 1:nrow(data)){
    try({
      
      ## In my particular data set, "Company_Source" is a concatenated field of Company and social media Source
      ## (e.g., "Southwest (Facebook)"), and "Content" represents the actual Facebook post or Tweet
      id <- data$Company_Source[i]
      value <- data$Content[i]
      
      ## Define the JSON payload for NLU
      body <- list(api_endpoint = value, 
                   features = list(
                     categories = {},
                     concepts = {},
                     keywords = {},
                     emotion = {},
                     sentiment = {}),
                   language = language,
                   return_analyzed_text = TRUE)
      
      ## Provide the correct type for each id
      names(body)[1] <- "text"
      
      if(verbose == T){
        print(paste("Sending", "text", "for", id, "to Watson NLU..."))
      }
      
      ## Hit the API and return JSON
      watsonResponse <- POST(base_url,
                             content_type_json(),
                             authenticate(username, password, type = "basic"),
                             body = toJSON(body, auto_unbox = T)) 
      
      ## Parse JSON into dataframes
      concepts <- data.frame(id = id, 
                             fromJSON(toJSON(content(watsonResponse), pretty = T), flatten = T)$concepts,
                             stringsAsFactors = F)
      
      keywords <- data.frame(id = id, 
                             fromJSON(toJSON(content(watsonResponse), pretty = T), flatten = T)$keywords,
                             stringsAsFactors = F)
      
      emotion <- data.frame(id = id, 
                             fromJSON(toJSON(content(watsonResponse), pretty = T), flatten = T)$emotion,
                             stringsAsFactors = F)
      
      sentiment <- data.frame(id = id, 
                              fromJSON(toJSON(content(watsonResponse), pretty = T), flatten = T)$sentiment,
                              stringsAsFactors = F)
      
      categories <- data.frame(id = id,
                               fromJSON(toJSON(content(watsonResponse), pretty = T), flatten = T)$categories,
                               stringsAsFactors = F)
      
      analyzedText <- data.frame(id = id,
                                 fromJSON(toJSON(content(watsonResponse), pretty = T), flatten = T)$analyzed_text,
                                 stringsAsFactors = F)
      
      
      ## Append results to output dataframes
      conceptsDF <- rbind(conceptsDF, concepts)
      keywordsDF <- rbind(keywordsDF, keywords)
      emotionDF <- rbind(emotionDF, emotion)
      sentimentDF <- rbind(sentimentDF, sentiment)
      categoriesDF <- rbind(categoriesDF, categories)
      analyzedTextDF <- rbind(analyzedTextDF, analyzedText)
      
      if(verbose == T) {
        print(paste("Iteration", i, "of", nrow(data), "complete."))
      }
    })
  }
  resultsList <- list(conceptsDF, keywordsDF, emotionDF, sentimentDF, categoriesDF, analyzedTextDF, watsonResponse)
  names(resultsList) <- c("conceptsDF", "keywordsDF", "emotionDF", "sentimentDF", "categoriesDF", "analyzedTextDF", "response")
  return(resultsList)
}
```

The last part of this code walks you through next steps of sending and collecting data to/from Watson NLU.


```R
## Now, send/receive data from via Watson's NLU API! Note: This process is SLOW... Large data sets may take hours to process.
responseList <- watsonNLUtoDF(df, username, password, verbose = T)

## Test that results were returned by viewing the Watson NLU responses you need. In my case, I mainly care about sentiment and emotions.
sentiment_table <- responseList$sentimentDF

emotion_table <- responseList$emotionDF

## Now we can calculate the mean Sentiment by Airline/Source, rename the columns, and plot using the ggplot2 package
mean_sentiment <- aggregate(sentiment_table[, 2], list(sentiment_table$id), mean)

colnames(mean_sentiment) <- c("Airline/Source", "Mean")

## Similarly to the above, calculate the mean of each Emotion by Airline/Source, and rename the columns
mean_emotion <- aggregate(emotion_table[, 2:6], list(emotion_table$id), mean)

colnames(mean_emotion) <- c("Airline/Source", "Sadness", "Joy", "Fear", "Disgust", "Anger")

## Given that the mean_emotion table ends up with side-by-side variables, we'll use the Reshape package
## to reformat the table in long format, as this will be easier for plotting with ggplot2
mean_emotion_long <- melt(mean_emotion, id.vars = "Airline/Source")

colnames(mean_emotion_long) <- c("Airline/Source", "Emotion", "Mean")
```

Lastly, below we'll visualize our results as bar charts with ggplot2: 


```R
## Create a bar chart for mean sentiment score by Airline and source
ggplot(mean_sentiment) + geom_col(aes(x=`Airline/Source`, y=Mean)) +
  geom_text(aes(x=`Airline/Source`, y=Mean + 0.02, label=round(Mean, 2))) +
  labs(title='Sentiment Score by Airline (Source)', x="Airline (Source)", y="Mean")

## Create a side-by-side bar chart for emotion score (emotions on x-axis)
ggplot(mean_emotion_long,aes(x=Emotion,y=Mean,fill=factor(`Airline/Source`))) +
  geom_bar(stat="identity",position="dodge") +  scale_fill_brewer(name="Airline (Source)") +
  labs(title='Emotion Scores by Airline (Source)', x="Emotion", y="Mean")

## Create a side-by-side bar chart for emotion score (airline/source on x-axis)
ggplot(mean_emotion_long,aes(x=`Airline/Source`,y=Mean,fill=factor(Emotion))) +
  geom_bar(stat="identity",position="dodge") +  scale_fill_brewer(name="Emotion") +
  labs(title='Emotion Scores by Airline (Source)', x="Airline (Source)", y="Mean")
```
