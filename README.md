# utl_sas_r_calculate_connection_duration_by_group
I need to calculate the duration of a connection given time points by user

    SAS/R Calculate total connection durration by user

    see
    https://goo.gl/8yhCCg
    https://stackoverflow.com/questions/47500186/r-identifying-the-first-and-last-elements-in-a-repeating-set-of-duplicates

    I need to calculate the duration of a connection given time points by user

    SOAPBOX ON

     I find SAS to be more readable.

     One of the issues I have is how functions like 'group by' and 'mutate' can
     change the input data structure and the type of the variables. Having many types
     and data structures is both a strength and a weakness.  It would be nice if SAS
     added 128bit floats (to handle bigint and virtual addresses) and longtext(up to 2gb).
     Some but not all these(redundancies?) dictionaries, tibbles, data.frames, data.tables, matrices and vectors.
     Also too many types(ie character), missings(subtypes), atomic, character, factor,....
     More is less.

    SOAPBOX OFF

    INPUT

    WORK.HAVE total obs=10  |                   RULES
                            |                   =====
      Obs    IMG    TIME    |  ELAPSEDTIME
                            |
        1     B       5     |      5     on first.img output and set tym1st=5
        2     B       6     |      2     6 -5 (tym1st) +1
        3     B       8     |      4     8 -5 (tym1st) +1
        4     B       9     |      5     9 -5 (tym1st) +1
        5     B      10     |      6    20 -5 (tym1st) +1
        6     B      11     |      7    11 -5 (tym1st) +1

        7     G      12     |     12    on first.img output and set tym1st=12
        8     G      14     |      3    14 -12 (tym1st) +1
        9     G      16     |      5    16 -12 (tym1st) +1
       10     G      21     |     10    21 -12 (tym1st) +1


    WORKING CODE
    ============

       R   (as.data.frame was not obvious to me)
       ==
       want<-as.data.frame(have %>%
         group_by(IMG) %>%
         mutate(ELAPSEDTIME = TIME - first(TIME) + 1));

       SAS
       ===
        by img;
        select;
          when ( first.img ) do;
             elapsedTime=time;
             tym1st=time;
          end;
          otherwise
             elapsedTime=time-tym1st+1;
        end;


    OUTPUT
    ======

    *                _                _       _
     _ __ ___   __ _| | _____      __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \    / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/   | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|    \__,_|\__,_|\__\__,_|

    ;

    libname xpt xport "d:/xpt/want.xpt";

    data wantsas;
      %utl_rens(xpt.want);
      set want;
    run;quit;

    proc contents data=wantsas;
    run;quit;


     p to 40 obs from wantsas total obs=10

     bs    IMG    TIME    ELAPSEDTIME * long variable name even though SAS V5 transport format

      1     B       5           1
      2     B       6           2
      3     B       8           4
      4     B       9           5
      5     B      10           6
      6     B      11           7
      7     G      12           1
      8     G      14           3
      9     G      16           5
     10     G      21          10

    *                          _       _   _
     ___  __ _ ___   ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _` / __| / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_| \__ \ \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\__,_|___/ |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    data want(drop=tym1st);
      retain tym1st .;
      set sd1.have;
      by img;
      select;
        when ( first.img ) do;
           elapsedTime=time;
           tym1st=time;
        end;
        otherwise
           elapsedTime=time-tym1st+1;
      end;
    run;quit;


    Up to 40 obs WORK.WANTSAS total obs=10

    Obs    IMG    TIME    ELAPSEDTIME

      1     B       5           1
      2     B       6           2
      3     B       8           4
      4     B       9           5
      5     B      10           6
      6     B      11           7
      7     G      12           1
      8     G      14           3
      9     G      16           5
     10     G      21          10

    *____               _       _   _
    |  _ \    ___  ___ | |_   _| |_(_) ___  _ __
    | |_) |  / __|/ _ \| | | | | __| |/ _ \| '_ \
    |  _ <   \__ \ (_) | | |_| | |_| | (_) | | | |
    |_| \_\  |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;


    proc datasets lib=work kill;
    run;quit;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     input Img$ Time;
    cards4;
    B 5
    B 6
    B 8
    B 9
    B 10
    B 11
    G 12
    G 14
    G 16
    G 21
    ;;;;
    run;quit;

    %utl_submit_r64('
    library(dplyr);
    library(SASxport);
    library(haven);
    library(Hmisc);
     have<-read_sas("d:/sd1/have.sas7bdat");
     want<-as.data.frame(have %>%
       group_by(IMG) %>%
       mutate(ELAPSEDTIME = TIME - first(TIME) + 1));
     label(want$ELAPSEDTIME)<-"ELAPSEDTIME";
     write.xport(want,file="d:/xpt/want.xpt",autogen.formats = FALSE);
    ');

    data want_r;
      %utl_rens(xpt.want);
      set want;
    run;quit;


    WORK.WANT_R total obs=10

    Obs    IMG    TIME    ELAPSEDTIME

      1     B       5           1
      2     B       6           2
      3     B       8           4
      4     B       9           5
      5     B      10           6
      6     B      11           7
      7     G      12           1
      8     G      14           3
      9     G      16           5
     10     G      21          10


