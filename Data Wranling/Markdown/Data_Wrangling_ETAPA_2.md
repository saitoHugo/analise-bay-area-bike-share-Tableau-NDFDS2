
## Data Wrangling

Nessa etapa foram realizados os procedimentos explicados abaixos:

1) Foi criada uma função auxiliar para identificar (mapear) quais eram as cidades das estações - create_station_map

2) Foi criada uma função principal (summarise_data)com as seguintes funções:

    - Unir todos os dados em um único arquivo 
    - Mapear as estações finais e iniciais para as cidades (utilizando a função "create_station_map" para auxiliar)
    - Formatar data e dividi-la em ano, mes, hora e dia da semana 
    - Selecionar infomações de interesse para a análise: 'duration', 'start_date', 'start_year', 'start_month', 'start_hour', 'weekday', 'start_city', 'end_city', 'subscription_type'


```python
# Importa todas as bibliotecas necessárias
%matplotlib inline
import csv
from datetime import datetime
import numpy as np
import pandas as pd
from IPython.display import display
```


```python
def create_station_mapping(station_data):
    """
    Cria um mapeamento (tambémm conhecido como de-para) entre a estação 
    e a cidade
    """
    # TODO: Inicie esta variável de maneira correta.
    station_map = {}
    for data_file in station_data:
        with open(data_file, 'r') as f_in:
            # configura o objeto csv reader - note que está sendo usado o DictReader,
            # que usa a primeira linha do arquivo como cabeçalho e cria as chaves
            # do dicionário com estes valores.
            weather_reader = csv.DictReader(f_in)

            for row in weather_reader: #row irá percorrer todas as keys do disct weather_reader
                station_id = row['station_id']
                city = row['landmark']
                station_map[station_id] = city
    return station_map
```


```python
def summarise_data(trip_in, station_data, trip_out):
    """
    Esta função recebe informações de viagem e estação e produz um novo
    arquivo de dados com um resumo condensado das principais informações de viagem.Os 
    argumentos trip_in e station_data serão listas de arquivos de dados para
    as informações da viagem e da estação enquanto trip_out especifica o local
    para o qual os dados sumarizados serão escritos.
    """
    # gera o dicionário de mapeamento entre estações e cidades
    station_map = create_station_mapping(station_data)
    
    with open(trip_out, 'w') as f_out:
        # configura o objeto de escrita de csv       
        out_colnames = ['duration', 'start_date', 'start_year',
                        'start_month', 'start_hour', 'weekday',
                        'start_city', 'end_city', 'subscription_type']        
        trip_writer = csv.DictWriter(f_out, fieldnames = out_colnames)
        trip_writer.writeheader()
        
        for data_file in trip_in:
            with open(data_file, 'r') as f_in:
                # configura o leitor do csv
                trip_reader = csv.DictReader(f_in)

                # processa cada linha lendo uma a uma
                for row in trip_reader:
                    new_point = {}
                    
                    # converte a duração de segundos para minutos.
                    ### TODO: Pergunta 3a: Adicione uma operação matemática       ###
                    ### para converter a duração de segundos para minutos.  ###
                    new_point['duration'] = float(row['Duration'])/float(60)
                    
                    # reformate strings com datas para múltiplas colunas
                    ### TODO: Pergunta 3b: Preencha os __ abaixo para criar os        ###
                    ### campos experados nas colunas (olhe pelo nome da coluna) ###
                    trip_date = datetime.strptime(row['Start Date'], '%m/%d/%Y %H:%M')
                    new_point['start_date']  = trip_date.strftime('%d/%m/%Y')
                    #print '1) ', new_point['start_date'] 
                    new_point['start_year']  = trip_date.year
                    #print '2) ', new_point['start_year']
                    new_point['start_month'] = trip_date.month
                    #print '3) ', new_point['start_month']
                    new_point['start_hour']  = trip_date.strftime('%H')
                    #print '4) ', new_point['start_hour']
                    new_point['weekday']     = trip_date.weekday()
                    #print '5) ', new_point['weekday']

                    # TODO: mapeia o terminal de inicio e fim com o a cidade de inicio e fim
                    new_point['start_city'] =  station_map[row['Start Terminal']]
                    #print '6) ', new_point['start_city']
                    new_point['end_city'] =  station_map[row['End Terminal']]
                    #print '7) ', new_point['end_city']
                    # TODO: existem dois nomes diferentes para o mesmo campo. Trate cada um deles.
                    #o arquivo csv 201402_trip_data tem 'Subscription_type' e o 201408 tem 'Subscriber type' 
                    if 'Subscription Type' in row:
                        new_point['subscription_type'] = row['Subscription Type']
                    else:
                        new_point['subscription_type'] = row['Subscriber Type']

                    # escreve a informação processada para o arquivo de saída.
                    trip_writer.writerow(new_point)
```

