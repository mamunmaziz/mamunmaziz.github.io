---
title: "Python Code: DataFrame manipulation and visualisation"
Date: 2021-05-02
tags: [Data Science Projects with Python]
header:
  image: "/images/CF.jpg"
excerpt: "Data Science, Data Analysis, Python"
mathjax: "true"
---

## Introduction

A simple python code was written for find out the cryptocurrenbcy real time price from webpage .


```python
import pandas as pd
import matplotlib.pyplot as plt
print(" Welcome to The DataFrame Statistician!\n Programmed by Mamun Monzurul Aziz\n ") 
MAIN_MENU = '''Please choose from the following options: 
    1 – Load data from a file 
    2 – View data 
    3 – Clean data 
    4 – Analyse data 
    5 – Visualise data 
    6 - Save data to a file 
    7 - Quit'''

def main():
    print(MAIN_MENU)
    menu_choice = input(">>>")
    
    data_frame = pd.DataFrame()
    while menu_choice != "7": 
   
        if menu_choice == "1":
            data_frame = load_data()
            
        elif menu_choice == "2":
            if data_frame.empty:
                print("No data to display")  
            else:
                view_data(data_frame)
                print(data_frame)               
        elif menu_choice == "3":
            if data_frame.empty:
                print("No data loaded")               
            else:
                clean_data(data_frame)
                print(clean_data)                                   
        elif menu_choice == "4":
            if data_frame.empty:
                print("No data loaded")
            else:
                analyse_data(data_frame)
        elif menu_choice == "5":
            if data_frame.empty:
                print("No data loaded")
            else:
                visualise_data(data_frame)
        elif menu_choice == "6":
            if data_frame.empty:
                print("No dataframe to save")
            else:
                save_data(data_frame)
        else:
            print("Invalid selection!.")
            
        print(MAIN_MENU)
        menu_choice = input(">>>")   
    print("Goodbye")       

    
def load_data():
     
    
    filename = input("Enter the filename: ")
    try:
        data_frame = pd.read_csv(filename, na_values=["NULL", 'none'])
        if filename =="":
                print("Enter a filename! ") 
        elif data_frame.empty:
                print("Enter a valid filename! ")
        else: 
            print("Data has been loaded successfully")
            print("Which column do you want to set as index? (leave blank for none) ")
            for col in data_frame.columns:
                print(col)
            is_valid = False
            while not is_valid:
                try:
                    col_name = input(">>>")
                    if col_name == "":
                        return data_frame
                    else:
                        data_frame = data_frame.set_index(col_name)
                        print(col_name, "set as index ")
                        is_valid = True
                except ValueError:
                    print("Invalid selection!\n")
                except KeyError:    
                    print("Invalid selection!\n") 
            return data_frame
    except ValueError:
          print("Unable to load data.")     
    except FileNotFoundError:    
          print("File not found.")
    except UnboundLocalError:
          print("Invalid selection!")
    
    
            
def view_data(data_frame):
    
     return data_frame
      
  
def clean_data(data_frame):   
  
    CLEAN_MENU = '''Cleaning data:
        1 - Drop rows with missing values 
        2 - Fill missing values 
        3 - Drop duplicate rows 
        4 - Drop column 
        5 - Rename column 
        6 - Finish cleaning'''
        
        
    print("Cleaning...")
    print(data_frame)    
    print(CLEAN_MENU)
       
    choice = input(">>>")   
    while choice != "6": 
            if choice == "1":
                drop_rows(data_frame)
            elif choice == "2":
                 fill_missing(data_frame)
            elif choice == "3":
                 try:
                   count_drop = data_frame.duplicated().sum()
                   data_frame.drop_duplicates(inplace=True)
                   print(f" {count_drop} rows dropped")
                 except UnboundLocalError:
                   return data_frame
            elif choice == "4":
                 drop_column(data_frame)
            elif choice == "5":
                 rename_column(data_frame)
            else:
                print("Invalid selection!")
            
            print(data_frame)
            print(CLEAN_MENU)
            choice = input(">>>")       

def drop_rows(data_frame):
    is_valid = False
    while not is_valid:
        try:
            threshold_value = int(input("Enter the threshold for dropping rows: "))
            data_frame.dropna(thresh=threshold_value, inplace=True)
            is_valid = True
        except ValueError:
            print("Please enter a valid number.") 
    return data_frame

def fill_missing(data_frame):
    is_valid_replace = False
    while not is_valid_replace:
        try:
            replacement_value = int(input("Enter the replacement value: "))
            data_frame.fillna(replacement_value, inplace=True)
            is_valid_replace = True
        except ValueError:
            print("Please enter a valid number.")
                   
    return data_frame

def drop_column(data_frame):
   print("Which column do you want to drop? (leave blank for none)")
   for col in data_frame.columns:
       print(col)
       
   is_valid_col = False
   while not is_valid_col:
       col_name = input(">>>") 
       if col_name == "":
           print("No column dropped.")
           is_valid_col=True
       else:
           try:
               for column in data_frame:
                    if column in col_name:
                        data_frame.drop([col_name], axis=1, inplace=True)
                        print(f"{col_name} dropped. ")
                        is_valid_col=True
                   
           except ValueError:
              print("Invalid selection!")
              is_valid_col=True
   return data_frame 

def rename_column(data_frame):
    print("Which column do you want to rename?")
    for col in data_frame.columns:
       print(col)
    col_headers = data_frame.columns.values.tolist()
    
    is_valid = False
    while not is_valid:       
        col_name = input(">>>")
        try:
             if col_name in col_headers:                    
                 is_valid_col = False
                 while not is_valid_col: 
                     rename = input("Enter the new name:")
                     if rename in col_headers:                                      
                        print('Column name must be unique and non-blank.')
                        is_valid_col = False
                     elif rename == '':
                         print('Column name must be unique and non-blank.')
                         is_valid_col = False
                     else:
                         data_frame.rename(columns = {col_name:rename}, inplace = True)
                         print(f"{col_name} renamed to {rename}. ")
                         is_valid_col=True
                 return data_frame
             else:
                  print("Invalid selection!")
                  is_valid = False
        
        except ValueError:                             
             print("Invalid selection!")
         
    
  
 
   
def analyse_data(data_frame):
    for column in data_frame:
        print(column)
        print("--------") # should this be length-adjusted? 
        print("number of values(n):", len(data_frame[column]))
        print("            minimum:", data_frame[column].min().round(2))
        print("            maximum:", data_frame[column].max().round(2)) 
        print("               mean:", data_frame[column].mean().round(2))
        print("             median:", data_frame[column].median().round(2)) 
        print(" standard deviation:", data_frame[column].std().round(2))
        print("  std. err. of mean:", data_frame[column].sem().round(2)) 
        print()  
    
    Corr_Matrix= data_frame.corr(method ='pearson')
    print(f'{Corr_Matrix}\n')
    
    return data_frame


def visualise_data(data_frame):
    print("Please choose from the following kinds: line, bar, box")
    plot_type = input(">>>").lower()
    while plot_type not in ['line','bar','box']:
      print("Invalid selection!")  
      plot_type = input(">>>").lower()  
    print("Do you want to use subplots? (y/n))")
       
    subplot = input(">>>")
    while subplot not in ['y','n']:
        print("Invalid selection!")
        subplot = input(">>>")
       
       
    print("Please enter the title for this plot (leave blank for no title): ")
    plot_title = input(">>>")
    
    print("Please enter the x-axis label (leave blank for no label): ")
    x_label = input(">>>")
    
    print("Please enter the y-axis label (leave blank for no label): ")
    y_label = input(">>>")
    
    
    if subplot == "y":
            data_frame.plot(kind = plot_type,title=plot_title,xlabel=x_label,ylabel=y_label,subplots = True)
    
    else :
            data_frame.plot(kind = plot_type,title=plot_title,xlabel=x_label,ylabel=y_label, subplots = False)
    plt.tight_layout()
    plt.show()    
   
    return data_frame          
    
def save_data(data_frame):
    
    file_name = input("Enter the filename,including extension: ")
    
    if file_name == '':
            print("Cancelling save operation.")
    else:
        data_frame.to_csv(file_name)
        print(f"Data saved to {file_name}")
     
    return data_frame  

main()          
       
```

