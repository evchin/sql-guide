# Table Guide

*Very important: The guide below is **one** path out of hundreds. It's just the path I personally took, and what made sense to me. It is **very** probable that you can think of a different way of going about things that make more sense to you. Please take those liberties and trust your intelligence.*

### What is Table?

Table is exactly what it sounds like: a table of data. 

### Header File

Here is the header file with all of the *required* functions (ie. you may have/need more private functions), and what they should do. Methods marked optional are not required methods, but rather helper methods that I thought would be good to have.

```cpp
class Table
{
public:
    // PURPOSE: default table ctor. should honestly never be used.
    Table();
    // PURPOSE: table constructor that takes in a previously created table's name, and reconstructs it
    Table(string table);
    // PURPOSE: table constructor that creates a brand new table called `table` with fields of `fields`
    Table(string table, vectorstr fields);
    // (OPTIONAL) PURPOSE: reconstructs the table named `table`, and creates a new table that only contains `recnos` 
    Table(string table, vector<long> recnos);
    // PURPOSE: insert a vector of data into the table
    void insert_into(vectorstr v);
    // PURPOSE: select the `fields` in your table where `field op value` (ex. age = 90). return these values as a new table
    Table select(vectorstr fields, string field, string op, string value);
    // PURPOSE: select the `fields` in your table that fits the postfix condition within your queue of tokens. return these values as a new table.
    Table select(vectorstr fields, Queue<Token*> q);
    // PURPOSE: select the `fields` in your table that fits the infix condition. return these values as a new table.
    Table select(vectorstr fields, vectorstr conditions);
    // PURPOSE: return a new copy of table
    Table select_all();
    // (OPTIONAL) PURPOSE: select `fields` within the table and return a new table containing only those fields. 
    Table select_fields(vectorstr fields);
    // PURPOSE: return select_recnos
    vector<long> select_recnos();
    // PURPOSE: return field_names vector
    vectorstr get_field_names();
    // PURPOSE: print
    friend ostream& operator<<(ostream& os, const Table& t);
private:
    // (OPTIONAL) PURPOSE: takes an infix condition and returns it as a postfix queue
    Queue<Token*> infix_to_postfix(vectorstr infix);
    // (OPTIONAL) PURPOSE: reads field names of the table from external file and returns them as a vectorstr
    vectorstr read_fields(string table);

    // PURPOSE: name of the table
    string name;
    // PURPOSE: vectorstr of the table's fields
    vectorstr field_names;
    // PURPOSE: a vector of rec numbers for testing purposes only. serves no real functional purpose.
    vector<long> _select_recnos;
    // PURPOSE: a vector of mmaps for each field within the table. in each map, there are maps from values to recnos
    // for example, in the "name" map, there is an index for {Joe: 2, 3, 6}
    // that means that rec numbers 2, 3, and 6 all have `Joe` as a value for the `name` field 
    vector<MMap<string, long> > indices;
    // PURPOSE: maps field names to their indices within the `indices` vector.
    Map<string, int> field_name_indices;
};
```

A lot of that might have just gone over your head. No worries, we'll go over them in detail.


### Thought Process

You might be thinking why we *need* all of these methods, and that would be an excellent question. We'll start at the very beginning - why does this table class exist? Well, we want to:

1. Create a table 
2. Insert data into it
3. Access data using conditions
4. Rebuild tables even *after* our program ends

That's all - we want to be able to **create**, **insert**, **access**, and **rebuild**. `CIAR` if you will. It's important to keep these four main functionalities in mind as we go through each method.


### Private Members

Really quickly, before we go over the methods, let's make sure we understand our private members and why we need them.

1. `string name`: table name. pretty straightforward.
2. `vectorstr field_names`: a vector of field names. Ex: `{name, age, occupation}`
3. `vector<long> _select_recnos`: a vector of rec numbers for testing purposes only. serves no real functional purpose
4. `vector<MMap<string, long> > indices`: a vector of mmaps that correspond to the table's field names. 
    - For example, if our field names were `{name, age, occupation}`, we would have a vector of three mmaps, where the mmap at the 0th index corresponds to the field `name`, the 1st index with the field `age`, etc. Each mmap contains keys of field values that map to record numbers. For example, in the `name` map, say one of the keys `Joe` looks like `{Joe: 2, 3, 6}`. That means that rec numbers 2, 3, and 6 all have names of `Joe`. 
