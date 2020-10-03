# Brands occupancy and warehouse reshuffling 

In order to have an efficiently organized warehouse, regular housekeepings are required. They mainly consist of rethinking the brands space disposition inside the racks.

### 1. Problem definition
My goal was to determine which brands were overflowed and which ones were not, so that we can reevaluate the locations attribution and reshuffle the warehouse accordingly. For this task, I focused only on one floor of the warehouse. 

### 2. Warehouse layout 
Before diving into the analysis, let's first understand how the warehouse is arranged.

![Settings Window](https://github.com/BriceChivu/Data-Warehouse-visualization/blob/master/layout%20lvl4%20screenshot.png) <br/>
*The picture above shows a section of the warehouse layout*

Each aisle has of two sides: even (left) and odd (right). Each of those sides are made of numerous bays (around 14), each bay has several levels (around 10), and each level is made of few positions (mostly 3). <br/>
>The aisle 4P17 for example has 28 bays (from 03 to 30). Its even bays have 9 levels with 3 positions each, its odd bays have 10 levels with 3 positions each, counting for 798 locations in total. Therefore, the aisle 4P17 can host up to 798 pallets.

### 3. Data Analysis
Getting a snapshot of the current warehouse inventory is nice to have a first idea of the brands occupancy but it would not necessarily show the real picture since the snapshot could have been taken just after a big shipment (in that case the occupancy would be low) or just after an important volume of good receiving (the occupancy would be high). <br/>
To counter that, I considered a discrete distribution of the warehouse allocation over the time and looked at the average.

I gathered few snapshots of the inventory for each month and combined them into a big dictionnary of dataframes.

```ruby
# Reading all the snapshots inventory and creating the dictionnary d_init to store them
path = "C:\\Users\\btg168\\Desktop\\Inventory reports test\\"
d_init={}

for file in os.listdir(path):
    d_init[file] = pd.read_excel(path+file)
    d_init[file]['Date File created'] = time.ctime(os.path.getmtime(path+file))
    d_init[file]['Date File created'] = pd.to_datetime(d_init[file]['Date File created'])
    d_init[file]['Date File created'] = d_init[file]['Date File created'].dt.date
```
```ruby
# Concatenating all the dataframes from the dictionnary into one big dataframe called df_concat_raw
df_concat_raw = pd.concat(d_init.values())
```
```ruby

df_concat_filtered = df_concat_raw.merge(df_shelv,on='DSP_LOCN',how='left')
df_concat_filtered = df_concat_filtered.merge(df_S_NS,on='ITEM_NAME',how='left' )
df_concat_filtered = df_concat_filtered.merge(df_FPE[['DSP_LOCN','HP/FP']],on='DSP_LOCN',how='left' )
df_concat_filtered.dropna(subset=['DSP_LOCN'],inplace=True)
df_concat_filtered[['REF_FIELD1']] = df_concat_filtered['REF_FIELD1'].replace(to_replace=\
                                                                          ['HR','R. LAUREN','URBAN DECAY','KERASTASE',\
                                                                           'SKINCEUTICALS','Martin MARGIELA','VICHY',\
                                                                           'Atelier Cologne','VIKTOR ET ROLF',\
                                                                           'ROCHE POSAY','HOUSE 99',\
                                                                           'IMARQUES INTER-DEPARTMENT','CLARISONIC']\
                                                                          ,value = 'Mixed Brands')
df_concat_filtered['to keep'] = df_concat_filtered['DSP_LOCN'].apply(lambda x: False if len(x)-sum(c.isdigit() for c in x)>2 else True)
df_concat_filtered = df_concat_filtered.loc[(df_concat_filtered.DSP_LOCN.str[:2] != 'LT')\
                                        & (df_concat_filtered['INVENTORY_TYPE']=='U') & (df_concat_filtered['ZINDEX']==1)\
                                        & (df_concat_filtered['Shelving'].isna()) & (df_concat_filtered['to keep']== True)\
                                        & (df_concat_filtered.DSP_LOCN.str[:1] == '4')\
                                        & (df_concat_filtered['REF_FIELD1'] != "L'OREAL PARIS")]
df_concat_filtered = df_concat_filtered[['REF_FIELD1','DSP_LOCN','ITEM_NAME','SALE_GRP','Date File created','HP/FP']]
```
