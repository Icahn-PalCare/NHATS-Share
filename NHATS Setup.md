# NHATS Setup - 01/07/2021

## Stata code to clean and create variables fro Rounds 1-9 of NHATS data. 

## Heading

```
/* 
********************HEADING******************** 

Project Name: NHATS Setup

Date Started: 

Primary Investigator:
Funding Source:

Created by: Many Authors

Primary Analyst:
Secondary Analyst:

Datasets Used: NHATS Raw public, sensitive, metro, and NSOC

Simple Outline: NHATS data cleanup, variable creation. 


*/
 
//STATA
// Global Macros use $ symbol to be called. 
//File paths for raw data

//NEED TO CHANGE WAVE NUMBER FOR EACH WAVE
global w 9

global work D:\NHATS\Shared\base_data\NHATS cleaned\working
global logs D:\NHATS\Shared\base_data\NHATS cleaned\logs

forvalues w=1/$w{
global r`w'raw D:\NHATS\Shared\raw\NHATS\NHATS Public\round_`w'\
global r`w's D:\NHATS\Shared\raw\NHATS\NHATS Sensitive\r`w'_sensitive\
}

foreach w in 1 5 7{
global r`w'nsoc D:\NHATS\Shared\raw\NHATS\NHATS Sensitive\r`w'_sensitive
}

global nsoc_wave 7

cd "${work}"
```

## NHATS Data Cleaning Part 1 (1_combine_waves)
```
H="NHATS Data Cleaning Part 1 (1_combine_waves)"
/*

Created by: --
Date Created: --

Updated by: MH
Date Updated: 12/17/2019

Description: Inital dataset cleaning, renaming varaibles, merging variables from 
sensitive dataset, and broad structuring.



**************************************************
*/

local date = subinstr("$S_DATE"," ","_",.) 
local name 1-combine_waves_`date'
di "`name'"

capture log close 
clear all

set more off
version 12
set linesize 80
set maxvar 10000


log using "${logs}/`name'.smcl", text replace

cd "${work}"

/*Combines rounds 1 and 2 sample person SP interview files,
sensitive demo files and the round 2 cumulative tracker file case 
status l into a single file

Data format is multiple observations per subject, one for each round
*/
*********************************************

forvalues w=1/$w {
//round w
use "${r`w'raw}NHATS_Round_`w'_SP_File.dta"
//check to make sure sample ids are unique
sort spid 
quietly by spid: gen dup = cond(_N==1,0,_n)
tab dup
capture gen wave=`w'
la var wave "Survey wave"
save round_`w'_1.dta, replace
clear
}

//need to have each wave sepeartely done becuase not all waves have same variables to keep. 
//round 1
forvalues w=1/$w{
use round_`w'_1.dta

if `w'!=1 local pd pd* 

//keep selected variables only
local keepallwaves spid wave r`w'dresid w`w'varunit w`w'anfinwgt0 w`w'varstrat  ///
	mo* r`w'd2intvrage hh`w'martlstat ///
	ip`w'cmedicaid ip`w'mgapmedsp ip`w'nginsnurs ip`w'covmedcad ip`w'covtricar ///
	hh* hc* ss* pc* cp* cg* ha* sc* mc* sd* pa* hw* ///
	is`w'* ht`w'placedesc fl`w'* ir* cm* ew* hp* sn* dt* `pd' gr* wa* r`w'dorigwksc ///
	r`w'dnhatswksc r`w'dnhatsgrav r`w'dnhatsgrb wb* ho* cs*  

if `w'==1 {	
keep `keepallwaves' r`w'dgender rl`w'dracehisp rl`w'spkothlan rl`w'condspanh el`w'higstschl ///
	ia`w'*  re`w'resistrct re1dcensdiv va1serarmfor
	
rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* rl`w'* el`w'* ia`w'* re`w'* va`w'*) ///
(*) 
destring dfavact, replace
}

if `w'==2 {	
keep `keepallwaves' re2intplace re2newstrct re2spadrsnew re2dresistrct ///
	re2dadrscorr re2dcensdiv ip2nginslast ep2eoltalk ep2poweratty ep2livngwill lm*
	
rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* re`w'* pd`w'* ep`w'* lm`w'* ) ///
(*) 
}

if `w'==3 {	
keep `keepallwaves' re3intplace re3newstrct re3spadrsnew re3dresistrct ///
	re3dcensdiv ip3nginslast ia* lm*
	
rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* re`w'* ia`w'* pd`w'* lm`w'* ) ///
(*) 
}
if `w'==4 {	
keep `keepallwaves' re4intplace re4newstrct re4spadrsnew re4dresistrct ///
	re4dcensdiv ip4nginslast lm*
	
rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* re`w'* pd`w'* lm`w'*) ///
(*) 
}
if `w'==5 {	
keep `keepallwaves' r`w'dgender rl`w'dracehisp rl`w'spkothlan rl`w'condspanh el`w'higstschl re5intplace re5newstrct re5spadrsnew re5dresistrct ///
	re5dcensdiv ip5nginslast ia`w'* va5serarmfor w5an2011wgt0 lm* rh`w'rehab
	
rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* rl`w'* el`w'* ia`w'* re`w'* va`w'* pd`w'* lm`w'* rh`w'*) ///
(*) 
}	
if `w'==6 {	
keep `keepallwaves' re6intplace re6newstrct re6spadrsnew re6dresistrct ///
	re6dcensdiv ip6nginslast w6an2011wgt0 lm* rh`w'rehab 

rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* re`w'* pd`w'* lm`w'* rh`w'*) ///
(*) 
}
if `w'==7 {	
keep `keepallwaves' re7intplace re7newstrct re7spadrsnew re7dresistrct ///
	re7dcensdiv ip7nginslast ia`w'* w7an2011wgt0 lm* rh`w'rehab
	
rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* re`w'* ia`w'* pd`w'* lm`w'* rh`w'*) ///
(*) 
}
if `w'==8 {	
keep `keepallwaves' re8intplace re8newstrct re8spadrsnew re8dresistrct ///
	re8dcensdiv ip8nginslast w8an2011wgt0 lm* ep`w'* rh`w'rehab
	
rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* re`w'* pd`w'* lm`w'* rh`w'*) ///
(*) 
rename (ep8eoltalk ep8poweratty ep8livngwill) (eoltalk poweratty livngwill)
}
save round_`w'_ltd.dta, replace


if `w'==9 {	
keep `keepallwaves' re9intplace re9newstrct re9spadrsnew re9dresistrct ///
	re9dcensdiv ip9nginslast w9an2011wgt0 lm* rh`w'rehab
	
rename (r`w'* w`w'* mo`w'* hh`w'* ip`w'* hc`w'* ss`w'* pc`w'* cp`w'* cg`w'* ha`w'* ///
sc`w'* mc`w'* sd`w'* pa`w'* hw`w'* is`w'* ht`w'* fl`w'* ir`w'* cm`w'* ew`w'* hp`w'* ///
sn`w'* dt`w'* gr`w'* wa`w'* wb`w'* ho`w'* cs`w'* re`w'* pd`w'* lm`w'* rh`w'*) ///
(*) 

}
save round_`w'_ltd.dta, replace
}

// 6/26/2020-commenting this out until we have r9 sens file
/*
//check sensitive data files, keep only some variables, merge with ltd datasets
forvalues w=1/$w{
	use "${r`w's}NHATS_Round_`w'_SP_Sen_Dem_File.dta"
	sort spid 
	quietly by spid: gen dup = cond(_N==1,0,_n)
	tab dup	
	clear
}
*/
//combine waves into single dataset
use round_1_ltd.dta
forvalues w=2/$w{
append using round_`w'_ltd.dta
}

preserve


forvalues w=1/$w{
use "${r`w's}NHATS_Round_`w'_SP_Sen_Dem_File.dta", clear
gen wave=`w'
save "${r`w's}NHATS_Round_`w'_SP_Sen_Dem_File_new.dta", replace

}

restore

//merge in sensitive data, use r1 as basis, added in cancer vars
merge m:1 spid wave using "${r1s}NHATS_Round_1_SP_Sen_Dem_File_new.dta", ///
	keepusing(r1dbirthmth r1dbirthyr r1dintvwrage hh1modob hh1yrdob hh1dspousage ///
	rl1primarace rl1hisplatno hc1cancerty*) nogen

//merge in additional r2 sensitive data
merge m:1 spid using "${r2s}NHATS_Round_2_SP_Sen_Dem_File_new.dta", ///
	keepusing(r2dintvwrage hh2dspousage r2ddeathage pd2mthdied pd2yrdied) nogen

//merge in additional r3 sensitive data
merge m:1 spid using "${r3s}NHATS_Round_3_SP_Sen_Dem_File_new.dta", ///
	keepusing(r3dintvwrage hh3dspousage r3ddeathage pd3mthdied pd3yrdied)  nogen

//merge in additional r4 sensitive data
merge m:1 spid using "${r4s}NHATS_Round_4_SP_Sen_Dem_File_new.dta", ///
	keepusing(r4dintvwrage hh4dspousage r4ddeathage pd4mthdied pd4yrdied hc4cancerty*) nogen

//merge in additional r5 sensitive data
merge m:1 spid using "${r5s}NHATS_Round_5_SP_Sen_Dem_File_new.dta", ///
	keepusing(r5dintvwrage hh5spageall r5ddeathage pd5mthdied pd5yrdied hc5cancerty*) nogen

//merge in additional r6 sensitive data
merge m:1 spid using "${r6s}NHATS_Round_6_SP_Sen_Dem_File_new.dta", ///
	keepusing(r6dintvwrage hh6spageall r6ddeathage pd6mthdied pd6yrdied hc6cancerty*) nogen

//merge in additional r7 sensitive data
merge m:1 spid using "${r7s}NHATS_Round_7_SP_Sen_Dem_File_new.dta", ///
	keepusing(r7dintvwrage hh7spageall r7ddeathage pd7mthdied pd7yrdied hc7cancerty*) nogen
	
//merge in additional r8 sensitive data
merge m:1 spid using "${r8s}NHATS_Round_8_SP_Sen_Dem_File_new.dta", ///
	keepusing(r8dintvwrage hh8spageall r8ddeathage pd8mthdied pd8yrdied hc8cancerty*) nogen

//merge in additional r9 sensitive data
merge m:1 spid using "${r9s}NHATS_Round_9_SP_Sen_Dem_File_new.dta", ///
	keepusing(r9dintvwrage hh9spageall r9ddeathage pd9mthdied pd9yrdied hc9cancerty*) nogen

//3B?
//merge in tracker status information


merge m:1 spid using "${r${w}raw}NHATS_Round_${w}_Tracker_File", keepusing( yearsample ///
w9trfinwgt0 w9tr2011wgt0 r9status r9spstat r9spstatdtmt r9spstatdtyr r9fqstatdtmt ///
w8trfinwgt0 w8tr2011wgt0 r8status r8spstat r8spstatdtmt r8spstatdtyr r8fqstatdtmt ///
w7trfinwgt0 w7tr2011wgt0 r7status r7spstat r7spstatdtmt r7spstatdtyr r7fqstatdtmt ///
w6trfinwgt0 w6tr2011wgt0 r6status r6spstat r6spstatdtmt r6spstatdtyr r6fqstatdtmt ///
w5trfinwgt0 w5tr2011wgt0 r5status r5spstat r5spstatdtmt r5spstatdtyr r5fqstatdtmt ///
w4trfinwgt0 			 r4status r4spstat r4spstatdtmt r4spstatdtyr r4fqstatdtmt ///
w3trfinwgt0 			 r3status r3spstat r3spstatdtmt r3spstatdtyr r3fqstatdtmt ///
w2trfinwgt0 			 r2status r2spstat r2spstatdtmt r2spstatdtyr r2fqstatdtmt ///
w1trfinwgt0 			 r1status r1spstat r1spstatdtmt r1spstatdtyr r1fqstatdt*)

gen trfinwgt0=.
gen tr2011wgt0=.
forvalues w=1/$w{
replace trfinwgt0= w`w'trfinwgt0 if wave==`w'
capture replace tr2011wgt0 = w`w'tr2011wgt0 if wave==`w'
capture drop w`w'trfinwgt0 w`w'tr2011wgt0
}


//merge in nsoc tracker information
merge m:1 spid using "${r1s}NSOC_Round_1_SP_tracker_file", nogen
//drop obs that are in tracker file but not sp file
drop if _merge==2
drop _merge	
save round_1_to_$w.dta, replace

//old version of stata
forvalues w=1/$w{
clear all
use "D:\NHATS\Shared\raw\NHATS\NHATS Public\round_`w'\NHATS_Round_`w'_SP_File.dta"
saveold "D:\NHATS\Shared\raw\NHATS\NHATS Public\round_`w'\NHATS_Round_`w'_SP_File_stata12.dta", replace version(12)
}

*********************************************
log close

translate "${logs}/`name'.smcl" "${logs}/`name'.pdf"
exit
```

## NHATS Data Cleaning Part 2 (1a_help_hours_imputation)

```
H="NHATS Data Cleaning Part 2 (1a_help_hours_imputation)"
/*

Created by: --
Date Created: --

Updated by: MH
Date Updated: 07/02/2019

Description: This section uses the SP and OP data files and imputes
hours of help provided to SP per methods in NHATS Technical Paper #7

Variables from this dataset are then brought in to the SP Rounds dataset
and also the NSOC caregiver dataset.



**************************************************
*/

local date = subinstr("$S_DATE"," ","_",.) 
local name 1a-help_hours_imputation_`date'
di "`name'"

capture log close 
clear all

set more off
version 12
set linesize 80
set maxvar 10000


log using "${logs}/`name'.smcl", text replace




*********************************************

cd "${work}"

**User note: This section can be commented out after run one time to set up datasets
********************************************************************
//combine op and sp data files

forvalues w=1/2{
use "${r`w'raw}NHATS_Round_`w'_OP_File_v2.dta", clear
sort spid opid 

merge m:1 spid using round_`w'_1.dta //bring in sorted sp ivw dataset
keep if _merge==3 //drop obs with no OP entries
save R`w'_OPSPlinked.dta, replace
}

forvalues w=3/$w{
use "${r`w'raw}NHATS_Round_`w'_OP_File.dta", clear
sort spid opid 

merge m:1 spid using round_`w'_1.dta //bring in sorted sp ivw dataset
keep if _merge==3 //drop obs with no OP entries
save R`w'_OPSPlinked.dta, replace
}


//now create limited datasets for each wave with only needed variables
forvalues w=1/$w{
	use R`w'_OPSPlinked.dta
	
		if inlist(`w',1,2,3,4){
			keep spid opid op`w'* wave w`w'anfinwgt0 w`w'varstrat w`w'varunit r`w'dresid ///
		mo`w'douthelp mo`w'dinsdhelp mo`w'dbedhelp sc`w'deathelp sc`w'dbathhelp ///
		sc`w'dtoilhelp sc`w'ddreshelp ha`w'dlaunreas ha`w'dshopreas ha`w'dmealreas ///
		ha`w'dbankreas mc`w'dmedsreas
		}
		//add new wave to list below
		if inrange(`w',5,$w){ 
			keep spid opid op`w'* wave w`w'anfinwgt0 w`w'varstrat w`w'varunit r`w'dresid ///
		mo`w'douthelp mo`w'dinsdhelp mo`w'dbedhelp sc`w'deathelp sc`w'dbathhelp ///
		sc`w'dtoilhelp sc`w'ddreshelp ha`w'dlaunreas ha`w'dshopreas ha`w'dmealreas ///
		ha`w'dbankreas mc`w'dmedsreas w`w'an2011wgt0 
		}
		
	rename (op`w'* w`w'* r`w'* mo`w'* sc`w'* ha`w'* mc`w'*) (*) 

	save R`w'_OPSPltd.dta, replace
}



********************************************************************
//merge into single file for initial setup
use R1_OPSPltd.dta, clear
forvalues w=2/$w{
append using R`w'_OPSPltd.dta
}

save R1234_OPSPltd_n.dta, replace
********************************************************************

tab wave, missing

********************************************************************
//recode helper time variables to -1 (na) if not helper per flag variable
**Note, these OP variables are backfilled from wave to wave
gen opishelper_allw=.
forvalues w=1/$w{
tab ishelper wave if wave==`w', missing //helper flag variable
replace opishelper_allw=ishelper if wave==`w'
}
tab opishelper_allw wave, missing

forvalues w=1/$w{
	foreach v in helpsched numdayswk numdaysmn numhrsday {
		tab `v' if ishelper==-1 & wave==`w', missing
		replace `v'=-1 if ishelper==-1 & wave==`w'
	}
}

//generate indicator of help with any task that has monthly reference period
gen month=.
forvalues w=1/$w{
replace month=0 if wave==`w'
replace month=1 if (outhlp==1 | insdhlp==1 | bedhlp==1 | tkplhlp1==1 | ///
 tkplhlp2==1 | launhlp==1 | shophlp==1 | mealhlp==1 | bankhlp==1 | ///
 eathlp==1 | bathhlp==1 | toilhlp==1 | dreshlp==1 | medshlp==1) & wave==`w'
} 
tab month wave, missing

 //generate indicator of help with any task that has yearly reference period
gen year=.
forvalues w=1/$w{
replace year=0 if wave==`w'
replace year=1 if (moneyhlp==1 | dochlp==1 | insurhlp==1) & wave==`w'
tab year month if wave==`w', missing
}


//generate indicator of help in last year but not last month
gen yearnotmonth=0
replace yearnotmonth=1 if year==1 & month==0
tab yearnotmonth wave, missing
********************************************************************
//pull variable for sits in with doctor
gen op_doc_help=.
forvalues w=1/$w {
	replace op_doc_help=dochlp if wave==`w'
}
********************************************************************
//generate SP received help with self care or mobility activity or 
//household activity for health and functioning reasons
gen disab=.
forvalues w=1/$w{
replace disab=0 if wave==`w'
//self care or mobility, assumed to be for health reasons
replace disab=1 if (douthelp==2 | dinsdhelp==2 | dbedhelp==2 | ///
deathelp==2 | dbathhelp==2 | dtoilhelp==2 | ddreshelp==2) & wave==`w'
//household activity for health or functioning reasons or in residential care
**note this is different from tech paper #7 because codes disab=1 if 1,3 or 4
**not just if 3 or 4
foreach v in dlaunreas dshopreas dmealreas dbankreas dmedsreas{
replace disab=1 if inlist(`v',1,3,4) & wave==`w'
}
}
tab disab wave, missing

********************************************************************
//flag variable for hours/month reported, valid
tab dhrsmth wave, missing //derived hours/month from HL section
gen op_ind_hrsmth=.
forvalues w=1/$w{
replace op_ind_hrsmth=dhrsmth if wave==`w'
}
recode op_ind_hrsmth (1/750=1)

la def hrsmth -13"-13:Deceased" -12"Zero days/wk" -11"-11:hours missing" ///
 -10"-10:days missing" -9 "-9:days+hours mssing" -1"-1:NA" 1"1:Valid" 9999"9999:<1hr/day",replace
la val op_ind_hrsmth hrsmth 
tab op_ind_hrsmth wave, missing

//generate variable indicating one day per month or one day per week
gen justone=.
forvalues w=1/$w{
replace justone=0 if wave==`w'
replace justone=1 if (numdayswk==1 | numdaysmn==1) & wave==`w'
}
tab justone op_ind_hrsmth, missing

//recode help flags to 0=(no or inapplicable) or 1=yes
forvalues w=1/$w{
foreach v in outhlp insdhlp bedhlp tkplhlp1 tkplhlp2 launhlp shophlp mealhlp ///
bankhlp eathlp bathhlp toilhlp dreshlp medshlp moneyhlp dochlp insurhlp{
tab `v' if wave==`w', missing
replace `v'=0 if `v'==-1 & wave==`w'
}
}

//generate variable indicating help with only one activity
gen numact=.
forvalues w=1/$w{
replace numact=outhlp + insdhlp + bedhlp + tkplhlp1 + tkplhlp2 + launhlp + ///
shophlp + mealhlp + bankhlp + eathlp + bathhlp + toilhlp + dreshlp + medshlp ///
+ moneyhlp + dochlp + insurhlp if wave==`w'
}

tab numact wave, missing
gen oneact=0 //note this =0 if help with no activities
replace oneact=1 if numact==1
tab oneact opishelper_allw, missing

********************************************************************
//generate variables indicating relationship is spouse, other rel, other nonrel
gen spouse=.
gen otherrel=.
gen nonrel=.
gen daughter=0
gen son=0
gen oth_fam=0

label var otherrel "Non-spouse relative"
label var oth_fam "Family member of SP, not spouse, son, or daughter"
label var nonrel "Non-relative"

forvalues w=1/$w{
replace spouse=0 if wave==`w'
replace spouse=1 if relat==2 & wave==`w'
replace otherrel=0 if wave==`w'
replace otherrel=1 if ((relat>2 & relat<=29) | relat==91) & wave==`w'
replace nonrel=0 if wave==`w'
replace nonrel=1 if (relat>=30 & relat~=91) & wave==`w'
replace daughter=1 if inlist(relat,3,5,7) & wave==`w'
replace son=1 if inlist(relat,4,6,8) & wave==`w'
replace oth_fam=1 if ((relat>8 & relat<=29) | relat==91) & wave==`w'

tab relat spouse
tab relat otherrel
tab relat nonrel
}

tab spouse wave, missing
tab otherrel wave, missing
tab nonrel wave, missing

********************************************************************
//generate variables indicating help w/ adls & iadls
local adl insd bed eat bath toil dres
local iadl laun shop meal bank meds 
local nonadl out money tkpl

gen tkplhlp=.
forvalues w=1/$w{
replace tkplhlp=tkplhlp1==1 | tkplhlp2==1 if wave==`w'
}

foreach x in `adl' {
	gen op_adl_`x'=0
}
foreach x in `iadl' {
	gen op_iadl_`x'=0
}
foreach x in `nonadl' {
	gen op_nonadl_`x'=0
}

forvalues w=1/$w {
foreach x in `adl' {
	replace op_adl_`x'=1 if `x'hlp==1 & wave==`w'
}
foreach x in `iadl' {
	replace op_iadl_`x'=1 if `x'hlp==1 & wave==`w'
}
foreach x in `nonadl' {
	replace op_nonadl_`x'=1 if `x'hlp==1 & wave==`w'
}
}

********************************************************************
//generate variables indicating help on a regular schedule
gen regular=.
forvalues w=1/$w{
replace regular=0 if wave==`w'
replace regular=1 if helpsched==1 & wave==`w'
}
tab regular wave, missing

la def hrsmth -13"-13:Deceased" -12"-12:Zero days/wk" -11"-11:hours missing" ///
 -10"-10:days missing" -9 "-9:days+hours mssing" -1"-1:NA" 1"1:Valid" 9999"9999:<1hr/day",replace
la val op_ind_hrsmth hrsmth 

tab op_ind_hrsmth wave

tab op_ind_hrsmth yearnotmonth if wave==1 & opishelper_allw==1
tab op_ind_hrsmth yearnotmonth if wave==2 & opishelper_allw==1

tab opishelper_allw

//generate indicator of imputation category
//1=impute, 2=recode to zero, 3=recode to small number hrs/day, 4=no imputation needed
gen impute_cat=-1

//wave=1
	//1=impute
	replace impute_cat=1 if inlist(op_ind_hrsmth,-11,-10) | ///
	(op_ind_hrsmth==-9 & yearnotmonth==0) & ishelper==1 & wave==1
	replace impute_cat=1 if op_ind_hrsmth==9999 & (numdayswk==1 | numdaysmn==1) ///
	 & yearnotmonth==0 & ishelper==1 & wave==1
	//2=recode to zero
	replace impute_cat=2 if op_ind_hrsmth==-9 & yearnotmonth==1 & ishelper==1 & wave==1
	replace impute_cat=2 if op_ind_hrsmth==9999 & (numdayswk==1 | numdaysmn==1) ///
	& yearnotmonth==1 & ishelper==1 & wave==1
	//3=set to small number
	replace impute_cat=3 if op_ind_hrsmth==9999 & ///
	(numdayswk~=1 & numdaysmn~=1) & ishelper==1 & wave==1
	//4=imputation not necessary, complete info
	replace impute_cat=4 if op_ind_hrsmth==1 & ishelper==1 & wave==1

//if wave==2 | wave==3 | wave == 4
	//1=impute, wave 2,3 criteria
	replace impute_cat=1 if inlist(op_ind_hrsmth,-11,-9) & opishelper_allw==1 & inrange(wave,2,$w)
	replace impute_cat=1 if op_ind_hrsmth==-12 & yearnotmonth==0 & opishelper_allw==1 & inrange(wave,2,$w)
	//2=recode to zero, wave 2, 3
	replace impute_cat=2 if op_ind_hrsmth==-12 & yearnotmonth==1 & opishelper_allw==1 & inrange(wave,2,$w)
	//3=recode to small number
	replace impute_cat=3 if op_ind_hrsmth==9999 & opishelper_allw==1 & inrange(wave,2,$w)
	//4=imputation not necessary
	replace impute_cat=4 if op_ind_hrsmth==1 & opishelper_allw==1 & inrange(wave,2,$w)

la def impute_cat -1"N/A, not helper" 1"Impute" 2"Recode to zero" ///
3"Recode to small number" 4"No imputation needed"
la val impute_cat impute_cat
tab impute_cat wave, missing

forvalues w=1/$w{
tab impute_cat op_ind_hrsmth if wave==`w', missing
}

******************************************************************** 
//save separate datasets for each round so can run imputation with weighting
forvalues w=1/$w{
preserve  
/*keep op* wave numact yearnotmonth disab anfinwgt0 varstrat varunit ///
 dresid spid justone spouse otherrel nonrel regular oneact impute_cat son ///
 daughter oth_fam */
keep if wave==`w'
save R`w'_for_hrs_impute.dta,replace
restore
}
*/
********************************************************************
//implement matching, do separately for each wave
forvalues w=1/$w{
use R`w'_for_hrs_impute.dta, clear

svyset varunit [pweight=anfinwgt0], strata(varstrat)

//wave 1 imputation... need to check what is different if anything for wave 2
gen op_hrsmth_i=0
replace op_hrsmth_i=dhrsmth if impute_cat==4  //use actual value
replace op_hrsmth_i=0 if impute_cat==2 //set to zero
replace op_hrsmth_i=(.5*numdayswk*4.3) if impute_cat==3 & helpsched==1 //assign small hours
replace op_hrsmth_i=(.5*numdaysmn) if impute_cat==3 & inlist(helpsched,2,-7,-8)  //assign small hours

//impute where op_ind_hrsmth=-10, missing days per month
gen op_numdays_i=0
replace op_numdays_i=numdayswk*4.3 if inlist(impute_cat,3,4) & helpsched==1
replace op_numdays_i=numdaysmn if inlist(impute_cat,3,4) & inlist(helpsched,2,-7,-8)

gen ln_opnumdays_i=ln(op_numdays_i) if inlist(impute_cat,3,4)

local adjvars spouse otherrel regular yearnotmonth oneact disab ///
 outhlp insdhlp bedhlp tkplhlp1 tkplhlp2 launhlp ///
 shophlp mealhlp bankhlp eathlp bathhlp toilhlp ///
 dreshlp medshlp moneyhlp dochlp insurhlp

svy: reg ln_opnumdays_i `adjvars' if inlist(impute_cat,3,4)
predict ln_opnumdays_i2 if impute_cat==1 & op_ind_hrsmth==-10
gen op_numdays_i2=exp(ln_opnumdays_i2) if impute_cat==1 & op_ind_hrsmth==-10
replace op_hrsmth_i= op_numdays_i2*numhrsday if impute_cat==1 & ///
 op_ind_hrsmth==-10 & numhrsday>=1
replace op_hrsmth_i= op_numdays_i2*.5 if impute_cat==1 & ///
 op_ind_hrsmth==-10 & numhrsday==0

//impute where op_ind_hrsmth=-11, missing hours per day
gen op_numhrsday_i=0
replace op_numhrsday_i=.5 if impute_cat==3
replace op_numhrsday_i=numhrsday if impute_cat==4
gen ln_opnumhrsday_i=ln(op_numhrsday_i) if inlist(impute_cat,3,4)
svy: reg ln_opnumhrsday_i `adjvars' if inlist(impute_cat,3,4)
predict ln_opnumhrsday_i2 if impute_cat==1 & op_ind_hrsmth==-11
gen op_numhrsday_i2=exp(ln_opnumhrsday_i2) if impute_cat==1 & op_ind_hrsmth==-11
replace op_hrsmth_i=numdayswk*4.3*op_numhrsday_i2 if impute_cat==1 & ///
 op_ind_hrsmth==-11 & helpsched==1
replace op_hrsmth_i=numdaysmn*op_numhrsday_i2 if impute_cat==1 & ///
 op_ind_hrsmth==-11 & inlist(helpsched,2,-7,-8) & numdaysmn>=1
replace op_hrsmth_i=numdayswk*4.3*op_numhrsday_i2 if impute_cat==1 & ///
 op_ind_hrsmth==-11 & inlist(helpsched,2,-7,-8) & ///
 numdaysmn<1 & numdayswk>=1 

//impute where op_ind_hrsmth=-9, -12 or 9999, missing hours and days or <1hr/day
gen ln_opdhrsmth_i=ln(op_hrsmth_i) if inlist(impute_cat,3,4)
svy: reg ln_opdhrsmth_i `adjvars' if inlist(impute_cat,3,4)
predict ln_opdhrsmth_i2 if impute_cat==1 & inlist(op_ind_hrsmth,-9,-12,9999)
gen op_hrsmth_i2=exp(ln_opdhrsmth_i2) if impute_cat==1 & inlist(op_ind_hrsmth,-9,-12,9999)
replace op_hrsmth_i=op_hrsmth_i2 if impute_cat==1 & inlist(op_ind_hrsmth,-9,-12,9999)
replace op_hrsmth_i=-1 if ishelper~=1 

//look at imputed hours per month by imputation category
di "where op_ind_hrsmth==-12"
capture noisily: mean op_hrsmth_i if impute_cat==1 & op_ind_hrsmth==-12 [pweight=anfinwgt0]
di "where op_ind_hrsmth==-11"
mean op_hrsmth_i if impute_cat==1 & op_ind_hrsmth==-11 [pweight=anfinwgt0]
di "where op_ind_hrsmth==-10"
capture noisily: mean op_hrsmth_i if impute_cat==1 & op_ind_hrsmth==-10 [pweight=anfinwgt0]
di "where op_ind_hrsmth==-9"
mean op_hrsmth_i if impute_cat==1 & op_ind_hrsmth==-9 [pweight=anfinwgt0] 
di "where op_ind_hrsmth==9999"
capture noisily: mean op_hrsmth_i if impute_cat==1 & op_ind_hrsmth==9999 [pweight=anfinwgt0] 

di "impute_cat==2"
mean op_hrsmth_i if op_hrsmth_i~=-1 & impute_cat==2 [pweight=anfinwgt0]
di "impute_cat==3"
mean op_hrsmth_i if op_hrsmth_i~=-1 & impute_cat==3 [pweight=anfinwgt0]
di "impute_cat==4"
mean op_hrsmth_i if op_hrsmth_i~=-1 & impute_cat==4 [pweight=anfinwgt0]

save R`w'_hrs_imputed_added.dta,replace

}

