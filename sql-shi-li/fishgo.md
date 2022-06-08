# FishGo

### **查留存(所有版本)**

```
with toDate('2022-04-08') as start_date, toDate('2022-04-13') as end_date
select create_date,
countIf(distinct player_id, event_date - create_date = 0) as retention_0,
countIf(distinct player_id, event_date - create_date = 1) / retention_0 as retention_1,
countIf(distinct player_id, event_date - create_date = 2) / retention_0 as retention_2,
countIf(distinct player_id, event_date - create_date = 3) / retention_0 as retention_3,
countIf(distinct player_id, event_date - create_date = 4) / retention_0 as retention_4,
countIf(distinct player_id, event_date - create_date = 5) / retention_0 as retention_5,
countIf(distinct player_id, event_date - create_date = 6) / retention_0 as retention_6,
countIf(distinct player_id, event_date - create_date = 7) / retention_0 as retention_7,
countIf(distinct player_id, event_date - create_date = 14) / retention_0 as retention_14,
countIf(distinct player_id, event_date - create_date = 30) / retention_0 as retention_30
from fgi.event
where
event_date between start_date and addDays(end_date, 30)
and create_date between start_date and end_date
and channel_id = 'android_helloworld'
group by create_date
```

### **查留存(特定版本)**

```
drop table if exists fgi.player_create_version;;
create table fgi.player_create_version(player_id String, create_version String) engine=Memory as select player_id,min(version) from fgi.event group by player_id;;
with toDate('2022-04-08') as start_date, toDate('2022-04-13') as end_date
select create_date,
countIf(distinct player_id, event_date - create_date = 0) as retention_0,
countIf(distinct player_id, event_date - create_date = 1) / retention_0 as retention_1,
countIf(distinct player_id, event_date - create_date = 2) / retention_0 as retention_2,
countIf(distinct player_id, event_date - create_date = 3) / retention_0 as retention_3,
countIf(distinct player_id, event_date - create_date = 4) / retention_0 as retention_4,
countIf(distinct player_id, event_date - create_date = 5) / retention_0 as retention_5,
countIf(distinct player_id, event_date - create_date = 6) / retention_0 as retention_6,
countIf(distinct player_id, event_date - create_date = 7) / retention_0 as retention_7,
countIf(distinct player_id, event_date - create_date = 14) / retention_0 as retention_14,
countIf(distinct player_id, event_date - create_date = 30) / retention_0 as retention_30
from fgi.event join fgi.player_create_version using player_id
where
event_date between start_date and addDays(end_date, 30)
and create_date between start_date and end_date
and channel_id = 'android_helloworld'
and create_version >= '2040000'
group by create_date
```

### **查事件留存**

```
with toDate('2022-04-01') as start_date, toDate('2022-04-07') as end_date
select create_date,sum(event_date = create_date) as retention_0,
sum(event_date - create_date = 1) as retention_1,
sum(event_date - create_date = 2) as retention_2,
sum(event_date - create_date = 3) as retention_3,
sum(event_date - create_date = 4) as retention_4,
sum(event_date - create_date = 5) as retention_5,
sum(event_date - create_date = 6) as retention_6,
sum(event_date - create_date = 13) as retention_13,
sum(event_date - create_date = 29) as retention_29
from fgi.event where 
event_date between start_date and addDays(end_date, 29)
and create_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
and event_type = 'RV_Show'
group by create_date
```

### **查分组**

查某个版本有多少玩家分组

```
select version, user_group from fgi.event where
version = '2035000'
group by version, user_group
order by version
```

### **事件按参数分类**

```
with toDate('2022-04-01') as start_date, toDate('2022-04-07') as end_date
select string_params['method'] as method,
sum(event_date - start_date = 0) as day0,
sum(event_date - start_date = 1) as day1,
sum(event_date - start_date = 2) as day2,
sum(event_date - start_date = 3) as day3,
sum(event_date - start_date = 4) as day4,
sum(event_date - start_date = 5) as day5,
sum(event_date - start_date = 6) as day6,
sum(event_date - start_date = 13) as day13,
sum(event_date - start_date = 29) as day29
from fgi.event where
event_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
and event_type = 'RV_Show'
group by method
```

