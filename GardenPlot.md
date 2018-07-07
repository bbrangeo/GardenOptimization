#Garden Optimization
##Optimizing the use of a garden space using the veggies growth rate and size

The data set comes from Iowa State University's "Vegetable Harvest Guide" at https://hortnews.extension.iastate.edu/2004/7-23-2004/vegguide.html.  This gives the name of the vegetable, the number of days to maturity, the size of the plant, color attributes, and the comments for each veggie.  We wanto to collect the name of the veggie, the days to maturity, and the size into a pandas data frame.

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

