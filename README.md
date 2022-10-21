# Brasileirao - Serie A - 2021
In this repository we will use web scraping to get data about Campeonato Brasileiro 2021 - Série A - from CBF site. After that, we will make some analysis using Excel and Power BI

## Getting final table of Brazilian Championship - 2021 - Série A
First of all we need to get data from CBF site using webscraping, for that the code below will be used and will access the site 'https://www.cbf.com.br/futebol-brasileiro/competicoes/campeonato-brasileiro-serie-a'

### Importing libraries 
```
import pandas as pd
import numpy as np
from bs4 import BeautifulSoup as bs
import requests
```

### Code used to get the headers of the table 
```
url = 'https://www.cbf.com.br/futebol-brasileiro/competicoes/campeonato-brasileiro-serie-a/2021'
#Object page
page = requests.get(url)

# parser-lxml = Change html to Python friendly format
# Obtain page's information
soup = bs(page.text, 'lxml')
soup

# Obtain information from tag <table>
table1 = soup.find('table',class_="table m-b-20 tabela-expandir")

# Obtain every title of columns with tag <th>

# Por conta da existencia de um <th> na <body>, o programa puxa a primeira coluna referente a pontos, portanto, precisamos usar um for para pegar apenas os 12 primeiros termos da <head>, que seria o cabeçalho. 
headers = []
cont = 0
for i in table1.find_all('th'):
 title = i.text
 cont = cont +1
 if cont<=12:
     headers.append(title)
#Adição do nome do time na segunda coluna
headers.insert(1, 'Time')

#Create a dataframe
mydata = pd.DataFrame(columns = headers)
mydata.head()
```
After the code above, we will get something like that:

