# Import The Relevant Libraries 
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
#Phase 1 Project Description
##Project Overview
For this project, you will use exploratory data analysis to generate insights for a business stakeholder.

##Business Problem
Microsoft sees all the big companies creating original video content and they want to get in on the fun. They have decided to create a new movie studio, but they don’t know anything about creating movies. You are charged with exploring what types of films are currently doing the best at the box office. You must then translate those findings into actionable insights that the head of Microsoft's new movie studio can use to help decide what type of films to create.

##The Data
(data/bom.movie_gross.csv) 
(data/rt.movie_info.tsv)
(data/rt.reviews.tsv) 
(data/tmdb.movies.csv)
(data/tn.movie_budgets.csv)

I opted to use (data/bom.movie_gross.csv),   and (data/tn.movie_budgets.csv)
Because it was collected from various locations, the different files have different formats. Some are compressed CSV (comma-separated values) or TSV (tab-separated values) files that can be opened using spreadsheet software or pd.read_csv, while the data from IMDB is located in a SQLite database.
# Load the datasets ('bom.movie_gross.csv')
bom_movie_gross_df= pd.read_csv('data/bom.movie_gross.csv')
#Display the first few rows of the DataFrame to understand its structure
bom_movie_gross_df.head()

# Load the datasets ('tn.movie_budgets')
tn_movie_budgets_df = pd.read_csv('data/tn.movie_budgets.csv')
#Display the first few rows of the DataFrame to understand its structure
tn_movie_budgets_df.head()

#Step 2: Clean the Data
#We will clean the column names for consistency and convert appropriate columns to numeric types.
# Clean column names for consistency
bom_movie_gross_df.columns = [col.lower().replace(' ', '_') for col in bom_movie_gross_df.columns]
tn_movie_budgets_df.columns = [col.lower().replace(' ', '_') for col in tn_movie_budgets_df.columns]

bom_movie_gross_df['domestic_gross'] = pd.to_numeric(bom_movie_gross_df['domestic_gross'], errors='coerce')
bom_movie_gross_df['foreign_gross'] = pd.to_numeric(bom_movie_gross_df['foreign_gross'], errors='coerce')

# Convert columns to numeric types
tn_movie_budgets_df['production_budget'] = tn_movie_budgets_df['production_budget'].replace('[\$,]', '', regex=True).astype(float)
tn_movie_budgets_df['domestic_gross'] = tn_movie_budgets_df['domestic_gross'].replace('[\$,]', '', regex=True).astype(float)
tn_movie_budgets_df['worldwide_gross'] = tn_movie_budgets_df['worldwide_gross'].replace('[\$,]', '', regex=True).astype(float)

# Display the first few rows to verify changes
print("Cleaned bom.movie_gross.csv:")
bom_movie_gross_df.head()
# Display the first few rows to verify changes
print("Cleaned tn.movie_budgets.csv:")
tn_movie_budgets_df.head()

#Step 3: Merge the DataFrames We'll merge the two DataFrames on the movie title.

# Merge the DataFrames on the movie title
merged_df = pd.merge(bom_movie_gross_df, tn_movie_budgets_df, left_on='title', right_on='movie', how='inner')

# Display the first few rows of the merged DataFrame to verify
merged_df.head()
#Step 4: Analyze the Data

#We will calculate key metrics such as profitability and ROI, and then later create visualizations to highlight key insights.
# Calculate additional metrics: Profitability and return on investment (ROI)
merged_df['profitability'] = merged_df['worldwide_gross'] - merged_df['production_budget']
merged_df['roi'] = (merged_df['profitability'] / merged_df['production_budget']) * 100

# Display the first few rows to verify the new columns
merged_df[['title', 'production_budget', 'worldwide_gross', 'profitability', 'roi']].head()
#Visualization
# Import visualization libraries
import seaborn as sns
import matplotlib.pyplot as plt

