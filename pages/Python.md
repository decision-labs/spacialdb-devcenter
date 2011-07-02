# Connecting to SpacialDB from Python

To connect to SpacialDB from your python framework, use [[Psycopg|http://initd.org/psycopg/]] the most popular PostgreSQL adaptor for Python. Check the documentation for information on how to [[install|http://initd.org/psycopg/install/]] this for your system. Psycopg can also be installed from the PyPI by:

```console
$ easy_install psycopg2
```

## Basic usage

The basic usage is common to all database adapters, requiring a connection string:

```python
import psycopg2

conn = psycopg2.connect("dbname=test user=postgres")
```

## Retrieve some data

```python
# Open a cursor to perform database operations
cur = conn.cursor()

# Get the time
cur.execute("SELECT NOW() AS when;")

# Fetch the next row of the query
result = cur.fetchone()

# Do something fancy with it
print result[0]

# Close cursor and connection
cur.close()
conn.close()
```
