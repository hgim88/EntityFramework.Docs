---
title: SQLite Database Provider - Limitations - EF Core
author: bricelam
ms.date: 07/16/2020
ms.assetid: 94ab4800-c460-4caa-a5e8-acdfee6e6ce2
uid: core/providers/sqlite/limitations
---
# SQLite EF Core Database Provider Limitations

The SQLite provider has a number of migrations limitations. Most of these limitations are a result of limitations in the underlying SQLite database engine and are not specific to EF.

## Modeling limitations

The common relational library (shared by Entity Framework relational database providers) defines APIs for modelling concepts that are common to most relational database engines. A couple of these concepts are not supported by the SQLite provider.

* Schemas
* Sequences

## Query limitations

SQLite doesn't natively support the following data types. EF Core can read and write values of these types, and querying for equality (`where e.Property == value`) is also supported. Other operations, however, like comparison and ordering will require evaluation on the client.

* DateTimeOffset
* Decimal
* TimeSpan
* UInt64

Instead of `DateTimeOffset`, we recommend using DateTime values. When handling multiple time zones, we recommend converting the values to UTC before saving and then converting back to the appropriate time zone.

The `Decimal` type provides a high level of precision. If you don't need that level of precision, however, we recommend using double instead. You can use a [value converter](../../modeling/value-conversions.md) to continue using decimal in your classes.

``` csharp
modelBuilder.Entity<MyEntity>()
    .Property(e => e.DecimalProperty)
    .HasConversion<double>();
```

## Migrations limitations

The SQLite database engine does not support a number of schema operations that are supported by the majority of other relational databases. If you attempt to apply one of the unsupported operations to a SQLite database then a `NotSupportedException` will be thrown.

A rebuild will be attempted in order to perform certain operations. Rebuilds are only possible for database artifacts that are part of your EF Core model. If a database artifact isn't part of the model--for example, if it was created manually inside a migration--then a `NotSupportedException` is still thrown.

| Operation            | Supported?  | Requires version |
|:---------------------|:------------|:-----------------|
| AddCheckConstraint   | ✔ (rebuild) | 5.0              |
| AddColumn            | ✔           | 1.0              |
| AddForeignKey        | ✔ (rebuild) | 5.0              |
| AddPrimaryKey        | ✔ (rebuild) | 5.0              |
| AddUniqueConstraint  | ✔ (rebuild) | 5.0              |
| AlterColumn          | ✔ (rebuild) | 5.0              |
| CreateIndex          | ✔           | 1.0              |
| CreateTable          | ✔           | 1.0              |
| DropCheckConstraint  | ✔ (rebuild) | 5.0              |
| DropColumn           | ✔ (rebuild) | 5.0              |
| DropForeignKey       | ✔ (rebuild) | 5.0              |
| DropIndex            | ✔           | 1.0              |
| DropPrimaryKey       | ✔ (rebuild) | 5.0              |
| DropTable            | ✔           | 1.0              |
| DropUniqueConstraint | ✔ (rebuild) | 5.0              |
| RenameColumn         | ✔           | 2.2.2            |
| RenameIndex          | ✔ (rebuild) | 2.1              |
| RenameTable          | ✔           | 1.0              |
| EnsureSchema         | ✔ (no-op)   | 2.0              |
| DropSchema           | ✔ (no-op)   | 2.0              |
| Insert               | ✔           | 2.0              |
| Update               | ✔           | 2.0              |
| Delete               | ✔           | 2.0              |

## Migrations limitations workaround

You can workaround some of these limitations by manually writing code in your migrations to perform a rebuild. Table rebuilds involve creating a new table, copying data to the new table, dropping the old table, renaming the new table. You will need to use the `Sql(string)` method to perform some of these steps.

See [Making Other Kinds Of Table Schema Changes](https://sqlite.org/lang_altertable.html#otheralter) in the SQLite documentation for more details.
