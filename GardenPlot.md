# Garden Optimization
## Planning a Planting Calendar for a Garden Plot using the Veggies' Growth Rates and Sizes

If you have more space then you know what to do with, you would not have to worry about what to plant where and when.  You could plant everything, anywhere, all the time, and enjoy what grows.  But in my case, and many others', we are limited on space to grow food.  Succession planting can increase the amount of food you grow by continually starting plants throughout the year.  It is not always obvious where and when to start new plants while parts of the growing space is occupied.  To help with this, I've made the following program to plan out a 8 foot by 4 foot garden plot based on the size of the plant and the time it takes to grow.  The result is a 6 month graphical calendar and a few iterations of the program produce a list of what to plant and when.  It also shows us where we may have a few extra spots to pack in a couple more plants because there are always extra seedlings.  



# Part 1 - Collecting the Data
The data set comes from Iowa State University's "Vegetable Harvest Guide" at https://hortnews.extension.iastate.edu/2004/7-23-2004/vegguide.html.  This gives the name of the vegetable, the number of days to maturity, the size of the plant, color attributes, and the comments for each veggie.  Note that this data will be specific to Iowa, although it is applicable to most growing conditions.  We wanto to collect the name of the veggie, the days to maturity, and the size of the vegetable into a pandas data frame.

To start with, lets import some packages we will need.
```python
import pandas as pd 
import requests #to pull html pages#
import bs4 #BeautifulSoup4 to scrape the webpage#
```
We pull the web page data for scraping and put it into a BeautifulSoup element.  This is straight forward
```python
url = 'https://hortnews.extension.iastate.edu/2004/7-23-2004/vegguide.html'
response=requests.get(url)
html=response.content
soup=bs4.BeautifulSoup(html)
``` 
BeautifulSoup4 is used to scrape the data.  This website does not have special tags for each type of data point, so a few extra lines are needed to parse the info. Each cell entry in the table is tagged with **td**  So we can first make a list called "data_cells" of every entry using **findAll('td')**.  This list will be a list of tags which means they will have the class attributes and functionality of BeautifulSoup elements

```python
data_cells=soup.table.findAll('td')
```
Next, we parse this further by separating every cell to its own element in a new list called "data".  This is done with a list comprehension and we use the bs4 class function **.text** to pull out the text within the tags.  A quick check of the data shows that each element has a line break character "\n", so we **strip** each of these as we scrape

```python
data=[data_cells[i].text.strip('\n') for i in range(len(data_cells)-1)]
```
The first few elements of this list are
```python
in: data[:10]
out: ['Vegetable Harvest Guide', 'Vegetable', 'Days  to Maturity ', 'Size  ', 'Color', 'Comment', 
     'Beet', '50-70', '2-3   in diameter', 'red,  varies with cultivar']
```
This is pretty good. It contains the data that we want, plus some extra items that need to be cleaned up

# Part 2 - Cleaning the Data
This data will be put into a pandas data frame, so first, let's make a data frame named 'farm' with the columns 'Veg', 'Days', and 'Size'

```python
farm=pd.DataFrame(columns=('Veg','Days','Size'))
```
We'll fill this frame column by column using the the structure of the list data.  We see from **data[:10]** that we dont want the first 7 elements.  Then, each veggie contains 6 cells of information, so we can index by 5 for each subsequent piece of information that we want.  We can also find with **len(data)** that the total length of the data list is 204, and remember that the **len** function is 1 based, not zero based.  So we can make a list of veggies using

```python
veglist=[data[i] for i in range(6,203,5)] # each veg name is separated by 5 other peices of data
```
Similarly for the days to maturity and the size of each plant. We do the same but move the starting point by 1 for each

