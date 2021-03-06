# Spotify Artist Popularity Predictor

# Introduction: 
In this project I analyzed the discographic characteristics of 300,000 tracks scraped from Spotify’s API to identify which are the largest influencers on artist popularity and produce a predictive model for artist popularity. 

Given that hundreds of albums are produced each year, There has to be an objective way for us to tell the greatness of an artist without relying on critics or our own instincts.

# Project Goals: 
Analyze the trend of music development over the past 100 years and produce models to predict artist popularity through constructing Multiple Linear Regreession and Random Forrest models. 

Here’s a quick summary of my approach:

* Aquire Spotify API Client ID and Secret ID
* Get the data using Spotify's API
* Process the data to extract audio features for each artist
* Analyze and visualize each discographic feature for all the artists
* Apply Multiple Linear Regression and Random Forest modeling to predict artist popularity

# Data Extraction and Cleaning: 
I used [Spotify’s awesome API](https://developer.spotify.com/documentation/web-api/) to extract discographic data and number of followers for each of 2185 celebrities over the past 100 years. (Follow steps on Spotify's webpage to set up a developer account!) 
```R
devtools::install_github('charlie86/spotifyr')
library(spotifyr)

#set system enviroment variables for client ID and Secret ID
credentials <- read.csv('developer_credentials.csv')
Sys.setenv(SPOTIFY_CLIENT_ID = as.character(credentials$client_id))
Sys.setenv(SPOTIFY_CLIENT_SECRET = as.character(credentials$client_secret_id))

access_token <- get_spotify_access_token()

#pull in artist metrics such as genere, num followers, and popularity
spotify_artists <- read.csv('spotify_artists.csv')
for (i in 1:nrow(spotify_artists)) {
  if (i == 1){
    numFollowers <- get_artists(as.character(spotify_artists$artist_name[i]))
  }else {
    tryCatch({
      numFollowers <- rbind(numFollowers, get_artists(as.character(spotify_artists$artist_name[i])))},error=function(cond)
      {
        NA
      })
  }
  cat(paste0('\nFinished: ', i, ". Starting next loop on " , i+1, sep =' '))
}

#pull in album audio features 
for (i in 1:nrow(spotify_artists)) {
  if (i == 1){
    final_data <- get_artist_audio_features(as.character(spotify_artists$artist_name[i]))
  }else {
    tryCatch({
      final_data <- rbind(final_data, get_artist_audio_features(as.character(spotify_artists$artist_name[i])))},error=function(cond)
      {
        NA
      })
  }
  cat(paste0('\nFinished: ', i, ". Starting next loop on " , i+1, sep =' '))
}
```
I merged all the data into a single data frame and started feature manipulation to ensure the models work as intended. Some examples of feature manipulation are removing NA’s, converting categorical features to numeric, converting character string to an actual date/time type, and simplifying the genres feature to include only the first element of each tracks genre list. 
The final cleaned data includes (check out the final_data_csv file to get a glimps of the cleaned dataset used for analyses + prediciton modeling):   
    • Aggregated discographic features for each artist’s album   
    • Number of Spotify followers (updated 8/7/2018)                                           
    • Non-numeric features such as: artist name, album name, genres, and key mode 
    
   ### Sample Data-Set Output
   
artist_name | album_name | album_release_date | album_popularity | danceability | energy | loudness | speechiness | acousticness | instrumentalness | liveness | valence | tempo | duration_ms | track_popularity | artist_generes | artist_popularity | artist_num_followers   
--- | --- | --- |--- | --- | --- |--- | --- | --- |--- | --- | --- | --- | --- | --- | --- | --- | ---   
Drake | What A Time To Be Alive | 2015-09-25 | 43 | 0.7553636 | 0.5016364 | -8.651182 | 0.28931818 | 0.14069227 | 2.729091e-04 | 0.1334273 | 0.2900909 | 139.35818 | 221451.36 | 33.54545455 | canadian hip hop | 100 | 23980695
Post Malone | Stoney | 2016-12-09 | 71 | 0.6064286 | 0.6110714 | -6.498643 | 0.06784286 | 0.28190714 | 4.127143e-06 | 0.1321143 | 0.3184286 | 116.58379 | 217631.36 | 59.57142857 | pop | 94 | 5429208
J Balvin | Vibras | 2018-05-25 | 84 | 0.6547571 | 0.6224286 | -7.313286 | 0.15067857 | 0.29990000 | 5.273435e-02 | 0.1759071 | 0.6045286 | 135.57214 | 187903.86 | 73.00000000 | latin | 92 | 9449493 

   ### Audio features description
   
The description of each feature from the Spotify Web API Guidance can be found below:

* **Danceability:** describes the suitability of a track for dancing. This is based on a combination of musical elements including tempo, rhythm stability, beat strength, and overall regularity. A value of 0.0 is least danceable and 1.0 is most danceable.
* **Energy:** a measure from 0.0 to 1.0, and represents a perceptual measure of intensity and activity. Typically, energetic tracks feel fast, loud, and noisy. For example, death metal has high energy, while a Bach prelude scores low on the scale. Perceptual features contributing to this attribute include dynamic range, perceived loudness, timbre, onset rate, and general entropy.
* **Speechiness:** Speechiness detects the presence of spoken words in a track. The more exclusively speech-like the recording (e.g. talk show, audio book, poetry), the closer to 1.0 the attribute value. Values above 0.66 describe tracks that are probably made entirely of spoken words. Values between 0.33 and 0.66 describe tracks that may contain both music and speech, either in sections or layered, including such cases as rap music. Values below 0.33 most likely represent instrumental music and other non-speech-like tracks.
* **Acousticness:** a confidence measure from 0.0 to 1.0 of whether the track is acoustic. 1.0 represents high confidence that the track is acoustic.
* **Loudness:** the overall loudness of a track in decibels (dB). Loudness values are averaged across the entire track and are useful for comparing the relative loudness of tracks. Loudness is the quality of a sound that is the primary psychological correlate of physical strength (amplitude). Values typical range between -60 and 0 db.
* **Instrumentalness:** predicts whether a track contains no vocals. “Ooh” and “aah” sounds are treated as instrumental in this context. Rap or spoken word tracks are clearly “vocal”. The closer the instrumentalness value is to 1.0, the greater likelihood the track contains no vocal content. Values above 0.5 are intended to represent instrumental tracks, but confidence is higher as the value approaches 1.0.
* **Liveness:** detects the presence of an audience in the recording. Higher liveness values represent an increased probability that the track was performed live. A value above 0.8 provides strong likelihood that the track is live.
Valence: a measure from 0.0 to 1.0 describing the musical positiveness conveyed by a track. Tracks with high valence sound more positive (for example happy, cheerful, euphoric), while tracks with low valence sound more negative (for example sad, depressed, angry).
* **Tempo:** the overall estimated tempo of a track in beats per minute (BPM). In musical terminology, tempo is the speed or pace of a given piece, and derives directly from the average beat duration.
* **Duration_ms:** the duration of the track in milliseconds.
___
# Exploratory Data Analysis and Data visualization for Determining Feature Predictive Importance:(SECTION NOT COMPLETE)
  ## Popularity Analysis by numeric discographic features(MISSING DESC.)
 ![alt text](https://github.com/TigranMelkonian/Spotify_Artist_Popularity_Predictor/blob/master/discography_histograms.png "Numeric Discography Distributions")
 
 * track_popularity: 
 * artist_num_followers: 
 * loudness: 
 * danceability: 
 * energy: 
 * accousticness: 
 * instrumentalness: 
 * tempo: 
 * speechiness: 
 * liveness: 
 * valence: 
 * duration_ms: 
 
 
 ![alttext](https://github.com/TigranMelkonian/Spotify_Artist_Popularity_Predictor/blob/master/regression_plot_artist_popularity.png "Regression plots")
 
 * track_popularity: 
 * artist_num_followers: 
 * loudness: 
 * danceability: 
 * energy: 
 * tempo: 
 * accousticness: 
 * instrumentalness: 
 * speechiness: 
 * valence: 
 * duration_ms: 
 * liveness: 
 
 
   ### Correlation Matrix Between Numeric Discographic Features(MISSING DESC.)
   ![alttext](https://github.com/TigranMelkonian/Spotify_Artist_Popularity_Predictor/blob/master/correlation_matrix.png "Correlation Matrix")
      The above plot shows that our predictor variables are generally indepedent, except for a strong positive correlation between loudness and energy and a strong negative correlation between energy and acousticness.
   
  ## Popularity Analysis by Genres(MISSING DESC. and visual)
  
  ## Prediction Model Discographic Feature Selection
Although initially I scraped 15 variables from Spotify pertaining to album level discographic data, many variables  were not applicable to predict artist popularity due to poor correlations, so I  only selected the 7 variables that showcased relatively strong corelations with artist popularity and promising regression relations from the plots above.

  ## Creating the Training and Testing Data-sets
The original dataset was randomly divided into two parts, 75% of the albums were treated as the training set, and the rest 25% belonged to the testing set.
```R
#75% of the sample size
smp_size <- floor(0.75 * nrow(aggData))

#set the seed to make partition reproducible
set.seed(123)
train_ind <- sample(seq_len(nrow(aggData)), size = smp_size)

train <- aggData[train_ind, ]
test <- aggData[-train_ind, ]
```
  ## Multiple Linear Regression (INCOMPLETE)
   Multiple Linear Regression model was fitted to predict artist popularity using the following variables:
   * Track Popularity
   * Number of Spotify Followers
   * Loudness
   * Danceability
   * Energy
   * Acousticness
   * Instrumentalness
```R
#multiple linear regression model
linearModel <- lm(artist_popularity ~ track_popularity + artist_num_followers + loudness + danceability + energy + acousticness + instrumentalness, data = train)
linearPred <- predict(linearModel, newdata = test)
lmModelAccuracyAdjusted <- accuracy(test$artist_popularity, linearPred)

#create dataset for Multiple Linear Regression
lmPred <- data.frame(linearPred)
lmPred$actual <- test$artist_popularity
```
  ### Predicted artist_popularity Accurracy(MISSING DESC.)
  
_ | ME | RMSE | MAE | MPE | MAPE 
--- | --- | --- |--- | --- | --- 
Test set | -0.2322359 | 10.6027 | 8.028139 | -0.20649 | 12.78574
  
  ## Random Forrest Regression (INCOMPLETE) 
   Random Forest model was fitted to predict artist popularity using the following variables:
   * Track Popularity
   * Number of Spotify Followers
   * Loudness
   * Danceability
   * Energy
   * Acousticness
   * Instrumentalness
```R
#Random Forest model
model <- randomForest(artist_popularity ~ track_popularity + artist_num_followers + loudness + danceability + energy + acousticness + instrumentalness, data = train)
pred <- predict(model, newdata = test)
rfModelAccuracyAdjusted <- accuracy(test$artist_popularity, pred)

#create dataset for  Random Foreest results
rfPred <- data.frame(pred)
rfPred$actual <- test$artist_popularity
```
  ### Predicted artist_popularity Accurracy(MISSING DESC.)
  
_ | ME | RMSE | MAE | MPE | MAPE 
--- | --- | --- |--- | --- | --- 
Test set | -0.05248856 | 4.049456 | 2.78827 | 0.1950373 | 4.853123

 ## Results
After testing my model I observed that the Random Forrest model is more accurate than the multiple linear regression model with considerably lower mean percent error and mean average percent error. Moving forward, I would like to explore how additional features such as artist hometown location (Data available from Wikipedia), genre, album release date, and celebrity status (score I am planing on creating which will be derived from social media (Facebook, Twitter, Instagram) following and Wikipedia page views) can influence the Random Forrest mdoel's prediction accuracy. Additionally I believe the accuracy of the model may increase if I use a larger dataset including more artists. Ultimatly I am satisfied with the Random Forrest model's current accuracy, but am looking forward to continue building on this project. 
