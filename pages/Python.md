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

