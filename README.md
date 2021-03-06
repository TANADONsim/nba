# Observing NBA Game Score as a Stat Line
Observing Game Score of daily leaders and what it shows us about a player's/team's match performance.

## What is Game Score?
1. This statistic was invented by John Hollinger to provide a rough measure of a player's performance in a given game.  The scale upon which the player's game score is based is the same as points scored.  If a player has a game score of 40, that is amazing, while a game score of 10 is average.
2. The formula for calculating game score is as follows: (Points x 1.0) + (FGM x 0.4) + (FGA x -0.7) + ((FTA-FTM) x -0.4) + (OREB x 0.7) + (DREB x 0.3) + (STL x 1.0) + (AST x 0.7) + (BLK x 0.7) + (PF x -0.4) + (TO x -1.0). 

As can be plainly seen, this statistic takes into account almost everything a player does during a game that can be quantified.  These statistics are then weighted and added together to get a game score.  This method is similar to Dr. Dean Oliver's Four Factors, but attempts to add value by considering all the stats, rather than just the four most important ones.

## The Data 
We take a look at the NBA Daily Stat Leaders according to [https://www.basketball-reference.com](https://www.basketball-reference.com/friv/dailyleaders.fcgi?month=11&day=1&year=2018) from 01/11/2018 to 27/11/2018.

## Building the Python code to get the data 
```
import urllib3
import csv
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
from bs4 import BeautifulSoup
import datetime
```
These are the modules we imported into our Python codes:    
**urllib3** is a powerful HTTP client for Python.  
**csv** allows us to import or export spreadsheets and databases for use in the Python interpreter.  
**BeautifulSoup** is a Python library for pulling data out of HTML and XML files.  
**datetime** provides a number of function to deal with dates, times and time intervals.

```
http = urllib3.PoolManager()
url = "https://www.basketball-reference.com/friv/dailyleaders.fcgi?month=11&day="+str(day)+"&year=2018"
response = http.request('GET', url)
```
class **urllib3.poolmanager.PoolManager** allows for arbitrary requests while transparently keeping track of necessary connection pools for you.  

```
soup = BeautifulSoup(response.data, 'html.parser')
tags = soup.find_all("div", attrs={"class": "overthrow table_container", "id": "div_stats"})
tags = tags[0].tbody.contents
team_stats = []
```
Scrape the whole of webpage in url and remove unwanted content, leaving only the stats table we need. Also creating an empty list, "team_stats", to be used later on. 

```
for i in range(1, len(tags), 2):
    contents = tags[i].contents
    stats = []
    name = contents[1].text
    if name != "Rk":
        stats.append(name)
        for j in range(2, len(contents)):
            stats.append(contents[j].text)
        team_stats.append(stats)

```
Scraping the contents within the tags of the stats table. For each new name down the table, take their stats and add it to "team_stats" list created earlier. 

```
with open(str(day)+".csv", "w") as f:
    writer = csv.writer(f)
    writer.writerows(team_stats)
```
Once the stats has been scraped, open and append the data into a new excel sheet according to the current date. 

```
today = datetime.datetime.now().day
scrape(today - 1)
```
When automated to get the stats daily, the code will scrape the data from the url's latest update once a day. Adjusted for the timezone difference between Singapore time and American time.

## Using VBA to automate the process of updating the data in Excel
```
Set dataWb = Workbooks.Open("C:\Users\Tanadon Sim\Desktop\nba\data.xlsx")
dataWb.Sheets(1).Rows("2:" & Rows.Count).ClearContents
xStrPath = ("C:\Users\Tanadon Sim\Desktop\nba")
If xStrPath = "" Then Exit Sub
xFile = Dir(xStrPath & "\" & "*.csv")
```
Set the path for the VBA to the relevant folder. 

```
Do While xFile <> ""
    Set wb = Workbooks.Open(xStrPath & "\" & xFile)
    Set ws = wb.Worksheets(1)
    ws.Columns(1).SpecialCells(xlBlanks).EntireRow.Delete
    lastrow = ws.Cells(ws.Rows.Count, "A").End(xlUp).Row
    Set currentws = dataWb.Worksheets(1)
    startrow = currentws.Cells.SpecialCells(xlCellTypeLastCell).Offset(1, 0).Row
    currentws.Range("B" & startrow & ":Y" & startrow + lastrow - 1).Value = ws.Range("A1:Y" & lastrow).Value
    LArray = Split(xFile, ".")
    currentws.Range("A" & startrow & ":A" & startrow + lastrow - 1).Value = LArray(0)
    Application.DisplayAlerts = False
    wb.Close False
    xFile = Dir
Loop
```
Appending the statistics in an excel file and formatted according to the table taken from the url.  



## Data Visualisation 
<img src="nba.gif" width="1000" height="600" />

Using Tableau, we desgined a dashboard which the GIF above illustrates. It shows the progression of the acquired data sets from 01/11/2018 to 27/11/2018. We shall explore each sheet of the dashbaord further below.   


![nba_sheet1](https://user-images.githubusercontent.com/44894094/49791898-5d651100-fd6c-11e8-84df-c5bac1343e5b.png)

The heat map above depicts the average game score of an individual player across the 27 days in descending order. It is not surprising to have players like Nikola Jokic and Karl-Anthony Towns, both who are amongst the league's elite and key players for their respective teams, leading the way. However, it is interesting to see players such as Joe Ingles and T.J. Warren, both solid role players for their sqaud, ahead of the likes of MVP calibre players such as James Harden and Lebron James. This highlights the importance of role players and how their contribution may be overlooked easily without first weighing the influence of each statistic.  


![nba_sheet2](https://user-images.githubusercontent.com/44894094/49797257-d9b22100-fd79-11e8-90ff-ffc8635349ab.png)

The figure above is an attempt to illustrate the difference in a player's performance at and away from home. It plots the average field goal, 3 point and free throw percentage separately for home (Null) and away (@) matches. Albeit small, a difference in shooting percentages at home and away can be observed. 


![nba_sheet3](https://user-images.githubusercontent.com/44894094/49856715-8698a680-fe2b-11e8-986d-d78c8b9a81d3.png)

The figure above shows the average stats per player per team across the observed dates. 


![nba_sheet4](https://user-images.githubusercontent.com/44894094/49857400-65d15080-fe2d-11e8-998f-dfeaecbed035.png)
The box plot shown above shows the influence of average game score on wins and losses of a team. We could say from the box plot that a low average game score has a larger impact on a team's loss than a high average game score has on a team's win. 


![nba_sheet5](https://user-images.githubusercontent.com/44894094/49948421-656cae80-ff2e-11e8-8ce1-ae24c5a13197.png)

The bar graph above shows the top 5 players in average points per game from 01/11/2018 to 27/11/2018.


![nba_sheet6](https://user-images.githubusercontent.com/44894094/49950708-4b819a80-ff33-11e8-94fe-c3ec282e00bc.png)

The bar graph above shows the top 5 players in average assists per game from 01/11/2018 to 27/11/2018.


![nba_sheet7](https://user-images.githubusercontent.com/44894094/49950768-694eff80-ff33-11e8-99df-3f80d1d46aed.png)

The bar graph above shows the top 5 players in average rebounds per game from 01/11/2018 to 27/11/2018.


![nba_sheet8](https://user-images.githubusercontent.com/44894094/49950812-7bc93900-ff33-11e8-86d0-83c434362b57.png)

The bar graph above shows the top 5 players in average steals per game from 01/11/2018 to 27/11/2018.