5. `Map<string, int> field_name_indices`: a map from field names to their indices. Using the above example, `name` would map to 0, `age` would map to 1, and `occupation` would map to 2. 

Ok, now we can go over methods. Keep in mind that I'll be going over these methods in a linear fashion - but your development does *not* have to be linear. In fact, ideally, you'd have more time to jump around and get a sense of what's needed yourself. 

### `Table()`

We don't care about this one. This should never be called.

### `Table(string table)`

Ok, *now* we're getting somewhere. 

**Purpose**: Reconstruct a previously built table that was named the parameter variable `table`.

How do we even *begin* to do that? Well, let's start by assuming that the rest of our class has been built perfectly.

What are the essential things we need to rebuild our table? Think about this and write down your thoughts before moving on.

Here's my list:

1. Field Names
2. Records

Huh, that's a pretty short list right? But you'll find that you really *can* rebuild a table from scratch, as long as you can find these pieces of information.

Okay, how do we get these pieces of information? Well, if you think back to the `binary_files` project, you remember we were just taking things and storing them into files. There's no reason we can't do the same thing here, right? So, in keeping with the assumption that the rest of our class has been built perfectly, let's assume that when we created our table, we also created two files along with that named `tablename_fields.bin` and `tablename.bin`. `tablename_fields.bin` should contain the field names for the table, and `tablename.bin` should contain the records for the table. All we'll need to do here is read off the data and and rebuild our private data structures.

In summary, this method should:

1. Set private member `name` to parameter `table`.
2. Read field names from `tablename_fields.bin`. Use this info to set up `indices` and `field_name_indices`.
3. Read records from `tablename.bin`. Insert each field value from each record into the correct `indices` map and key.
4. Close your files!

### `Table(string table, vectorstr fields)`

**PURPOSE**: Create a brand new table called `table` with fields of `fields`.

Remember CIAR - **create**, **insert**, **access**, and **rebuild**. When we create our table, we want to make sure that it's *also* insertable, accessible, and rebuildable. Well, if we think back to the constructor above we just went over, remember that we made some assumptions about what needed to exist before rebuilding our table. We needed two files: `tablename_fields.bin` and `tablename.bin`. Okay, that's a start - let's make sure we properly create these files when we create our table. And for now, since we haven't attempted writing our `insert_into` and `select` functions, we'll leave it at that for now. We may have to come back later and adjust things once we're more familiar with those methods.

In summary:

1. Set up your data structures (`name`, `field_names`, `indices`, `field_name_indices`)
2. Create the two necessary binary files.
3. Write field names into the fields file. Since there's no data to store yet, we can leave the normal bin file empty for now.
4. Close files!

### `Table(string table, vector<long> recnos)`

**PURPOSE**: Reconstruct the previously created table named `table`, except store only the recnos specified in `recnos`.

Ok, this sounds pretty similar to `Table(string table)`, except now, we just want to read only *certain* records from `tablename.bin` now, and we want to create a brand new table instead of just rebuilding the old one.

In summary, this method should:

1. Create a unique `select_table` name and set it to private member `name`.
2. Read field names from `tablename_fields.bin`. Use this info to set up `indices` and `field_name_indices`.
3. Read only *certain* records from `tablename.bin`. Insert each field value from each record into the correct `indices` map and key.
4. Close your files!

### `void insert_into(vectorstr v)`

**PURPOSE**: Insert a vector of data into the table.

A vector of data would look like `[Emily, 19, firefighter]`. Remember the files that come with each table - the fields file might stay the same, but your records file will not. If you're inserting new data into the table, you'll also probably want to write it into your file as well.

1. Create a FileRecord with the vector of data.
2. Write it to `tablename.bin`, *without* overwriting any of the existing data.
3. Don't forget to also insert the data values into your `indices` structure.
4. Close files!

### `Table select(vectorstr fields, string field, string op, string value)`

