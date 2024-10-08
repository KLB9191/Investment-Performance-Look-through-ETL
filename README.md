**Project:**
 
The Investment-Performance-Look-through-ETL is a python script that will automate the routine ETL of the Affiliate_Borrower_Master, Affiliate_IRR_Report, and Affiliate_IRR_Cashflow tables in the Enterprise Datawarehouse. The script will extract the affiliate loader file in the designated file path and perform numerous tasks, which includes: 

**Lines 48 & 857**

These are the file paths for where the script will extract the affiliate loader file. Please update this to the correct folder depending on the reporting year and period.

**Lines 143-236**

Appending any new borrowers from the ‘New Borrowers’ tab in the loader file to the existing Affiliate Borrower Master table. The appended table is then split into two dataframes, one (df1) which contains only the borrowers where the Affiliate_Borrower_ID is not null/blank, and the other (df2) which contains only the borrowers where the Affiliate_Borrower_ID is null/blank. After unique IDs are generated for df2 via a UUID.transform lambda function, df2 is appended back to df1, resulting in the preliminary Affiliate_Borrower_Master df.

**Lines 237-336**

Performing additional data wrangling for the preliminary Affiliate_Borrower_Master df, which includes pulling any and all realized deal realization dates from the ‘Realizations’ tab in the affiliate loader to replace their previous placeholder Realization_Date values of ‘1899-12-30’. There are also joins, transformations, and checks performed for the ‘Country’ and ‘Restructure_ID’ fields in the Affiliate_Borrower_Master df to ensure data accuracy/integrity. Once these tasks and additional data transformation logic are completed, the final Affiliate_Borrower_Master df, named ‘Affiliate_Borrower_Master_Final_v2’, is loaded to the EnterpriseDataWarehouse via a ‘to_sql’ command on line 331.

**Lines 337-486**

Performing IRR calculations using the FMV, flows, and dates data from the ‘Pricing’ and ‘Cashflows’ tab from the loader file. Any current quarter pricing that has already been loaded to the database is pulled and subsequently appended to the pricing data contained in the affiliate loader (Pricing tab). Next, a date column containing the same period end date for all rows is added to the combined Pricing df (ex: 3/31/2024). Historical flows are also pulled from the database and subsequently appended to the flows contained in the affiliate loader (Cashflows tab). Next, the final affiliate borrower master df is filtered for WOGA_Reporting = ‘Yes’ and Asset_Type != ‘FACTORING’ and assigned to a new df named ‘Affiliate_Borrower_Base’. This table, which sets the scope for what affiliate_code_names are included in the IRR calc df, is then merged with the combined pricing df and is assigned to the object ‘Pricing_for_IRR2’ on line 455. Finally, the ‘Pricing_for_IRR2’ df and the ‘Combined_Flows’ df are appended to each other, assigned as ‘Pricing_Flows_Combined’. This is the table that is eventually used to perform IRR calculations for each Affiliate_Code_Name.

**Lines 487-577**

Appending all loader file current quarter flows to all flows in the edw.affiliate_IRR_Cashflow_Report. This table is then split into two dataframes, one (df1) with contains only the flows where cashflow_ID column is not null/blank, and the other (df2) which contains only the flows where cashflow_ID column is null/blank. After unique cashflow_IDs are generated for df2, df2 is appended back to df1 to create the new Affiliate_IRR_Cashflow_Report. Finally, any cashflow_IDs in the Affiliate_IRR_Cashflow_Report listed in the ‘delete_test’ tab in the loader file are deleted from the Affiliate_IRR_Cashflow_Report to create the final dataframe for the edw.Affiliate_IRR_Cashflow_Report. And ods.Affiliate_IRR_Cashflow_Report.

**Lines 578-768**

Calculating IRRs at three different levels- each Affiliate_Code_Name, each Affiliate, and Total Affiliate. These correspond with Pricing_df1, Pricing_df2, and Pricing_df3, respectively. As mentioned under the ‘Lines 337-486’ section in the previous page, the affiliate borrower master filter which sets the scope for what affiliate_code_names are included in all three IRR calculations are: WOGA_Reporting = ‘Yes’ and Asset_Type != ‘FACTORING’. After the calculations, the IRR values are then joined back to their respective Pricing dataframes. Additional columns are then added either manually or through another join to the affiliate borrower master table. 

**Lines 769-844**

The loader file has a ‘Pricing’ tab, which includes current quarter Fair Market Values (FMV) for Affiliate investments. The main purposes of this tab are to provide Ending Market Value books for the Affiliate IRR Report, as well as to provide terminal values for IRR calculations. An IRR will be calculated for each Affiliate and each Affiliate-borrower combination in the Affiliate Borrower Master table where WOGA_Reporting = ‘Yes’ and asset_type <> ‘Factoring’. An IRR will also be calculated for all affiliates combined. Finally, we also create a separate factoring IRR df, which assigns an IRR value of ‘NA’ for all deals from the Affiliate Borrower Master table where asset_type = ‘Factoring’ and WOGA_Reporting = ‘Yes’. The combination of all these dataframes produces the final Affiliate_IRR_Report.
All three final dataframes, the Affiliate_Borrower_Master, Affiliate_IRR_Cashflow_Report, and Affiliate_IRR_Report are loaded directly to the EnterpriseDatawarehouse via ‘to_sql’ commands in the Python script. The Affiliate_IRR_Cashflow_Report and Affiliate_IRR_Report tables are loaded to their respective ‘edw’ and ‘ods’ schemas in the database. The Affiliate_Borrower_Master is only loaded to the ‘edw’ schema. For the ‘edw’ direct loads, the Python command replaces the existing table with the current day’s version. For the ‘ods’ version, the Python command append the current day’s tables to the existing tables.

**Lines 845 - 851 and 1027-1033**

Finally, the script logic contains automated email notifications triggered by any exceptions raised during execution of the script. The emails are sent from helpdesk@xx.com to both xx@xx.com and xx@wxx.com.

