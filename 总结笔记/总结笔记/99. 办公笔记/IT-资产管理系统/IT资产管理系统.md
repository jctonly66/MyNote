### 数据库信息

**数据库名：**db_assetmanagement

```
use db_assetmanagement;
```

#### 员工信息表

```
employee_id employee_name employee_type employee_company employee_department employee_level     employee_superiors employee_email office_location remark

# 创建员工信息表
create table if not exists employee (
	`employee_id` varchar(100) not null,# 员工编码
	`employee_name` varchar(100) not null,# 员工名称
	`employee_type` varchar(100) not null,# 员工性质
	`employee_company` varchar(100) not null,# 成本公司
	`employee_department` varchar(100) not null,# 部门路径
	`employee_level` varchar(100) not null,# 职位
	`employee_superiors` varchar(100),# 汇报上级
	`employee_email` varchar(100) not null,# 邮箱
	`office_location` varchar(100) not null, # 办公地点
	`remark` varchar(100), # 备注
	primary key(employee_id) # 主键
)engine=InnoDB DEFAULT CHARSET=utf8;
insert into employee values ('L00000', '管理员', 'Admin', 'Faraday Future', 'IT', 'Admin', '', 'ITSupport_CN@ff.com', '北京', '资产管理员');
```

#### 设备表 equipment

```
equipment_brand equipment_model equipment_type equipment_sn equipment_address put_in_date employee_id receive_date remark

# 创建语句
create table if not exists equipment (
	`equipment_brand` varchar(100) not null,# 设备品牌 
	`equipment_model` varchar(100) not null,# 设备型号
	`equipment_type` varchar(100) not null, # 设备类型
	`equipment_sn` varchar(100) not null,   # 设备SN号
	`equipment_address` varchar(100) not null, # 设备库存地址
	`put_in_date` DATETIME not null, # 入库时间
	`employee_id` varchar(100),	# 领用者ID
	`receive_date` DATETIME default '1970-01-01', # 领用日期
	`remark` varchar(100), # 备注
	primary key(equipment_sn), # 主键
	foreign key(employee_id) references employee(employee_id) # 外键
)engine=InnoDB DEFAULT CHARSET=utf8;
```

#### 日志表 record

```
id equipment_sn employee_id operator_date operator_type operator 

# 创建语句
create table if not exists record (
	`id` int unsigned auto_increment,# 自增ID
	`equipment_sn` varchar(100) not null,# 设备ID
	`employee_id` varchar(100) not null,# 员工ID
	`operator_date` DATETIME default '1970-01-01',# 操作日期
	`operator_type` varchar(10) not null, # 操作类型 0,入库; 1,变更
	`operator` varchar(100) not null, # 操作人
	primary key(id), # 主键
	foreign key(equipment_sn) references equipment(equipment_sn),# 外键
	foreign key(employee_id) references employee(employee_id) # 外键
)engine=InnoDB DEFAULT CHARSET=utf8;
```

### 接口

#### 入库

```
{
  "access":"add_Storage",
  "data":[
    {
      "equipment_brand":"惠普",
      "equipment_model":"G430",
      "equipment_type":"笔记本",
      "equipment_sn":"HP00001",
      "equipment_address":"北京",
      "employee_id":"",
      "receive_date":"",
      "remark":""
    },
    {
      "equipment_brand":"惠普",
      "equipment_model":"G430",
      "equipment_type":"笔记本",
      "equipment_sn":"HP00002",
      "equipment_address":"北京",
      "employee_id":"L00139",
      "receive_date":"",
      "remark":"测试"
    }]
}
```

#### 库存变更

```
{
	"access":"receive_equipment",
	"data":{
		"equipment_sn":"HP00001",
		"employee_id":"L00000"
		"remark":"fffffff"
	}
}
```

#### 库存查询

```
{
	"access":"query_Storage",
	"data":{
		"employee_id":"L00000"
	}
}
```

#### 日志查询

```
{
    "access":"query_record",
    "data":{
        "equipment_sn":"FSDJJDA3"
    }
}
```

#### 员工信息录入

```
{
    "access":"enter_employees",
    "data":{
        "employee_id":"L00139",
        "employee_name":"测试1",
		"employee_type":"用来测试1",
		"employee_company":"法法汽车1",
		"employee_department":"IT1",
		"employee_level":"测试生1",
		"employee_superiors":"没有人1",
		"employee_email":"cs1@ff.com",
		"office_location":"上海1"
    }
}
```

#### 获取某条员工信息

```
{
    "access":"getEmployeeInfoById",
    "data":{
        "employee_id":"L00139"
    }
}
```

