**PURPOSE**: Create and return a new table with only select field names of `fields`, and only with records where `field op value` (ex. age = 90).

Ok, this sounds a little complex. Let's break it down.

1. Identify the record numbers that fit the given condition description, and set them to `_select_recnos`. Hint: this is where your `indices` data structure as well as the linked list feature of your bplustree becomes incredibly helpful! 
2. Create a new table with a *unique* select table name, with *only* the fields that were specified in the parameters. Keep in mind - these fields may be *out of order*. This can be *very* bad mews when you're inserting/writing data into your files and mmaps. Be sure to handle this properly.
3. Remember `_select_recnos`? In this newly created table, read each record number from the *original* table file and insert it into to the *new* table. Remember that you might not need all of the field values, and that they should also be in the correct order!
4. Close files.
5. Return new table. 

### `Table select(vectorstr fields, Queue<Token*> q)`

**PURPOSE**: Create and return a new table with only select field names of `fields`, and only with records that fit the postfix condition specified in the queue. 

An example of a postfix condition is `3 4 * 2 5 * +`. This is equal to the infix condition `(3 * 4) + (2 * 5)`. In the context of SQL, our postfix condition could look something like `age 19 > name Emily = and`, which would be equivalent to `age > 19 and name = Emily`. There are never any parentheses in a postfix condition, because there is never any room for misunderstanding, unlike infix conditions.

To select record numbers that fit this condition, you'll want to gradually filter out your table by creating new tables with each condition your encounter. So as you pop values from your queue, you'll want to keep track of when you have a realized condition that you can use to filter out a new table.

Hints: 
1. Use if else statements to see what type of token something is, which should help guide you on what to do with it. 
2. Stacks will be helpful to temporarily store data as you sort through your postfix condition!
3. Make full utility of the `Table select(vectorstr fields, string field, string op, string value)` function.

Summary:

1. Instantiate stack structure(s) that you think may help you sort through the postfix condition.
2. Pop each value from the queue and use their token types to guide you on what to do next.
3. Once you reach your final table, remember to filter out unnecessary fiels that you don't want to include. (I created a new function `Table select_fields(vectorstr fields)` to help me out with this part - I talk more about it below.)
4. Return the new table.

Please carefully test this function as much as possible!

### `Table select(vectorstr fields, vectorstr conditions)`

**PURPOSE**: Create and return a new table with only select field names of `fields`, and only with records that fit the infix condition specified in the vectorstr. 

This is surprisingly easy once you have your `Table select(vectorstr fields, Queue<Token*> q)` function written. All you'll need to do is convert your infix expression to a postfix expression, and send it over to the previous method. There are many resources online to help out with this conversion.

### `Table select_all()`

**PURPOSE**: Create and return a new copy of table.

Also a nice and straightforward function. Remember to copy the necessary files with a new unique select table name, and reconstruct your data structures.

### `Table select_fields(vectorstr fields)`

**PURPOSE**: Create and return a new table with only select field names of `fields`. 

Keep in mind this can be tricky if the fields are out of order (ex. `{age, name}` when the original field names were `{name, age, occupation}`). This is where our `field_name_indices` data structure really shines :)

1. Since we're creating a new table, create a unique select table name.
2. Use `field_name_indices` to filter out unnecessary indices.
3. Read records from file, filter out unnecessary fields, and insert the new vector of data into the new table.
4. Close your files.
5. Return the table.

### `Queue<Token*> infix_to_postfix(vectorstr infix)`

**PURPOSE**: Convert an infix condition into a postfix condition, and return it as a postfix queue.

This method can be surprisingly complex! Be *very, very* careful. Test as much as possible.

Here are some resources that can help:
1. [3. Infix to Postfix Conversion The Easy Way](https://www.youtube.com/watch?v=vXPL6UavUeA&ab_channel=YaarPadhaDe)
2. [3.6 Infix to Postfix using stack | Data structures](https://www.youtube.com/watch?v=PAceaOSnxQs&ab_channel=Jenny%27slecturesCS%2FITNET%26JRF)

Hint: Be very careful with order of operations, in the context of relational and logical operators. What takes precedence? IE what operation should be performed first?