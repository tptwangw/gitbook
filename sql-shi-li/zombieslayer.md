# ZombieSlayer

### 常规数据（DNU, DAU, 活跃）

```
select
`pst_event_date` as `date`,
countIf(DISTINCT player_id, pst_create_date = pst_event_date) as `DNU`,
count(DISTINCT player_id) as `DAU`,
countIf(event_type='heartbeat') / 2 / `DAU` as `活跃时长`
from zs.event
where `pst_event_date` between '2022-04-23' and '2022-04-28' -- 此处修改日期
group by `pst_event_date`
```

### 留存

```
with toDate('2022-04-23') as start_date, toDate('2022-04-28') as end_date
select pst_create_date,
countIf(distinct player_id, pst_event_date = pst_create_date) retention_0,
countIf(distinct player_id, pst_event_date - pst_create_date = 1) / retention_0 retention_1,
countIf(distinct player_id, pst_event_date - pst_create_date = 2) / retention_0 retention_2,
countIf(distinct player_id, pst_event_date - pst_create_date = 3) / retention_0 retention_3,
countIf(distinct player_id, pst_event_date - pst_create_date = 4) / retention_0 retention_4,
countIf(distinct player_id, pst_event_date - pst_create_date = 5) / retention_0 retention_5,
countIf(distinct player_id, pst_event_date - pst_create_date = 6) / retention_0 retention_6,
countIf(distinct player_id, pst_event_date - pst_create_date = 7) / retention_0 retention_7,
countIf(distinct player_id, pst_event_date - pst_create_date = 14) / retention_0 retention_14,
countIf(distinct player_id, pst_event_date - pst_create_date = 30) / retention_0 retention_30
from zs.event where
pst_event_date between start_date and addDays(end_date, 30)
and pst_create_date between start_date and end_date
group by pst_create_date
```

### Guide漏斗

```
select
`pst_event_date` as `date`,
countIf(event_type='guide' and int_params['guide_id'] = 1) as guide_1,
countIf(event_type='guide' and int_params['guide_id'] = 2) as guide_2,
countIf(event_type='guide' and int_params['guide_id'] = 3) as guide_3,
countIf(event_type='guide' and int_params['guide_id'] = 4) as guide_4,
countIf(event_type='guide' and int_params['guide_id'] = 5) as guide_5,
countIf(event_type='guide' and int_params['guide_id'] = 6) as guide_6,
countIf(event_type='guide' and int_params['guide_id'] = 7) as guide_7
from zs.event
where `pst_event_date` between '2022-04-23' and '2022-04-28' -- 此处修改日期
and pst_event_date = pst_create_date
group by `pst_event_date`
```

### 关卡通过漏斗

```
select
`pst_event_date` as `date`,
countIf(event_type='stage_over' and int_params['stageId'] = 1 and int_params['is_win'] = 1) as stage_1,
countIf(event_type='stage_over' and int_params['stageId'] = 2 and int_params['is_win'] = 1) as stage_2,
countIf(event_type='stage_over' and int_params['stageId'] = 3 and int_params['is_win'] = 1) as stage_3,
countIf(event_type='stage_over' and int_params['stageId'] = 4 and int_params['is_win'] = 1) as stage_4,
countIf(event_type='stage_over' and int_params['stageId'] = 5 and int_params['is_win'] = 1) as stage_5,
countIf(event_type='stage_over' and int_params['stageId'] = 6 and int_params['is_win'] = 1) as stage_6,
countIf(event_type='stage_over' and int_params['stageId'] = 7 and int_params['is_win'] = 1) as stage_7,
countIf(event_type='stage_over' and int_params['stageId'] = 8 and int_params['is_win'] = 1) as stage_8,
countIf(event_type='stage_over' and int_params['stageId'] = 9 and int_params['is_win'] = 1) as stage_9,
countIf(event_type='stage_over' and int_params['stageId'] = 10 and int_params['is_win'] = 1) as stage_10,
countIf(event_type='stage_over' and int_params['stageId'] = 11 and int_params['is_win'] = 1) as stage_11,
countIf(event_type='stage_over' and int_params['stageId'] = 12 and int_params['is_win'] = 1) as stage_12,
countIf(event_type='stage_over' and int_params['stageId'] = 13 and int_params['is_win'] = 1) as stage_13,
countIf(event_type='stage_over' and int_params['stageId'] = 14 and int_params['is_win'] = 1) as stage_14,
countIf(event_type='stage_over' and int_params['stageId'] = 15 and int_params['is_win'] = 1) as stage_15,
countIf(event_type='stage_over' and int_params['stageId'] = 16 and int_params['is_win'] = 1) as stage_16,
countIf(event_type='stage_over' and int_params['stageId'] = 17 and int_params['is_win'] = 1) as stage_17,
countIf(event_type='stage_over' and int_params['stageId'] = 18 and int_params['is_win'] = 1) as stage_18,
countIf(event_type='stage_over' and int_params['stageId'] = 19 and int_params['is_win'] = 1) as stage_19,
countIf(event_type='stage_over' and int_params['stageId'] = 20 and int_params['is_win'] = 1) as stage_20
from zs.event
where `pst_event_date` between '2022-04-23' and '2022-04-28' -- 此处修改日期
group by `pst_event_date`
```