### **活跃玩家等级分布**

若要查段位等级，把下面的level1全部替换成level2即可

```
select level1, count() from (
with toDate('2022-04-01') as start_date, toDate('2022-04-07') as end_date
select player_id, max(level1) as level1 from fgi.event where
event_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
group by player_id
) group by level1
```

### **查看有多少事件类型**

```
select event_type from fgi.event
where event_date = '2022-04-01'
group by event_type order by event_type
```

### **查看事件有多少参数类型**

```
select string_params.keys as keys from fgi.event where
event_type = 'RV_Show'
and event_date = '2022-04-01'
group by keys

select int_params.keys as keys from fgi.event where
event_type = 'RV_Show'
and event_date = '2022-04-01'
group by keys
```

### **查看事件有多少种参数类型，例如RV\_Show有多少种method**

```
select string_params['method'] as method from fgi.event where
event_type = 'RV_Show'
and event_date between '2022-04-01' and '2022-04-30'
group by method order by method
```

### **新用户平均在线时长**

```
select create_date, t1.player_num as player_num,
round(t2.retention_0 / 2 / player_num, 2) as time0,
round(t2.retention_1 / 2 / player_num, 2) as time1,
round(t2.retention_2 / 2 / player_num, 2) as time2,
round(t2.retention_3 / 2 / player_num, 2) as time3,
round(t2.retention_4 / 2 / player_num, 2) as time4,
round(t2.retention_5 / 2 / player_num, 2) as time5,
round(t2.retention_6 / 2 / player_num, 2) as time6,
round(t2.retention_13 / 2 / player_num, 2) as time13,
round(t2.retention_29 / 2 / player_num, 2) as time29
from
(
with toDate('2022-04-01') as start_date, toDate('2022-04-07') as end_date
select create_date, countDistinct(player_id) as player_num from fgi.event where
event_date between start_date and addDays(end_date, 29)
and create_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
group by create_date
) t1
right join
(
with toDate('2022-04-01') as start_date, toDate('2022-04-07') as end_date
select create_date,sum(event_date = create_date) as retention_0,
sum(event_date - create_date = 1) as retention_1,
sum(event_date - create_date = 2) as retention_2,
sum(event_date - create_date = 3) as retention_3,
sum(event_date - create_date = 4) as retention_4,
sum(event_date - create_date = 5) as retention_5,
sum(event_date - create_date = 6) as retention_6,
sum(event_date - create_date = 13) as retention_13,
sum(event_date - create_date = 29) as retention_29
from fgi.event where 
event_date between start_date and addDays(end_date, 29)
and create_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
and event_type = 'heartbeat'
group by create_date
) t2 using create_date
```

### **活跃用户平均在线时长**

```
select event_date,
t1.player_num as player_num,
round(t2.heartbeat / 2 / player_num, 2) as time
from
(
with toDate('2021-12-01') as start_date, toDate('2021-12-07') as end_date
select event_date, countDistinct(player_id) as player_num from fgi.event where
event_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
group by event_date
) t1
right join
(
with toDate('2021-12-01') as start_date, toDate('2021-12-07') as end_date
select event_date, count() as heartbeat
from fgi.event where 
event_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
and event_type = 'heartbeat'
group by event_date
) t2 using event_date
```

### 每日活跃玩家数量

```
with toDate('2021-12-01') as start_date, toDate('2021-12-07') as end_date
select event_date, countDistinct(player_id) as active_num
from fgi.event where
event_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
group by event_date
```

### 活跃玩家国家分布

