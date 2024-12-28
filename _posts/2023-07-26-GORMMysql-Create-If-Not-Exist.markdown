---
layout: post
thumbnail: ""
title: "[GORM][Mysql] Create If Not Exist"
createdAt: 2023-07-26 11:16:36.252000
updatedAt: 2023-07-26 11:16:36.252000
category: "Go"
---
자주 마추지는 상황이다.

- **데이터를 추가**하면서
- **PK가 중복**되는 경우는
    - **추가하지 않**으면서
    - **업데이트** or **무시할 때**

### Mysql

mysql에서는 다양한 방식으로 위 문제를 해결할 수 있다. 이 중에서 가장 효율적인 방법은 insert into 쿼리문에 on duplicate key clause를 추가하는 것이다.

- 중복되는 row를 특정 값으로 업데이트 할 때

``````sql
INSERT INTO table_name (``col1_name``, ``col2_name``, ``col2_name``, ...) 
		VALUES 
			(value11, value12, value13 ,....), 
			(value21, value22, value23 ,....)
    ON DUPLICATE KEY 
		UPDATE 
  	  col_name = special_value1,
      col_name = special_value2,
      col_name = special_value3
;
``````

- 전부 새로운 값으로 업데이트 할 때
``````sql
INSERT INTO table_name (``col1_name``, ``col2_name``, ``col2_name``, ...) 
		VALUES 
			(value11, value12, value13 ,....), 
			(value21, value22, value23 ,....)
    ON DUPLICATE KEY 
		UPDATE 
  	  col_name = VALUES(``col1_name``),
      col_name = VALUES(``col2_name``),
      col_name = VALUES(``col3_name``)
;
``````

### GORM

gorm에서는 OnConflict 객체를 db.Clauses에 추가하는 방식으로 기능을 구현하였다. ``DoNothing``, ``DoUpdates`` ,``UpdateAll`` 으로 원하는 방식을 결정한다.

``````go

type OnConflict struct {
    Columns      []Column
    Where        Where
    TargetWhere  Where
    OnConstraint string
    DoNothing    bool
    DoUpdates    Set
    UpdateAll    bool
}
``````

claused.OnConflict 객체를 생성하여 쿼리문에 넣어준다.

``````go
err := db.Clauses(clause.OnConflict{
		Columns:   []clause.Column{{Name: "PrimaryKeyName"}},
		UpdateAll: true,
	}).Create(&datas).Error
``````

## Reference

- [https://stackoverflow.com/questions/9537710/is-there-a-way-to-use-on-duplicate-key-to-update-all-that-i-wanted-to-insert](https://stackoverflow.com/questions/9537710/is-there-a-way-to-use-on-duplicate-key-to-update-all-that-i-wanted-to-insert)
- [https://stackoverflow.com/questions/39333102/how-to-create-or-update-a-record-with-gorm](https://stackoverflow.com/questions/39333102/how-to-create-or-update-a-record-with-gorm)
- [https://gorm.io/docs/sql_builder.html#Clauses](https://gorm.io/docs/sql_builder.html#Clauses)
