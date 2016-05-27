# ORACLE PL SQL

## 基础

### 常用的用户命令

Oracle 在安装成功后，会默认生成三个用户

- sys 用户: 超级管理员 角色是dba 密码：change_on_install
- system 用户: 系统管理员，角色是dbaoper 密码: manager
- scott 用户: 普通用户，密码是tiger

```sql
    -- 创建新用户
    create user carl001 identified by 1111;
    -- 显示当前用户
    show user;
    -- 修改密码:
    password  scott;
    -- 删除用户
    drop user `[用户名]` `[cascade]`
    -- 赋予用户权限
    grant [permission] to [username]
    -- 收回用户权限
    revoke [permission] from [username] 
```

Oracle中，权限分为2种：
1.对于系统的权限
2.对于对象的权限

```sql
    grant create session to zhangsan;//授予zhangsan用户创建session的权限，即登陆权限
    grant unlimited tablespace to zhangsan;//授予zhangsan用户使用表空间的权限
    grant create table to zhangsan;//授予创建表的权限
    grant drop table to zhangsan;//授予删除表的权限
    grant insert table to zhangsan;//插入表的权限
    grant update table to zhangsan;//修改表的权限
    grant all to public;//这条比较重要，授予所有权限(all)给所有用户(public)
```

## PLSQL

在Oracle中的模块化程序在表现上分为4种：过程，函数，触发器，包

### 触发器是一种隐式的存储过程

1. 建立一个触发器, 当职工表 emp 表被删除一条记录时，把被删除记录写到职工表删除日志表中去。
```sql
CREATE TABLE emp_his AS SELECT * FROM EMP WHERE 1=2; 
CREATE OR REPLACE TRIGGER tr_del_emp 
   BEFORE DELETE --指定触发时机为删除操作前触发
   ON scott.emp 
   FOR EACH ROW   --说明创建的是行级触发器 
BEGIN
   --将修改前数据插入到日志记录表 del_emp ,以供监督使用。
   INSERT INTO emp_his(deptno , empno, ename , job ,mgr , sal , comm , hiredate )
       VALUES( :old.deptno, :old.empno, :old.ename , :old.job,:old.mgr, :old.sal, :old.comm, :old.hiredate );
END;
DELETE emp WHERE empno=7788;
DROP TABLE emp_his;
DROP TRIGGER del_emp;
```

2. 限制对Departments表修改（包括INSERT,DELETE,UPDATE）的时间范围，即不允许在非工作时间修改departments表。
```sql
CREATE OR REPLACE TRIGGER tr_dept_time
BEFORE INSERT OR DELETE OR UPDATE 
ON departments
BEGIN
 IF (TO_CHAR(sysdate,'DAY') IN ('星期六', '星期日')) OR (TO_CHAR(sysdate, 'HH24:MI') NOT BETWEEN '08:30' AND '18:00') THEN
     RAISE_APPLICATION_ERROR(-20001, '不是上班时间，不能修改departments表');
 END IF;
END;
```

3. 限定只对部门号为80的记录进行行触发器操作。

```sql
CREATE OR REPLACE TRIGGER tr_emp_sal_comm
BEFORE UPDATE OF salary, commission_pct
       OR DELETE
ON HR.employees
FOR EACH ROW
WHEN (old.department_id = 80)
BEGIN
 CASE
     WHEN UPDATING ('salary') THEN
        IF :NEW.salary < :old.salary THEN

           RAISE_APPLICATION_ERROR(-20001, '部门80的人员的工资不能降');
        END IF;
     WHEN UPDATING ('commission_pct') THEN

        IF :NEW.commission_pct < :old.commission_pct THEN
           RAISE_APPLICATION_ERROR(-20002, '部门80的人员的奖金不能降');
        END IF;
     WHEN DELETING THEN
          RAISE_APPLICATION_ERROR(-20003, '不能删除部门80的人员记录');
     END CASE;
END; 

/*
实例：
UPDATE employees SET salary = 8000 WHERE employee_id = 177;
DELETE FROM employees WHERE employee_id in (177,170);
*/
```

4. 利用行触发器实现级联更新。
在修改了主表regions中的region_id之后（AFTER），级联的、自动的更新子表countries表中原来在该地区的国家的region_id。

```sql
CREATE OR REPLACE TRIGGER tr_reg_cou
AFTER update OF region_id
ON regions
FOR EACH ROW
BEGIN
 DBMS_OUTPUT.PUT_LINE('旧的region_id值是'||:old.region_id
                  ||'、新的region_id值是'||:new.region_id);
 UPDATE countries SET region_id = :new.region_id
 WHERE region_id = :old.region_id;
END;
```

5. 在触发器中调用过程。

```sql
CREATE OR REPLACE PROCEDURE add_job_history
 ( p_emp_id          job_history.employee_id%type
   , p_start_date      job_history.start_date%type
  , p_end_date        job_history.end_date%type
   , p_job_id          job_history.job_id%type
   , p_department_id   job_history.department_id%type
   )
IS
BEGIN
 INSERT INTO job_history (employee_id, start_date, end_date,
                           job_id, department_id)
  VALUES(p_emp_id, p_start_date, p_end_date, p_job_id, p_department_id);
END add_job_history;

--创建触发器调用存储过程...
CREATE OR REPLACE TRIGGER update_job_history
 AFTER UPDATE OF job_id, department_id ON employees
 FOR EACH ROW
BEGIN
 add_job_history(:old.employee_id, :old.hire_date, sysdate,
                  :old.job_id, :old.department_id);
END;
```