# Debt Specific Risk Default Modeling

In the debt specific risk (DSR) model, we applied a conservative treatment to the default events (for CDS positions only) in the following way. For long positions (i.e., sold protections) the loss is defined as usual (i.e., LGD´Notional), while for short positions we give conservative values when default occurs. For short positions (i.e., bought protections), we restrict positive P/L due to defaults. This fully conservative treatment is being applied to CDS feed, while all other feeds are treated conservatively on the un-hedged portion only. 

The full conservative treatment has proved to be overly conservative, manifesting itself in unreasonably large economic capital VaR numbers. These large economic capital VaR numbers are mainly generated from default scenarios of CDS positions, due to the conservative treatment of defaults, regardless of whether or not the default risk is being properly hedged. They are not generated from idiosyncratic spread risk (P/L due to spread changes), which is being appropriately captured. 

The solution is to recognize the benefit from taking short positions in CDS (i.e., bought protections) to hedge long CDS positions (i.e., sold protections) with the same underlying name and apply the conservative approach to the un-hedged portion as for all other feeds.

DSR analytic engines use various database tables as inputs, one of which is the “Portfolio” table under the “DSR” database. This table contains the list of all positions with their corresponding positional data required by the engines and it is used as the input portfolio for the DSR calculation on any given date. 

Currently CDS positions appear individually under this table. This means that long and short CDS positions of the same underlying name (issuer) with identical default risk characteristics appear in two separate entries, which are considered as two separate positions by the DSR engine. If a default event is generated in a scenario for the issuer, the bought protection (short CDS position) should offset the loss from the sold protection (long CDS position). 

However, the DSR engine does not make any assumption on whether the long and short CDS positions have the same underlying issuer with identical default risk profile. Instead, it applies a uniform conservative treatment for both long and short CDS positions. This has led to little benefit from taking the short positions to hedge the long positions, and, as a result, it has produced overly conservative VaR numbers.

To appropriately recognize the benefit of the hedges, we use the aggregated notional of those long and short CDS positions with the same underlying issuer to calculate the losses (P/Ls) in the default scenarios. No change is required in non-default scenarios in which P/Ls due to spread changes are calculated as it is. By applying the aggregated notional of these CDS positions, the hedges are correctly recognized in the default scenarios while the conservative approach is still applied to the remaining part of the un-hedged positions. 

The aggregated notional is calculated by netting the notionals (TotalAmount) of the CDS positions that must have the same issuer name (IssuerID), InstrumentType (i.e., CDS), RiskFactorName, CreditGroup, RelatedBondIndex, Currency, RiskRating, LGDRating, and AggregationLevel. By aggregating same currency CDS positions only, we ensure that we do not aggregate across different instrument types and currencies. 

Debt specific risk is contained in bond and CDS. You can find different types of bonds at https://finpricing.com/lib/FiBondCoupon.html

By netting notionals of CDSs for the same issuer with the same risk factor name, credit group and index, we ask that idiosyncratic risk factors be not affected. By further requiring risk rating and LGD rating to be the same, we impose that only the CDS positions with identical default risk (default probability and recovery rate) be aggregated. Finally, by including aggregation level as an aggregation criterion, we suggest that hedging benefit be only recognized within the same business hierarchy (desk).
