---
title: Primary Key Constraint
summary: A primary key constraint specifies columns that can be used to uniquely identify rows in a table.
toc: true
docs_area: reference.sql
---

The `PRIMARY KEY` [constraint]({% link {{ page.version.version }}/constraints.md %}) specifies that the constrained columns' values must uniquely identify each row. A table can only have one primary key, but it can have multiple [unique constraints]({% link {{ page.version.version }}/unique.md %}).

You should explicitly define a table's primary key in the [`CREATE TABLE`]({% link {{ page.version.version }}/create-table.md %}) statement. If you don't define a primary key at table creation time, CockroachDB will create a `rowid` column that is `NOT VISIBLE`, use the [`unique_rowid()` function]({% link {{ page.version.version }}/functions-and-operators.md %}#id-generation-functions) as its default value, and use that as the table's primary key.

You can [change the primary key](#changing-primary-key-columns) of an existing table with an [`ALTER TABLE ... ALTER PRIMARY KEY`]({% link {{ page.version.version }}/alter-table.md %}#alter-primary-key) statement, or by using [`DROP CONSTRAINT`]({% link {{ page.version.version }}/alter-table.md %}#drop-constraint) and then [`ADD CONSTRAINT`]({% link {{ page.version.version }}/alter-table.md %}#add-constraint) in the same transaction. You cannot fully drop the `PRIMARY KEY` constraint on a table without replacing it as it provides an intrinsic structure to the table's data.

## Syntax

`PRIMARY KEY` constraints can be defined at the [table level](#table-level). However, if you only want the constraint to apply to a single column, it can be applied at the [column level](#column-level).

### Column level

<div>
{% remote_include https://raw.githubusercontent.com/cockroachdb/generated-diagrams/{{ page.release_info.crdb_branch_name }}/grammar_svg/primary_key_column_level.html %}
</div>

 Parameter | Description
-----------|-------------
 `table_name` | The name of the table you're creating.
 `column_name` | The name of the Primary Key column. For [multi-region tables]({% link {{ page.version.version }}/multiregion-overview.md %}#table-locality), you can use the `crdb_region` column within a composite primary key in the event the original primary key may contain non-unique entries across multiple, unique regions.
 `column_type` | The Primary Key column's [data type]({% link {{ page.version.version }}/data-types.md %}).
 `column_constraints` | Any other column-level [constraints]({% link {{ page.version.version }}/constraints.md %}) you want to apply to this column.
 `column_def` | Definitions for any other columns in the table.
 `table_constraints` | Any table-level [constraints]({% link {{ page.version.version }}/constraints.md %}) you want to apply.

**Example**

{% include_cached copy-clipboard.html %}
~~~ sql
> CREATE TABLE orders (
    order_id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_date      TIMESTAMP NOT NULL,
    order_mode      STRING(8),
    customer_id     INT,
    order_status    INT
  );
~~~

### Table level

<div>
{% remote_include https://raw.githubusercontent.com/cockroachdb/generated-diagrams/{{ page.release_info.crdb_branch_name }}/grammar_svg/primary_key_table_level.html %}
</div>

 Parameter | Description
-----------|-------------
 `table_name` | The name of the table you're creating.
 `column_def` | Definitions for any other columns in the table.
 `name` | The name you want to use for the constraint, which must be unique to its table and follow these [identifier rules]({% link {{ page.version.version }}/keywords-and-identifiers.md %}#identifiers).
 `column_name` | The name of the column you want to use as the `PRIMARY KEY`.<br/><br/>The order in which you list columns here affects the structure of the primary index.
 `table_constraints` | Any other table-level [constraints]({% link {{ page.version.version }}/constraints.md %}) you want to apply.

**Example**

{% include_cached copy-clipboard.html %}
~~~ sql
> CREATE TABLE IF NOT EXISTS inventories (
    product_id        INT,
    warehouse_id      INT,
    quantity_on_hand  INT NOT NULL,
    PRIMARY KEY (product_id, warehouse_id)
  );
~~~

## Details

The columns in the `PRIMARY KEY` constraint are used to create its primary [index]({% link {{ page.version.version }}/indexes.md %}#creation), which CockroachDB uses by default to access the table's data. This index does not take up additional disk space (unlike secondary indexes, which do) because CockroachDB uses the primary index to structure the table's data in the key-value layer. For more information, see our blog post [SQL in CockroachDB: Mapping Table Data to Key-Value Storage](https://www.cockroachlabs.com/blog/sql-in-cockroachdb-mapping-table-data-to-key-value-storage/).

To ensure each row has a unique identifier, the `PRIMARY KEY` constraint combines the properties of both the [`UNIQUE`]({% link {{ page.version.version }}/unique.md %}) and [`NOT NULL`]({% link {{ page.version.version }}/not-null.md %}) constraints. The properties of both constraints are necessary to make sure each row's primary key columns contain distinct sets of values. The properties of the `UNIQUE` constraint ensure that each value is distinct from all other values. However, because `NULL` values never equal other `NULL` values, the `UNIQUE` constraint is not enough (two rows can appear the same if one of the values is `NULL`). To prevent the appearance of duplicated values, the `PRIMARY KEY` constraint also enforces the properties of the `NOT NULL` constraint.

For best practices, see [Select primary key columns]({% link {{ page.version.version }}/schema-design-table.md %}#select-primary-key-columns).

{{site.data.alerts.callout_danger}}
{% include {{page.version.version}}/sql/add-size-limits-to-indexed-columns.md %}
{{site.data.alerts.end}}

## Example

{% include_cached copy-clipboard.html %}
~~~ sql
> CREATE TABLE IF NOT EXISTS inventories (
    product_id        INT,
    warehouse_id      INT,
    quantity_on_hand  INT NOT NULL,
    PRIMARY KEY (product_id, warehouse_id)
  );
~~~

{% include_cached copy-clipboard.html %}
~~~ sql
> INSERT INTO inventories VALUES (1, 1, 100);
~~~

{% include_cached copy-clipboard.html %}
~~~ sql
> INSERT INTO inventories VALUES (1, 1, 200);
~~~

~~~
pq: duplicate key value (product_id,warehouse_id)=(1,1) violates unique constraint "primary"
~~~

{% include_cached copy-clipboard.html %}
~~~ sql
> INSERT INTO inventories VALUES (1, NULL, 100);
~~~

~~~
pq: null value in column "warehouse_id" violates not-null constraint
~~~

## Changing primary key columns

You can change the primary key of an existing table by doing one of the following:

- Issuing an [`ALTER TABLE ... ALTER PRIMARY KEY`]({% link {{ page.version.version }}/alter-table.md %}#alter-primary-key) statement. When you change a primary key with `ALTER PRIMARY KEY`, the old primary key index becomes a secondary index. This helps optimize the performance of queries that still filter on the old primary key column.
- Issuing an [`ALTER TABLE ... DROP CONSTRAINT ... PRIMARY KEY`]({% link {{ page.version.version }}/alter-table.md %}#drop-constraint) statement to drop the primary key, followed by an [`ALTER TABLE ... ADD CONSTRAINT ... PRIMARY KEY`]({% link {{ page.version.version }}/alter-table.md %}#add-constraint) statement, in the same transaction, to add a new primary key. This replaces the existing primary key without creating a secondary index from the old primary key. For examples, see [Add constraints]({% link {{ page.version.version }}/alter-table.md %}#add-constraints) and [Drop constraints]({% link {{ page.version.version }}/alter-table.md %}#drop-constraints).

{{site.data.alerts.callout_info}}
You can use an [`ADD CONSTRAINT ... PRIMARY KEY`]({% link {{ page.version.version }}/alter-table.md %}#add-constraint) statement without a [`DROP CONSTRAINT ... PRIMARY KEY`]({% link {{ page.version.version }}/alter-table.md %}#drop-constraint) if the primary key was not explicitly defined at [table creation]({% link {{ page.version.version }}/create-table.md %}), and the current [primary key is on `rowid`]({% link {{ page.version.version }}/indexes.md %}#creation).
{{site.data.alerts.end}}

## See also

- [Constraints]({% link {{ page.version.version }}/constraints.md %})
- [`CHECK` constraint]({% link {{ page.version.version }}/check.md %})
- [`DEFAULT` constraint]({% link {{ page.version.version }}/default-value.md %})
- [`REFERENCES` constraint (Foreign Key)]({% link {{ page.version.version }}/foreign-key.md %})
- [`PRIMARY KEY` constraint]({% link {{ page.version.version }}/primary-key.md %})
- [`NOT NULL` constraint]({% link {{ page.version.version }}/not-null.md %})
- [`UNIQUE` constraint]({% link {{ page.version.version }}/unique.md %})
- [`SHOW CONSTRAINTS`]({% link {{ page.version.version }}/show-constraints.md %})
- [`ALTER PRIMARY KEY`]({% link {{ page.version.version }}/alter-table.md %}#alter-primary-key)
- [`ALTER TABLE`]({% link {{ page.version.version }}/alter-table.md %})
