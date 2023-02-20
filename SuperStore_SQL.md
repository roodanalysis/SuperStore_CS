## Exploring the Superstore Dataset :department_store:

### What is the total sales and profit of the store?
````sql
SELECT SUM(Sales) AS TotalSales, SUM(Profit) AS TotalProfit
FROM ss_data;
````

Answer:

![total_sales_total_profit](https://user-images.githubusercontent.com/125606674/220188754-64295035-8aa4-4273-8c4c-c553f178e835.png)

Seems like a pretty thin profit margin, but it doesn't tell us much, since there are a few factors we have to use to get a clearer idea of what is working, and what is not. Let's see what has been the most and least profitable.
There are categories and sub-categories in this dataset. We can group these two columns while also querying total sales and profit to save us some time.
````sql
SELECT Category, Sub_Category, SUM(Sales) AS TotalSales, SUM(Profit) AS TotalProfit
FROM superstore.ss_data
GROUP BY Category, Sub_Category
ORDER BY TotalSales DESC, TotalProfit DESC;
````

Answer:

![categories_sales_profit](https://user-images.githubusercontent.com/125606674/220189611-df0e0ce9-5290-487a-82a9-09da0d8e1534.png)

## Taking a closer look, we can see a couple anomalies within the dataset. It seems like tables have lost a lot of profit. Let's do a quick visualization to see how it compares:

![totprofit_subcategory_bar](https://user-images.githubusercontent.com/125606674/220194956-43aa658a-00f4-4e84-a2c8-6eb732fe5260.png)

We can tell right away that Tables have lost the most profit, while Bookcases come in second and Art in third. 

--

## What is the overall sales trend?

````sql
SELECT EXTRACT(YEAR FROM Order_Date) AS OrderYear, SUM(Sales) AS TotalSales
FROM superstore.ss_data
GROUP BY OrderYear
ORDER BY OrderYear;
````

Answer:

![year_totalsales_sql](https://user-images.githubusercontent.com/125606674/220196639-8d87d4da-0cfa-4237-b191-ac6c793c99e7.png)

![totalsales_orderyear_graph](https://user-images.githubusercontent.com/125606674/220196859-7b298e43-a3d3-4b0c-92d4-9a2772e8c53e.png)

Seems like a steady increase in sales, except a small dip in 2015. Since we already investigated what sub-categories are responsible for profit loss, we can make an educated guess that Tables, Bookcases, and Art (Supplies) are the culprits. Just to be sure, let's take a quick look using this query:

````sql
SELECT 
  EXTRACT(YEAR FROM Order_Date) AS OrderYear, 
  Sub_Category, 
  SUM(Profit) AS TotalProfit
FROM 
  `superstore.ss_data`
WHERE 
  EXTRACT(YEAR FROM Order_Date) BETWEEN 2014 AND 2016
GROUP BY 
  OrderYear, 
  Sub_Category
HAVING 
  TotalProfit < 0
ORDER BY 
  OrderYear, 
  TotalProfit ASC;
  ````
  
  Answer:
  
  ![subcategory_loss](https://user-images.githubusercontent.com/125606674/220198006-27daf323-ac27-400a-9f7b-9d6a3db52ad9.png)

## Now that we have confidently established what is making the least profit, what is making the most?  

````sql
SELECT Category, Sub_Category, SUM(Sales) AS TotalSales, SUM(Profit) AS TotalProfit
FROM superstore.ss_data
GROUP BY Category, Sub_Category
ORDER BY TotalSales DESC, TotalProfit DESC
LIMIT 10;
````

Answer:

![categories_sales_profit](https://user-images.githubusercontent.com/125606674/220198709-7ee05052-f956-4001-bcc7-d303f66d70b6.png)

- Phones not only have the highest sales, but highest profits as well! 

## The WHO and WHERE: :globe_with_meridians: :mag:

Who are the top 5 customers in terms of sales and profit?

````sql
SELECT Customer_Name, SUM(Sales) AS TotalSales, SUM(Profit) AS TotalProfit
FROM ss_data
GROUP BY Customer_Name
ORDER BY TotalSales DESC, TotalProfit DESC
LIMIT 10;
````

Answer:

![top5_customers](https://user-images.githubusercontent.com/125606674/220201542-ed1b9021-0085-46ac-b75e-dc32d64e61a4.png)


Now we have some names, we can dig a little deeper here. 

Where are these customers located? Assuming they are repeat customers, we can use the ```STRING_AGG``` function to concatenate the customer names into a comma-separated list for each group of region and city. 
The ```DISTINCT``` keyword is used to remove any duplocates, and the ```ORDER BY``` clause is used to sort the names alphabetically.


````sql
SELECT 
  Region,
  City,
  STRING_AGG(DISTINCT Customer_Name, ', ' ORDER BY Customer_Name ASC) AS Customers
FROM 
  `superstore.ss_data`
WHERE 
  Customer_Name IN ('Sean Miller', 'Tamara Chand', 'Raymond Buch', 'Tom Ashbrook', 'Adrian Barton')
GROUP BY 
  Region, 
  City;
````

Answer: 

![customers_by_region_1](https://user-images.githubusercontent.com/125606674/220202669-b3608558-92d6-4003-a655-aad77f59d761.png)

![customers_by_region_2](https://user-images.githubusercontent.com/125606674/220203749-9936cedf-8205-4849-8283-e2a345c4d395.png)

Let's merge these two queries, then bring the results over to Google Sheets to do some visualizations! 

````sql
SELECT 
  s.Region,
  s.City,
  s.Customer_Name,
  SUM(s.Sales) AS TotalSales
FROM 
  `superstore.ss_data` s
WHERE 
  s.Customer_Name IN ('Sean Miller', 'Tamara Chand', 'Raymond Buch', 'Tom Ashbrook', 'Adrian Barton')
GROUP BY 
  s.Region,
  s.City,
  s.Customer_Name
ORDER BY 
  TotalSales DESC;
````

Answer: 

![customer_sales_region_1](https://user-images.githubusercontent.com/125606674/220204563-41900db1-1542-4c22-b27b-0ebcbdc397ca.png)

![customer_sales_region_2](https://user-images.githubusercontent.com/125606674/220204585-9f3ed42a-f02a-4c32-9828-deda937cd24c.png)

![total_sales_vs_city](https://user-images.githubusercontent.com/125606674/220206073-eabbe635-a06e-4cdb-9edb-5d175697bd09.png)

While the city where the top customer generates the most revenue may not necessarily be the city where the top sales are generated overall, it is important to know where to focus on marketing and expansion. 
Additionally, analyzing the difference in sales and profit between the two cities can help identify areas where the company can improve, such as increasing marketing efforts or adjusting pricing strategies. 
By understanding the nuances of how revenue is generated in different regions, a company can make more informed decisions about where to allocate resources and how to drive growth.
With that being said, let's find the city with the highest revenue:

````sql
SELECT 
  City,
  SUM(Sales) as Total_Sales,
  SUM(Profit) as Total_Profit
FROM 
  `superstore.ss_data`
GROUP BY 
  City
ORDER BY 
  Total_Sales DESC, Total_Profit DESC
LIMIT 1;
````

Answer:

![highest_sales_city](https://user-images.githubusercontent.com/125606674/220207609-589a0ae9-4f70-4e9b-af87-1f942d0740d2.png)


New York City generates the most revenue. How does it reflect by state?

````sql
SELECT 
  State,
  SUM(Sales) as Total_Sales,
  SUM(Profit) as Total_Profit
FROM 
  `superstore.ss_data`
GROUP BY 
  State
ORDER BY 
  Total_Sales DESC, Total_Profit DESC
LIMIT 5;
````

![top_sales_by_state](https://user-images.githubusercontent.com/125606674/220207947-e03ff9be-672c-485e-b06e-72fcafca3738.png)

![total_sales_total_profit_state](https://user-images.githubusercontent.com/125606674/220208179-42522dc7-c4ac-419e-84af-84f573f08b1c.png)


California is the most profitable state, while Texas is the least profitable. What is not working in texas?

````sql
SELECT 
  Sub_Category,
  SUM(Profit) AS Total_Profit
FROM 
  `superstore.ss_data`
GROUP BY 
  Sub_Category
ORDER BY 
  Total_Profit ASC;
  `````
  
  ![texas_sub_category_sales](https://user-images.githubusercontent.com/125606674/220208574-b0733957-2433-4485-92ff-f4f98153cec8.png)


--

### Key Takeaways and Suggestions:

- It may be necessary to reevaluate the company's strategy for doing business in Texas. 
This could involve things like exploring new products or markets, adjusting pricing or promotions, or finding ways to reduce costs.

-  The company should focus on growing sales in the West region, which has the highest sales and profits.

- While the top customer is located in Florida, the state with the highest profit is California, indicating the importance of looking beyond individual customers to analyze broader trends and patterns.

- The company could benefit from further analyzing sales and profits at the city level, as this data can reveal key insights about customer behavior and preferences.

## Summary:

The Superstore dataset was used to conduct an analysis of sales and profits. The dataset contained information on customer orders and included columns for product details, order date, shipping date, customer location, and product category.

The analysis was conducted using SQL queries, and visualizations were created using Excel and Google Sheets. The analysis revealed that the top customer generated the highest sales and profits in Florida, while the state of California had the highest overall sales and profits. Additionally, the subcategories of technology and office supplies were the most profitable, while furniture was the least profitable.

The analysis also showed that the state of Texas was the least profitable, and it was noted that the company may want to investigate this further to understand the reasons behind this. Other key takeaways from the analysis included the importance of understanding which products and customer segments are generating the most revenue and profits, as well as the importance of analyzing data at a granular level, such as by subcategory or region.

Overall, the Superstore dataset provided valuable insights into the company's sales and profits, and the analysis can be used to inform strategic decisions and optimize business performance.

## Some quick tableau vizzes: 

![Dashboard 1](https://user-images.githubusercontent.com/125606674/220209707-cb71b806-debf-490e-bcf8-b989b5b91eab.png)

![Viz2](https://user-images.githubusercontent.com/125606674/220209802-f87ff38e-b959-4a27-b021-c4fb399a9024.png)
