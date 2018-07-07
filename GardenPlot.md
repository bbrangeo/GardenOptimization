# Garden Optimization
## Optimizing the use of a garden space using the veggies growth rate and size


The data set comes from Iowa State University's "Vegetable Harvest Guide" at https://hortnews.extension.iastate.edu/2004/7-23-2004/vegguide.html.  This gives the name of the vegetable, the number of days to maturity, the size of the plant, color attributes, and the comments for each veggie.  We wanto to collect the name of the veggie, the days to maturity, and the size into a pandas data frame.

### Part 1 - Collecting the Data
To start with, lets import everything we will need.
```
import pandas as pd #to work with the data#
import requests #to pull html pages#
import bs4 #BeautifulSoup4 to scrape the webpage#
```
First we need to collect the web page data and put it into a BeautifulSoup element.  This is straight forward
```
url = 'https://hortnews.extension.iastate.edu/2004/7-23-2004/vegguide.html'
response=requests.get(url)
html=response.content
soup=bs4.BeautifulSoup(html)
``` 

BeautifulSoup4 is used to scrape the data.  This website does not have special tags for each type of data point, so a few extra lines are needed to parse the info. Each cell entry in the table is tagged with 'td'.  So we can first make a list called "data_cells" of every entry using **findAll('td')**.  This list will be a list of tags which means they will have the class attributes and functionality of BeautifulSoup elements
```
data_cells=soup.table.findAll('td')
```
Next, we parse this further by separating every cell to its own element in a new list called "data".  This is done with a list comprehension and we use the class function **.text** to pull out the text within the tags.  Finally, we see that these elements each have a line break character "\n", so we **strip** each of these as we scrape
```
data=[data_cells[i].text.strip('\n') for i in range(len(data_cells)-1)]
```
The first few elements of this list are
```
in: data[:10]
out: ['Vegetable Harvest Guide', 'Vegetable', 'Days  to Maturity ', 'Size  ', 'Color', 'Comment', 
     'Beet', '50-70', '2-3   in diameter', 'red,  varies with cultivar']
```
This is pretty good. Tt contains the data that we want, plus some extra items that need to be cleaned up

### Part 2 - Cleaning the Data
This data will be put into a pandas data frame, so first, let's make a data frame named 'farm' with the columns 'Veg', 'Days', and 'Size'
```
farm=pd.DataFrame(columns=('Veg','Days','Size'))
```
We'll fill this frame column by column, using the pattern of the 'data' list above.  We see from **data[:10]** that we dont want the first 7 elements.  Then, each veggie repeats every 6 items.  We can also find that the total lenght of our data list is 204, and remember that the **len** function is 1 based, not zero based.  So we can make a list of veggies using
```
veglist=[data[i] for i in range(6,203,5)]
```
Similarly for the days to maturity and the size of each plant, we do the same but index our starting point by 1 
```
timelist=[data[i] for i in range(7,203,5)]
sizelist=[data[i] for i in range(8,203,5)]
```
Checking these lists, we see that the end of veglist and timelist has a strange entry about "*from set bulbs*'.  This comes from the original table on the webpage that had joined cells at the bottom.  This would not allow the dataframe to be constructed by columns since they have different lengths.  We can check the length of each list just made and we see that **len(veglist)=40 , len(sizelist)=40, len(timelist)=39**.  This can be fixed by deleting the last element of veglist and sizelist 
```
del veglist[-1]
del timelist[-1]
```
Now that the columns are all equal sizes and lined up, they go into the data frame as
```
farm['Veg']=veglist
farm['Days']=timelist
farm['Size']=sizelist
```
N.B.  it may help to change the display properties so that the data shows up explicity, not as "...." depending on your interface.  The following fixed my issue
```
pd.set_option('display.max_columns', 500)
pd.set_option('display.width', 1000)
```