********************************************************************
log close

translate "${logs}/`name'.smcl" "${logs}/`name'.pdf"
exit

```

## NHATS Data Cleaning Part 3 (2_r12_cleaning_1_1)
```

H="NHATS Data Cleaning Part 3 (2_r12_cleaning_1_1)"
/*

Created by: --
Date Created: --

Updated by: MH
Date Updated: 07/02/2019

Description: This section uses the SP and OP data files and imputes
hours of help provided to SP per methods in NHATS Technical Paper #7

Variables from this dataset are then brought in to the SP Rounds dataset
and also the NSOC caregiver dataset.



**************************************************
*/

local date = subinstr("$S_DATE"," ","_",.) 
local name 2-r12_cleaning_1_`date'
di "`name'"

capture log close 
clear all

set more off
version 12
set linesize 80
set maxvar 10000


log using "${logs}/`name'.smcl", text replace




*********************************************

global miss -9,-1

cd "${work}"

use round_1_to_${w}.dta, clear
*********************************************
tab wave, missing

*********************************************
//interview type, varies by wave, use tracker status variables

tab r3spstat if wave==3
tab r3status if wave==3

gen ivw_type=3
replace ivw_type=1 if r1spstat==20 & wave==1 

forvalues w=2/$w{
	replace ivw_type=1 if (r`w'spstat==20 & r`w'status!=62) & wave==`w'
	replace ivw_type=2 if (r`w'spstat==20 & r`w'status==62) & wave==`w'
}

la def ivw_type 1 "Alive, SP interview completed" 2"Died, proxy SP LML interview completed" ///
	3"SP interview not completed"
la val ivw_type ivw_type
tab ivw_type wave, missing

tab r1status r1spstat if wave==1, missing
tab dresid r1spstat if wave==1, missing
//so use ivw type=1 is the same thing for r1

gen sp_ivw_yes=1 if r1spstat==20 & wave==1
replace sp_ivw_yes=0 if inlist(r1spstat,11,24) & wave==1

forvalues w=2/$w{
	tab r`w'status r`w'spstat if wave==`w', missing
	replace sp_ivw_yes=1 if r`w'spstat==20 & wave==`w'
	replace sp_ivw_yes=0 if inlist(r`w'spstat,11,12,24) & wave==`w'
}

la var sp_ivw_yes "SP interview conducted? yes=1" //note this includes lml interviews
tab sp_ivw_yes wave, missing

gen lml_ivw_yes = 0
forvalues w=2/$w{
	replace lml_ivw_yes=1 if r`w'spstat==20 & r`w'status==62 & wave==`w'
}
la var lml_ivw_yes "LWL interview? yes=1"

tab sp_ivw_yes lml_ivw_yes, missing
tab lml_ivw_yes wave, missing

tab outoft r2status if wave==2, missing
tab outoft r3status if wave==3, missing

//identify proxy SP interviews
gen proxy_ivw=.
forvalues w=1/$w{
replace proxy_ivw=1 if resptype==2
replace proxy_ivw=0 if resptype==1
}
la var proxy_ivw "Interview via proxy"
tab proxy_ivw wave, missing
tab proxy_ivw wave if ivw_type==1, missing
tab proxy_ivw wave if ivw_type==2, missing

/*missing data convention per user guide
For our purposes, code all as missing
The following codes were used at the item level for missing data of different types:
-7 = Refusal
-8 = Don�t Know
-1 = Inapplicable
-9 = Not Ascertained*/


*********************************************
*********************************************
//definitions of homebound
*********************************************
*********************************************
//how often go out of the house?
gen freq_go_out=.
forvalues w=1/$w{
	tab outoft if wave==`w', missing
	replace freq_go_out=outoft if wave==`w'
}

replace freq_go_out=. if inlist(freq_go_out,$miss)
la def freq_go_out 1 "Every day" 2 "Most days (5-6/week)" ///
	3 "Some days (2-4/week)" 4 "Rarely" 5 "Never"
la val 	freq_go_out freq_go_out
la var freq_go_out "How often go outside"

tab freq_go_out sp_ivw_yes, missing
tab freq_go_out sp_ivw_yes if wave==1, missing
tab freq_go_out sp_ivw_yes if wave==2 & lml_ivw_yes==0, missing
tab freq_go_out sp_ivw_yes if wave==2 & lml_ivw_yes==1, missing
tab freq_go_out sp_ivw_yes if wave==3 & lml_ivw_yes==0, missing


//MO questions not asked in lml interviews where persons were not alert(PD6)
//or not mobile(PD7) in last month so lml has more missingless
tab freq_go_out outoft if wave==2 & lml_ivw_yes==1, missing

*********************************************
//Did anyone ever help you - asked if report going outside
//left missing if report never go outside
gen help_go_out=.
forvalues w=1/$w{
	tab outhlp if wave==`w', missing
	replace help_go_out=1 if outhlp==1 & wave==`w'
	replace help_go_out=0 if outhlp==2 & wave==`w'
	replace help_go_out=-7 if outhlp==-7 & wave==`w'
	replace help_go_out=-8 if outhlp==-8 & wave==`w'
}
la var help_go_out "Anyone ever help you leave home?"
tab help_go_out wave if sp_ivw_yes==1, missing

*********************************************
//go out of the house on own?
//only asked if report having help when go outside (help_go_out==1)
gen out_on_own=.
forvalues w=1/$w{
	tab outslf help_go_out if wave==`w', missing
	replace out_on_own=outslf if wave==`w'
}
replace out_on_own=. if inlist(out_on_own,$miss)
replace out_on_own=5 if help_go_out==0
replace out_on_own=6 if freq_go_out==5
la def out_on_own 1"Most times" 2"Sometimes" 3"Rarely" 4"Never" 5"NA, reported no help" ///
6"NA, never left home"
la val out_on_own out_on_own
la var out_on_own "When left house, left by yourself?"
tab out_on_own wave if sp_ivw_yes==1, missing

*********************************************
//needs help when go outside? gets help and also goes outside on own != never
gen needshelp=0
replace needshelp=1 if help_go_out==1 & out_on_own~=4
la var needshelp "Leaves home but with help"
tab needshelp wave, missing

*********************************************
//has difficulty? when you used your walker,cane,etc? how much difficulty
//did you have leaving the house by yourself? 1=none, 2=a little, 3=some, 4=a lot
//inapplicable if don't report using any device to help, so set to 0 here
gen hasdifficulty=0
forvalues w=1/$w{
	tab outdif if wave==`w', missing
	replace hasdifficulty=1 if inlist(outdif,2,3,4,5) & wave==`w'
	replace hasdifficulty=0 if outdif==1 & wave==`w'
	replace hasdifficulty=-7 if outdif==-7 & wave==`w'
	replace hasdifficulty=-8 if outdif==-8 & wave==`w'

}
la var hasdifficulty "Leaves home on own but has difficulty"
tab hasdifficulty wave if sp_ivw_yes==1, missing
*********************************************
//needs help or has difficulty going outside?
gen helpdiff=.
replace helpdiff=0 if needshelp==0 & hasdifficulty==0
replace helpdiff=1 if needshelp==1 | hasdifficulty==1
la var helpdiff "Either needs help or has difficulty going outside"
tab helpdiff wave if sp_ivw_yes==1, missing
*********************************************
//independent - go out some/most days but are independent
gen independent=0
replace independent=1 if help_go_out==0 & hasdifficulty==0
la var independent "Independently leaves home"
tab independent wave if sp_ivw_yes==1, missing

tab independent helpdiff, missing //no overlap
*********************************************
/*homebound categories
0 = never go out
1 = rarely go out
2 = go out but never by self
3 = go out but need help/have difficulty
4 = go out, independent - don't need help or have difficulty
*/
gen homebound_cat=.
replace homebound_cat=1 if freq_go_out==4 | freq_go_out==5
replace homebound_cat=2 if inlist(freq_go_out,3,2,1) & out_on_own==4
replace homebound_cat=3 if inlist(freq_go_out,3,2,1) & helpdiff==1
replace homebound_cat=4 if inlist(freq_go_out,3,2,1) & independent==1

//from technical paper
replace homebound_cat=3 if spid==10009011 & wave==1

la var homebound_cat "Homebound status, categorical"
la def hb 1 "Never/Rarely leaves home" 2 "Leaves home but never by self" ///
3 "Leaves home but needs help/has difficulty" 4 "Independent, not homebound"
la val homebound_cat hb
tab homebound_cat wave if sp_ivw_yes==1, missing
/*binary for never go out, since never/rarely together*/
gen never_go_out=freq_go_out==5 if freq!=.
label var never_go_out "Never leaves home"
*********************************************
/*subcategories for level 4 homeboud category
independent but go out only rarely
independent and go out frequently*/
gen indeplow=0
replace indeplow=1 if freq_go_out==3 & independent==1
la var indeplow "Independent, go out rarely" 

gen indephigh=0
replace indephigh=1 if inlist(freq_go_out,2,1) & independent==1
la var indephigh "Independent, go out often"

tab indeplow wave if sp_ivw_yes==1, missing
tab indephigh wave if sp_ivw_yes==1, missing

*********************************************
*********************************************
//demographic information
*********************************************
*********************************************
gen age=.

forvalues w=1/$w{
	replace age=r`w'dintvwrage if wave==`w'
	sum age if wave==`w', detail
	}	
//for last month of life interviews, use age at death variable
tab ivw_type if age==-1, missing
replace age=r2ddeathage if ivw_type==2 & r2dintvwrage==-1
sum age
//some observations that died still have missing age, so set to missing
tab ivw_type if inlist(age,-7,-8)
replace age=. if inlist(age,-7,-8)

la var age "Age at interview or death"
sum age if ivw_type==1, detail
sum age if ivw_type==2, detail
sum age, detail

//gender, only asked in round 1,5
gen female=.
replace female=1 if dgender==2 
replace female=0 if dgender==1 

la var female "Female"
la def female 0"Male" 1"Female"
la val female female
tab female if wave==1, missing
tab dracehisp, missing

//race, ethnicity
gen race_cat=dracehisp
replace race_cat=-8 if inlist(dracehisp,5,6)
label define race 1 "White Non-His" 2 "Black Non-His" 4 "Hispanic" ///
	3 "Other (Am Indian/Asian/Nat. Hawaii)" -8 "DKRF"
label values race_cat race
la var race_cat "Race, categorical"
tab race_cat if wave==1, missing

//education
generate education=.
replace education=1 if inlist(higstschl,1,2,3) 
replace education=2 if higstschl==4
replace education=3 if inlist(higstschl,5,6,7) 
replace education=4 if inlist(higstschl,8,9) 

la var education "Education level, categorical"
label define edulbl 1 "<High School" 2 "High School/GED" 3 "Some College" ///
	4 ">=Bachelors" 5 "DKRF"
label values education edulbl
tab education higstschl if wave==1, missing

gen educ_hs_ind=education>=2 if education>0 & education!=.

//income, just wave 1, reported every other wave
sum totinc if wave==1, detail //this is the actual reported income, only present 40% sample

forvalues i = 1/5 {
	gen imputed_inc`i'=.
	forvalues w=1 3 : $w{
		di "`w'"
		replace toincim`i'=. if inlist(toincim`i',-9,-1) & wave==`w'
		sum toincim`i' if wave==`w',detail //imputed income variables
		replace imputed_inc`i' = toincim`i' if wave==`w'
}
		label var imputed_inc`i' "Imputed Income `i'"
}


egen aveincome=rowmean(imputed_inc1-imputed_inc5)
forvalues w=1 3 : $w{
	replace aveincome=totinc if wave==`w' & totinc>=0 & ///
	!missing(totinc)
}

sort spid wave
by spid: carryforward aveincome, replace
//use first income imputation to set income categorical variable
gen income_cat=0 if aveincome<15000 & aveincome!=.
replace income_cat=1 if aveincome>=15000 & aveincome<30000 & aveincome!=.
replace income_cat=2 if aveincome>=30000 & aveincome<60000 & aveincome!=.
replace income_cat=3 if aveincome>=60000 & aveincome!=.
label define income_cat 0 "<15000" 1 "15-29,999" 2 "30-59,999" 3 ">60000"
label values income_cat income_cat
tab income_cat, missing

//speak language other than english? Asked when new cohort
gen otherlang=spkothlan if wave==1
replace otherlang=0 if spkothlan==2 & wave==1
replace otherlang=1 if condspanh==1 & wave==1
replace otherlang = . if inlist(otherlang,-9,-1)
la var otherlang "Speak language other than English?"
tab otherlang wave, missing

replace otherlang=0 if spkothlan==2 & wave==5
replace otherlang=1 if spkothlan==1 & wave==5
replace otherlang=1 if condspanh==1 & wave==5
replace otherlang=. if inlist(spkothlan,-9,-1) & wave==5


//fill in gender, race, education, income, etc. for wave 2 interviews
sort spid wave
by spid (wave): carryforward female race_cat education income_cat ///
otherlang imputed_inc1 imputed_inc2 imputed_inc3 imputed_inc4 imputed_inc5, replace

gen white=race_cat==1
gen black=race_cat==2
gen hisp=race_cat==4
gen other_race=race_cat==3
label var white "White Non-Hisp"
label var black "Black Non-Hisp"
label var hisp "Hispanic"
label var other_race "Other race/ethnicity"
//merge white & other for reportability
gen whiteoth=white | other_race if !missing(white) & !missing(other_r)
label var whiteoth "White Non-Hispanic or other"

//marital status
//wave 2,3, use derived variable which is populated if not asked b/c no change 
gen maritalstat=martlstat if wave==1
replace maritalstat=dmarstat if wave!=1
replace maritalstat=-9 if inlist(maritalstat,-9, -1)



la var maritalstat "Marital status"
label define maritallbl 1 "Married" 2 "Live w/Part" 3 "Separated" ///
	4 "Divorced" 5 "Widowed" 6 "Never Married" -7 "Refused" -8 "DK"
la val maritalstat maritallbl
tab maritalstat wave, missing


*********************************************
*********************************************
//Insurance coverage
*********************************************
*********************************************
//medicaid status
gen medicaid=.
forvalues w=1/$w{
	tab cmedicaid if wave==`w' , missing
	replace medicaid=1 if cmedicaid==1 & wave==`w'
	replace medicaid=0 if cmedicaid==2 & wave==`w'
	replace medicaid=-7 if cmedicaid==-7 & wave==`w'
	replace medicaid=-8 if cmedicaid==-8 & wave==`w'
}
la var medicaid "Medicaid"
tab medicaid wave,missing
tab medicaid if wave==1 & ivw_type==1,missing
//medigap status
gen medigap=.
forvalues w=1/$w{
	tab mgapmedsp if wave==`w' , missing
	replace medigap=1 if mgapmedsp==1 & wave==`w'
	replace medigap=0 if mgapmedsp==2 & wave==`w'
	replace medigap=-7 if mgapmedsp==-7 & wave==`w'
	replace medigap=-8 if mgapmedsp==-8 & wave==`w'
}
la var medigap "Medigap"
tab medigap wave, missing

//medicare part d
gen mc_partd=.
forvalues w=1/$w{
	tab covmedcad if wave==`w' , missing
	replace mc_partd=1 if covmedcad==1 & wave==`w'
	replace mc_partd=0 if covmedcad==2 & wave==`w'
	replace mc_partd=-7 if covmedcad==-7 & wave==`w'
	replace mc_partd=-8 if covmedcad==-8 & wave==`w'
}
la var mc_partd "Medicare Part D"
tab mc_partd wave,missing

//tricare - va insurance
gen tricare=.
forvalues w=1/$w{
	tab covtricar if wave==`w' , missing
	replace tricare=1 if covtricar==1 & wave==`w'
	replace tricare=0 if covtricar==2 & wave==`w'
	replace tricare=-7 if covtricar==-7 & wave==`w'
	replace tricare=-8 if covtricar==-8 & wave==`w'
}
la var tricare "Tricare - veterans insurance"
tab tricare wave,missing

//long term care nursing home insurance, private
gen private_ltc=.
forvalues w=1/$w{
	tab nginsnurs if wave==`w' , missing
	replace private_ltc=1 if nginsnurs==1 & wave==`w'
	replace private_ltc=0 if nginsnurs==2 & wave==`w'
	replace private_ltc=-7 if nginsnurs==-7 & wave==`w'
	replace private_ltc=-8 if nginsnurs==-8 & wave==`w'
}
la var private_ltc "Private nursing home insurance"
tab private_ltc wave,missing

//if report nh insurance in prev wave, asked to verify still have it curr wave
forvalues w=1/$w{
tab nginslast if wave==`w' & nginsnurs==-1, missing
replace private_ltc=1 if nginslast==1 & nginsnurs==-1 & wave==`w'
replace private_ltc=0 if nginslast==2 & nginsnurs==-1 & wave==`w'
replace private_ltc=-7 if nginslast==-7 & nginsnurs==-1 & wave==`w'
replace private_ltc=-8 if nginslast==-8 & nginsnurs==-1 & wave==`w'
}
tab private_ltc wave,missing

*********************************************
*********************************************
//Residence information
*********************************************
*********************************************
//residence physical structure variable only in wave 1
tab resistrct if wave==1, missing
tab resistrct placedesc if wave==1, missing
//derived variable in waves 2, 3
tab dresistrct if wave==2, missing
//per user guide, need to fix this variable since incorrectly coded for 71 obs
replace dresistrct = newstrct if inlist(spadrsnew,1,3) & inlist(newstrct,1,2,3,4,91)

tab dresistrct if wave==3, missing

//housing type in each wave, however missing if same residence as prior wave
forvalues w=1/$w{
tab placedesc if wave==`w', missing
}

//if missing housing type in waves 2,3, backfill with wave 1 info
//get indicators for moving in waves 2 or 3
tab dadrscorr if wave==2, missing

gen moved=1 if dadrscorr==2 & wave==2
replace moved=0 if (dadrscorr==1 | dadrscorr==3) & wave==2
tab moved

forvalues w=3/$w{
tab spadrsnew if wave==`w', missing
replace moved=1 if spadrsnew==2 & wave==`w'
replace moved=0 if (spadrsnew==1 | spadrsnew==3) & wave==`w'
tab moved
}

gen ht_new=.

forvalues w=1/$w{
replace ht_new=placedesc if wave==`w'
}

forvalues w=2/$w{
replace ht_new=. if placedesc==-1 & moved==0 & wave==`w'
}

tab ht_new wave, missing

sort spid wave
by spid (wave): carryforward ht_new, replace

tab ht_new wave, missing

//most residence information from wave 1
//used derived variables wave 2,3
gen residence=.
replace residence=resistrct if wave==1
forvalues w=2/$w{
replace residence=dresistrct if wave==`w'
}

replace residence=3 if inlist(residence,4,91)

tab residence wave

//now overwrite with the new housing type description as done in orig code
replace residence=1 if ht_new==1
replace residence=2 if inlist(ht_new,2,3,4,91)
replace residence=. if inlist(residence,-9,-1)
tab residence wave, missing

label define residlbl 1 "Private Res." 2 "Group Home/Facility" ///
	3 "Mobile Home/Multi-Unit"
label values residence residlbl
forvalues w=1/$w{
tab residence if wave==`w', missing
}


***********************************************************
//living arrangement, NHATS derived variable
gen livearrang=.
forvalues w=1/$w{
tab dlvngarrg if wave==`w' & ivw_type==1, missing
replace livearrang=dlvngarrg if wave==`w'
replace livearrang=1 if (dlvngarrg==-9 & dhshldnum==1)  & wave==`w'
replace livearrang=-7 if dlvngarrg == -7 & wave == `w'
replace livearrang=-8 if dlvngarrg == -8 & wave == `w'
}
replace livearrang=. if livearrang==-1 

la var livearrang "Living arrangmeent, categorical"
la def livearrang 1 "Alone" 2 "With spouse/partner" ///
	3"With spouse/partner + others" 4 "With others"
la val livearrang livearrang
tab livearrang wave if ivw_type==1, missing

//live alone?
gen livealone=1 if livearrang==1
replace livealone=0 if inlist(livearrang,2,3,4)
la var livealone "Lives alone"
tab livealone livearrang, missing

save round_1_${w}_cleanv1.dta, replace
local ivars age female race_cat education  income_cat otherlang maritalstat ///
medicaid medigap mc_partd tricare private_ltc moved ht_new ///
residence livearrang livealone

foreach v in `ivars'{
tab `v'
}

*********************************************
log close

translate "${logs}/`name'.smcl" "${logs}/`name'.pdf"
exit
```

## NHATS Data Cleaning Part 4 (3_r12_cleaning_2_1)

```
H="NHATS Data Cleaning Part 4 (3_r12_cleaning_2_1)"
/*

Created by: --
Date Created: --

Updated by: MH
Date Updated: 07/02/2019

Description: This section uses the SP and OP data files and imputes
hours of help provided to SP per methods in NHATS Technical Paper #7

Variables from this dataset are then brought in to the SP Rounds dataset
and also the NSOC caregiver dataset.



**************************************************
*/

local date = subinstr("$S_DATE"," ","_",.) 
local name 3_nhats_cleaning_`date'
di "`name'"

capture log close 
clear all

set more off
version 12
set linesize 80
set maxvar 10000


log using "${logs}/`name'.smcl", text replace



cd "${work}"

*********************************************

use round_1_${w}_cleanv1.dta, clear

*********************************************
tab wave, missing

*********************************************
*********************************************
//self reported health status/conditions
*********************************************
*********************************************
gen srh=.
forvalues w=1/$w{
tab health if wave==`w', missing
replace srh=health if wave==`w'
}
replace srh=. if inlist(srh,-9,-1)

la var srh "Self reported health, categorical"
la def srh 1 "Excellent" 2 "Very good" 3 "Good" 4 "Fair" 5 "Poor"
la val srh srh
tab srh if ivw_type==1, missing

gen srh_fp=1 if srh==4 | srh==5
replace srh_fp=0 if inlist(srh,1,2,3)
replace srh_fp=-7 if srh == -7
replace srh_fp=-8 if srh == -8
la var srh_fp "Self reported health=fair/poor"
tab srh_fp srh, missing

*********************************************
//Self-Report Disease
*********************************************
//Round 1 - ever told you had xx
//Round 2 - since the last interview, new condition? 
/*
1 heart attack
2 heart disease
3 high blood pressure
4 arthritis
5 osteoporosis
6 diabetes
7 lung disease
8 stroke
9 dementia
10 cancer
create two sets of indicator variables for waves 2 and later
sr_disease_ever = 1 if ever report having the disease (only this variable in wave 1)
sr_disease_new = 1 if newly reported the disease this wave
*/

//heart attack / ami, asked both waves, for Wave 1 ask ever had, wave 2 ask in last year
//same questions for stroke, cancer

capture program drop disease
program define disease 
	args dis num
	
	gen sr_`dis'_raw=.
	forvalues w=1/$w{
		replace sr_`dis'_raw=1 if disescn`num'==1 & wave==`w'
		replace sr_`dis'_raw=0 if disescn`num'==2 & wave==`w'
		replace sr_`dis'_raw=-7 if disescn`num'==-7 & wave==`w'
		replace sr_`dis'_raw=-8 if disescn`num'==-8 & wave==`w'
	}
	//create two new variables from this...
	gen sr_`dis'_ever=sr_`dis'_raw
	replace sr_`dis'_ever=. if inrange(wave, 2, $w) & sr_`dis'_raw==0 //set to missing if report no new wave
	replace sr_`dis'_ever=-7 if sr_`dis'_raw == -7
	replace sr_`dis'_ever=-8 if sr_`dis'_raw == -8

	sort spid wave
	by spid (wave): carryforward sr_`dis'_ever if ivw_type==1 , replace
	
	replace sr_`dis'_ever = sr_`dis'_raw if inrange(wave,2,$w) & sr_`dis'_ever == . & sr_`dis'_raw != .

	gen sr_`dis'_new=sr_`dis'_raw if inrange(wave,2,$w)

	end

//now run programs for specific disease variables
disease ami 1
disease stroke 8
disease cancer 10

la var sr_ami_raw "Self report Heart Attack, Raw"
la var sr_ami_ever "Ever Self report Heart Attack"
la var sr_ami_new "Self report Heart Attack since prev year"
la var sr_stroke_raw "Self report Stroke, Raw"
la var sr_stroke_ever "Ever Self report Stroke"
la var sr_stroke_new "Self report Stroke since prev year"
la var sr_cancer_raw "Self report Cancer, Raw"
la var sr_cancer_ever "Ever Self report Cancer"
la var sr_cancer_new "Self report Cancer since prev year"

notes sr_ami_raw: disescn1 
notes sr_ami_raw: Had a heart attack or myocardial infarction (self-report).
notes sr_stroke_raw: disescn8
notes sr_stroke_raw: Had a stroke (self-report).
notes sr_cancer_raw: disescn10
notes sr_cancer_raw: Had cancer (self-report).

notes sr_ami_ever: sr_ami_raw
notes sr_ami_ever: If ever had a heart attack or myocardial infarction (self-report).
notes sr_stroke_ever: sr_stroke_raw
notes sr_stroke_ever: If ever had a stroke (self-report).
notes sr_cancer_ever: sr_cancer_raw
notes sr_cancer_ever: If ever had cancer (self-report). 

notes sr_ami_new: sr_ami_raw
notes sr_ami_new: If heart attack or myocardial infarction within last year (self-report).
notes sr_stroke_new: sr_stroke_raw
notes sr_stroke_new: If stroke within last year (self-report).
notes sr_cancer_new: sr_cancer_raw
notes sr_cancer_new: If cancer found within last year (self-report).

//same question pattern, different variable name though, for hip fracture
gen sr_hip_raw=.
forvalues w=1/$w{
	replace sr_hip_raw=1 if brokebon1==1 & wave==`w'
	replace sr_hip_raw=0 if brokebon1==2 & wave==`w'
	replace sr_hip_raw = -7 if brokebon1==-7 & wave==`w'
	replace sr_hip_raw = -8 if brokebon1==-8 & wave==`w'
}
//create two new variables from this...
gen sr_hip_ever=sr_hip_raw
replace sr_hip_ever=. if (inrange(wave,2,$w)) & sr_hip_raw==0 //set to missing if report no new wave

sort spid wave
by spid (wave): carryforward sr_hip_ever if ivw_type==1 , replace

replace sr_hip_ever = sr_hip_raw if inrange(wave,2,$w) & sr_hip_ever == . & sr_hip_raw != .

gen sr_hip_new=sr_hip_raw if inrange(wave,2,$w)
la var sr_hip_raw "Self report Hip Fracture, Raw"
la var sr_hip_ever "Ever Self report Hip Fracture (since age 50)"
la var sr_hip_new "Self report Hip Fracture since prev year"

notes sr_hip_raw: brokebon1
notes sr_hip_raw: Had broken or fractured hip (self-report).
notes sr_hip_ever: sr_hip_raw
notes sr_hip_ever: Had ever broken or fractured hip (self-report).
notes sr_hip_new: sr_hip_raw
notes sr_hip_new: Had hip fracture since prev interview (self-report). 

foreach x in sr_ami_raw sr_stroke_raw sr_cancer_raw sr_ami_ever sr_stroke_ever ///
sr_ami_new sr_stroke_new sr_cancer_new sr_cancer_ever sr_hip_raw sr_hip_ever sr_hip_new{
notes `x': Located in NHATS Setup, under Part 4.
notes `x': Updated: 04/15/19-MH
}

foreach dis in ami stroke cancer hip{
tab sr_`dis'_raw wave if ivw_type==1, missing
tab sr_`dis'_ever wave if ivw_type==1, missing
tab sr_`dis'_new wave if ivw_type==1, missing
}

**************************************************************************
//Many diseases asked 1st wave, then only if report no, asked again 2nd wave
//wave 2,3=7 if report yes in wave 1
//use a program over several diseases since pattern is the same

la def sr_prev_rept 0 "No" 1 "Yes" 7 "Previously reported"

capture program drop prev_report
program define prev_report
	args dis num
 
	gen sr_`dis'_raw=.
		forvalues w=1/$w{
			replace sr_`dis'_raw=1 if disescn`num'==1 & wave==`w'
			replace sr_`dis'_raw=0 if disescn`num'==2 & wave==`w'
			replace sr_`dis'_raw=7 if disescn`num'==7 & wave==`w'
			replace sr_`dis'_raw=-7 if disescn`num'==-7 & wave==`w'
			replace sr_`dis'_raw=-8 if disescn`num'==-8 & wave==`w'
		}

	la val sr_`dis'_raw sr_prev_rept
	tab sr_`dis'_raw wave if ivw_type==1, missing

	gen sr_`dis'_ever = 1 if inlist(sr_`dis'_raw,1,7) //if prev or curr report yes
	replace sr_`dis'_ever = 0 if sr_`dis'_raw==0
	replace sr_`dis'_ever = -7 if sr_`dis'_raw==-7
	replace sr_`dis'_ever = -8 if sr_`dis'_raw==-8
	tab sr_`dis'_ever sr_`dis'_raw, missing
	tab sr_`dis'_ever wave if ivw_type==1, missing

	gen sr_`dis'_new=sr_`dis'_raw if inrange(wave,2,$w)
	replace sr_`dis'_new=0 if sr_`dis'_raw==7
	tab sr_`dis'_new if inrange(wave,2,$w) & ivw_type==1, missing

	end

//now run programs for specific disease variables
prev_report heart_dis 2
prev_report htn 3
prev_report ra 4
prev_report osteoprs 5
prev_report diabetes 6
prev_report lung_dis 7
prev_report dementia 9

la var sr_heart_dis_ever "Self report Heart Disease"
la var sr_heart_dis_new "Self report Heart Disease since prev year"
la var sr_htn_ever "Self report high blood pressure"
la var sr_htn_new "Self report high blood pressure since prev year"
la var sr_ra_ever "Self report arthritis"
la var sr_ra_new "Self report arthritis since prev year"
la var sr_osteoprs_ever "Self report osteoporosis"
la var sr_osteoprs_new "Self report osteoporosis since prev year"
la var sr_diabetes_ever "Self report diabetes"
la var sr_diabetes_new "Self report diabetes since prev year"
la var sr_lung_dis_ever "Self report Lung Disease"
la var sr_lung_dis_new "Self report Lung Disease since prev year"
la var sr_dementia_ever "Self report Dementia"
la var sr_dementia_new "Self report Dementia since prev year"


notes sr_heart_dis_raw: disescn2
notes sr_heart_dis_raw: Had any heart disease including angina or CHF (self-report)
notes sr_heart_dis_ever: sr_heart_dis_raw
notes sr_heart_dis_ever: Had heart disease ever (self-report)
notes sr_heart_dis_new:  sr_heart_dis_raw
notes sr_heart_dis_new: Had heart disease since prev interview (self-report). 

notes sr_htn_raw: disescn3
notes sr_htn_raw: Had High blood pressure (self-report)
notes sr_htn_ever: sr_htn_raw
notes sr_htn_ever: Had High blood pressure ever (self-report)
notes sr_htn_new:  sr_htn_raw
notes sr_htn_new: Had High blood pressure since prev interview (self-report). 

notes sr_ra_raw: disescn4
notes sr_ra_raw: Had arthritis (including osteo or rheumatoid) (self-report)
notes sr_ra_ever: sr_ra_raw
notes sr_ra_ever: Had arthritis ever (self-report)
notes sr_ra_new:  sr_ra_raw
notes sr_ra_new: Had arthritis since prev interview (self-report). 

notes sr_osteoprs_raw: disescn5
notes sr_osteoprs_raw: Had osteoporosis (self-report)
notes sr_osteoprs_ever: sr_osteoprs_raw
notes sr_osteoprs_ever: Had osteoporosis ever (self-report)
notes sr_osteoprs_new:  sr_osteoprs_raw
notes sr_osteoprs_new: Had osteoporosis since prev interview (self-report). 

notes sr_diabetes_raw: disescn6
notes sr_diabetes_raw: Had arthritis (including osteo or rheumatoid) (self-report)
notes sr_diabetes_ever: sr_diabetes_raw
notes sr_diabetes_ever: Had diabetes ever (self-report)
notes sr_diabetes_new:  sr_diabetes_raw
notes sr_diabetes_new: Had diabetes since prev interview (self-report). 

notes sr_lung_dis_raw: disescn7
notes sr_lung_dis_raw: Had any lung disease, such as emphysema, asthma, or chronic bronchitis (self-report)
notes sr_lung_dis_ever: sr_lung_dis_raw
notes sr_lung_dis_ever: Had lung disease ever (self-report)
notes sr_lung_dis_new:  sr_lung_dis_raw
notes sr_lung_dis_new: Had lung disease since prev interview (self-report). 

notes sr_dementia_raw: disescn9
notes sr_dementia_raw: Had dementia or Alzheimer's disease (self-report)
notes sr_dementia_ever: sr_dementia_raw
notes sr_dementia_ever: Had dementia ever (self-report)
notes sr_dementia_new:  sr_dementia_raw
notes sr_dementia_new: Had dementia since prev interview (self-report). 

foreach x in sr_heart_dis_raw sr_heart_dis_ever sr_heart_dis_new sr_htn_raw sr_htn_ever ///
sr_htn_new sr_ra_raw sr_ra_ever sr_ra_new sr_osteoprs_raw sr_osteoprs_ever sr_osteoprs_new ///
sr_diabetes_raw sr_diabetes_ever sr_diabetes_new sr_lung_dis_raw sr_lung_dis_ever ///
sr_lung_dis_new sr_dementia_raw sr_dementia_ever sr_dementia_new{

notes `x': Located in NHATS Setup, under Part 4.
notes `x': Updated: 04/15/19-MH
}	

*************************************************
//Depression & Anxiety
//Over the last month how often have you:
//responses are 1=not at all, 2=several days, 3=more than half days, 4=nearly every day
*************************************************
gen sr_depr_pqq1=.
gen sr_depr_pqq2=.
gen sr_anx_gad1=.
gen sr_anx_gad2=.

forvalues w=1/$w{
	//set up new variables for individual wave variables
	foreach q in 1 2 3 4 {
	gen temp_hc`w'deprsan`q'=. if inlist(depresan`q',-9,-8,-7,-1) & wave==`w'
	replace temp_hc`w'deprsan`q'=depresan`q'-1 if inlist(depresan`q',1,2,3,4) & wave==`w'
	replace temp_hc`w'deprsan`q'=0 if inlist(depresan`q',-8,-7) & wave==`w'
	}
	//now create clean depression, anxiety variables
	replace sr_depr_pqq1=temp_hc`w'deprsan1 if wave==`w' //little interest or pleasure in doing things
	replace sr_depr_pqq2=temp_hc`w'deprsan2 if wave==`w' //felt down/depressed/hopeless
	replace sr_anx_gad1=temp_hc`w'deprsan3 if wave==`w' //felt nervous or anxious
	replace sr_anx_gad2=temp_hc`w'deprsan4 if wave==`w' //unable to stop worrying
	

	foreach q in 1 2 3 4 {
	drop temp_hc`w'deprsan`q'
	}
	
}

la def depr_anx 0"Not at all" 1"Several days" 2"More than half the days" ///
3"Nearly every day"
la val sr_depr_pqq1 sr_depr_pqq2 sr_anx_gad1 sr_anx_gad2 depr_anx
la var sr_depr_pqq1 "How often little interest in doing things?"
la var sr_depr_pqq2 "How often felt down, depressed, hopeless?"
la var sr_anx_gad1 "How often felt nervous or anxious?"
la var sr_anx_gad2 "How often unable to stop worrying?"

foreach v in sr_depr_pqq1 sr_depr_pqq2 sr_anx_gad1 sr_anx_gad2{
	tab `v' wave if ivw_type==1,missing
}

//combined phq-2 score for depression, cutoff 3+=depressed
gen sr_phq2_score=sr_depr_pqq1+sr_depr_pqq2
gen sr_phq2_depressed=1 if sr_phq2_score>=3 & sr_phq2_score~=.
replace sr_phq2_depressed=0 if sr_phq2_score<3 & sr_phq2_score~=.
la var sr_phq2_score "PHQ-2 combined score 0-6 scale"
la var sr_phq2_depressed "Depressed per PHQ-2, score=3+"
tab sr_phq2_score sr_phq2_depressed if ivw_type==1, missing

//combined gad-2 score for anxiety, cutoff 3+=anxiety
gen sr_gad2_score=sr_anx_gad1+sr_anx_gad2
gen sr_gad2_anxiety=1 if sr_gad2_score>=3 & sr_gad2_score~=.
replace sr_gad2_anxiety=0 if sr_gad2_score<3 & sr_gad2_score~=.
la var sr_gad2_score "GAD-2 combined score 0-6 scale"
la var sr_gad2_anxiety "Anxiety per GAD-2, score=3+"
tab sr_gad2_score sr_gad2_anxiety if ivw_type==1, missing
tab sr_gad2_anxiety wave, missing

notes sr_depr_pqq1: deprsan1
notes sr_depr_pqq1: Last month had little interest/pleasure in doing things.
notes sr_depr_pqq2: deprsan2
notes sr_depr_pqq2: Last month felt down/depressed/hopeless. 
notes sr_anx_gad1: deprsan3
notes sr_anx_gad1: Last month felt nervous/anxious/on edge. 
notes sr_anx_gad2: deprsan4
notes sr_anx_gad2: Last month how often been unable to stop/control worrying. 

notes sr_phq2_score: deprsan1 deprsan2
notes sr_phq2_score: sr_depr_pqq1 plus sr_depr_pqq2, measuring depression
notes sr_phq2_depressed: deprsan1 deprsan2
notes sr_phq2_depressed: sr_phq2_score >=3, depression indicator

notes sr_gad2_score: sr_anx_gad1 sr_anx_gad2
notes sr_gad2_score: sr_anx_gad1 plus sr_anx_gad2, measuring anxiety
notes sr_gad2_anxiety: sr_anx_gad1 sr_anx_gad2
notes sr_gad2_anxiety: sr_gad2_score >=3, anxiety indicator

foreach x in sr_depr_pqq1 sr_depr_pqq2 sr_anx_gad1 sr_anx_gad2 sr_phq2_score ///
sr_phq2_depressed sr_gad2_score sr_gad2_anxiety{
notes `x': Located in NHATS Setup, under Part 4.
notes `x': Updated: 04/15/19-MH
}	

*************************************************
//Dementia, based on NHATS technical paper no. 5, expanded for 3 waves
//separate variables for proxy vs self interviews
*************************************************
tab proxy_ivw ivw_type, missing

//Initialize 3 category dementia variable
//probable dementia, possible dementia, no dementia
//probable = diagnosis reported or 2+ ad8 questions (proxy) or <1.5 SD below mean
gen dem_3_cat=-1 if ivw_type!=1 //set to na if not SP interview
replace dem_3_cat=1 if sr_dementia_ever==1 //if reported directly

//for proxy interviews, use ad8 score
forvalues i = 1/8{
gen pr_ad8_`i'=-1 //initialize to n/a
replace pr_ad8_`i'=. if proxy_ivw==1 & dem_3_cat==. //set missing if proxy ivw w/o direct report dementia
forvalues w=1/$w{
	replace pr_ad8_`i'=1 if inlist(chgthink`i',1,3) & wave==`w' & dem_3_cat==. & proxy_ivw==1 //yes or dementia reported
	replace pr_ad8_`i'=0 if inlist(chgthink`i',2,-8) & wave==`w' & dem_3_cat==. &  proxy_ivw==1 //no or don't know
	}	
tab pr_ad8_`i' wave, missing
tab pr_ad8_`i' wave if proxy_ivw==1, missing
}

tab pr_ad8_3 chgthink3 if wave==1, missing
tab chgthink1 dad8dem if wave==2 & proxy_ivw==1, missing //previous proxy ivw ad8 items indicated dementia

tab pr_ad8_1 wave, missing
tab pr_ad8_1 dad8dem if wave==2 & proxy_ivw==1, missing //previous proxy ivw ad8 items indicated dementia
tab pr_ad8_1 sr_dementia_ever if wave==2 & proxy_ivw==1, missing // reported dementia directly

gen pr_ad8_score=(pr_ad8_1+pr_ad8_2+pr_ad8_3+pr_ad8_4+pr_ad8_5+pr_ad8_6+pr_ad8_7+pr_ad8_8) if proxy_ivw==1 & dem_3_cat==.
tab pr_ad8_score proxy_ivw, missing
tab pr_ad8_score wave, missing

gen dem_via_proxy=1 if pr_ad8_score>=2 & pr_ad8_score~=. //from ad8 score directly
replace dem_via_proxy=0 if inlist(pr_ad8_score,0,1)
replace dem_via_proxy=1 if (dad8dem==1 | dad8dem==1) & proxy_ivw==1 //if previous interview ad8 indicated dementia
replace dem_via_proxy=-1 if sr_dementia_ever==1
tab dem_via_proxy wave if proxy_ivw==1 & ivw_type==1, missing

//set 3 category dementia state variable
//first xwave variable for proxy allowed sp to answer cg section
/*2/14/19 Speaktosp already exists since removing prefixes. 
gen speaktosp=.
forvalues w=1/$w{
	replace speaktosp=cg`w'speaktosp if wave==`w'
}*/

replace dem_3_cat=1 if dem_3_cat==. & dem_via_proxy==1 //if proxy ivw indicates dem
replace dem_3_cat=3 if dem_3_cat==. & dem_via_proxy==0 & speaktosp==2  //proxy ind no dem
tab dem_3_cat wave, missing


notes dem_3_cat: sr_dementia_ever dem_via_proxy speaktosp
notes dem_3_cat: 3 Category dementia 

forvalues i=1/8{
notes pr_ad8_`i': chgthink`i'
}
notes pr_ad8_1: Change caused by memeory problems remembering month or year. (from proxy)
notes pr_ad8_2: Change caused by memeory problems repeating questions, stories, statements. (from proxy)
notes pr_ad8_3: Change caused by memeory problems amount of difficulty remembering appointments. (from proxy)
notes pr_ad8_4: Change caused by memeory problems amount of interest in hobbies/activities. (from proxy)
notes pr_ad8_5: Change caused by memeory problems handling money matters. (from proxy)
notes pr_ad8_6: Change caused by memeory problems trouble to use tools/gadgets/tv. (from proxy)
notes pr_ad8_7: Change caused by memeory problems, problems with judgement. (from proxy)
notes pr_ad8_8: Change caused by memeory problems, faily problems with thinking or memeory. (from proxy)
 

notes pr_ad8_score: chgthink*
notes pr_ad8_score: adding up pr_ad8_* (from proxy)

notes dem_via_proxy: chgthink* pr_ad8_* pr_ad8_score
notes dem_via_proxy: Dementia indicator from proxy. 

foreach x in dem_3_cat pr_ad8_1 pr_ad8_2 pr_ad8_3 pr_ad8_4 pr_ad8_5 pr_ad8_6 ///
pr_ad8_7 pr_ad8_8 pr_ad8_score dem_via_proxy{

notes `x': Located in NHATS Setup, under Part 4.
notes `x': Updated: 04/15/19-MH
}	

*************************************************
//for self interviews
*************************************************
//current date questions
tab todaydat1 speaktosp if wave==1 & ivw_type==1, missing

forvalues i = 1/4{
	gen date_item`i'=.
	forvalues w=1/$w{
		replace date_item`i'=1 if todaydat`i'==1 & wave==`w' //yes
		replace date_item`i'=0 if inlist(todaydat`i',2,-7) & wave==`w' //no, dont know, refused
	}
}
//count date questions correct
gen date_sum=date_item1+date_item2+date_item3+date_item4
tab date_sum wave, missing

local i=1
foreach x in Month Day Year "Day of Week"{
notes date_item`i': todaydat`i'
notes date_item`i': Did SP say correct `x'
notes date_item`i': Located in NHATS Setup, under Part 4.
notes date_item`i': Updated: 04/16/19-MH
local i=`i'+1
}

notes date_sum: todaydat1 todaydat2 todaydat3 todaydat4
notes date_sum: Count of date_item1-date_item4
notes date_sum: Located in NHATS Setup, under Part 4.
notes date_sum: Updated: 04/16/19-MH


//add categories for proxy says can't speak to sp or proxy says can speak but sp unble to answer
forvalues w=1/$w{
	replace date_sum=-2 if date_sum==. & speaktosp==2 & wave==`w'
	replace date_sum=-3 if (date_item1==. | date_item2==. | date_item3==. | ///
		date_item4==. ) & speaktosp==1 & wave==`w'
}
tab date_sum wave, missing

//new version date sum measure, without the extra categories
gen date_sumr=date_sum
replace date_sumr=. if date_sum==-2
replace date_sumr=0 if date_sum==-3

//president/vp items and count
capture program drop president 
program define president
	args newv oldv
	gen `newv'=.	
	forvalues w=1/$w{
		replace `newv'=1 if `oldv'==1  & wave==`w' //yes
		replace `newv'=0 if inlist(`oldv',2,-7) & wave==`w' //no, refused
	}
	tab `newv' wave, missing
end

president preslast presidna1	
president presfirst presidna3
president vplast vpname1
president vpfirst vpname3

//count president questions correct
gen presvp=preslast+presfirst+vplast+vpfirst
tab presvp wave, missing



//add categories for proxy says can't speak to sp or proxy says can speak but sp unble to answer
forvalues w=1/$w{
	replace presvp=-2 if presvp==. & speaktosp==2 & wave==`w'
	replace presvp=-3 if presvp==. & (preslast==. | presfirst==. | vplast==. | ///
		vpfirst==. ) & speaktosp==1 & wave==`w'
}
tab presvp wave, missing

//new version president sum measure, without the extra categories
gen presvpr=presvp
replace presvpr=. if presvp==-2
replace presvpr=0 if presvp==-3

//total orientation domain, date and president questions
gen date_prvp=date_sumr+presvpr
tab date_prvp, missing

//executive function domain, clock drawing score
gen clock_scorer=.
forvalues w=1/$w{
	replace clock_scorer=dclkdraw if wave==`w'
	replace clock_scorer=. if inlist(dclkdraw,-2,-9) & wave==`w' //missing
	replace clock_scorer=0 if inlist(dclkdraw,-3,-4,-7) & wave==`w' //refused,unable
//assign mean values where missing response
	replace clock_scorer=2 if dclkdraw==-9 & speaktosp==1 & wave==`w' //proxy
	replace clock_scorer=3 if dclkdraw==-9 & speaktosp==-1 & wave==`w' //self
	}
