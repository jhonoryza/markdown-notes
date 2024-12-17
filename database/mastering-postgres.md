# Mastering Postgres

## Data Type

### Integer

- there is no `unsigned` all is `signed`
- very fast

```sql
smallint --32,768
int2 --32,768

integer --2,147,483,648
int4 --2,147,483,648

big integer --9,223,372,036,854,775,807
int8 --9,223,372,036,854,775,807
```

### Numeric

- support fraction
- very slow
- very accurate
- unlimited size or predefined

`::numeric(precision, scale)`

- precision is total digit
- scale is total digit after comma

```sql
select 12.345::numeric -- 12.345
select 12.345::numeric(5, 3) -- 12.345
select 12.345::numeric(5, 2) -- 12.35
```

### Floating Point

- support fraction
- very fast
- not very accurate (approximation)

```sql
real -- 4bytes 1E-37 - 1E37
float4 -- 4bytes 1E-37 - 1E37
-- float (1-24) -> transform to real

double -- 8bytes 1E-307 - 1E307
float8 -- 8bytes 1E-307 - 1E307
-- float (25-53) -> transform to double
```

approximation

```sql
select 7.0::float8 * 0.2 = 1.4 -- FALSE
select 7.0::float8 * 0.2 - 1.4 < .001 -- TRUE
```

### Money

```sql
create table money_example (
    item text,
    price money
);

insert into money_example (item, price) VALUES
('Laptop', 1999.99),
('Smartphone', 799),
('Pen', .25),
('Smartwatch', '$199.99');
```

```sql
select * from money_example;
```

| item       | price     |
| ---------- | --------- |
| Laptop     | $1,999.99 |
| Smartphone | $799.00   |
| Pen        | $0.25     |
| Smartwatch | $199.99   |

default currency is dollar

```sql
show lc_monetary;
```

set currency to poundsterling

```sql
set lc_monetary = 'en_GB.UTF-8';
```

### NaN and Infinity

```sql
select `NaN`::numeric --Nan
select `NaN`::float8 --Nan
select `NaN`::integer --Error
select `inf`::integer --Error
select `inf`::numeric --Infynity
select `inf`::float8 --Infynity
select `Infynity`::numeric --Infynity
```

### Speed between Float and Numeric

generate a series

```sql
select * from generate_series(1, 20) num
```

lets see the speed difference

```sql
select
    sum(num :: float8 / (num + 1 )) :: float8
from
    generate_series(1, 20000000) num;
```

run in 2.3s

```sql
select
    sum(num :: numeric / (num + 1 )) :: numeric
from
    generate_series(1, 20000000) num;
```

run in 4.5s

### Cast

```sql
select 100::int8;
select cast(100 as int8);

select
pg_typeof(100::int8),
pg_typeof(cast(100 as int8))
```

decorator literal

```sql
select integer '100' -- 100
select pg_typeof(integer '100') -- Error
```

size of bytes a column is occupied

```sql
select pg_column_size(100::int2)
```

### Character

recommend way just using `text` data type

behind the hood this type using same data structure:

```sql
char(5) -- fixed length
character varying -- char varying
text
```

`char` is an alias from `character`

no performance gained between using this 3 different type

```sql
create table char_example (
    abbr char(5)
);

insert into
    char_example (abbr)
values
    ('abcde');

select * from char_example;
```

### Check Constraint

enforcing data integrity you can use constraint

```sql
create table check_example (
    price numeric, -- must be > 0
    abbr text -- must be === 5 char
);
```

lets add the constrain

```sql
create table check_example (
    price numeric check (price > 0),
    abbr text check (length(abbr) = 5)
);

insert into check_example (price, abbr) values (-1, 'foo'); -- Error

insert into check_example (price, abbr) values (1, 'fooooo'); -- Error

insert into check_example (price, abbr) values (1, 'foo'); -- Ok
```

lets make custom error `price_must_be_positive`

```sql
create table check_example (
    price numeric constraint price_must_be_positive check (price > 0),
    abbr text check (length(abbr) = 5)
);
```

alternate syntax

```sql
create table check_example (
    price numeric check (price > 0),
    discount_price numberic check (discount_price > 0)
    abbr text,
    check (length(abbr) = 5),
    check (price > discount_price)
);

insert into check_example (price, abbr) values (10, 9, 'foo'); -- Ok
insert into check_example (price, abbr) values (5, 9, 'foo'); -- Error
```

### Domain Types

Domain is a custom data type

it will wraps up the `data type` and `constraint` in a `custom name`

let say we have column postal: `01234-2345`

```sql
create domain us_postal_code as text constraint format check (
    value ~ '^\d{5}$' or value ~ '\d{5}-\d{4}$'
);

create table domain_example (
    street text not null,
    city text not null,
    postal us_postal_code not null
);

insert into domain_example (street, city, postal)
values ('main', 'dallas', '7432'); -- Error
insert into domain_example (street, city, postal)
values ('main', 'dallas', '74320'); -- Ok
insert into domain_example (street, city, postal)
values ('main', 'dallas', '74320-4999'); -- Ok
```

### Chars and Collation

default encoding UTF8

Encoding is a set defined for a legal character

example, maybe an emoji is not valid for a specific encoding

default collation en_US.UTF-8

Collation is a set of rules that defines how does a character relate to each
other

example, comparing char e with e'

to show client encoding use this command :

```sql
show client_encoding;
```

comparing collation

```sql
select 'abc' = 'ABC' COLLATE "en_US.UTF-8" as result; -- False
```
