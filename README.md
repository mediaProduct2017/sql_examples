# sql_examples

[mysql](https://github.com/arfu2016/nlp/tree/master/nlp_models/mysql)

flat files on disk

durable storage, data structures for searching data

give the column a name:

    select 2+2 as sum
    
where restrictions (restriction for rows)
    
Related tables

The same value appears in different tables and different columns

joins: producing new talbes connecting a row from one table with a row from another table

Uniqueness and keys (such as id, primary key)

We need unique values to relate rows in one table to another

zipcode for uniqueness of an address

Joining tables: producing new tables

join two tables

    select animals.name, animals.species, diet.food
        from animals join diet
            on animals.species = diet.species
                where food = 'fish';
Join several tables

    select ordernames.name, count(*) as num
      from animals, taxonomy, ordernames
      where animals.species = taxonomy.name
        and taxonomy.t_order = ordernames.t_order
      group by ordernames.name
      order by num desc
      
    select ordernames.name, count(*) as num
      from (animals join taxonomy 
                    on animals.species = taxonomy.name)
                    as ani_tax
            join ordernames
                 on ani_tax.t_order = ordernames.t_order
      group by ordernames.name
      order by num desc
          
            
SQLite: sqlite3

PostgreSQL: psycopg2

ODBC: pydobc

mysql: MYsql.conector (or pymysql)

    import sqlite3
    
    conn = sqlite3.connect("cookies")
    
    cursor = conn.cursor()
    
    cursor.execute("select host_key from cookies limit 10")
    
    results = cursor.fetchall()
    
    print(results)
    
    cursor.close()
    
    conn.close()
    
    
    import sqlite3

    db = sqlite3.connect(dbname="testdb")
    # db = psycopg2.connect(dbname="forum")
    c = db.cursor()
    c.execute("insert into balloons values ('blue', 'water') ")
    db.commit()
    db.close()

Commit an insertion

atomicity: a transction happens as a whole or not at all

sql injection attack

    '); delete from posts; --
    
Use query parameters for insertion

    db.cursor.execute("insert into posts values (%s)", (content,))
    # cur.execute(SQL, data)
    
    cur.executemany(
                SQL_TEXT_TPL_SAVE, many_items
            )
            
Script injection attack

[Bleach](http://bleach.readthedocs.io/en/latest/)

Update 

    update table
        set column = 'value'
         where restriction
         # where cotent like "%spam%"
         
If you are familiar with regular expressions, think of the % in like patterns as being like the regex .* (dot star).

    delete from table
        where restriction
        
declare relationships

    create table sales (
        sku text references products(sku),
        sale_date date,
        count integer);
        
    create table sales (
        sku text references products,
        sale_date date,
        count integer);
        
sku in the sales table is a foreign key

self joins

    select a.id, b.id, a.building, a.room
       from residences as a, residences as b
         where a.building = b.building
           and a.room = b.room
           and a.id < b.id
         order by a.building, a.room;
         
Counting rows in a single table

    select count(*) from animals;
    
    select count(*) from animals where species = 'gorilla';
    
    select species, count(*) from animals group by species;
    
Things get a little more complicated if you want to count the results of a join

Suppose that we want to know how many times we have sold each product. In other words, for each sku value in the products table, we want to know the number of times it occurs in the sales table. We might start out with a query like this (a plain join, an inner join):

select from a joined table

    select products.name, products.sku, count(*) as num
      from products join sales
        on products.sku = sales.sku
      group by products.sku;
      
If a particular sku has never been sold — if there are no entries for it in the sales table — then this query will not return a row for it at all.

If we wanted to see a row with the number zero in it, we’ll be disappointed!

However, there is a way to get the database to give us a count with a zero in it. To do this, we’ll need to change two things about this query —

    select products.name, products.sku, count(sales.sku) as num
      from products left join sales
        on products.sku = sales.sku
      group by products.sku;
      
left join: 以左边为基准来记录行，而count是用sales.sku来算

This query will give us a row for every product in the products table, even the ones that have no sales in the sales table.

What’s changed? First, we’re using count(sales.sku) instead of count(*). This means that the database will count only rows where sales.sku is defined, instead of all rows.

Second, we’re using a left join instead of a plain join.

what’s a left (right) join?

SQL supports a number of variations on the theme of joins. The kind of join that you have seen earlier in this course is called an inner join, and it is the most common kind of join — so common that SQL doesn’t actually make us say "inner join" to do one.

But the second most common is the left join, and its mirror-image partner, the right join. The words “left” and “right” refer to the tables to the left and right of the join operator. (Above, the left table is products and the right table is sales.)

A regular (inner) join returns only those rows where the two tables have entries matching the join condition. A left join returns all those rows, plus the rows where the left table has an entry but the right table doesn’t. And a right join does the same but for the right table.

(Just as “join” is short for “inner join”, so too is “left join” actually short for “left outer join”. But SQL lets us just say “left join”, which is a lot less typing. So we’ll do that.)

inner join:

    select programs.name, count(*) as num
       from programs join bugs
         on programs.filename = bugs.filename
       group by programs.name
       order by num;
       
left outer join:

    select programs.name, count(bugs.filename) as num
       from programs left join bugs
         on programs.filename = bugs.filename
       group by programs.name
       order by num;
       
Something to watch out for: What do you put in the count aggregation? If you leave it as count(*) or use a column from the programs table, your query will count entries that don't have bugs as well as ones that do.

In order to correctly report a zero for programs that don't have any entries in the bugs table, you have to use a column from the bugs table as the argument to count.

For instance, count(bugs.filename) will work, and so will count(bugs.description).

One Query not two

    select avg(bigscore) from
        (select max(score) as bigscore
            from mooseball 
            group by team)
        as maxes;
        
In subquery, the name maxes is required in postgresql

Here are some sections in the PostgreSQL documentation that discuss other forms of subqueries:

[Scalar Subqueries](http://www.postgresql.org/docs/9.4/static/sql-expressions.html#SQL-SYNTAX-SCALAR-SUBQUERIES)

[Subquery Expressions](http://www.postgresql.org/docs/9.4/static/functions-subquery.html)

[The FROM clause](http://www.postgresql.org/docs/9.4/static/sql-select.html#SQL-FROM)

    def lightweights(cursor):
    """Returns a list of the players in the db whose weight is less than the average."""
    # cursor.execute("select avg(weight) as av from players;")
    # av = cursor.fetchall()[0][0]  # first column of first (and only) row
    cursor.execute("select name, weight from players, "
    "(select avg(weight) as av from players) as subq where weight < av")
    # the reult table of (select avg(weight) as av from players) is subq
    # join players with subq table, and then select
    return cursor.fetchall()
    
Views

A view is a select query stored in the database in a way that lets you use it like a table

You can use it repeatedly

    create view topfive as
    select species, count(*) as num
    from animals
    group by species
    order by num desc
    limit 5;

    select * from topfive;

[PostgreSQL create table documentation](http://www.postgresql.org/docs/9.4/static/sql-createtable.html)

There are a lot of restrictions that can be put on a column or a row. primary key and references are just two of them. See the "Examples" section of the [create table documentation](http://www.postgresql.org/docs/9.4/static/sql-createtable.html) for many, many more.

Rules for normalized tables:
1.Every row has the same number of columns.
2.There is a unique key and everything in a row says something about the key.
3.Facts that don't relate to the key belong in different tables.
4.Tables shouldn't imply relationships that don't exist.

The example here was the job_skills table, where a single row listed one of a person's technology skills (like 'Linux') and one of their language skills (like 'French'). This made it look like their Linux knowledge was specific to French, or vice versa ... when that isn't the case in the real world. Normalizing this involved splitting the tech skills and job skills into separate tables.

The serial type

For more detail on the serial type, take a look at the last section of this page in the [PostgreSQL manual](http://www.postgresql.org/docs/9.4/static/datatype-numeric.html).

[Intro to Relational Database -- deeper into sql](https://classroom.udacity.com/courses/ud197)

sql中也有in操作符，当然也可以用=来实现，只是需要多条sql语句。即使考虑网络的成本，认为一条和多条差别不大，in操作在算法复杂度上也是有优势的。假设待查询的字段的值一共是n个，in操作符后面的集合的元素个数是m个，如果用=来做的话，时间复杂度是mlogn，也就是逐个元素来看，然后查询B-tree。如果用in操作符，如果是一批一批集合中的元素来看，比较的时间复杂度是一样的，也是mlogn，估计sql也是这样实现的。如果自己来实现的话，对于in操作符，可以把集合中的元素放成类似python中的set或者dict，这样逐个扫描字段中的值，看在不在set中，时间复杂度是n，适用于m特别大的情况，可以通过数据结构set或者dict，实现以空间换时间。当然，如果n特别大，也可以在数据库的该字段使用hash index，而不用B-tree index，这样的话，也能实现以空间换时间，时间复杂度是m。