tab clock_scorer wave, missing //left as -1 if inapplicable

//memory domain, word recall, uses derived scores, not raw responses
capture program drop recall
program define recall
	args newv oldv
	gen `newv'=.
	forvalues w=1/$w{
		replace `newv'=`oldv' if wave==`w'
		replace `newv'=. if inlist(`oldv',-2,-1) & wave==`w' //n/a or not asked
		replace `newv'=0 if inlist(`oldv',-7,-3) & wave==`w' //refused or unable
	}
	tab `newv' wave, missing
end

recall irecall dwrdimmrc
recall drecall dwrddlyrc

gen wordrecall0_20=irecall+drecall

//overall cognitive domain score, assigns cutoffs for each of the domain variables
gen clock65=0 if clock_scorer>1 & clock_scorer<=5
replace clock65=1 if inlist(clock_scorer,0,1)

gen word65=0 if wordrecall0_20>3 & wordrecall0_20<=20
replace word65=1 if inlist(wordrecall0_20,0,1,2,3)

gen datena65=0 if date_prvp>3 & date_prvp<=8
replace datena65=1 if inlist(date_prvp,0,1,2,3)

gen domain65=clock65+word65+datena65
tab domain65 wave, missing

//now apply to dementia 3 category variable
replace dem_3_cat=1 if dem_3_cat==. & inlist(speaktosp,1,-1) & inlist(domain65,2,3)
replace dem_3_cat=2 if dem_3_cat==. & inlist(speaktosp,1,-1) & domain65==1
replace dem_3_cat=3 if dem_3_cat==. & inlist(speaktosp,1,-1) & domain65==0

la def dem3 1"Probable dementia" 2"Possible dementia" 3"No dementia"
la val dem_3_cat dem3
la var dem_3_cat "Dementia likelihood, 3 categories"

tab dem_3_cat wave, missing

gen dem_2_cat=1 if inlist(dem_3_cat,1,2)
replace dem_2_cat=0 if dem_3_cat==3
la var dem_2_cat "Ind prob/possible dementia"
tab dem_2_cat wave, missing
*************************************************
//rate memory
gen memory=.
forvalues w=1/$w{
replace memory=ratememry if wave==`w' & proxy_ivw==0 //self interview
replace memory=memrygood if wave==`w' & proxy_ivw==1 //proxy interview

}
replace memory=. if inlist(memory,-1)
la var memory "Rate memory 1=Excell, 5=Poor"
la def mem 1"Excellent" 2"Very good" 3"Good" 4"Fair" 5"Poor"
la val memory mem
tab memory wave, missing

notes memory: ratememry memrygood
notes memory: Rate your memory.
notes memory: Located in NHATS Setup, under Part 4.
notes memory: Updated: 04/17/19-MH
*************************************************
//total number self reported medical conditions, ever
//note that depression and anxeity are from the specific interview questions
//the others are all if the person has reported the condition in any interview wave
egen sr_numconditions1=anycount(sr_ami_ever sr_stroke_ever sr_cancer_ever ///
	sr_hip_ever sr_heart_dis_ever sr_htn_ever sr_ra_ever sr_osteoprs_ever ///
	sr_diabetes_ever sr_lung_dis_ever sr_dementia_ever sr_phq2_depressed ///
	sr_gad2_anxiety), values(1)
replace sr_numconditions1=. if ivw_type!=1
la var 	sr_numconditions1 "Count medical conditions"
tab sr_numconditions1 wave, missing
notes sr_numconditions1: Counting number of medical conditions. 
notes sr_numconditions1: Located in NHATS Setup, under Part 4.
notes sr_numconditions1: Updated: 04/17/19-MH

local ivars srh srh_fp sr_ami_raw sr_ami_ever sr_ami_new sr_stroke_raw sr_stroke_ever sr_stroke_new sr_cancer_raw sr_cancer_ever sr_cancer_new ///
sr_hip_raw sr_hip_ever sr_hip_new sr_heart_dis_raw sr_heart_dis_ever sr_heart_dis_new sr_htn_raw sr_htn_ever sr_htn_new sr_ra_raw ///
sr_ra_ever sr_ra_new sr_osteoprs_raw sr_osteoprs_ever sr_osteoprs_new sr_diabetes_raw sr_diabetes_ever sr_diabetes_new sr_lung_dis_raw ///
sr_lung_dis_ever sr_lung_dis_new sr_dementia_raw sr_dementia_ever sr_dementia_new
foreach w in `ivars'{
tab `w'
}

*************************************************
save round_1_${w}_cleanv2.dta, replace

*************************************************
log close

translate "${logs}/`name'.smcl" "${logs}/`name'.pdf"
exit
```

## NHATS Data Cleaning Part 5 (4_r12_cleaning_3_1)
```

/*

Created by: --
Date Created: --

Updated by: MH
Date Updated: 07/02/2019

Description: This section uses the SP and OP data files and imputes
hours of help provided to SP per methods in NHATS Technical Paper #7

Variables from this dataset are then brought in to the SP Rounds dataset
and also the NSOC caregiver dataset.



**************************************************
*/

local date = subinstr("$S_DATE"," ","_",.) 
local name 4_nhats_cleaning_`date'
di "`name'"

capture log close 
clear all

set more off
version 12
set linesize 80
set maxvar 10000


log using "${logs}/`name'.smcl", text replace



cd "${work}"

*********************************************

use round_1_${w}_cleanv2.dta

*********************************************
//fall in the last month
gen fall_last_month=.
forvalues w=1/$w{
	replace fall_last_month=1 if fllsinmth==1 & wave==`w'
	replace fall_last_month=0 if fllsinmth==2 & wave==`w'
	replace fall_last_month = -7 if fllsinmth==-7 & wave==`w'
	replace fall_last_month = -8 if fllsinmth==-8 & wave==`w'
}
la var fall_last_month "Fall down in last month"
tab fall_last_month wave, missing
notes fall_last_month: fllsinmth
notes fall_last_month: Fallen down in last month.
notes fall_last_month: Located in NHATS Setup, under Part 5.
notes fall_last_month: Updated: 04/17/19-MH

*********************************************
//self care help, using the derived variables in the SC section
//uses the "has help" variables (has difficulty, uses devices other options)
//if don't know, refused, or not done, set to missing
tab deathelp, missing
rename eatslfdif eatdif
rename eatslfoft eatoft
capture program drop selfcare
program define selfcare
	args act
	gen adl_`act'_diff=.
	gen adl_`act'_help=.
	gen adl_`act'_not_done=.
	forvalues w=1/$w{
		replace adl_`act'_help=1 if `act'hlp==1 & wave==`w' //yes
		replace adl_`act'_help=0 if `act'hlp==2 & wave==`w' //no
		replace adl_`act'_help=0 if `act'hlp==-1 & sp_ivw==1 & wave==`w' //bias to null
		replace adl_`act'_help=-7 if `act'hlp==-7 & wave==`w' //refused
		replace adl_`act'_help=-8 if `act'hlp==-8 & wave==`w' //DK
		replace adl_`act'_diff=`act'dif>1 if `act'dif>0 & wave==`w'
		replace adl_`act'_not_done=`act'oft==4 if `act'oft>0 & wave==`w'
}
	tab adl_`act'_help wave, missing
	tab adl_`act'_help wave if ivw_type==1, missing
	*replace adl_`act'_diff=1 if adl_`act'_help==1
	
end	
	
selfcare eat
selfcare bath
selfcare toil
selfcare dres

notes adl_eat_not_done: eatslfoft

foreach x in bath toil dres{
notes adl_`x'_not_done: `x'oft
}
foreach x in eat bath toil {
notes adl_`x'_not_done: How often `x' by self and without help.
}
notes adl_dres_not_done: How often you dress.  

foreach x in eat bath toil dres{
notes adl_`x'_not_done: Located in NHATS Setup, under Part 5.
notes adl_`x'_not_done: Updated: 04/17/19-MH
}

la var adl_eat_help "Has help while eating"
la var adl_bath_help "Has help while bathing"
la var adl_toil_help "Has help while toileting"
la var adl_dres_help "Has help while dressing"
*********************************************
//household activity help, using derived variables in the HA section
//if did not do by self b/c of health
//reason only or health and other reason, then new help var ==1
tab dlaunsfdf dlaunreas if wave==1, missing
tab dlaunsfdf dlaunreas if wave==2, missing

