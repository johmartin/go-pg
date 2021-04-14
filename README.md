# go-pg
GO언어를 위한 Postgresql DB 전용 ORM

### 왜 go-pg 인가?
- 가장 유명한 GO언어 ORM인 GORM의 경우 여러 DB르 동시에 지원하는 범용 ORM이라고 볼 수있음
- GORM의 경우 범용이다보니, 확장성은 좋으나 Postgresql만을 사용한다고 할때,
  아무래도 퍼포먼스가 go-pg에 비하여 떨어질 수 밖에 없음
- 퍼포먼스에 민감하다면 go-pg, 그냥 적당히 편하게 범용으로 사용하기 원한다면 gorm

### install & import
```
go get github.com/go-pg/pg/v10
```
```
import (
    "github.com/go-pg/pg/v10"
    "github.com/go-pg/pg/v10/orm"
)
```

### modeling examples
```
type Author struct {
    tableName struct{} `pg:"author,alias:ar"`
    Id int `pg:"id,pk"`
    Name string `pg:"name,type:varchar(20),notnull"`
    Gender string `pg:"gender,type:varchar(1),notnull"`
    Email string `pg:"email,type:varchar(100),notnull,unique"`
    Comment string `pg:"comment"`
    Books *[]Book `pg:"rel:has-many"`
}
type Book struct {
    tableName struct{} `pg:"book"`
    Id int `pg:"id,pk"`
    Name string `pg:"name,type:varchar(100),notnull"`
    AuthorID  int
    Author    Author `pg:"rel:has-one"`
    CreatedAt time.Time `pg:"default:now()"`
    UpdatedAt time.Time
}
```



### Connect to DB
```
db := pg.Connect(&pg.Options{
    Addr:     ":5432",
    User:     "user",
    Password: "pass",
    Database: "db_name",
})
defer db.Close()
```
or
```
opt, err := pg.ParseURL("postgres://user:pass@localhost:5432/db_name")
if err != nil {
   panic(err)
}

db := pg.Connect(opt)
defer db.Close()
```


### Create tables
```
err := createSchema(db)
if err != nil {
    panic(err)
}
```
```
func createSchema(db *pg.DB) error {
    models := []interface{}{
        (*Author)(nil),
        (*Book)(nil),
    }

    for _, model := range models {
        err := db.Model(model).CreateTable(&orm.CreateTableOptions{
            Temp: true,
        })
        if err != nil {
            return err
        }
    }
    return nil
}
```



### Select examples
```
// Select author by primary key.
author := &Author{Id: 1}
err = db.Model(author).WherePK().Select()
if err != nil {
    panic(err)
}
fmt.Println(author.id)
```
```
// Select all authors.
var authors []Author
err = db.Model(&authors).Select()
if err != nil {
    panic(err)
}
for _, author := range authors {
  fmt.Println(author.id)
}
```
```
// Select book and associated author in one query.
book := new(Book)
err = db.Model(book).
    Relation("Author").
    Where("book.id = ?", 1).
    Select()
if err != nil {
    panic(err)
}
fmt.Println(book)
```



### Insert examples
editting......





### Struct tags
|	Tag	|	Comment	|
|	---	|	---	|
|	tableName struct{} `pg:"table_name"`	|	Overrides default table name.	|
|	tableName struct{} `pg:"alias:table_alias"`	|	Overrides default table alias name.	|
|	tableName struct{} `pg:"select:view_name"`	|	Overrides table name for SELECT queries.	|
|	tableName struct{} `pg:",discard_unknown_columns"`	|	Silently discards uknown columns instead of returning an error.	|
|	pg:"-"	|	Ignores the field.	|
|	pg:"column_name"	|	Overrides default column name.	|
|	pg:"alias:alt_name"	|	Alternative column name. Useful when you are renaming the column.	|
|	pg:",pk"	|	Marks column as a primary key. Multiple primary keys are supported.	|
|	pg:",nopk"	|	Not a primary key. Useful for columns like id and uuid.	|
|	pg:"type:uuid"	|	Overrides default SQL type.	|
|	pg:"default:gen_random_uuid()"	|	SQL default value for the column. go-pg uses the DEFAULT placeholder and PostgreSQL replaces it with the provided expression.	|
|	pg:",notnull"	|	Adds NOT NULL SQL constraint.	|
|	pg:",unique"	|	Makes CreateTable to add an unique constraint.	|
|	pg:",unique:group_name"	|	Unique constraint for a group of columns.	|
|	pg:"on_delete:RESTRICT"	|	ON DELETE clause for foreign keys.	|
|	pg:",array"	|	Treats the column as a PostgreSQL array.	|
|	pg:",hstore"	|	Treats the column as a PostgreSQL hstore.	|
|	pg:"composite:type_name"	|	Treats the column as a PostgreSQL composite.	|
|	pg:",use_zero"	|	Disables marshaling Go zero values as SQL NULL.	|
|	pg:",json_use_number"	|	Uses json.Decoder.UseNumber to decode JSON.	|
|	pg:",msgpack"	|	Encodes/decodes data using MessagePack.	|
|	pg:"partition_by:RANGE (time)"	|	Specifies table partitioning for CreateTable.	|
|	DeletedAt time.Time `pg:",soft_delete"`	|	Enables soft delete.	|![image](https://user-images.githubusercontent.com/82506516/114711913-78942100-9d6a-11eb-90ee-782aae792df5.png)





### Go type & Postgresql type
|	Go type	|	PostgreSQL type	|
|	---	|	---	|
|	int8, uint8, int16	|	smallint	|
|	uint16, int32	|	integer	|
|	uint32, int64, int	|	bigint	|
|	uint, uint64	|	bigint	|
|	float32	|	real	|
|	float64	|	double precision	|
|	bool	|	boolean	|
|	string	|	text	|
|	[]byte	|	bytea	|
|	struct, map, array	|	jsonb	|
|	time.Time	|	timestamptz	|
|	net.IP	|	inet	|
|	net.IPNet	|	cidr	|
|	pg:",unique:group_name"	|	Unique constraint for a group of columns.	|
|	pg:"on_delete:RESTRICT"	|	ON DELETE clause for foreign keys.	|
|	pg:",array"	|	Treats the column as a PostgreSQL array.	|
|	pg:",hstore"	|	Treats the column as a PostgreSQL hstore.	|
|	pg:"composite:type_name"	|	Treats the column as a PostgreSQL composite.	|
|	pg:",use_zero"	|	Disables marshaling Go zero values as SQL NULL.	|
|	pg:",json_use_number"	|	Uses json.Decoder.UseNumber to decode JSON.	|
|	pg:",msgpack"	|	Encodes/decodes data using MessagePack.	|
|	pg:"partition_by:RANGE (time)"	|	Specifies table partitioning for CreateTable.	|
|	DeletedAt time.Time `pg:",soft_delete"`	|	Enables soft delete.	| 


