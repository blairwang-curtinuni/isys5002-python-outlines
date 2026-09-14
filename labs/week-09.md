# ISYS5002 - Week 9

Back in [Week 6](week-06.md) we used a **list** to hold many temperature readings, and wrote **loops** to work through them one at a time—summing, counting, filtering by hand. That was the right way to learn what is really going on. But it does not scale. Real datasets have thousands of rows and dozens of columns, they arrive in files rather than typed in by hand, and nobody wants to write an accumulator loop every time they need an average.

This week we meet **pandas**, the standard Python library for working with tabular data. Everything you did by hand in Week 6—reading data in, selecting the rows you care about, summarising, counting—pandas does in a single line. We'll also draw our first charts.

## Key Concepts

- **DataFrame:** in practice kind of like a data table with rows and columns, a bit like a spreadsheet or a list of the dictionaries from [Week 7](week-07.md).
- **Reading a CSV:** loading a comma-separated values file straight into a DataFrame.
- **Selecting rows and columns:** pulling out just the data you care about, including **boolean filtering** (the pandas version of the `if` conditions from [Week 3](week-03.md)).
- **Aggregating and summarising:** getting totals, averages, counts, and per-group summaries with `groupby()`.
- **Visualisation:** turning a table into a bar chart or line chart.

### Installs

pandas (and matplotlib, which draws charts for pandas) are probably already installed in Colab. If you're working in Codespaces or on your own laptop, and a `ModuleNotFoundError` appears, you can install them the same way you installed pyinputplus in [Week 4](week-04.md):

**Colab / Jupyter:**

```python
!pip install pandas matplotlib
```

**Codespaces / VS Code terminal:**

```bash
pip install pandas matplotlib
```

### Today's dataset - CSV format!

Throughout this worksheet we'll use a small weather dataset. Create a file called `weather.csv` in the same folder as your script/notebook, and paste in the following:

```csv
date,city,temperature,humidity,conditions
2025-06-01,Perth,17.2,62,Sunny
2025-06-01,Sydney,15.8,71,Cloudy
2025-06-01,Darwin,31.5,68,Sunny
2025-06-02,Perth,16.0,80,Rain
2025-06-02,Sydney,14.9,77,Rain
2025-06-02,Darwin,32.1,70,Sunny
2025-06-03,Perth,19.4,55,Sunny
2025-06-03,Sydney,17.2,60,Sunny
2025-06-03,Darwin,30.8,74,Cloudy
2025-06-04,Perth,14.1,88,Rain
2025-06-04,Sydney,13.5,82,Rain
2025-06-04,Darwin,33.0,66,Sunny
2025-06-05,Perth,18.7,58,Cloudy
2025-06-05,Sydney,16.6,64,Cloudy
2025-06-05,Darwin,31.9,69,Sunny
```

**Tip (Colab):** If you're using Colab, then create this file using Notepad. Make sure you have file extensions turned on in your computer, so you can save the file as `weather.csv` and not `weather.csv.txt`. Then, in Colab, click the folder icon on the left, then the upload button, and upload a `weather.csv` you have saved on your computer.

## 1. Opening a CSV File

The `read_csv()` function in Python "ingests" this file in just one line!

```python
import pandas as pd

df = pd.read_csv("weather.csv")
print(df)
```

Output:

```
          date    city  temperature  humidity conditions
0   2025-06-01   Perth         17.2        62      Sunny
1   2025-06-01  Sydney         15.8        71     Cloudy
2   2025-06-01  Darwin         31.5        68      Sunny
3   2025-06-02   Perth         16.0        80       Rain
...
14  2025-06-05  Darwin         31.9        69      Sunny
```

Observations:

- We `import pandas as pd`. The `as pd` gives the library a short nickname so we don't have to type `pandas.` in front of everything, and yes, `pd` is pretty much universally understood as "pandas". It's one of those conventions, like driving on the left-hand side of the road in Australia.
- The numbers on the side are the **index** generated automatically by pandas. It starts from 0, just like lists.

If you have a big file (most files are big files!): `pandas` provides functions to inspect the overall 'vibe' of the file:

```python
print(df.head())        # the first 5 rows
print(df.shape)         # (number of rows, number of columns)
print(df.columns)       # the column names
```

Output:

```
          date    city  temperature  humidity conditions
0   2025-06-01   Perth         17.2        62      Sunny
1   2025-06-01  Sydney         15.8        71     Cloudy
2   2025-06-01  Darwin         31.5        68      Sunny
3   2025-06-02   Perth         16.0        80       Rain
4   2025-06-02  Sydney         14.9        77       Rain
(15, 5)
Index(['date', 'city', 'temperature', 'humidity', 'conditions'], dtype='object')
```

So our table has 15 rows and 5 columns. `head()` shows the first five rows by default; try `df.head(3)` or `df.tail()` to see how they differ.

> **Note:** The last line (`dtype='object'`) might say `dtype='str'` depending on which version of Python you are using. It doesn't make a huge difference.

## 2. Selectively retrieving data

**Note:** If you're taking the databases class and/or have experience with SQL, you might find section 5 interesting!

