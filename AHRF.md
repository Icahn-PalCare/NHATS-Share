# AHRF Data Setup

```
clear all
capture log close
log using ahrf.smcl, replace
set maxvar 7000
use "C:\AHRF\ahrf.dta" 
destring *, replace

foreach x in "66" "72" "78" {
drop if f00011==`x'
}

foreach x in _all {
capture drop f*****00
capture drop f*****01
capture drop f*****03
capture drop f*****05
capture drop f*****06
capture drop f*****07
capture drop f*****08
capture drop f*****09
capture drop f*****16
capture drop f*****17
}


qui compress

forvalues i=10/15 {
rename f13214`i' hha`i'
}

rename f1484010 elderly
rename f00011 state
rename f00012 county_num
rename f00010 county_name_ahrf
rename f04530 tot_pop_cnty_10

capture drop f*****

save "C:\AHRF\ahrf_cleaned.dta", replace

```

# Merge to Georgraphic NHATS

```
capture log close
cd "C:\NHATS Geo\logs"
log using main.smcl, replace
set more off

clear all

forvalues w=1/6 {
use "C:\NHATS Geo\NHATS_Restricted_GEO_Files\Round`w'\SP_STATA\NHATS_Round_`w'_SP_GeoRestricted.dta"
gen wave=`w'
save "C:\NHATS Geo\NHATS_Restricted_GEO_Files\Round`w'\SP_STATA\geo_rnd`w'_res.dta", replace
clear all
}                                          						 
clear all
use "C:\NHATS Geo\sp_round_1_6_public_sens_only.dta"


forvalues i=1/6{
replace ivw_year=201`i' if wave==`i'
}
//use "C:\NHATS Geo\NHATS Files GEO1_4_HRR1_2-Ornstein,Katherine\NHATS Files GEO1_4_HRR1_2-Ornstein,Katherine\NHATS_Restricted_GEO_Files\Round1\SP_STATA\geo_rnd1_res.dta"


forvalues w=1/6 {
merge 1:1 spid wave using "C:\NHATS Geo\NHATS_Restricted_GEO_Files\Round`w'\SP_STATA\geo_rnd`w'_res.dta"
rename _merge merge`w'
tab merge`w'
}

//merge 1:1 spid wave using "C:\NHATS Geo\NHATS Files GEO1_4_HRR1_2-Ornstein,Katherine\NHATS Files GEO1_4_HRR1_2-Ornstein,Katherine\NHATS_Restricted_GEO_Files\Round1\SP_STATA\geo_rnd1_res.dta"

forvalues w=1/6 {
qui tab re`w'spadrsstt, sort
qui table re`w'spadrsstt homebound_cat
}
keep if nhres==0

capture drop lastw 
capture drop firsthb
capture drop hbw
capture drop everhb
//last wave the person was reported
by spid, sort: egen lastw=max(wave)
//wave the person was homebound(never/rarely leaves)
gen hbw=wave if homebound==1
//first wave the person reported being homebound(never/rarely)
by spid, sort: egen firsthb=min(hbw)
//If the person was ever homebound (never/rarely), if not then missing. 
gen everhb=!missing(firsthb)

keep if wave==1

gen hb=homebound
replace homebound=homebound==1 if !missing(homebound)
gsort spid -wave
by spid, sort:gen unique5=_n==1
tab unique5

gen sr_cond_cat=1 if sr_numcond<2
replace sr_cond_cat=2 if inrange(sr_numcond,2,4)
replace sr_cond_cat=3 if inrange(sr_numcond,5,15)
label define sr_cond_cat 1 "0-1 SR Conditions" 2 "2-4 SR Conditions" 3 "5+ SR Conditions"
label values sr_cond_cat sr_cond_cat

gen num_helpers_cat1=num_helpers_cat
replace num_helpers_cat1=2 if num_helpers_cat==3
label define num_helpers_cat1 0 "No Helpers" 1 "1-3 Helpers" 2 "4+ Helpers"
label values num_helpers_cat1 num_helpers_cat1

capture drop fips
capture gen fips=.
forvalues i=1/4{
replace fips=re`i'spfpsstcy if re`i'spfpsstcy!=.
}
//attaching RUCC codes --Urban Matches well
//22856
merge m:1 fips using "C:\urban\rucc.dta"
keep if _merge==3
rename _merge merge5
//22855
tab rucc
tab metro_ind
tab rucc metro_ind
/*
keep if wave==1 & merge1==3
*/
//only walkability for round 1
capture drop fips_tr
gen fips_tr=re1censtract
forvalues i=2/4{
replace fips_tr=re`i'censtract if fips_tr==""
}

qui capture destring fips_tr, replace
merge m:1 fips_tr using "C:\Walkability\walkind.dta"
keep if _merge==3

forvalues i=1/4 {
sum tot_walk if tot_walk_four==`i'
}
tab tot_walk_four homebound, row

