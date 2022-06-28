QPS (Query per second) （每秒查询量）

TPS(Transaction per second) （每秒事务量，如果是InnoDB会显示，没有InnoDB就不会显示）

计算方法

QPS
```
Questions = SHOW GLOBAL STATUS LIKE 'Questions';
Uptime = SHOW GLOBAL STATUS LIKE 'Uptime';
QPS=Questions/Uptime
TPS
Com_commit = SHOW GLOBAL STATUS LIKE 'Com_commit';
Com_rollback = SHOW GLOBAL STATUS LIKE 'Com_rollback';
Uptime = SHOW GLOBAL STATUS LIKE 'Uptime';
TPS=(Com_commit + Com_rollback)/Seconds
QPS 
mysqladmin -h192.168.160.100 -uroot -p extended-status --relative --sleep=1|grep -w Questions 
```

TPS :下面两个值相加 
```
Com_commit=mysqladmin -h192.168.160.100 -uroot -p extended-status --relative --sleep=1|grep -w Com_commit 

Com_rollback=mysqladmin -h192.168.160.100 -uroot -p extended-status --relative --sleep=1|grep -w Com_rollback 

TPS = $Com_commit + $Com_rollback
```
统计QPS、TPS的脚本
```
#!/bin/bash
mysqladmin -uroot -h192.168.160.43 -p'000000' extended-status -i1|awk 'BEGIN{local_switch=0;print "QPS   Commit Rollback   TPS    Threads_con Threads_run \n------------------------------------------------------- "}
     $2 ~ /Queries$/            {q=$4-lq;lq=$4;}
     $2 ~ /Com_commit$/         {c=$4-lc;lc=$4;}
     $2 ~ /Com_rollback$/       {r=$4-lr;lr=$4;}
     $2 ~ /Threads_connected$/  {tc=$4;}
     $2 ~ /Threads_running$/    {tr=$4;
        if(local_switch==0) 
                {local_switch=1; count=0}
        else {
                if(count>10) 
                        {count=0;print "------------------------------------------------------- \nQPS   Commit Rollback   TPS    Threads_con Threads_run \n------------------------------------------------------- ";}
                else{ 
                        count+=1;
                        printf "%-6d %-8d %-7d %-8d %-10d %d \n", q,c,r,c+r,tc,tr;
                }
        }
}'
```