```python
timelist=[data[i] for i in range(7,203,5)]
sizelist=[data[i] for i in range(8,203,5)]
```
Checking these lists, we see that the end of veglist and timelist has a strange entry about "*from set bulbs*'.  This comes from the original table on the webpage that had joined cells at the bottom.  This would not allow the dataframe to be constructed by columns since they have different lengths.  We can check the length of each list just made and we see that **len(veglist)=40 , len(sizelist)=40, len(timelist)=39**.  This can be fixed by deleting the last element of veglist and sizelist 

```python
del veglist[-1]
del timelist[-1]
```
Now that the columns are all equal sizes and lined up, they go into the data frame as

```python
farm['Veg']=veglist
farm['Days']=timelist
farm['Size']=sizelist
```
N.B.  it may help to change the display properties so that the data shows up explicity, not as "...." depending on your interface.  The following fixed my issue
```
pd.set_option('display.max_columns', 500)  ##changes display properties on the screen
pd.set_option('display.width', 1000)

                         Veg                 Days                             Size
0                       Beet                50-70                2-3   in diameter
1                   Broccoli               50-65*                  6 to  7  across
2                    Cabbage               60-90*            varies  with cultivar
3                     Carrot                60-80                3/4   in diameter
4                Cauliflower               55-80*                  6 to 8   across
5                   Cucumber                                                      
6                   Pickling                55-65                       2-4   long
7                    Slicing                55-65                       6-8   long
8                   Eggplant               75-90*            varies  with cultivar
9                     Garlic                 90**                              2-3
10                  Kohlrabi                55-70                   2-3   diameter
11           Lettuce  (leaf)                45-60                       4-6   long
12   Muskmelon    Cantaloupe               75-100               5-10   in diameter
13                      Okra                50-65                         3   long
14                     Onion  100-120  90-100**              varies  with cultivar
15                   Parsnip              110-130                      8-18"  long
16                      Peas                                                      
17              Snow (Sugar)                55-85                    3   long pods
18                     Snap                 55-85                    3  long  pods
19            Garden (Shell)                55-85                    3   long pods
20                    Pepper                                                      
21                     Hot                 60-90*                    1 to  3  long
22                    Pepper               70-90*            2 to  4  in diameter 
23                    Potato               90-120            varies  with cultivar
24                   Pumpkin               85-120            varies  with cultivar
25                    Radish                                                      
26                    Spring                25-40          1/2   to 2  in diameter
27                    Winter                45-70                      6-12   long
28   Snap  Bean (Green Bean)                50-70         4 to  6      long       
29                   Spinach                45-60                       6-8   tall
30            Summer  squash                                                      
31                   Scallop                50-60             3 to  5  in diameter
32                  Zucchini                50-60                   6 to  12  long
33               Sweet  Corn               70-105  5 to  10 , varies with cultivar
34             Sweet  Potato              100-125            varies  with cultivar
35                    Tomato               70-90*            varies  with cultivar
36                    Turnip                45-70                2-3   in diameter
37                Watermelon               80-100            varies  with cultivar
38            Winter  Squash               85-120            varies  with cultivar   
```
The next thing that needs to be cleaned is the rows that are missing the "days" and "size" data.  The reason for this is that there are  multiple varieties for some of the veggie entries.  Since the data was not tagged in the html source, it needs to be cleaned up manually.  For this, we simply look at the veggies that have multiple varieties below and then modify the name in the "Veg" column to contain the name and variety.  For example, 'pickling' in row 6 will become 'Cucumber pickling'.  After we change the names, we can delete the row without data.  
```python
farm.loc[6][0]= ((farm.loc[5][0]+farm.loc[6][0]).replace("   "," "))#change name to add 'cucumbers' 
farm.loc[7][0]= ((farm.loc[5][0]+farm.loc[7][0]).replace("   "," "))

farm.loc[17][0]= ((farm.loc[16][0]+farm.loc[17][0]).replace("   "," "))#change peas
farm.loc[18][0]= ((farm.loc[16][0]+farm.loc[18][0]).replace("   "," "))
# do also for the other duplicate varieties
```
Then we remove these partially empty rows with **drop(index=[rows])** and reindex the array with **reset_index**.  The **drop = True** ensures that the old indices are dropped and not added as a new column.