tab metro_ind tot_walk_four

capture drop tot_walk_4


//Quantiles in terms of rounds (in this case round 1)

xtile tot_walk_4= tot_walk, nq(4)

forvalues i=1/4 {
sum tot_walk if tot_walk_4==`i'
}
*
foreach x in homebound reg_doc_homevisit howgetdoc metro_ind rucc{
tab `x' tot_walk_4, row
tab `x' tot_walk_4, column
}
**
forvalues i=1/9{
drop state`i'
}
rename state state_name
rename sfips1 state
rename fips_tr geoid10
rename _merge merge6

//ses 
merge m:1 geoid10 using "C:\census\ses\SES_census_tract_clean.dta"
keep if _merge==3

forvalues i=1/4 {
sum nses if nses_four==`i'
}
tab nses_four homebound, row

tab metro_ind tot_walk_four


//Quantiles in terms of the rounds


xtile nses_4= nses, nq(4)


forvalues i=1/4 {
sum nses if nses_4==`i'
}
tab nses_4 homebound, row

tab metro_ind nses_4

 
**
//rename _merge merge7
//merge m:1 STATE using "C:\LTC\state_abbrev.dta"
rename county county_num
rename county_name county
gen year=ivw_year
capture drop _merge


//LTC Focus by county already cleaned up
//use "C:\LTC\county_2011.dta"
//append using "C:\LTC\county_2012.dta" "C:\LTC\county_2013.dta" "C:\LTC\county_2014.dta"

merge m:1 year state_name county_num using "C:\LTC\LTC_11_14.dta"

drop if _merge==2
//25659
by state county, sort: gen unique=_n==1
//number of unique counties
tab unique


sort homebound
by homebound: sum totbeds_cty 

tab income_cat homebound, row
tab income_cat homebound, column
by homebound: sum aveincome
by homebound: sum acuindex2_cty
by homebound: sum alzunit_cty
by homebound: sum pctnhdayssnf_cty
by homebound: sum hospptyr_cty

recode cohesion_knowwell cohesion_peop cohesion_willing (3=0) (2=1) (1=2), gen(co_know co_peop co_will) 
label define cohesion 0 "Do Not Agree" 1 "Agree a Little" 2 "Agree a Lot"
label values co_know cohesion
label values co_peop cohesion
label values co_will cohesion

capture drop cohesion
gen cohesion=(cohesion_k+ cohesion_p+cohesion_w==3) if !missing(cohesion_k) | ///
!missing(cohesion_p) | !missing(cohesion_w)

sort ivw_year
by ivw_year: logit homebound i.nses_4 i.tot_walk_4 hospptyr_cty
margins, dydx(*)


sort rucc
by rucc: sum nses tot_walk if homebound==1 & wave==1

//Dartmouth 
rename _merge merge8
merge m:1 id using "C:\Dartmouth\PCSA.dta"
keep if _merge==3
rename _merge merge9

tempfile full
save "`full'"

forvalues i=1/2 {
use "C:\NHATS Geo\NHATS_Restricted_HRR_Files\Round`i'\NHATS_Round_`i'_HRR_xref.dta", clear
gen ivw_year=201`i'
tempfile hrr1`i'
save "`hrr1`i''"
}

use "`full'", clear
merge 1:1 spid ivw_year using "`hrr11'"
rename _merge merge10
drop if merge10==2
//25716


merge 1:1 spid ivw_year using "`hrr12'"
rename _merge merge11
drop if merge11==2

capture drop hrrnum
gen hrrnum=.
replace hrrnum=hrrnum_2011
replace hrrnum=hrrnum_2012 if hrrnum==.  

tempfile hrr_full
save "`hrr_full'"

use "C:\Dartmouth\2011_2014_DAP.dta", clear
drop if year==2013 | year==2014
rename year ivw_year

tempfile dart
save "`dart'"

use "`hrr_full'", clear

merge m:1 hrrnum ivw_year using "`dart'"
rename _merge merge12
drop if merge12==2

merge m:1 state county_num using "C:\AHRF\ahrf_cleaned.dta"
drop if _merge==2
capture drop hha
gen hha=.
forvalues i=10/15{
capture replace hha=hha`i' if year==20`i'
}
tab hb
//formula from LTC Focus
capture drop hha_elderly
gen hha_elderly=(hha/elderly)*1000
sum hha_elderly, d

```
