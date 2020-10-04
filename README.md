# Brands occupancy visualization and warehouse reshuffling 

In order to have an efficiently organized warehouse, regular housekeepings are required. They mainly consist of rethinking the brands space disposition inside the racks.

### 1. Problem definition
Our goal is to determine which brands are overflowed and which ones are not, so that we can reevaluate the locations attribution and reshuffle the warehouse accordingly. For this task, we will focus only on one floor of the warehouse. 

### 2. Warehouse layout 
Before diving into the analysis, let's first understand how the warehouse is arranged.

![Settings Window](https://github.com/BriceChivu/Data-Warehouse-visualization/blob/master/layout%20lvl4%20screenshot.png) <br/>
*The picture above shows a section of the warehouse layout*

Each aisle has of two sides: even (left) and odd (right). Each of those sides are made of numerous bays (around 14), each bay has several levels (around 10), and each level is made of few positions (mostly 3). <br/>
>The aisle 4P17 for example has 28 bays (from 03 to 30). Its even bays have 9 levels with 3 positions each, its odd bays have 10 levels with 3 positions each, counting for 798 locations in total. Therefore, the aisle 4P17 can host up to 798 pallets.

### 3. Data Analysis
Getting a snapshot of the current warehouse inventory is nice to have a first idea of the brands occupancy but it would not necessarily show the real picture since the snapshot could have been taken just after a big shipment (in that case the occupancy would be low) or just after an important volume of good receiving (the occupancy would be high). <br/>
To counter that, we will consider a discrete distribution of the warehouse allocation over the time and look at the average.

I gathered few snapshots of the inventory for each month so that we can combine them into a big dictionnary of dataframes.

```ruby
# Reading all the snapshots inventory and creating the dictionnary d_init to store them

path = "C:\\Users\\btg168\\Desktop\\Inventory reports test\\"
d_init = {}

for file in os.listdir(path):
    d_init[file] = pd.read_excel(path+file)
    d_init[file]['Date File created'] = time.ctime(os.path.getmtime(path + file))
    d_init[file]['Date File created'] = pd.to_datetime(d_init[file]['Date File created'])
    d_init[file]['Date File created'] = d_init[file]['Date File created'].dt.date
```
```ruby
# Concatenating all the dataframes from the dictionnary into one big dataframe called df_concat_raw

df_concat_raw = pd.concat(d_init.values())
```
```ruby
# Cleaning the dataframe

df_concat_filtered = df_concat_raw.merge(df_shelv, on='DSP_LOCN', how='left')
df_concat_filtered = df_concat_filtered.merge(df_S_NS, on='ITEM_NAME', how='left' )
df_concat_filtered.dropna(subset = ['DSP_LOCN'], inplace = True)
# Regrouping small brands into the group 'Mixed Brands'
df_concat_filtered[['REF_FIELD1']] = df_concat_filtered['REF_FIELD1'].replace(to_replace =\
                                                                          ['HR','R. LAUREN','URBAN DECAY','KERASTASE',\
                                                                           'SKINCEUTICALS','Martin MARGIELA','VICHY',\
                                                                           'Atelier Cologne','VIKTOR ET ROLF',\
                                                                           'ROCHE POSAY','HOUSE 99',\
                                                                           'IMARQUES INTER-DEPARTMENT','CLARISONIC']\
                                                                          ,value = 'Mixed Brands')
# Creating a new column to spot locations that have more than 1 letter like '6STAGE0404', '4COPACKR01', or 'LT00002524'
df_concat_filtered['to keep'] = df_concat_filtered['DSP_LOCN'].apply(lambda x: False if len(x) - sum(c.isdigit() for c in x)  >1 else True)
# Filtering the locations that are in the racks
df_concat_filtered = df_concat_filtered.loc[(df_concat_filtered['INVENTORY_TYPE'] == 'U') \
                                            & (df_concat_filtered['ZINDEX'] == 1)\
                                            & (df_concat_filtered['Shelving'].isna()) & (df_concat_filtered['to keep'] == True)\
                                            & (df_concat_filtered.DSP_LOCN.str[:1] == '4')\
                                            & (df_concat_filtered['REF_FIELD1'] != "L'OREAL PARIS")]
df_concat_filtered = df_concat_filtered[['REF_FIELD1','DSP_LOCN','ITEM_NAME','SALE_GRP','Date File created']]
df_concat_filtered = df_concat_filtered.reset_index(drop = True)
```
```ruby
# Taking a look at our final dataframe

df_concat_filtered.head()
```
<img src="https://github.com/BriceChivu/Data-Warehouse-visualization/blob/master/df_concat_filtered.png" alt="alt text" width="500" height="200"> <br/>

Great! We have now a big dataframe df_concat_filtered containing all the different snapshots. Next, in order to get the occupancy ratio, we need to have the master file with the maximum occupancy originally allocated for each brand. Let's create this dataframe.

```ruby
# Creating the master file that regroups each brand max occupancy

data = pd.DataFrame({'REF_FIELD1':['Mixed Brands','BIOTHERM','SHU UEMURA','YSL','G.ARMANI','KIEHLS',\
                                   'LANCOME',"L'OREAL PARIS"],'Max Occupancy':[1540,1491,1190,2156,1652,3948,5087,4533]})
```
```ruby
# Taking a look at the master dataframe

df_master
```
<img src="https://github.com/BriceChivu/Data-Warehouse-visualization/blob/master/df_master.png" alt="alt text" width="500" height="200"> <br/>

Finally, we will plot a bar graph for each brand to show this evolution of the ratio occupancy.

```ruby
# Displaying each brand plot

fig,axes = plt.subplots(len(df_concat_filtered['REF_FIELD1'].unique()),figsize=(12,30))
fig.tight_layout(pad=7.0)
i = 0

for ref in list(df_concat_filtered['REF_FIELD1'].unique()):
    df_plot = df_concat_filtered[df_concat_filtered['REF_FIELD1'] ==f'{ref}'\
                                ].groupby(['REF_FIELD1','Date File created'])[['DSP_LOCN']].nunique().reset_index()
    df_plot = df_plot.merge(df_master, on='REF_FIELD1', how='left')
    df_plot['Ratio'] = df_plot.apply(lambda x: x['DSP_LOCN']*100/x['Max Occupancy'], axis=1)
    df_plot = df_plot[['REF_FIELD1','Ratio','Date File created']]
    df_plot.plot(kind='bar',ax=axes[i],x='Date File created')
    axes[i].set_xlabel('')
    axes[i].axhline(y=100, color='r',label='100% occupancy')
    axes[i].axhline(y=df_plot['Ratio'].mean()\
                    , color='g',label = f"average occupancy: {round(df_plot['Ratio'].mean())} %")
    axes[i].legend(loc ='lower left')
    axes[i].set_title(f"{ref}")
    
    i+=1
```
