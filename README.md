# utl-append-and-split-tables-into-two-tables-one-with-common-variables-and-one-without-dosubl-hash
Append and split tables into two tables one with common variables and one without dosubl hash.

    Append and split tables into two tables one with common variables and one without dosubl hash

    see github
    https://tinyurl.com/ydacr22p
    https://github.com/rogerjdeangelis/utl-append-and-split-tables-into-two-tables-one-with-common-variables-and-one-without-dosubl-hash

    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories


    The solution below is an academic exercise use Bartosz more 'production'
    friendly code. On end.

    Bartosz Jablonski
    yabwon@gmail.com

    I was interested in
       1. Single datastep solution
       2. Experiment with DOSUBL
       3. Hash to output dynamic datasets
       4. Experiment with DO_OVER
       5. Keep varlist active on my synapses (use burns it in)
       6. If SAS develops a smart compiler it may be the fastest

    Two Solutions
       1. Bartosz Jablonski on end
       2. DOSUBL plust HASH


    INPUT  (8 tables with different sets of variables thanks to Bartosz )
    =====

    WORK.Y1 total obs=1

      A    B    C    D    E
      1    2    3    4    5

    WORK.Y2 total obs=1

      A    B    C    D    E    F    G    H
      1    2    3    4    5    6    7    8
    ...
    ...

    WORK.Y8 total obs=1

      A    B     X     Y     Z

      1    2    24    25    26

    EXAMPLE OUTPUT
    ==============

    COMMON VARIABLES
    ----------------

    WORK.WANTCOM total obs=8

      A    B

      1    2
      1    2
      1    2
      1    2
      1    2
      1    2
      1    2
      1    2

    DISJOINT VARIABLES
    ------------------

    WORK.WANTNON total obs=8

     C D E F G H I J K L M N O P Q R S T U V W X Y Z

     3 4 5 6 7 8 . . . . . . . . . . . . . . . . . .
     3 4 5 . . . . . . . . . 5 6 7 . . . . . . . . .
     3 4 5 . . . . . . . . . . . . . . . . . . . . .
     3 4 5 . . . 9 0 . . . . . . . . . . . . . . .
     3 . . . . . . . . . . . . . . . . . 3 3 . . .
     3 4 5 . . . . . . 2 3 4 . . . . . . . . . . . .
     3 4 . . . . . . . . . . . . . 8 9 0 . . . . . .
     . . . . . . . . . . . . . . . . . . . . . 4 5 6


    PROCESS
    =======

    %array(vars,values=1-8);

    data _null_;

      if _n_=0 then do; %let rc=%sysfunc(dosubl('

          options obs=1;
          proc sql;
            create table havCom as
              %do_over(vars,phrase=%str(
                 select * from y?),between=union corr)
          ;quit;
          options obs=max;
          data havComVue/view=havComVue;
             set %do_over(vars,phrase=y?);
             key=_n_;
          run;quit;
          options obs=max;run;quit;
          %let vars=%varlist(havCom);
          '));

      end;

      declare hash hh(dataset: 'havComVue');
      hh.definekey("key") ;
      hh.defineData(all: 'yes');
      hh.definedone() ;

      set havComVue;
   
      hh.output(dataset:"wantNon(drop=&vars drop=key)");
      hh.output(dataset:"wantCom(keep=&vars)");

    run;quit;


    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    %symdel nams;

    proc datasets lib=work;
     delete hav:;
    run;quit;

    data
    y1(keep=a b c d e)
    y2(keep=a b f g h c d e)
    y3(keep=a b i j k c d e)
    y4(keep=a b l m n c d e)
    y5(keep=a b o p q c d e)
    y6(keep=a b r s t c d)
    y7(keep=a b u v w c)
    y8(keep=a b x y z)
    ; /* common vars are: a b and partially: c d e */
    a=1;b=2;
    c=3;d=4;e=5;
    f=6;g=7;h=8;
    i=9;j=10;k=11;
    l=12;m=13;n=14;
    o=15;p=16;q=17;
    r=18;s=19;t=20;
    u=21;v=22;w=23;
    x=24;y=25;z=26;
    ;
    run;

