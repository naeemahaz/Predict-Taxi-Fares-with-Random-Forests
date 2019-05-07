# Predict-Taxi-Fares-with-Random-Forests
To drive a yellow New York taxi, you have to hold a "medallion" from the city's Taxi and Limousine Commission. Recently, one of those changed hands for over one million dollars, which shows how lucrative the job can be.  But this is the age of business intelligence and analytics! Even taxi drivers can stand to benefit from some careful investigation of the data, guiding them to maximize their profits. In this project, we will analyze a random sample of 49999 New York journeys made in 2013. We will also use regression trees and random forests to build a model that can predict the locations and times when the biggest fares can be earned.  Let's start by taking a look at the data!

Task 1: Instructions





Read in the data consisting of 49999 New York taxi journeys.
Load in the tidyverse package (which includes dplyr and ggplot2).
Read in the datasets/taxi.csv file using the read_csv() function (not read.csv()) and store the resulting data frame into taxi. 
Take a look at the first couple of rows in taxi by using the head() function.


Good to know

This Project assumes you have used the dplyr and ggplot2 packages and that you are familiar with the pipe operator (%>%). Before taking on this Project, we recommend that you have completed the course Introduction to the Tidyverse.

RStudio has created some very helpful cheat sheets, including two that will be helpful for this Project: Data Wrangling and Data Visualization with ggplot2. We recommend that you keep them open in a separate tab to make it easy to refer to them. 

Hint



If you've loaded in the tidyverse
library(tidyverse)


you can read in path_to/my_data.csv like this:
my_data <- read_csv("path_to/my_data.csv")


This is an introduction to the programming language R, focused on a powerful set of tools known as the "tidyverse". In the course you'll learn the intertwined processes of data manipulation and visualization through the tools dplyr and ggplot2. You'll learn to manipulate data by filtering, sorting and summarizing a real dataset of historical country data in order to answer exploratory questions. You'll then learn to turn this processed data into informative line plots, bar plots, histograms, and more with the ggplot2 package. This gives a taste both of the value of exploratory data analysis and the power of tidyverse tools. This is a suitable introduction for people who have no previous experience in R and are interested in learning to perform data analysis.

code:
# Loading the tidyverse
# .... YOUR CODE FOR TASK 1 ....

# Reading in the taxi data
taxi <- ....

# Taking a look at the first couple of rows in taxi
# .... YOUR CODE FOR TASK 1 ....




-----------------------------------------------------------------------------------------------------------------------------------


2. Cleaning the taxi data

As you can see above, the taxi dataset contains the times and price of a large number of taxi trips. Importantly we also get to know the location, the longitude and latitude, where the trip was started.

Cleaning data is a large part of any data scientist's daily work. It may not seem glamorous, but it makes the difference between a successful model and a failure. The taxi dataset needs a bit of polishing before we're ready to use it.

Task 2: Instructions





Rename, cleanup and add a new variable to the taxi dataset. All this as part of a single %>%-pipeline.
rename the columns pickup_latitude and pickup_longitude into the shorter lat and long.
filter so to keep only those rows where fare_amount or tip_amountis larger than zero (> 0).
Finally, mutate taxi to include a new column called total calculated as the log() of fare_amount + tip_amount.


The reason to define total as the the log() of fare_amount + tip_amount is that by taking the log we remedy the effect of outliers by making really large numbers smaller.

Hint



dplyr has a function called rename(), which leaves the data unchanged but changes the variable name in a data frame instead. If you downloaded data on the price of gold and it had an ugly variable name like gpr1_, you could rename it like this:
golddata <- golddata %>% rename(gpr1_,gold_price)


There is also a function called mutate(), which makes a new variable that is a function of some existing variables. Suppose your gold prices were easier to interpret as a percentage of the price at the start of the data, you could do it like this:
golddata <- golddata %>% mutate(gold_price_index = 100 * gold_price / gold_price[1])


code: 

# Renaming the location variables,
# dropping any journeys with zero fares and zero tips,
# and creating the total variable as the log sum of fare and tip
taxi <- taxi %>%
   ....



---------------------------------------------------------------------------------------------------

