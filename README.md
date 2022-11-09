# Final Project Submission

Please fill out:
* Student name: 
* Student pace: self paced / part time / full time
* Scheduled project review date/time: 
* Instructor name: 
* Blog post URL:


# STUDENT NAME :MERCY MORAA ONDUSO
#STUDENT PACE: PART TIME
#SCHEDULED PROJECT REVIEW DATE: 09/11/2022

expected-to-do:
1. Load and read the data,
2. Check for films that are doing well at the box office,
3. Translate the findings into actionable insights
have a pdf presentation for the heads of dept in Microsoft

# 1.0 Importing libraries

import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
%matplotlib inline
import numpy as np

# 1.1 Loading and reading datasets

movie_titlebasic = pd.read_csv("E:/Data Science/EDA files/title.basics.csv")
movie_titlebasic

movie_gross = pd.read_csv("E:/Data Science/EDA files/bom.movie_gross.csv")
movie_gross

movie_titleratings = pd.read_csv("E:/Data Science/EDA files/title.ratings.csv")
movie_titleratings

# 2.0 Data Understanding

# 2.1 Dataframe Merging

The two files: title.ratings.csv and title.basics.csv have a common column. The column is tconst. This column can help us narrow down our analysis to one dataframe instead of counterchecking from both datasets.
I will use the pd.merge method

#merge titleratings and titlebasic using tconst
movie_ratings = pd.merge(movie_titleratings, movie_titlebasic)
movie_ratings.head()

In the next step, I am merging the 3 datasets as one dataframe to have an easy time analysing the datasets. 
In this case, the 'primary_title' column, in our already merged file, is the same as the 'title' column in the movie_gross dataset.



movie_ratings_gross = movie_ratings.merge(movie_gross, left_on = 'primary_title', right_on = 'title')
movie_ratings_gross.head()

movie_ratings_gross.tail()

From the analysis above: using the head() and the tail() method, there is no criteria in which these datasets were arranged in.

movie_ratings_gross.columns

# 2.2 Dataframe Inspection

Checking the number of columns and rows in the dataset

movie_ratings_gross.shape

Next is checking the data types of the dataframes. As well as checking for missing values

movie_ratings_gross.info()

movie_ratings_gross['foreign_gross'] = pd.to_numeric(movie_ratings_gross['foreign_gross'], errors = 'ignore')

From the above information, the columns with missing values are: 'runtime_minutes','genres', 'studio', 'domestic_gross' and 'foreign_gross'.


The 'genres' column is very sensitive since it will help in the final decision as to which type of movies Microsoft should start investing in. We cannot guess which genre a particular movie is in and so, dropping the null values in this column will help narrow down our analysis.

The 'studio' column as well wil have minimal impact to our final analysis.


# 3.0 Data Cleaning and Scrubbing

movie_ratings_gross1 = movie_ratings_gross.drop(['runtime_minutes', 'studio'], axis = 1)
movie_ratings_gross1.head()

The 'original_title' , 'title' and the 'primary_title' column are the same. Only difference being that, in some instances the 'original_title' is written in the movies' original language and not English. 
In conclusion of the columns overall analysis, there are columns that are better off dropped than maintained from this point onwards.

movie_ratings_gross2 = movie_ratings_gross1.drop(['original_title','year', 'primary_title', 'title'], axis =1)
movie_ratings_gross2.head()

# 3.1 Checking for missing values in columns

movie_ratings_gross2['foreign_gross'].isnull().sum()

movie_ratings_gross2['foreign_gross'].nunique()

There are 1195 null values in the column 'foreign_gross'. So it will be difficult to replace the null values with median, mean and mode.

movie_ratings_gross2.describe()

# 3.2 Checking for duplicates

movie_ratings_gross2.duplicated().sum()

From the analysis above, there are no duplicates in our dataset. So, we proceed to the analytical part of it where the missing values for domestic_gross can be replaced by either the mean and median. 

# 3.3 Statistical aggregation

movie_ratings_gross2['domestic_gross'].mean()

The mean of the domestic_gross is high due to the number of movies that generate exceedingly more revenues as compared to other movies. So we cannot use the mean to replace the null values in the column

