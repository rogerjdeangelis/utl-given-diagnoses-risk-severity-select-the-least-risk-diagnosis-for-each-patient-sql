# utl-given-diagnoses-risk-severity-select-the-least-risk-diagnosis-for-each-patient-sql
Given diagnoses and risk severity select the least risk diagnosis for each patient sql
    %let pgm=utl-given-diagnoses-risk-severity-select-the-least-risk-diagnosis-for-each-patient-sql;

    %stop_submission;

    Given diagnoses and risk severity select the least risk diagnosis for each patient sql

      CONTENTS
         1 sas sql
           use sql arrays to generalize to to mare diagnoses
           may not be as slow as you thing
         2 sas sql with arrays
           if you highlight the code and
           type debug on the command line the generated
           conde will be in the log, then just cut and paste
         3 r sql (same code in python and excel(
           see
           https://tinyurl.com/4e6yaap8
         4 r related repos

    github
    https://tinyurl.com/r843uvnn
    https://github.com/rogerjdeangelis/utl-given-diagnoses-risk-severity-select-the-least-risk-diagnosis-for-each-patient-sql

    communities.sas
    https://tinyurl.com/4vx6fuuk
    https://communities.sas.com/t5/SAS-Programming/Select-value-from-a-list-based-on-ordered-preference/m-p/807580#M318423

    /**************************************************************************************************************************/
    /*        INPUT                       |        PROCESS                                        |    OUTPUT                 */
    /*                                    |                                                       |                           */
    /* SD1.ORDR total obs=5               | 1 SAS SQL                                             |                           */
    /*                                    | =========                        OUTPUT               |           LEAST_          */
    /*  DIAG        ORDR                  |                                | LEAST                | CASEID    RISK            */
    /*                                    | CASEID  DX1     DX2      DX3   | SEVERE               |                           */
    /*  CANCER        5  most severe      |                                |                      |   112     COPD            */
    /*  MI            4                   | 112   OTHER     COPD           | COPD (OTHER NOT USED)|   123     OBESITY         */
    /*  DIABETES      3                   | 123   CANCER    OBESITY        | OBESITY              |   456     OBESITY         */
    /*  COPD          2                   | 456   COPD      OBESITY        | OBESITY              |   789     COPD            */
    /*  OBESITY       1  least severe     | 789   DIABETES  MI       COPD  | DIABETES             |                           */
    /*                                    |                                                       |                           */
    /*                                    |                                                       |                           */
    /* SD1.HAVE total obs=4               | STEPS                                                 |                           */
    /*                                    |   1 pivot sd1.have long                               |                           */
    /*  CASEID  DX1     DX2      DX3      |   2 join sd1.have with sd1.order                      |                           */
    /*                                    |     on diag and select min(ordr)                      |                           */
    /*  112   OTHER     COPD              |                                                       |                           */
    /*  123   CANCER    OBESITY           |-----------------------------------------------------------------------------------*/
    /*  456   COPD      OBESITY           |   1 SAS SQL                                           |            LEAST_         */
    /*  789   DIABETES  MI       COPD     |   =========                                           |  CASEID    RISK           */
    /*                                    |                                                       |                           */
    /* options validvarname=upcase;       |   proc datasets lib=sd1                               |    112     COPD           */
    /* libname sd1 "d:/sd1";              |     nolist nodetails;                                 |    123     OBESITY        */
    /* data sd1.have;                     |    delete want havlon;                                |    456     OBESITY        */
    /* input caseId (Dx1 - Dx3) ($) ;     |   run;quit;                                           |    789     COPD           */
    /* cards4;                            |                                                       |                           */
    /* 123 CANCER OBESITY .               |   proc sql;                                           |                           */
    /* 456 COPD OBESITY .                 |     create                                            |                           */
    /* 789 DIABETES MI COPD               |       view havLon as                                  |                           */
    /* 112 OTHER COPD .                   |     select                                            |                           */
    /* ;;;;                               |       caseid                                          |                           */
    /* run;quit;                          |      ,dx1 as diag                                     |                           */
    /*                                    |     from                                              |                           */
    /* data sd1.ordr;                     |       sd1.have                                        |                           */
    /* input diag $ ordr;                 |     union                                             |                           */
    /* cards4;                            |       all                                             |                           */
    /* CANCER 5                           |     select                                            |                           */
    /* MI 4                               |       caseid                                          |                           */
    /* DIABETES 3                         |      ,dx2 as diag                                     |                           */
    /* COPD 2                             |     from                                              |                           */
    /* OBESITY 1                          |       sd1.have                                        |                           */
    /* ;;;;                               |     union                                             |                           */
    /* run;quit;                          |       all                                             |                           */
    /*                                    |     select                                            |                           */
    /*                                    |       caseid                                          |                           */
    /*                                    |      ,dx3 as diag                                     |                           */
    /*                                    |     from                                              |                           */
    /*                                    |       sd1.have                                        |                           */
    /*                                    |     order                                             |                           */
    /*                                    |       by caseid                                       |                           */
    /*                                    |   ;                                                   |                           */
    /*                                    |     create                                            |                           */
    /*                                    |        table want as                                  |                           */
    /*                                    |     select                                            |                           */
    /*                                    |        h.caseid                                       |                           */
    /*                                    |       ,h.diag as least_risk                           |                           */
    /*                                    |     from                                              |                           */
    /*                                    |       havlon as h left join sd1.ordr as o             |                           */
    /*                                    |     on                                                |                           */
    /*                                    |       h.diag = o.diag                                 |                           */
    /*                                    |     where                                             |                           */
    /*                                    |       ordr ne . and not missing(h.diag)               |                           */
    /*                                    |     group                                             |                           */
    /*                                    |       by caseid                                       |                           */
    /*                                    |     having                                            |                           */
    /*                                    |       ordr = min(ordr)                                |                           */
    /*                                    |     order                                             |                           */
    /*                                    |       by h.caseid, o.ordr desc                        |                           */
    /*                                    |   ;quit;                                              |                           */
    /*                                    |                                                       |                           */
    /*                                    |                                                       |                           */
    /*                                    |-----------------------------------------------------------------------------------*/
    /*                                    |   2 SAS SQL WITH ARRAYS                               |            LEAST_         */
    /*                                    |   =====================                               |  CASEID    RISK           */
    /*                                    |                                                       |                           */
    /*                                    |                                                       |    112     COPD           */
    /*                                    |   proc datasets lib=sd1                               |    123     OBESITY        */
    /*                                    |     nolist nodetails;                                 |    456     OBESITY        */
    /*                                    |    delete want havlon;                                |    789     COPD           */
    /*                                    |   run;quit;                                           |                           */
    /*                                    |                                                       |                           */
    /*                                    |   %array(_us,values=1-3);                             |                           */
    /*                                    |                                                       |                           */
    /*                                    |   proc sql;                                           |                           */
    /*                                    |     create                                            |                           */
    /*                                    |       view havlon as                                  |                           */
    /*                                    |     %do_over(_us,phrase=%str(                         |                           */
    /*                                    |         select                                        |                           */
    /*                                    |           caseid                                      |                           */
    /*                                    |          ,dx? as diag                                 |                           */
    /*                                    |         from                                          |                           */
    /*                                    |           sd1.have),between=union all)                |                           */
    /*                                    |     ;                                                 |                           */
    /*                                    |     create                                            |                           */
    /*                                    |        table want as                                  |                           */
    /*                                    |     select                                            |                           */
    /*                                    |        h.caseid                                       |                           */
    /*                                    |       ,h.diag as least_risk                           |                           */
    /*                                    |     from                                              |                           */
    /*                                    |       havlon as h left join sd1.ordr as o             |                           */
    /*                                    |     on                                                |                           */
    /*                                    |       h.diag = o.diag                                 |                           */
    /*                                    |     where                                             |                           */
    /*                                    |       ordr ne . and not missing(h.diag)               |                           */
    /*                                    |     group                                             |                           */
    /*                                    |       by caseid                                       |                           */
    /*                                    |     having                                            |                           */
    /*                                    |       ordr = min(ordr)                                |                           */
    /*                                    |     order                                             |                           */
    /*                                    |       by h.caseid, o.ordr desc                        |                           */
    /*                                    |   ;quit;                                              |                           */
    /*                                    |                                                       |                           */
    /*                                    |   %arraydelete(_us);                                  |                           */
    /*                                    |                                                       |                           */
    /*                                    |  code generated log(use debug to see)                 |                           */
    /*                                    |                                                       |                           */
    /*                                    |  select caseid ,1 as iddx,dx1 as diag                 |                           */
    /*                                    |     from sd1.have union all                           |                           */
    /*                                    |  select caseid ,2 as iddx,dx2 as diag                 |                           */
    /*                                    |     from sd1.have union all                           |                           */
    /*                                    |  select caseid ,3 as iddx,dx3 as diag                 |                           */
    /*                                    |    from sd1.have                                      |                           */
    /*                                    |                                                       |                           */
    /*                                    |-----------------------------------------------------------------------------------*/
    /*                                    |  3 R SQL (SAME CODE AS SAS)                           | R                         */
    /*                                    |  ===========================                          |   caseid least_risk       */
    /*                                    |                                                       | 1    112       COPD       */
    /*                                    |  proc datasets lib=sd1 nolist nodetails;              | 2    123    OBESITY       */
    /*                                    |   delete want;                                        | 3    456    OBESITY       */
    /*                                    |  run;quit;                                            | 4    789       COPD       */
    /*                                    |                                                       |                           */
    /*                                    |  %utl_rbeginx;                                        | SAS                       */
    /*                                    |  parmcards4;                                          |           LEAST_          */
    /*                                    |  library(haven)                                       | CASEID    RISK            */
    /*                                    |  library(sqldf)                                       |                           */
    /*                                    |  source("c:/oto/fn_tosas9x.R")                        |   112     COPD            */
    /*                                    |  options(sqldf.dll = "d:/dll/sqlean.dll")             |   123     OBESITY         */
    /*                                    |  have<-read_sas("d:/sd1/have.sas7bdat")               |   456     OBESITY         */
    /*                                    |  ordr<-read_sas("d:/sd1/ordr.sas7bdat")               |   789     COPD            */
    /*                                    |  print(have)                                          |                           */
    /*                                    |  want<-sqldf('                                        |                           */
    /*                                    |  with                                                 |                           */
    /*                                    |     havlon as (                                       |                           */
    /*                                    |  select caseid ,1 as iddx,dx1 as diag                 |                           */
    /*                                    |     from have union all                               |                           */
    /*                                    |  select caseid ,2 as iddx,dx2 as diag                 |                           */
    /*                                    |     from have union all                               |                           */
    /*                                    |  select caseid ,3 as iddx,dx3 as diag                 |                           */
    /*                                    |    from have)                                         |                           */
    /*                                    |  select                                               |                           */
    /*                                    |     h.caseid                                          |                           */
    /*                                    |    ,h.diag as least_risk                              |                           */
    /*                                    |  from                                                 |                           */
    /*                                    |    havlon as h left join ordr as o                    |                           */
    /*                                    |  on                                                   |                           */
    /*                                    |    h.diag = o.diag                                    |                           */
    /*                                    |  where                                                |                           */
    /*                                    |    ordr is not null  and                              |                           */
    /*                                    |    h.diag is not null                                 |                           */
    /*                                    |  group                                                |                           */
    /*                                    |    by caseid                                          |                           */
    /*                                    |  having                                               |                           */
    /*                                    |    ordr = min(ordr)                                   |                           */
    /*                                    |  order                                                |                           */
    /*                                    |    by h.caseid, o.ordr desc                           |                           */
    /*                                    |  ')                                                   |                           */
    /*                                    |  want                                                 |                           */
    /*                                    |  fn_tosas9x(                                          |                           */
    /*                                    |        inp    = want                                  |                           */
    /*                                    |       ,outlib ="d:/sd1/"                              |                           */
    /*                                    |       ,outdsn ="want"                                 |                           */
    /*                                    |       )                                               |                           */
    /*                                    |  ;;;;                                                 |                           */
    /*                                    |  %utl_rendx;                                          |                           */
    /*                                    |                                                       |                           */
    /*                                    |  proc print data=sd1.want;                            |                           */
    /*                                    |  run;quit;                                            |                           */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    input caseId (Dx1 - Dx3) ($) ;
    cards4;
    123 CANCER OBESITY .
    456 COPD OBESITY .
    789 DIABETES MI COPD
    112 OTHER COPD .
    ;;;;
    run;quit;

    data sd1.ordr;
    input diag $ ordr;
    cards4;
    CANCER 5
    MI 4
    DIABETES 3
    COPD 2
    OBESITY 1
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /* SD1.ORDR total obs=5               |  SD1.HAVE total obs=4                                                             */
    /*                                    |                                                                                   */
    /*  DIAG        ORDR                  |   CASEID  DX1     DX2      DX3                                                    */
    /*                                    |                                                                                   */
    /*  CANCER        5  most severe      |   112   OTHER     COPD                                                            */
    /*  MI            4                   |   123   CANCER    OBESITY                                                         */
    /*  DIABETES      3                   |   456   COPD      OBESITY                                                         */
    /*  COPD          2                   |   789   DIABETES  MI       COPD                                                   */
    /*  OBESITY       1  least severe     |                                                                                   */
    /**************************************************************************************************************************/

    /*                             _
    / |  ___  __ _ ___   ___  __ _| |
    | | / __|/ _` / __| / __|/ _` | |
    | | \__ \ (_| \__ \ \__ \ (_| | |
    |_| |___/\__,_|___/ |___/\__, |_|
                                |_|
    */

    proc datasets lib=sd1
      nolist nodetails;
     delete want havlon;
    run;quit;

    proc sql;
      create
        view havLon as
      select
        caseid
       ,dx1 as diag
      from
        sd1.have
      union
        all
      select
        caseid
       ,dx2 as diag
      from
        sd1.have
      union
        all
      select
        caseid
       ,dx3 as diag
      from
        sd1.have
      order
        by caseid
    ;
      create
         table want as
      select
         h.caseid
        ,h.diag as least_risk
      from
        havlon as h left join sd1.ordr as o
      on
        h.diag = o.diag
      where
        ordr ne . and not missing(h.diag)
      group
        by caseid
      having
        ordr = min(ordr)
      order
        by h.caseid, o.ordr desc
    ;quit;

     proc print data=want;
     run;quit;

    /**************************************************************************************************************************/
    /*            LEAST_                                                                                                      */
    /*  CASEID    RISK                                                                                                        */
    /*                                                                                                                        */
    /*    112     COPD                                                                                                        */
    /*    123     OBESITY                                                                                                     */
    /*    456     OBESITY                                                                                                     */
    /*    789     COPD                                                                                                        */
    /**************************************************************************************************************************/

    /*___                              _
    |___ \   ___  __ _ ___   ___  __ _| |   __ _ _ __ _ __ __ _ _   _ ___
      __) | / __|/ _` / __| / __|/ _` | |  / _` | `__| `__/ _` | | | / __|
     / __/  \__ \ (_| \__ \ \__ \ (_| | | | (_| | |  | | | (_| | |_| \__ \
    |_____| |___/\__,_|___/ |___/\__, |_|  \__,_|_|  |_|  \__,_|\__, |___/
                                    |_|                         |___/
    */

     proc datasets lib=sd1
       nolist nodetails;
      delete want havlon;
     run;quit;

     %array(_us,values=1-3);

     proc sql;
       create
         view havlon as
       %do_over(_us,phrase=%str(
           select
             caseid
            ,dx? as diag
           from
             sd1.have),between=union all)
       ;
       create
          table want as
       select
          h.caseid
         ,h.diag as least_risk
       from
         havlon as h left join sd1.ordr as o
       on
         h.diag = o.diag
       where
         ordr ne . and not missing(h.diag)
       group
         by caseid
       having
         ordr = min(ordr)
       order
         by h.caseid, o.ordr desc
     ;quit;

     %arraydelete(_us);

     proc print data=want;
     run;quit;

    /**************************************************************************************************************************/
    /*            LEAST_                                                                                                      */
    /*  CASEID    RISK                                                                                                        */
    /*                                                                                                                        */
    /*    112     COPD                                                                                                        */
    /*    123     OBESITY                                                                                                     */
    /*    456     OBESITY                                                                                                     */
    /*    789     COPD                                                                                                        */
    /**************************************************************************************************************************/

    /*____                    _
    |___ /   _ __   ___  __ _| |
      |_ \  | `__| / __|/ _` | |
     ___) | | |    \__ \ (_| | |
    |____/  |_|    |___/\__, |_|
                           |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete want;
    run;quit;

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    options(sqldf.dll = "d:/dll/sqlean.dll")
    have<-read_sas("d:/sd1/have.sas7bdat")
    ordr<-read_sas("d:/sd1/ordr.sas7bdat")
    print(have)
    want<-sqldf('
    with
       havlon as (
    select caseid ,1 as iddx,dx1 as diag
       from have union all
    select caseid ,2 as iddx,dx2 as diag
       from have union all
    select caseid ,3 as iddx,dx3 as diag
      from have)
    select
       h.caseid
      ,h.diag as least_risk
    from
      havlon as h left join ordr as o
    on
      h.diag = o.diag
    where
      ordr is not null  and
      h.diag is not null
    group
      by caseid
    having
      ordr = min(ordr)
    order
      by h.caseid, o.ordr desc
    ')
    want
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /*  R                  |            LEAST_                                                                                */
    /*  caseid least_risk  |  CASEID    RISK                                                                                  */
    /*                     |                                                                                                  */
    /*     112       COPD  |    112     COPD                                                                                  */
    /*     123    OBESITY  |    123     OBESITY                                                                               */
    /*     456    OBESITY  |    456     OBESITY                                                                               */
    /*     789       COPD  |    789     COPD                                                                                  */
    /**************************************************************************************************************************/

    /*  _              _       _           _
    | || |    _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
    | || |_  | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
    |__   _| | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
       |_|   |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                                       |_|
    */

    https://github.com/rogerjdeangelis/utl-complex-join-of-seven-tables-left-self-join-using-sql-arrays
    https://github.com/rogerjdeangelis/utl-computing-counts-using-sql-arrays
    https://github.com/rogerjdeangelis/utl-computing-maxs-and-mins-of-all-numeric-and-character-variables-using-sql-arrays
    https://github.com/rogerjdeangelis/utl-pivot-long-pivot-wide-transpose-partitioning-sql-arrays-wps-r-python
    https://github.com/rogerjdeangelis/utl-simple-example-of-sql-array
    https://github.com/rogerjdeangelis/utl-simplifying-repetitive-code-using-sql-arrays-and-Barts-array-macro
    https://github.com/rogerjdeangelis/utl-sql-arrays-for-very-fast-in-database-processing
    https://github.com/rogerjdeangelis/utl-stack-columns-and-count-values-using-with-and-without-sql-arrays-in-sas-r-and-python
    https://github.com/rogerjdeangelis/utl-transpose-pivot-wide-to-long-using-sql-arrays-in-sas-r-and-python
    https://github.com/rogerjdeangelis/utl-transposing-and-summarizing-by-group-using-sql-arrays
    https://github.com/rogerjdeangelis/utl-transposing-multiple-variables-using-transpose-macro-sql-arrays-proc-report
    https://github.com/rogerjdeangelis/utl-transposing-pivoting-minimums-using-sql-arrays
    https://github.com/rogerjdeangelis/utl-using-sql-arrays-and-do_over-to-cast_0_1-to-Y-N
    https://github.com/rogerjdeangelis/utl-using-sql-arrays-substract-the-values-in-row2-row4-from-row1-by-column
    https://github.com/rogerjdeangelis/utl-using-sql-arrays-to-implement-datastep-variable-ranges
    https://github.com/rogerjdeangelis/utl-very-fast-processing-using-sql-arrays-on-large-core-systems

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