Based on a particular column:

```python
print(df["temperature"])
```

Based on particular conditions:

```python
# Only the rows where the city is Perth
perth = df[df["city"] == "Perth"]
print(perth)

# Only the hot readings
hot = df[df["temperature"] > 30]
print(hot)

# Perth readings that were also warm
warm_perth = df[(df["city"] == "Perth") & (df["temperature"] > 18)]
print(warm_perth)
```

**Tip:** If you see the error message `ValueError: The truth value of a Series is ambiguous`, check if you accidentally wrote the actual word `and` rather than the symbol `&`. Also check your brackets.

### Challenge: Query the weather

Using boolean filtering, produce:

- All the readings from Sydney.
- All the readings where it was raining (`conditions` equals `"Rain"`).
- All the readings that were both hot (over 25°C) **and** humid (humidity over 65).
- How many rows are in each of the results above? (Tip: see if you can research something about `shape`, or `len()`...)

## 3. Aggregating and summarising data

```python
# Average temperature for each city
print(df.groupby("city")["temperature"].mean())
```

Output:

```
city
Darwin    31.86
Perth     17.08
Sydney    15.60
Name: temperature, dtype: float64
```

```python
summary = df.groupby("city")["temperature"].agg(["mean", "max", "min", "count"])
print(summary)
```

Output:

```
         mean   max   min  count
city
Darwin  31.86  33.0  30.8      5
Perth   17.08  19.4  14.1      5
Sydney  15.60  17.2  13.5      5
```

In three lines we have turned fifteen raw readings into a clear per-city summary. That is the whole point of pandas.

### Challenge: Summarise the weather

- Find the average **humidity** for each city.
- Find the **highest** temperature recorded for each city, and the **lowest**.
- Count how many readings each city has (they should all be the same here—but on a real, messy dataset they often aren't, which is exactly why you check).
- **Extension:** group by `conditions` instead of `city`. On average, is it warmer on sunny days or rainy days in this dataset?

## 4. Simple data visualisations (charts)

We can use matplotlib to help pandas draw some nice charts:

```python
import matplotlib.pyplot as plt
```

Now we can do something like this:

```python
avg_temp = df.groupby("city")["temperature"].mean()

avg_temp.plot(kind="bar", title="Average temperature by city")
plt.ylabel("Temperature (°C)")
plt.tight_layout()
plt.show()
```

**Note:** `plt.tight_layout()` arranges the labels for you, and `plt.show()` displays the finished chart. In Colab you might be able to get away without the `show()` but it's good to include it so that your intentions are clear.

Here's another example:

```python
perth = df[df["city"] == "Perth"]

perth.plot(x="date", y="temperature", kind="line", marker="o",
           title="Perth temperature over time")
plt.ylabel("Temperature (°C)")
plt.tight_layout()
plt.show()
```

### Challenge: Draw charts of the weather

- Draw a bar chart of average **humidity** per city.
- Draw a line chart of **Darwin's** temperature over the five days, and compare its shape with Perth's. What is different about the two cities?
- **Extension:** investigate how to plot more than one city's temperature line on the **same** axes, so the comparison is direct. (Hint: look up `df.pivot()` or search for "pandas plot multiple lines"—part of this course is learning to find answers the way a working programmer does, just as you discovered pyinputplus in Week 4.)

## 5. (Optional) Selectively retrieving / aggregating and summarising using SQL, via pandasql

If you're familiar with SQL and prefer to use that for sections 2 and 3 (and equivalent tasks for any future work), you can use `pandasql` which does exactly what it sounds like (run SQL in pandas).

**Disclaimer:** because SQL is outside the scope of ISYS5002, this section is 100% OPTIONAL.

But, basically, you can run SQL code directly on a CSV file! No need to set up a DBMS, define data dictionaries, draw ERDs, etc. Of course this means you also do not get all the benefits of a proper DBMS setup. But for ad-hoc analysis (especially as part of a data analysis pipeline), this can be incredibly valuable.

First, do the necessary setup:

```python
# Only if in Jupyter/colab, otherwise just run the pip command in the terminal
!pip install pandasql

from pandasql import sqldf
```

Now if you want to selectively retrieve data (equiv. section 2), just do this:

```python
perth = sqldf("SELECT * FROM df WHERE city = 'Perth'", globals())
print(perth)

warm_perth = sqldf("SELECT * FROM df WHERE city = 'Perth' AND temperature > 18", globals())
print(warm_perth)
```

Group functions are an SQL classic:

```python
summary = sqldf("""
    SELECT city,
           AVG(temperature) AS avg_temp,
           MAX(temperature) AS max_temp,
           MIN(temperature) AS min_temp,
           COUNT(*)         AS n
    FROM df
    GROUP BY city
""", globals())
print(summary)
```

Here's a challenge (**optional** of course): see if you can store the SQL query in a file like `descriptive_stats.sql` and then load that in using Python. This is very helpful in a Visual Studio Code 'script' paradigm, keep various files separate. In a workplace setting, it means you can more easily delegate tasks (e.g., junior & senior data analyst; statistics specialist & IT specialist; etc.)
