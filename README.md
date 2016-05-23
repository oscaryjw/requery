![requery](http://requery.github.io/logo.png)

A light but powerful object mapping and SQL generator for Java/Android with RxJava and Java 8 support.
Easily map to or create databases, perform queries and updates from any platform that uses Java.

[![Build Status](https://travis-ci.org/requery/requery.svg?branch=master)](https://travis-ci.org/requery/requery)
[![Download](https://api.bintray.com/packages/requery/requery/requery/images/download.svg)](https://bintray.com/requery/requery/requery/_latestVersion)

Examples
--------

Define entities from an abstract class:

```java
@Entity
abstract class AbstractPerson {

    @Key @Generated
    int id;

    @Index(name = "name_index")              // table specification
    String name;

    @OneToMany                               // relationships 1:1, 1:many, many to many
    Set<Phone> phoneNumbers;

    @Converter(EmailToStringConverter.class) // custom type conversion
    Email email;

    @PostLoad                                // lifecycle callbacks
    void afterLoad() {
        updatePeopleList();
    }
    // getter, setters, equals & hashCode automatically generated into Person.java
}

```
or from an interface:

```java
@Entity
public interface Person {

    @Key @Generated
    int getId();

    String getName();

    @OneToMany
    Set<Phone> getPhoneNumbers();

    String getEmail();
}
```
or use immutable types such as those generated by [@AutoValue](https://github.com/google/auto/tree/master/value):

```java
@AutoValue
@Entity
abstract class Person {

    @AutoValue.Builder
    static abstract class Builder {
        abstract Builder setId(int id);
        abstract Builder setName(String name);
        abstract Builder setEmail(String email);
        abstract Person build();
    }

    static Builder builder() {
        return new AutoValue_Person.Builder();
    }

    @Key
    abstract int getId();

    abstract String getName();
    abstract String getEmail();
}
```
(Note some features will not be available when using immutable types, see [here](https://github.com/requery/requery/wiki/Immutable-types))

**Queries:** dsl based query that maps to SQL

```java
Result<Person> query = data
    .select(Person.class)
    .where(Person.NAME.lower().like("b%")).and(Person.AGE.gt(20))
    .orderBy(Person.AGE.desc())
    .limit(5)
    .get();
```

**Relationships:** represent relations more efficiently with Java 8 Streams, RxJava Observables or
plain iterables. (sets and lists are supported to)

```java
@Entity
abstract class AbstractPerson {

    @Key @Generated
    int id;

    @ManyToMany
    Result<Group> groups;
    // equivalent to:
    // data.select(Group.class)
    // .join(Group_Person.class).on(Group_ID.equal(Group_Person.GROUP_ID))
    // .join(Person.class).on(Group_Person.PERSON_ID.equal(Person.ID))
    // .where(Person.ID.equal(id))
}
```

**Java 8 [streams](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html):**

```java
data.select(Person.class)
    .orderBy(Person.AGE.desc())
    .get()
    .stream().forEach(System.out::println);
```

**Java 8 optional and time support:**

```java
public interface Person {

    @Key @Generated
    int getId();

    String getName();
    Optional<String> getEmail();
    ZonedDateTime getBirthday();
}
```

**[RxJava](https://github.com/ReactiveX/RxJava) [Observables](http://reactivex.io/documentation/observable.html):**

```java
Observable<Person> observable = data
    .select(Person.class)
    .orderBy(Person.AGE.desc())
    .get()
    .toObservable();
```

**[RxJava](https://github.com/ReactiveX/RxJava) observe query on table changes:**

```java
Observable<Person> observable = data
    .select(Person.class)
    .orderBy(Person.AGE.desc())
    .get()
    .toSelfObservable().subscribe(::updateFromResult);
```

**Read/write separation** Along with immutable types optionally separate queries (reading)
and updates (writing):

```java
int rows = data.update(Person.class)
    .set(Person.ABOUT, "student")
    .where(Person.AGE.lt(21)).get().value();
```

Features
--------

- No Reflection
- Fast startup & performance
- No dependencies (RxJava is optional)
- Typed query language
- Table generation
- Supports JDBC and most popular databases (MySQL, Oracle, SQL Server, Postgres and more)
- Supports Android (SQLite, RecyclerView, Databinding, SQLCipher)
- Blocking and non-blocking API
- Partial objects/refresh
- Upsert support
- Caching
- Lifecycle callbacks
- Custom type converters
- Compile time entity validation
- JPA annotations (however requery is not a JPA provider)

Reflection free
---------------

requery uses compile time annotation processing to generate entity model classes and mapping
attributes. On Android this means you get about the same performance reading objects from a query
as if it was populated using the standard Cursor and ContentValues API.

Query with Java
---------------

The compiled classes work with the query API to take advantage of compile time generated attributes.
Create type safe queries and avoid hard to maintain, error prone string concatenated queries.

Relationships
-------------

You can define One-to-One, One-to-Many, Many-to-One, and Many-to-Many relations in your models using
annotations. Relationships can be navigated in both directions. Of many type relations can be loaded
into standard java collection objects or into a more efficient
[Result](http://requery.github.io/javadoc/io/requery/query/Result.html) type.
From a [Result](http://requery.github.io/javadoc/io/requery/query/Result.html)
easily create a Stream, RxJava Observable, Iterator, List or Map.

Many-to-Many junction tables can be generated automatically. Additionally the relation model is
validated at compile time eliminating runtime errors.

vs JPA
------

requery provides a modern set of interfaces for persisting and performing queries. Some key
differences between requery and JPA providers like Hibernate or EclipseLink:

- Queries maps directly to SQL as opposed to JPQL.
- Dynamic Queries easily done through a DSL as opposed to the verbose `CriteriaQuery` API.
- Uses easily understandable extended/generated code instead of reflection/bytecode weaving for
  state tracking and member access

Android
-------

Designed specifically with Android support in mind. Comparison to other Android libraries:

Feature               |  requery |  ORMLite |  Squidb  |  DBFlow   | GreenDao
----------------------|----------|----------|----------|-----------|-----------
Relational mapping    |  Y       |  Y(1)    |  N       |  Y        | Y(1)
Inverse relationships |  Y       |  N       |  N       |  N        | N
Compile time          |  Y       |  N       |  Y       |  Y        | Y(2)
Query DSL             |  Y       |  N       |  N(3)    |  N(3)     | N(3)
JDBC Support          |  Y       |  Y       |  N       |  N        | N
Table Generation      |  Y       |  Y       |  Y       |  Y        | Y
JPA annotations       |  Y       |  Y       |  N       |  N        | N
RxJava support        |  Y       |  N       |  Y(4)    |  N        | N

1) Excludes Many-to-Many
2) Not annotation based
3) Builder only not DSL
4) Table changes only

See [requery-android/example](https://github.com/requery/requery/tree/master/requery-android/example)
for an example Android project using databinding and interface based entities. For more information
see the [Android](https://github.com/requery/requery/wiki/Android) page.

Supported Databases
-------------------
Tested on some of the most popular databases:

- PostgresSQL (9.1+)
- MySQL 5.x
- Oracle 12c+
- Microsoft SQL Server 2012 or later
- SQLite (Android or with the [xerial](https://github.com/xerial/sqlite-jdbc) JDBC driver)
- Apache Derby 10.11+
- H2 1.4+
- HSQLDB 2.3+

JPA Annotations
---------------

A subset of the JPA annotations that map onto the requery annotations are supported.
See [here](https://github.com/requery/requery/wiki/JPA-Annotations) for more information.

Upserts
-------

Upserts are generated with the appropriate database specific query statements:
- Oracle/SQL Server/HSQL: `merge into when matched/not matched`
- PostgresSQL: `on conflict do update` (requires 9.5 or later)
- MySQL: `on duplicate key update`

Using it
--------

Currently beta versions are available on bintray jcenter / maven central.

```gradle
repositories {
    jcenter()
}

dependencies {
    compile 'io.requery:requery:1.0.0-beta19'
    compile 'io.requery:requery-android:1.0.0-beta19' // for android
    apt 'io.requery:requery-processor:1.0.0-beta19'   // use an APT plugin
}
```

For information on gradle and annotation processing & gradle see the [wiki](https://github.com/requery/requery/wiki/Gradle-&-Annotation-processing#annotation-processing).

License
-------

    Copyright (C) 2016 requery.io

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

