---
title: "Calibrate Sequences PostgreSQL"
datePublished: Wed Jul 24 2024 07:59:05 GMT+0000 (Coordinated Universal Time)
cuid: clyzjztdm000909jq1ugbfu1q
slug: calibrate-sequences-postgresql
tags: postgresql, sql, query

---

# Sequences

Sequences in PostgreSQL are special kinds of database objects designed to generate unique numeric identifiers. These identifiers are often used as primary keys in tables. A sequence is essentially a counter that can be incremented or decremented, and it ensures that each value it generates is unique and not reused.

When you create a sequence, you can specify various parameters such as the starting value, increment step, minimum and maximum values, and whether the sequence should cycle when it reaches its limit. You can also control the caching behavior to improve performance.

To use a sequence, you can call the `nextval` function, which increments the sequence and returns the next value. The `currval` function returns the current value of the sequence, and the `setval` function allows you to set the sequence to a specific value.

Sequences are particularly useful in multi-user environments where multiple transactions might need to generate unique identifiers concurrently. By using sequences, PostgreSQL ensures that each transaction gets a unique identifier without conflicts or the need for complex locking mechanisms.

In summary, sequences in PostgreSQL provide a robust and efficient way to generate unique numeric identifiers, making them an essential tool for database design and management.

## Query for calibrate Sequences

```sql
DO $$
DECLARE
i TEXT;
BEGIN
  FOR i IN (
    SELECT 'SELECT SETVAL('
        || quote_literal(quote_ident(PGT.schemaname) || '.' || quote_ident(S.relname))
        || ', COALESCE(MAX(' ||quote_ident(C.attname)|| '), 1) ) FROM '
        || quote_ident(PGT.schemaname)|| '.'||quote_ident(T.relname)|| ';'
      FROM pg_class AS S,
           pg_depend AS D,
           pg_class AS T,
           pg_attribute AS C,
           pg_tables AS PGT
     WHERE S.relkind = 'S'
       AND S.oid = D.objid
       AND D.refobjid = T.oid
       AND D.refobjid = C.attrelid
       AND D.refobjsubid = C.attnum
       AND T.relname = PGT.tablename
  ) LOOP
      EXECUTE i;
  END LOOP;
END $$;
```

## Check

```sql
Select nextval(pg_get_serial_sequence('my_table', 'id')) as new_id;
```

src :

* [https://stackoverflow.com/questions/35734701/postgresql-change-all-sequences-with-for-loop](https://stackoverflow.com/questions/35734701/postgresql-change-all-sequences-with-for-loop)
    
* [https://stackoverflow.com/questions/18232714/postgresql-next-serial-value-in-a-table](https://stackoverflow.com/questions/18232714/postgresql-next-serial-value-in-a-table)