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
![headers](https://user-images.githubusercontent.com/79097913/197279280-4c032bb4-2859-4253-9626-58918d55460f.png)

### Completing the table with data
To complete the table, the following codes were used:
```
# Create a for loop to fill mydata
#Obtendo posições dos times
posicoes = []
for i in range(1,21):
    posicoes.append(str(i)+"°")
mydata['Posição']=posicoes

#Obtendo times
times = table1.find_all('span',class_="hidden-xs")
dict_times =[]
for i in range(len(times)):
    dict_times.append(times[i].text.split(" - ")[0].strip())
mydata['Time'] = dict_times

#Obtendo pontos
pontos = []
for i in table1.find_all('th')[14:]:
 pontos.append(i.text)
mydata['PTS'] = pontos

#Obtendo valores restantes
matrix = []
for j in table1.find_all('tr',class_="expand-trigger"):
 row_data = j.find_all('td')
 row = [i.text for i in row_data][1:11]
 matrix.append(row)

jogo = []
vitoria = []
empate = []
derrota = []
gp = []
gc = []
sg = []
ca = []
cv = []
porcent = []

for linha in range(len(matrix)):
        jogo.append(matrix[linha][0])
        vitoria.append(matrix[linha][1])
        empate.append(matrix[linha][2])
        derrota.append(matrix[linha][3])
        gp.append(matrix[linha][4])
        gc.append(matrix[linha][5])
        sg.append(matrix[linha][6])
        ca.append(matrix[linha][7])
        cv.append(matrix[linha][8])
        porcent.append(matrix[linha][9])
mydata['J'] = jogo
mydata['V'] = vitoria
mydata['E'] = empate
mydata['D'] = derrota
mydata['GP'] = gp
mydata['GC'] = gc
mydata['SG'] = sg
mydata['CA'] = ca
mydata['CV'] = cv
mydata['%'] = porcent

print(mydata)
```
### Transforming in .csv file
```
#Transformando em .csv
mydata.to_csv('brasileirao2021.csv', index=False)
```
To check if everything is fine, just use the following code:

```
# Try to read csv
mydata2 = pd.read_csv('brasileirao2021.csv')
mydata2
```
The final table will be given by:

![tabelafinal_2021](https://user-images.githubusercontent.com/79097913/197279292-63be80ea-f6a2-4e16-95a8-598239ccac82.png)

## Getting all data related with the matches during the seasson 
In this part, we will use the 'Brasileirao_jogos.ipynb'

### Importing libraries
```
import requests
from bs4 import BeautifulSoup
import pandas as pd
import matplotlib.pyplot as plt
import datetime 
from datetime import date 
from unicodedata import normalize as nm
from dateutil import parser
```
Creating a tool to help with game dates:
```
##Padronizar mês da data do Jogo.
def retMes(mes):
    if mes == 'Janeiro':
        return '01'
    elif mes == 'Fevereiro':
        return '02'
    elif mes == 'Março':
        return '03'
    elif mes == 'Abril':
        return '04'
    elif mes == 'Maio':
        return '05'
    elif mes == 'Junho':
        return '06'
    elif mes == 'Julho':
        return '07'
    elif mes == 'Agosto':
        return '08'
    elif mes == 'Setembro':
        return '09'
    elif mes == 'Outubro':
        return '10'
    elif mes == 'Novembro':
        return '11'
    elif mes == 'Dezembro':
        return '12'
```
Differently than the first table we get, we will need to acces the url of all the 380 matches to get the data, for that we will use the url `https://www.cbf.com.br/futebol-brasileiro/competicoes/campeonato-brasileiro-serie-a` and at the final it's necessary to add `+str(yearofmatch)+'/'+str(matchnumber)` .The code used to get the data can be seen below:

```
url = 'https://www.cbf.com.br/futebol-brasileiro/competicoes/campeonato-brasileiro-serie-a'
#No Total, existem 380 partidas e 20 times disputando, logo para cada rodada do campeonato ocorrem 10 partidas.
#Armazenando jogos
lista_jogos =[]

for ano in range(2019,2021):
    for numero_partida in range(1,381):
        url_jogo = url+'/'+str(ano)+'/'+str(numero_partida)
        x = requests.get(url_jogo)
        soup = BeautifulSoup(x.content,'html.parser')
        
        #Numero do Jogo + rodada + turno
        str_jogo = soup.find(class_='color-white block text-1')
        num_jogo = int(str_jogo.get_text().strip().replace('Jogo: ',''))
        
        if (num_jogo%10 ==0):
            rodada = int(int(num_jogo)/10)
        else:
            rodada = int((int(num_jogo)//10)+1)
        if num_jogo <=190:
            turno = 1
        else:
            turno = 2
        
        
        #Localização do jogo
        local_aux = soup.find_all(class_='text-2 p-r-20')
        estadio = local_aux[0].get_text().split(" - ")[0].strip()
        cidade = local_aux[0].get_text().split(" - ")[1].strip()
        estado = local_aux[0].get_text().split(" - ")[2].strip()
        
        #Data do jogo
        data_soup = soup.find_all(class_='text-2 p-r-20')
        data = data_soup[1].get_text().split(",")[1].strip()
        #Formatando data
        aux_data = data.split(" de ")
        dia = aux_data[0]
        #formatando mes
        mes = aux_data[1]
        mes = retMes(mes)
        ano = aux_data[2]
        
        data_completa = datetime.date(int(ano),int(mes),int(dia))
        data_completa = data_completa.strftime("%d-%m-%Y")
        
        #Pegando nomes dos mandantes e visitantes
        partida_times = soup.find_all(class_='time-nome color-white')
        mandante = partida_times[0].get_text().split(" - ")[0].strip()
        visitante = partida_times[1].get_text().split(" - ")[0].strip()
        
        #Pegando resultados da partida
        resultado_partida = soup.find_all(class_='time-gols block hidden-sm hidden-md hidden-lg')
        mandante_gols = int(resultado_partida[0].get_text())
        visitante_gols = int(resultado_partida[1].get_text())
        gols_partida = mandante_gols + visitante_gols
        placar = str(str(mandante_gols) + '-'+ str(visitante_gols))
        
        if mandante_gols == visitante_gols:
            resultado = 'Empate'
            resultado_mandante = 'Empate'
            resultado_visitante = 'Empate'
        elif mandante_gols > visitante_gols:
            resultado = 'Mandante'
            resultado_mandante = 'Vitoria'
            resultado_visitante = 'Derrota'
        
        else:
            resultado = 'Visitante'
            resultado_mandante = 'Derrota'
            resultado_visitante = 'Vitoria'
        lista_resultado = [resultado, resultado_mandante, resultado_visitante]
        
        placar = str(mandante_gols) + ' x ' + str(visitante_gols)
        
        lista_dados_partida = [str(ano),rodada, num_jogo, turno, mandante, visitante,
                               estadio, cidade, estado, data_completa, mandante_gols, visitante_gols, gols_partida, 
                               resultado, resultado_mandante, resultado_visitante, placar, url_jogo]
        lista_jogos.append(lista_dados_partida)

print(lista_jogos)
```
### Creating a DataFrame
```
df = pd.DataFrame(lista_jogos, columns=['Temporada','Rodada',
                                   'Número do Jogo','Turno','Mandante',
                                   'Visitante','Estadio','Cidade','UF','Data',
                                   'Gols_Mandante','Gols_Visitante',
                                   'Total_Gols_partida','Resultado partida',
                                   'Rsultado Mandante','Resultado Visitante',
                                   'Placar','URL'])
```
### Exporting to excel
```
df.to_excel("CampeonatoBrasileiro_Jogos_CBF_SerieA.xlsx")

```
### Analysis using Power BI
After export to excel, we can do our analysis in Excel properly or in Power BI. In this case I preferred to use Power Bi, because it's easier to create dashboards and prettier. The final result obtained can be seen below:

![PowerBI_Palmeiras](https://user-images.githubusercontent.com/79097913/197280750-11ddadca-87e3-40c1-a6b7-4d61eddc0d23.png)