```
with toDate('2021-12-01') as start_date, toDate('2021-12-07') as end_date
select country, countDistinct(player_id) as Number from fgi.event where
event_date between start_date and end_date
and channel_id = 'android_helloworld'
and version >= '2030000'
group by country
```

### 活跃玩家版本分布

```
with toDate('2021-12-01') as start_date, toDate('2021-12-07') as end_date
select version, countDistinct(player_id) as Number from fgi.event where
event_date between start_date and end_date
and channel_id = 'android_helloworld'
group by version
```

### 活跃玩家留存（只能一天天查）

```
select
  sum(r[1]) as retention_0,
  round(sum(r[2]) / sum(r[1]), 4) as retention_1,
  round(sum(r[3]) / sum(r[1]), 4) as retention_2,
  round(sum(r[4]) / sum(r[1]), 4) as retention_3,
  round(sum(r[5]) / sum(r[1]), 4) as retention_4,
  round(sum(r[6]) / sum(r[1]), 4) as retention_5,
  round(sum(r[7]) / sum(r[1]), 4) as retention_6,
  round(sum(r[8]) / sum(r[1]), 4) as retention_7,
  round(sum(r[9]) / sum(r[1]), 4) as retention_8,
  round(sum(r[10]) / sum(r[1]), 4) as retention_9,
  round(sum(r[11]) / sum(r[1]), 4) as retention_10,
  round(sum(r[12]) / sum(r[1]), 4) as retention_11,
  round(sum(r[13]) / sum(r[1]), 4) as retention_12,
  round(sum(r[14]) / sum(r[1]), 4) as retention_13,
  round(sum(r[15]) / sum(r[1]), 4) as retention_14,
  round(sum(r[16]) / sum(r[1]), 4) as retention_29
from
  (
    with toDate('2021-12-01') as dt                -- 这里改日期
    select
      player_id,
      retention(
        toDate(event_date) = dt,
        toDate(subtractDays(event_date, 1)) = dt,
        toDate(subtractDays(event_date, 2)) = dt,
        toDate(subtractDays(event_date, 3)) = dt,
        toDate(subtractDays(event_date, 4)) = dt,
        toDate(subtractDays(event_date, 5)) = dt,
        toDate(subtractDays(event_date, 6)) = dt,
        toDate(subtractDays(event_date, 7)) = dt,
        toDate(subtractDays(event_date, 8)) = dt,
        toDate(subtractDays(event_date, 9)) = dt,
        toDate(subtractDays(event_date, 10)) = dt,
        toDate(subtractDays(event_date, 11)) = dt,
        toDate(subtractDays(event_date, 12)) = dt,
        toDate(subtractDays(event_date, 13)) = dt,
        toDate(subtractDays(event_date, 14)) = dt,
        toDate(subtractDays(event_date, 29)) = dt
      ) as r
    from
      fgi.event
    where
      event_date >= dt
      and event_date <= addDays(dt, 29)
      and channel_id = 'android_helloworld'         -- 这里改渠道
      and version >= '2030000'                     -- 这里改版本号
    group by
      player_id
  )
```

### 活跃留存（后面留存了，前面统统算留存）

