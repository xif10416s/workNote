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

scan 'dyd:user_recommend_posts' ,{LIMIT =>10 , STARTROW => '1a43690a46ab7b57',STOPROW=> '1a43690a46ab7b57'}

scan 'dyd:user_recommend_posts' ,{LIMIT =>10}


scan 'dyd:user_recommend_posts', { STARTROW => '000027266e566b25-',STOPROW=> '000027266e566b25.', FILTER=>"SKIP ValueFilter(=,'binary:0') ",LIMIT => 3 }

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

get 'dyd:user_recommend_posts' ,'1a43690a46ab7b57-6294360860145305962'

put 'test:mt2','row-key-9','f:ugs',-1
put 'test:mt2','row-key-9','f:d',-1

scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND ColumnPrefixFilter('ugs')",LIMIT => 100 }

scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>" ColumnPrefixFilter('ugs')  AND SKIP ValueFilter(!=,'binary:-1') ",LIMIT => 100 }

scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') ",LIMIT => 100 }

scan 'dyd:user_recommend_posts', { STARTROW => 'de0ea837adf0a575-',STOPROW=> 'de0ea837adf0a575.',LIMIT => 10 }


scan 'test:mt2', { STARTROW => 'row-key-',STOPROW=> 'row-key.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND SKIP ColumnPrefixFilter('ugs')",LIMIT => 100 }

 put 'test:mt2','row-key-12','f:20141225',0 , {'TTL'=>10000}

scan 'dyd:user_recommend_posts', { STARTROW => 'de0ea837adf0a575-',STOPROW=> 'de0ea837adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')",LIMIT => 100 }

scan 'dyd:user_recommend_posts', { STARTROW => 'de0ea837adf0a575-',STOPROW=> 'de0ea837adf0a575.', FILTER=>"ValueFilter(=,'binary:-1') AND QualifierFilter(=,'binary:d')",LIMIT => 1000 }

scan 'dyd:user_recommend_posts', { STARTROW => '000bbf57adf0a575-',STOPROW=> '000bbf57adf0a575.',LIMIT => 1000 }

scan 'dyd:user_recommend_posts', {LIMIT => 1000}

put 'dyd:user_recommend_posts','row-key-9','f:d',-1

diyidan,6293576039318286780 , cb1f97e101647575

scan 'dyd:user_recommend_posts', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.',LIMIT => 1000 }

scan 'dyd:user_recommend_posts', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.', FILTER=>"ValueFilter(=,'binary:-1') AND QualifierFilter(=,'binary:d')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts', { STARTROW => 'cb1f97e101647575-',STOPROW=> 'cb1f97e101647575.', FILTER=>"QualifierFilter(=,'binary:cb')",LIMIT => 5000 }


scan 'dyd:user_recommend_posts', {FILTER=>"ValueFilter(=,'binary:-1') AND QualifierFilter(=,'binary:d')",LIMIT => 1000}

ROWS=167961215 
245624382
ROWS=799938384



scan 'dyd:user_recommend_posts', { STARTROW => '00000147adf0a575-',STOPROW=> '00000147adf0a575.', FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:ugs')",LIMIT => 5000 }

scan 'dyd:user_recommend_posts', {STARTROW => '00000147adf0a575-',STOPROW=> '00000147adf0a575.',FILTER=>"ValueFilter(=,'binary:-1') AND QualifierFilter(=,'binary:d')",LIMIT => 5000}

/**
 cb1f97e101647575-ffc81547adf0a57 column=f:ugs, timestamp=1519458855435, value=0
 5
 cb1f97e101647575-ffdc4b47adf0a57 column=f:ugs, timestamp=1519458855503, value=0
 5
1281 row(s) in 1.2940 seconds

get 'dyd:user_recommend_posts' ,  'cb1f97e101647575-ffc81547adf0a575'

put 'dyd:user_recommend_posts','cb1f97e101647575-ffc81547adf0a575','f:test',0 , {'TTL'=>100000}
/


scan 'dyd:cb_user_profile', { STARTROW => 'cb1f97e101647575',STOPROW=> 'cb1f97e101647575',LIMIT => 5000 }

get 'dyd:cb_user_profile' ,  'cb1f97e101647575'



scan 'dyd:user_recommend_posts', {  FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND QualifierFilter(=,'binary:cb')",LIMIT => 10 } 

scan 'dyd:user_recommend_posts', {STARTROW => '00004667adf0a575-',  FILTER=>"SKIP ValueFilter(!=,'binary:-1') AND (QualifierFilter(=,'binary:cb') OR QualifierFilter(=,'binary:ugs') )",LIMIT => 10 } 