### Realizando o processamento dos dados


```python
#Definindo os dados com informacao sobre as estacoes
station_data = ['201402_station_data.csv',
                '201408_station_data.csv',
                '201508_station_data.csv' ]
#Definindo 
trip_in = ['201402_trip_data.csv',
           '201408_trip_data.csv',
           '201508_trip_data.csv' ]
trip_out = 'trip_data.csv'

# Esta função irá ler as informações das estações e das viagens
# e escreverá um arquivo processado com o nome trip_out
summarise_data(trip_in, station_data, trip_out)
```

### Visualizando o arquivo 'summary_Bay_Area_Bike_Share.csv'


```python
trip_data = pd.read_csv('trip_data.csv')

display(trip_data.head(20))

display(trip_data.tail(20))

```


<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>duration</th>
      <th>start_date</th>
      <th>start_year</th>
      <th>start_month</th>
      <th>start_hour</th>
      <th>weekday</th>
      <th>start_city</th>
      <th>end_city</th>
      <th>subscription_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.050000</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>14</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.166667</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>14</td>
      <td>3</td>
      <td>San Jose</td>
      <td>San Jose</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.183333</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>10</td>
      <td>3</td>
      <td>Mountain View</td>
      <td>Mountain View</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.283333</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>11</td>
      <td>3</td>
      <td>San Jose</td>
      <td>San Jose</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.383333</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>12</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1.716667</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>18</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.816667</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>13</td>
      <td>3</td>
      <td>San Jose</td>
      <td>San Jose</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1.850000</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>14</td>
      <td>3</td>
      <td>San Jose</td>
      <td>San Jose</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1.883333</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>17</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1.900000</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>11</td>
      <td>3</td>
      <td>San Jose</td>
      <td>San Jose</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2.083333</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>13</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2.100000</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>13</td>
      <td>3</td>
      <td>San Jose</td>
      <td>San Jose</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2.150000</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>19</td>
      <td>3</td>
      <td>Mountain View</td>
      <td>Mountain View</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2.166667</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>13</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2.233333</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>12</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2.300000</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>16</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2.350000</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>11</td>
      <td>3</td>
      <td>San Jose</td>
      <td>San Jose</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2.366667</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>12</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2.366667</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>22</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2.400000</td>
      <td>29/08/2013</td>
      <td>2013</td>
      <td>8</td>
      <td>22</td>
      <td>3</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>duration</th>
      <th>start_date</th>
      <th>start_year</th>
      <th>start_month</th>
      <th>start_hour</th>
      <th>weekday</th>
      <th>start_city</th>
      <th>end_city</th>
      <th>subscription_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>669939</th>
      <td>25.600000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>8</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669940</th>
      <td>25.750000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>8</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669941</th>
      <td>21.500000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>8</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669942</th>
      <td>10.500000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>8</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669943</th>
      <td>5.550000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>8</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669944</th>
      <td>115.616667</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>8</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669945</th>
      <td>7.500000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>8</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669946</th>
      <td>2.683333</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>8</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669947</th>
      <td>289.933333</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>7</td>
      <td>0</td>
      <td>Mountain View</td>
      <td>Mountain View</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669948</th>
      <td>288.283333</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>7</td>
      <td>0</td>
      <td>Mountain View</td>
      <td>Mountain View</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669949</th>
      <td>2.816667</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>7</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669950</th>
      <td>94.450000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>7</td>
      <td>0</td>
      <td>San Jose</td>
      <td>San Jose</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669951</th>
      <td>7.350000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>6</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669952</th>
      <td>6.633333</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>5</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669953</th>
      <td>4.000000</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>4</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669954</th>
      <td>10.316667</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>4</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Subscriber</td>
    </tr>
    <tr>
      <th>669955</th>
      <td>111.866667</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669956</th>
      <td>8.966667</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>0</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669957</th>
      <td>9.466667</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>0</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Customer</td>
    </tr>
    <tr>
      <th>669958</th>
      <td>9.483333</td>
      <td>01/09/2014</td>
      <td>2014</td>
      <td>9</td>
      <td>0</td>
      <td>0</td>
      <td>San Francisco</td>
      <td>San Francisco</td>
      <td>Customer</td>
    </tr>
  </tbody>
</table>
</div>

