# scores_selenium

    from selenium import webdriver
    from selenium.webdriver.chrome.options import Options
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.common.by import By
    from selenium.webdriver.support import expected_conditions as EC
    import csv
    import re, time
    import datetime
    import numpy as np
    import pandas as pd

    start_URL = ('https://www.scoresandodds.com/gameDate/2019-09-26')
    start_URL = start_URL[:-10] #Strips the date from the start_URL
    date1 = '2019-09-05' #Starting date of URL to scrape
    date2 = '2019-09-17' #Ending date of URL to scrape
    start = datetime.datetime.strptime(date1, '%Y-%m-%d') #Puts starting date into date format
    end = datetime.datetime.strptime(date2, '%Y-%m-%d') #Puts ending date into date format
    step = datetime.timedelta(days=1) #One day between each date
    new_URL_list = [] #Open up a new URL list
    while start <= end:
        date = start.date()
        if date.weekday() == 0 or date.weekday() == 3 or date.weekday() == 6: #For Mondays, Thursdays, and Sundays only
            date = str(start.date())
            new_URL = start_URL + date
            new_URL_list.append(new_URL)
        start += step 
    new_URL_list #New URLs created from taking stripped down URLs and attaching filtered dates back onto them

    driver = webdriver.Chrome() 
    csv_file = open('scores.csv', 'w', encoding='utf-8', newline='')
    writer = csv.writer(csv_file)

    print(new_URL_list[:2])
    for new_URL in new_URL_list[:2]: #Loops the first two URLs in new_URL_list. (Sometimes can only get data on a couple dates at time, sometimes more)
        print(new_URL)


        driver.get(new_URL)
        print('connected to:', new_URL)
        try:
        
            time_ = driver.find_elements_by_xpath('//div[@class="league-wrapper ng-scope"]')
            time_ = time_[0].find_elements_by_xpath('.//div[@class="inner"]/span[@class="time ng-binding ng-scope"]')
            time_ = list(map(lambda x: x.text,time_)) #Creates a list of all the NFL game times on webpage

            away_scores = driver.find_elements_by_xpath('//div[@class="league-wrapper ng-scope"]')
            away_scores=away_scores[0].find_elements_by_xpath('.//tr[@class="away"]/td[@class="bold-cell game-progress-score ng-scope"]/span')
            away_scores=list(map(lambda x: x.text,away_scores)) #Creates a list of all the NFL away scores on webpage

            home_scores = driver.find_elements_by_xpath('//div[@class="league-wrapper ng-scope"]')
            home_scores=home_scores[0].find_elements_by_xpath('.//tr[@class="home"]/td[@class="bold-cell game-progress-score ng-scope"]/span')
            home_scores=list(map(lambda x: x.text,home_scores)) #Creates a list of all the NFL home scores on webpage

            opened = driver.find_elements_by_xpath('//div[@class="league-wrapper ng-scope"]')
            opened = opened[0].find_elements_by_xpath('.//td[@class="open ng-binding"]')
            opened = list(map(lambda x: x.text,opened)) #Creates list of numbers in the opened columns for NFL games
            opened = [i.split('o', 1)[0] for i in opened]
            opened = [i.split('u', 1)[0] for i in opened]
            opened = [i.split(' ', 1)[0] for i in opened] #These all clean up the characters attached to numbers on webpage
            nested_opened = [] 
            len_opened = int(len(opened)/2)
            for i in range(len_opened):     #This loop seperates the spreads from the totals because they don't follow the same pattern
                nested_opened.append([]) 
                if opened[2*i] > opened[2*i+1]:
                    nested_opened[i].append(opened[2*i])
                    nested_opened[i].append(opened[2*i+1])
                else:
                    nested_opened[i].append(opened[2*i+1])
                    nested_opened[i].append(opened[2*i])

            closed = driver.find_elements_by_xpath('//div[@class="league-wrapper ng-scope"]')
            closed = closed[0].find_elements_by_xpath('.//td[@class="current"]/span')
            closed = list(map(lambda x: x.text,closed)) #Creates list of numbers in the closed columns for NFL games
            closed = [i.split('o', 1)[0] for i in closed]
            closed = [i.split('u', 1)[0] for i in closed]
            closed = [i.split(' ', 1)[0] for i in closed] #These all clean up the characters attached to numbers on webpage
            nested_closed = []   
            len_closed = int(len(closed)/2)
            for i in range(len_closed):   #This loop seperates the spreads from the totals because they don't follow the same pattern
                nested_closed.append([]) 
                if closed[2*i] > closed[2*i+1]:
                    nested_closed[i].append(closed[2*i])
                    nested_closed[i].append(closed[2*i+1])
                else:
                    nested_closed[i].append(closed[2*i+1])
                    nested_closed[i].append(closed[2*i])

            do = dict(zip(['opened_total','opened_spread'],map(list,(zip(*nested_opened))))) #Creates a dictonary between opened total and opened spread
            dc = dict(zip(['closed_total','closed_spread'],map(list,(zip(*nested_closed))))) #Creates a dictonary between closed total and closed spread
            print('do:',do['opened_total'])
            print('dc:',dc['closed_total'])
            for i in range(len(away_scores)): #Assigns each value in dictonary to own columns in the csv file instead of the dictonary appearing in one cell
                d = {}
                d['Time'] = time_[i]
                d['Away Score'] = away_scores[i]
                d['Home Score'] = home_scores[i]
                d['Open Total'] = do['opened_total'][i]
                d['Open Spread'] = do['opened_spread'][i]
                d['Closed Total'] = dc['closed_total'][i]
                d['Closed Spread'] = dc['closed_spread'][i]
                writer.writerow(d.values())

    except Exception as e:
        print(e)
    
        break
# scores_R

    library(plyr)
    count(scores_data,"total_result") #Counts frequency of total_results
    count(scores_data,"spread_result") #Counts frequency of spread_results

    slices <- c(138, 152, 40) #Slices of my pie chart
    labels <- c("Failure", "Sucess", "Stagnant") #Labels of my pie chart
    pct <- round(slices/sum(slices)*100) #Converts to percentage
    labels <- paste(labels, pct) # Add percents to labels
    labels <- paste(labels,"%",sep="") # Add %'s to labels
    pie(slices,labels = labels,col=rainbow(3),main="All Totals") #Formats pie chart

    slices <- c(138, 152) #Slices of my pie chart
    labels <- c("Failure", "Sucess") #Labels of my pie chart
    pct <- round(slices/sum(slices)*100) #Converts to percentage
    labels <- paste(labels, pct) # Add percents to labels
    labels <- paste(labels,"%",sep="") # Add %'s to labels
    pie(slices,labels = labels,col=rainbow(3),main="Only Moving Totals") #Formats pie chart

    slices <- c(120, 126, 84) #Slices of my pie chart
    labels <- c("Failure", "Sucess", "Stagnant") #Labels of my pie chart
    pct <- round(slices/sum(slices)*100) #Converts to percentage
    labels <- paste(labels, pct) # Add percents to labels
    labels <- paste(labels,"%",sep="") # Add %'s to labels
    pie(slices,labels = labels,col=rainbow(3),main="All Spreads") #Formats pie chart

    slices <- c(120, 126) #Slices of my pie chart
    labels <- c("Failure", "Sucess") #Labels of my pie chart
    pct <- round(slices/sum(slices)*100) #Converts to percentage
    labels <- paste(labels, pct) # Add percents to labels
    labels <- paste(labels,"%",sep="") # Add %'s to labels
    pie(slices,labels = labels,col=rainbow(3),main="Only Moving Spreads") #Formats pie chart
