<img src="https://github.com/ManuHernandezz/CarMarketAnalysis/assets/130862652/917d94f9-6dbe-43f6-abee-6f9dae9d777e" width="220" height="200">


# Comprehensive Analysis of Used Car Market Trends

This project involves analyzing a dataset related to vehicle details. The aim is to gain insights into various aspects, including the impact of factors like year, mileage, and ownership on selling prices. Additionally, it explores common brands and models, differences in prices based on fuel type and transmission, and identifies characteristics associated with high-priced cars. The analysis also investigates potential seasonal trends and assesses the relationship between engine performance (MPG) and selling prices.

#### Prior to analysis, data preprocessing tasks such as normalization and data modification are performed. This includes creating and populating new columns to enhance the dataset. In this context, a 'Brand' column is generated to categorize vehicles by their brand names. Following this, data is loaded into the newly created column. Subsequently, a 'Model' column is also created to further classify the vehicles by their specific models, enriching the dataset for subsequent analys

    ALTER TABLE "Car details V3"
    ADD COLUMN Brand TEXT;
    UPDATE "Car details V3"
    SET Brand = SUBSTR(name, 1, INSTR(name, ' ') -1);

    ALTER TABLE "Car details V3"
    ADD COLUMN Model TEXT;
    UPDATE "Car details V3"
    SET Model = SUBSTR(name, INSTR(name, ' ') + 1);

#### The column name 'mileage' is changed to 'MPG,' and the numerical values are converted from strings to floating-point numbers.

    ALTER TABLE 'Car details V3'
    RENAME COLUMN mileage TO MPG;
    UPDATE "Car details V3"
    SET MPG = CAST(SUBSTR(MPG, 1, INSTR(MPG, ' ') - 1) AS FLOAT);

#### The column name 'engine' is changed to 'Engine CC,' and the values are converted to integers.

    ALTER TABLE 'Car details V3'
    RENAME COLUMN engine TO EngineCC;
    UPDATE "Car details V3"
    SET EngineCC = CAST(SUBSTR(EngineCC, 1, INSTR(EngineCC, ' ') - 1) AS INTEGER);

#### The same process is applied to the 'max power' column, where 'bhp' is removed, and the column is renamed.

    ALTER TABLE 'Car details V3'
    RENAME COLUMN max_power TO max_power_HP;
    UPDATE "Car details V3"
    SET max_power_HP = CAST(SUBSTR(max_power_HP, 1, INSTR(max_power_HP, ' ') - 1) AS FLOAT);

####  The 'torque' column is also modified to extract only the numeric values followed by 'Nm', which represents the torque measurement. Additionally, the values are split into a new column called 'RPM' where applicable.

    ALTER TABLE "Car details V3"
    ADD COLUMN Torque_NM INTEGER;
    UPDATE "Car details V3"
    SET Torque_NM = ( CASE
          WHEN torque LIKE '%Nm%' THEN CAST(REPLACE(SUBSTR(torque, 1, INSTR(torque, 'Nm') - 1), ',', '') AS INTEGER)
          WHEN torque LIKE '% - %' THEN CAST(REPLACE(SUBSTR(torque, 1, INSTR(torque, ' - ') - 1), ',', '') AS INTEGER)
          ELSE NULL
      END);

    ALTER TABLE "Car details V3"
    ADD COLUMN RPM INTEGER;

    UPDATE "Car details V3"
    SET RPM = (CASE
          WHEN torque LIKE '%rpm%' AND Torque_NM IS NULL THEN CAST(REPLACE(SUBSTR(torque, INSTR(torque, ' ') + 1, INSTR(torque, 'rpm') - INSTR(torque, ' ') - 1), ',', '') AS INTEGER)
          ELSE NULL
      END);

#### Null values in the columns 'MPG,' 'RPM,' 'Torque_NM,' 'EngineCC,' and 'max_power_HP' are replaced with their respective column averages to ensure data consistency and accuracy in the analysis.

#### MPG
    UPDATE "Car details V3"
    SET 
      MPG = (SELECT CAST(AVG(MPG) AS INTEGER)
          FROM "Car details V3"
          WHERE MPG IS NOT NULL)
    WHERE MPG IS NULL;
    
#### EngineCC
    UPDATE "Car details V3"
    SET 
      EngineCC = (SELECT CAST(AVG(EngineCC) AS INTEGER)
          FROM "Car details V3"
          WHERE EngineCC IS NOT NULL)
    WHERE EngineCC IS NULL;

#### max_power_HP
    UPDATE "Car details V3"
      SET max_power_HP = (SELECT CAST(AVG(max_power_HP) AS INTEGER)
        FROM "Car details V3"
        WHERE max_power_HP IS NOT NULL)
    WHERE max_power_HP IS NULL;

