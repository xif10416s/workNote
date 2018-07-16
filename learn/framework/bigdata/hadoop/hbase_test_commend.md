scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"FirstKeyOnlyFilter() AND ValueFilter(>=,'binary:0') AND SKIP ValueFilter(=,'binary:-1') ",LIMIT => 3 }

 SingleColumnValueFilter ('f', 'q',=,'binary:-1')


scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>" SingleColumnValueExcludeFilter ('f', 'q',=,'-1') ",LIMIT => 3 }

scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>" ValueFilter(>=,'binary:-1')  ",LIMIT => 3 }

scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"  SingleColumnValueFilter ('f', 'q',=,'binary:-1')  ",LIMIT => 3 }


scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(=,'binary:2') ",LIMIT => 3 }



scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>" RowFilter( = ,'regexstring:1') ",LIMIT => 3 ,REVERSED => TRUE}


scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(!=,'binary:-2') AND RowFilter( = ,SubstringComparator.new('1')) ",LIMIT => 3 }

get 'test:mt2','row-key-6', FILTER=>"SingleColumnValueFilter('f', 'regexstring:32345',=,'binary:-1' ,true ,true) "

get 'test:mt2','row-key-6', FILTER=>"FirstKeyOnlyFilter() AND ValueFilter(!=,'binary:-1')  AND ColumnCountGetFilter (4)  "
ColumnCountGetFilter (4)


get 'test:mt2','row-key-6', FILTER=>"ValueFilter(!=,'binary:-1')  AND ColumnCountGetFilter (3)  "

scan 'dyd:user_recommend_posts_v2' ,{LIMIT =>10 , STARTROW => '1a43690a46ab7b57',STOPROW=> '1a43690a46ab7b57'}

scan 'dyd:user_recommend_posts_v2' ,{LIMIT =>10}


scan 'dyd:user_recommend_posts_v2', { STARTROW => '000027266e566b25-',STOPROW=> '000027266e566b25.', FILTER=>"SKIP ValueFilter(=,'binary:0') ",LIMIT => 3 }

 put 'test:mt2','row-key-7','f:11111','als'
 put 'test:mt2','row-key-7','f:11111','ugs'

[(k,v) for (k,v) in  a.items() if k in b]

+-------------------+
|       read_user_id|
+-------------------+
|6294360860144955629|
|6294360860145305962|
|6293274501955978831|
|6294360860144133183|
|6294196636900010216|
|6293877835966439817|
|6294360860118806471|
|6294196636883066993|
|6294360860144893338|
|6294196636863332144|
|6294360860122623332|
|6294360860143088237|
|6294196636904651536|
|6294360860144369802|
|6294189047310624874|
|6294196636908134144|
|6294196636886622860|
|6293877835980780913|
|6294360860124934197|
|6294360860119152293|
+-------------------+

get 'dyd:user_recommend_posts_v2' ,'1a43690a46ab7b57-6294360860145305962'

put 'test:mt2','row-key-9','f:ugs',-1
put 'test:mt2','row-key-9','f:d',-1

scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND ColumnPrefixFilter('ugs')",LIMIT => 100 }

scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>" ColumnPrefixFilter('ugs')  AND SKIP ValueFilter(!=,'binary:-1') ",LIMIT => 100 }

scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') ",LIMIT => 100 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'de0ea837adf0a575-',STOPROW=> 'de0ea837adf0a575.',LIMIT => 10 }


scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND SKIP ColumnPrefixFilter('ugs')",LIMIT => 100 }

 put 'test:mt2','row-key-12','f:20141225',0 , {'TTL'=>10000}

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'de0ea837adf0a575-',STOPROW=> 'de0ea837adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')",LIMIT => 100 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'de0ea837adf0a575-',STOPROW=> 'de0ea837adf0a575.', FILTER=>"ValueFilter(=,'binary:-1') AND QualifierFilter(=,'binary:d')",LIMIT => 1000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '000bbf57adf0a575-',STOPROW=> '000bbf57adf0a575.',LIMIT => 1000 }

scan 'dyd:user_recommend_posts_v2', {LIMIT => 1000}

put 'dyd:user_recommend_posts_v2','row-key-9','f:d',-1

diyidan,6293576039318286780 , cb1f97e101647575

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.',LIMIT => 1000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.', FILTER=>"ValueFilter(=,'binary:-1') AND QualifierFilter(=,'binary:d')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.', FILTER=>"QualifierFilter(=,'binary:cb')",LIMIT => 5000 }


scan 'dyd:user_recommend_posts_v2', {FILTER=>"ValueFilter(=,'binary:-1') AND QualifierFilter(=,'binary:d')",LIMIT => 1000}

ROWS=167961215 
245624382
ROWS=799938384



