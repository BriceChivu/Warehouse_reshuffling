# Brands occupancy visualization

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
Getting a snapshot of the current warehouse inventory is nice to have a first idea of the brands occupancy but it would not necessarily show the real picture since the snapshot could have been taken just after a big shipement (in that case the occupancy would be low) or just after an important volume of good receiving (the occupancy would be high). <br/>
To counter that, I thought it would be great to have a discrete distribution of the warehouse allocation over the time and get the average.