#### Torque_NM

    UPDATE "Car details V3"
    SET 
      Torque_NM = (SELECT CAST(AVG(Torque_NM) AS INTEGER)
        FROM "Car details V3"
        WHERE Torque_NM IS NOT NULL)
    WHERE Torque_NM IS NULL;

#### RPM
    UPDATE "Car details V3"
      SET 
      RPM = (SELECT CAST(AVG(RPM) AS INTEGER)
        FROM "Car details V3"
        WHERE RPM IS NOT NULL)
        WHERE RPM IS NULL;

#### The mode is calculated for the 'seats' column, which represents the number of seats in the vehicles. This provides insight into the most common seating capacity among the cars in the dataset.
    UPDATE "Car details V3"
    SET 
      seats = (SELECT seats
          FROM (SELECT seats, COUNT(*) AS count
              FROM "Car details V3"
              WHERE seats IS NOT NULL
              GROUP BY seats
              ORDER BY count DESC
              LIMIT 1)
              )
    WHERE seats IS NULL;

# **Tasks**

## 1) What is the relationship between the vehicle's year and its selling price?

    SELECT year, ROUND(AVG(selling_price),2) AS AVG_Price
    FROM 'Car details V3'
    GROUP BY year
    ORDER BY year ASC
    LIMIT 10

|year|	AVG_Price|
| :---- | :------------|
|1983|	300000.0|
|1991|	55000.0|
|1994|	88000.0|
|1995|	107500.0|
|1996|	81666.67|
|1997|	90181.73|
|1998|	73100.0|
|1999|	75833.33|
|2000|	93041.55|
|2001|	48498.3|


## 2) How does the mileage affect the selling price of a car?

    SELECT km_driven, ROUND(AVG(selling_price),2) AS AVG_Price
    FROM 'Car details V3'
    GROUP BY km_driven
    ORDER BY AVG_PRICE DESC
    LIMIT 20

|km_driven	|AVG_Price|
| :-------| :---------|
|23600	|6523000.0|
|28156	|6000000.0|
|8500	|5339064.52|
|7500	|5000500.0|
|7976	|4600000.0|
|17100	|3900000.0|
|31800	|3500000.0|
|17601	|3251000.0|
|13663	|3200000.0|
|29500	|2593333.33|
|21000	|2270416.67|
|9000	|2201410.26|
|2000	|2166142.86|
|68089	|2000000.0|
|24857	|2000000.0|
|17000	|1997040.0|
|2700|	1950000.0|
|1600|	1900000.0|
|76131|	1850000.0|
|18890|	1850000.0|

## 3) What are the most common brands and models in the dataset?

    SELECT
      Brand,
      Model,
	    COUNT(*) AS Count_Models
    FROM "Car details V3"
    GROUP BY Brand , Model
    ORDER BY Count_Models DESC
    LIMIT 25

|Brand	|Model|	Count_Models|
| :-------| :---------|:---------|
|Maruti	|Swift Dzire VDI|	129|
|Maruti	|Alto 800 LXI|	82|
|Maruti	|Alto LXi	|71|
|BMW	|X4 M Sport X xDrive20d|	62|
|Maruti	|Swift VDI	|61|
|Maruti	|Swift VDI BSIV	|59|
|Maruti	|Wagon R LXI	|53|
|Maruti|	Alto K10 VXI|	50|
|Hyundai|	EON Era Plus|	48|
|Maruti	|Ertiga VDI	|45|
|Maruti	|Wagon R VXI BS IV|	45|
|Maruti	|Alto LX|	44|
|Toyota	|Innova 2.5 VX (Diesel) 7 Seater|	44|
|Maruti	|Ritz VDi	|42|
|Maruti	|800 AC	|38|
|Maruti	|Swift Dzire VXI|	38|
|Tata	|Safari Storme EX	|38|
|Maruti	|Baleno Alpha 1.3	|37|
|Hyundai|	Grand i10 1.2 CRDi Sportz	|34|
|Jaguar	|XF 2.0 Diesel Portfolio	|34|
|Lexus	|ES 300h	|34|
|Maruti	|Swift Dzire VDi	|33|
|Renault|	KWID RXT	|33|
|Toyota	|Camry 2.5 Hybrid	|33|
|Toyota	|Etios VX	|33|

## 4) Is there a significant difference in selling price between different types of fuel?

    SELECT fuel, ROUND(AVG(selling_price),2) AS AVG_Price
    FROM 'Car details V3'
    GROUP BY fuel
    ORDER BY AVG_Price DESC

|fuel	|AVG_Price|
| :-------| :---------|
|Diesel	|791452.92|
|Petrol	|462441.06
|CNG	|301017.49|
|LPG	|200421.05|

## 5) What is the distribution of cars by transmission type (manual vs. automatic)?
  
    SELECT transmission AS Transmission, COUNT(*) AS Count_Transmission
    FROM 'Car details V3'
    GROUP BY transmission
    ORDER BY Count_Transmission DESC

