---
tags:
  - pandas
  - aggregate
  - merge
  - data
---
## Learn to aggregate and merge data in pandas while analysing a dataset of LEGO pieces
[source](https://rebrickable.com/downloads/)
[Jupyter notebook](http://localhost:8888/notebooks/100_days_of_code/day_74_aggregate_and_merge_data/LEGO%20Notebook%20and%20Data%20(start)/Lego_Analysis_for_Course_(start).ipynb)

### We will answer:
1. What is the most enormous LEGO set ever created and how many parts did it have?
2. In which year were the first LEGO sets released and how many sets did the company sell when it first launched?
3. Which LEGO theme has the most sets? Is it harry Potter, Ninjago, Friends or something else?
4. When did the LEGO company really take off based on its product offering? How many themes and sets did it release every year?
5. Did LEGO sets grow in size and complexity over time? Do older LEGO sets tend to have more or fewer parts than newer sets?

### What you'll learn today:
1. How to combine a Notebook with HTML markup
2. Apply Python list slicing techniques to Pandas DataFrames
3. How to aggregate data using the `.agg()` method
4. How to create scatter plots, bar charts, and line charts with two axes in Matplotlib.
5. Understand database schemas that are organised by primary and foreign keys.
6. How to merge DataFrames that share a common key

## Project

### Insert a Markdown cell



### Import Statements

`import pandas as pd`
### Finding unique colors

* Use the [`nunique()` method](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.nunique.html?highlight=nunique#pandas.DataFrame.nunique)*
if the `axis` parameter is 0 or 'rows', it will return a Series, where the columns become the rows and the values are the number of unique elements in that column.

```python
colors_df.nunique(axis='rows')
# Output:
	id          135
	name        135
	rgb         124
	is_trans      2
	dtype: int64

colors_df.name.nunique()
# Output:
	135
```

### Find the number of transparent colours where `is_trans == 't'` versus the number of opaque colours where `is_trans == 'f'`. See if you can accomplish this in two different ways.

#### Method 1
`(colors_df.is_trans == 't').sum() + (colors_df.is_trans == 'f').sum()`

#### Method 2
`len(colors_df.query("is_trans == 't'"))`

### Method 3
`colors.groupby('is_trans').count()`

#### Method 4:
`colors_df.is_trans.value_counts()`

### When were the first LEGO sets released?

#### sort_values(by='year')
`sets_df.sort_values('year')`
* defaults to ascending order

### How many different products was the company selling during their first year?

#### Filtering a dataframe by condition

`sets_df[sets_df['year'] == 1949]`

### Which LEGO set has the most parts?

`sets_df.sort_values(by='num_parts', ascending=False).head()`

### Analyzing sets released on a yearly basis

`import matplotlib.pyplot as plt`

#### Create a new series `sets_by_year`

* has years as the index
* has number of sets as values

## Using the Pandas `.agg()` method

Often you will need to summarise data. This is where the `.groupby()` function comes in really handy. however, sometimes you want to run even more operations based on a particular DataFrame column. This is where the `.agg()` method comes in.

### Calculate the Number of Different Themes by Calendar Year

* Group the data by year
* count unique theme_ids for that year

##### Number of themes per calendar year

* chain `groupby()` and `.agg()` methods.
`.agg()` takes a dictionary as an argument. In this dictionary, we specify which operation we'd like to apply to each column. In our case, we just want to calculate the number of unique entries in the theme_id column by using our old friend, the `.nunique()` method.
`df.groupby('col1').agg({'col2': operation})`
every row will be a year. every value will be `operation` applied to the column. So in our new df, each year row has one value, which is the number of unique theme_ids for that year.
year => the theme_ids that have that  year => how many of those theme_ids are unique? **unique themes / year**
`sets_df.groupby('year').agg({'theme_id': pd.Series.nunique})`

##### Renaming a column:

`df.rename(columns={'orig_col_name': 'new_col_name'}, inplace=True)`

## Super Imposing Line Charts

How to we plot the number of themes and the number of sets on the same chart, while accounting for the vast difference in scale? The number of sets will tend to dwarf the number of themes, since any given theme will tend to have multiple sets.

### Let's use two separate axes

In order to configure and plot data along separate axes, we must configure the chart
```
ax1 = plt.gca()  # Get current ax
ax2 = ax1.twinx()
```

Create another axis object: `ax2`. The `twinx()` method allows `ax1` and `ax2` to share the same x-axis. 
to apply the new axes, execute the `.plot() `method on them.
```python
ax1.plot(x1, y1)
ax2.plot(x2, y2)
###
ax1 = plt.gca()
ax2 = ax1.twinx()
ax1.plot(uniq_themes_by_year.index[:-2], uniq_themes_by_year.nr_themes.rolling(3).mean()[:-2])
ax2.plot(sets_by_year.index[:-2], sets_by_year.set_num[:-2])
```

#### Let's add some styling.

##### color in the lines

`ax1.plot(x1, y1, color='color')`
##### color in the axes and add some labels

`ax1.set_xlabel('label', color='color')`
`ax1.set_ylabel('label', color='color')`

```python
ax1 = plt.gca()
ax2 = ax1.twinx()
ax1.plot(uniq_themes_by_year.index[:-2], uniq_themes_by_year.nr_themes.rolling(3).mean()[:-2], color='g')
ax2.plot(sets_by_year.index[:-2], sets_by_year.set_num[:-2], color='b')
ax1.set_xlabel('Year')
ax1.set_ylabel('Number of themes', color='g')
ax2.set_ylabel('Number of sets', color='b')
```

## Scatter plots: average parts per set

### Parts per set

* A scatter plot simply uses dots to represent each data point

Challenge: Create a Pandas Series called `parts_per_set` that has the year as the index and contains the average number of parts per LEGO set in that year. 
`num_parts`

```python
parts_per_set = sets_df.groupby('year').agg({'num_parts': pd.Series.mean}).rename(columns={'num_parts': 'average num_parts'})
parts_per_set.head()
```

`plt.scatter(parts_per_set.index[:-2], parts_per_set['average num_parts'][:-2])`

### Sets per theme

```python
# sets_df['theme_id'][sets_df.groupby('theme_id').set_num.count().idxmax()]
# sets_df.groupby('theme_id').set_num.count().max()
set_theme_count = sets_df['theme_id'].value_counts()
set_theme_count.head()
```

This will tell us which *theme_id* has the greatest number of sets, but in order ot find out what the theme is actually called, we have to compare that with the data in `themes.csv`

## How to Search a DataFrame

`.where()`: `series.where(series == 'value').dropna()` returns a new series where False returns NaN
`.query()`: `dataframe.query('column == "value"')` returns a df.
`sw_themes = themes.query('name == "Star Wars"')`
`dataframe[dataframe.column == 'values']`
`sets_df[(sets_df.theme_id == 18)]`

### Combine data on theme names with number of sets per theme

use `.merge()` to combine 2 dataframes into one. the merge method works on columns with the same **name** in both dataframes.

```
merged_df = pd.merge(themes, set_theme_count, on='id')
merged_df = pd.merge(set_theme_count, themes, on='id')
merged_df
```

## Plotting the top 10 themes on a chart

### Create a bar chart

```python
plt.figure(figsize=(15, 8))
plt.xticks(fontsize=14, rotation = 45)
plt.yticks(fontsize=14)
plt.ylabel('Nr of Sets', fontsize=14)
plt.xlabel('Theme Name', fontsize=14)
plt.bar(merged_df.name[:10], merged_df.set_count[:10])
```

output:

![[Untitled.png]]

## Summary

* use HTML Markdown in Notebooks, such as section headings and # and how to embed images with the \<img> tag.
* combing the `.groupby()` and `.count()` methods to aggregate data
* use the `.value_counts()` method
* slice DataFrames using the square bracket notation e.g. `df[:-2]` or `df[:10]
* use the `.agg()` method to run an operaton on a particular column
* `.rename()` columns of DataFrames
* create a line chart with two separate axes to visualise the data that have different scales
* create a scatter plot in Matplotlib
* work with tables in a relational database by using primary and foreign keys
* `.merge()` DataFrames along a particular column
* create a bar chart with Matplotlib