capture program drop hhact
program define hhact
	args act
	
	gen iadl_`act'_help=.
	gen iadl_`act'_diff=.
	gen iadl_`act'_not_done=.
	forvalues w=1/$w{
		replace iadl_`act'_help=1 if inlist(d`act'sfdf,1,8) & ///
			inlist(d`act'reas,1,3) & wave==`w'
		replace iadl_`act'_help=0 if inlist(d`act'sfdf,2,3,6) & wave==`w' //did by self
		replace iadl_`act'_help=0 if inlist(d`act'sfdf,1,8) & ///
			inlist(d`act'reas,2,4) & wave==`w'		
		replace iadl_`act'_help=-8 if inlist(d`act'sfdf,4,5,7) & wave==`w' //dkrf
		replace iadl_`act'_help=-8 if inlist(d`act'sfdf,1,8) & ///
			inlist(d`act'reas,-7,-8) & wave==`w'
		replace iadl_`act'_diff=`act'dif>1 if `act'dif>0 & wave==`w'
		replace iadl_`act'_not_done=`act'==5 if `act'>0 & wave==`w'
	}	
	tab iadl_`act'_help wave, missing
	tab iadl_`act'_help wave if ivw_type==1, missing
	*replace iadl_`act'_diff=1 if iadl_`act'_help==1
	end
	
hhact laun
hhact shop
hhact meal
hhact bank

la var iadl_laun_help "Rec'd help doing laundry last month"
la var iadl_shop_help "Rec'd help shopping last month"
la var iadl_meal_help "Rec'd help preparing meals last month"
la var iadl_bank_help "Rec'd help with banking last month"

foreach x in laun shop meal bank{ 
notes iadl_`x'_help: d`x'sfdf d`x'reas
notes iadl_`x'_help: `x' difficulty level and if `x' done by others
notes iadl_`x'_help: Located in NHATS Setup, under Part 5.
notes iadl_`x'_help: Updated: 04/17/19-MH
notes iadl_`x'_not_done: ha*`x'
notes iadl_`x'_not_done: How `x' made/completed.
notes iadl_`x'_not_done: Located in NHATS Setup, under Part 5.
notes iadl_`x'_not_done: Updated: 04/17/19-MH
}

//taking medications, questions slightly different and in mc section
//set to =0 if do not take medications
tab dmedssfdf dmedsreas if wave==1, missing
tab dmedssfdf dmedsreas if wave==2, missing

gen iadl_meds_help=.
gen iadl_meds_diff=.
gen iadl_meds_not_done=.
forvalues w=1/$w{
	replace iadl_meds_help=1 if inlist(dmedssfdf,1,8) & ///
		inlist(dmedsreas,1,3) & wave==`w'
	//did by self or don't take medications
	replace iadl_meds_help=0 if inlist(dmedssfdf,2,3,6,9) & wave==`w' 
	replace iadl_meds_help=0 if inlist(dmedssfdf,1,8) & ///
		inlist(dmedsreas,2,4) & wave==`w'		
	replace iadl_meds_help=-8 if dmedssfdf == 7 & wave==`w' 
	replace iadl_meds_help=-8 if inlist(dmedssfdf,1,8) & ///
		inlist(dmedsreas,-7,-8) & wave==`w'
	replace iadl_meds_diff=medsdif>1 if medsdif>0 & wave==`w'
	replace iadl_meds_not_done=inlist(dmedssfdf,1,8) if wave==`w'
}	

la var iadl_meds_help "Rec'd help taking medications last month"
tab iadl_meds_help wave, missing
tab iadl_meds_help wave if ivw_type==1, missing
replace iadl_meds_diff=1 if iadl_meds_help==1

notes iadl_meds_help: dmedssfdf dmedsreas
notes iadl_meds_help: Difficulty level of medications, medicine reason with others. 

notes iadl_meds_diff: medsdif
notes iadl_meds_diff: Difficulty in keeping track of medicines.
   
notes iadl_meds_not_done: dmedssfdf
notes iadl_meds_not_done: Difficulty level of medications

foreach x in help diff not_done{
notes iadl_meds_`x': Located in NHATS Setup, under Part 5.
notes iadl_meds_`x': Updated: 04/17/19-MH
}

//other meds questions, for treatment burden
gen ind_meds_diff=.
gen ind_meds_mistake=.
forvalues w=1/$w{
replace ind_meds_diff=inlist(medsdif,2,3,4) if medsdif>0 & !missing(medsdif) & wave==`w'
replace ind_meds_diff=0 if meds==2 & wave==`w'
replace ind_meds_diff=0 if dmedsrea==2 & inlist(medstrk,2,3) & wave==`w'
replace ind_meds_mistake=medsmis==1 if medsmis>0 & !missing(medsmis) & wave==`w'
}
label var ind_meds_diff "Any difficulty keeping track of meds"
label var ind_meds_mistake "Any mistake taking meds"
gen ind_meds_problem=ind_meds_diff==1 | ind_meds_mistake==1 if !missing(ind_meds_diff) ///
| !missing(ind_meds_mistake)
label var ind_meds_prob "SR difficulty keeping track of or mistake taking meds"
*********************************************
//has , sees regular doctor
gen reg_doc_have = .
forvalues w=1/$w{
	replace reg_doc_have=1 if havregdoc==1 & wave==`w'
	replace reg_doc_have=0 if havregdoc==2 & wave==`w'
	replace reg_doc_have=-7 if havregdoc==-7 & wave==`w'
	replace reg_doc_have=-8 if havregdoc==-8 & wave==`w'
}
la var reg_doc_have "Have regular doctor"
tab reg_doc_have wave, missing
tab reg_doc_have wave if ivw_type==1, missing

notes reg_doc_have: haveregdoc
notes reg_doc_have: Do you have a go to doctor, who you go to for advice.

gen reg_doc_seen = .
forvalues w=1/$w{
	replace reg_doc_seen=1 if regdoclyr==1 & wave==`w'
	replace reg_doc_seen=0 if regdoclyr==2 & wave==`w'
	replace reg_doc_seen=-7 if regdoclyr==-7 & wave==`w'
	replace reg_doc_seen=-8 if regdoclyr==-8 & wave==`w'
}
la var reg_doc_seen "Seen regular doctor within last year"
tab reg_doc_seen wave, missing
tab reg_doc_seen wave if ivw_type==1, missing

notes reg_doc_seen: regdoclyr
notes reg_doc_seen: Have you seen your regular doctor within the last year. 

//home visit, missing if did not see regular doctor within the last year
tab regdoclyr hwgtregd8 if wave==1, missing

gen reg_doc_homevisit = .
forvalues w=1/$w{
	replace reg_doc_homevisit=1 if hwgtregd8==1 & wave==`w'
	replace reg_doc_homevisit=0 if hwgtregd8==2 & wave==`w'
	replace reg_doc_homevisit=-7 if hwgtregd8==-7 & wave==`w'
	replace reg_doc_homevisit=-8 if hwgtregd8==-8 & wave==`w'
	//replace reg_doc_homevisit=0 if mc`w'regdoclyr==2
}
la var reg_doc_homevisit "Was regular doctor visit a home visit?"
tab reg_doc_homevisit wave, missing
tab reg_doc_homevisit wave if ivw_type==1, missing

notes reg_doc_homevisit: hwgtregd8
notes reg_doc_homevisit: Was doctor visit a home visit. 

foreach x in reg_doc_have reg_doc_seen reg_doc_homevisit{
notes `x': Located in NHATS Setup, under Part 5.
notes `x': Updated: 04/17/19-MH
}

//hospital stay last 12 months?
tab hosptstay, missing

gen sr_hosp_ind=.
forvalues w=1/$w{
	replace sr_hosp_ind=1 if hosptstay==1 & wave==`w'
	replace sr_hosp_ind=0 if hosptstay==2 & wave==`w'
	replace sr_hosp_ind=-7 if hosptstay==-7 & wave==`w'
	replace sr_hosp_ind=-8 if hosptstay==-8 & wave==`w'
}
la var sr_hosp_ind "Hospital stay in the last year 1=yes"
tab sr_hosp_ind wave, missing
tab sr_hosp_ind wave if ivw_type==1, missing

notes sr_hosp_ind: hosptstay
notes sr_hosp_ind: Had an overnight hospital stay since last interview/year.

//number of separate hospital stays?
tab hosovrnht hosptstay, missing //if ind=no above, raw variable is n/a
gen sr_hosp_stays=.
forvalues w=1/$w{
	replace sr_hosp_stays=hosovrnht if hosovrnht>0 & wave==`w'
}
replace sr_hosp_stays=0 if sr_hosp_ind==0 //set to 0 if none reported in y/n ? above
la var sr_hosp_stays "Number unique hospital stays last year"
tab sr_hosp_stays wave, missing
tab sr_hosp_stays wave if ivw_type==1, missing

notes sr_hosp_stays: hosovrnht hosptstay
notes  sr_hosp_stays: How many seperate overnight hospital stays. 

foreach x in sr_hosp_ind sr_hosp_stay{
notes `x': Located in NHATS Setup, under Part 5.
notes `x': Updated: 04/17/19-MH
} 

save round_1_${w}_cleanv3.dta, replace

local ivars fall_last_month adl_eat_help adl_bath_help adl_toil_help adl_dres_help iadl_laun_help iadl_shop_help ///
iadl_meal_help iadl_bank_help iadl_meds_help reg_doc_have reg_doc_seen reg_doc_homevisit sr_hosp_ind sr_hosp_stays
foreach w in `ivars'{
tab `w' wave,missing
}

*********************************************
log close


translate "${logs}/`name'.smcl" "${logs}/`name'.pdf"
exit
```


## NHATS Data Cleaning Part 6 (5_r12_add_helper)

```
/*

Created by: --
Date Created: --

Updated by: MH
Date Updated: 07/02/2019

Description: 

Collapse helper information to the SP - wave level
and merge into SP dataset

variables created are:
Total # helpers for the SP
Total hours/month help provided for SP
Indicators for relationship of primary helper (identified as helper 
with the most hours)

Uses imputed hours per code 1a_help_hours_imputation.do
(see NHATS Technical Paper #7 for imputation methodology)



**************************************************
*/

local date = subinstr("$S_DATE"," ","_",.) 
local name 5_nhats_cleaning_`date'
di "`name'"

capture log close 
clear all

set more off
version 12
set linesize 80
set maxvar 10000


log using "${logs}/`name'.smcl", text replace




*********************************************

cd "${work}"
***************************************************************
//get all 3 waves helper imputed hours into a single dataset

use R1_hrs_imputed_added.dta

local keepvars spid wave opid op_hrsmth_i numact yearnotmonth disab ///
 justone spouse otherrel nonrel regular oneact impute_cat opishelper_allw paid ///
 op_adl* op_iadl* op_nonadl* son daughter oth_fam rc_staff op_doc_help op_sn op_age

forvalues w=2/$w{
qui append using R`w'_hrs_imputed_added.dta
 
}
 
tab wave opishelper_allw, missing

//  age categories
egen op_age=rowmax(dage)
label val op_age dage 
replace op_age=. if inlist(op_age, -8, -7, -1, .) 



//identify paid helpers and staff at residence (not helpers)
gen paid=0
gen rc_staff=0
forvalues w=1/$w {
	replace paid=1 if paidh==1 & ((relat>=30 & relat<=40)| ///
	relat==92) & wave==`w'
	foreach x of varlist hlp *hlp1 *hlp2 *hlpmst {
		replace rc_staff=1 if relat==37 & `x'==1 & wave==`w'
}
}

gen op_sn=0 
forvalues w=1/$w {
	replace op_sn=socl==1 if wave==`w'
}
label var op_sn "OP in SP's social network"

keep `keepvars'

replace opishelper_allw=0 if opishelper_allw==-1
replace op_hrsmth_i=. if op_hrsmth_i==-1
***************************************************************
//identify primary helper, person with the most hours / month using imputed hours
sort spid wave opid spouse

gen primary_cg=0
egen max_hrs_helped=max(op_hrsmth_i) if opishelper_allw==1, by(spid wave)
replace primary_cg=1 if op_hrsmth_i==max_hrs_helped & opishelper_allw==1
tab primary_cg wave, missing

//check that each sp has 1 primary cg identified, doesn't matter!
egen num_primary_cg=total(primary_cg), by(spid wave)
tab num_primary_cg wave, missing

//assign relationship to primary helper
//if same number of hours for two different helpers, then assign to spouse first
gen prim_helper_cat=1 if primary_cg==1 & spouse==1
replace prim_helper_cat=2 if primary_cg==1 & (son==1 | daughter==1)
replace prim_helper_cat=3 if primary_cg==1 & otherrel==1 & prim_helper_cat==.
replace prim_helper_cat=4 if prim_helper_cat==.
la def helpcat 1"Spouse" 2 "Child" 3 "Other relative" 4 "Other, not relative"
la val prim_helper_cat helpcat
tab prim_helper_cat wave, missing

//get categorical relationship
gen op_relationship_cat=6
local i=1
foreach x in spouse daughter son oth_fam paid {
	replace op_relationship_cat=`i' if `x'==1
	local i=`i'+1
}
label define op_relationship_cat 1 "Spouse/Partner" 2 "Daughter (incl. step-/-in-law)" ///
3 "Son (incl. step-/-in-law)" 4 "Other family" 5 "Paid CG" 6 "Unpaid non-family CG"
label values op_relationship_cat op_relationship_cat

//get total number hours/month help received and number of helpers per spid/wave
egen tot_hrsmonth_help_i=total(op_hrsmth_i), by(spid wave)
egen n_helpers=count(op_hrsmth_i), by(spid wave)
gen tot_hrswk_help_i=tot_hrsmonth_help_i/(52/12)
//family caregivers
gen op_family=op_relationship_cat<=4 if opishelper==1
label var op_family "OP is family helper"
egen n_family_helpers=total(op_family),by(spid wave)
label var n_family_helpers "Number family helpers"
gen ind_family_helper=n_family_helpers>0
label var ind_family_helper "Indicatory any family helpers"
//paid caregivers
egen n_paid_helpers=total(paid), by(spid wave)
gen ind_paid_helper=n_paid_helpers>0 
label var n_paid_helpers "Number paid helpers"
label var ind_paid_helper "Indicator any paid helpers"
egen ind_rc_staff=max(rc_staff), by(spid wave)
label var ind_rc_staff "Indicator Facility Staff in OP file"
gen num_helpers_cat=0
replace num_helpers_cat=1 if n_helpers>0 & n_helpers<.
replace num_helpers_cat=2 if n_helpers>3 & n_helpers<.
replace num_helpers_cat=3 if n_helpers>=7 & n_helpers<.
label define num_helpers_cat 0 "No helpers" 1 "1-3 helpers" 2 "4-6 helpers" ///
3 "7+ helpers"
label values num_helpers_cat num_helpers_cat

gen phrs=op_hrsmth_i if paid==1
egen tot_hrsmonth_paid_i=total(phrs), by(spid wave)
gen tot_hrswk_paid_i=tot_hrsmonth_paid_i/(52/12)
label var tot_hrsmonth_paid_i "Hours/month help recived, paid helpers, imputed"
label var tot_hrswk_paid_i "Hours/week help received, paid helpers, imputed"
gen op_hrswk_i=op_hrsmth_i/(52/12)
label var op_hrswk_i "OP hours/week help SP"

la var tot_hrsmonth_help_i "Hours / month help received, all helpers, imputed"
sum tot_hrsmonth_help_i, detail
la var tot_hrswk_help_i "Hours / week help received, all helpers, imputed"

la var n_helpers "Number helpers reported by SP"
tab n_helpers wave, missing
la var prim_helper_cat "Primary helper relationship, missing if no helper"
tab prim_helper_cat wave, missing

la var tot_hrsmonth_help_i "Hours / month help received, all helpers, imputed"
sum tot_hrsmonth_help_i, detail
la var tot_hrswk_help_i "Hours / week help received, all helpers, imputed"

la var n_helpers "Number helpers reported by SP"
tab n_helpers wave, missing
la var prim_helper_cat "Primary helper relationship, missing if no helper"
tab prim_helper_cat wave, missing


//number in social network
by spid wave, sort: egen n_sn=total(op_sn)
replace n_sn=0 if missing(n_sn)
label var n_sn "Number in social network"

//save as op file before dropping down to one obs per spid
saveold "D:\NHATS\Shared\base_data\NHATS cleaned\op_round_1_${w}.dta", replace version(12)

//keep the entry for the primary caregiver only, this will then be the 
//information merged into the SP dataset
keep if primary_cg==1

//sort and keep only first primary cg, again preference given if spouse
gsort spid wave -spouse otherrel opid
duplicates drop spid wave, force

//now one obs per sp - wave

sum tot_hrsmonth_help_i, detail
tab n_helpers wave, missing
tab prim_helper_cat wave, missing
***************************************************************
//merge this helper data with the main SP dataset
save helper_by_sp.dta, replace

use round_1_${w}_cleanv3, clear

merge 1:1 spid wave using helper_by_sp.dta, keepusing(opid n_helpers n_fam ind_fam ///
 tot_hrsmonth_* prim_helper_cat ind_pa n_pai tot_hrswk* num_helpers_cat ind_rc_*)

drop _merge 

foreach x in n_helpers ind_pa n_paid num_helpers_cat{
replace `x'=0 if `x'==.
}

gen ind_no_helpers=.
replace ind_no_helpers=1 if n_helpers==0
replace ind_no_helpers=0 if n_helpers>0
la var ind_no_helpers "Indicator no Helpers reported"
tab ind_no_helpers wave, missing

la var tot_hrsmonth_help_i "Hours / month help received, all helpers, imputed"
sum tot_hrsmonth_help_i, detail
la var tot_hrswk_help_i "Hours / week help received, all helpers, imputed"

la var n_helpers "Number helpers reported by SP"
tab n_helpers wave, missing
la var prim_helper_cat "Primary helper relationship, missing if no helper"
tab prim_helper_cat wave, missing

rename opid prim_helper_opid
destring prim_helper_opid, replace
la var prim_helper_opid "Primary helper OPID"

***************************************************************
//save the dataset with helper information added

save round_1_${w}_clean_helper_added.dta, replace

saveold round_1_${w}_clean_helper_added_old.dta, replace

***************************************************************
log close


translate "${logs}/`name'.smcl" "${logs}/`name'.pdf"
exit

```

## NHATS Data Cleaning Part 7--Variable Construction

```

/*

Created by: --
Date Created: --

Updated by: MH
Date Updated: 05/26/2019

Description: This section uses the SP and OP data files and imputes
hours of help provided to SP per methods in NHATS Technical Paper #7


Collapse helper information to the SP - wave level
and merge into SP dataset

variables created are:
Total # helpers for the SP
Total hours/month help provided for SP
Indicators for relationship of primary helper (identified as helper 
with the most hours)

Uses imputed hours per code 1a_help_hours_imputation.do
(see NHATS Technical Paper #7 for imputation methodology)

Bring in day of month from restricted tracker


**************************************************
*/

local date = subinstr("$S_DATE"," ","_",.) 
local name 6_nhats_cleaning_`date'
di "`name'"

capture log close 
clear all

set more off
version 12
set linesize 80
set maxvar 10000


log using "${logs}/`name'.smcl", text replace



*********************************************


//Pull dates from restricted tracker
use "D:\NHATS\Shared\raw\NHATS\NHATS Restricted\NHATS Round 7-8 Tracker Restricted STATA\NHATS_Round_8_Tracker_Restricted.dta", clear
keep spid r*spstatdtdy
rename r*spstatdtdy spstatdtdy*
reshape long spstatdtdy, i(spid) j(wave)
tempfile resttrack
save `resttrack'

//Does not need to change, unless get new metro file. 

//7/13/17--add in metro/nonmetro var
use "D:\NHATS\Shared\raw\NHATS\NHATS Public\NHATS_Round_1_8_MetNonmet_STATA\NHATS_Round_1_8_MetNonmet.dta", clear
forvalues w=1/8 {
gen metro_ind`w'=r`w'd==1 if inrange(r`w'd,1,2)
}
reshape long metro_ind, i(spid) j(wave)
label var metro_ind "Lives in metropolitan area"
note metro_ind: From metro non-metro file, r(wave)dmetnomet raw variable. 
note metro_ind: Metropolitan area. 
label define metro_ind  0 "Non-Metro" 1 "Metro" 
label values metro_ind metro_ind

tempfile metro
save `metro'

cd "${work}"

use round_1_${w}_clean_helper_added.dta
merge 1:1 spid wave using `metro', keep(match master) nogen





// 10/17/19
rename rehab rehabil

gen rehab=.
replace rehab=0 if rehabil==2
replace rehab=1 if rehabil==1
label var rehab "Went to Rehabilitation facility"

note rehab: rehab
note rehab: Last year recieved rehab services. Only asked after round 5. 
note rehab: Located in NHATS Setup, under Part 7.
note rehab: Created: 12/17/19-MH   

gen surgery=.
replace surgery=0 if knesrgyr==0 & hipsrgyr==0 & catrsrgyr==0 & backsrgyr==0 & hartsrgyr==0 
replace surgery=1 if knesrgyr==1 | hipsrgyr==1 | catrsrgyr==1 | backsrgyr==1 | hartsrgyr==1 
label var surgery "Had surgery (from specified list)"

note surgery: knesrgyr hipsrgyr catrsrgyr backsrgyr hartsrgyr
note surgery: Use any of the surgery's listed in the past year to make variable. 
note surgery: Located in NHATS Setup, under Part 7.
note surgery: Created: 12/17/19-MH   

// 5/1/2019 P01 Aldridge

gen unmet_relg_lml=.
replace unmet_relg_lml=1 if relg==2
replace unmet_relg_lml=0 if relg==1
label var unmet_relg_lml "Unmet Spiritual need, LML"

note unmet_relg_lml: relg
note unmet_relg_lml: Doctors, nurses, or other health care professionals talk ///
about religious beliefs. 
note unmet_relg_lml: Located in NHATS Setup, under Part 7.
note unmet_relg_lml: Created: 5/1/19-MH  
// 3/21/2019--Adding LM questions from LML interview section, P01 Aldridge

gen qoc_lml_e=.
replace qoc_lml_e=1 if ratecare==1
replace qoc_lml_e=0 if inrange(ratecare,2,5)
note qoc_lml_e: ratecare (1=1)
note qoc_lml_e: Quality of Care rated as excellent. Coming from LM11 (Waves 2+) 
note qoc_lml_e: Located in NHATS Setup, under Part 7.
note qoc_lml_e: Updated: 4/16/19-MH Created: 3/21/19-MH  
label var qoc_lml_e "Quality of Care rated excellent, LML"
/*
gen qoc_lml=ratecare
replace qoc_lml=. if inlist(ratecare,-8,-7,-1,6)
*/

gen relat_proxy_cat=1 if prxyrelat==2
replace relat_proxy_cat=2 if inlist(prxyrelat,3,5,7)
replace relat_proxy_cat=3 if inlist(prxyrelat,4,6,8)
replace relat_proxy_cat=4 if inrange(prxyrelat,9,29) | prxyrelat==91
replace relat_proxy_cat=5 if prxyrelat>=30 & prxyrelat!=91

label define relat_proxy_cat 1 "Spouse" 2 "Daughter (incl. step/in-law)" 3 "Son (incl. step/in-law)" 4 "Other relative" 5 "Nonrelative"
label values relat_proxy_cat relat_proxy_cat
label var relat_proxy_cat "Proxy relationship to SP"

note relat_proxy_cat: prxyrelat
note relat_proxy_cat: Proxy relationship to SP
note relat_proxy_cat: Located in NHATS Setup, Part 7
note relat_proxy_cat: Created 11/20/20, EBL

gen familiar_proxy_cat=famrrutin if inrange(famrrutin,1,4)
gen familiar_proxy_ind=familiar_proxy_cat==1 if !missing(familiar_proxy_cat)

label define familiar_proxy_cat 1 "Very" 2 "Somewhat" 3 "A little" 4 "Not at all"
label values familiar_proxy_cat familiar_proxy_cat
label var familiar_proxy_cat "Proxy familiarity with SP's daily routine, categorical"
label var familiar_proxy_ind "Proxy very familiar with SP's daily routine"

note familiar_proxy_cat: famrrutin
note familiar_proxy_cat: Proxy familiarity with SP's daily routine
note familiar_proxy_cat: Located in NHATS Setup, Part 7
note familiar_proxy_cat: Created 11/20/20, EBL 

note familiar_proxy_ind: famrrutin
note familiar_proxy_ind: Proxy familiarity with SP's daily routine, binary at very/other
note familiar_proxy_ind: Located in NHATS Setup, Part 7
note familiar_proxy_ind: Created 11/20/20, EBL 


foreach x in pain bre sad{
gen unmet_`x'_lml=.
replace unmet_`x'_lml=0 if inrange(`x', 1, 2)
replace unmet_`x'_lml=1 if (`x'==1 & `x'hlp==2) | (`x'==1 & `x'hlp==1 & `x'hlpam==1)  
}
note unmet_pain_lml: pain painhlp painhlpam
note unmet_pain_lml: Unmet needs for pain management. Coming from LM1, LM1A, LM1B (Waves 2+) 
note unmet_pain_lml: Located in NHATS Setup, under Part 7.
note unmet_pain_lml: Updated: 4/16/19-MH Created: 3/21/19-MH  
label var unmet_pain_lml "Unmet need for pain management, LML"

note unmet_bre_lml: bre brehlp brehlpam
note unmet_bre_lml: Unmet needs for dyspnea (breathing) management. Coming from LM2, LM2A, LM2B (Waves 2+) 
note unmet_bre_lml: Located in NHATS Setup, under Part 7.
note unmet_bre_lml: Updated: 4/16/19-MH Created: 3/21/19-MH  
label var unmet_bre_lml "Unmet need for dyspnea (breathing) management, LML"

note unmet_sad_lml: sad sadhlp sadhlpam
note unmet_sad_lml: Unmet needs for sadness (anxiety) management. Coming from LM3, LM3A, LM3B (Waves 2+) 
note unmet_sad_lml: Located in NHATS Setup, under Part 7.
note unmet_sad_lml: Updated: 4/16/19-MH Created: 3/21/19-MH  
label var unmet_sad_lml "Unmet need for sadness (anxiety) management, LML"

foreach x in respect informed{
gen `x'_lml=.
replace `x'_lml=0 if inrange(`x', 1, 4)
replace `x'_lml=1 if inrange(`x', 2,4)
}

note respect_lml: respect
note respect_lml: Not always treated with respect. Coming from LM7 (Waves 2+) 
note respect_lml: Located in NHATS Setup, under Part 7.
note respect_lml: Updated: 4/16/19-MH Created: 3/21/19-MH    
label var respect_lml "Not always treated with respect, LML"

note informed_lml: informed
note informed_lml: Family not always informed for individuals condition. Coming from LM8 (Waves 2+) 
note informed_lml: Located in NHATS Setup, under Part 7.
note informed_lml: Updated: 4/16/19-MH Created: 3/21/19-MH  
label var informed_lml "Family not informed about SP's condition, LML"

foreach x in caredecis carenowan{
gen `x'_lml=. 
replace `x'_lml=0 if inlist(`x', 1, 2)
replace `x'_lml=1 if `x'==1
}

note caredecis_lml: caredecis
note caredecis_lml: Inadequate input about care decisions. Coming from LM4 (Waves 2+) 
note caredecis_lml: Located in NHATS Setup, under Part 7.
note caredecis_lml: Updated: 4/16/19-MH Created: 3/21/19-MH  
label var caredecis_lml " Inadequate input about care decisions, LML"

note carenowan_lml: carenowan
note carenowan_lml: Care not consistent with goals. Unwanted Care. Coming from LM5 (Waves 2+) 
note carenowan_lml: Located in NHATS Setup, under Part 7.
note carenowan_lml: Updated: 4/16/19-MH Created: 3/21/19-MH  
label var carenowan_lml "Care not consistent with goals, LML"

//1/28/2019-Advance Care Planning based off of Ornstein instructions

gen no_acp=0
replace no_acp=1 if inlist(eoltalk, 2, -8, -7) & inlist(poweratty, 2, -7, -8) & inlist(livngwill, 2, -7, -8)
replace no_acp=. if inlist(eoltalk, -1, -9, .) | inlist(poweratty, -1, -9, .) | inlist(livngwill, -1, -9, .) 
note no_acp: Only in Rd2. no_acp=1 if eoltalk=2,-7,-8 & poweratty=2,-7,-8 & livingwill=2,-7,-8. 
note no_acp: No Advance Care Planning based off of talk about EOL care, legal arrangements, and living will all being no. 
note no_acp: Located in NHATS Setup, under Part 7.
note no_acp: Created: 1/28/19-MH 

gen formal_acp=0
replace formal_acp=1 if inlist(eoltalk, 2, -8, -7) & (poweratty==1 | livngwill==1)
replace formal_acp=. if inlist(eoltalk, -1, -9, .) | inlist(poweratty, -1, -9, .) | inlist(livngwill, -1, -9, .) 
note formal_acp: Only in Rd2. formal_acp=1 if eoltalk=2,-7,-8 & (poweratty=1 | livingwill=1)
note formal_acp: Formal Advance Care Planning based off of not talking about EOL care and either having ///
having legal arrangements or having a living will.
note formal_acp: Located in NHATS Setup, under Part 7.
note formal_acp: Created: 1/28/19-MH 

gen informal_acp=0
replace informal_acp=1 if eoltalk==1 & inlist(poweratty, 2, -7, -8) & inlist(livngwill, 2, -7, -8)
replace informal_acp=. if inlist(eoltalk, -1, -9, .) | inlist(poweratty, -1, -9, .) | inlist(livngwill, -1, -9, .) 
note informal_acp: Only in Rd2. informal_acp=1 if eoltalk=1 & poweratty=2,-7,-8 & livingwill=2,-7,-8
note informal_acp: Informal Advance Care Planning based off of talking about EOL care and not having ///
having legal arrangements and not having a living will.
note informal_acp: Located in NHATS Setup, under Part 7.
note informal_acp: Created: 1/28/19-MH 

gen acp=0
replace acp=1 if eoltalk==1 & (poweratty==1 | livngwill==1)
replace acp=. if inlist(eoltalk, -1, -9, .) | inlist(poweratty, -1, -9, .) | inlist(livngwill, -1, -9, .)
note acp: Only in Rd2. eoltalk=1 & (poweratty=1 | livingwill=1)
note acp: Having ACP means having both Formal and Informal ACP.
note acp: Located in NHATS Setup, under Part 7.
note acp: Created: 1/28/19-MH


label var no_acp "No Advance care planning, 1=yes"
label var formal_acp "Formal Advance care planning, 1=yes"
label var informal_acp "Informal Advance care planning, 1=yes"
label var acp "Advance care planning (Formal & Informal), 1=yes"
label define acp  0 "No" 1 "Yes" 
foreach x in no_acp formal_acp informal_acp acp {
label values `x' acp
}


foreach x in no_acp formal_acp informal_acp acp {
tab `x' if wave==2 , m
}

// 8/10/2018 Add in 2011 weight for rd 5,6 
rename an2011wgt0 an2011wgt 
label var an2011wgt "2011 weights"
note an2011wgt: an2011wgt0
note an2011wgt: Analytic Weight for 2011 cohort, for rounds 5+
note formal_acp: Located in NHATS Setup, under Part 7.
note formal_acp: Created: 8/10/18-MH


rename trfinwgt0 trfinwgt
rename tr2011wgt0 tr2011wgt

//Add in 7/3/18

foreach w in 1 5{
	replace serarmfor=1 if serarmfor==1 & wave==`w'
	replace serarmfor=0 if serarmfor==2 & wave==`w'
	replace serarmfor=. if serarmfor==-7 & wave==`w'
	replace serarmfor=. if serarmfor==-8 & wave==`w'
}
label var serarmfor "Served in Armed Forces"
sort spid wave
by spid: carryforward serarmfor, replace
notes drop serarmfor
note serarmfor: serarmfor
note serarmfor: Served in Arm Forces variable, carried previous value to new wave. Asked in waves 1,5.
note serarmfor: Located in NHATS Setup, under Part 7.
note serarmfor: Created: 7/3/18-MH


//Add in 4/6/18
label var dresslf "Often you Dress Yourself"

label var dresdif "Difficulty when using Special Items by Self"

label var ntlvrmslp "Often had to Stay in Bed"

label var eatdev "Ever use Adapted Utensils"


////// 4/2/2018--Adding in income quartile
sort wave
forvalues w=1/$w {
xtile income_quart`w'=aveincome [aw=anfinwgt0] if wave==`w', nq(4) 
}
egen income_quart=rowmax(income_quart*)

forvalues w=1/$w{
drop income_quart`w'
}

note income_quart: Use aveincome to create variable.  
note income_quart: Create an income quartile measure. Weighted. 
note income_quart: Located in NHATS Setup, under Part 7.
note income_quart: Created: 4/2/18-MH

label var income_quart "Quartile of Income"
label define income_quart 1 "Bottom" 2 "Second" 3 "Third" 4 "Top"
label values income_quart income_quart

///// 3/26/18 --Adding in Hospice and place died variables.

label var placedied "Place of Death"
replace placedied=. if inlist(placedied,-1,-8)
replace placedied=6 if placedied==91
note placedied: placedied (6=91) (.=-1,-8) all else the same.   
note placedied: Place died variable from LML interview. 
note placedied: Located in NHATS Setup, under Part 7.
note placedied: Created: 3/26/18-MH

replace hospice2=. if inlist(hospice2,-1,-8) 
note hospice2: hospice2 variable, (.=-1,-1)
note hospice2: Hospice at other place, list.
note hospice2: Located in NHATS Setup, under Part 7.
note hospice2: Created: 3/26/18-MH

rename hospcelml hospicelml
foreach x in hospice1 hospicelml {
replace `x'=. if inlist(`x',-1,-8)
replace `x'=0 if `x'==2
} 

note hospice1: hospice1 variable (0=2) (.=-1,-8)
note hospice1: Hospice at Nursing Home.
note hospice1: Located in NHATS Setup, under Part 7.
note hospice1: Created: 3/26/18-MH

note hospicelml: hospicelml variable (0=2) (.=-1,-8)
note hospicelml: Hospice at other place, list.
note hospicelml: Located in NHATS Setup, under Part 7.
note hospicelml: Created: 3/26/18-MH
 
label var hospice1 "Hopsice at Nursing Home"
label var hospice2 "Hospice Other Place"
label var hospicelml "Hospice Care in Last Month"
 
 
///// 3/14/18 --Adding in Frailty underlying series

//activities done twice (grip test, walk test..)

forvalues i=1/2{
capture drop gripsc`i'
rename grp`i'rdng gripsc`i'
rename wlkc`i'secs walksec`i'
rename wlk`i'hndr walkhndr`i'

replace gripsc`i'=. if inlist(gripsc`i', -1, -9, -7, -8)
replace walksec`i'=. if inlist(walksec`i', -1, -9, -7, -8)
replace walkhndr`i'=. if inlist(walkhndr`i', -1, -9, -7, -8)
}