# Plot profitability of top 10 movies
top_10_profitable = merged_df.sort_values(by='profitability', ascending=False).head(10)
plt.figure(figsize=(10, 6))
sns.barplot(data=top_10_profitable, x='profitability', y='title')
plt.title('Top 10 Most Profitable Movies')
plt.xlabel('Profitability')
plt.ylabel('Movie Title')
plt.show()
# Plot ROI of top 10 movies
top_10_roi = merged_df.sort_values(by='roi', ascending=False).head(10)
plt.figure(figsize=(10, 6))
sns.barplot(data=top_10_roi, x='roi', y='title')
plt.title('Top 10 Movies by ROI')
plt.xlabel('ROI (%)')
plt.ylabel('Movie Title')
plt.show()
# Plot Production Budget vs. Worldwide Gross
plt.figure(figsize=(10, 6))
sns.lineplot(data=merged_df, x='production_budget', y='worldwide_gross')
plt.title('Production Budget vs. Worldwide Gross')
plt.xlabel('Production Budget')
plt.ylabel('Worldwide Gross')
plt.show()
# Plot Domestic vs. Foreign Gross
plt.figure(figsize=(10, 6))
sns.lineplot(data=merged_df, x='domestic_gross_x', y='foreign_gross')
plt.title('Domestic Gross vs. Foreign Gross')
plt.xlabel('Domestic Gross')
plt.ylabel('Foreign Gross')
plt.show()
# Bar Graph: Total Domestic vs. Foreign Gross for Top Movies
# Calculate total domestic and foreign gross for top 10 movies by worldwide gross
top_10_movies = merged_df.sort_values(by='worldwide_gross', ascending=False).head(10)
plt.figure(figsize=(12, 6))
top_10_movies.set_index('title')[['domestic_gross_x', 'foreign_gross']].plot(kind='bar', stacked=True)
plt.title('Total Domestic vs. Foreign Gross for Top 10 Movies')
plt.xlabel('Movie Title')
plt.ylabel('Gross Revenue ($)')
plt.legend(['Domestic Gross', 'Foreign Gross'])
plt.show()
# Convert 'release_date' to datetime and extract the year
merged_df['release_date'] = pd.to_datetime(merged_df['release_date'])
merged_df['year'] = merged_df['release_date'].dt.year

# Group by year and calculate the mean production budget and worldwide gross
yearly_data = merged_df.groupby('year').agg({'production_budget': 'mean', 'worldwide_gross': 'mean'}).reset_index()

# # Convert 'release_date' to datetime and extract the year
merged_df['release_date'] = pd.to_datetime(merged_df['release_date'])
merged_df['year'] = merged_df['release_date'].dt.year

# Group by year and calculate the mean production budget and worldwide gross
yearly_data = merged_df.groupby('year').agg({'production_budget': 'mean', 'worldwide_gross': 'mean'}).reset_index()

# Line Graph: Production Budget vs. Worldwide Gross over the Years
plt.figure(figsize=(12, 6))
sns.lineplot(data=yearly_data, x='year', y='production_budget', label='Production Budget')
sns.lineplot(data=yearly_data, x='year', y='worldwide_gross', label='Worldwide Gross')
plt.title('Production Budget vs. Worldwide Gross over the Years')
plt.xlabel('Year')
plt.ylabel('Amount ($)')
plt.legend()
plt.show()
plt.figure(figsize=(12, 6))
sns.lineplot(data=yearly_data, x='year', y='production_budget', label='Production Budget')
sns.lineplot(data=yearly_data, x='year', y='worldwide_gross', label='Worldwide Gross')
plt.title('Production Budget vs. Worldwide Gross over the Years')
plt.xlabel('Year')
plt.ylabel('Amount ($)')
plt.legend()
plt.show()
#Recommendations
Based on the analysis, generate actionable recommendations for Microsoft's new movie studio. Identify high profitability, high ROI, and other relevant factors.

Recommendations 
Focus on Sequels and Franchises: High-grossing franchises like "Toy Story" and "Avengers" tend to have high profitability and ROI.
Invest in Animation: Animated movies such as "Toy Story 3" show significant profitability.
Prioritize Big-Budget Films: Movies with substantial production budgets often yield high returns.
