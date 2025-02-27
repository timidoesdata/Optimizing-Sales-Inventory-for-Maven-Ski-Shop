# Sales-Inventory-Analytics-with-Python-and-Looker-Studio
This project analyzes sales trends, inventory management, and product performance for the Maven Ski Shop. The goal is to provide insights towards improving revenue, reducing overstocking, and to optimize pricing.

## Key insights:

**Sales peformance:**
- Total revenue & average order value
- Best & worst selling products
- Seasonal sales trend

**Customer behaviour:**
- Geograhic distribution
- Buyer frequency and order patterns

**Inventory management:**
- Stock levels vs. Demand
- Potential overstocking

**Pricing strategy:**
- Profit margins per product
- Price competitiveness across currencies

## Tools used:
I used **Python** to process & analyze the data, and **Google Looker Studio** for visualization. 

## Dataset used:
The [Maven Ski Shop dataset](https://docs.google.com/spreadsheets/d/1WnLOQtJeGWXA11dYp_5tTxBtDCmc5wP1/edit?usp=sharing&ouid=107860954201781863308&rtpof=true&sd=true) contains three worksheets; 
- the item_info sheet containing the product names, their sizing and prices
- the inventory_levels sheet containing how much stock is available
- the orders_info sheet containing customer purchase records 
---

## Step one: Loading the dataset
- First, I imported the Pandas module using:
```
import pandas as pd
```

- Next, I loaded the dataset and each sheet into a DataFrame
```
file_path = r"C:\Users\DELL\Downloads\maven_ski_shop_data"
xls = pd.ExcelFile(file_path)

item_info_df = pd.read_excel(xls,"Item_Info")
inventory_levels_df = pd.read_excel(xls, "Inventory_Levels")
orders_info_df = pd.read_excel(xls, "Orders_Info")
```
I also used the .head() function to return the first five rows to be sure it loaded 

![image](https://github.com/user-attachments/assets/9fde1c71-55db-4ded-b134-9032fcd2ffca)

## Step two: Cleaning and processing the data
- In the item_info dataset, there is one missing value in the *Available Sizes* column, and I will be filling that in with "unknown."
```
item_info_df.fillna({"Available Sizes" : "Unknown"}, inplace= True)
```

- For the *Inventory Levels* sheet there are no blanks or inconsistencies, so I moved to the *Orders Info* sheet with blank "Tax" and "Total" columns.<br>
"Tax" is 8% of the "Sub-total", and "Total" is the addition of "Tax" and "Sub-total," therefore;
```
orders_info_df["Tax"] = orders_info_df["Subtotal"] * 0.08
orders_info_df["Total"] = orders_info_df["Tax"] + orders_info_df["Subtotal"]
```

- I also converted the "Items Ordered" column from comma separated values to a list for easier analysis
```
orders_info_df["Items Ordered"] = orders_info_df["Items Ordered"].apply(lambda x:x.split(", ") if isinstance(x, str) else x)
```

- Next, I fixed the data type of the "Order Date" column from string to datetime
```
orders_info_df["Order_Date"] = pd.to_datetime(orders_info_df["Order_Date"])
```
- Lastly, I handled duplicate values across all sheets using the .drop_duplicates() function. 
```
item_info_df.drop_duplicates(inplace= True)
inventory_levels_df.drop_duplicates(inplace= True)
orders_info_df.drop_duplicates(inplace= True)
```
![image](https://github.com/user-attachments/assets/d3755434-000c-40df-8782-e363a2bb0e70)

## Step three: 
To analyze this data in alignment with the end goal, I have to create some new calculated columns using the present inputs. 
- Profit margin
Profit is *Price - Cost*, but Profit Margin is *[Profit/Price] * 100*.
```
item_info_df["Profit Margin"] = (item_info_df["Price"] - item_info_df["Cost"]) / item_info_df["Price"] * 100
```

- Total Revenue