3. Zooming in on Manhattan

While the dataset contains taxi trips from all over New York City, the bulk of the trips are to and from Manhattan, so let's focus only on trips initiated there.


Task 3: Instructions





Filter the data (taxi) down to a rectangle containing Manhattan.
filter taxi so that you keep only those rows in where lat is between 40.70 and 40.83, and long is between -74.025 and -73.93.
Assign the result back to taxi.


dplyr contains a function called between which can simplify the code for this task. For how to use between with filter check out this stack overflow answer.






Hint



If dates$month is a vector of month numbers (1-12) and dates$month is a vector with day numbers (1-31) then the following would keep all the days of the first week in January, February and March:
dates  %>% 
    filter(between(day, 1, 7) &
           between(month, 1, 3))
           
           
code:
# Reducing the data to taxi trips starting in Manhattan
# Manhattan is bounded by the rectangle with 
# latitude from 40.70 to 40.83 and 
# longitude from -74.025 to -73.93
taxi <- taxi  %>% 
    filter(....)

--------------------------------------------------------------------------------------------------


4. Where does the journey begin?

It's time to draw a map! We're going to use the excellent ggmap package together with ggplot2 to visualize where in Manhattan people tend to start their taxi journeys.

Task 4: Instructions





Draw a map of Manhattan with a summary of the number of journeys on top.
Load in the ggmap package and the viridis package (which includes a nice color scale).
Complete the ggmap call by adding (+) a geom_bin2d layer with data = taxi, long on the x-axis and lat on the y-axis, bins = 60, and alpha = 0.6.
Add (+) labels using the labs command for the x, y and fill dimensions.


For the syntax of geom_bin2d, and for examples of usage, check out the official ggplot2 documentation.






Hint



Here is how the ggmap call could look, just replace the ....-parts.
ggmap(manhattan, darken=0.5)+
    scale_fill_viridis(option='plasma') +
    geom_bin2d(data = ...., aes(x = ...., y = ....), bins = 60, alpha = 0.6) +
    labs(x='Longitude', y='Latitude', fill='Journeys')

code:

# Loading in ggmap and viridis for nice colors
# .... YOUR CODE FOR TASK 4 ....

# Retrieving a stored map object which originally was created by
# manhattan <- get_map("manhattan", zoom = 12, color = "bw")
manhattan <- readRDS("datasets/manhattan.rds")

# Drawing a density map with the number of journey start locations
ggmap(manhattan, darken = 0.5) +
   scale_fill_viridis(option = 'plasma') +
   # .... YOUR CODE FOR TASK 4 .... 
   
--------------------------------------------------------------------------------------------------------------


5. Predicting taxi fares using a tree

The map from the previous task showed that the journeys are highly concentrated in the business and tourist areas. We also see that some taxi trips originating in Brooklyn slipped through, but that's fine. 

We're now going to use a regression tree to predict the total fare with lat and long being the predictors. The tree algorithm will try to find cutpoints in those predictors that results in the decision tree with the best predictive capability. 






Task 5: Instructions





Fit and visualize a regression tree to predict total using lat and long.
Load in the tree package.
Use the tree() function to fit a regression tree with total as the outcome and lat and long as the predictors. Assign the result to fitted_tree.
Visualize fitted_tree by using the plot() and text() functions.


tree() takes a formula (outcome ~ predictor1 + predictor2 + ...) as it's first argument and the data frame as the second. For more information see the documentation of the tree function. If fitted_tree is the output of tree then
plot(fitted_tree)
text(fitted_tree)


draws the tree.






Hint



Here is some template code you can use, just replace the ....:
# Fitting a tree to lat and long
fitted_tree <- tree(total ~ .... + ...., data = ....)

# draw a diagram of the tree structure
plot(....)
text(....)

Code:
# Loading in the tree package
# .... YOUR CODE FOR TASK 5 HERE ....

# Fitting a tree to lat and long
fitted_tree <- ....

# Draw a diagram of the tree structure
# .... YOUR CODE FOR TASK 5 HERE ....

----------------------------------------------------------------------------------------------------------------

6. It's time. More predictors.¶