label var gripsc1 "GRP1 Display Reading"
label var gripsc2 "GRP2 Display Reading"
label var walksec1 "Secs Hld Walking Course1"
label var walksec2 "Secs Hld Walking Course2"
label var walkhndr1 "Hnrds Hld Walking Course1"
label var walkhndr2 "Hnrds Hld Walking Course2"

foreach y in gripsc walksec walkhndr{
foreach x in 1 2{
note `y'`x': `y'`x' (.=-1,-7,-8,-9)

if "`y'"=="gripsc"{
note gripsc`x': Grip Strength `x' Reading 
}
else if "`y'"=="walksec"{
note walksec`x': Walking Course Completion in Seconds place `x' Reading 
}
else if "`y'"=="walkhndr"{
note walkhndr`x': Walking Course Completion Hundredths place `x' Reading
}
note `y'`x': Located in NHATS Setup, under Part 7.
note `y'`x': Created: 3/14/18-MH
}
}


foreach x in origwksc nhatswksc nhatsgrav nhatsgrb{
capture drop `x'
gen `x'=.
replace `x'=d`x' if  inlist( d`x', 0, 1, 2, 3, 4)
}

label var origwksc "Original Walk Score"
label var nhatswksc "NHATS Walk Score"
label var nhatsgrav "NHATS Avg. Grip Score"
label var nhatsgrb "NHATS Best Grip Score"

foreach x in origwksc nhatswksc nhatsgrav nhatsgrb{
note `x': d`x' (.=-1,-9)

if "`x'"=="origwksc"{
note `x': Original Walk Score 
}
else if "`x'"=="nhatswksc"{
note `x': NHATS Walk Score
}
else if "`x'"=="nhatsgrav"{
note `x': NHATS Avg Grip Score
}
else if "`x'"=="nhatsgrb"{
note `x': NHATS Best grip Score
}
note `x': Located in NHATS Setup, under Part 7.
note `x': Created: 3/14/18-MH

}

local a lst10pnds trytolose 
local b evrgowalk vigoractv

foreach x in `a' `b'{
capture drop `x'_n
gen `x'_n=.
}


foreach x in `a' `b'{
replace `x'_n=0 if `x'==2 
replace `x'_n=1 if `x'==1
drop `x'
rename `x'_n `x' 
note `x': `x' (0=2) (1=1) (.=-1, -9)
}

note lst10pnds: Lost 10 Pounds in Last Year (Y/N)
note trytolose: Were you trying to lose Weight (Y/N)
note evrgowalk: Ever go walking for exercise (Y/N)
note vigoractv: Ever Vigorous Activities (Y/N)

foreach x in `a' `b'{
note `x': Located in NHATS Setup, under Part 7.
note `x': Created: 04/05/18-MH
}

label var lst10pnds "Lost 10 Pounds in Last Yr."
label var trytolose "Were You Trying to Lose Weight"
label var evrgowalk "Ever Go Walking for Exercise"
label var vigoractv "Ever Vigorous Activities"

capture drop unwgtloss lowphysact
gen unwgtloss=. 
gen lowphysact=.
replace unwgtloss=0 if lst10pnds==0 | trytolose==1
replace unwgtloss=1 if lst10pnds==1 & trytolose==0
replace lowphysact=0 if evrgowalk==1 | vigoractv==1
replace lowphysact=1 if evrgowalk==0 & vigoractv==0

label var unwgtloss "Unintentional Weight Loss"
label var lowphysact "Low Physical Activity"

note unwgtloss: Created from lst10pnds and trytolose, if lst10pnds is yes and trytolose is no.
note unwgtloss: Unintentional Weight Loss
note lowphysact: Created frin evrgowalk and vigoractv, if evrgowalk is no and vigoractv is no. 
note lowphysact: Low Physical Activity

replace lowenergy=. if inlist(lowenergy, -9, -8, -7, -1)
replace lowenergy=0 if lowenergy==2
label var lowenergy "Low Energy in Last Month"

note lowenergy: lowenergy is (.=-9,-8,-7,-1) (0=2) (1=1)
note lowenergy: Low Energy in Last Month (Y/N)

foreach x in unwgtloss lowphysact lowenergy{
note `x': Located in NHATS Setup, under Part 7.
note `x': Created: 04/05/18-MH
} 

///// 3/7/18 --Adding in Well-Being questions 
/// Inapplicable is people who have no interview, had a proxy, or died. turned to missing. 

foreach j in 1 2 4 {
	forvalues i=1/4 {
		capture drop wbq`j'`i'
		gen wbq`j'`i'=.
	}
}
drop wbq44

note wbq11: Often 

forvalues i=1/4 {
replace wbq1`i'=1 if offelche`i'==4 
replace wbq1`i'=2 if offelche`i'==3 
replace wbq1`i'=3 if offelche`i'==2
replace wbq1`i'=4 if offelche`i'==1 
replace wbq1`i'=0 if offelche`i'==5 
replace wbq1`i'=5 if (offelche`i'==-7 | offelche`i'==-8) 
note wbq1`i': offelche`i' (0=5) (1=4) (2=3) (3=2) (4=1) (5=-7,-8)
if `i'==1{
note wbq1`i': Often you feel Cheerful (wb)
}
else if `i'==2{
note wbq1`i': Often you feel Bored (wb)
}
else if `i'==3{
note wbq1`i': Often you feel Full of Life (wb)
}
else if `i'==4{
note wbq1`i': Often you feel Upset (wb)
}
note wbq1`i': Located in NHATS Setup, under Part 7.
note wbq1`i': Created: 04/05/19-MH
}


label var wbq11 "Often you Feel Cheerful"
rename wbq11 oftcheer 
label define oftcheer 0 "Never" 1 "Rarely (Once a Week or Less)" 2 "Some Days(2-4 Days a Week)" ///
3 "Most Days(5-6 Days a Week)" 4 "Every Day (7 Days a Week)" 5 "RF/DK"
label values oftcheer oftcheer
label var wbq12 "Often you Feel Bored"
rename wbq12 oftbored
label values oftbored oftcheer
label var wbq13 "Often you Feel Full of Life"
rename wbq13 oftfol
label values oftfol oftcheer
label var wbq14 "Often you Feel Upset"
rename wbq14 oftupset
label values oftupset oftcheer

note drop oftcheer in 1
note renumber oftcheer

forvalues i=1/4 {
replace wbq2`i'=1 if truestme`i'==3 
replace wbq2`i'=2 if truestme`i'==2 
replace wbq2`i'=3 if truestme`i'==1
replace wbq2`i'=4 if (truestme`i'==-7 | truestme`i'==-8) 
note wbq2`i': truestme`i' (1=3) (2=2) (3=1) (4=-7,-8)
if `i'==1{
note wbq2`i': Life has Meaning and Purpose (wb)
}
else if `i'==2{
note wbq2`i': I Feel Confident and Good about Myself (wb)
}
else if `i'==3{
note wbq2`i': I gave up trying ot imporve my life a long time ago (wb)
}
else if `i'==4{
note wbq2`i': I like my living situation very much (wb)
}
note wbq2`i': Located in NHATS Setup, under Part 7.
note wbq2`i': Created: 04/05/18-MH
}


label var wbq21 "Life has Meaning Purpose"
rename wbq21 lifemean 
label define lifemean 1 "Agree Not at All" 2 "Agree a Little" 3 "Agree A Lot" 4 "RF/DK"
label values lifemean lifemean
label var wbq22 "Feels Confident"
rename wbq22 confident
label values confident lifemean
label var wbq23 "Gave up Improving Life"
rename wbq23 giveup
label values giveup lifemean
label var wbq24 "Likes Living Situation"
rename wbq24 livingsit
label values livingsit lifemean


forvalues i=1/3 {
replace wbq4`i'=1 if agrwstmt`i'==3 
replace wbq4`i'=2 if agrwstmt`i'==2 
replace wbq4`i'=3 if agrwstmt`i'==1 
replace wbq4`i'=4 if (agrwstmt`i'==-7 | agrwstmt`i'==-8)
note wbq4`i': truestme`i' (1=3) (2=2) (3=1) (4=-7,-8)
if `i'==1{
note wbq4`i': Other people determine most of what I can and cannot do (wb)
}
else if `i'==2{
note wbq4`i': When I really want to do something, I usually find a way to do it. (wb)
}
else if `i'==3{
note wbq4`i': I have an easy time adjusting to change.  (wb)
}
note wbq4`i': Located in NHATS Setup, under Part 7.
note wbq4`i': Created: 04/05/18-MH
}


label var wbq41 "Self Determination"
rename wbq41 selfdeter 
label values selfdeter lifemean
label var wbq42 "Want-Finds Way to Do"
rename wbq42 findway
label values findway lifemean
label var wbq43 "Easily Adjust to Change"
rename wbq43 adjchg
label values adjchg lifemean
	

capture drop agefeel
egen agefeel= rowmax(ageyofeel) 
replace agefeel=. if agefeel==-9 | agefeel==-8 | agefeel==-7 | agefeel==-1
label var agefeel "Age you Feel Most Times"

note agefeel: From ageyofeel 
note agefeel: Age you feel most of the time
note agefeel: Located in NHATS Setup, under Part 7.
note agefeel: Created: 04/05/18-MH

///// 3/2/18 --Adding in meal skippped questions; update 3/13 added in EW18
//missing if in NH-FQ, dead, in NH-FQ R1 & R2

local ew mealskip1 nopayhous nopayutil nopaymed

foreach x in `ew'{
tab `x'
replace `x'=0 if `x'==2 
replace `x'=1 if `x'==1
replace `x'=. if inlist(`x', -9,-8,-7,-1)
note `x': `x' (0=2) (1=1) (.=-9,-8,-7,-1)
if "`x'"=="mealskip1"{
note `x': Skip any meals b/c there was not enough food, or money to buy food
}
else if "`x'"=="nopayhous"{
note `x': Not enough money to pay rent or mortgage
}
else if "`x'"=="nopayutil"{
note `x':  Not enough money to pay utility bills(gas, electric)
}
else if "`x'"=="nopaymed"{
note `x':  Not enough money to pay medical or prescription drug bills
}
note `x': Located in NHATS Setup, under Part 7.
note `x': Created: 04/05/18-MH
}


label var mealskip1 "No Money, Meals Skipped"
label var nopayhous "No Money for Housing"
label var nopayutil "No Money for Utilities"
label var nopaymed "No Money for Medical"

label define mealskip 0 "No" 1 "Yes" 
foreach x in `ew' {
label values `ew' mealskip
}
rename mealskip1 mealskip

gen mealskipnum=.
replace mealskipnum=0 if mealskip2==-1 & mealskip==0
replace mealskipnum=1 if mealskip2==4
replace mealskipnum=2 if mealskip2==3
replace mealskipnum=3 if mealskip2==2
replace mealskipnum=4 if mealskip2==1


label var mealskipnum "How Often Meals Skipped"
label define mealskipnum1 0 "No" 1 "A Few Days" 2 "Several Days(less than half)" ///
3 "More than Half the Days" 4 " Nearly Every Day" 5 "RF/DK"
label values mealskipnum mealskipnum1

note mealskipnum: mealskip2 mealskip1 (0=mealskip2 -1 & mealskip1 0) (1=4) (2=3) (3=2) (4=1)
note mealskipnum: How often are meals skipped
note mealskipnum: Located in NHATS Setup, under Part 7.
note mealskipnum: Created: 04/05/18-MH

///// 12/5/17 --Adding in Cancer 

// recode hc1cancerty1 (-1 2=0) (-9 -8 =.), gen (sr_skincancer_had)

// From sensitive data, cancerty1 was from non-sensitive
//recode cancerty1 (-1 2=0) (-9 -8 =.), gen (sr_skincancer_had)
foreach x in sr_skincancer_had sr_breastcancer_had sr_prostatecancer_had sr_bladdercancer_had ///
sr_crvovrnutrncancer_had sr_coloncancer_had sr_kidneycancer_had sr_othercancer_had{
gen `x'=.
}

replace sr_skincancer_had=cancerty1 if wave==1
replace sr_breastcancer_had=hc1cancerty2 if wave==1
replace sr_prostatecancer_had=hc1cancerty3 if wave==1
replace sr_bladdercancer_had=hc1cancerty4 if wave==1
replace sr_crvovrnutrncancer_had=hc1cancerty5 if wave==1
replace sr_coloncancer_had=hc1cancerty6 if wave==1
replace sr_kidneycancer_had=hc1cancerty7 if wave==1
replace sr_othercancer_had=hc1cancerty8 if wave==1

//NEED TO CHANGE TO $w when sensitive released for rd 8. 
forvalues w=4/7{
replace sr_skincancer_had=cancerty1 if wave==`w'
replace sr_breastcancer_had=hc`w'cancerty2 if wave==`w'
replace sr_prostatecancer_had=hc`w'cancerty3 if wave==`w'
replace sr_bladdercancer_had=hc`w'cancerty4 if wave==`w'
replace sr_crvovrnutrncancer_had=hc`w'cancerty5 if wave==`w'
replace sr_coloncancer_had=hc`w'cancerty6 if wave==`w'
replace sr_kidneycancer_had=hc`w'cancerty7 if wave==`w'
replace sr_othercancer_had=hc`w'cancerty8 if wave==`w'
}

local r=1
foreach x in sr_skincancer_had sr_breastcancer_had sr_prostatecancer_had sr_bladdercancer_had ///
sr_crvovrnutrncancer_had sr_coloncancer_had sr_kidneycancer_had sr_othercancer_had{
replace `x'=0 if inlist(`x',-1,2)
replace `x'=. if inlist(`x', -9,-8)
notes `x': cancerty`r' (0=-1,2) (.=-9,-8)
if "`x'"=="sr_skincancer_had"{
notes `x': Had Skin Cancer
}
else if "`x'"=="sr_breastcancer_had"{
notes `x': Had Breast Cancer
}
else if "`x'"=="sr_prostatecancer_had"{
notes `x': Had Prostate Cancer
}
else if "`x'"=="sr_bladdercancer_had"{
notes `x': Had Bladder Cancer
}
else if "`x'"=="sr_crvovrnutrncancer_had"{
notes `x': Had Cervical/Ovarian/Uterine Cancer
}
else if "`x'"=="sr_coloncancer_had"{
notes `x': Had Colon Cancer
}
else if "`x'"=="sr_kidneycancer_had"{
notes `x': Had Kidney Cancer
}
else if "`x'"=="sr_othercancer_had"{
notes `x': Had Other Type of Cancer
}
note `x': Located in NHATS Setup, under Part 7.
note `x': Created: 04/05/18-MH
local r=`r'+1
}

label var sr_skincancer_had "Sensitive: Skin Cancer"
label var sr_breastcancer_had "Sensitive: Breast Cancer"
label var sr_prostatecancer_had "Sensitive: Prostate Cancer"
label var sr_bladdercancer_had "Sensitive: Bladder Cancer"
label var sr_crvovrnutrncancer_had "Sensitive: Cervical/Ovarian/Uterine Cancer"
label var sr_coloncancer_had "Sensitive: Colon Cancer"
label var sr_kidneycancer_had "Sensitive: Kidney Cancer"
label var sr_othercancer_had "Sensitive: Other Cancer"

//recoding all refuse, dk, na, as missing
qui destring *, replace
foreach x of varlist * {
	replace `x'=. if inlist(`x',-1,-7,-8,-9)
}


//how do you get to the doctor?
gen howgetdoc=.
forvalues i=1/9 {
replace howgetdoc=`i' if hwgtregd`i'==1 
}
note howgetdoc: hwgtredd# used to create variable
note howgetdoc: How did you get a ride to the doctor; transportation method
note howgetdoc: Located in NHATS Setup, under Part 7.
note howgetdoc: Created: 04/05/18-MH

label define howgetdoc 1 "Drove" 2 "Got a Ride" 3 "Van/Shuttle from place lives" ///
4 "Van/Shuttle for seniors & disabled" 5 "Pulic transit" 6 "Taxi" 7 "Walked" ///
8 "Home visit" 9 "Other" 0 "Did not See Doctor" 10 "Refused/DK"
label values howgetdoc howgetdoc


//note 11/14/17--add importance of activities
gen ind_imp_visit=.
gen ind_imp_relig=.
gen ind_imp_club=.
gen ind_imp_goout=.

replace ind_imp_visit=inlist(impvstfam,1,2) 
replace ind_imp_relig=inlist(imprelser,1,2) 
replace ind_imp_club=inlist(imparclub,1,2) 
replace ind_imp_goout=inlist(impouteny,1,2) 

note ind_imp_visit: impvstfam (1=1,2) (0=3)
note ind_imp_visit: Very/somewhat important to visit friends/family in person 
note ind_imp_relig: imprelser (1=1,2) (0=3)
note ind_imp_relig: Very/somewhat important to attend religious services 
note ind_imp_club: imparclub (1=1,2) (0=3)
note ind_imp_club: Very/somewhat important to participate in clubs, classes, or other organized activities.
note ind_imp_goout: impouteny (1=1,2) (0=3)
note ind_imp_goout: Very/somewhat important to go out for enjoyment

foreach x in visit relig club goout{
note ind_imp_`x': Located in NHATS Setup, under Part 7.
note ind_imp_`x': Created: 04/05/19-MH
}

label var ind_imp_visit "Very/somewhat important to visit in person with friends/family"
label var ind_imp_relig "Very/somewhat important to attend religious services"
label var ind_imp_club "Very/somewhat important to participate in clubs, classes, etc."
label var ind_imp_goout "Very/somewhat important to go out for enjoyment"


foreach x in out insd bed eat bath toil dres {
	gen device_`x'=.
}

label var device_out "Uses any assistive device for mobility outside"
label var device_ins "Uses any assistive device for mobility inside"
label var device_bed "Uses any assistive device to get out of bed"
label var device_eat "Uses any assistive device for eating"
label var device_bath "Uses any assistive device for bathing"
label var device_toil "Uses any assistive device for toileting"
label var device_dres "Uses any assistive device for dressing"

/*
foreach x in eat bath toil dres {
	gen device_`x'=.
	label var device_`x' "Uses assistive device for `x'ing"
}
*/

foreach x in out insd bed eat bath toil dres{
	replace device_`x'=d`x'devi==2 if inlist(d`x'devi,1,2)
	note device_`x': d`x'devi (2=1,2); derived variable 
	if "`x'"=="out"{
		note device_`x': Go outside using devices
	}
	else if "`x'"=="insd"{
		note device_`x': Move inside with devices
	}
	else if "`x'"=="bed"{
		note device_`x': Device used to get out of bed
	}	
	else if "`x'"=="eat"{
		note device_`x': Uses devices while eating
	}
	else if "`x'"=="bath"{
		note device_`x': Uses devices while bathing
	}
	else if "`x'"=="toil"{
		note device_`x': Uses devices while toileting
	}
	else if "`x'"=="dres"{
		note device_`x': Uses devices while dressing
	}
	note device_`x': Located in NHATS Setup, under Part 7.
	note device_`x': Created: 04/05/18-MH
}

/*
foreach x in eat bath toil dres {
	replace device_`x'=d`x'devi==2 if inlist(d`x'devi,1,2) 
}
*/
replace device_bath=0 if dbathdevi==9 


gen device_any=0
foreach x in out ins bed eat bath toil dres {
	replace device_any=1 if device_`x'==1
}
note device_any: Derived using device_ variables. 1 is any device used. 
note device_any: Any devices used for ADL's. 
note device_any: Located in NHATS Setup, under Part 7.
note device_any: Created: 04/05/18-MH

label var device_any "Uses any assistive device for ADLs"

rename drives dtdrives //a drive flag exists in nhats
//making our own drive variable
gen drives=.
replace drives=oftedrive>0 & oftedrive<5 

label var drives "Drives"

note drives: oftedrive (1=1,2,3,4) (0=5)
note drives: Did you ever drive. 
note drives: Located in NHATS Setup, under Part 7.
note drives: Created: 04/05/18-MH

gen avoids=.
replace avoids=0 if oftedrive>0 & oftedrive!=5 & avoidriv1==2 & avoidriv2==2 & avoidriv4==2

local avoids avoidriv1 avoidriv2 avoidriv4

foreach x of varlist `avoids' {
	replace avoids=1 if `x'==1 
	replace `x'=. if `x'<0 
}

label var avoids "Avoids driving alone/at night/in weather*"

note avoids: oftedrive avoidriv1 avoidriv2 avoidriv4
note avoids: a person drives and doesn't avoid driving.
note avoids: Located in NHATS Setup, under Part 7.
note avoids: Created: 04/05/18-MH

gen miss=0
gen ind_transp_disadv=.

replace ind_transp_disadv=0 if oftedrive==. | inlist(oftedrive,5,6)
local probs trkpfrvis trprrelsr trprkpfgr trprgoout
foreach x in `probs' {
replace ind_transp_disadv=1 if `x'==1
gen `x'2=`x'==1 if `x'>=0 & !missing(`x') 
replace `x'=`x'2
drop `x'2
replace miss=miss+1 if missing(`x') 
}

replace ind_transp_disadv=0 if !miss & missing(ind_transp_disadv)
replace ind_transp_disadv=. if miss==4
label var ind_transp_disadv "Any transportation disadvantage"
drop miss

note ind_transp_disadv: oftedrive trkpfrvis trprrelsr trprkpfgr trprgoout
note ind_transp_disadv: Indicate any transportation disadvantage 
note ind_transp_disadv: Located in NHATS Setup, under Part 7.
note ind_transp_disadv: Created: 04/05/18-MH

local lab1 Walked
local lab2 Got ride
local lab3 Van or shuttle provided by place of residence
local lab4 Van or shuttle for seniors
local lab5 Public transportation
local lab6 Taxi
local lab7 Other
local l=1

forvalues i=1/7 {
gen get_places`i'=.
}

replace getoplcs3=0 if !missing(getoplcs7) & missing(getoplcs3)
forvalues x=1/7  {
	replace get_places`x'=getoplcs`x'==1 
	replace get_places`x'=. if (inlist(getoplcs7,-1,.) | getoplcs`x'==.) 
	label var get_places`x' "Get places: `lab`x''"
	note get_places`x': getoplcs`x'
	note get_places`x': `lab`x'' to get to places
	note get_places`x': Located in NHATS Setup, under Part 7.
	note get_places`x': Created: 04/05/18-MH
	local l=`l'+1
	
}

//marital status-3 vars: married, married_or_partnered

gen married=maritalstat==1 if maritalstat!=. //married
gen marital_sep=maritalstat==3 if !mi(maritalstat)
gen marital_div=maritalstat==4 if !mi(maritalstat)
gen marital_wid=maritalstat==5 if !mi(maritalstat)
gen marital_nev=maritalstat==6 if !mi(maritalstat)
gen marriedpartnered=married
replace marriedpartnered=1 if maritalstat==2 //living with a partner
gen marital_sd=marital_sep
replace marital_sd=1 if marital_div==1

label var married "Married"
label var marriedpartnered "Married or Live w/ Partner"
label var marital_sep "Separated"
label var marital_div "Divorced"
label var marital_wid "Widowed"
label var marital_nev "Never married"
notes married : martlstat
notes married : (Y/N)
notes married : E:\nhats\nhats_code\NHATS data setup\NHATS_Setup, step-3 header for intermediate var and step-7 for final
notes married : Updated: 04/05/18-MH

foreach x in marital_sep marital_div marital_wid marital_nev marriedpartnered marital_sd{
notes `x': martlstat dmarstat 
	if "`x'"=="marital_sep"{
	notes `x': Seperated, marital status.
	}
	if "`x'"=="marital_div"{
	notes `x': Divorced, marital status.
	}
	if "`x'"=="marital_wid"{
	notes `x': Widowed, marital status.
	}
	if "`x'"=="marital_nev"{
	notes `x': Never Married, marital status.
	}
	if "`x'"=="marriedpartnered"{
	notes `x': Married or Living with a partner.
	}
	if "`x'"=="marital_sep"{
	notes `x': Seperated or Divorced, marital status.
	}
note `x': Located in NHATS Setup part 3 initial cleanup, part 7 final cleanup.
note `x': Created: 04/05/18-MH
}

//lives with spouse
gen resspouse=.
replace resspouse=livwthspo==1 if livwthspo>0
replace resspouse=0 if marriedpart==0

label var resspouse "Lives with Spouse/Partner"
notes resspouse : livwthspo martlstat
notes resspouse : Living with spouse or partner (Y/N)
notes resspouse : Located in NHATS Setup, under Part 7.
notes resspouse : Updated: 04/08/19-MH

//household members
gen hhm=.
replace hhm=dhshldnum if dhshldnum>0 

label var hhm "Number of household members"
notes hhm : dhshldnum 
notes hhm : Count of the number of household members. 
notes hhm : Located in NHATS Setup, under Part 7.
notes hhm : Updated: 04/08/19-MH

//age categorical

sort spid wave
by spid: replace age=age[_n-1]+1 if missing(age)
label var age "Sensitive: age"

gen agecat = .
replace agecat= d2intvrage
replace agecat=. if agecat==-1
label values agecat d2intvrage
label var agecat "Age, 5 yr categories"

notes agecat : d2intvrage (same)
notes agecat : Age cateory 65-59, 70-74, 75-79, 80-84, 85-89, 90+ 
notes agecat : Located in NHATS Setup, under Part 7.
notes agecat : Updated: 04/09/19-MH

//activities past month

gen attrelig=. //attend services
gen imprelig=. //importance of religious services
gen visitfrfam=. //visit fr. fam.
gen impvisit=. //importance of visiting friends & fam
gen attclub=. //participate in club, organized activity, class, etc.
gen impclub=. //importance of such activities
gen vigact=vigoractv //vigorous activities


replace attrelig=attrelser==1 if attrelser>0 
replace imprelig=imprelser if imprelser>0  
replace visitfrfam=vistfrfam==1 if vistfrfam>0 
replace impvisit=impvstfam if impvstfam>0 
replace attclub=clbmtgrac==1 if clbmtgrac>0 
replace impclub=imparclub if imparclub>0 
//replace vigact=vigoractv==1 if vigoractv>0 


gen religvimp=imprelig==1 if imprelig!=. //very important binary

label var attrelig "Past mo. Attend religious services"
label var imprelig "Importance of religious services"
label var religvimp "Religious services very important, yes/no"
label var visitfrfam "Past mo. visit friends or family"
label var impvisit "Importance of visiting friends/family"
label var attclub "Past mo. participate in clubs/classes/organized activities"
label var impclub "Importance of club/class/etc. participation"
label var vigact "Past mo. engage in vigorous activity"

label values imprelig pa1imprelser
label values impvisit pa1impvstfam
label values impclub pa1imparclub

notes attrelig : attrelser (same)
notes attrelig : Ever attend religious services 

notes imprelig : imprelser (same)
notes imprelig : How important are religious services, very, somewhat, or not so important. 

notes visitfrfam : visfrfam (same)
notes visitfrfam : Ever visit in person with friends or family, not living with.

notes impvisit : impvstfam (same)
notes impvisit : How important to visit in person with friends or family not living with. 

notes impclub: imparclub 
notes impclub: Importance of club/class/etc. participation

notes attclub : clbmtgrac (same)
notes attclub : Ever participate in clubs, classes, or other organized activites. 

notes vigact : vigoractv (same)
notes vigact : Ever spend time on vigorous activites that increased heart rate. ///
Working out, swimming, running, biking, or any sport.  

notes religvimp: imprelig 
notes religvimp: Religious services are important, binary. 

foreach x in attrelig imprelig visitfrfam impvisit impclub attclub vigact religvimp {
notes `x' : Located in NHATS Setup, under Part 7.
notes `x' : Updated: 04/09/19-MH
}

//smokin'
gen smokedever=. 
gen smoke_start_age=.
gen smoke_stop_age=.

forvalues i=1(4)5 {
	replace smokedever=smokedreg==1 if wave==`i' & smokedreg>0
	replace smoke_start_age=agesrtsmk if wave==`i' & agesrtsmk>0
	replace smoke_stop_age=agelstsmk if wave==`i' & agelstsmk>0
}

sort spid wave
by spid: carryforward smokedever smoke_start_age smoke_stop_age, replace
notes drop smokedever smoke_start_age smoke_stop_age
rename smokesnow smokesnow_old

notes smokedever: smokedreg 
notes smokedever: Only in rounds 1,5. Ever smoke cigarettes regularly. 

notes smoke_start_age: agesrtsmk 
notes smoke_start_age: How old when first smoked, age.

notes smoke_stop_age: agelstsmk  
notes smoke_stop_age: How old when last smoked cigarettes regularly.

foreach x in smokedever smoke_start_age smoke_stop_age{
notes `x': Located in NHATS Setup, under Part 7.
notes `x' : Updated: 04/10/19-MH
}

gen smokesnow=.
//Numcigever asked differently in rounds 1,5 then in other rounds. (Do you/ Did you)
//Wanted to see the num. of cigs. smoked by non-smokers. 
by spid, sort: egen numcigsever=max(numcigday) if inlist(wave,1,5) & smokesnow_old==2
by spid: carryforward numcigsever if smokesnow_old==2, replace

notes drop numcigsever

notes numcigsever: numcigday
notes numcigsever: Take max if they were a non-smoker. 

gen numcigsnow=.

replace smokesnow=smokesnow_old==1 if smokesnow_old>0
replace numcigsnow=numcigday if smokesnow==1

replace smokedever=1 if smokesnow==1
replace smokesnow=0 if smokedever==0
replace numcigsnow=0 if smokesnow==0

notes smokesnow: smokedever
notes smokesnow: Smokes at this wave
notes smokesnow: Located in NHATS Setup, under Part 7.
notes smokesnow: Updated: 05/03/19-MH

label var smokedever "Ever smoked"
label var smokesnow "Smokes now"
label var numcigsever "Max number of cigarettes/day when smoked for current non-smokers"
label var numcigsnow "Number of cigarettes/day currently"
label var smoke_start_age "Age started smoking"
label var smoke_stop_age "Age stopped smoking"

notes numcigsever: Located in NHATS Setup, under Part 7.
notes numcigsever: Updated: 04/10/19-MH

notes numcigsnow: numcigday smokesnow
notes numcigsnow: Number of cigarettes/day currently
notes numcigsnow: Located in NHATS Setup, under Part 7.
notes numcigsnow: Updated: 05/03/19-MH

//pain
gen ind_pain=.
replace ind_pain=painb==1 if inrange(painb,0,2) 
label var ind_pain "Bothered by pain"

notes ind_pain: painbothr
notes ind_pain: Have been bothered by pain in the last month. 
//weight

gen wgt_curr=.
gen height_ft=.
gen height_in=.

replace wgt_curr=currweigh if currweigh>0 
replace height_ft=howtallft if howtallft>0 
replace height_in=howtallin if howtallin>0 

label var wgt_curr "Current weight, lbs."
label var height_ft "Current height, feet"
label var height_in "Current height, inches"

notes wgt_curr: currweigh
notes wgt_curr: Current Weight in pounds. 
notes height_ft: howtallft
notes height_ft: How tall in feet. 
notes height_in: howtallin
notes height_in: How tall in inches, after feet. 

gen bmi=(wgt_curr/((height_ft*12+height_in)^2))*703

label var bmi "Current BMI"

notes bmi: wgt_curr height_ft height_in
notes bmi: Body Mass Index calculation from weight and height. 

foreach x in wgt_curr height_ft height_in bmi{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/10/19-MH
}

//get the per round weight, cluster, and strata vars (and why is moved created like that?)
//gen moved=.
rename varunit varunit 
rename varstrat varstrat 
rename anfinwgt0 anfinwgt 

/*
forvalues i=2/7 {
	replace moved=w`i'_moved if wave==`i'
	foreach x in anfinwgt varstrat varunit {
		replace `x'=w`i'`x' if wave==`i'
}
drop w`i'_moved w`i'anfinwgt0 w`i'varstrat w`i'varunit 
}
*/
label var moved "Moved since last ivw"

******************************************
/*these are taken from Rebecca's code*/

gen community_dwelling = 0
gen SP_completed = 0
forvalues w=1/$w {
replace community_dwelling=1 if inlist(dresid,1,2) & wave==`w'
replace SP_completed=1 if (inlist(dresid,1,2) & wave==1) | (inlist(dresid,1,2,4) & wave==`w')
}
notes community_dwelling: dresid = 1,2 
notes community_dwelling: In the community. 

gen nhres=dresid==4 if dresid!=. & wave==1
gen rcfres=inlist(dresid,2,3) if dresid!=. & wave==1
gen res_care=dresid>1 if dresid!=. & wave==1
gen sp_status=1 
gen deceased=0
gen prob_dem=dem_3_cat==1 if !missing(dem_3_cat)
label var prob_dem "Indicator probable dementia"

forvalues w=2/$w {	
replace nhres=inlist(dresid,4,5,8) if wave==`w'
replace res_care=inlist(dresid,2,3,4,5,7,8) if wave==`w'
replace rcfres=inlist(dresid,2,3,7) if wave==`w'
replace deceased=inlist(r`w'status,62,86) if wave>=`w'
}

note nhres : dresid
notes nhres : Resid=4,5,8, Nursing Home. 
notes nhres : E:\nhats\nhats\code\NHATS data setup\NHATS_Setup, step-7 header

notes rcfres: dresid (2,3)
notes rcfres: Residential Care, excluding Nursing Home. 

notes res_care: dresid (2-8)
notes res_care: Residential Care, excluding Nursing Home. 

notes deceased: status 62 68
notes deceased: Died after round 1

notes prob_dem: dem_3_cat, sr_dementia_ever
notes prob_dem: Probable Dementia. 

foreach x in community_dwelling rcfres res_care deceased prob_dem{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/10/19-MH
}


replace sp_status=2 if res_care==1
replace sp_status=3 if nhres==1
replace sp_status=4 if deceased==1
label var sp_status "Residential status"
label var nhres "Nursing Home Resident"
label var rcfres "Residential Care, excl. NH"
label var res_care "Residential Care, incl. NH"
label define sp_status 1 "Non-res care community" 2 "Res-care not NH" 3 "NH resident" ///
4 "Deceased"
label values sp_status sp_status

//these ADL variables were missing previously
gen adl_ins_help = .
gen adl_ins_diff=.
gen adl_ins_not_done=.



replace adl_ins_help=1 if insdhlp==1 
replace adl_ins_help=0 if insdhlp==2 
replace adl_ins_help=0 if insdhlp==-1 & sp_ivw==1 
replace adl_ins_help=-7 if insdhlp==-7 
replace adl_ins_help=-8 if insdhlp==-8 
replace adl_ins_diff=0 if insddif==1 
replace adl_ins_diff=1 if insddif>1 
replace adl_ins_diff=0 if insddif==-1 & sp_ivw==1 
replace adl_ins_diff=-7 if insddif==-7 
replace adl_ins_diff=-8 if insddif==-8 
replace adl_ins_not_done=inlist(dinsdsfdf,1,8) 


*replace adl_ins_dif=1 if adl_ins_help==1
label var adl_ins_help "Received help getting around inside"
notes adl_ins_help : insdhlp
notes adl_ins_help : Anyone help you get arounbd inside house
notes adl_ins_help : E:\nhats\nhats\code\NHATS data setup\NHATS_Setup, step-7 header

notes adl_ins_diff: insddif
notes adl_ins_diff: How much difficulty getting around inside. 
notes adl_ins_not_done: dinsdsfdf
notes adl_ins_not_done: Move inside by self
notes adl_ins_not_done:  E:\nhats\nhats\code\NHATS data setup\NHATS_Setup, step-7 header

//8/23/19--ad outside help & difficulty (don't include it in the indices)
gen adl_out_help = .
gen adl_out_diff=.
gen adl_out_not_done=.
replace adl_out_help=1 if outhlp==1 
replace adl_out_help=0 if outhlp==2 
replace adl_out_help=0 if outhlp==-1 & sp_ivw==1 
replace adl_out_help=-7 if outhlp==-7 
replace adl_out_help=-8 if outhlp==-8 
replace adl_out_diff=0 if outdif==1 
replace adl_out_diff=1 if outdif>1 
replace adl_out_diff=0 if outdif==-1 & sp_ivw==1 
replace adl_out_diff=-7 if outdif==-7 
replace adl_out_diff=-8 if outdif==-8 
replace adl_out_not_done=inlist(doutsfdf,1,8) 


*replace adl_ins_dif=1 if adl_ins_help==1
label var adl_out_help "Received help getting around outide"
notes adl_out_help : outhlp
notes adl_out_help : Anyone help you go outside
notes adl_out_help : E:\nhats\nhats\code\NHATS data setup\NHATS_Setup, step-7 header

notes adl_out_diff: outdif
notes adl_out_diff: How much difficulty going outside by self
notes adl_out_not_done: doutsfdf
notes adl_out_not_done: Go outside by self
notes adl_out_not_done:  E:\nhats\nhats\code\NHATS data setup\NHATS_Setup, step-7 header


gen adl_bed_help = .
gen adl_bed_diff=.
gen adl_bed_not_done=.

replace adl_bed_help=1 if bedhlp==1 
replace adl_bed_help=0 if bedhlp==2 
replace adl_bed_help=0 if bedhlp==-1 & sp_ivw==1 
replace adl_bed_help=-7 if bedhlp==-7 
replace adl_bed_help=-8 if bedhlp==-8 
replace adl_bed_diff=0 if beddif==1 
replace adl_bed_diff=1 if beddif>1 
replace adl_bed_diff=0 if beddif==-1 & sp_ivw==1 
replace adl_bed_diff=-7 if beddif==-7 
replace adl_bed_diff=-8 if beddif==-8 
replace adl_bed_not_done=inlist(dbedsfdf,1,8) 

label var adl_bed_help "Received help getting out of bed"
*replace adl_bed_diff=1 if adl_bed_help==1
notes adl_bed_help : mo*bedhlp
notes adl_bed_help : "_"
notes adl_bed_help : E:\nhats\nhats\code\NHATS data setup\NHATS_Setup, step-7 header

notes adl_bed_not_done: dbedsfdf
notes adl_bed_not_done: Get out of bed
notes adl_bed_not_done: Located in NHATS Setup, under Part 7.
notes adl_bed_not_done: Updated: 05/03/19-MH

//construct date of ivw--use 1st of month for all, because only have mo & yr
// can take out master when not beta
merge 1:1 spid wave using `resttrack', keep(master match) nogen
gen ivw_day=spstatdtdy
replace ivw_day=1 if missing(ivw_day)
label define ivw_day 1 "Not provided" 
label values ivw_day ivw_day
gen ivw_month=.
gen ivw_year=.
forvalues w=1/$w {
	replace ivw_month=r`w'spstatdtmt if wave==`w'
	replace ivw_month=r`w'fqstatdtmt if wave==`w' & r`w'spstatdtmt==.
	replace ivw_year=r`w'spstatdtyr if wave==`w'
	replace ivw_year=201`w' if wave==`w' & r`w'spstatdtyr==.
}

