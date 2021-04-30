# Gender-Prediction-using-Sound

![Forks](https://img.shields.io/github/forks/shukkkur/CodeForces.svg)
![Stars](https://img.shields.io/github/stars/shukkkur/CodeForces.svg)
![Watchers](https://img.shields.io/github/watchers/shukkkur/CodeForces.svg)
![Last Commit](https://img.shields.io/github/last-commit/shukkkur/CodeForces.svg) 

<p>The same name can be spelled out in a many ways, for example, Marc and Mark. Sound can, therefore, be a better way to match names than spelling.In this project, I will use the <b>Python package Fuzzy</b> to find out the genders of authors that have appeared in the New York Times Best Seller list for Children's Picture books. </p>
<h3>1. Sound it out!</h3>

<p>Grey and Gray. Colour and Color. Words like these have been the cause of many heated arguments between Brits and Americans.<br>One way to tackle this challenge is to write a program that checks if two strings sound the same, instead of checking for equivalence in spellings. We'll do that here using fuzzy name matching.</p>

```python
print(fuzzy.nysiis('gray'))
>>> GRY

fuzzy.nysiis('colour') == fuzzy.nysiis('color')
>>> True
```

<h3>2. Authoring the authors</h3>

<p>Let's begin by reading in the data on the best selling authors from 2008 to 2017.</p>

```python
author_df = pd.read_csv('datasets/nytkids_yearly.csv', delimiter=';')

first_name = []
for name in author_df['Author']:
    first_name.append(name.split()[0])
author_df['first_name'] = first_name

author_df.head()
```

|   | Year |                       Book Title |                Author | Besteller this year | first_name |
|--:|-----:|---------------------------------:|----------------------:|--------------------:|-----------:|
| 0 | 2017 | DRAGONS LOVE TACOS               | Adam Rubin            | 49                  | Adam       |
| 1 | 2017 | THE WONDERFUL THINGS YOU WILL BE | Emily Winfield Martin | 48                  | Emily      |
| 2 | 2017 | THE DAY THE CRAYONS QUIT         | Drew Daywalt          | 44                  | Drew       |
| 3 | 2017 | ROSIE REVERE, ENGINEER           | Andrea Beaty          | 38                  | Andrea     |
| 4 | 2017 | ADA TWIST, SCIENTIST             | Andrea Beaty          | 28                  | Andrea     | 


<h3>3. Time to bring on the phonics!</h3>
<p>When we were young children, we were taught to read using phonics; sounding out the letters that compose words. So let's relive history and do that again, but using python this time.</p>

```python
nysiis_name = []
for name in author_df['first_name']:
    nysiis_name.append(fuzzy.nysiis(name))

author_df['nysiis_name'] = nysiis_name

author_df.head()
```

|   | Year |                       Book Title |                Author | Besteller this year | first_name | nysiis_name |
|--:|-----:|---------------------------------:|----------------------:|--------------------:|-----------:|------------:|
| 0 | 2017 | DRAGONS LOVE TACOS               | Adam Rubin            | 49                  | Adam       | ADAN        |
| 1 | 2017 | THE WONDERFUL THINGS YOU WILL BE | Emily Winfield Martin | 48                  | Emily      | ENALY       |
| 2 | 2017 | THE DAY THE CRAYONS QUIT         | Drew Daywalt          | 44                  | Drew       | DR          |
| 3 | 2017 | ROSIE REVERE, ENGINEER           | Andrea Beaty          | 38                  | Andrea     | ANDR        |
| 4 | 2017 | ADA TWIST, SCIENTIST             | Andrea Beaty          | 28                  | Andrea     | ANDR        |


<h3>4. The inbetweeners</h3>
<p>We'll use babynames_nysiis.csv, a dataset that is derived from the Social Security Administration’s baby name data, to identify author genders. The dataset contains unique NYSIIS versions of baby names, and also includes the percentage of times the name appeared as a female name (perc_female) and the percentage of times it appeared as a male name (perc_male). </p>

```
babies_df = pd.read_csv('datasets/babynames_nysiis.csv', delimiter=';')

gender = []
for i in range(len(babies_df)):
    if babies_df.iloc[i]['perc_male'] > babies_df.iloc[i]['perc_female']:
        gender.append('M')
    elif babies_df.iloc[i]['perc_male'] < babies_df.iloc[i]['perc_female']:
        gender.append('F')
    else:
        gender.append('N')
        
babies_df['gender'] = gender

babies_df.head()
```

|   | babynysiis | perc_female | perc_male | gender |
|--:|-----------:|------------:|----------:|-------:|
| 0 | NaN        | 62.50       | 37.50     | F      |
| 1 | RAX        | 63.64       | 36.36     | F      |
| 2 | ESAR       | 44.44       | 55.56     | M      |
| 3 | DJANG      | 0.00        | 100.00    | M      |
| 4 | PARCAL     | 25.00       | 75.00     | M      | 

<h3>5. Playing matchmaker</h3>

<p>Now that we have identified the likely genders of different names, let's find author genders by searching for each author's name in the babies_df DataFrame, and extracting the associated gender. </p>

```python
def locate_in_list(a_list, element):
    loc_of_name = a_list.index(element) if element in a_list else -1
    return(loc_of_name)

author_gender = []
for name in author_df['nysiis_name']:
    nloc = locate_in_list(list(babies_df['babynysiis']), name)
    if nloc == -1:
        author_gender.append('Unknown')
    else:
        author_gender.append(babies_df['gender'][nloc])

author_df['author_gender'] = author_gender

author_df['author_gender'].value_counts()
```

|                                 F | 395 |
|----------------------------------:|----:|
|                 M                 | 191 |
|              Unknown              | 9   |
|                 N                 | 8   |
| Name: author_gender, dtype: int64 |     |

<h3>6. Tally up</h3>

<p>From the results above see that there are more female authors on the New York Times best seller's list than male authors. Our dataset spans 2008 to 2017. Let's find out if there have been changes over time.</p>

```
years = sorted(author_df.Year.unique())

males_by_yr = []
females_by_yr = []
unknown_by_yr = []

for year in years:
    males_by_yr.append(len(author_df[(author_df['author_gender']=='M') & (author_df['Year']==year)]))
    females_by_yr.append(len(author_df[(author_df['author_gender']=='F') & (author_df['Year']==year)]))
    unknown_by_yr.append(len(author_df[(author_df['author_gender']=='Unknown') & (author_df['Year']==year)]))

males_by_yr 

>>> [8, 19, 27, 21, 21, 11, 21, 18, 25, 20]
```

<h3>7. Foreign-born authors? </h3>
<p>Our gender data comes from social security applications of individuals born in the US. Hence, one possible explanation for why there are "unknown" genders associated with some author names is because these authors were foreign-born. </p>

```
years_shifted = list(np.array(years) + 0.25)

plt.bar(years, males_by_yr, width=0.25, color='lightblue')
plt.bar(years_shifted, females_by_yr, width=0.25, color='pink')

plt.xlabel('years')
plt.show()
```
<p align='center'>
  <img src='https://github.com/shukkkur/Gender-Prediction-using-Sound/blob/c7d80a0e888b0458d532dc686927f6afd7b2690d/datasets/img1.jpg/'>
</p>
