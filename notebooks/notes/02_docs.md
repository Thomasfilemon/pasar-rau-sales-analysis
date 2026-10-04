Notebook 01 asked:
What problems exist?

Notebook 02 asks:
What transformations should we apply so downstream analysis is consistent, reproducible, and defensible?

Your workflow should be:


RAW
 

↓


Parse


 ↓


Standardize
 

↓


Validate
 

↓


Flag questionable records
 

↓


Create analytical fields
 

↓


Save processed dataset



The important word there is flag.


We're not going around deleting every row that looks funny like some trigger-happy intern with `dropna()`.