replace ivw_day=30 if ivw_day==31 & inlist(ivw_month,4,6,9,11)
replace ivw_day=28 if inrange(ivw_day,30,31) & ivw_month==2
replace ivw_month=r1fqstatdtmt if wave==1 & ivw_month==.
replace ivw_year=r1fqstatdty if wave==1 & ivw_year==.


gen ivw_date=mdy(ivw_month,ivw_day,ivw_year)
format ivw_date %td
label var ivw_date "Interview date from restricted tracker"

note ivw_date: ivw_month ivw_day ivw_year
note ivw_date: Interview date from the restricted file. 

gen ivw_date_sens=mdy(ivw_month,1,ivw_year)
format ivw_date_sens %td
label var ivw_date "Interview date, day of month set to one"

note ivw_date_sens: ivw_month ivw_year 
note ivw_date_sens: Using only month and year, all dates put as first of the month. 

foreach x in ivw_date ivw_date_sens{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/10/19-MH
} 

/// Changes to Dressing and Eating
/*These changes were made because the help variables were not capturing everyone 
who may have been impaired for the ADL*/ 
//changes made 5/20/18

	replace adl_dres_help=1 if dresoft==5
	replace adl_eat_help=1 if eatdev==7
	replace adl_ins_help=1 if ntlvrmslp==1


gen adl_index=0
foreach i in eat bath toil dres bed ins {
replace adl_index=adl_index+adl_`i'_help if adl_`i'_help>0
}
*gen adl_index=adl_eat_help+adl_bath_help+adl_toil_help+adl_dres_help+adl_ins_help+adl_bed_help
replace adl_index=. if adl_index<0
gen adl_independent=adl_index==0 if adl_index!=.
gen adl_impair=adl_index>0 if adl_index!=.
gen iadl_index=iadl_laun_help+iadl_shop_help+iadl_bank_help+iadl_meal_help+iadl_meds_help
replace iadl_index=. if iadl_index<0
gen iadl_independent=iadl_index==0 if iadl_index!=.
gen iadl_impair=iadl_index>0 if iadl_index!=.


egen adl_diff_index=rowtotal(adl_eat_diff adl_bath_diff adl_toil_diff adl_dres_diff adl_bed_diff adl_ins_diff)
gen adl_diff_ind=adl_diff_index>=1 if !missing(adl_diff_index)

egen iadl_diff_index=rowtotal(iadl_*_diff)
gen iadl_diff_ind=`x'iadl_diff_index>=1 if !missing(iadl_diff_index)

label var adl_independent "Independent in ADLs"
label var adl_impair "Help with 1+ ADL"
label var adl_index "Index of help w/ ADLs"

foreach x in independent impair index {
	notes adl_`x' : sc*eathlp sc*bathhlp sc*toilhlp sc*dreshlp
	notes adl_`x' : "_"
	notes adl_`x' : E:\nhats\nhats\code\NHATS data setup\NHATS_Setup, step-7 header
}

note adl_index: adl_*_help
note adl_index: Number of ADL's needing helped with.

note adl_independent: adl_index =0 
note adl_independent: No ADL help for the individual.

note adl_impair: adl_index >0 
note adl_impair: ADl help for the individual.

note iadl_index: iadl_*_help
note iadl_index: Number of IADL's needing helped with.

note iadl_independent: iadl_index =0 
note iadl_independent: No IADL help for the individual.

note iadl_impair: iadl_index >0 
note iadl_impair: IADl help for the individual.

label var iadl_independent "Independent in IADLs"
label var iadl_impair "Help with 1+ IADL"
label var iadl_index "Index of help w/ IADLs"

foreach x in adl_index adl_independent adl_impair iadl_index iadl_independent iadl_impair{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/10/19-MH
} 

//addition of variables from Eric's homebound code
gen meals_wheels=.
gen noone_talk=.
gen restrnt_takeout=.


replace meals_wheels=dmealwhl 
replace noone_talk=noonetalk
replace restrnt_takeout=dmealtkot 
foreach x in meals_wheels noone_talk restrnt_takeout {
replace `x'=0 if `x'==.
}

notes meals_wheels: dmealwhl
notes meals_wheels: Helper is meals on wheels

notes noone_talk: noonetalk
notes noone_talk: No one to talk to

notes restrnt_takeout: dmealtkot
notes restrnt_takeout: Helper is Restaurant takeout

foreach x in meals_wheels noone_talk restrnt_takeout{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/10/19-MH
}

label var meals_wheels "Meals on wheels"
label var noone_talk "SP has no one to talk to"
label var restrnt_takeout "Received help with restaurant/takeout"
//pull in date of death, constructed from claims and nhats
merge m:1 spid using "D:\NHATS\Shared\raw\CMS\NHATS CMS DUA 28016\Merged\STATA\death_date.dta", nogen

gen died_12=mofd(death_date)-mofd(ivw_date)<=12
gen died_24=mofd(death_date)-mofd(ivw_date)<=24

notes died_12: death_date ivw_date
notes died_12: death_date-ivw_date<=12

notes died_24: death_date ivw_date
notes died_24: death_date-ivw_date<=24

foreach x in died_12 died_24{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/10/19-MH
}

//note-the following has been superseded by the death date using claims
/*

//use status to get death
gen died_12=.
gen died_24=.
forvalues i=1/3 {
replace died_12=inlist(r`=`i'+1'status,62,86) if wave==`i' & r`=`i'+1'status!=79
}
forvalues i=1/2 {
replace died_24=inlist(r`=`i'+2'status,62,86) if wave==`i' & !inlist(r`=`i'+2'status,79,.)
}
replace died_24=1 if died_12==1

foreach i in 12 24 {
label var died_`i' "SP died w/in `i' months, per future wave status"
}*/

//update 8/2/17--pulled in nhats dod to share off server

//look at death month & year. don't have, so set date to last of month
gen dd=.
replace pd2yrdied=pd2yrdied+2010 if pd2yrdied>0

gen nhats_death_day=. 
forvalues w=2/$w {
	foreach j in mth yr {
		replace pd`w'`j'died=. if pd`w'`j'<0
}
	replace nhats_death_day=31 if inlist(pd`w'm,1,3,5,7,8,10,12)
	replace nhats_death_day=30 if inlist(pd`w'm,4,6,9,11)
	replace nhats_death_day=28 if inlist(pd`w'm,2)
	replace dd=mdy(pd`w'mthdied,nhats_death_day,pd`w'yrdied) if pd`w'yrdied>0 & pd`w'yrdied!=.
}

by spid, sort: egen nhats_death_date=max(dd)
label var nhats_death_date "Sensitive: SP death date, from NHATS (day imputed)"
format nhats_death_date %td
gen nhats_death_month=month(nhats_death_date)
gen nhats_death_year=year(nhats_death_date)
foreach x in month year {
label var nhats_death_`x' "Sensitive: SP death `x', from NHATS"
}

notes nhats_death_date: pdmthdied nhats_death_day pdwyrdied
notes nhats_death_date: Death date from sensitive variables. 
notes nhats_death_date: Located in NHATS Setup, under Part 7.
notes nhats_death_date: Updated: 04/11/19-MH 

drop dd

gen eligible_nsoc=fl1dnsoc==1
gen completed_nsoc=fl1dnsoccomp>0 if fl1dnsoccomp!=.
gen total_caregiver_comp=fl1dnsoccomp if fl1dnsoccomp>0 & fl1dnsoccomp!=.
gen total_caregiver_elig=fl1dnsoccnt if fl1dnsoccnt>0 & fl1dnsoccnt!=.
label var eligible  "Sensitive: Eligible for NSOC interview, wave 1 tracker"
label var complete  "Sensitive: Completed Interview"
label var total_caregiver_comp "Sensitive: Total Caregivers that Completed Interstatus"

notes eligible_nsoc: fl1dnsoc
notes eligible_nsoc: Eligible for NSOC

notes completed_nsoc: fl1dnsoccomp
notes completed_nsoc: Had an NSOC interview. 

notes total_caregiver_comp: fl1dnsoccomp
notes total_caregiver_comp: Count of caregivers NSOC.

notes total_caregiver_elig: fl1dnsoccnt
notes total_caregiver_elig: Count of helpers eligible NSOC.

foreach x in eligible_nsoc completed_nsoc total_caregiver_comp total_caregiver_elig{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}

//adverse consequences of unmet need

gen adverse=0
forvalues w=1/$w {
	foreach x of varlist launwout shopwout mealwout bankwout eatwout bathwout toilwout dreswout {
		replace adverse=1 if `x'==1 & wave==`w'
}
}
label var adverse "Any adverse consequence of unmet need (excluding mobility)

local ha laun shop meal bank
local sc eat bath toil dres
local mo out insd bed

forvalues w=1/$w{
	foreach x in `ha' `sc' `mo'{
		capture gen adverse_`x'=0
		replace adverse_`x'=1 if `x'wout==1 & wave==`w'
		
	}
}
	
label var adverse_laun "Ever go w/o clean laundry"
label var adverse_shop "Ever go w/o groceries/personal items"
label var adverse_meal "Ever go w/o hot meal"
label var adverse_bank "Ever go w/o handling bills and banking matters" 
label var adverse_eat "Ever go w/o eating"
label var adverse_bath "Ever go w/o showering/bath/washup"
label var adverse_toil "Ever go w/o using toilet"
label var adverse_dres "Ever go w/o getting dressed"
label var adverse_out "Ever go w/o going outside; stayed inside"
label var adverse_insd "Ever go w/o moving inside b/c unmet need" 
label var adverse_bed "Ever go w/o leaving bed"

notes adverse: launwout shopwout mealwout bankwout eatwout bathwout toilwout dreswout
notes adverse: Ever go without doing a task from the list. 
notes adverse_laun: launwout
notes adverse_laun: Ever go without clean laundry, b/c too difficult or no one there to help.
notes adverse_shop: shopwout
notes adverse_shop: Ever go without groceries/personal items, b/c too difficult or no one there to help.
notes adverse_meal: mealwout
notes adverse_meal: Ever go without hot meal, b/c too difficult or no one there to help.
notes adverse_bank: bankwout
notes adverse_bank: Ever go without handling bills and banking matters, b/c too difficult or no one there to help.
notes adverse_eat: eatwout
notes adverse_eat: Ever go without eating, b/c too difficult or no one there to help.
notes adverse_bath: bathwout
notes adverse_bath: Ever go without showering/taking a bath/washing up, b/c too difficult or no one there to help.
notes adverse_toil: toilwout
notes adverse_toil: Ever wet/soil self b/c no help or too difficult to get to toilet by self.
notes adverse_dres: dreswout
notes adverse_dres: Ever go without getting dressed, b/c too difficult or no one there to help.
notes adverse_out: outwout
notes adverse_out: Ever go without leaving home/building, b/c too difficult or no one there to help.
notes adverse_insd: insdwout
notes adverse_insd: Ever go without going into any places in home/building, b/c too difficult or no one there to help.
notes adverse_bed: bedwout
notes adverse_bed: Ever go without leaving your bed, b/c too difficult or no one there to help.

foreach x in adverse adverse_laun adverse_shop adverse_meal adverse_bank adverse_eat ///
adverse_bath adverse_toil adverse_dres adverse_out adverse_insd adverse_bed{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}

//limited favorite activity
capture drop lim_fav
gen lim_fav=.

forvalues w=1/5 { 
	replace lim_fav=helmfvact==1 if helmfvact>=0 & wave==`w'
}
label var lim_fav "Last month ever kept from favorite activity by health/function"

capture drop lim_fav_yr
////NEW limited favorite activity
gen lim_fav_yr=.
forvalues w=6/7 {
	replace lim_fav_yr=fvactlimyr==1 if fvactlimyr>=0 & wave==`w'
}

label var lim_fav_yr "Last year ever kept from favorite activity by health/function"

notes lim_fav: helmfvact
notes lim_fav: Did health or functioning ever keep you from favorite activity, last month

notes lim_fav_yr: helmfvact
notes lim_fav_yr: Did health or functioning ever keep you from favorite activity, last year.

 foreach x in lim_fav lim_fav_yr {
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}



//social cohesion
gen cohesion_knowwell=.
gen cohesion_willnghlp=.
gen cohesion_peoptrstd=.

forvalues w=1/$w {
	foreach x in knowwell willnghlp peoptrstd {
		replace cohesion_`x'=`x' if !missing(`x') & wave==`w'
}
}
rename cohesion_willnghlp cohesion_willing
rename cohesion_peoptrstd cohesion_peop

gen cohesion=(cohesion_p+cohesion_k+cohesion_w==3) ///
if !missing(cohesion_k) | !missing(cohesion_w) | !missing(cohesion_p)

label var cohesion_knowwell "People know each other well in the community"
label var cohesion_w "People are willing to help each other in the commmunity"
label var cohesion_p "People can be trusted in the community"
label define cohesion_ 1 "Agree a lot" 2 "Agree a little" 3 "Do not agree"
label values cohesion_* cohesion_
label var cohesion "High community cohesion (Highest on all 3 statements)"

notes cohesion_knowwell: knowwell
notes cohesion_knowwell: People in the community know each other well. Agree a lot, a little, do not agree.
notes cohesion_willing: willinghlp
notes cohesion_willing: People in the community are willing to help each other. Agree a lot, a little, do not agree.
notes cohesion_peop: peoptrstd
notes cohesion_peop: People in the community can be trusted. Agree a lot, a little, do not agree.
notes cohesion: knowwell willinghlp peoptrstd
notes cohesion: Agree a lot on all to get high community cohesion. 

 foreach x in cohesion_knowwell cohesion_willing cohesion_peop cohesion {
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}

gen home_disorder_insd=.
gen home_disorder_outsd=.
gen home_disorder_area=.
gen home_disorder_clutter=.

foreach x of varlist condhome* {
replace `x'=. if `x'==7
replace `x'=0 if `x'==2
}
foreach x of varlist areacond* {
replace `x'=. if `x'==5
replace `x'=0 if `x'==1
replace `x'=`x'>1 if !missing(`x')
}
foreach x of varlist clutter* {
replace `x'=. if `x'==4
replace `x'=0 if `x'==1
replace `x'=`x'>1 if !missing(`x')
}
foreach x of varlist condihom* {
replace `x'=`x'==1 if !missing(`x')  
}
/*
//forvalues w=1/$w{
	foreach x of varlist condihom* condhome* areacond* clutter* {
		replace `x'=. if   & wave==`w'
		qui sum `x'
		if r(max)==2 replace `x'=`x'==1 if !missing(`x') & wave==`w'
		if inlist(r(max),3,4) replace `x'=`x'>1 if !missing(`x') & wave==`w'
}
*/
	egen insd=rowtotal(condihom*), m 
	egen outsd=rowtotal(condhome*), m 
	egen area=rowtotal(areacond*), m 
	egen clutter=rowtotal(clutter*), m 
	

	foreach x in insd outsd area clutter {
		replace home_disorder_`x'=`x'>=1 if !missing(`x')
		drop `x'
}

notes home_disorder_outsd: condhome1 condhome2 condhome3 condhome4 condhome5 condhome6
notes home_disorder_outsd: If yes to any problems, then flagged
notes home_disorder_area: areacond1 areacond2 areacond3 areacond4
notes home_disorder_area: If yes to any problems, then flagged 
notes home_disorder_clutter: clutterr1 clutterr2
notes home_disorder_clutter: If yes to any problems, then flagged 
notes home_disorder_insd: condihom1 condihom2 condihom3 condihom4 condihom5
notes home_disorder_insd: If yes to any problems, then flagged 

 foreach x in home_disorder_outsd home_disorder_area home_disorder_clutter home_disorder_insd {
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}

label var home_disorder_insd "Disorder in home (peelng paint, pests, brkn furniture or floorbrds, trippng hazards)"
label var home_disorder_outsd "Disorder outside (brkn windows, foundation, bricks/siding, roof, steps)"
label var home_disorder_clutter "Disorder in home (clutter in interview or other rooms)"
label var home_disorder_area "Disorder in immed area (litter/glass/graffiti/vacant houses/stores/foreclosures)"	

//rename finhlpfam finhlpfam_old
//rename fingftfam fingftfam_old
//economic well-being---look at this section for strangeness 3/1/19
foreach x in pycredbal crecardeb credcdmed amtcrdmed medpaovtm ampadovrt ///
whohelfi1 whohelfi2 atchhelyr whregoth1 whregoth2 ///
whregoth3 amthlpgiv progneed1 progneed2 progneed3 mealskip ///
mealskip2 nopayhous nopayutil nopaymed {
	replace `x'=. if `x'<0
}

foreach x in creditdebt medcreditdebt medbillsovertime medpaynotcash ///
govtasst {
	gen `x'=.
}

replace creditdebt=inlist(pycredbal,2,3) | !missing(crecardeb) if ///
 !missing(pycredbal) | !missing(crecardeb)
replace medcreditdebt=credcdmed==1 if !missing(credcdmed)
replace medbillsovertime=medpaovtm==1 if !missing(medpaovtm)
replace medpaynotcash=medbillsovertime==1 | medcreditdebt==1 if !missing(medbillsovertime) | ///
!missing(medcreditdebt)

gen finhlpfam_n=finhlpfam==1 if !missing(finhlpfam)
gen fingftfam_n= fingftfam==1 if !missing(fingftfam)
replace govtasst=progneed1==1 | progneed2==1 | progneed3==1 if !missing(progneed1) ///
| !missing(progneed2) | !missing(progneed3)


replace medpaynotcash=medbillsovertime==1 | medcreditdebt==1 if !missing(medbillsovertime) | !missing(medcreditdebt)

label var creditdebt "Has credit card debt or does not pay off in full monthly"
label var medpaynotcash "Has medical credit card debt or pays bills over time"
label var govtasst "Receives foodstamps or other food or gas/energy assistance"

notes creditdebt: pycredbal crecardeb
notes creditdebt: Have some balance on credit card.
notes medpaynotcash: credcdmed
notes medpaynotcash: Any amount owed on credit cards for medical care.
notes govtasst: progneed1 progneed2 progneed3
notes govtasst: Recieved food stamps, other food assistance, or gas energy assistance. 
notes medcreditdebt: credcdmed
notes medcreditdebt: Any amount owed on credit cards for medial care.
notes medbillsovertime: medpaovtm
notes medbillsovertime: Have any medical bills being paid off over time. 
 foreach x in creditdebt medpaynotcash govtasst medcreditdebt medbillsovertime{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}

drop finhlpfam fingftfam
rename finhlpfam_n finhlpfam
rename fingftfam_n fingftfam

notes finhlpfam: finhlpfam
notes finhlpfam: Last year, you/spouse recieve any financial help or gifts from children/relatives; ///
either regulary or as needed.
notes fingftfam: fingftfam
notes fingftfam: Last year, you/spouse give any financial help or gifts to children/relatives; ///
either regulary or as needed.
 foreach x in finhlpfam fingftfam{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}

//home ownership

local homevars ownhome rent mortgagepaidoff paysmortgage /*mortpaymt*/ ///
timeinmort owedonmort homevalue_n rentpaid section8

foreach x of local homevars {
	gen `x'=.
}

label var ownhome "SP owns home"
label var rent "SP rents"
label var mortgagepaidoff "Mortgage is paid off"
label var mortpaymt "Mortgage payment"
label define mortpaymt 1 "<$250" 2 "$250-499" 3 "$500-999" 4 "$1000+"
label values mortpaymt mortpaymt
label var timeinmort "Time left in mortgage"
label define timeinmort 1 "5 years" 2 "10 years" 3 "Longer than 10 years"
label values timeinmort timeinmort
label var owedonmort "Amount left on mortgage"
label define owedonmort 1 "<$50,000" 2 "$50,000-99,999.99" 3 "$100,000+"
label values owedonmort owedonmort
label var homevalue "Home value"
label define homevalue_n 1 "<$50K" 2 "50-75K" 3 "75-100K" 4 "100-200K" 5 "200-300K" ///
6 "300-500K" 7 "500-750K" 8 "$750K+"
label values homevalue_n homevalue_n