```
select
    sum(r[1]) as retention_0,
    round(sum(r[2]) / retention_0, 4) as retention_1,
    round(sum(r[3]) / retention_0, 4) as retention_2,
    round(sum(r[4]) / retention_0, 4) as retention_3,
    round(sum(r[5]) / retention_0, 4) as retention_4,
    round(sum(r[6]) / retention_0, 4) as retention_5,
    round(sum(r[7]) / retention_0, 4) as retention_6,
    round(sum(r[8]) / retention_0, 4) as retention_7,
    round(sum(r[9]) / retention_0, 4) as retention_8,
    round(sum(r[10]) / retention_0, 4) as retention_9,
    round(sum(r[11]) / retention_0, 4) as retention_10,
    round(sum(r[12]) / retention_0, 4) as retention_11,
    round(sum(r[13]) / retention_0, 4) as retention_12,
    round(sum(r[14]) / retention_0, 4) as retention_13,
    round(sum(r[15]) / retention_0, 4) as retention_14,
    round(sum(r[16]) / retention_0, 4) as retention_29
from
    (
        with toDate('2021-12-01') as dt                -- 这里改日期
        select
            player_id,
            retention(
            event_date = dt,
            event_date - dt >= 1,
            event_date - dt >= 2,
            event_date - dt >= 3,
            event_date - dt >= 4,
            event_date - dt >= 5,
            event_date - dt >= 6,
            event_date - dt >= 7,
            event_date - dt >= 8,
            event_date - dt >= 9,
            event_date - dt >= 10,
            event_date - dt >= 11,
            event_date - dt >= 12,
            event_date - dt >= 13,
            event_date - dt >= 14,
            event_date - dt >= 29
            ) as r
        from
            fgi.event
        where
            event_date >= dt
          and event_date <= addDays(dt, 29)
          and channel_id = 'android_helloworld'         -- 这里改渠道
          and version >= '2030000'                     -- 这里改版本号
        group by
            player_id
    )
```

### 活跃玩家创建时间分布

```
with toDate('2021-12-01') as start_date, toDate('2021-12-07') as end_date
select create_date, countDistinct(player_id) as Number from fgi.event where
event_date between start_date and end_date
and channel_id = 'android_helloworld'
and version > '2030000'
group by create_date
```

### 开屏广告收入

```
select `date`, sum(estimated_earnings) / 1000000 as op_revenue from admob.admob_network 
where 
app_name = 'Fish Go.io - Be the fish king'
and `format`='app_open'
and `date` between '2022-02-15' and '2022-02-30'
group by `date`
```

### 视频插屏广告收入

```
select `date`, sumIf(estimated_revenue, ad_format = 'REWARD') as rv_revenue,
sumIf(estimated_revenue, ad_format = 'INTER') as in_revenue
from max.ad_revenue 
where bundle_id = 'com.whitedot.bfg'
and `date` between '2022-02-15' and '2022-02-30'
group by `date`
```

### 内购收入

```
select `date`, sum(revenues) as iap_revenue from tenjin.spend where 
bundle_id = 'com.whitedot.bfg' 
and `date` between '2022-02-15' and '2022-02-30'
group by `date`
```

### 日报

（分两个查询，先修改参数，执行第一个查询，注意，第一个查询有两句SQL，需要全选，然后点run selected，不要点run current，执行完第一个查询再执行第二个查询即可）

```
truncate table tools.query_setting;;
-- 海外安卓
-- insert into tools.query_setting values ('com.whitedot.bfg', 'android_helloworld', 'ca-app-pub-1425591255748421~3069300090', '2022-04-01', '2022-04-12');;
-- 海外iOS
insert into tools.query_setting values ('com.marsgame.bfg', 'ios_helloworld', 'ca-app-pub-1425591255748421~4969437643', '2022-04-01', '2022-04-12');;
-- 国内iOS
--insert into tools.query_setting values ('com.marsgame.qmmy', 'ios_hg', '', '2022-04-01', '2022-04-12');;
```

