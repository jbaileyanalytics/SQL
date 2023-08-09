# Using a .CSV file to create a MySQL Database, a MySQL Table, and a few sample SQL Queries for Data Analysis #

We will be using a mock(fake) financial dataset from the healthcare industry for our tutorial. The dummy dataset was created using Python. Now, let's get started with our SQL analysis!  
##### First, we are creating a new database called Test_DB:  
CREATE DATABASE IF NOT EXISTS Test_DB; 

##### Specify the database we want to use which is the one we just created above:   
USE Test_DB; 

##### Create a new table in our new database for the data we will load from our .CSV file:  
CREATE TABLE Test_Table   
(  
&nbsp;&nbsp;&nbsp;&nbsp;ServiceDate DATE,  
 &nbsp;&nbsp;&nbsp;&nbsp;ClaimStatus VARCHAR(50),  
 &nbsp;&nbsp;&nbsp;&nbsp;PersonID INT,  
 &nbsp;&nbsp;&nbsp;&nbsp;FinancialGroup VARCHAR(255),  
 &nbsp;&nbsp;&nbsp;&nbsp;InsuranceName VARCHAR(255),  
 &nbsp;&nbsp;&nbsp;&nbsp;AllocationIns DOUBLE,  
 &nbsp;&nbsp;&nbsp;&nbsp;AllocationPt DOUBLE,  
 &nbsp;&nbsp;&nbsp;&nbsp;ChargeTotal DOUBLE  
);

##### Checking for success of table creation:  
SELECT * FROM Test_Table; 

![image](https://github.com/jbaileyanalytics/SQL/assets/117615131/0463f8ab-df4e-4d6e-a8f9-3fb1f38d3d89)
##### As you can see above, we now have a blank table created with our column names. Let's load the data now.  
##### Put your .csv file in the DB directory folder(LocalDriveHere\ProgramData\MySQL\MySQL Server x.x\Data\database_here) before proceeding with this next query. ##
- ##### Load the .CSV file into the table using the LOAD DATA INFILE ('filename') INTO TABLE statement. Specify the delimiter used, in our case the most common with a .CSV file is a comma. Skip the header row of the .CSV file as it is not needed.  
LOAD DATA INFILE 'test1_db.csv' INTO TABLE Test_Table   
FIELDS TERMINATED BY ','  
IGNORE 1 LINES;  

![image](https://github.com/jbaileyanalytics/SQL/assets/117615131/582adb0a-d779-410e-ad4a-1604fa9afc91)  
##### We get a message from MySQL stating the amount of rows loaded. Now, let's check if the data load was successful by running a Select * statement. ##
SELECT * FROM Test_Table;

![image](https://github.com/jbaileyanalytics/SQL/assets/117615131/c75e8fcc-5750-4328-b67a-2d0da93e87c4)

##### Success! Our table is now populated with the data from our .CSV file. Now we can run some sample SQL queries on the data for further analysis.  
    
#### Sample queries for our dataset:

#### 1.) Let's design a query to show the Top 10 Financial Groups by Highest Charges and Percent of Total Charges:
SELECT FinancialGroup,  
	&nbsp;&nbsp;&nbsp;&nbsp;ROUND(SUM(ChargeTotal),2) as Charges_Total,  
    &nbsp;&nbsp;&nbsp;&nbsp;ROUND(100.0 * SUM(ChargeTotal)/  
			&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(SELECT SUM(ChargeTotal) FROM Test_Table), 2) as Percent_of_Charges_Total  
FROM Test_Table  
GROUP BY FinancialGroup  
ORDER BY Charges_Total DESC  
LIMIT 10;

![image](https://github.com/jbaileyanalytics/SQL/assets/117615131/92ee6417-bc15-40eb-9da1-bb40401d066b)


#### 2.) Let's design a query to show the Top 10 Insurance Carriers by Highest Charges and Percent of Total Charges:
SELECT InsuranceName,   
	&nbsp;&nbsp;&nbsp;&nbsp;ROUND(SUM(ChargeTotal),2) as Charges_Total,  
  &nbsp;&nbsp;&nbsp;&nbsp;ROUND(100.0 * SUM(ChargeTotal)/(SELECT SUM(ChargeTotal) FROM Test_Table), 2) as Percent_of_Charges_Total  
FROM Test_Table  
GROUP BY InsuranceName  
ORDER BY Charges_Total DESC  
LIMIT 10;  

![image](https://github.com/jbaileyanalytics/SQL/assets/117615131/ad30c785-56e9-48f1-964d-515545b3fcf9)


#### 3.) Let's design a query to show the Total Charges and Percent of Charges by Claim Status:
SELECT ClaimStatus,   
	&nbsp;&nbsp;&nbsp;&nbsp;ROUND(SUM(ChargeTotal),2) as Charges_Total,  
    &nbsp;&nbsp;&nbsp;&nbsp;ROUND(100.0 * SUM(ChargeTotal)/(SELECT SUM(ChargeTotal) FROM Test_Table), 2) as Percent_of_Charges_Total  
FROM Test_Table  
GROUP BY ClaimStatus  
ORDER BY Charges_Total DESC  
LIMIT 10;  

![image](https://github.com/jbaileyanalytics/SQL/assets/117615131/2ed9f055-6f1a-49b4-b6ed-a2ea4e415937)


#### 4.) Let's design a query to show the highest charge month and charge amount in that month for each Financial Group. We will utilize a CTE(common table expression) for this query:
WITH CTE_FG AS (  
	&nbsp;&nbsp;&nbsp;&nbsp;SELECT FinancialGroup,  
			&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MONTH(ServiceDate) as Service_Month,  
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;YEAR(ServiceDate) as Service_Year,  
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ROUND(SUM(ChargeTotal),2) as Monthly_Total  
	&nbsp;&nbsp;&nbsp;&nbsp;FROM Test_Table  
    &nbsp;&nbsp;&nbsp;&nbsp;GROUP BY FinancialGroup, YEAR(ServiceDate), MONTH(ServiceDate)  
    &nbsp;&nbsp;&nbsp;&nbsp;ORDER BY Monthly_Total DESC  
),  
	CTE_RK AS (  
	&nbsp;&nbsp;&nbsp;&nbsp;SELECT *,   
		&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;RANK() OVER(PARTITION BY FinancialGroup ORDER BY Monthly_Total DESC) as RNK  
	&nbsp;&nbsp;&nbsp;&nbsp;FROM CTE_FG  
)  
SELECT FinancialGroup, Service_Month, Service_Year, Monthly_Total  
FROM CTE_RK  
WHERE RNK = 1;  

![image](https://github.com/jbaileyanalytics/SQL/assets/117615131/2eef88a6-c326-4705-a794-8115a091a776)


#### This concludes our demonstration of how to take a .CSV file and load it into MySQL and perform a short analysis on that dataset via SQL. Thank you for your time and I hope this was informative!
