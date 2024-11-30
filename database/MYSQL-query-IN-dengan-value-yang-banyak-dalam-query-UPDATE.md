Menggunakan klausa `IN` dengan sejumlah besar nilai dalam query UPDATE juga bisa
menyebabkan masalah kinerja. Namun, ada beberapa cara untuk mengatasi masalah
ini dan memastikan bahwa update dilakukan dengan efisien.

### Contoh Basic Update dengan `IN`

```sql
UPDATE my_table
SET column_name = 'new_value'
WHERE id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
```

### Praktik Terbaik

1. **Batching**: Pertimbangkan untuk membagi update menjadi batch yang lebih
   kecil jika jumlah nilai dalam klausa `IN` sangat besar.

   ```sql
   -- Batch 1
   UPDATE my_table
   SET column_name = 'new_value'
   WHERE id IN (1, 2, 3, 4, 5);

   -- Batch 2
   UPDATE my_table
   SET column_name = 'new_value'
   WHERE id IN (6, 7, 8, 9, 10);
   ```

2. **Penggunaan Join**: Jika nilai-nilai tersebut berasal dari tabel lain,
   menggunakan join bisa menjadi solusi yang lebih efisien.

   ```sql
   UPDATE my_table a
   JOIN other_table b ON a.id = b.id
   SET a.column_name = 'new_value'
   WHERE b.some_column IN (...);
   ```

3. **Temporary Tables**: Anda juga bisa menggunakan tabel sementara untuk
   menyimpan nilai-nilai yang akan di-update dan kemudian melakukan join dengan
   tabel tersebut.

   ```sql
   CREATE TEMPORARY TABLE temp_ids (id INT);

   INSERT INTO temp_ids (id) VALUES (1), (2), (3), (4), (5), (6), (7), (8), (9), (10);

   UPDATE my_table a
   JOIN temp_ids t ON a.id = t.id
   SET a.column_name = 'new_value';
   ```

4. **Stored Procedure**: Anda juga bisa menggunakan stored procedure untuk
   mengelola update dalam batch.

   ```sql
   DELIMITER //

   CREATE PROCEDURE batch_update()
   BEGIN
       DECLARE done INT DEFAULT 0;
       DECLARE start INT DEFAULT 0;
       DECLARE batch_size INT DEFAULT 100;

       DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

       REPEAT
           START TRANSACTION;

           UPDATE my_table
           SET column_name = 'new_value'
           WHERE id IN (SELECT id FROM some_source_table LIMIT start, batch_size);

           SET start = start + batch_size;

           COMMIT;
       UNTIL done END REPEAT;
   END //

   DELIMITER ;

   CALL batch_update();
   ```

Dengan menggunakan teknik-teknik ini, Anda bisa melakukan update dengan lebih
efisien meskipun jumlah nilai dalam klausa `IN` sangat besar.

### Contoh subquery pada table yang sama

```sql
UPDATE users 
SET remember_token = 'bs' 
WHERE id IN (
	SELECT id FROM (
	   SELECT id from users WHERE id != 1
	) as subquery
);

UPDATE users
JOIN (
	SELECT id from users WHERE id != 1
) as subquery
on users.id = subquery.id
SET remember_token = 'ws';
```
