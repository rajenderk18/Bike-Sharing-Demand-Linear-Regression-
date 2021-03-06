# Bike-Sharing-Demand-Linear-Regression
This project focuses on conducting Exploratory Data Analysis and running Linear Regression on Bike Sharing Demand data set which was provided by Hadi Fanaee Tork using data from Capital Bikeshare.

## What is Bike Sharing Systems ?
Bike sharing systems are a means of renting bicycles where the process of obtaining membership, rental, and bike return is automated via a network of kiosk locations throughout a city. Using these systems, people are able rent a bike from a one location and return it to a different place on an as-needed basis. Currently, there are over 500 bike-sharing programs around the world.

## Objective
The main objective of this project is to explore and create a linear Regression Model so as to try to predict bike sharing demand.

```{r libraries}
library(ggplot2)
library(dplyr)
```

```{r input data}
bikeshare <- read.csv("E:/Udemy Courses/R programming AZ Udemy/bike-sharing-demand/train.csv")
View(bikeshare)
```

## Features of Data

```{r data feature}
print(head(bikeshare))

```
The data set contains following features :-

<li> datetime - hourly date + timestamp </br>
<li> season - 1 = spring, 2 = summer, 3 = fall, 4 = winter </br>
<li> holiday - whether the day is considered a holiday </br>
<li> workingday - whether the day is neither a weekend nor holiday </br>
<li> weather - </br>
    
     1: Clear, Few clouds, Partly cloudy, Partly cloudy 
     2: Mist + Cloudy, Mist + Broken clouds, Mist + Few clouds, Mist 
     3: Light Snow, Light Rain + Thunderstorm + Scattered clouds, Light Rain + Scattered clouds 
     4: Heavy Rain + Ice Pallets + Thunderstorm + Mist, Snow + Fog 

<li> temp - temperature in Celsius </br>
<li> atemp - "feels like" temperature in Celsius </br>
<li> humidity - relative humidity </br>
<li> windspeed - wind speed </br>
<li> casual - number of non-registered user rentals initiated </br>
<li> registered - number of registered user rentals initiated </br>
<li> count - number of total rentals </br>
</li>

## What we are trying to predict ?

We are trying to predict count variable i.e. number of total rentals which will be behaving as a dependent variable for our analysis. 

## Exploratory Data Analysis

First I will be conducting exploratory data analysis which is an essential step to undestand the data.

### Loading the ggplot library

```{r library}
library(ggplot2)
```

## Scatter Plot to show the relationship between count (number of total rentals) and temp (temperature in Celsius)
```{r scatter_plot_1}
ggplot(data = bikeshare, aes(temp,count)) + geom_point(alpha = 0.3, aes(color = temp)) + theme_bw() +
  geom_smooth(method="loess", se=F) +
  labs(title="Bike Count Vs Temperature", 
       y="Bike Count", 
       x="Temperature")
```

The above scatter plot shows that as the temperature increases the count i.e. the number of total rentals also increases.

## Scatter Plot to show the relationship between count (number of total rentals) and date time.
```{r scatter_plot_2}
bikeshare$datetime <- as.POSIXct(bikeshare$datetime)
pl <- ggplot(bikeshare,aes(datetime,count)) + geom_point(aes(color=temp),alpha=0.5)
pl + scale_color_continuous(low = '#55D8CE',high = '#FF6E2E') + theme_bw()
```

There is a clear seasonal trend where the total rental bikes seems to decrease during Winters i.e month of January and February of the year and the total rental bikes seems to increase during summers.

The other trend which is quite evident is that the number of rental bike counts is increasing from year 2011 to year 2013.

## Correlation between temperature and count.

```{r correlation between temp and count}
cor(bikeshare[,c('temp','count')])
```
There is not so strong correlation between temp and count.

## Box Plot

```{r box plot}
ggplot(bikeshare,aes(factor(season),count)) + geom_boxplot(aes(color = factor(season))) + theme_bw()
```

The box plot between the number of bike rentals and season shows that the line can not capture the non linear relationship and that there's is more rentals in winter as compared to spring.


## Feature Engineering
As part of feature engineering I have added an hour column in the dataset.

```{r feature engineering}
bikeshare$hour <- sapply(bikeshare$datetime,function(x){format(x,"%H")})
bikeshare$hour <- sapply(bikeshare$hour,as.numeric)
print(head(bikeshare))
```

## Relationship between hour of the working day and the count of bikes rented.

```{r scatter_plot_3}
pl1 <- ggplot(filter(bikeshare,workingday == 1), aes(hour,count))
pl1 <- pl1+ geom_point()
print(pl1)
```

This scatter plot shows an interesting trend where count of rented bikes increases during the evening hours when people leave from office i.e. around 5 PM and morning hours when people leave for office i.e. around 8 AM.

```{r scatter_plot_3_more}
pl1 <- pl1 + geom_point(position=position_jitter(w=1,h=0),aes(color = temp),alpah=0.5)
pl1 <- pl1 + scale_color_gradientn(colours = c('dark blue','blue','light blue','light green','yellow','orange','red'))
print(pl1 + theme_bw())
```

This plot gives an interesting finding regarding temperature and bike rental count.
As the temperature increases i.e. gets hotter the count of bike rental increases and for cold temperature there is a decline in count of bike rental.

## Relationship between hour of the non-working day and the count of bikes rented.

```{r non-working days}
pl2 <- ggplot(filter(bikeshare,workingday == 0), aes(hour,count))
pl2 <- pl2+ geom_point()
pl2 <- pl2 + geom_point(position=position_jitter(w=1,h=0),aes(color = temp),alpah=0.5)
pl2 <- pl2 + scale_color_gradientn(colours = c('dark blue','blue','light blue','light green','yellow','orange','red'))
print(pl2 + theme_bw())
```

During non working days there is very less bike rental during morning hours and it eventually increases after noon.

## Model Building

This model will be predicting the count of the bike rental based on the temp variable.

```{r model_build1}
temp.model <- lm(count ~ temp, bikeshare)
print(summary(temp.model))
```

## Model Interpretation
** Based on the value of Intercept which is 6.0462, linear regression model predicts that there will be 6 bike rental when the temperature is 0.
** For temp variable Estimated Std. value is 9.1705 which signifies that a temperature increase of 1 Celsius holding all things equal is associated with a rental increase of about 9.1 bikes. </br>
** The above findings is not a Causation and Beta 1 would be negative if an increase in temperature was associated with a decrease in rentals.</br>

Next we want to know is how many bikes would we predict to be rented if the temperature was 25 degrees Celsius.

## How many rented bikes at temperature 25 degrees Celsius
```{r calculate}
6.0462 + 9.1705 * 25
temp.test <- data.frame(temp=c(25))
predict(temp.model,temp.test)

```
Based on the above calculation we can say that the number of bikes rented at 25 degrees Celsius temperature will be 235.30

## Building Second Model with more features
Model that attempts to predict count based off of the following features :-
 season
 holiday
 workingday
 weather
 temp
 humidity
windspeed
hour (factor) 

```{r model_build2}
model <- lm(count ~ . -casual - registered - datetime - atemp, bikeshare)
print(summary(model))
```

## Important Insights
This sort of model doesn't work well given our seasonal and time series data. We need a model that can account for this type of trend. We will get thrown off with the growth of our dataset accidentally attributing to the winter season instead of realizing it's just overall demand growing.

## Acknowledgements
This dataset was provided by Hadi Fanaee Tork using data from <a href= 'https://www.capitalbikeshare.com/system-data'>Capital Bikeshare </a>. We also thank the UCI machine learning repository for hosting the dataset. 