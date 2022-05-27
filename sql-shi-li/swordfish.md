# SwordFish

### 常规数据（DNU, DAU, 活跃）

```
select
`event_date` as `date`,
countIf(DISTINCT player_id, create_date = event_date) as `DNU`,
count(DISTINCT player_id) as `DAU`,
countIf(event_type='heartbeat') / 2 / `DAU` as `heartbeat`
from sfi.event
where `event_date` between '2022-04-01' and '2022-04-13' -- 此处修改日期
and user_group = 'B' -- 次数修改分组
group by `event_date`
```

### 留存

```
with toDate('2022-04-01') as start_date, toDate('2022-04-13') as end_date, 'A' as group
select create_date as `date`,
       countIf(distinct player_id, event_date - create_date = 0) as user_0d,
       countIf(distinct player_id, event_date - create_date = 1) as user_1d,
       countIf(distinct player_id, event_date - create_date = 2) as user_2d,
       countIf(distinct player_id, event_date - create_date = 3) as user_3d,
       countIf(distinct player_id, event_date - create_date = 4) as user_4d,
       countIf(distinct player_id, event_date - create_date = 5) as user_5d,
       countIf(distinct player_id, event_date - create_date = 6) as user_6d
from sfi.event
where event_date between start_date and addDays(end_date, 7)
  and create_date between start_date and end_date
  and user_group = group
group by create_date
```

### 对局漏斗

```
select
`event_date` as `date`,
countIf(event_type='game_over' and int_params['game_round'] = 1) as round_1,
countIf(event_type='game_over' and int_params['game_round'] = 2) as round_2,
countIf(event_type='game_over' and int_params['game_round'] = 3) as round_3,
countIf(event_type='game_over' and int_params['game_round'] = 4) as round_4,
countIf(event_type='game_over' and int_params['game_round'] = 5) as round_5,
countIf(event_type='game_over' and int_params['game_round'] = 6) as round_6,
countIf(event_type='game_over' and int_params['game_round'] = 7) as round_7,
countIf(event_type='game_over' and int_params['game_round'] = 8) as round_8,
countIf(event_type='game_over' and int_params['game_round'] = 9) as round_9,
countIf(event_type='game_over' and int_params['game_round'] = 10) as round_10,
countIf(event_type='game_over' and int_params['game_round'] = 11) as round_11,
countIf(event_type='game_over' and int_params['game_round'] = 12) as round_12,
countIf(event_type='game_over' and int_params['game_round'] = 13) as round_13,
countIf(event_type='game_over' and int_params['game_round'] = 14) as round_14,
countIf(event_type='game_over' and int_params['game_round'] = 15) as round_15,
countIf(event_type='game_over' and int_params['game_round'] = 16) as round_16,
countIf(event_type='game_over' and int_params['game_round'] = 17) as round_17,
countIf(event_type='game_over' and int_params['game_round'] = 18) as round_18,
countIf(event_type='game_over' and int_params['game_round'] = 19) as round_19,
countIf(event_type='game_over' and int_params['game_round'] = 20) as round_20
from sfi.event
where `event_date` between '2022-04-01' and '2022-04-13' -- 此处修改日期
and user_group = 'B' -- 次数修改分组
group by `event_date`
```

### 留存时常漏斗

```
select
dt,
count() as new_player,
countIf(minutes >= 1) as `1 min`,
countIf(minutes >= 2) as `2 min`,
countIf(minutes >= 3) as `3 min`,
countIf(minutes >= 30) as `30 min`
from
(
with toDate('2022-04-18') as start_date, toDate('2022-04-27') as end_date
select player_id, min(create_date) as dt, countIf(event_type='heartbeat') / 2 as minutes from sfi.event
where event_date between start_date and end_date
and create_date between start_date and end_date
and user_group = 'A'
group by player_id
) t1 
group by dt
```

### 按分钟留存漏斗

```
select
dt,
count() as new_player,
countIf(minutes >= 1) as `1 min`,
countIf(minutes >= 2) as `2 min`,
countIf(minutes >= 3) as `3 min`,
countIf(minutes >= 30) as `30 min`
from
(
with toDate('2022-04-18') as start_date, toDate('2022-04-27') as end_date
select player_id, min(create_date) as dt, countIf(event_type='heartbeat') / 2 as minutes from sfi.event
where event_date between start_date and end_date
and create_date between start_date and end_date
and user_group = 'A'
group by player_id
) t1 
group by dt
```

### 查询玩家FPS分布

```
select fps,
count() as `玩家数`
from (
select player_id,
round(avg(int_params['game_fps'])) as fps 
from sfi.event
where event_type = 'game_over'
and event_date between '2022-04-01' and '2022-04-30'
group by player_id)
group by fps order by fps
```
