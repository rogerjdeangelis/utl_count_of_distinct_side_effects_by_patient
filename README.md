# utl_count_of_distinct_side_effects_by_patient
Count of distinct side effects by patient. 
    Count of distinct side effects by patient

    github
    https://github.com/rogerjdeangelis/utl_count_of_distinct_side_effects_by_patient

      Four Solutions
         1. Proc sort then proc freq
         2. Hash
         3. Nested SQL
         4. R

    Hash was built on KSharp technique
    https://communities.sas.com/t5/user/viewprofilepage/user-id/18408

    INPUT
    =====

     ALGORITHM

       Only count distinct side effects by patient
       If a patient hab the same side effect twice cout it as one

     WORK.HAVE total obs=15  |  RULES
                             |
      PATIENT    SIDEEFFECT  |  Consider Patient 2
                             |
         2       DRY MOUTH   |   2       DRY MOUTH
         2       COUGH       |   2       COUGH
         1       RASH        |   2       COUGH     |  Only count once
         1       RASH        |   2       RASH
         2       COUGH       |   2       FEVER
         2       RASH        |
         3       FEVER       |   So
         1       COUGH       |
         4       FEVER       |   Patient 2 has 4 distinct side effects
         3       COUGH       |
         2       FEVER       |
         3       RASH        |
         4       COUGH       |
         3       DRY MOUTH   |
         4       COUGH       |

      EXAMPLE OUTPUT

         PATIENT    Frequency
         --------------------
            1           2
            2           4
            3           4
            4           2

    PROCESS
    =======

      1. Proc sort then proc freq

         proc sort data=have out=havUnq nodupkey noequals;
           by patient sideEffect;
         run;quit;

         proc freq data=havUnq;
           tables patient/list out=wantfrq;
         run;quit;

      2. Hash

         data wanthsh;

           if 0 then set have;

           declare hash ha(dataset:'have',ordered:'y');  * load ordered;
            declare hiter htr('ha');
            ha.definekey('patient','sideEffect');
            ha.definedata('patient','sideEffect');
            ha.definedone();

            do while(htr.next()=0);
               n+1;
               lagpatient=lag(patient);
               if (patient ne lagpatient) and not missing(lagpatient) then do;
                  patient=lagpatient;
                  n=n-1;
                  output;
                  n=1;
               end;
         end;
         output;
         keep patient n;
         run;

      3. Nested SQL

         proc sql;
          create
            table wantSql as
          select
            patient
           ,count(patient) as N
          from
            (select distinct patient, sideEffect from have)
          group
            by patient
         ;quit;

       4. R

          %utl_submit_wps64('
          options set=R_HOME "C:/Program Files/R/R-3.3.1";
          libname wrk "%sysfunc(pathname(work))";
          options validvarname=upcase;
          libname sd1 "d:/sd1";
          data sd1.have;
            set wrk.have;
          run;quit;
          proc r;
          submit;
          library(haven);
          have<-read_sas("d:/sd1/have.sas7bdat");
          library(dplyr);
          want <- have   %>%
              distinct() %>%
              count(PATIENT);
          endsubmit;
          import r=want data=wrk.wantwps;
          run;quit;
          ');


    OUTPUT  (from proc freq)
    ======

     WORK.WANTFRQ total obs=4

       PATIENT    COUNT    PERCENT

          1         2      16.6667
          2         4      33.3333
          3         4      33.3333
          4         2      16.6667

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    data have;
     input patient 1. @3 sideEffect $11.;
    cards4;
    2 dry mouth
    2 cough
    1 rash
    1 rash
    2 cough
    2 rash
    3 fever
    1 cough
    4 fever
    3 cough
    2 fever
    3 rash
    4 cough
    3 dry mouth
    4 cough
    ;;;;
    run;quit;


    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __  ___
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/

    ;

    see process

