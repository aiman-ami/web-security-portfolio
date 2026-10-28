# Day 20: SQL Injection, UNION Attacks Continued

- Lab: SQL injection attack, listing the database contents on Oracle, solved
- Lab: SQL injection attack, listing the database contents on non-Oracle databases, solved
- Lab: SQL injection UNION attack, determining the number of columns returned by the query, solved
- Lab: SQL injection UNION attack, finding a column containing text, solved
- Lab: SQL injection UNION attack, retrieving data from other tables, solved
- Lab: SQL injection UNION attack, retrieving multiple values in a single column, solved

Roadmap: 2C SQL injection, UNION attacks group complete.

## Writeups

**Listing database contents on Oracle**
```
Bug: category parameter injectable, no input sanitization
Payload: ' union select USERNAME_TCJHHW,PASSWORD_LDKLVF from USERS_ULIPXA--
Why it worked: Oracle allows UNION SELECT to pull from any table once you control the
query structure. Used all_tables then all_tab_columns to find the right table and
column names before extracting the actual credentials.
```

**Listing database contents on non-Oracle databases**
```
Bug: category parameter injectable, same root cause as the Oracle lab
Payload: ' union select username_xxgtnt,password_kbufcf from users_ihtcnu--
Why it worked: non-Oracle databases use information_schema.tables and
information_schema.columns instead of Oracle's all_tables and all_tab_columns.
Found a custom table by spotting the one name that was not a pg_ system table,
then pulled its columns and the credentials the same way as the Oracle lab.
```

**UNION attack, determining the number of columns**
```
Bug: category parameter injectable
Payload: Gifts' union select NULL,NULL,NULL--
Why it worked: a UNION SELECT must return the same number of columns as the
original query. Adding NULLs until the page loads normally instead of erroring
finds that number. This lab's query needed 3.
```

**UNION attack, finding a column containing text**
```
Bug: category parameter injectable
Payload: ' union select NULL,'7xv5SS',NULL--
Why it worked: not every column in a UNION SELECT accepts text, some only accept
numbers. Testing a string in each position shows which column can carry text data,
needed before extracting real string data like usernames.
```

**UNION attack, retrieving data from other tables**
```
Bug: category parameter injectable
Payload: ' union select username,password from users--
Why it worked: same technique as the Oracle and Postgres database contents labs,
except the table and column names (users, username, password) were given directly
in the lab description instead of needing to be found first.
```

**UNION attack, retrieving multiple values in a single column**
```
Bug: category parameter injectable, only one column in the query accepts text
Payload: ' union select NULL,username||'~'||password from users--
Why it worked: with only one text-compatible column available, username and
password cannot go in separate columns like the previous lab. Concatenating them
into one string with || lets both values travel through the single usable column.
```

## Key takeaways
- Oracle uses all_tables / all_tab_columns, other databases use information_schema.tables / information_schema.columns, to find table and column names
- pg_ prefixed tables are PostgreSQL system tables, not application data, a useful signal for identifying both the database type and the real target table
- Column count errors go opposite directions depending on the method: UNION SELECT NULL fails until the count is right, ORDER BY succeeds until the count is exceeded
- A lab can state an exact value to retrieve, check the lab page itself, not just whether the injection technique works in general
- When only one column accepts text, concatenate values into it with || (Oracle, PostgreSQL) or CONCAT() (MySQL)
- A fresh request after a broken one (stream errors, blank responses) is often faster than debugging encoding by hand
