### создание своей папки
```
mkdir /user/denis/
hadoop fs -mkdir /user/denis
```
### Загружаем полученный файл на hdfs в вашу личную папку.
```
hadoop fs -put /home/tolstoy/voyna-i-mir-tom-1.doc /user/denis/
```
### просмотр содержимого папки
```
hadoop fs -ls /user/denis
```
output

    Found 1 items   
    -rw-r--r--   3 root supergroup    1580032 2023-11-08 15:29 /user/denis/voyna-i-mir-tom-1.doc

### Изменить права доступа
```
hadoop fs -chmod 766 /user/denis/voyna-i-mir-tom-1.doc
```
### Проверка изменений прав доступа
```
hadoop fs -ls /user/denis
```
output

    Found 1 items
    -rwxrw-rw-   3 root supergroup    1580032 2023-11-08 15:29 /user/denis/voyna-i-mir-tom-1.doc
### Просмор размера файла, занимаемого на диске
```
hadoop fs -du /user/denis/voyna-i-mir-tom-1.doc
```
output

    1580032  /user/denis/voyna-i-mir-tom-1.doc
### Просмор размера файла, занимаемого на диске в удобочитаемом виде
```
# hadoop fs -du -s -h /user/denis/voyna-i-mir-tom-1.doc
```
output

    1.5 M  /user/denis/voyna-i-mir-tom-1.doc

### Получение реального числа копий
```
hdfs fsck /user/denis/voyna-i-mir-tom-1.doc
```
output

    Connecting to namenode via http://namenode:50070/fsck?ugi=root&path=%2Fuser%2Fdenis%2Fvoyna-i-mir-tom-1.doc
    FSCK started by root (auth:SIMPLE) from /172.25.0.7 for path /user/denis/voyna-i-mir-tom-1.doc at Thu Nov 09 13:20:31 UTC 2023
    .
    /user/denis/voyna-i-mir-tom-1.doc:  Under replicated BP-146466588-172.19.0.4-1699454664781:blk_1073741830_1006. Target Replicas is 3 but found 1 replica(s).
    Status: HEALTHY
    Total size:    1580032 B
    Total dirs:    0
    Total files:   1
    Total symlinks:                0
    Total blocks (validated):      1 (avg. block size 1580032 B)
    Minimally replicated blocks:   1 (100.0 %)
    Over-replicated blocks:        0 (0.0 %)
    Under-replicated blocks:       1 (100.0 %)
    Mis-replicated blocks:         0 (0.0 %)
    Default replication factor:    3
    Average block replication:     1.0
    Corrupt blocks:                0
    Missing replicas:              2 (66.666664 %)
    Number of data-nodes:          1
    Number of racks:               1
    FSCK ended at Thu Nov 09 13:20:31 UTC 2023 in 30 milliseconds

    The filesystem under path '/user/denis/voyna-i-mir-tom-1.doc' is HEALTHY

### Изменить коэффициент репликации для файла
```
hadoop fs -setrep 2 /user/denis/voyna-i-mir-tom-1.doc
```
output 

    Replication 2 set: /user/denis/voyna-i-mir-tom-1.doc

### Проверить изменения
    hdfs fsck /user/denis/voyna-i-mir-tom-1.doc

output 
```
Connecting to namenode via http://namenode:50070/fsck?ugi=root&path=%2Fuser%2Fdenis%2Fvoyna-i-mir-tom-1.doc
FSCK started by root (auth:SIMPLE) from /172.25.0.7 for path /user/denis/voyna-i-mir-tom-1.doc at Thu Nov 09 14:06:04 UTC 2023
.
/user/denis/voyna-i-mir-tom-1.doc:  Under replicated BP-146466588-172.19.0.4-1699454664781:blk_1073741830_1006. Target Replicas is 2 but found 1 replica(s).
Status: HEALTHY
 Total size:    1580032 B
 Total dirs:    0
 Total files:   1
 Total symlinks:                0
 Total blocks (validated):      1 (avg. block size 1580032 B)
 Minimally replicated blocks:   1 (100.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       1 (100.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     1.0
 Corrupt blocks:                0
 Missing replicas:              1 (50.0 %)
 Number of data-nodes:          1
 Number of racks:               1
FSCK ended at Thu Nov 09 14:06:04 UTC 2023 in 3 milliseconds


The filesystem under path '/user/denis/voyna-i-mir-tom-1.doc' is HEALTHY
```

### Подсчет колличества строк Linux командой
```
wc -l voyna-i-mir-tom-1.doc
```
output 

    901 voyna-i-mir-tom-1.doc
### Подсчет колличества строк в Hadoop
```
hadoop fs -cat /user/denis/voyna-i-mir-tom-1.doc | wc -l
```
output

    901