movie_ratings_gross2['domestic_gross'].mode()

The mode of the domestic_gross can be an option, if there is need to replace the null values with the necessary aggregate.

movie_ratings_gross2['domestic_gross'].median()

movie_ratings_gross2.isnull().sum()

movie_ratings_gross2['domestic_gross'] = movie_ratings_gross['domestic_gross'].fillna(13000).astype(float)

The number of missing values is less and we can afford to either replace it using mode or median. The other option is 
dropping the missing values and using the remaining dataset in data analysis.

movie_ratings_gross2.shape

movie_ratings_gross2.isna().sum()

Next, checking for outliers in our dataset. Let us sort our domestic_gross col in descending order first
First, Lets fill the missing values in the domestic_gross column using the m

# 3.4 Sorting data columns

rating_genres = movie_ratings_gross2.sort_values('domestic_gross' , ascending = False)
rating_genres

# 3.5 Use IQR to check for the outliers

Q1 = np.percentile(rating_genres['domestic_gross'], 25,
                   interpolation = 'midpoint')
 
Q3 = np.percentile(rating_genres['domestic_gross'], 75,
                   interpolation = 'midpoint')
IQR = Q3 - Q1
lower = (Q1 - 1.5*IQR)
lower

The outliers are extreme. Hence will be ignored

rating_genres1 = rating_genres[rating_genres['averagerating']> 5 ]
rating_genres1.head()

rating_genres1.tail()

rating_genres2 = rating_genres1[rating_genres1['domestic_gross'] >= 200000]
rating_genres2.head()

rating_genres2.tail()

rating_genres2.shape

From the above analysis, looking at the averagerating and the domestic_gross, we can conclude that the most watched and liked genres fall in either categories: action, adventure, Sci-Fi, animation, comedy, drama , documentary or fantasy.Splitting the genres into categories

rating_genres2 = rating_genres.copy()
rating_genres2['genres'] = rating_genres2['genres'].str.split(',').copy()
rating_genres2 = rating_genres2.explode('genres')
rating_genres2['genres'].unique()

# 4.0 Data Visualization

# 4.1 Plotting the categories of genres

rating_genres2.groupby('genres').size().plot(kind='bar')

# 4.2 Comparing the domestic gross and the genres using a barplot

fig, ax = plt.subplots(figsize = (10,6))
ax = sns.barplot(x = 'genres', y = 'domestic_gross', data = rating_genres2)
plt.xticks(rotation = 60, fontsize = 12)
ax.set_xlabel('genres', fontsize = 15)
ax.set_ylabel('domestic_gross', fontsize = 15)
ax.set_title('Domestic Gross vs Genres', fontsize = 25);

From the bar graph above, the leading 5 genres in terms of domestic gross are: Sci-fi, Adventure, Animation, Action and Fantasy

# 4.3 Comparing Average Rating and Domestic Gross using a barplot

plt.figure(figsize = (8, 6))
x = rating_genres2.averagerating
y = rating_genres2.domestic_gross
plt.bar(x,y, color = 'green')
plt.xlabel('averagerating')
plt.ylabel('Domestic gross')
plt.title('Domestic Gross vs Average Rating')
plt.show();

From the bar graph above, the higher the Average rating, the more the domestic gross and the lower the Average rating the lesser the domestic gross

# 4.4  Comparing the genres and domestic gross using a scatterplot

sns.catplot(data=rating_genres2, x=rating_genres2['domestic_gross'], y= rating_genres2['genres']);

From the above scatter plot, the leading 5 genres in the box office are: Action, Adventure, Sci-Fi, Fantasy and Animation 

# 5. Recommendation

From the above analysis, I would advise the new head of Microsoft's studio to invest in producing movies that are expected to have good returns to the company. The genres doing well are : Action, Adventure, Sci-Fi, Animation, Fantasy, Comedy, Thriller and Comedy.
A combination of either of the top genres is also recommended as is seen in the datasets provided. 
The genres to extremely avoid are those with the least returns in terms of domestic gross. The least in returns are: News, War, Western, Musical and Music