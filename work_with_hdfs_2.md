hadoop fs -rm -r /user/denis/out_csv
hadoop fs -ls /user/denis/

### Загружаем полученный файл на hdfs в вашу личную папку.
```
hadoop fs -put /home/out_csv /user/denis/
```

hadoop fs -ls /user/denis/out_csv


CREATE DATABASE mydb

### скрипт для customers_df


    set hive.enforce.bucketing=true;


    CREATE TABLE IF NOT EXISTS customers(
        id INT,
        customer_id STRING,
        first_name STRING,
        last_name STRING,
        company STRING,
        city STRING,
        country STRING,
        phone_1 STRING,
        phone_2 STRING,
        email STRING,
        subscription_date STRING,
        website STRING,
        groupp INT
    )
    COMMENT 'customers details'
    PARTITIONED BY(year INT)
    CLUSTERED BY(groupp) INTO 10 BUCKETS
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;



Необходимо создать временную таблицу для вставки бакетированных данных

    CREATE TEMPORARY TABLE temp_customers(
        id INT,
        customer_id STRING,
        first_name STRING,
        last_name STRING,
        company STRING,
        city STRING,
        country STRING,
        phone_1 STRING,
        phone_2 STRING,
        email STRING,
        subscription_date STRING,
        website STRING,
        year INT,
        groupp INT
    )
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;


    describe temp_customers

Загрузить данные во временную таблицу

    LOAD DATA INPATH '/user/denis/out_csv/customers_df.csv' 
    OVERWRITE INTO TABLE temp_customers

    select * from temp_customers limit 10


вставка из времменой таблицы в бакетированную

SET hive.exec.dynamic.partition = true;
set hive.exec.dynamic.partition.mode=nonstrict

    INSERT OVERWRITE TABLE customers PARTITION (year)
        SELECT 
            id,
            customer_id,
            first_name,
            last_name,
            company,
            city,
            country,
            phone_1,
            phone_2,
            email,
            subscription_date,
            website,
            groupp,
            year
        FROM temp_customers;

динамическая вставка медленнее т.к. для вставки файл считывается построчно и используется MapReduce.
Динамические партиции актуальн при ETL процессах


при статичееском добавлении сканируется только колонка партиционирования
set hive.mapred.mode = nonstrict
set hive.strict.checks.bucketing = false
set sethive.strict.checks.bucketing = false
настройки сохраняются в hive-site.xml

CREATE TABLE IF NOT EXISTS t1_customers(
    id INT,
    customer_id STRING,
    first_name STRING,
    last_name STRING,
    company STRING,
    city STRING,
    country STRING,
    phone_1 STRING,
    phone_2 STRING,
    email STRING,
    subscription_date STRING,
    website STRING,
    groupp INT
)
COMMENT 'customers details'
PARTITIONED BY(year INT)
CLUSTERED BY(groupp) INTO 10 BUCKETS
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

LOAD DATA INPATH '/user/denis/out_csv/customers_df.csv' INTO TABLE t1_customers PARTITION(year="2020");
LOAD DATA INPATH '/user/denis/out_csv/customers_df.csv' INTO TABLE t1_customers PARTITION(year="2021");
LOAD DATA INPATH '/user/denis/out_csv/customers_df.csv' INTO TABLE t1_customers PARTITION(year="2022");


 Для загрузки данных из файла organization_df.csv

    CREATE TEMPORARY TABLE temp_organization(
        id INT,
        organization_id STRING,
        name STRING,
        website STRING,
        country STRING,
        description STRING,
        founded INT,
        industry STRING,
        num_of_employees INT
    )
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;

    LOAD DATA INPATH '/user/denis/out_csv/organization_df.csv' 
    OVERWRITE INTO TABLE temp_organization

    CREATE TABLE IF NOT EXISTS organization(
        id INT,
        organization_id STRING,
        name STRING,
        website STRING,
        country STRING,
        description STRING,
        industry STRING,
        num_of_employees INT
    )
    COMMENT 'organization details'
    PARTITIONED BY(founded INT)
    CLUSTERED BY(num_of_employees) INTO 10 BUCKETS
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;


    INSERT OVERWRITE TABLE organization PARTITION (founded)
        SELECT 
            id,
            organization_id,
            name,
            website,
            country,
            description,
            industry,
            num_of_employees,
            founded
        FROM temp_organization;


создать таблицу people и загрузить в нее данные

    CREATE TEMPORARY TABLE temp_people(
        id INT,
        user_id STRING,
        first_name STRING,
        last_name STRING,
        sex STRING,
        email STRING,
        phone STRING,
        date_birth DATE,
        job_title STRING,
        group_people INT,
        year INT
    )
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;

    LOAD DATA INPATH '/user/denis/out_csv/people_df.csv' 
    OVERWRITE INTO TABLE temp_people

    CREATE TABLE IF NOT EXISTS people(
        id INT,
        user_id STRING,
        first_name STRING,
        last_name STRING,
        email STRING,
        phone STRING,
        date_birth DATE,
        job_title STRING,
        group_people INT,
        year INT
    )
    COMMENT 'people details'
    PARTITIONED BY(sex STRING)
    CLUSTERED BY(year) INTO 32 BUCKETS
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;

    INSERT OVERWRITE TABLE people PARTITION (sex)
        SELECT 
            id,
            user_id,
            first_name,
            last_name,
            email,
            phone,
            date_birth,
            job_title,
            group_people,
            year,
            sex
        FROM temp_people;


/user/hive/warehouse/ проверить какие партиции создались


Теперь ваша задача следующая: аналитики хотят сводную статистику на уровне 
каждой компании и 
на уровне каждого года 
получить целевую возрастную группу подписчиков — то есть, возрастную группу, представители которой чаще всего совершали подписку именно в текущий год на текущую компанию. 

select o.name, avg(2023 - c.year) avg_year, c.year
from organization o 
LEFT OUTER JOIN customers c on o.website = c.website 
LEFT OUTER JOIN people p on c.first_name = p.first_name and c.last_name = p.last_name
group by c.year, o.name