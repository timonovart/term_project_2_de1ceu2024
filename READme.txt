# Requirements

- To run the project there are two additional packages that are required in KNIME:

1. KNIME Database:
https://hub.knime.com/knime/extensions/org.knime.features.database/latest

2. Python Script:
https://hub.knime.com/knime/extensions/org.knime.features.python3.scripting/latest/org.knime.python3.scripting.nodes2.script.PythonScriptNodeFactory

The packages may be installed by dragging the icon from the website to the KNIME window.

- The data accessed through AWS cloud with the following credentials:
Host name:  project2.cv408kmm0zzs.eu-north-1.rds.amazonaws.com
Port: 3306
Username: admin
Password: admin123



# Data Description

- Primary Source:
https://happiness-report.s3.amazonaws.com/2023/WHR+23_Statistical_Appendix.pdf
- Data Source for MySQL rest file:
https://www.kaggle.com/datasets/sazidthe1/global-happiness-scores-and-factors
- Data Source for variables in csv file:
https://databank.worldbank.org/

Any further information about the data may be found on the links above. It is easier to consider it in relative terms amongst countries.

- Variables in MySQL:
	- country
	- region
	- happiness score
	- gdp_per_capita
	- social_support
	- healthy_life_expectancy
	- freedom_to_make_life_choices
	- generosity
	- perceptions_of_corruption

- Variables in csv table:
	- year
	- country
	- Unemployment rate
	- Internet access %
	- Urban population %

- Variables that can be included using Eurostat API:
	- pollution level
	- HousePrice-to-Income ratio
	- mobile internet usage
	- population



# Workflow description

- [Data part — @Saad]

- An AWS account was created for extracting, wrangling, storing, manipulating and serving the datasets that are used in the project.

- Data Files for World Happiness Report at a yearly level were extracted from the Kaggle data source mentioned above. It contained 9 csv files (one for each year) with happiness scores along with other variables on a country level. 

- These 9 CSV files were stored in AWS S3 bucket. The source of this file as mentioned above is:
https://www.kaggle.com/datasets/sazidthe1/global-happiness-scores-and-factors

- These yearly files would be downloaded and converted into a single dataframe (discussed below). 

- A notebook instance was created within Amazon Sagemaker for using Jupyter Notebook instance that connects to S3 for data extraction and wrangling.
 
- To allow Sagemaker to access files from S3, an IAM user policy was created that granted list, read and write access to Sagemaker on the S3 bucket.
 
- Once Sagemaker was able to access the csv files with S3, we used Jupyter Notebooks to access files in S3 and wrangled them using python. 
- Wrangling steps:
	o	Separate yearly csv combined together in a single dataset
	o	Year column extracted from csv name and appended as a column
	o	Data filtered for just the European countries
	o	European countries with all 9 years’ worth of data retained and rest filtered


- To store this wrangled data, we created an RDS within AWS to store the MYSQL Database that would host this final dataset that we want to access in KNIME. 

- The RDS was set to be publicly accessible so all group members could easily access the data at their end. 

- A security group was created using EC2 instance to manage inbound and outbound rules for RDS. It allowed access to the Jupyter Notebooks (within Sagemaker) to create a MYSQL database with in RDS, that would host our data. 

- Once all the permissions were sorted, we pushed the final dataset from the Sagemaker to the RDS using MySQL connector and a new table containing data we want to access in Knime was created. 

- The final RDS data frame contains the following variables at a country and year level:
	- happiness score
	- gdp_per_capita
	- social_support
	- healthy_life_expectancy
	- freedom_to_make_life_choices
	- generosity
	- perceptions_of_corruption


- For the stand alone csv file containing Unemployment rate, Internet access %, Urban population % we used the world bank website https://databank.worldbank.org/.

- Data for European countries which are present in our RDS dataset was extracted from 2015 till 2023 at a country and yearly level. 

- This standalone csv file would be another input into Knime data model. 

- It would be joined to our RDS dataframe for enriching the data. 

- File 'data2.csv' is stored in the data folder that is located in the project folder and is referenced locally;

- String replacer node is used to correct the country names, so that they look the same in both tables in order to properly join them (Slovak Repiblic -> Slovakia, Russian Federation -> Russia amendments are made in the data2);

- Then, an HTML query from a wikipedia webpage ( https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes ), and a Python script that processes the response is used to get a table of country names and 2-letter country codes in ISO 3166-1 A-2 format, as needed for Eurostat API;

- The three data sources are merged by year by country, so that all the data appears in one table;

- Then, four Eurostat APIs are collected and integrated into data: pollution level, HousePrice-to-Income ratio, mobile internet usage, and population. The API requests are generated by string manipulation with year and country code. Data sources:
	- https://doi.org/10.2908/ILC_MDDW02
	- https://ec.europa.eu/eurostat/databrowser/view/tipsho60
	- https://ec.europa.eu/eurostat/databrowser/bookmark/c815e093-d12b-4bd6-b2dc-dbcd3e970bba?lang=en
	- https://doi.org/10.2908/TPS00001

- The mobile internet usage is normalised by population;