scan 'dyd:user_recommend_posts_v2', { STARTROW => '00000147adf0a575-',STOPROW=> '00000147adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', {STARTROW => '00000147adf0a575-',STOPROW=> '00000147adf0a575.',FILTER=>"ValueFilter(=,'binary:-1') AND QualifierFilter(=,'binary:d')",LIMIT => 5000}

/**
 cb1f97e101647575-ffc81547adf0a57 column=f:ugs, timestamp=1519458855435, value=0
 5
 cb1f97e101647575-ffdc4b47adf0a57 column=f:ugs, timestamp=1519458855503, value=0
 5
1281 row(s) in 1.2940 seconds

get 'dyd:user_recommend_posts_v2' ,  'cb1f97e101647575-ffc81547adf0a575'

put 'dyd:user_recommend_posts_v2','cb1f97e101647575-ffc81547adf0a575','f:test',0 , {'TTL'=>100000}
/


scan 'dyd:cb_user_profile', { STARTROW => 'cb1f97e101647575',STOPROW=> 'cb1f97e101647575',LIMIT => 5000 }

get 'dyd:cb_user_profile' ,  'cb1f97e101647575'

//6292747451803655377
//1dc500e877454575
scan 'dyd:user_recommend_posts_v2', { STARTROW => '1dc500e877454575-',STOPROW=> '1dc500e877454575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cus_cf')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '1dc500e877454575-',STOPROW=> '1dc500e877454575.', FILTER=>"SingleColumnValueExcludeFilter('f','d',>=,'binary:20180605',false,false) AND SingleColumnValueExcludeFilter('f','r',!=,'binary:-1',false,false)    AND QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '1dc500e877454575-',STOPROW=> '1dc500e877454575.',FILTER=>"QualifierFilter(=,'binary:d')", LIMIT => 5000,TIMERANGE => [1528093959000, 1922397653000] }

incr 'dyd:user_recommend_posts_v2','1dc500e877454575-ffe024e8eb40b575','f:d',-1

get_counter 'dyd:user_recommend_posts_v2','1dc500e877454575-ffe024e8eb40b575','f:d'

scan 'dyd:user_recommend_posts_v2', { STARTROW => '1dc500e877454575-',STOPROW=> '1dc500e877454575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '1dc500e877454575-',STOPROW=> '1dc500e877454575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '1dc500e877454575-',STOPROW=> '1dc500e877454575.', FILTER=>"QualifierFilter(=,'binary:r')",LIMIT => 100 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '1dc500e877454575-',STOPROW=> '1dc500e877454575.', FILTER=>" QualifierFilter(=,'binary:ugs')",LIMIT => 500 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '1dc500e877454575-',STOPROW=> '1dc500e877454575.',FILTER=>"QualifierFilter(=,'binary:d')", LIMIT => 5000,TIMERANGE => [1525318061000, 1922397653000] }

get 'dyd:cb_user_profile' ,'1dc500e877454575'

get 'dyd:user_recommend_posts_v2', '00000747adf0a575-1f39b967adf0a575'

scan 'dyd:user_recommend_posts_v2', {FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')", LIMIT => 1000,TIMERANGE => [1222388858000, 1521265658000] }

scan 'dyd:user_recommend_posts_v2', {FILTER=>"QualifierFilter(=,'binary:cb')", LIMIT => 10,TIMERANGE => [1523149302000, 1523176907000] }

scan 'dyd:user_recommend_posts_v2', {FILTER=>"QualifierFilter(=,'binary:r')", LIMIT => 10 ,TIMERANGE => [1524882651000, 1923248098000] }

// zji 6294360860163905699
//3a811367adf0a575

get 'dyd:user_recommend_posts_v3','f54f96e101647575' , { FILTER=> "ColumnCountGetFilter(10) AND FamilyFilter(=,'binary:t')"}

get 'dyd:user_recommend_posts_v3','3a811367adf0a575' , { FILTER=> "ColumnCountGetFilter(10) AND FamilyFilter(=,'binary:t')"}

get 'dyd:user_recommend_posts_v3','3a811367adf0a575' , { FILTER=> "ColumnCountGetFilter(10) AND FamilyFilter(=,'binary:t') AND ValueFilter(=,'substring:ugs')"}

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>"SingleColumnValueExcludeFilter('f','d',<,'binary:20180103',false,false) AND SingleColumnValueExcludeFilter('f','r',!=,'binary:-1',false,false)    AND QualifierFilter(=,'binary:raib')",LIMIT => 5000 }

6.6 531  0
6.5 585  1
6.4 683  1   


scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>"SingleColumnValueExcludeFilter('f','d',>=,'binary:20180602',false,false) AND QualifierFilter(=,'binary:d')",LIMIT => 50 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v3', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', LIMIT => 10 }

val get = new Get(Bytes.toBytes("3a811367adf0a575-9fb47be8eb40b575"))

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cus_cf')",LIMIT => 2000,TIMERANGE => [1526436037000, 1823271969059] }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" QualifierFilter(=,'binary:ugs')",LIMIT => 5000,TIMERANGE => [1527835803000, 1926638812000]}

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 5000}


scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 2000,TIMERANGE => [0, 1523271969059] }


scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.',FILTER=>"QualifierFilter(=,'binary:d')", LIMIT => 5000,TIMERANGE => [1528093959000, 1922397653000] }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.',FILTER=>"ValueFilter(=,'binary:20180605') AND QualifierFilter(=,'binary:d')", LIMIT => 5000 }

395
274 4
121 5
3a811367adf0a575-f5b6e2e8eb40b57  6.5
// 派发或者阅读过的ugs推荐
scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" (SingleColumnValueFilter('f','d',=,'binary:-1',true,true) OR SingleColumnValueFilter('f','r',=,'binary:-1',true,true)) AND  QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" (SingleColumnValueFilter('f','r',=,'binary:-1',true,true)) AND  QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" QualifierFilter(=,'binary:ttl')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" QualifierFilter(=,'binary:r')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" QualifierFilter(=,'binary:d')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" QualifierFilter(=,'binary:cb')",LIMIT => 5000 }

put 'dyd:user_recommend_posts_v2','3a811367adf0a575-f95ed667adf0a575','f:test',2 , {'TTL'=>129600000,VISIBILITY=>'PRIVATE|SECRET'}


 scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>" QualifierFilter(=,'binary:test')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', {  FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 10 } 

scan 'dyd:user_recommend_posts_v2', {STARTROW => '00004667adf0a575-',  FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND (QualifierFilter(=,'binary:cb') OR QualifierFilter(=,'binary:ugs') )",LIMIT => 10 } 
---

scan 'dyd:user_recommend_posts_v2', { STARTROW => '1e156267adf0a575-',STOPROW=> '1e156267adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 5000 }


scan 'dyd:user_recommend_posts_v2', { STARTROW => '1e156267adf0a575-',STOPROW=> '1e156267adf0a575.', FILTER=>" (SingleColumnValueFilter('f','d',=,'binary:-1',true,true) OR SingleColumnValueFilter('f','r',=,'binary:-1',true,true)) AND QualifierFilter(=,'binary:cb')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts_v2', { STARTROW => '3a811367adf0a575-',STOPROW=> '3a811367adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND (QualifierFilter(=,'binary:ugs') OR QualifierFilter(=,'binary:ramb') OR QualifierFilter(=,'binary:raib') OR QualifierFilter(=,'binary:cus_cf') OR QualifierFilter(=,'binary:als_cb')  OR QualifierFilter(=,'binary:cb') ) ",LIMIT => 5000 }
ramb,ugs,raib,cus_cf,als_cb,cb

scan 'dyd:user_recommend_posts_v2', {  FILTER=>"QualifierFilter(=,'binary:cus_cf')",LIMIT => 10 }

scan 'dyd:user_recommend_posts_v2', {  FILTER=>"QualifierFilter(=,'binary:cb')",LIMIT => 10 }

val day = "2018-01-24"

00000747adf0a575-
scan 'dyd:user_recommend_posts_v2', { STARTROW => '00000747adf0a575-',STOPROW=> '00000747adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 5000}

scan 'dyd:user_recommend_posts_v2', {LIMIT => 10 }
----
scan 'dyd:user_recommend_posts_v2', {STARTROW => '',STOPROW=> '0aaaaaa',FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')", LIMIT => 10 ,TIMERANGE => [0, 1523131905000] }


scan 'dyd:user_recommend_posts_v2', {  FILTER=>"QualifierFilter(=,'binary:d')",LIMIT => 20,TIMERANGE => [1528116968000, 1823271969059] }

scan 'dyd:user_recommend_posts_v2', { LIMIT => 200,TIMERANGE => [1528110137000, 1823271969059] }

scan 'dyd:user_recommend_posts_v2', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.',FILTER=>"QualifierFilter(=,'binary:d')", LIMIT => 5000,TIMERANGE => [1528110137, 1922397653000] }


scan 'dyd:user_recommend_posts_v3' , { FILTER=> "ColumnCountGetFilter(1) AND FamilyFilter(=,'binary:t')  AND ValueFilter(=,'substring:ramb')" ,TIMERANGE => [1531202213000, 1922397653000], LIMIT => 20}

scan 'dyd:user_recommend_posts_v3' , { FILTER=> "ColumnCountGetFilter(1) AND FamilyFilter(=,'binary:t')" , LIMIT => 100 ,TIMERANGE => [1528359911000, 1922397653000]}

get 'dyd:user_recommend_posts_v3','83a027f4b38e7575' , { FILTER=> "ColumnCountGetFilter(10) AND FamilyFilter(=,'binary:t')"}