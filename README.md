magna_dap/ my_analysis.py #This is the main shim optimization model -> Shim_Model

magna_dap/ MPT_SHIM_MODEL_TABLES_CUM.py #This is the main DAP Pipeline Model. Instantiates Pipeline Class and Shim_Model Class from configs

magna_dap_client/ MPT_CLIENT_SQL.py #This instantiates the MagnaDAP Client Class from configs

magna_dap_client/ mpt_read_sql_fn.py #Two helper functions to read in new data from SQL Server and processes data to long format

configs / #Two configs. One for Shim_Model config and other for DAP Pipeline/Client config

data_files / #Input data_files provided to instantiate Shim_Model Class. data_files / fi_pinion_5.csv #Input file for top features for Pinion. This is obsolete now that we are not using rolling regression but fixing the betas according to predefined table. But is currently used when removing data outliers in backtesting.

TO RUN:

pip install -r requirements.txt

Run Pipeline: python magna_dap/MPT_SHIM_MODEL_TABLES_CUM.py

Call Client (send new data for offset prediction): python magna_dap_client/MPT_CLIENT_SQL.py

To get offset predictions, the new data df only requires two features. '6935' -> Pinion, '6967' -> Measured Shim 2. The betas of Pinion wrt Shim size are stored in look-up table (keys are levels of '6967'). If '6967' is not provided, the model will choose default_beta. If '6935' is missing then no optimization is possible and a default value of 0 is returned for optimal shim offset. Likewise, if the number of valid rows of new data (after range_check) is less than the forecast window, then no optimization is performed and optimal offset is set to 0.
Depending on how you want the pipeline to store new data sent from the client and new offset predictions, you can also pass in 'ids' and 'Prod_Date' from the client.

## UPDATE : Oct 11

added magna_dap / MPT_SHIM_MODEL_TABLES_CUM.py # Pipeline Class with cumulative shim

Using cumulative shim requires that there exist an SQL table which stores cumulative shim offsets. Pipeline_CUM reads the latest shim offset from the offset table and the new offset adjustment is added to the running cumulative sum and saved back to the SQL table. I have implemented locally in MySQL but this will have to implemented in VM environment and the correct database connection details stored in 'mysql' variable in test_config.yaml

CREATE TABLE offsets (TS Timestamp, Offset Decimal(5,3));

INSERT INTO offsets VALUES ("2022-10-10", 0);

## UPDATE : Oct 12

It is far cleaner to have a separate table that stores offsets (see above). 

The Shim 2 Offset ('10780') does not appear in the main results table. If this has to be calculated from '6956', '6967', '2129' (and not read separately) then all these features must also be passed by the Client in addition to '6935' and '6967'. Otherwise, the database will have to be queried twice. The current code makes this calculation. But if some features are missing and current offset cannot be calculated, then the model returns NULLs and it is not the most optimal solution. Hopefully, Clemens & Co can advise us how we can query for '10780' directly to avoid messy code and messier outcomes.

## On second thought, perhaps this is not a big issue as long as the features the Client sends for evaluation are fixed in the code.

## UPDATE : Oct 17

Modified the Shim_Model so that when the last shim offset is, for example, at the upper boundary, then any positive additional offset adjustments are disallowed 
and vica versa for the lower boundary. That way, the shim offset is kept within certain boundaries. 

Updated the MPT model config files

## UPDATE : Oct 18

Added last week's data (Oct 10 - Oct 17) to /data_files and in configs/test_config.yaml 

Will update regularly on weekly basis.