The tree above looks a bit frugal, it only includes one split: It predicts that trips where lat < 40.7237 are more expensive, which makes sense as it is downtown Manhattan. But that's it. It didn't even include long as tree deemed that it didn't improve the predictions. Taxi drivers will need more information than this and any driver paying for your data-driven insights would be disappointed with that. As we know from Robert de Niro, it's best not to upset New York taxi drivers.

Let's start by adding some more predictors related to the time the taxi trip was made.

Task 6: Instructions





Use the pickup_datetime column to add the hour, weekday, and month each trip was made.
Load in the lubridate package.
mutate the taxi dataset to include the new columns hour, wday, and month. Each new column should be created from the pickup_datetime using either of the functions hour(), wday(), and month().
Make sure to set the argument label = TRUE when using wday() and month().


taxi$pickup_datetime describes when each taxi trip was started and lubridate includes many functions, like hour(), which extract information from a datetime object. See the documentation for lubridate.






Hint



Here is some template code to get you started. Just replace the .... parts:
# Generate the three new time variables
taxi <- taxi %>% 
    mutate(hour = hour(....),
           wday = wday(...., label = TRUE),
           month = month(...., label = TRUE))

code: 
# Loading in the lubridate package
# .... YOUR CODE FOR TASK 6 HERE ....

# Generate the three new time variables
taxi <- taxi %>% 
    mutate(....)
    
-----------------------------------------------------------------------------------------------------------

7. One more tree!

Let's try fitting a new regression tree where we include the new time variables.

Task 7: Instructions





Fit a new regression tree that also includes hour, wday, and month.
Like before, fit a regression tree using tree() and assign the result to fitted_tree. But this time include both lat, long, hour, wday, and month as predictors.
Visualize the resulting fitted_tree using plot() and text().
Print a text summary of fitted_tree by using the summary() function.






Hint



Here is some template code you can use, just replace the ....:
# Fitting a tree to lat and long
fitted_tree <- tree(total ~ lat + long + .... + .... + ...., data = taxi)

# draw a diagram of the tree structure
plot(....)
text(....)

# Summarizing the performance of the tree
summary(....)

code: 
# Fitting a tree with total as the outcome and 
# lat, long, hour, wday, and month as predictors
fitted_tree <- ....

# draw a diagram of the tree structure
# .... YOUR CODE FOR TASK 7 HERE ....

# Summarizing the performance of the tree
# .... YOUR CODE FOR TASK 7 HERE ....

------------------------------------------------------------------------------------------------
8. One tree is not enough

The regression tree has not changed after including the three time variables. This is likely because latitude is still the most promising first variable to split the data on, and after that split, the other variables are not informative enough to be included. A random forest model, where many different trees are fitted to subsets of the data, may well include the other variables in some of the trees that make it up. 

Task 8: Instructions





Instead of fitting a regression tree to the data, let's fit a random forest instead.
Load in the randomForest package.
Use the randomForest() function to fit at random forest with the same predictors as the for tree in the last task. Assign the result to fitted_forest.
Set the following arguments to randomForest() to speed up the computation: ntree = 80 and sampsize = 10000
Print fitted_forest to show a summary of the result.


randomForest() uses the same basic syntax as the tree() function. That is, the first argument is a formula and the second argument is the data.






Hint



Here is some code to get you started, just replace the ....:
# Fitting a random forest
fitted_forest <- ....(total ~ lat + long + hour + wday + month,
    data=taxi, ntree=80, sampsize=10000)

# Printing the forest_fit object
fitted_forest


code: 

# Loading in the randomForest package
# .... YOUR CODE FOR TASK 8 HERE ....

# Fitting a random forest
fitted_forest <- ....

# Printing the fitted_forest object
# .... YOUR CODE FOR TASK 8 HERE ....

-------------------------------------------------------------------------------------------------
9. Plotting the predicted fare

In the output of fitted_forest you should see the Mean of squared residuals, that is, the average of the squared errors the model makes. If you scroll up and check the summary of fitted_tree you'll find Residual mean deviance which is the same number. If you compare these numbers, you'll see that fitted_forest has a slightly lower error. Neither predictive model is that good, in statistical terms, they explain only about 3% of the variance. 

