# Aerospike Typed Java Client
Statically typed [Aerospike Java Client](https://github.com/aerospike/aerospike-client-java).

[Aerospike](https://aerospike.com/) is the great In-Memory Key-Value database. Though it supports some types (`Integer`, `List`, `Blob`, etc.) neither the client nor the Aerospike server itself performs any validations on putting and obtaining values. So, for the same [namespace and set](https://docs.aerospike.com/server/architecture/data-model#:~:text=The%20Aerospike%20database%20does%20not,to%20first%20update%20any%20schema.) combination you might have [records](https://docs.aerospike.com/server/architecture/data-model#records) with different bins collections (names, values, and bins count can differ for each record). Mostly that's not a feature but an unpleasant obstacle that leads to bugs at runtime due to unexpected type of retrieved values.

Here are the features.

## Java Classes Generation

The data structure should be described as YAML file with special format. Take a look at the example below.

```yml
my-namespace:
  description: Default namespace
  sets:
    university:
      description: Set to store information about universities
      key:
        description: University numeric ID
        type: Integer
      bins:
        name:
          description: Name of the university
          type: String
        binary:
          description: Binary data about university
          type: Blob
```

Here we got a single namespace `my-namespace` and a single set `university`. The latter's key is of type `Integer` and it contains two bins. `String` bin `name` and `Blob` bin `binary`.

If we want to create a new university, we can do it directly. Take a look at the code snippet below.

```java
AerospikeClient client = getAerospikeClient();
client.put(
    new Key("my-namespace", "university", 789),
    new Bin("name", "Princeton"),
    new Bin("binary", new Value.Blob(new byte[] {5, 6, 12}))
);
```

The problem here is lack of static typing. We have to declare namespace, set, and bin names directly. Thankfully, the library provides a better approach. Take a look at the example below.

```java
import my.package.UNIVERSITY;

AerospikeClient client = getAerospikeClient();
client.put(
    UNIVERSITY.key(789), 
    UNIVERSITY.NAME.of("Princeton"), 
    UNIVERSITY.BINARY.of(new byte[] {5, 6, 12})
);
```

During the project's compilation the specified schema is parsed and the corresponding classes are generated.
So, the validity of types and names are checked by the compiler. Great!

Besides, the generated classes are declared as [generic ones](https://www.baeldung.com/java-generics).
It means that the underlined example won't compile.

```java
import my.package.UNIVERSITY;
import my.package.STUDENT;

AerospikeClient client = getAerospikeClient();
client.put(
    // compile error due to generic types mismatch
    STUDENT.key("some_student_id"), 
    UNIVERSITY.NAME.of("Princeton"), 
    UNIVERSITY.BINARY.of(new byte[] {5, 6, 12})
);
```
