---
title: About constraints
excerpt: Learn how constraints help you keep your data valid and consistent
products: [cloud, mst, self_hosted]
keywords: [schemas, constraints]
---

# About constraints

Constraints are rules that apply to your database columns. This prevents you
from entering invalid data into your database. When you create, change, or
delete constraints on your hypertables, the constraints are propagated to the
underlying chunks, and to any indexes.

Hypertables support all standard PostgreSQL constraint types, except for
foreign key constraints from a hypertable referencing another hypertable.

For example, you can create a table that only allows positive device IDs, and
non-null temperature readings. You can also check that time values for all
devices are unique. To create this table, with the constraints, use this
command:

```sql
CREATE TABLE conditions (
    time       TIMESTAMPTZ
    temp       FLOAT NOT NULL,
    device_id  INTEGER CHECK (device_id > 0),
    location   INTEGER REFERENCES locations (id),
    PRIMARY KEY(time, device_id)
);

SELECT create_hypertable('conditions', by_range('time'));
```

<Highlight type="note">
The `by_range` dimension builder is an addition to TimescaleDB 2.13.
</Highlight>

This example also references values in another `locations` table using a foreign
key constraint.

<Highlight type="note">
Time columns used for partitioning must not allow `NULL` values. A
`NOT NULL` constraint is added by default to these columns if it doesn't already
exist.
</Highlight>

For more information on how to manage constraints, see the
[PostgreSQL docs][postgres-createconstraint].

[postgres-createconstraint]: https://www.postgresql.org/docs/current/static/ddl-constraints.html
