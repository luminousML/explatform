## 详细开发说明
本文档将分角色，按照输入输出定义开发需求

----------------
管理员：
1. 输入（公告内容，公告班级）向公告表中写入具体的公告，返回成功写入的条数（1|0）
2. 输入（教师号，教师名）|（课程号，课程名）|（学期名）|（学生号|学生名），创建对应实体
3. 输入（课程号）查询课程下所有班级的占用教室情况，返回（班级号，教室，时间）
4. 输入（教室号）查询教室在本学期内所有被占用情况，返回按照教室号**group by**的结果（教室号，班级号，时间）
5. 输入（学期）查询所有报告,返回实验记录表中符合条件的元组
6. *根据教务处**excel文件**，指定学期后，导入当学期所有班级*，按照班级号group by之后返回插入的数据

-----------------

课程：
> 课程包含了课程负责人这一身份账号与课程这一实体
1. 输入（公告内容，公告班级）向公告表中写入具体的公告，返回成功写入的条数（1|0）
2. 输入（课程号）查询该课程的所有班级情况，返回（班级号，班级人数，班级教室，上课时间）
3. 批准负责班级的请求（从申请表项中复制一份到负责表中）
4. 输入（课程号），返回代课申请（课程号，班级号，教室号，状态）
5. 输入（栏目，操作，选择题）生成实验（模板），返回添加记录数（1|0）

示例

栏目：
```
[
    {
        'name':"课前预习",
        "content"：
        "
            \### 课前预习
            本实验的目的是：
            实验器材
        ”
    }
]
```
操作：
```
[
    {"name':"操作一,"grade":"5"}，
    {"name':"操作二,"grade":"5"}
]
```
选择题：
```
[
    {
        "quetion":"absdafasdf",
        "answer"：[
            {'id':'a','content':'asdfasdfasf'},
            {'id':'b','content':'asdfasdfasf'}
        ],
        "right":"a"
    }
]
```
5. 输入（班级号）查询学生的签到情况

-----------------------

教师：
1. 查找所有可以负责的班级（负责表中没有该班级）
2. 输入（教师号，班级号）提交负责申请（负责表和班级表用右外连接连接）
3. 输入（教师号）返回教师所有负责的课程，返回（教师号，班级号，上课时间，上课教室，课程名称）
4. 输入（学生号，班级号）向自己负责的班级中添加学生
5. 输入（班级，[学号，实验进度]）查询班级实验记录里的元组
6. 输入（班级号，进度）返回签到记录（实验名，学生id，签到时间）
7. 输入（班级号，教室号，进度），将教室内的所有实验桌进度修改为（班级号-进度），并使上电机上电
8. 输入（班级号，教室号），将教室内所有实验桌解除占用
9. 根据学生group by学生的实验记录，返回一张大表，便于下一步生成excel

---------------------------

学生
1. 输入（学号，学期），返回学生参加的所有班级（学号，班级号，上课时间，上课教室，上课课程）
2. 输入（学号，班级号，进度），返回对应的实验记录，调出其中需要填写的部分|修改需要填写的部分
3. 输入（学号，班级号），查询所有实验记录，便于后续api层统计分数

------------------------
试验台
1. 输入（MAC地址，学号），根据试验台的进度记录，修改学生对应的实验记录的签到信息，修改实验桌的占用信息
2. 输入（进度，学号，操作名，分数），修改实验记录中的分数

要实现的api为
```
### 管理员
/manager/insert_billboard?content=&class=
{
    code:1,
    info:""
}
/manager/create_object?type=0|1|2|3&id=&name=
{
    code:1,
    info:""
}
/manager/new_term/
{
    code:1,
    info:""
}
/manager/check_subject?subject=
{
    code:1,
    info:
    [
        {
            "classroom":202,
            "subject":数字逻辑电路,
            "weekday":"5",
            "peroid":1
        }
    ]
}
/manager/check_classroom?classroom=
{
    code:1,
    info:
    [
        {
            "classroom":202,
            "subject":数字逻辑电路,
            "weekday":"5",
            "peroid":1
        }
    ]
}
/manager/get_report?term=
{
    code:1,
    info:
    [
        {
            "sid":"16041519",
            "sname":"姜佐腾",
            "subject":"数字逻辑电路",
            "eid":"xxxxxx",
            ...示例中的全部内容
        }
    ]
}

### 课程员
/subject/insert_billboard?content=&class=
{
    code:1,
    info:""
}

/subject/get_class/
{
    code:1,
    info:
    [
        {
            cid:10086,
            joins:28,
            classsroom:202,
            weekday:3,
            period:2
        }
    ]
}

/subject/get_request?class=
{
    code:1,
    info:
    [
        {
            tid:10086,
            tname:”郑老师“,
            class:10000,
            classsroom:202,
            weekday:3,
            period:2
        }
    ]
}

/subject/exec_request?class=&tid=
{
    code:1,
    info:""
}

/subject/put_template?id=&content=
{
    code:1,
    info:""
}

### 教师
/teacher/found_class/
{
    code:1,
    info:""
}

/teacher/put_class/tid=&class=
{
    code:1,
    info:""
}

/teacher/check_class/tid=
{
    code:1,
    info:
    [
        {
            "tid":10086,
            "class":10000,
            "weekday":6,
            "period":2,
            "classroom":202,
            "subject":"数字逻辑电路"
        }
    ]
}

/teacher/put_student/sid=&class=
{
    coded:1,
    info:""
}

/teacher/check_object/class=&sid=|exp=
{
    code:1,
    info:""
}

/teacher/sign_record/class=&exp=
{
    code:1,
    info:
    [
        {
            "subject":"数字逻辑电路实验"
            "sid":17041802
            "stime":201807211158
        }
    ]
}

/teacher/exp_revamp/class=&classroom=&exp=
{
    code:
    info:""
}

/teacher/rel_occupy/class=&classroom=
{
    code:1,
    info:""
}

/teacher/get_report?term=
{
    code:1,
    info:
    [
        {
            "sid":"16041519",
            "sname":"姜佐腾",
            "subject":"数字逻辑电路",
            "eid":"xxxxxx",
            ...示例中的全部内容
        }
    ]
}

### 学生
/student/check_class/sid=&term=
{
    code:1,
    info:
    [
        {
            "sid":17041802,
            "class":10000,
            "weekday":3,
            "period":2,
            "classroom":202,
            "subject":"数字逻辑电路"
        }
    ]
}

/student/re_record/sid=&class=&exp=
{
    code:
    info:""
}

/stduent/exp_recode/sid=&class=
{
    code:
    info:
    [
        {
            ...所有实验记录
        }
    ]
}