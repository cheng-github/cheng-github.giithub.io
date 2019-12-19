# 会议室座位配置字段

## seat_layout_config

字段名称            | 字段类型     | 含义                         | 备注
--------------- | -------- | -------------------------- | --
_id             | ObjectId | 标识数据唯一
rows            | Number   | 表示会议室的座位行数
columns         | Number   | 表示会议室座位的列数
rostrum_seats   | Number   | 主席台座位数目
rostrum_layout  | String   | center、left、right表示居中居左和居右
rostrum_seatinfo  |  Array | 存储主席台座位信息  
normal_seatinfo | Array    | 座位区域座位的信息

### rostrun_seatinfo

字段名称         | 字段类型   | 含义                              | 备注
------------ | ------ | ------------------------------- | --
index    | Number | 标识座位在布局中的行号，从1开始
label        | String | 表示该座位的标签，如1-1等..用户自定义内容

### normal_seatinfo

字段名称         | 字段类型   | 含义                              | 备注
------------ | ------ | ------------------------------- | --
row_index    | Number | 标识座位在布局中的行号，从1开始
column_index | Number | 标识座位在布局中的列号，从1开始
label        | String | 表示该座位的标签，如1-1等..用户自定义内容
seat_type    | String | 表示座位类型，manager为高管，employee为普通座位
