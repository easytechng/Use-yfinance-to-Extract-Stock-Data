# Use-yfinance-to-Extract-Stock-Data
Use-yfinance-to-Extract-Stock-Data
Extracting Tesla and Gamestop Stock Data
Description
Extracting essential data from a dataset and displaying it is a necessary part of data science allowing stakeholders to make correct decisions based on the data. In this notebook, I will extract some stock data and then display this data in a graph. In this notebook, I will extract some stock data and then display this data in a graph.

Objective
My goal is to extract financial data like historical share price and quarterly revenue reportings from various sources using python libraries and webscraping on popular stocks. After collecting this data I will visualize it in a dashboard to identify patterns or trends. Stocks I will work with are Tesla, Amazon, AMD, and GameStop.

!pip install yfinance
#!pip install pandas
#!pip install requests
!pip install bs4
#!pip install plotly
import yfinance as yf
import pandas as pd
import requests
from bs4 import BeautifulSoup
import plotly.graph_objects as go
from plotly.subplots import make_subplots
Define Graphing Function
In this section, I define the function make_graph.

def make_graph(stock_data, revenue_data, stock):
    fig = make_subplots(rows=2, cols=1, shared_xaxes=True, subplot_titles=("Historical Share Price", "Historical Revenue"), vertical_spacing = .3)
    fig.add_trace(go.Scatter(x=pd.to_datetime(stock_data.Date, infer_datetime_format=True), y=stock_data.Close.astype("float"), name="Share Price"), row=1, col=1)
    fig.add_trace(go.Scatter(x=pd.to_datetime(revenue_data.Date, infer_datetime_format=True), y=revenue_data.Revenue.astype("float"), name="Revenue"), row=2, col=1)
    fig.update_xaxes(title_text="Date", row=1, col=1)
    fig.update_xaxes(title_text="Date", row=2, col=1)
    fig.update_yaxes(title_text="Price ($US)", row=1, col=1)
    fig.update_yaxes(title_text="Revenue ($US Millions)", row=2, col=1)
    fig.update_layout(showlegend=False,
    height=900,
    title=stock,
    xaxis_rangeslider_visible=True)
    fig.show()
Using yfinance to Extract Stock Data
Using the Ticker function, the ticker symbol of the stock I want to extract data is entered to create a ticker object. The stock is Tesla and its ticker symbol is TSLA.

Tesla = yf.Ticker("TSLA")
Using the ticker object and the function history, I extract stock information and save it in a dataframe named tesla_data. The period parameter is set to max so I get information for the maximum amount of time.

tesla_data = Tesla.history(period="max")
df=tesla_data
I then reset the index, save, and display the first five rows of the tesla_data dataframe using the head function.

df.reset_index(inplace=True)
df.head()
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }
.dataframe tbody tr th {
    vertical-align: top;
}

.dataframe thead th {
    text-align: right;
}
</style>
Date	Open	High	Low	Close	Volume	Dividends	Stock Splits
0	2010-06-29	3.800	5.000	3.508	4.778	93831500	0	0.0
1	2010-06-30	5.158	6.084	4.660	4.766	85935500	0	0.0
2	2010-07-01	5.000	5.184	4.054	4.392	41094000	0	0.0
3	2010-07-02	4.600	4.620	3.742	3.840	25699000	0	0.0
4	2010-07-06	4.000	4.000	3.166	3.222	34334500	0	0.0
Using Webscraping to Extract Tesla Revenue Data
I use the requests library to download the webpage https://www.macrotrends.net/stocks/charts/TSLA/tesla/revenue. Then save the text of the response as a variable named html_data.

url= "https://www.macrotrends.net/stocks/charts/TSLA/tesla/revenue"
html_data = requests.get(url).text
I Parse the html data using beautiful_soup. Then scrape the table on the webpage into a dataframe, it should look like the table from the website.

soup = BeautifulSoup(html_data,"html.parser")
I use beautiful soup extract the table with Tesla Quarterly Revenue and store it into a dataframe named tesla_revenue. The comma and dollar sign is removed from the Revenue column.

df=pd.read_html(html_data,header=0)
table=soup.find_all('table')
second_table= table[1]
tesla_revenue= pd.DataFrame(columns=["Date","Revenue"])
for row in second_table.find("tbody").find_all("tr"):
    col= row.find_all('td')
    date= col[0].string
    revenue= col[1].string
    tesla_revenue= tesla_revenue.append({"Date":date, "Revenue":revenue}, ignore_index = True)
tesla_revenue["Revenue"]= tesla_revenue['Revenue'].str.replace('$','').str.replace(',','')
print(tesla_revenue)
          Date Revenue