forvalues w=1/$w {
tab ownrentot 
replace ownhome=ownrentot==1 if wave==`w' & (!missing(ownrento) | res_care==1)
replace rent=ownrentot==2 if wave==`w' & (!missing(ownrento) | res_care==1)
tab mrtpadoff 
replace mortgagepaidoff=mrtpadoff==1 if !missing(mrtpadoff)
replace paysmortgage=mrtpadoff==2 if !missing(mrtpadoff)
tab mortpaymt 
replace mortpaymt= mortpaymt if wave==`w'
qui replace mortpaymt=1 if wave==`w' & mthlymort<250
qui replace mortpaymt=2 if wave==`w' & mthlymort>=250 & mthlymort<=499
qui replace mortpaymt=3 if wave==`w' & mthlymort>=500 & mthlymort<1000
qui replace mortpaymt=4 if wave==`w' & mthlymort>=1000 & !missing(mthlymort)
tab whnpayoff 
replace timeinmort=whnpayoff if wave==`w'
tab amoutowed 
replace owedonmort=amoutowed if wave==`w'
qui replace owedonmort=1 if wave==`w' & amtstlowe<50000 
qui replace owedonmort=2 if wave==`w' & amtstlowe>=50000 & amtstlowe<=100000
qui replace owedonmort=3 if wave==`w' & amtstlowe>=100000 & !missing(amtstlowe)
tab homvalamt 
replace homevalue_n= homvalamt if wave==`w'
qui replace homevalue_n=1 if wave==`w' & homevalue<50000 & homevalue>=0
qui replace homevalue_n=2 if wave==`w' & homevalue>=50000 & homevalue<75000
qui replace homevalue_n=3 if wave==`w' & homevalue>=75000 & homevalue<100000
qui replace homevalue_n=4 if wave==`w' & homevalue>=100000 & homevalue<200000
qui replace homevalue_n=5 if wave==`w' & homevalue>=200000 & homevalue<300000
qui replace homevalue_n=6 if wave==`w' & homevalue>=300000 & homevalue<500000
qui replace homevalue_n=7 if wave==`w' & homevalue>=500000 & homevalue<750000
qui replace homevalue_n=8 if wave==`w' & homevalue>=750000 & !missing(homevalue)

tab rentamou 
replace rentpaid =rentamou if wave==`w'
qui replace rentpaid=1 if wave==`w' & rentamt<250 & rentamt>=0
qui replace rentpaid=2 if wave==`w' & rentamt>=250 & rentamt<=499
qui replace rentpaid=3 if wave==`w' & rentamt>=500 & rentamt<1000
qui replace rentpaid=4 if wave==`w' & rentamt>=1000 & !missing(rentamt)
qui replace rentpaid=4 if inlist(rentpaid,5,6)
replace mortpaymt=4 if inlist(mortpaymt,5,6)
tab sec8pubsn
replace section8=sec8pubsn==1 if wave==`w' & (!missing(sec8pubsn) |ownhome==1 | rent==1)
}

drop homevalue
rename homevalue_n homevalue

notes ownhome: ownrentot
notes ownhome: Do you/spouse/partner own home (apartment/condo).
notes rent: ownrentot
notes rent: Do you/spouse/partner rent home (apartment/condo).
notes mortgagepaidoff: mrtpadoff
notes mortgagepaidoff: Is your/spouse/partner mortage paidoff.
notes paysmortgage: mrtpadoff
notes paysmortgage: Are you/spouse/partner still making mortgage payments. 
notes mortpaymt: mthlymort
notes mortpaymt: How much monthly mortgage payment
notes owedonmort: amoutowed amtstlowe
notes owedonmort: Amount owed on mortgage. 
notes homevalue: homvalamt
notes homevalue: What is the present value of the home. Amount if sold today.
notes rentpaid: rentamt
notes rentpaid: Rent paid each month.
notes section8: sec8pubsn
notes section8: Home in section 8 or public housing or housing for low-income seniors. 
notes timeinmort: whnpayoff
notes timeinmort: How much longer are you planning to be in mortgage. (5, 10, 10+)
 foreach x in ownhome rent mortgagepaidoff paysmortgage mortpaymt owedonmort homevalue rentpaid section8 timeinmort{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}

label var rentpaid "Amount rent paid"
label define rentpaid 1 "<$250" 2 "250-499" 3 "500-999" 4 "$1000+"
label values rentpaid rentpaid
label var section8 "Indicator lives in Section 8 housing"
label var medcreditdebt "Has Medical credit card debt"
label var medbillsovertime "Pays Medical bills over time"
label var finhlpfam "Received financial help/gifts from family"
label var fingftfam "Provided financial help/gifts to family"
label var paysmortgage "Pays a mortgage"
gen n_social_network=.

replace n_social_network=dnumsn if dnumsn>=0 

label var n_social_network "Number in social network"

gen division=.

replace division=dcensdiv if dcensdiv>0 

gen region=division 
recode region (1 2=1) (3 4=2) (5/7=3) (8 9=4)

la define region 1 "Northeast" 2 "Midwest" 3 "South" 4 "West"
label values region region

capture gen northeast=0
capture gen midwest=0
capture gen south=0
capture gen west=0

replace northeast=1 if region==1
replace midwest=1 if region==2
replace south=1 if region==3
replace west=1 if region==4

notes division: dcensdiv
notes division: Census division
notes region: dcensdiv
notes region: 4 regions
notes n_social_network: dnumsn
notes n_social_network: Number in social network
foreach x in northeast midwest south west {
notes `x': dcensdiv
notes `x': `x' census division
}
foreach x in division n_social_network northeast midwest south west region {
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/11/19-MH
}
/*and now drop all raw variables
forvalues i=1/7 {
	drop r`i'* ip`i'* is`i'* re`i'* hc`i'* ht`i'* hh`i'* ss`i'* fl`i'* ///
	 pc`i'* cp`i'* cg`i'* mo`i'* ha`i'* sc`i'* mc`i'* pa`i'* sd`i'* hw`i'* ///
	 cm`i'* sn`i'* hp`i'* ew`i'* ir`i'* dt`i'* wb`i'* wa`i'* gr`i'* ho`i'* cs`i'* 
}

drop el1* rl1* ia1* ep2* pd2* pd3* pd4* pd5* pd6* pd7* ia3* ia5* ia7*
*/
//rename round*status r*status

svyset varunit [pw=anfinwgt], strat(varstrat)

sort spid wave
by spid: carryforward educ*, replace
label var educ_hs_ind "Education: HS+"
notes drop educ*

//10/25/18--add in payer and other variables from Jenny's homebound project
preserve
/*local mo outoft outcane outwalkr outwlchr outsctr outhlp outslf outdif outyrgo ///
outwout oftgoarea oflvslepr insdcane insdwalkr insdwlchr insdsctr oftholdwl ///
insdhlp insdslf insddif insdyrgo insdwout beddev bedhlp bedslf beddif bedwout

local sc eatdevoft eathlp eatoft eatdif eatwout showrbat1 showrbat2 showrbat3 ///
prfrshbth scusgrbrs shtubseat bathhlp bathoft bathdif bathyrgo bathwout ///
usvartoi1 usvartoi2 usvartoi3 usvartoi4 toilhlp toiloft toildif toilwout ///
dresoft dresdev dreshlp dresyrgo dreswout 

local ha laun launslf whrmachi1 whrmachi2 whrmachi3 whrmachi4 whrmachi5 ///
whrmachi6 laundif launoft launwout shop shopslf howpaygr1 howpaygr2 howpaygr3 ///
 howpaygr4 howpaygr5 howpaygr6 howpaygr7 howgtstr1 howgtstr2 howgtstr3 howgtstr4 ///
 howgtstr5 howgtstr6 howgtstr7 howgtstr8 shopcart shoplean shopdif shopoft shopwout ///
 meal mealslf restamels oftmicrow mealdif mealoft mealwout bank bankslf bankdif ///
 bankoft bankwout money moneyhlp

local mc meds medstrk medsslf whrgtmed1 whrgtmed2 whrgtmed3 whrgtmed4 howpkupm1 ///
howpkupm2 howpkupm3 medsrem medsdif medsyrgo medsmis havregdoc regdoclyr hwgtregd1 ///
 hwgtregd2 hwgtregd3 hwgtregd4 hwgtregd5 hwgtregd6 hwgtregd7 hwgtregd8 hwgtregd9 ///
 ansitindr tpersevr1 tpersevr2 tpersevr3 tpersevr4 chginspln anhlpwdec
*/

forvalues wave=1/$w {
use "D:\NHATS\Shared\raw\NHATS\NHATS Public\round_`wave'\NHATS_Round_`wave'_SP_File.dta", clear
keep spid cs`wave'dnumchild ho`wave'entrst hh`wave'dhshldnum mo* sc* ha* mc* 
gen wave=`wave'
rename cs`wave' n_children
label var n_ch "Number of living children"
rename ho`wave' stairs_ind
label var stairs "Stairs to enter home"
gen livesalone_ind=hh`wave'dhshldnum==1 if hh`wave'dhshldnum>=0
label var livesal "Lives alone (only one in household)"
foreach x in n sta live {
replace `x'=. if `x'<0
}

local kidvars n_children stairs_ind livesalone_ind
gen ind_kids=n_children>=1 if !missing(n_children)
rename (mo`wave'* sc`wave'* ha`wave'* mc`wave'*) (* * * *)
rename (eatslfdif toiloft bathoft) (eatdif toilslf bathslf)

foreach x in eat toil insd bed dres bath out {
	gen `x'_jenny=inlist(`x'slf,3,4) | inlist(`x'dif,3,4) | `x'wout==1 |`x'hlp==1
}
egen adlcount=rowtotal(eat_j toil_j insd_j bed_j dres_j bath_j)

//for iadls, reason don't do alone b/c health/function or reported difficulty or went w/o
gen medswout=0
foreach x in laun meds shop meal bank {
	drop `x'
	gen `x'_jenny=inlist(d`x'reas,1,3) | inlist(`x'dif,3,4) | `x'wout==1 
}
drop medswout
egen iadlcount=rowtotal(laun_j meds_j shop_j meal_j bank_j)
label var eat_j "Some/lot difficulty eating by self, or help, or rarely/never, or had to go w/o"
label var toil_j "Some/lot difficulty toileting by self, or help, or rarely/never, or had to go w/o"
label var insd_j "Some/lot difficulty moving inside by self, or help, or rarely/never, or had to go w/o"
label var out_j "Some/lot difficulty going outside by self, or help, or rarely/never, or had to go w/o"
label var bed_j "Some/lot difficulty getting out of bed by self, or help, or rarely/never, or had to go w/o"
label var dres_j "Some/lot difficulty dressing by self, or help, or rarely/never, or had to go w/o"
label var bath_j "Some/lot difficulty bathing by self, or help, or rarely/never, or had to go w/o"
label var laun_j "Some/lot of difficulty with laundry by self, or not by self b/c health, or had to go w/o"
label var meds_j "Some/lot of difficulty with meds by self, or not by self b/c health, or had to go w/o"
label var shop_j "Some/lot of difficulty with shopping by self, or not by self b/c health, or had to go w/o"
label var meal_j "Some/lot of difficulty with hot meals by self, or not by self b/c health, or had to go w/o"
label var bank_j "Some/lot of difficulty with banking by self, or not by self b/c health, or had to go w/o"

gen device_mobility=inlist(insdwalk,1,2) | inlist(outwalk,1,2) | inlist(insdwl,1,2) ///
| inlist(outwl,1,2) | inlist(insdsct,1,2) | inlist(outsct,1,2) 

label var device_mobility "Uses a walker, wheelchair, or scooter inside or outside"

keep spid wave *_jenny device_mobility `kidvars' ind_kids adlcount iadlcount

tempfile kids`wave'
save `kids`wave''

use "D:\NHATS\Shared\raw\NHATS\NHATS Public\round_`wave'\NHATS_Round_`wave'_OP_File.dta", clear
describe op`wave'*pay*
tab op`wave'*payshlp
tab op`wave'progmpaid

rename op`wave'sppays op`wave'sppayhlp
foreach x in sp gov ins oth {
gen paid_by_`x'=op`wave'`x'payhlp==1 if op`wave'`x'payhlp>0 | op`wave'`x'payhlp<-1
local lab : var label op`wave'`x'payhlp
label var paid_by_`x' "`lab'"
}
gen paid_by_unk=(paid_by_sp+paid_by_oth+paid_by_gov+paid_by_ins)==0 | ///
((paid_by_sp+paid_by_oth+paid_by_gov+paid_by_ins)==-4 & op`wave'paid==1)
label var paid_by_unk "Unknown payer"


levelsof op`wave'progmpaid, local(levels)
foreach l of local levels {
if `l'>0 {
gen paid_by_gov_cat_`l'=op`wave'progmpaid==`l' if op`wave'progmpaid!=-1
local lab : label op`wave'progmpaid `l'
label var paid_by_gov_cat_`l' "`lab'"
}
}
gen paid_by_gov_cat_unknown=op`wave'progmpaid<-1 if op`wave'progmpaid!=-1
label var paid_by_gov_cat_unk "Unknown gov't payer (Missing,RF,DK)"
gen op_insur_help=op`wave'insurhlp==1 
gen op_out_help=op`wave'outhlp==1
gen op_money_help=op`wave'moneyhlp==1
gen op_places_help=op`wave'tkplhlp1==1 | op`wave'tkplhlp2==1
label var op_insur_help "Help with insurance decisions"
keep spid opid paid* op_*
tempfile paid
save `paid'
gen wave=`wave'
merge 1:m spid opid wave using "D:\NHATS\Shared\base_data\NHATS cleaned\op_round_1_7.dta", ///
keep(match master) gen(opmer)
merge 1:1 spid opid using `paid', keep(match master) gen(paidm)
foreach x of varlist paid_by* {
replace `x'=0 if paid==0
}
drop if n_helpers>=1 & opishelp==0

foreach x in sp gov ins unk oth {
by spid, sort: egen any_paid_by_`x'=max(paid_by_`x')
local lab :var label paid_by_`x'
label var any_paid_by_`x' "`lab'"
}
rename paid_by_gov_cat_91 paid_by_gov_cat_oth
foreach x of varlist paid_by_gov_cat* {
by spid, sort: egen any_`x'=max(`x')
local lab :var label `x'
label var any_`x' "`lab'"
}


by spid: egen any_paid_by_medicare=max(paid_by_gov_cat_2)
label var any_paid_by_medicare "Helper paid by Medicare"
by spid: egen any_paid_by_medicaid=max(paid_by_gov_cat_1)
label var any_paid_by_medicaid "Helper paid by Medicaid"
by spid: egen any_paid_by_state=max(paid_by_gov_cat_3)
label var any_paid_by_state "Helper paid by state program"
gen ind_gt20hrs_wk_inf=tot_hrswk_help_i>20 if !missing(tot_hrswk_help_i)
replace ind_gt20hrs_wk_inf=0 if tot_hrswk_help_i-tot_hrswk_paid_i<20
gen ind_gt20hrs_wk_paid=tot_hrswk_paid_i>20 if !missing(tot_hrswk_paid_i)
label var ind_gt20hrs_wk_i "Unpaid help: >20 hrs wk"
label var ind_gt20hrs_wk_p "Paid help: >20 hrs wk"
gen ind_inf=n_helpers>=1 if !missing(n_helpers)
replace ind_inf=0 if n_paid==n_helpers
label var ind_inf "Any family/unpaid help"
local pvars ind_inf ind_gt20hrs_wk_inf ind_paid ind_gt20hrs_wk_paid ///
any_paid_by_sp any_paid_by_gov ///
any_paid_by_medicaid any_paid_by_medicare any_paid_by_state 
replace any_paid_by_unk=1 if (any_paid_by_sp+any_paid_by_ins+any_paid_by_oth ///
+any_paid_by_gov==0) & ind_paid==1
local paidvars any_paid_by_sp any_paid_by_gov any_paid_by_ins any_paid_by_oth ///
any_paid_by_unk any_paid_by_medicaid any_paid_by_medicare ///
any_paid_by_gov_cat_oth any_paid_by_gov_cat_unknown any_paid_by_unk 
keep spid any_paid* ind_gt* wave

