# Live-Covid19-Data-Analysis

# importing Liberaries

import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import requests 
from bs4 import BeautifulSoup 
import geopandas as gpd
from prettytable import PrettyTable

url = 'https://www.mohfw.gov.in/' 

web_content = requests.get(url).content
print(web_content)

soup = BeautifulSoup(web_content, "html.parser")
soup

extract_contents = lambda row: [x.text.replace('\n', '') for x in row]

stats = [] 
all_rows = soup.find_all('tr') 

for i in all_rows:
  s=i.find_all('td')
  print(s)

for row in all_rows: 
    stat = extract_contents(row.find_all('td')) # find all data cells  
    
    
    if len(stat) == 5: 
        stats.append(stat)
    print(stat)

new_cols = ["Sr.No", "States/UT","Confirmed","Recovered","Deceased"]
state_data = pd.DataFrame(data = stats, columns = new_cols)
print(state_data)


state_data.info()

state_data.info()

state_data['Confirmed'] = state_data['Confirmed'].map(int)
state_data['Recovered'] = state_data['Recovered'].map(int)
state_data['Deceased']  = state_data['Deceased'].map(int)
print(state_data.head(15))
state_data.info()

sum(state_data['Confirmed'])

table = PrettyTable()
table.field_names = (new_cols)
for i in stats:
    table.add_row(i)
table.add_row(["","Total", 
               sum(state_data['Confirmed']), 
               sum(state_data['Recovered']), 
               sum(state_data['Deceased'])])
print(table)

#sns.set_style={darkgrid, whitegrid, dark, white, ticks}
sns.set_style("darkgrid")
plt.figure(figsize = (15,10))
plt.barh(state_data["States/UT"], state_data["Confirmed"].map(int),
         align = 'center', color = 'lightblue', edgecolor = 'blue')
plt.xlabel('No. of Confirmed cases', fontsize = 18)
plt.ylabel('States/UT', fontsize = 18)
plt.gca().invert_yaxis()
plt.xticks(fontsize = 20) 
plt.yticks(fontsize = 20)
plt.title('Total Confirmed Cases Statewise', fontsize = 25)

for index, value in enumerate(state_data["Confirmed"]):
    plt.text(value, index, str(value), fontsize = 12, verticalalignment = 'center')
plt.show()

group_size = [sum(state_data['Confirmed']), 
              sum(state_data['Recovered']), 
              sum(state_data['Deceased'])]

group_labels = ['Confirmed\n' + str(sum(state_data['Confirmed'])), 
                'Recovered\n' + str(sum(state_data['Recovered'])), 
                'Deceased\n'  + str(sum(state_data['Deceased']))]
custom_colors = ['skyblue','yellowgreen','tomato']

plt.figure(figsize = (5,5))
plt.pie(group_size, labels = group_labels, colors = custom_colors)
central_circle = plt.Circle((0,0), 0.5, color = 'white')
fig = plt.gcf() #get the current figure
fig.gca().add_artist(central_circle)#get the current Axes
plt.rc('font', size = 15) 
plt.title('Nationwide total Confirmed, Recovered and Deceased Cases', fontsize = 16)
plt.show()

map_data = gpd.read_file('Indian_States.shp')
map_data.rename(columns = {'st_nm':'States/UT'}, inplace = True)
map_data.head()


map_data['States/UT'] = map_data['States/UT'].str.replace('&', 'and')
map_data['States/UT'].replace('Arunanchal Pradesh', 'Arunachal Pradesh', inplace = True)
map_data['States/UT'].replace('Telangana', 'Telengana', inplace = True)
map_data['States/UT'].replace('NCT of Delhi', 'Delhi', inplace = True)

map_data

state_data

merged_data = pd.merge(map_data, state_data, how = 'left', on = 'States/UT')
#merged_data.fillna(0, inplace = True)
merged_data



merged_data.drop('Sr.No', axis = 1, inplace = True)#use to remove or drop column Sr. No
merged_data.head()

# create figure and axes for Matplotlib and set the title
fig, ax = plt.subplots(1, figsize=(20, 12))
ax.axis('off')
ax.set_title('Covid-19 Statewise Data - Confirmed Cases', fontdict = {'fontsize': '25', 'fontweight' : '3'})

merged_data.plot(column = 'Confirmed', cmap='YlOrRd', linewidth=0.8, ax=ax, edgecolor='0.8', legend = True)
plt.show()

