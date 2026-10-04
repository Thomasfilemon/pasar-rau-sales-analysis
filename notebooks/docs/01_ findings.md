1. Dataset contains 10,678 valid line-item records covering Jan-Dec 2021.

2. Transaction dates are complete but require explicit day-first parsing.

3. Outlet identifiers are preserved in the raw source, although
   outlet-to-name mappings require validation.

4. Invoice identifiers are partially corrupted due to scientific
   notation and should not be used blindly for transaction counts.

5. Financial columns use inconsistent number formatting and require
   locale-aware parsing.

6. Net sales reconcile closely with GSV + Discount + Tax.

7. Negative quantities and financial values indicate return/reversal
   activity.

8. Salesperson codes appear to be reassigned between employees.

9. Duplicate records exist and require investigation before removal.