duplicates drop 
tempfile paid`wave'
save `paid`wave''

use "D:\NHATS\Shared\base_data\NHATS cleaned\op_round_1_${w}" if wave==`wave' & opishelp==1, clear
tokenize spouse daughter son otherfamily paid otherinformal
drop spouse daughter son paid
forvalues i=1/6 {
	gen ``i''=op_relat==`i'
	by spid: egen ``i''_help_ind=max(``i'')
	label var ``i''_help "Has ``i'' helper"
	gen ``i''hrs=op_hrswk_i if ``i''==1
	by spid: egen tot_``i''hrs=total(``i''hrs) if ``i''_help_ind==1
	label var tot_``i''hrs "Hours of help/week from ``i'' sources"
	local ihelp `ihelp' ``i''_help_ind
	local chelp `chelp' tot_``i''hrs
}
label var otherfamily_he "Has other family helper"
label var otherinformal "Has non-family unpaid helper"
label var tot_otherf "Hours of help/week from other family sources"
label var tot_otheri "Hours of help/week from other unpaid sources"
keep spid `chelp' `ihelp' wave
duplicates drop

tempfile hours`wave'
save `hours`wave''
}

foreach x in hours paid kids {
	di "`x'"
	use ``x'1', clear
	forvalues i=2/$w {
		di "`i'"
		append using ``x'`i''
}
	di "`x'"
	tempfile `x'
	save ``x''
}

restore

merge 1:1 spid wave using `kids', keep(match master) gen(kidm)
merge 1:1 spid wave using `hours', keep(match master) gen(hrm)
merge 1:1 spid wave using `paid', keep(match master) gen(paidm)

// 9/18/2019--Adding social isolation and frailty derived vars

gen not_marriedpartnered=1 if marriedpartnered==0
gen not_visitfrfam=1 if visitfrfam==0
gen not_attrelig=1 if attrelig==0
gen not_attclub=1 if attclub==0

gen nhatswksc1=1 if inlist(nhatswksc,0,1)
replace nhatswksc1=0 if inrange(nhatswksc,2,4)

gen nhatsgrav1=1 if inlist(nhatsgrav,0,1)
replace nhatsgrav1=0 if inrange(nhatsgrav,2,4)
egen frail1=rowtotal(unwgtloss lowenergy lowphysact nhatswksc1 nhatsgrav1)

gen frail=0 if frail1==0
replace frail=1 if inlist(frail1,1,2)
replace frail=2 if frail1>2

tempfile nhats
save "`nhats'"

use "D:\NHATS\Shared\base_data\NHATS cleaned\op_round_1_8.dta" , clear

bys spid wave: egen no_fam_talk2=sum(op_sn) if inrange(op_relationship_cat,1,4)
bys spid wave: egen no_fam_talk1=max(no_fam_talk2)
replace no_fam_talk1=0 if no_fam_talk1==.

bys spid wave: egen no_friend_talk2=sum(op_sn) if inlist(op_relationship_cat,5,6)
bys spid wave: egen no_friend_talk1=max(no_friend_talk2)
replace no_friend_talk1=0 if no_friend_talk1==.

gen no_fam_talk=1 if no_fam_talk1==0
gen no_friend_talk=1 if no_friend_talk1==0

bys spid wave: keep if _n==1
keep spid wave no_fam_talk no_friend_talk
tempfile no_talk
save "`no_talk'"

use "`nhats'", clear
merge 1:1 spid wave using "`no_talk'", keepusing(no_fam_talk no_friend_talk)

egen soc_iso1=rowtotal(not_married not_visitfrfam not_attrelig not_attclub no_fam_talk no_friend_talk)
gen soc_iso=0 if soc_iso<4
replace soc_iso=1 if soc_iso>=4

notes soc_iso: marriedpartnered visitfrfam attrelig attclub op_sn op_relationship_cat
notes soc_iso: Social Isolation
notes soc_iso: Located in NHATS Setup, under Part 7.
notes soc_iso: Created: 09/18/19-MH
notes frail: unwgtloss lowenergy lowphysact nhatswksc nhatsgrav
notes frail: Frailty measure
notes frail: Located in NHATS Setup, under Part 7.
notes frail: Created: 09/18/19-MH

notes n_children: dnumchild
notes n_children: Number of children
notes stairs_ind: entrstair
notes stairs_ind: Entrance you use most often, are their stairs. 
notes livesalone_ind: dhshldnum
notes livesalone_ind: total number in household. 
notes ind_kids: dnumchild n_children
notes ind_kids: Indicator have children
foreach x in eat toil insd bed dres bath out {
notes `x'_jenny: `x'slf (3,4) `x'dif (3,4) `x'wout (1)
notes `x'_jenny: ADL if did not do or rarely do, needed help, go without doing. 
notes `x'_jenny: Located in NHATS Setup, under Part 7.
notes `x'_jenny: Updated: 08/23/19-EBL, 04/11/19-MH
}
notes adlcount: eat_jenny toil_jenny insd_jenny bed_jenny dres_jenny bath_jenny
notes adlcount: count of Jenny's ADL measures
foreach x in laun meds shop meal bank {
notes `x'_jenny: `x'reas (1,3) `x'dif (3,4) `x'wout (1)
notes `x'_jenny: ADL if did not do or rarely do, needed help, go without doing. 
notes `x'_jenny: Located in NHATS Setup, under Part 7.
notes `x'_jenny: Updated: 04/11/19-MH
}
notes iadlcount: laun_jenny meds_jenny shop_jenny meal_jenny bank_jenny
notes iadlcount: count of Jenny's IADL measures
notes device_mobility: insdwalk outwalk insdwl outwl insdsct outsct
notes device_mobility: If annswered most or everytime to having mobility device for ///
question indicated. 
notes any_paid_by_medicare: paid_by_gov_cat_2 from progmpaid 
notes any_paid_by_medicare: Medicare program paid for helper. 
notes any_paid_by_medicaid: paid_by_gov_cat_1 from progmpaid 
notes any_paid_by_medicaid: Medicaid program paid for helper. 
notes any_paid_by_state: paid_by_gov_cat_3 from progmpaid 
notes any_paid_by_state: State Program paid for helper. 
notes any_paid_by_sp: paid_by_sp from sppayshlp
notes any_paid_by_sp: SP pays for help
notes any_paid_by_gov: paid_by_gov from govpayshlp
notes any_paid_by_gov: Government pays for help.
notes any_paid_by_ins: paid_by_ins from inspayshlp
notes any_paid_by_ins: Insurance pays for help.
notes any_paid_by_oth: paid_by_oth from othpayshlp
notes any_paid_by_oth: Other pays for help.
notes any_paid_by_unk: paid_by_sp paid_by_oth paid_by_gov paid_by_ins ///
paid_by_sp paid_by_oth paid_by_gov paid_by_ins
notes any_paid_by_unk: Unknown who pays for help. 
notes otherfamily_help_ind: relat
notes otherfamily_help_ind: Other family helpers
notes spouse_help_ind: relat
notes spouse_help_ind: Spouse as helper.
notes daughter_help_ind: relat
notes daughter_help_ind: Daughter as helper
notes son_help_ind: relat
notes son_help_ind: Son as helper
notes daughter_help_ind: relat
notes daughter_help_ind: Daughter as helper
notes paid_help_ind: relat
notes paid_help_ind: Has paid helper
notes otherinformal: relat
notes otherinformal: Has other family helpers. 
notes tot_otherfamilyhrs: hrswk
notes tot_otherfamilyhrs: Other family hours. 
notes tot_otherinformalhrs: hrswk
notes tot_otherinformalhrs: From other unpaid sources. 
notes any_paid_by_gov_cat_1: op*progmpaid 
notes any_paid_by_gov_cat_1: Medicaid paid for helper.
notes any_paid_by_gov_cat_2: op*progmpaid 
notes any_paid_by_gov_cat_2: Medicare paid for helper.
notes any_paid_by_gov_cat_3: op*progmpaid 
notes any_paid_by_gov_cat_3: State Program paid for helper.
notes any_paid_by_gov_cat_oth: op*progmpaid 
notes any_paid_by_gov_cat_oth: Other program paid for helper.
notes any_paid_by_gov_cat_unknown: op*progmpaid 
notes any_paid_by_gov_cat_unknown: DK/RF, who paid for helper.
notes ind_gt20hrs_wk_inf: tot_hrswk_help_i
notes ind_gt20hrs_wk_inf: Greater than 20 hours of informal help, imputed
notes ind_gt20hrs_wk_paid: tot_hrswk_help_i
notes ind_gt20hrs_wk_paid: Greater than 20 hours of paid help, imputed
foreach x in n_children stairs_ind livesalone_ind ind_kids adlcount iadlcount ///
device_mobility any_paid_by_medicare any_paid_by_sp any_paid_by_gov any_paid_by_ins ///
any_paid_by_oth any_paid_by_unk otherfamily_help_ind spouse_help_ind daughter_help_ind ///
son_help_ind daughter_help_ind paid_help_ind otherinformal tot_otherfamilyhrs ///
tot_otherinformalhrs any_paid_by_gov_cat_1 any_paid_by_gov_cat_2 any_paid_by_gov_cat_3 any_paid_by_gov_cat_oth ///
any_paid_by_gov_cat_unknown any_paid_by_medicaid any_paid_by_state ind_gt20hrs_wk_inf ind_gt20hrs_wk_paid{ 
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/12/19-MH
}

label var aveincome "Income"
label var residence "Type of residence"
drop sr*raw ht_new pr_ad8* dem_via_proxy speaktosp date_item* date_sum* ///
preslast presfirst vp* presvp* date_prvp clock_scorer irecall drecall ///
wordrecall *65 kidm hrm paidm SP_completed deceased /*va*ser* el5 rl* */

foreach x in eat bath toil dres {
	label var adl_`x'_diff "Has difficulty while `x'ing last month"
	label var adl_`x'_not "Did not `x' by self in last month"
}

foreach x in laun shop meal bank meds {
	label var iadl_`x'_diff "Has difficulty while `x'ing last month"
	label var iadl_`x'_not "Did not `x' in last month"
}

//5/26/20-added here because code used is farther down. 
egen a1=rowtotal(toil_j dres_j bath_j meds_j bank_j)
gen severe=a1==5
gen sev_dem=severe==1 & prob_dem==1
label var sev_dem "Severe dementia"

notes sev_dem: toil_j dres_j bath_j meds_j bank_j prob_dem
notes sev_dem: Severe Dementia from Jenny''s Caregiving project
notes sev_dem: Located in NHATS Setup, under Part 7.
notes sev_dem: Created: 05/26/20-MH

label var iadl_meds_diff "Difficulty taking meds last month"
label var iadl_meds_not "Didn't take meds or not by self in last month"	
label var adl_ins_diff "Difficulty moving around inside by self in last month"
label var adl_ins_no "Didn't move around inside by self in last month"
label var adl_bed_dif "Difficulty getting out of bed by self in last month"
label var adl_bed_no "Didn't get out of bed by self in last month"
label var num_helpers_cat "Categorical number of helpers"
label var ivw_type "Type of interview"
label var income_cat "Categorical income"
label var trfinwgt "Tracker weight"
label var tr2011wgt "Tracker weight (2011 cohort)"
label var howgetdoc "Mode of transport to regular doctor"
label var ivw_day "Day of interview (Assigned to 1 for all)"
label var ivw_month "Month of interview"
label var ivw_year "Year of interview"
label var ivw_date "Date of interview (1st of the month assigned)"
label var community_dwelling "Community dwelling"
label var marital_sd "Separated or divorced"
label var iadl_diff_index "Count of IADL difficulties reported (missing coded as zero)"
label var iadl_diff_ind "Indicator of any IADL difficulty reported (missing coded as zero)"
label var adl_diff_index "Count of ADL difficulties reported (missing coded as zero)"
label var adl_diff_ind "Indicator of any ADL difficulty reported (missing coded as zero)"
label var nhats_death_day "Sensitive: Day of death from NHATS (Assigned to last day of month)"
label var nhats_death_month "Sensitive: Month of death from NHATS"
label var nhats_death_year "Sensitive: Year of death from NHATS"
label var division "Census division"
label var region "Region"
label var ind_kids "Indicator any living children"

rename adlcount adlcount_jenny 
label var adlcount_j "ADL count: some/lot of difficulty or adverse consequence of unmet need"
rename iadlcount iadlcount_jenny 
label var iadlcount_j "IADL count: some/lot of difficulty or adverse consequence of unmet need"

notes adl_bed_diff: mo*beddif
notes adl_bed_diff: How much difficulty getting out of bed. Any difficulty flagged. 
notes adl_ins_diff: mo*insddif
notes adl_ins_diff: How much difficulty getting around house. Any difficulty flagged. 
notes adl_eat_diff: eatslfdif
notes adl_eat_diff: How much difficulty eating by yourself. Any difficulty flagged. 
notes adl_bath_diff: sc*bathdif
notes adl_bath_diff: How much difficulty have bathing/showering. Any difficulty flagged. 
notes adl_toil_diff: sc*toildif
notes adl_toil_diff: How much difficulty using the toilet. Any difficulty flagged. 
notes adl_dres_diff: sc*dresdif
notes adl_dres_diff: How much difficulty getting dressed. Any difficulty flagged. 
notes iadl_laun_diff: ha*laundif
notes iadl_laun_diff: How much difficulty doing laundry. Any difficulty flagged. 
notes iadl_shop_diff: ha*shopdif
notes iadl_shop_diff: How much difficulty doing shop. Any difficulty flagged. 
notes iadl_meal_diff: ha*dmealdif
notes iadl_meal_diff: How much difficulty shopping for groceries or personal items. Any difficulty flagged. 
notes iadl_bank_diff: ha*bankdif
notes iadl_bank_diff: How much difficulty doing banking and handling bills. Any difficulty flagged.
notes iadl_meds_diff: mc*medsdif
notes iadl_meds_diff: How much difficulty keeping track of meds. Any difficulty flagged.
notes adl_diff_index: adl*index
notes adl_diff_index: Count of ADL difficulties. Missing is zero.
notes adl_diff_ind: adl_diff_index
notes adl_diff_ind: Indicator of any ADL difficulty reported. Missing is zero.
notes iadl_diff_index: iadl*index
notes iadl_diff_index: Count of IADL difficulties. Missing is zero.
notes iadl_diff_ind: iadl_diff_index
notes iadl_diff_ind: Indicator of any IADL difficulty reported. Missing is zero. 
notes ind_meds_diff: medsdif meds dmedsrea medstrk
notes ind_meds_diff: Any difficulty keping track of meds
notes ind_meds_mistake: medsmis
notes ind_meds_mistake: Mistake taking meds.
notes ind_meds_problem: ind_meds_diff ind_meds_mistake
notes ind_meds_problem: Keeping track of or mistake taking meds.

foreach x in adl_bed_diff adl_ins_diff adl_eat_diff adl_bath_diff adl_toil_diff adl_dres_diff ///
iadl_laun_diff iadl_shop_diff iadl_meal_diff iadl_bank_diff iadl_meds_diff adl_diff_index ///
adl_diff_ind iadl_diff_index iadl_diff_ind {
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 04/12/19-MH
}

// 11/8/18 Making National Estimates using 2011 population (using original weights) Rds 1-4
rename anfinwgt old_anfinwgt
gen double anfinwgt=old_anfinwgt
replace anfinw=old_anfinwgt*5491440/4649902 if inrange(age,65,69) & female==0 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*4087340/4097017 if inrange(age,70,74) & female==0 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*3027820/3001146 if inrange(age,75,79) & female==0 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*2169060/2173979 if inrange(age,80,84) & female==0 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*1194880/1224397 if inrange(age,85,89) & female==0 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*524540/463619 if inrange(age,90,150) & female==0 & inrange(wave,1,4)

replace anfinw=old_anfinwgt*6124980/5250466 if inrange(age,65,69) & female==1 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*4781940/4843444 if inrange(age,70,74) & female==1 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*3886940/3878674 if inrange(age,75,79) & female==1 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*3249940/3215654 if inrange(age,80,84) & female==1 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*2230480/2257969 if inrange(age,85,89) & female==1 & inrange(wave,1,4)
replace anfinw=old_anfinwgt*1382160/1329677 if inrange(age,90,150) & female==1 & inrange(wave,1,4)

// Making National Estimates using 2015 population (using original weights) Rds 5-7
replace anfinw=old_anfinwgt*6855860/5671878 if inrange(age,65,69) & female==0 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*4941100/5071147 if inrange(age,70,74) & female==0 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*3367300/3524465 if inrange(age,75,79) & female==0 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*2257880/2208455 if inrange(age,80,84) & female==0 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*1307620/1358952 if inrange(age,85,89) & female==0 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*648680/624319 if inrange(age,90,150) & female==0 & inrange(wave,5,7)

replace anfinw=old_anfinwgt*7601440/6226214 if inrange(age,65,69) & female==1 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*5695060/6077509 if inrange(age,70,74) & female==1 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*4182460/4196832 if inrange(age,75,79) & female==1 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*3165560/3137807 if inrange(age,80,84) & female==1 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*2253980/2157300 if inrange(age,85,89) & female==1 & inrange(wave,5,7)
replace anfinw=old_anfinwgt*1577880/1534438 if inrange(age,90,150) & female==1 & inrange(wave,5,7)

// Making National Estimates using 2011 Population (2011 Weights) for Rds 5-7
rename an2011wgt old_an2011wgt
gen an2011wgt=old_an2011wgt
replace an2011wgt=old_an2011wgt*5491440/4649902 if inrange(age,65,69) & female==0 & inrange(wave,5,7) 
replace an2011wgt=old_an2011wgt*4087340/4097017 if inrange(age,70,74) & female==0 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*3027820/3001146 if inrange(age,75,79) & female==0 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*2169060/2173979 if inrange(age,80,84) & female==0 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*1194880/1224397 if inrange(age,85,89) & female==0 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*524540/463619 if inrange(age,90,150) & female==0 & inrange(wave,5,7)

replace an2011wgt=old_an2011wgt*6124980/5250466 if inrange(age,65,69) & female==1 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*4781940/4843444 if inrange(age,70,74) & female==1 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*3886940/3878674 if inrange(age,75,79) & female==1 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*3249940/3215654 if inrange(age,80,84) & female==1 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*2230480/2257969 if inrange(age,85,89) & female==1 & inrange(wave,5,7)
replace an2011wgt=old_an2011wgt*1382160/1329677 if inrange(age,90,150) & female==1 & inrange(wave,5,7)

label var anfinwgt "Updated Analytic Weights"
label var an2011wgt "Updated 2011 Analytic Weight for Rds 5+"

notes anfinwgt : r*anfinwgt0
notes anfinwgt : Analytic final weight, post-stratified to most recent sample frame as in the technical paper
notes anfinwgt : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-7 header
notes anfinwgt : "_"
notes anfinwgt : Freedman, VA and BC Spillman. Making National Estimates... NHATS Technical Paper#17. 

notes an2011wgt : r*an2011wgt0
notes an2011wgt : Analytic final weight, post-stratified to most 2011 sample frame as in the technical paper
notes an2011wgt : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-7 header
notes an2011wgt : "_"
notes an2011wgt : Freedman, VA and BC Spillman. Making National Estimates... NHATS Technical Paper#17. 

notes old_anfinwgt : r*anfinwgt0
notes old_anfinwgt : Original analytic weight
notes old_anfinwgt : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-7 header
notes old_anfinwgt : "_"
notes old_anfinwgt : Freedman, VA and BC Spillman. Making National Estimates... NHATS Technical Paper#17. 

notes old_an2011wgt : r*an2011wgt0
notes old_an2011wgt : Original analytic weight for 2011 sample frame
notes old_an2011wgt : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-7 header
notes old_an2011wgt : "_"
notes old_an2011wgt : Freedman, VA and BC Spillman. Making National Estimates... NHATS Technical Paper#17. 


/*Add notes for raw variables or variables that were created in earlier headers*/
notes varstrat : r*varstrat
notes varstrat: Stratified sampling
notes varunit : r*varunit
notes wave : "_"
notes wave : Wave assigned to round number
notes wave : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-1 header
notes proxy_ivw : is*resptype
notes proxy_ivw : Is the interview a proxy? 1=yes
notes proxy_ivw : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-3 header
notes age : r*dintvwrage r*ddeathage
notes age : Age at interview if living or death if LML
notes age : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-3 header
notes educ_hs_ind : el*higstschl
notes educ_hs_ind : Completed HS+ (includes GED)
notes educ_hs_ind : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-3 header
notes white : rl*dracehisp
notes white : Non-Hispanic White
notes white : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-3 header
notes black : rl*dracehisp
notes black : Non-Hispanic Black
notes black : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-3 header
notes hisp : rl*dracehisp
notes hisp : Any Hispanic
notes hisp : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-3 header
notes other_race : rl*dracehisp
notes other_race : Any non-White, non-Black, non-Hispanic race
notes other_race : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-1 header
notes medicaid : ip*cmedicaid
notes medicaid : "_"
notes medicaid : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-3 header
notes medigap : ip*mgapmedsp
notes medigap : "_"
notes medigap : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-3 header
notes srh_fp : hc*health
notes srh_fp : SR health is poor or fair (as opposed to good, v. good, or excellent)
notes srh_fp : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-4 header
notes adl_eat_help : sc*eathelp
notes adl_eat_help : Help eating in the last month
notes adl_eat_help : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-5 header
notes adl_bath_help : sc*bathhelp
notes adl_bath_help : Help bathing in the last month
notes adl_bath_help : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-5 header
notes adl_toil_help : sc*toilhelp
notes adl_toil_help : Help toiling in the last month
notes adl_toil_help : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-5 header
notes adl_dres_help : sc*dreshelp
notes adl_dres_help : Help dresing in the last month
notes adl_dres_help : E:\nhats\nhats_code\NHATS_data_setup\NHATS_Setup.otl, Step-5 header
notes fall_last_month: fllsinmth
notes fall_last_month: Fallen down in last month.
notes fall_last_month: Located in NHATS Setup, under Part 5.
notes fall_last_month: Updated: 04/17/19-MH
notes adl_eat_not_done: eatslfoft
foreach x in bath toil dres{
notes adl_`x'_not_done: `x'oft
}
foreach x in eat bath toil {
notes adl_`x'_not_done: How often `x' by self and without help.
}
notes adl_dres_not_done: How often you dress.  
foreach x in eat bath toil dres{
notes adl_`x'_not_done: Located in NHATS Setup, under Part 5.
notes adl_`x'_not_done: Updated: 04/17/19-MH
}
foreach x in laun shop meal bank{ 
notes iadl_`x'_help: d`x'sfdf d`x'reas
notes iadl_`x'_help: `x' difficulty level and if `x' done by others
notes iadl_`x'_help: Located in NHATS Setup, under Part 5.
notes iadl_`x'_help: Updated: 04/17/19-MH
notes iadl_`x'_not_done: ha*`x'
notes iadl_`x'_not_done: How `x' made/completed.
notes iadl_`x'_not_done: Located in NHATS Setup, under Part 5.
notes iadl_`x'_not_done: Updated: 04/17/19-MH
}
notes iadl_meds_help: dmedssfdf dmedsreas
notes iadl_meds_help: Difficulty level of medications, medicine reason with others. 
notes iadl_meds_diff: medsdif
notes iadl_meds_diff: Difficulty in keeping track of medicines.
notes iadl_meds_not_done: dmedssfdf
notes iadl_meds_not_done: Difficulty level of medications
foreach x in help diff not_done{
notes iadl_meds_`x': Located in NHATS Setup, under Part 5.
notes iadl_meds_`x': Updated: 04/17/19-MH
}
notes reg_doc_have: haveregdoc
notes reg_doc_have: Do you have a go to doctor, who you go to for advice.
notes reg_doc_seen: regdoclyr
notes reg_doc_seen: Have you seen your regular doctor within the last year. 
notes reg_doc_homevisit: hwgtregd8
notes reg_doc_homevisit: Was doctor visit a home visit. 
foreach x in reg_doc_have reg_doc_seen reg_doc_homevisit{
notes `x': Located in NHATS Setup, under Part 5.
notes `x': Updated: 04/17/19-MH
}
notes sr_hosp_ind: hosptstay
notes sr_hosp_ind: Had an overnight hospital stay since last interview/year.
notes sr_hosp_stays: hosovrnht hosptstay
notes  sr_hosp_stays: How many seperate overnight hospital stays. 
foreach x in sr_hosp_ind sr_hosp_stay{
notes `x': Located in NHATS Setup, under Part 5.
notes `x': Updated: 04/17/19-MH
} 
foreach x in female race_cat education income_cat ///
otherlang imputed_inc1 imputed_inc2 imputed_inc3 imputed_inc4 imputed_inc5 aveincome{
notes drop `x'
}
notes female: dgender
notes female: Gender variable only asked in wave 1 & 5.
notes race_cat: dracehisp
notes race_cat: Race and Hispanic ethnicity only asked in wave 1 & 5.
notes education: higstschl 
notes education: Highest degree/ level of schooling only asked in wave 1 & 5.
notes educ_hs_ind: higstschl
notes educ_hs_ind: Indicator having HS+
forvalues i=1/5{
notes imputed_inc`i': toincim`i'
notes imputed_inc`i': Imputed income
notes imputed_inc`i': Located in NHATS Setup, under Part 3.
notes imputed_inc`i': Updated: 04/18/19-MH
}
notes aveincome: imputed_inc1-imputed_inc5
notes aveincome: Took mean of imputed income. Not done for wave 2. 
notes income_cat: aveincome imputed_inc1-imputed_inc5
notes income_cat: Make income categories. 
notes otherlang: spkothlan condspanh
notes otherlang: Speak other language than English; Interview being conducted in Spanish. 
notes freq_go_out: outoft
notes freq_go_out: How often went outside.
notes help_go_out: outhlp
notes help_go_out: Anyone help you go outside.
notes out_on_own: outoft outhlp
notes out_on_own: Left home by self.
notes needshelp: help_go_out out_on_own
notes needshelp: Leaves home with help if never leaves by self and has help to leave.
notes hasdifficulty: outdif
notes hasdifficulty: How much difficulty to go outside (even with devices)
notes helpdiff: help_go_out out_on_own outdif
notes helpdiff: Has difficulty or needs help going outside. 
notes independent: outhlp outdif
notes independent: Doesn't have difficulty and no help to leave home.
notes homebound_cat: outoft outoft outhlp help_go_out out_on_own outdif 
notes homebound_cat: homebound categorical variable. 
notes indeplow: outoft outhlp outdif
notes indeplow: Rarely leaves home, but is independent. 
notes indephigh: outoft outhlp outdif
notes indephigh: Goes out often, but is independent. 
notes never_go_out: freq_go_out outoft
notes never_go_out: Never goes out.
notes whiteoth: race_cat
notes whiteoth: White non-hispanic or other, combined category.
notes maritalstat: martlstat dmarstat 
notes maritalstat: Marital status category. 
notes mc_partd: covmedcad
notes mc_partd: Currently have Medicare, Part D  coverage.
notes tricare: covtricar 
notes tricare: Health care for active and retired Armed forces, their families, and survivors. 
notes private_ltc: nginsnurs
notes private_ltc: Have insurance that would pay for more than a year of care in ///
nursing home, assisted living, and in own home.  
notes moved: dadrscorr spadrsnew
notes moved: Address same or different address.
notes residence: resistrct dresistrct 
notes residence: Residence physical structure. 
notes livearrang: lvngarrg dlvngarrg
notes livearrang: Living arrangement.
notes livealone: livearrang dlvngarrg
notes livealone: Lives alone
foreach x in female race_cat education income_cat otherlang aveincome freq_go_out ///
help_go_out out_on_own needshelp hasdifficulty helpdiff independent homebound_cat ///
indeplow indephigh never_go_out whiteoth maritalstat mc_partd tricare private_ltc ///
moved residence livearrang livealone {
notes `x': Located in NHATS Setup, under Part 3.
notes `x': Updated: 04/18/19-MH
}

notes sr_ami_ever: sr_ami_raw disescn1 
notes sr_ami_ever: If ever had a heart attack or myocardial infarction (self-report).
notes sr_stroke_ever: sr_stroke_raw disescn8
notes sr_stroke_ever: If ever had a stroke (self-report).
notes sr_cancer_ever: sr_cancer_raw disescn10
notes sr_cancer_ever: If ever had cancer (self-report). 
notes sr_ami_new: sr_ami_raw disescn1
notes sr_ami_new: If heart attack or myocardial infarction within last year (self-report).
notes sr_stroke_new: sr_stroke_raw disescn8 
notes sr_stroke_new: If stroke within last year (self-report).
notes sr_cancer_new: sr_cancer_raw disescn10
notes sr_cancer_new: If cancer found within last year (self-report).
notes sr_hip_ever: sr_hip_raw brokebon1
notes sr_hip_ever: Had ever broken or fractured hip (self-report).
notes sr_hip_new: sr_hip_raw brokebon1
notes sr_hip_new: Had hip fracture since prev interview (self-report). 
notes sr_heart_dis_ever: sr_heart_dis_raw disescn2
notes sr_heart_dis_ever: Had heart disease ever (self-report)
notes sr_heart_dis_new:  sr_heart_dis_raw disescn2
notes sr_heart_dis_new: Had heart disease since prev interview (self-report). 
notes sr_htn_ever: sr_htn_raw disescn3
notes sr_htn_ever: Had High blood pressure ever (self-report)
notes sr_htn_new:  sr_htn_raw disescn3
notes sr_htn_new: Had High blood pressure since prev interview (self-report). 
notes sr_ra_ever: sr_ra_raw disescn4
notes sr_ra_ever: Had arthritis ever (self-report)
notes sr_ra_new:  sr_ra_raw disescn4
notes sr_ra_new: Had arthritis since prev interview (self-report). 
notes sr_osteoprs_ever: sr_osteoprs_raw disescn5
notes sr_osteoprs_ever: Had osteoporosis ever (self-report)
notes sr_osteoprs_new:  sr_osteoprs_raw disescn5
notes sr_osteoprs_new: Had osteoporosis since prev interview (self-report). 
notes sr_diabetes_ever: sr_diabetes_raw disescn6
notes sr_diabetes_ever: Had diabetes ever (self-report)
notes sr_diabetes_new:  sr_diabetes_raw disescn6
notes sr_diabetes_new: Had diabetes since prev interview (self-report). 
notes sr_lung_dis_ever: sr_lung_dis_raw disescn7
notes sr_lung_dis_ever: Had lung disease ever (self-report)
notes sr_lung_dis_new:  sr_lung_dis_raw disescn7
notes sr_lung_dis_new: Had lung disease since prev interview (self-report). 
notes sr_dementia_ever: sr_dementia_raw disescn9
notes sr_dementia_ever: Had dementia ever (self-report)
notes sr_dementia_new:  sr_dementia_raw disescn9
notes sr_dementia_new: Had dementia since prev interview (self-report). 
foreach x in sr_ami_ever sr_stroke_ever sr_ami_new sr_stroke_new sr_cancer_new ///
sr_cancer_ever sr_hip_ever sr_hip_new sr_heart_dis_ever sr_heart_dis_new ///
sr_htn_ever sr_htn_new sr_ra_ever sr_ra_new sr_osteoprs_ever sr_osteoprs_new ///
sr_diabetes_ever sr_diabetes_new sr_lung_dis_ever sr_lung_dis_new sr_dementia_ever sr_dementia_new{
notes `x': Located in NHATS Setup, under Part 4.
notes `x': Updated: 04/15/19-MH
}
notes lml_ivw_yes: r*spstat r*status
notes lml_ivw_yes: Last wave of life interview. 
notes srh: hc*health
notes srh: Self-reported health. As is.
foreach x in lml_ivw_yes srh{
notes `x': Located in NHATS Setup, under Part 3.
notes `x': Updated: 04/25/19-MH
}
notes prim_helper_opid: Opid
notes prim_helper_opid: Primary helper opid. 
notes prim_helper_cat: op_hrsmth_i max_hrs_helped  opishelper_allw
notes prim_helper_cat: Primary helper is the OP with most hours. 
notes tot_hrsmonth_help_i: op_hrsmth_i 
notes tot_hrsmonth_help_i: imputed total number of helper hours received per month. 
notes n_helpers: op_hrsmth_i 
notes n_helpers: Number of helpers per spid wave. 
notes tot_hrswk_help_i: op_hrsmth_i
notes tot_hrswk_help_i: imputed total number of helper hours received per week.
notes n_family_helpers: op_relationship_cat
notes n_family_helpers: OP is a family helper.
notes ind_family_helper: op_relationship_cat
notes ind_family_helper: Indicate a family helper. 
notes n_paid_helpers: paid
notes n_paid_helpers: Number paid helpers.
notes ind_paid_helper: n_paid_helper
notes ind_paid_helper: Indicate any paid helper.
notes ind_rc_staff: rc_staff
notes ind_rc_staff: Indicate facility staff.
notes num_helpers_cat: n_helpers
notes num_helpers_cat: Categorical number of helpers, 0-2, 3-6, 7+
notes tot_hrsmonth_paid_i: phrs
notes tot_hrsmonth_paid_i:  Imputed help received hours per month, for paid helpers
notes tot_hrswk_paid_i: tot_hrsmonth_paid_i
notes tot_hrswk_paid_i: Imputed help received hours per week, for paid helpers
notes ind_no_helpers: n_helpers
notes ind_no_helpers: No reported helpers.
foreach x in prim_helper_opid prim_helper_cat tot_hrsmonth_help_i n_helpers tot_hrswk_help_i ///
n_family_helpers ind_family_helper n_paid_helpers ind_paid_helper ind_rc_staff num_helpers_cat ///
tot_hrsmonth_paid_i tot_hrswk_paid_i ind_no_helpers{
notes `x': Located in NHATS Setup, under Part 6.
notes `x': Updated: 04/25/19-MH
}
notes sr_depr_pqq1: depresan1
notes sr_depr_pqq1: Little interest or pleasure doing things.
notes sr_depr_pqq2: depresan2
notes sr_depr_pqq2: Felt down, depressed, hopeless.
notes sr_anx_gad1: depresan3
notes sr_anx_gad1: Felt nervous, anxious, on edge.
notes sr_anx_gad2: depresan4
notes sr_anx_gad2: Been unable to stop or control worrying. 
notes sr_phq2_score: depresan1 depresan2 made into sr_depr_pqq1 sr_depr_pqq2
notes sr_phq2_score: Adding sr_depr_pqq1 and sr_depr_pqq2 to get a score. Had little ///
interest doing things; felt hopeless, down, or depressed.
notes sr_phq2_depressed: sr_phq2_score
notes sr_phq2_depressed: If score is greater 3 or more, then classified as depressed per PHQ-2.
notes sr_gad2_score: sr_anx_gad1 sr_anx_gad2
notes sr_gad2_score: Adding sr_anx_gad1 and sr_anx_gad2 to get a score. Felt nervous, anxious, or on edge; ///
been unable to stop or control worrying. 
notes sr_gad2_anxiety: sr_gad2_score
notes sr_gad2_anxiety: If score is greater 3 or more, then classified as having anxiety per GAD-2.
notes dem_3_cat: disescn9 dem_via_proxy speaktosp
notes dem_3_cat: Indicates if SP reported dementia, from either SP themselves or through proxy.
notes dem_2_cat: disescn9 dem_via_proxy speaktosp
notes dem_2_cat: Indicates if SP reported dementia. Possible/ probable dementia status. 
notes memory: ratememry memrygood 
notes memory: Rate your memory. 
foreach x in sr_phq2_score sr_phq2_depressed sr_gad2_score sr_gad2_anxiety dem_3_cat memory{
notes `x': Located in NHATS Setup, under Part 4.
notes `x': Updated: 04/29/19-MH
}

notes spid: spid
notes spid: Sample person identifier
notes ntlvrmslp: ntlvrmslp
notes ntlvrmslp: Did not leave bedroom flag
notes eatdev: eatdev
notes eatdev: Use eating adapted eating utensils. 
notes dresslf: dresslf
notes dresslf: How often get dressed by self with no help.
notes dresdif: dresdif
notes dresdif: Use special items to help you get dressed. 
notes yearsample: "_"
notes yearsample: Year SP entered sample. 
forvalues w=1/$w{
notes r`w'status: r`w'status
notes r`w'status: status from traker file
}
notes trfinwgt: trfinwgt
notes trfinwgt: Tracker weight
notes tr2011wgt: tr2011wgt
notes tr2011wgt: 2011 Tracker weight
notes sp_status: nhres res_care deceased
notes sp_status: Status of the SP in that wave. 
notes ivw_day: spstatdtdy
notes ivw_day: From restricted tracker day of interview
notes ivw_month: spstatdtmt fqstatdtmt 
notes ivw_month: From restricted tracker month of interview
notes ivw_year: spstatdtyr 
notes ivw_year: From restricted tracker year of interview
notes death_date: dd_claims dd_nhats
notes death_date: from claims or tracker
notes death_month: dd_claims dd_nhats
notes death_month: from claims or tracker
notes death_year: dd_claims dd_nhats
notes death_year: from claims or tracker
notes death_source: dd_claims dd_nhats
notes death_source: Death date source from claims, nhats, both. 
notes nhats_death_day: pd*mthdied pd*yrdied
notes nhats_death_day: NHATS death date
notes nhats_death_month: pd*mthdied
notes nhats_death_month: NHATS death month
notes nhats_death_year: pd*yrdied
notes nhats_death_year: NHATS death year
notes tot_spousehrs: op_hrswk_i 
notes tot_spousehrs: Hours of help from spouse sources
notes tot_daughterhrs: op_hrswk_i 
notes tot_daughterhrs: Hours of help from daughter sources
notes tot_sonhrs: op_hrswk_i 
notes tot_sonhrs: Hours of help from son sources
notes tot_paidhrs: op_hrswk_i 
notes tot_paidhrs: Hours of help from paid sources
foreach x in sp_status nhats_death_day nhats_death_month nhats_death_year ///
death_source death_year death_month death_date ivw_year ivw_month ivw_day ///
tot_spousehrs tot_daughterhrs tot_sonhrs tot_paidhrs{
notes `x': Located in NHATS Setup, under Part 7.
notes `x': Updated: 05/03/19-MH
}
notes ivw_type: r*spstat r*status
notes ivw_type: Determining if SP or Proxy completed interview. 
notes sp_ivw_yes: r*spstat
notes sp_ivw_yes: If SP completed interview

keep spid old_anfinwgt varstrat varunit wave yearsample r8status r7status r6status r5status r4status r3status r2status r1status ivw_type sp_ivw_yes lml_ivw_yes proxy_ivw freq_go_out ///
help_go_out out_on_own needshelp hasdifficulty helpdiff independent homebound_cat never_go_out indeplow indephigh age female race_cat education educ_hs_ind imputed_inc1 ///
imputed_inc2 imputed_inc3 imputed_inc4 imputed_inc5 aveincome income_cat otherlang white black hisp other_race whiteoth maritalstat medicaid medigap mc_partd tricare ///
private_ltc residence livearrang livealone srh srh_fp sr_ami_ever sr_ami_new sr_stroke_ever sr_stroke_new sr_cancer_ever sr_cancer_new sr_hip_ever sr_hip_new ///
sr_heart_dis_ever sr_heart_dis_new sr_htn_ever sr_htn_new sr_ra_ever sr_ra_new sr_osteoprs_ever sr_osteoprs_new sr_diabetes_ever sr_diabetes_new sr_lung_dis_ever ///
sr_lung_dis_new sr_dementia_ever sr_dementia_new sr_depr_pqq1 sr_depr_pqq2 sr_anx_gad1 sr_anx_gad2 sr_phq2_score sr_phq2_depressed sr_gad2_score sr_gad2_anxiety ///
dem_3_cat dem_2_cat memory sr_numconditions1 fall_last_month adl_eat_diff adl_eat_help adl_eat_not_done adl_bath_diff adl_bath_help adl_bath_not_done adl_toil_diff ///
adl_toil_help adl_toil_not_done adl_dres_diff adl_dres_help adl_dres_not_done iadl_laun_help iadl_laun_diff iadl_laun_not_done iadl_shop_help iadl_shop_diff ///
iadl_shop_not_done iadl_meal_help iadl_meal_diff iadl_meal_not_done iadl_bank_help iadl_bank_diff iadl_bank_not_done iadl_meds_help iadl_meds_diff iadl_meds_not_done ///
 ind_meds_diff ind_meds_mistake ind_meds_problem reg_doc_have reg_doc_seen reg_doc_homevisit sr_hosp_ind sr_hosp_stays prim_helper_opid prim_helper_cat ///
 tot_hrsmonth_help_i n_helpers tot_hrswk_help_i n_family_helpers ind_family_helper n_paid_helpers ind_paid_helper ind_rc_staff num_helpers_cat tot_hrsmonth_paid_i ///
 tot_hrswk_paid_i ind_no_helpers metro_ind no_acp formal_acp informal_acp acp old_an2011wgt trfinwgt tr2011wgt serarmfor dresslf dresdif ntlvrmslp eatdev income_quart ///
 placedied hospice1 hospice2 hospicelml gripsc1 walksec1 walkhndr1 gripsc2 walksec2 walkhndr2 origwksc nhatswksc nhatsgrav nhatsgrb lst10pnds trytolose evrgowalk vigoractv ///
 unwgtloss lowphysact lowenergy oftcheer oftbored oftfol oftupset lifemean confident giveup livingsit selfdeter findway adjchg agefeel mealskip nopayhous nopayutil ///
 nopaymed mealskipnum sr_skincancer_had sr_breastcancer_had sr_prostatecancer_had sr_bladdercancer_had sr_crvovrnutrncancer_had sr_coloncancer_had sr_kidneycancer_had ///
 sr_othercancer_had howgetdoc ind_imp_visit ind_imp_relig ind_imp_club ind_imp_goout device_out device_insd device_bed device_eat device_bath device_toil device_dres ///
 device_any drives avoids ind_transp_disadv get_places1 get_places2 get_places3 get_places4 get_places5 get_places6 get_places7 married marital_sep marital_div ///
 marital_wid marital_nev marriedpartnered marital_sd resspouse hhm agecat attrelig imprelig visitfrfam impvisit attclub impclub vigact religvimp smokedever ///
 smoke_start_age smoke_stop_age smokesnow numcigsever numcigsnow ind_pain wgt_curr height_ft height_in bmi moved community_dwelling nhres rcfres res_care sp_status ///
 prob_dem adl_ins_help adl_ins_diff adl_ins_not_done adl_bed_help adl_bed_diff adl_bed_not_done ivw_day ivw_month ivw_year adl_index adl_independent adl_impair ///
 iadl_index iadl_independent iadl_impair adl_diff_index adl_diff_ind iadl_diff_index iadl_diff_ind ivw_date meals_wheels noone_talk restrnt_takeout nhats_death_day ///
 nhats_death_date nhats_death_month nhats_death_year eligible_nsoc completed_nsoc total_caregiver_comp total_caregiver_elig adverse adverse_laun adverse_shop ///
 adverse_meal adverse_bank adverse_eat adverse_bath adverse_toil adverse_dres adverse_out adverse_insd adverse_bed lim_fav lim_fav_yr cohesion_knowwell cohesion_willing ///
 cohesion_peop cohesion home_disorder_insd home_disorder_outsd home_disorder_area home_disorder_clutter creditdebt medcreditdebt medbillsovertime medpaynotcash finhlpfam ///
 fingftfam govtasst ownhome rent mortgagepaidoff paysmortgage mortpaymt timeinmort owedonmort homevalue rentpaid section8 n_social_network division region northeast ///
 midwest south west n_children stairs_ind livesalone_ind ind_kids eat_jenny toil_jenny insd_jenny bed_jenny dres_jenny bath_jenny adlcount_jenny laun_jenny meds_jenny ///
 shop_jenny meal_jenny bank_jenny iadlcount_jenny device_mobility spouse_help_ind tot_spousehrs daughter_help_ind tot_daughterhrs son_help_ind tot_sonhrs otherfamily_help_ind ///
 tot_otherfamilyhrs paid_help_ind tot_paidhrs otherinformal_help_ind tot_otherinformalhrs any_paid_by_sp any_paid_by_gov any_paid_by_ins any_paid_by_unk any_paid_by_oth ///
 any_paid_by_gov_cat_1 any_paid_by_gov_cat_2 any_paid_by_gov_cat_3 any_paid_by_gov_cat_oth any_paid_by_gov_cat_unknown any_paid_by_medicare any_paid_by_medicaid any_paid_by_state ///
 ind_gt20hrs_wk_inf ind_gt20hrs_wk_paid anfinwgt an2011wgt death_* died_12 died_24 *_lml qoc_lml_e adl_out* out_jenny soc_iso frail rehab surgery sev_dem relat*proxy* familiar*
 

saveold "D:\NHATS\Shared\base_data\NHATS cleaned\sp_round_1_${w}", replace version(12)

gen time2d=mofd(nhats_death_date)-mofd(ivw_date)

foreach x in 12 24 {
	gen nhats_died_`x'=time2d<=`x' if !lml
	label var nhats_died_`x' "Died w/in `x' months (from NHATS)"

}

capture log close

log using "D:\NHATS\Shared\base_data\NHATS cleaned\Codebooks\sp_round_1_${w}_variable_list.txt", text replace
describe , fullnames
log close
 
drop death_* died_12 died_24 ivw_day ivw_date

drop time2d 

saveold "D:\NHATS\Shared\base_data\NHATS cleaned\sp_round_1_${w}_public_sens_only.dta", replace version(12)


log using "D:\NHATS\Shared\base_data\NHATS cleaned\Codebooks\sp_round_1_${w}_pub_sens_only_variable_list.txt", text replace
describe , fullnames
log close

local senslabs
local i=1
foreach x of varlist * {
di `i'
local lab : var label `x'
tokenize "`lab'"
if "`1'"=="Sensitive:" {
local senslabs `senslabs' `x'
local i=`i'+1
}
}

log using "D:\NHATS\Shared\base_data\NHATS cleaned\Codebooks\sp_round_1_${w}_sensitive_variables.txt", text replace
describe `senslabs', fullnames
log close


translate "${logs}/`name'.smcl" "${logs}/`name'.pdf"
exit
```


