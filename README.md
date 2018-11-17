# utl-replace-values-with-thier-frequency-count-across-columns-within-each-row
Replace values with thier frequency count across columns within each row
    Replace values with thier frequency count across columns within each row

    Deceptivley difficut if you are looking for an non brute force solution.

    I thought this would be easier ie count(tx1, in (txs(*))
    Tried
    WHICHN (in R and SAS)
    Summary IDGROUP
    HASH

    There may be an IML solution using sortndx to get indexes of equal value elements

      Three Solutions (all brute force and nor elegant)

          1. Datastep array
          2. Normalize and proc sql
          3. Normalize first.dot and spread

    github
    https://tinyurl.com/yactlo4n
    https://github.com/rogerjdeangelis/utl-replace-values-with-thier-frequency-count-across-columns-within-each-row

    INPUT
    =====

                                            | RULES
    WORK.HAVE total obs=6                   | -----
                                            | 100   103   .   100  100
                                            |  3x    1x   1x   3x   3x
                                            |
     ID    TX1    TX2    TX3    TX4    TX5  |  TX1  TX2  TX3  TX4  TX5
                                            |
      1    100    103      .    100    100  |   3    1    1    3    3
      2      .    102    103    100    101  |
      3    102    100      .      .    100  |
      4    102    101      .    103      .  |
      5      .    102    100    100    101  |


    EXAMPLE OUTPUT
    --------------

     WORK.WANT_DAT total obs=5

     ID    EQ1    EQ2    EQ3    EQ4    EQ5

      1     3      1      1      3      3   (are 3 100s in this row
                                             100 in Tx1 Tx3 Tx4)
      2     1      1      1      1      1
      3     1      2      2      2      2
      4     1      1      2      1      2
      5     1      1      2      2      1


    PROCESS
    =======

    1. Datastep array
    -----------------

    data want_dat;

      set have;
      array txs[*] tx1-tx5;
      array eqs[*] eq1-eq5;
      do beg=1 to dim(txs);
        do end=1 to dim(txs);
          eqs[beg] = sum(eqs[beg],
            (txs[beg] = txs[end]) or (txs[beg]=. and txs[end]=.));
        end;
      end;
      output;
      keep id eq:;

    run;quit;


    2. Normalize and proc sql
    -------------------------

    data havVue/view=havVue;
      set have;
      array txs[*] tx1-tx5;
      do idx=1 to dim(txs);
        var=vname(txs[idx]);
        val=txs[idx];
        output;
      end;
      keep id var val;
    run;quit;

    /*
    WORK.HAVVUE total obs=25

     ID    VAR    VAL

      1    TX1    100
      1    TX2    103
      1    TX3      .
      1    TX4    100
      1    TX5    100
      ....
    */

    proc sql;
      create
         table want_sql as
      select
         *
        ,count((val=.) or (val ne .)) as vals
      from
         havVue
      group
         by id, val
      order
         by id, var;
    ;quit;

    proc transpose data=want_sql out=wantSqlXpo(drop=_name_);
    by id;
    var vals;
    id var;
    run;quit;

    /*
     WANTSQLXPO total obs=5

      ID    TX1    TX2    TX3    TX4    TX5

       1     3      1      1      3      3
       2     1      1      1      1      1
       3     1      2      2      2      2
       4     1      1      2      1      2
       5     1      1      2      2      1
    */


    3. Normalize first.dot and spread
    ---------------------------------

    data havVue/view=havVue;
      set have;
      array txs[*] tx1-tx5;
      do idx=1 to dim(txs);
        var=vname(txs[idx]);
        val=txs[idx];
        output;
      end;
      keep id var val;
    run;quit;

    proc sort data=havVue out=havSrt noequals;
      by id val;
    run;quit;

    /*
     WORK.HAVSRT total obs=25

      ID    VAR    VAL

       1    TX3      .
       1    TX4    100
       1    TX5    100
       1    TX1    100
       1    TX2    103
    */


    data havRol;
       retain id eq1-eq5;
       array eqs[5] eq1-eq5;
       do until (last.val);
         set havSrt;
         by id val;
         if first.val then cnt=0;
         cnt+1;
       end;
       do until (last.val);
         set havSrt;
         by id val;
         idx=input(substr(var,3),3.);
         eqs[idx]=cnt;
         if last.id then output;
       end;
    run;quit;

    /*
     WORK.HAVROL total obs=5

      ID    EQ1    EQ2    EQ3    EQ4    EQ5    VAR    VAL    CNT    IDX

       1     3      1      1      3      3     TX2    103     1      2
       2     1      1      1      1      1     TX3    103     1      3
       3     1      2      2      2      2     TX1    102     1      1
       4     1      1      2      1      2     TX4    103     1      4
       5     1      1      2      2      1     TX2    102     1      2
    */

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;


    data have;
      retain id;
      array txs[*] tx1-tx5;
      do id=1 to 5;
        do xcr=1 to 5;
           txs[xcr]=100+int(4*uniform(8078));
           if uniform(1876)>.75 then txs[xcr]=.;
        end;
        output;
      end;
      drop xcr;
    run;quit;