```python
farm=farm.drop(index=[5,16,20,25,30]).reset_index(drop=True)
```
This is a bit better and has all the information we need.  Thinking ahead to the optimization, it would be better if the days to maturity were a single number, instead of a range.  For anyone who has grown their own veggies before, you know that there is always a lot of variation depending on sun, temperature, soil, water, and other variables that are not part of this model.  So to make this easier to quantify, we'll change the time ranges for days to maturity to average values. In order to do this, we need to parse each cell to find the two numbers, and then take the average.  There are some foot notes on these timelines, signaled by asteriks, that refer to whether they are started from seeds or transplants (one asterik), or from bulbs, like garlic or onion, which is signaled with two asteriks.  For simplicity, we will remove these special conditions since they don't have much effect on the overall timelines nor does our optimization cover these details.  There is one data point, however, for onions with two time ranges.  Looking at these days, they are again similar, so we will remove the second day range from this cell then replace it with a string so that it is the same data type for the averaging over the entire list

```python
farm.loc[13][1]='100-120'# change the days for row 13
```
Then we want to remove the asteriks and trailing whitespaces.  The single and double asteriks can be removed together with the expression **strip(&ast;'*')** which removes any number of asteriks, and then we split the data over the hyphen and put the pair into a list. In order to find the average, we need to change every entry to a pair of integers.  This is accomplished with the **map(int, [list])** function. The **map** works very well here because each element in the list is also a list, and that is ok for **map**  All this can go into a single list comprehension to make a temporary list of the number pairs 

```python
days=[list(map(int,days.strip(' ').strip(*'*').split('-'))) for days in farm['Days']] #change all entries to pairs of integers
```
Then finding the average is simple, just be aware that **not every cell has two entries, so dont divide by 2, divide by len([list])** and then put this list back into the farm data frame

```python
days=[sum(time)/len(time) for time in days] #compute averages of each cell
farm['Days']=days  #replace Days column with average values
```
This now brings the farm data frame to this