|Transmission	|Count_Transmission1
| :-------| :---------|
|Manual	|7078|
|Automatic	|1050|

## 6) What is the impact of the number of previous owners on the selling price?

    SELECT owner,  ROUND(AVG(selling_price),2) AS AVG_Price
    FROM 'Car details V3'
    GROUP BY owner 
    ORDER BY AVG_PRICE DESC

|owner	|AVG_Price|
| :-------| :---------|
|Test Drive Car	|4403800.0|
|First Owner	|783086.41|
|Second Owner|	392964.47|
|Third Owner	|284015.33|
|Fourth & Above Owner|	225813.17|

## 7) What features (such as engine type and maximum power) are more associated with high-priced cars?

    SELECT Brand, EngineCC, MAX(max_power_HP) AS HP, ROUND(AVG(selling_price),2) AS AVG_Price
    FROM 'Car details V3'
    GROUP BY EngineCC
    ORDER BY AVG_Price DESC
    LIMIT 15

|Brand	|EngineCC	|HP	|AVG_Price|
| :-------| :---------| :---------| :---------|
|Lexus|	2487|	214	|5150000.0|
|Ambassador	|1995|	52	|4139764.71|
|Jeep	|3604	|280	|4100000.0|
|Volvo	|1969	|400	|4051428.57|
|Jaguar	|2993	|270	|3849411.76|
|Mercedes-Benz	|2987	|282|	3778636.36|
|Mercedes-Benz	|1950	|194|	3300000.0|
|Jaguar	|1999	|177	|3028956.52|
|Ford	|3198	|197	|2816000.0|
|Volvo	|1984	|181	|2419531.25|
|Ford	|2198	|158	|2250000.0|
|Toyota	|2755	|174	|2223781.25|
|Isuzu	|2999	|174	|2186666.67|
|Mercedes-Benz	|2143	|204|	2151000.0|
|Audi	|1968	|188	|1959409.05|


## 8) Are there any seasonal trends in the data?
    SELECT 	Brand, 
		    COUNT(*) AS Total, 
		    year, 
		    km_driven,
		    ROUND(AVG(selling_price),2) AS AVG_Price
    FROM 'Car details V3'
    GROUP BY year
    ORDER BY Total DESC
    LIMIT 10

|Brand|	Total|	year|	km_driven|	AVG_Price|
| :-------| :---------| :---------| :---------| :---------|
|yundai	|1018	|2017|	45000|	889246.53|
|Mahindra|	859	|2016|	40000|	699880.06|
|Tata	|807	|2018|	35000|	957769.49|
|Maruti	|776	|2015|	40000|	596613.35|
|Ford	|670	|2013|	169000|	460005.92|
|Hyundai|	651	|2012|	53000	|351164.32|
|Maruti	|621	|2014|	145500	|516193.17|
|Toyota	|592	|2011|	90000	|323775.29|
|Maruti	|583	|2019|	10000	|1776986.25|
|Hyundai|	394	|2010|	127000	|272621.79|

## 9) Performance Comparison: What is the relationship between engine performance and selling price?

    SELECT fuel, MPG, selling_price
    FROM 'Car details V3'
    Group by MPG
    Limit 10

|fuel	|MPG|	selling_price|
| :-------| :---------| :---------|
|Diesel	|10.0|	1600000|
|Petrol	|10.1|	325000|
|Diesel	|10.5|	441000|
|Diesel	|10.71|	235000|
|Petrol	|10.75	|1500000|
|Petrol	|10.8	|975000|
|Diesel	|10.9	|480000|
|Diesel	|10.91	|3250000|
|Petrol	|10.93	|1190000|
|Diesel	|11.0	|750000|

## 10) MPG Consumption: Do cars with lower gas mileage have a different selling price?

    SELECT
      CASE
          WHEN MPG BETWEEN 1 AND 15 THEN '1-15 MPG'
          WHEN MPG BETWEEN 16 AND 20 THEN '16-20 MPG'
          WHEN MPG BETWEEN 21 AND 30 THEN '21-30 MPG'
          ELSE '31+ MPG'
      END AS MPG_Category,
      COUNT(*) AS Car_Count,
      AVG(MPG) AS AVG_MPG,
      ROUND(AVG(selling_price), 2) AS AVG_Price
    FROM 'Car details V3'
    GROUP BY MPG_Category
    ORDER BY MPG_CATEGORY ASC;

|MPG_Category|	Car_Count|	AVG_MPG|	AVG_Price|
| :-------| :---------| :---------|:---------|
|1-15 MPG	|1037	|13.1372227579556	|811622.43|
|16-20 MPG	|3061	|18.150137210062	|678141.45|
|21-30 MPG	|2768	|23.6585440751444|	565680.23|
|31+ MPG	|1262	|18.284944532488	|558341.48|
