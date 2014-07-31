marketing campaigns

proc surveyselect
==========

step1 %var_check code
/* create dataset for campaign 1 same number of observaions (n=2000) or count as campaing 2 and same respone rate */
proc surveyselect data=campaign1 (where (DV=1) method=srs n=2000
seed = 12345 out= dv1_campaign1; 
run;

proc surveyselect data=campaign1 (where (DV=0) method n=646380
seed = 12345 out=dv0_campaing1;
run;

data campaign1;
   set dv1_campaign1
   set dv0_campaign1;
run;


STEP 2 – PROC LOGISTIC and ODS Output

%macro var_check(var)

/* simple logisitic on campaign 1 */
proc logistic data=campaign1 desc;
model DV=&var / STB PARMLABEL;

/*ODS ouptut for campaign 1 */
ods output parameterestimates=est_campaign1 (keep = Variable Estimate StdErr  /*keeping 3 variables only*/
                                            rename = (Estimate=Est_Campaign1 StdErr=StdErr_campaign1));
run;


/* simple logisitic on campaign 2 */                                       
proc logistic data=campaign2 desc;
model DV=&var / STB PARMLABEL;

/*ODS ouptut for campaign 2 */
ods output parameterestimates=est_campaign2 (keep = Variable Estimate StdErr  /*keeping 3 variables only*/
                                            rename = (Estimate=Est_Campaign2 StdErr=StdErr_campaign2));
run;


STEP 3 – Merge the two ODS output datasets and create one dataset for each independent variable using Proc SQL

data &var._campaing1;
   set est_campaing1;
   if Variable = 'Intercept' then delete;
run;

data &var._campaing2;
   set est_campaing2;
   if Variable = 'Intercept' then delete;
run;


proc sql;
   create table &var._cl_2 as
   select a.variable as Variable length = 15, a.Est_campaign1, b.Est_campaign2, a.StdErr_campaign1,   b.StdErr_campaign2&var
   from &var._campaign1 as a,
        &var._campaing2 as b
  where a.Variable = b.Variable;
run;


/* 2-tailed t-test on campaign 1 & 2 coefficients */

data &var._cl_2;
 set &var._cl_2;
 
 z = (Abs(Est_campaign2 - Est_campaign1))/Sqrt(StdErr_campaign1**2 + StdErr_campaign2**2);
 pvalue = 2 * (1 - cdf('normal', Z, 0, 1)); /* 2-tailed test */
 if 0 < Pvalue < 0.1 then Sig_Flag_at_90=1; else if Pvalue="" then Sig_Flag_at_90=.; else Sig_Flag_at_90=0;
 if 0 < Pvalue < 0.05 then Sig_Flag_at_95=1; else if Pvalue="" then Sig_Flag_at_95=.; else Sig_Flag_at_95=0;
 if 0 < Pvalue < 0.01 then Sig_Flag_at_99=1; else if Pvalue="" then Sig_Flag_at_99=.; else Sig_Flag_at_99=0;
run;

%mend;

proc contents data=campaing1 out=campaign1_contents norpint; run;

DATA _NULL_;
   SET campaign1_contents(where = (name not in
   ('/* Space to add not needed in the analysis IDs, Keys, etc.')));
   Var_Check = '%Var_Check(' !! TRIM(LEFT(name))  !!  ');';
   CALL EXECUTE(Var_Check);
run;

proc sql no print;
   select cats(name,"_cl_2")
          into :set_datasets 
          separated by ' '
          from campaign1_contents(where (name not in 
          ('/* Space to add not needed in the analysis IDs, Keys, etc.')));
quit;


data dt.var_check_by_cmpn;
   set &set_datasets.;
run;

ods html file = "var_check_by_cmpn.xls" style = minimal;
proc print data = var_check_by_cmpn; 
run;

ods html close;













   
   
   
   
   
   
   
   
   























   
   
   
   
   
   
   
   
   
   
   
   
   