```
                         Veg   Days                             Size
0                       Beet   60.0                2-3   in diameter
1                   Broccoli   57.5                  6 to  7  across
2                    Cabbage   75.0            varies  with cultivar
3                     Carrot   70.0                3/4   in diameter
4                Cauliflower   67.5                  6 to 8   across
5         Cucumber  Pickling   60.0                       2-4   long
6          Cucumber  Slicing   60.0                       6-8   long
7                   Eggplant   82.5            varies  with cultivar
8                     Garlic   90.0                              2-3
9                   Kohlrabi   62.5                   2-3   diameter
10           Lettuce  (leaf)   52.5                       4-6   long
11   Muskmelon    Cantaloupe   87.5               5-10   in diameter
12                      Okra   57.5                         3   long
13                     Onion  110.0            varies  with cultivar
14                   Parsnip  120.0                      8-18"  long
15         Peas Snow (Sugar)   70.0                    3   long pods
16                Peas Snap    70.0                    3  long  pods
17            Garden (Shell)   70.0                    3   long pods
18              Pepper Hot     75.0                    1 to  3  long
19              PepperPepper   80.0            2 to  4  in diameter 
20                    Potato  105.0            varies  with cultivar
21                   Pumpkin  102.5            varies  with cultivar
22            Radish  Spring   32.5          1/2   to 2  in diameter
23            Radish  Winter   57.5                      6-12   long
24   Snap  Bean (Green Bean)   60.0         4 to  6      long       
25                   Spinach   52.5                       6-8   tall
26   Summer  squash  Scallop   55.0             3 to  5  in diameter
27  Summer  squash  Zucchini   55.0                   6 to  12  long
28               Sweet  Corn   87.5  5 to  10 , varies with cultivar
29             Sweet  Potato  112.5            varies  with cultivar
30                    Tomato   80.0            varies  with cultivar
31                    Turnip   57.5                2-3   in diameter
32                Watermelon   90.0            varies  with cultivar
33            Winter  Squash  102.5            varies  with cultivar
```
Next step in cleaning the data is to quantify the column of sizes. This is not as straightforward as computing the averages since many of the plants have the entry "varies with cultivar", which means there is a large variation depending on which variety and which lineage of seeds you have.  In this case we can fill in the data with estimated values based on educated guesses.  Turns out I know a bit about gardening [js gardens portfolio](https://www.instagram.com/jsgardens/), so the numbers will be appropriate.  The value for the size should represent the amount of space the plant will occupy when at it's largest size during harvest.  Some of the values in this current list are the size of the vegetable or fruit which is not the right value for our purposes.  A good rule of thumb is that a plant usually needs about a foot square, or 12 inches square.  Some plants, like tomatoes, beans, , and cucumbers could take a lot of vertical space and require trellises, but we wont include those details here.  These plants can be very happy in a 12 in square cage.  Squashes, though, tend to spread pretty far.

Here is a list of numbers that are appropriate, using the exiting numbers where available, and then that list goes into the farm data frame with

```python
new_size_list=[2, 7, 10, 0.75, 8, 16, 15, 12, 3, 3, 6, 16, 12, 3, 3, 12, 12, 12, 12, 12, 16, 20, 2, 2, 12, 5, 16, 16, 8, 16, 16, 2, 20, 20]
farm.Size=new_size_list
```
Then finally here is our farm data frame.

```
                        Veg   Days   Size
0                       Beet   60.0   2.00
1                   Broccoli   57.5   7.00
2                    Cabbage   75.0  10.00
3                     Carrot   70.0   0.75
4                Cauliflower   67.5   8.00
5         Cucumber  Pickling   60.0  16.00
6          Cucumber  Slicing   60.0  15.00
7                   Eggplant   82.5  12.00
8                     Garlic   90.0   3.00
9                   Kohlrabi   62.5   3.00
10           Lettuce  (leaf)   52.5   6.00
11   Muskmelon    Cantaloupe   87.5  16.00
12                      Okra   57.5  12.00
13                     Onion  110.0   3.00
14                   Parsnip  120.0   3.00
15         Peas Snow (Sugar)   70.0  12.00
16                Peas Snap    70.0  12.00
17            Garden (Shell)   70.0  12.00
18              Pepper Hot     75.0  12.00
19              PepperPepper   80.0  12.00
20                    Potato  105.0  16.00
21                   Pumpkin  102.5  20.00
22            Radish  Spring   32.5   2.00
23            Radish  Winter   57.5   2.00
24   Snap  Bean (Green Bean)   60.0  12.00
25                   Spinach   52.5   5.00
26   Summer  squash  Scallop   55.0  16.00
27  Summer  squash  Zucchini   55.0  16.00
28               Sweet  Corn   87.5   8.00
29             Sweet  Potato  112.5  16.00
30                    Tomato   80.0  16.00
31                    Turnip   57.5   2.00
32                Watermelon   90.0  20.00
33            Winter  Squash  102.5  20.00
```
Pretty good.  Next is to make a list of the plants that are planted, with their sizes, and see how they can fill the space.  Let change our data frame so that we can index based on the vegetable name.  We can make a new data frame, set the index as the list of column values from our original data frame, and then transfer the columns for 'Days' and 'Size'

```python
farm2=pd.DataFrame(index=farm['Veg'].tolist(), columns =('Days','Size')) #new dataframe with index as the list of veggies
farm2['Days']=farm.Days.tolist()
farm2.Size=farm.Size.tolist() 

                           Days   Size
Beet                       60.0   2.00
Broccoli                   57.5   7.00
Cabbage                    75.0  10.00
Carrot                     70.0   0.75
Cauliflower                67.5   8.00
Cucumber  Pickling         60.0  16.00
Cucumber  Slicing          60.0  15.00
Eggplant                   82.5  12.00
Garlic                     90.0   3.00
Kohlrabi                   62.5   3.00
Lettuce  (leaf)            52.5   6.00
Muskmelon    Cantaloupe    87.5  16.00
Okra                       57.5  12.00
Onion                     110.0   3.00
Parsnip                   120.0   3.00
Peas Snow (Sugar)          70.0  12.00
Peas Snap                  70.0  12.00
 Garden (Shell)            70.0  12.00
Pepper Hot                 75.0  12.00
PepperPepper               80.0  12.00
Potato                    105.0  16.00
Pumpkin                   102.5  20.00
Radish  Spring             32.5   2.00
Radish  Winter             57.5   2.00
Snap  Bean (Green Bean)    60.0  12.00
Spinach                    52.5   5.00
Summer  squash  Scallop    55.0  16.00
Summer  squash  Zucchini   55.0  16.00
Sweet  Corn                87.5   8.00
Sweet  Potato             112.5  16.00
Tomato                     80.0  16.00
Turnip                     57.5   2.00
Watermelon                 90.0  20.00
Winter  Squash            102.5  20.00
```

If you are a hobby farmer with limited space, you'll want to plant things are not too big and grow quickly.  So lets look at what is available that grows in 60 days or less and is smaller that 16 inches square and call this a new dataframe called 'plants'

```python
plants=farm2.loc[farm2.Days<61 and farm2.Size<16]

                         Days  Size
Beet                     60.0   2.0
Broccoli                 57.5   7.0
Cucumber  Slicing        60.0  15.0
Lettuce  (leaf)          52.5   6.0
Okra                     57.5  12.0
Radish  Spring           32.5   2.0
Radish  Winter           57.5   2.0
Snap  Bean (Green Bean)  60.0  12.0
Spinach                  52.5   5.0
Turnip                   57.5   2.0
```

This is the list of plants that we will be putting into the garden space.

# Part 3 - Modeling a Garden Space #
To decide what to plant, we will set up a graphical description of what plants are growing and when. Let's import a few packages that we need

```python
import numpy as np
import matplotlib
from matplotlib.patches import Circle, Wedge, Polygon
import matplotlib.pyplot as pp
from matplotlib.collections import PatchCollection
import pandas as pd
```
In my old garden, we had a 4 foot by 8 foot plot.  We can make this into a figure element with a little bit of padding around the edges using matplotlib.  For ease of visualization of the locations, we can also plot a grid of dots on top of the box using a list comprehenstion to generate list of discrete lists, and then a for loop to plot the rows of discrete points.   

```python
fig=pp.figure(figsize=(10,5))
ax=fig.gca()

pp.xlim(-5,75)
pp.ylim(-5,50)

x=np.arange(0,72,1)
y=[[i for xp in x] for i in range(48)]
for i in range(48):
    pp.plot(x,y[i],'bo', markersize=1)
pp.show()
```
![Empty Garden Box](https://github.com/jeffsecor/GardenOptimization/blob/master/emptybox.png)

Next, lets plant some veggies! To optimize the space, we want to keep the spacing tight.  So we make a **current_y** variable that tells us where things are planted.  Then we simply line up the plants along the y direction, and make sure they are spaced by one size of each plant.  We've also added a new row to our data frame to give each plant its own color.  Heres the code and how a box will look

```python
current_y=0 #track y value

def plant(veg): #define a function to place the plants in the garden box
    global current_y
    for spot in np.arange((plants.loc[veg].Size)/2,72-((plants.loc[veg].Size)/2)+1,plants.loc[veg].Size): #horizontal spacing
        ax.add_patch(Circle((spot,current_y+((plants.loc[veg].Size)/2)),(plants.loc[veg].Size)/2,color=plants.loc[veg].Color,label=veg,alpha=1))#plots a circle the size and color of specified plant
    current_y=current_y+(plants.loc[veg].Size)  # increments y value in the box
    
plant('Beet')
plant('Beet')
plant('Spinach')
plant('Spinach')
plant('Radish  Spring')
plant('Lettuce  (leaf)')
plant('Radish  Spring')
plant('Broccoli')
plant('Turnip')

pp.show()
```
![Example of rows of plants in the garden box](https://github.com/jeffsecor/GardenOptimization/blob/master/gridtest1.png)

Now we will want to show the garden plot for a series of months.  We can do with with subplots, and our grid constructor will go into a nested for loop structure, and titles will be pulled from a list of months. We need to keep track of our subplot layout in order to number the graphs correctly. For visual clarity let's remove the tick marks because they are not descriptive and take up space  That together chagnges the figure construction and now looks like this

```python
plants=pd.read_csv('plantlist.csv',index_col=0) #im running this off a saved list of plants that was created above
colors=['red', 'orange', 'blue', 'pink', 'grey', 'cyan', 'purple', 'pink', 'green', 'brown']
plants['Color']=colors #add colors to the plants dataframe
print(plants) #test to make sure the data frame looks good

months=['April','May','June','July','Aug','Sept'] 


fig,ax=pp.subplots(3,2,figsize=(8,8))  #three rows and two columns of graphs, 6 months
pp.xlim(-5,75)  #initialize figure window a bit larger than the graph area
pp.ylim(-5,50)

#plot a unit spaced grid of dots for a 72 inch by 48 inch box with month as title
x=np.arange(0,73,1)
y=[[i for xp in x] for i in range(49)]#this makes a list of points along the horizontal for each y value

# remove ticks and labels for each subplot
for j in range(3):
    for k in range(2):
         ax[j][k].set_xticklabels([])
         ax[j][k].set_yticklabels([])
         ax[j][k].set_xticks([])
         ax[j][k].set_yticks([])
         ax[j][k].set_title(months[(2*j+k)]) #strange indices to move through 3,2 matrix of subplots
         for i in range(49):
            ax[j][k].plot(x,y[i],'go', markersize=.5) #plots row of dots
          

current_y=0  #track y value of latest plant
def plant(veg):#define a function to place the plants in the garden box
    global current_y
    for spot in np.arange((plants.loc[veg].Size)/2,72-((plants.loc[veg].Size)/2)+1,plants.loc[veg].Size):
        ax[0][0].add_patch(Circle((spot,current_y+((plants.loc[veg].Size)/2)),(plants.loc[veg].Size)/2,color=plants.loc[veg].Color,label=veg,alpha=1))
    current_y=current_y+(plants.loc[veg].Size)
    
plant('Beet')
plant('Beet')
plant('Spinach')
plant('Spinach')
plant('Radish  Spring')
plant('Lettuce  (leaf)')
plant('Radish  Spring')
plant('Broccoli')
plant('Turnip')

pp.show()
```
![GridTest2](https://github.com/jeffsecor/GardenOptimization/blob/master/gridtest2.PNG)

To plan out the space for the growing season,  we need to include how long each plan will be there.  Most of the plants have a growth rate between 30-60 days, and we chose only those plants that grow in less than 61 days.  This means that each plant will be there for at most two months, and this helps index our spacing according to our subplot structure.  (**class variables would be much more appropriate here, maybe in the next version...**).  Lets add the parameter 'month' to the **plant** function.   Then, in order to index the months onto the 3,2 grid of plots, a bit of math and headscratching leads us to **[math.floor(monthIndex)][monthIndex%2]** to place the plant in the right month.  And if the number of days is more than thrity, we put the plant into the next months plot as well in the same position.  In order to track what is there next month, I've made a list  of lists called **y_occupied** of 48 values, one for every horizontal row, and then for each month.  When a plant goes into a spot, it changes the value to True for that month and the next if it will be there.  That will tell us where to start planting for the next month, and then **current_y** starts filling from there.  

We also add the legend through the patches package.  The pathces package is not very user friendly, so this requires a bit of a workaround to make a customized legend.  To do so, define a list of planted plants, and then put that list into a legend and reference the dataframe for the colors.

```python
      
current_y=0
current_veglist=[] # track what plants have been planted
def plant(veg,month):
    if veg not in current_veglist:#add plant if its new
        current_veglist.append(veg)
    global current_y
    m=months.index(month)
    for spot in np.arange((plants.loc[veg].Size)/2,72-((plants.loc[veg].Size)/2)+1,plants.loc[veg].Size):
        ax[math.floor(m/2)][m%2].add_patch(mpatches.Circle((spot,current_y+((plants.loc[veg].Size)/2)),(plants.loc[veg].Size)/2,color=plants.loc[veg].Color,label=veg,alpha=1))
    if plants.loc[veg].Days>30:
        for spot in np.arange((plants.loc[veg].Size)/2,72-((plants.loc[veg].Size)/2)+1,plants.loc[veg].Size):
            ax[math.floor((m+1)/2)][(m+1)%2].add_patch(mpatches.Circle((spot,current_y+((plants.loc[veg].Size)/2)),(plants.loc[veg].Size)/2,color=plants.loc[veg].Color,label=veg,alpha=1))
            
    current_y=current_y+(plants.loc[veg].Size)
plant('Beet','April')
plant('Beet','April')
plant('Spinach','April')
plant('Spinach','April')
plant('Radish  Spring','April')
plant('Lettuce  (leaf)','April')
plant('Radish  Spring','April')
plant('Broccoli','April')
plant('Turnip','April')
plant('Cucumber  Slicing','April')

leg=[mpatches.Patch(color=plants.loc[plant].Color, label= plant) for plant in current_veglist] #legend creation
fig.legend(handles=leg, ncol=5,loc=('upper center'))

pp.show()
 ```
 
In order to make this a bit more interesting, lets change the number of days for the spring radish and lettuce to 29.  (Actually, Ive grownn radishes and lettuces this quick). This way there will be some turnover in May because these will have already been harvested. Then if we plant the rest of the box for the month of April this is what we get
![TestPlotWithLegend](https://github.com/jeffsecor/GardenOptimization/blob/master/testplot.PNG)

Notice that there is a gap in the middle now because some plants are finished in 30 days.  This is exactly what we need.  We can see where we have space to plant the next crop.  A few iterations of the program with additional plantings gets us to our final planting plan.  

![Final Plot](https://github.com/jeffsecor/GardenOptimization/blob/master/finalplot.PNG)

and the final code for the graphical portion is:
```python
import numpy as np
import matplotlib
import matplotlib.patches as mpatches
import matplotlib.pyplot as pp
from matplotlib.collections import PatchCollection
import pandas as pd
import math

plants=pd.read_csv('plantlist.csv',index_col=0)
colors=['red', 'orange', 'blue', 'lawngreen', 'slateblue', 'cyan', 'purple', 'goldenrod', 'green', 'peru']#colors for the graphic
plants['Color']=colors
print(plants)
months=['April','May','June','July','Aug','Sept']


##initialize figure window a bit larger than the graph area
fig,ax=pp.subplots(3,2,figsize=(8,8))
pp.xlim(-5,75)
pp.ylim(-5,50)

#plot a unit spaced grid of dots for a 72 inch by 48 inch box with month as title
x=np.arange(0,73,1)
#this makes a list of points along the horizontal for each y value
y=[[i for xp in x] for i in range(49)]

#for each subplot remove ticks and set title to appropriate month
for j in range(3):
    for k in range(2):
         ax[j][k].set_xticklabels([])
         ax[j][k].set_yticklabels([])
         ax[j][k].set_xticks([])
         ax[j][k].set_yticks([])
         ax[j][k].set_title(months[(2*j+k)])
         for i in range(49):
            ax[j][k].plot(x,y[i],'go', markersize=.5)
           
            

#track y value
current_y=0
current_veglist=[]
y_occupied=[[False for x in range(47)] for m in months]

#define a function to place the plants in the garden box
def plant(veg,month):
    if veg not in current_veglist:
        current_veglist.append(veg) ##compiles aggegate list of all plants planted

    global current_y
    current_y=y_occupied[months.index(month)].index(False)
    m=months.index(month)

    for spot in np.arange((plants.loc[veg].Size)/2,72-((plants.loc[veg].Size)/2)+1,plants.loc[veg].Size): ##horizontal x values spaced by the plant diameter
        ax[math.floor(m/2)][m%2].add_patch(mpatches.Circle((spot,current_y+((plants.loc[veg].Size)/2)),(plants.loc[veg].Size)/2,color=plants.loc[veg].Color,label=veg,alpha=1))###plants row of plants as circles
        i=current_y ##to make a list of occupied spaces
        while i <current_y+plants.loc[veg].Size:
            y_occupied[months.index(month)][i]=True
            i+=1

    if plants.loc[veg].Days>30: ### puts the plant in next months plot if days >30 and occupies the spot 
        for spot in np.arange((plants.loc[veg].Size)/2,72-((plants.loc[veg].Size)/2)+1,plants.loc[veg].Size):
            ax[math.floor((m+1)/2)][(m+1)%2].add_patch(mpatches.Circle((spot,current_y+((plants.loc[veg].Size)/2)),(plants.loc[veg].Size)/2,color=plants.loc[veg].Color,label=veg,alpha=1))
            i=current_y
            while i <current_y+plants.loc[veg].Size:
                y_occupied[months.index(month)+1][i]=True
                i+=1

    current_y=current_y+(plants.loc[veg].Size) ##update current spot for next veg to be planted

#springtime!!  
plant('Beet','April')
plant('Beet','April')
plant('Spinach','April')
plant('Spinach','April')
plant('Radish  Spring','April')
plant('Radish  Spring','April')
plant('Radish  Spring','April')
plant('Radish  Spring','April')
plant('Lettuce  (leaf)','April')
plant('Lettuce  (leaf)','April')
plant('Lettuce  (leaf)','April')
plant('Spinach','April')
plant('Beet','April')

##put in the big summer plants
plant('Cucumber  (slicing)','May')
plant('Lettuce  (leaf)','May')
plant('Radish  Spring','May')
plant('Radish  Spring','May')

##more big summer plants and filling left over space
plant('Snap  Bean (Green Bean)','June')
plant('Cucumber  (slicing)','July')
plant('Radish  Spring','June')
plant('Radish  Spring','June')
plant('Radish  Spring','June')
plant('Broccoli','June')
plant('Broccoli','June')

#lettuce to fill the gap under the cucumbers
plant('Lettuce  (leaf)','July')

#plants for the cool fall weather
plant('Spinach','Aug')
plant('Spinach','Aug')
plant('Beet','Aug')
plant('Beet','Aug')
plant('Beet','Aug')
plant('Spinach','Aug')
plant('Spinach','Aug')
plant('Radish  Winter','Aug')
plant('Radish  Winter','Aug')
plant('Radish  Winter','Aug')

#september planting, this could have bigger plants, but this program ends in september.  maybe we should build a greenhouse?
plant('Lettuce  (leaf)','Sept')
plant('Lettuce  (leaf)','Sept')
plant('Radish  Spring','Sept')

leg=[mpatches.Patch(color=plants.loc[plant].Color, label= plant) for plant in current_veglist]##custom legend 
fig.legend(handles=leg, ncol=4,loc=('upper center'))

pp.show()



```