Now, let's take a look at the predictions of fitted_forest projected back onto Manhattan.
Task 9: Instructions





Extract and plot the prediction of the fitted random forest.
fitted_forest$predicted contains the predictions for the datapoints in taxi. Assign fitted_forest$predicted to taxi$pred_total.
Copy in the plotting code from task 4.
Replace geom_bin2d by stat_summary_2d but keep the arguments. Add z = pred_total to the aes call and fun = mean as an argument.
Change fill label to something suitable.


stat_summary_2d is similar to geom_bin2d but takes the data connected to z and applies a function to it defined by the fun = argument. See the documentation for stat_summary_2d.






Hint



Here is some code to get you started, just replace the ....:
# Plotting the predicted mean trip prices from according to the random forest
ggmap(manhattan, darken=0.5) +
    scale_fill_viridis(option = 'plasma') +
    stat_summary_2d(data=taxi, aes(x = long, y = lat, z = ....),
                    fun = ...., alpha = 0.6, bins = 60) +
    labs(x = 'Longitude', y = 'Latitude', fill = '....')


code:
# Extracting the prediction from fitted_forest
taxi$pred_total <- ....

# Plotting the predicted mean trip prices from according to the random forest
# .... COPY CODE FROM TASK 4 AND MODIFY HERE ....

--------------------------------------------------------------------------------------------------------------
10. Plotting the actual fare

Looking at the map with the predicted fares we see that fares in downtown Manhattan are predicted to be high, while midtown is lower. This map only shows the prediction as a function of lat and long, but we could also plot the predictions over time, or a combination of time and space, but we'll leave that for another time.

For now, let's compare the map with the predicted fares with a new map showing the mean fares according to the data.

Task 10: Instructions





Plot the mean fare of the fitted random forest.
Copy in the plotting code from the previous task.
Change z = pred_total to z = total.
Change fun = mean to use the defined function, that is, into fun = mean_if_enough_data.


You could use fun = mean for this plot, but that will leave a lot of squares with just a few data points which will be visually distracting. Therefore you'll use mean_if_enough_data instead which only returns the mean if there are 15 or more datapoints.






Hint



Here is some code to get you started, just replace the ....:
# Plotting the mean trip prices from the data
ggmap(manhattan, darken=0.5) +
    stat_summary_2d(data=taxi, aes(x = long, y = lat, z = ....),
                    fun = ....,
                    alpha = 0.6, bins = 60) +
  scale_fill_viridis(option = 'plasma') +
  labs(x = 'Longitude', y = 'Latitude', fill = 'Log fare+tip'



code:
# Function that returns the mean *if* there are 15 or more datapoints
mean_if_enough_data <- function(x) { 
    ifelse( length(x) >= 15, mean(x), NA) 
}

# Plotting the mean trip prices from the data
# .... COPY CODE FROM TASK 9 AND MODIFY HERE ....

----------------------------------------------------------------------------------------------------------
11. Where do people spend the most?

So it looks like the random forest model captured some of the patterns in our data. At this point in the analysis, there are many more things we could do that we haven't done. We could add more predictors if we have the data. We could try to fine-tune the parameters of randomForest. And we should definitely test the model on a hold-out test dataset. But for now, let's be happy with what we have achieved!

So, if you are a taxi driver in NYC, where in Manhattan would you expect people to spend the most on a taxi ride?

Task 11: Instructions





Given the data and our model, where do people spend the most on taxi trips?
Change the value of spends_most_on_trips to reflect where people spend the most on taxi trips. "uptown" or "downtown"?

Hint: Downtown is the lower tip of Manhattan while uptown refers to the upper half of Manhattan.


If you want to know more

Regression trees and random forests are covered in the DataCamp course Supervised Learning In R: Regression.


Read more about the ggmap package here.







Hint



Look at the two last plots you produced. Do people pay more downtown (the lower tip of Manhattan) or uptown?


code: 
# Where are people spending the most on their taxi trips?
spends_most_on_trips <- "...." # "uptown" or "downtown"