```
select
`date` as `日期`,
`DNU`,
`DAU`,
`heartbeat` / 2 / `DAU` as `活跃时长`,
`RV_Show` as `RV次数`,
`IN_Show` as `IN次数`,
`OP_Close` as `OP次数`,
`rv_revenue` as `RV收入`,
`in_revenue` as `IN收入`,
`op_revenue` as `OP收入`,
`iap_revenue` as `内购收入`,
`spend` as `投放`,
`player_num_1` / `DNU` as `次留`,
`player_num_6` / `DNU` as `7留`
from
(
select * from
(
select
`event_date` as `date`,
countIf(DISTINCT player_id, create_date = event_date) as `DNU`,
count(DISTINCT player_id) as `DAU`,
countIf(event_type='heartbeat') as `heartbeat`,
countIf(event_type='RV_Show') as `RV_Show`,
countIf(event_type='IN_Show') as `IN_Show`,
countIf(event_type='OP_Close') as `OP_Close`
from fgi.event join tools.query_setting using channel_id
where `event_date` between start_date and end_date
group by `event_date`
) 
t1 full outer join
(
select create_date as `date`,
countIf(distinct player_id, event_date - create_date = 1) as player_num_1,
countIf(distinct player_id, event_date - create_date = 6) as player_num_6
from fgi.event join tools.query_setting using channel_id
where
event_date between start_date and addDays(end_date, 7)
and create_date between start_date and end_date
group by create_date
) 
t2 using `date`
)
t1 full outer join
(
select * from
(
select `date`, sum(estimated_earnings) / 1000000 as op_revenue 
from admob.admob_network join tools.query_setting on app_id = admob_app_id
where `format`='app_open'
and `date` between start_date and end_date
group by `date`
)
t1 full outer join
(
select * from
(
select `date`, sumIf(estimated_revenue, ad_format = 'REWARD') as rv_revenue,
sumIf(estimated_revenue, ad_format = 'INTER') as in_revenue
from max.ad_revenue join tools.query_setting using bundle_id
where `date` between start_date and end_date
group by `date`
)
t1 full outer join
(
select `date`, sum(spend) as spend, sum(revenues) as iap_revenue 
from tenjin.spend join tools.query_setting using bundle_id
where `date` between start_date and end_date
group by `date`
)
t2 using `date`
)
t2 using `date`
)
t2 using `date`;;
```

### 特定版本创建的玩家的某个事件的人均触发数队列

```
drop table if exists fgi.player_create_version;;
create table fgi.player_create_version(player_id String, create_version String) engine=Memory as select player_id,min(version) from fgi.event group by player_id;;
with toDate('2022-04-08') as start_date, toDate('2022-04-13') as end_date, 'RV_Show' as event_name -- 这里改日期和时间名
select create_date,
countIf(distinct player_id, event_date - create_date = 0) as dnu,
countIf(event_type = event_name and event_date - create_date = 0) / dnu as retention_0,
countIf(event_type = event_name and event_date - create_date = 1) / dnu as retention_1,
countIf(event_type = event_name and event_date - create_date = 2) / dnu as retention_2,
countIf(event_type = event_name and event_date - create_date = 3) / dnu as retention_3,
countIf(event_type = event_name and event_date - create_date = 4) / dnu as retention_4,
countIf(event_type = event_name and event_date - create_date = 5) / dnu as retention_5,
countIf(event_type = event_name and event_date - create_date = 6) / dnu as retention_6,
countIf(event_type = event_name and event_date - create_date = 7) / dnu as retention_7
from fgi.event join fgi.player_create_version using player_id
where
event_date between start_date and addDays(end_date, 8)
and create_date between start_date and end_date
and channel_id = 'android_helloworld' -- 渠道号
and create_version >= '2040000' -- 版本号
group by create_date
```

### 第X分钟看广告次数

```
select create_date,
sumIf(num, diff_minute = 1) `0 min`,
sumIf(num, diff_minute = 1) `1 min`,
sumIf(num, diff_minute = 2) `2 min`,
sumIf(num, diff_minute = 30) `30 min`
from (
with toDate('2022-05-01') as start_date, toDate('2022-05-04') as end_date -- 这里改时间
select create_date, dateDiff('minute', create_time, event_time) as diff_minute, count() as num
from fgi.event where
event_date between start_date and end_date
and create_date between start_date and end_date
and event_type = 'RV_Show' -- 这里改事件
and diff_minute between 0 and 30 -- 这里改最大分钟数
-- and channel_id = 'android_helloworld'
-- and country = 'US'
-- and user_group like 'A%'
group by create_date, diff_minute
) group by create_date order by create_date
```