0   2020-12-31   10744
1   2020-09-30    8771
2   2020-06-30    6036
3   2020-03-31    5985
4   2019-12-31    7384
5   2019-09-30    6303
6   2019-06-30    6350
7   2019-03-31    4541
8   2018-12-31    7226
9   2018-09-30    6824
10  2018-06-30    4002
11  2018-03-31    3409
12  2017-12-31    3288
13  2017-09-30    2985
14  2017-06-30    2790
15  2017-03-31    2696
16  2016-12-31    2285
17  2016-09-30    2298
18  2016-06-30    1270
19  2016-03-31    1147
20  2015-12-31    1214
21  2015-09-30     937
22  2015-06-30     955
23  2015-03-31     940
24  2014-12-31     957
25  2014-09-30     852
26  2014-06-30     769
27  2014-03-31     621
28  2013-12-31     615
29  2013-09-30     431
30  2013-06-30     405
31  2013-03-31     562
32  2012-12-31     306
33  2012-09-30      50
34  2012-06-30      27
35  2012-03-31      30
36  2011-12-31      39
37  2011-09-30      58
38  2011-06-30      58
39  2011-03-31      49
40  2010-12-31      36
41  2010-09-30      31
42  2010-06-30      28
43  2010-03-31      21
44  2009-12-31    None
45  2009-09-30      46
46  2009-06-30      27
47  2008-12-31    None
I remove the columns in the dataframe that are empty strings

tesla_revenue.dropna(subset=['Revenue'], inplace=True)
for i in tesla_revenue : tesla_revenue[i] = tesla_revenue[i].astype(str)
#tesla_revenue['Date'] = tesla_revenue['Date'].astype(str)
#tesla_revenue['Revenue'] = tesla_revenue['Revenue'].astype(str)
I display the last 5 row of the tesla_revenue dataframe using the tail function.

tesla_revenue.tail(5)
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }
.dataframe tbody tr th {
    vertical-align: top;
}

.dataframe thead th {
    text-align: right;
}
</style>
Date	Revenue
41	2010-09-30	31
42	2010-06-30	28
43	2010-03-31	21
45	2009-09-30	46
46	2009-06-30	27
Using yfinance to Extract Stock Data pt.2
Using the Ticker function, I enter the ticker symbol of the stock I want to extract data on to create a ticker object. The stock is GameStop and its ticker symbol is GME.

 GME= yf.Ticker("GME")
Using the ticker object and the function history, I extract stock information and save it in a dataframe named gme_data. I then set the period parameter to max so I get information for the maximum amount of time.

GME_share_price_data = GME.history(period="max")
df=GME_share_price_data
I reset the index, save, and display the first five rows of the gme_data dataframe using the head function.

df.reset_index(inplace=True)
df.head()
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }
.dataframe tbody tr th {
    vertical-align: top;
}

.dataframe thead th {
    text-align: right;
}
</style>
Date	Open	High	Low	Close	Volume	Dividends	Stock Splits
0	2002-02-13	6.480513	6.773399	6.413183	6.766666	19054000	0.0	0.0
1	2002-02-14	6.850831	6.864296	6.682506	6.733003	2755400	0.0	0.0
2	2002-02-15	6.733001	6.749833	6.632006	6.699336	2097400	0.0	0.0
3	2002-02-19	6.665671	6.665671	6.312189	6.430017	1852600	0.0	0.0
4	2002-02-20	6.463681	6.648838	6.413183	6.648838	1723200	0.0	0.0
Using Webscraping to Extract GME Revenue Data
I use the requests library to download the webpage https://www.macrotrends.net/stocks/charts/GME/gamestop/revenue. Then I save the text of the response as a variable named html_data.

url= "https://www.macrotrends.net/stocks/charts/GME/gamestop/revenue"
html_data = requests.get(url).text
I parse the html data using beautiful_soupand scrape the table on the webpage into a dataframe. It should look like the table from the website.

soup = BeautifulSoup(html_data,"html.parser")
Using beautiful soup, I extract the table with Tesla Quarterly Revenue and store it into a dataframe named gme_revenue. I make sure the comma and dollar sign is removed from the Revenue column.

df=pd.read_html(html_data,header=0)
table=soup.find_all('table')
second_table= table[1]
gme_revenue= pd.DataFrame(columns=["Date","Revenue"])
for row in second_table.find("tbody").find_all("tr"):
    col= row.find_all('td')
    date= col[0].string
    revenue= col[1].string
    gme_revenue= gme_revenue.append({"Date":date, "Revenue":revenue}, ignore_index = True)
gme_revenue["Revenue"]= gme_revenue['Revenue'].str.replace('$','').str.replace(',','')

gme_revenue.dropna(subset=['Revenue'], inplace=True)
for i in gme_revenue : gme_revenue[i] = gme_revenue[i].astype(str)
print(gme_revenue)
          Date Revenue
0   2020-10-31    1005
1   2020-07-31     942
2   2020-04-30    1021
3   2020-01-31    2194
4   2019-10-31    1439
..         ...     ...
59  2006-01-31    1667
60  2005-10-31     534
61  2005-07-31     416
62  2005-04-30     475
63  2005-01-31     709

[64 rows x 2 columns]
I display the last five rows of the gme_revenue dataframe using the tail function.

gme_revenue.tail()
<style scoped> .dataframe tbody tr th:only-of-type { vertical-align: middle; }
.dataframe tbody tr th {
    vertical-align: top;
}

.dataframe thead th {
    text-align: right;
}
</style>
Date	Revenue
59	2006-01-31	1667
60	2005-10-31	534
61	2005-07-31	416
62	2005-04-30	475
63	2005-01-31	709
Plotting Tesla Stock Graph
I use the make_graph function to graph the Tesla Stock Data, also provide a title for the graph.

make_graph(tesla_data,tesla_revenue,"Tesla Stock Data")
