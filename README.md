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
import ast
orders_info_df["Items_Ordered"] = orders_info_df["Items_Ordered"].apply(lambda x: ast.literal_eval(x) if isinstance(x, str) else x)
orders_exploded = orders_info_df.explode("Items_Ordered")
```

- I added product price & quanitity information from my item_info & inventory_levels dataframe
```
orders_merged = orders_exploded.merge(item_info_df[["Product_ID", "Price"]], 
                                      how="left", left_on="Items_Ordered", right_on="Product_ID")
orders_merged = orders_merged.merge(inventory_levels_df[["Product_ID", "Quantity_in_stock"]], 
                                    how="left", on="Product_ID")
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

![image](https://github.com/user-attachments/assets/611275b3-2868-4129-a24f-b7ddad8ed733)

## Step three: 
To analyze this data in alignment with the end goal, I have to create some new calculated columns using the present inputs. 

- Profit margin
Profit is *Price - Cost*, but Profit Margin is *[Profit/Price] * 100*.
```
item_info_df["Profit Margin"] = (item_info_df["Price"] - item_info_df["Cost"]) / item_info_df["Price"] * 100
```

- Total Revenue
For this I would actually have to create a new table because while each item is listed in the *Items Info* sheet, they were recurring in the *Orders Info* sheet based on quantity ordered. I need to generate number of times each item was bought first.
```
# get total revenue for each item
orders_exploded = orders_info_df.explode("Items_Ordered")

orders_merged = orders_exploded.merge(
    item_info_df[["Product_ID", "Price"]], 
    how="left", 
    left_on="Items_Ordered", 
    right_on="Product_ID"
)

orders_merged['Revenue'] = orders_merged['Price'] * orders_merged['Quantity_in_stock']

# group by product
revenue_per_product = orders_merged.groupby('Product_ID')['Revenue'].sum().reset_index()

# save files
item_info_df.to_csv("cleaned_item_info.csv", index=False)
inventory_levels_df.to_csv("cleaned_inventory_levels.csv", index=False)
orders_info_df.to_csv("cleaned_orders_info.csv", index=False)
revenue_per_product.to_csv("revenue_per_product.csv", index=False)
```

## Step four: Create visuals 
To do this, I uploaded each sheet into my *Maven Ski Cleaned* workbook in Google Sheets, and connected to Looker Studio as my data source.



