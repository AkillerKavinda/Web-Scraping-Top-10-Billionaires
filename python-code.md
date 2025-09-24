# Downloading the contents


```python
import requests
```


```python
url = "https://en.wikipedia.org/wiki/The_World%27s_Billionaires"

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36"
}

page = requests.get(url, headers=headers)

print(page.status_code,end = '\n\n')
print(page.text[:500])   
```


```python
page.content
```


```python
page.text
```


```python
page.encoding
```


```python
page.headers
```

# Extracting the data


```python
from bs4 import BeautifulSoup
```


```python
soup = BeautifulSoup(page.content, 'html.parser')
```


```python
soup.title
```


```python
soup.p
```


```python
soup.find_all('p')
```


```python
for para in soup.find_all('p'):
    print(para.text)
```


```python
for link in soup.find_all('a'):
    print(link.get('href'))
```


```python
for content in soup.find_all('div', class_='vector-toc-text' ):
    print(content.text)
```

# Filtering and storing the data


```python
# Getting data in all the tables

for table in soup.find_all('table', class_='wikitable sortable'):
    for row in table.find_all('tr'):
        cols = row.find_all(['th','td'])  # get both headers and data cells
        cols_text = [col.get_text(strip=True) for col in cols]  # clean text
        print(cols_text)
```


```python
# Getting the first table

tables = soup.find_all('table', class_='wikitable sortable')
print(f"Found {len(tables)} tables")

if tables:
    for row in tables[0].find_all('tr'):
        cols = row.find_all(['th','td'])
        cols_text = [col.get_text(strip=True) for col in cols]
        print(cols_text)

```


```python
# Converting the first table into a dataframe

tables = soup.find_all('table', class_='wikitable sortable')
print(f"Found {len(tables)} tables", end='\n\n')

if tables:
    data = []
    for row in tables[0].find_all('tr'):
        cols = row.find_all(['th','td'])
        cols_text = [col.get_text(strip=True) for col in cols]
        if cols_text:  # skip empty rows
            data.append(cols_text)

    df = pd.DataFrame(data[1:], columns=data[0])  # Setting the first row as a header

df
```

# Saving the data to a csv


```python
df.to_csv('billionaires.csv')
```

# Visualizations


```python
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
df.dtypes
```

# Billionaires by Age


```python
df['Age'] = df['Age'].astype(int)
```


```python
df.dtypes
```


```python
df_age_sort = df.sort_values(by='Age')

sns.barplot(data = df_age_sort, x = 'Name', y = 'Age', hue = 'Name',  palette = 'viridis')
plt.title("Billionaires by Age")
plt.xticks(rotation = 75)
plt.show()
```

# Billionaires by net worth


```python
df['Net worth_numeric'] = (
    df['Net worth(USD)']
      .str.replace('$','', regex=False)           # remove $
      .str.replace('\xa0billion','', regex=False) # remove non-breaking space + 'billion'
      .astype(float)             # convert to number
)

```


```python
sns.barplot(data = df, x = 'Name', y = 'Net worth_numeric', hue = 'Name',  palette = 'viridis')
plt.title("Billionaires by net worth")
plt.ylabel("Billions")
plt.xticks(rotation = 75)
plt.show()
```


```python
# Billionaires by country

df.columns
```


```python
df_nationality = df.groupby("Nationality")['Name'].count().reset_index()
df_nationality = df_nationality.rename(columns={'Name': 'Count'}) 
df_nationality = df_nationality.sort_values(by='Count', ascending=False)

sns.barplot(data=df_nationality, x='Nationality', y='Count', hue = 'Nationality', palette='viridis')
plt.title("Billionaires by country")
plt.xticks(rotation=75)
plt.show()

```


```python
# What reset index does
```


```python
 df.groupby("Nationality")['Name'].count()
```


```python
 df.groupby("Nationality")['Name'].count().reset_index()
```
