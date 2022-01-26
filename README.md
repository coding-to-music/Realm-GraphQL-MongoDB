# Create a Realm App with Realm UI

https://docs.mongodb.com/realm/manage-apps/create/create-with-realm-ui/#std-label-create-a-realm-app

https://coding-to-music.github.io/Realm-GraphQL-MongoDB/

https://github.com/coding-to-music/Realm-GraphQL-MongoDB

https://docs.mongodb.com/realm/sdk/node/examples/read-and-write-data/

## Read & Write Data - Node.js SDK
## About the Examples on this Page
The examples on this page use the following schemas:

```java
const TaskSchema = {
  name: "Task",
  properties: {
    _id: "int",
    name: "string",
    priority: "int?",
    progressMinutes: "int?",
  },
  primaryKey: "_id",
};
const PersonSchema = {
  name: "Person",
  properties: {
    name: "string",
    age: "int?",
  },
};
const DogSchema = {
  name: "Dog",
  properties: {
    name: "string",
    owner: "Person?",
    age: "int?",
  },
};
const CatSchema = {
  name: "Cat",
  properties: {
    name: "string",
  },
};
```

## Read Operations
Find a Specific Object by Primary Key
If you know the primary key for a given object, you can look it up directly with Realm.objectForPrimaryKey().

```java
const myTask = realm.objectForPrimaryKey("Task", 12342245); // search for a realm object with a primary key that is an int.
```

## Query an Object Type
To query for objects of a given type in a realm, pass the type name to Realm.objects().

Query operations return a collection of Realm objects that match the query as a Realm.Results object. A basic query matches all objects of a given type in a realm, but you can also apply a filter to the collection to find specific objects.

// Query realm for all instances of the "Task" type.
const tasks = realm.objects("Task");

## Filter Queries
A filter selects a subset of results based on the value(s) of one or more object properties. Realm Database provides a full-featured query engine that you can use to define filters.

To filter a query, call the filtered() method on the query results collection.

## EXAMPLE
In the following example, we use the query engine's comparison operators to:

Find high priority tasks by comparing the value of the priority property value with a threshold number, above which priority can be considered high.
Find just-started or short-running tasks by seeing if the progressMinutes property falls within a certain range.

```java
// retrieve the set of Task objects
const tasks = realm.objects("Task");
// filter for tasks with a high priority
const highPriorityTasks = tasks.filtered("priority > 5");
// filter for tasks that have just-started or short-running progress
const lowProgressTasks = tasks.filtered(
  "1 <= progressMinutes && progressMinutes < 10"
);
console.log(
  `Number of high priority tasks: ${highPriorityTasks.length} \n`,
  `Number of just-started or short-running tasks: ${lowProgressTasks.length}`
);
```

## TIP
## Filter on Related and Embedded Object Properties
To filter a query based on a property of an embedded object or a related object, use dot-notation as if it were in a regular, nested object.

## Sort Query Results
A sort operation allows you to configure the order in which Realm Database returns queried objects. You can sort based on one or more properties of the objects in the results collection. Realm Database only guarantees a consistent order of results if you explicitly sort them.

To sort a query, call the sorted() method on the query results collection.

```java
// retrieve the set of Task objects
const tasks = realm.objects("Task");
// Sort tasks by name in ascending order
const tasksByName = tasks.sorted("name");
// Sort tasks by name in descending order
const tasksByNameDescending = tasks.sorted("name", true);
// Sort tasks by priority in descending order and then by name alphabetically
const tasksByPriorityDescendingAndName = tasks.sorted([
  ["priority", true],
  ["name", false],
]);
// Sort dogs by dog's owner's name.
let dogsByOwnersName = realm.objects("Dog").sorted("owner.name");
```

## TIP
## Sort on Related and Embedded Object Properties
To sort a query based on a property of an embedded object or a related object, use dot-notation as if it were in a regular, nested object.

## Write Operations
## Create a New Object
To add an object to a realm, instantiate it as you would any other object and then pass it to Realm.create() inside of a write transaction. If the realm's schema includes the object type and the object conforms to the schema, then Realm stores the object, which is now managed by the realm.

```java
// Declare the variable that will hold the dog instance.
let dog;
// Open a transaction.
realm.write(() => {
  // Assign a newly-created instance to the variable.
  dog = realm.create("Dog", { name: "Max", age: 5 });
});
// use newly created dog object
```

## Update an Object
You can add, modify, or delete properties of a Realm object inside of a write transaction in the same way that you would update any other JavaScript object.

```java
// Open a transaction.
realm.write(() => {
  // Get a dog to update.
  const dog = realm.objects("Dog")[0];
  // Update some properties on the instance.
  // These changes are saved to the realm.
  dog.name = "Maximilian";
  dog.age += 1;
});
```

## TIP
## Update Related and Embedded Objects
To update a property of an embedded object or a related object, modify the property with dot-notation or bracket-notation as if it were in a regular, nested object.

## Upsert an Object
To upsert an object, call Realm.create() with the update mode set to modified. The operation either inserts a new object with the given primary key or updates an existing object that already has that primary key.

```java
realm.write(() => {
  // Add a new person to the realm. Since nobody with ID 1234
  // has been added yet, this adds the instance to the realm.
  person = realm.create(
    "Person",
    { _id: 1234, name: "Joe", age: 40 },
    "modified"
  );
  // If an object exists, setting the third parameter (`updateMode`) to
  // "modified" only updates properties that have changed, resulting in
  // faster operations.
  person = realm.create(
    "Person",
    { _id: 1234, name: "Joseph", age: 40 },
    "modified"
  );
});
```

## Bulk Update a Collection
To apply an update to a collection of objects, iterate through the collection (e.g. with forEach()). In the iterator callback, update each object individually:

```java
realm.write(() => {
  // Create someone to take care of some dogs.
  const person = realm.create("Person", { name: "Ali" });
  // Find dogs younger than 2.
  const puppies = realm.objects("Dog").filtered("age < 2");
  // Loop through to update.
  puppies.forEach((puppy) => {
    // Give all puppies to Ali.
    puppy.owner = person;
  });
});
```

## NOTE
## Inverse Relationships
Thanks to an inverse relationship from Dog.owner to Person.dogs, Realm Database automatically updates Ali's list of dogs whenever we set her as a puppy's owner.

## Delete an Object
To delete an object from a realm, pass the object to Realm.delete() inside of a write transaction.

```java
realm.write(() => {
  // Delete the dog from the realm.
  realm.delete(dog);
  // Discard the reference.
  dog = null;
});
```

## IMPORTANT
## Do not use objects after delete
You cannot access or modify an object after you have deleted it from a Realm. If you try to use a deleted object, Realm Database throws an error.

## Delete Multiple Objects
To delete a collection of objects from a realm, pass the collection to Realm.delete() inside of a write transaction.

```java
realm.write(() => {
  // Find dogs younger than 2 years old.
  const puppies = realm.objects("Dog").filtered("age < 2");
  // Delete the collection from the realm.
  realm.delete(puppies);
});
```

## Delete All Objects of a Specific Type
To delete all objects of a given object type from a realm, pass Realm.objects(<ObjectType>) to the Realm.delete() method inside of a write transaction.

```java
realm.write(() => {
  // Delete all instances of Cat from the realm.
  realm.delete(realm.objects("Cat"));
});
```

## Delete All Objects in a Realm
To delete all objects from the realm, call Realm.deleteAll() inside of a write transaction. This clears the realm of all object instances but does not affect the realm's schema.

```java
realm.write(() => {
  // Delete all objects from the realm.
  realm.deleteAll();
});
```

## TIP
## Delete All In Development
Realm.deleteAll() is a useful method to quickly clear out your realm in the course of development. For example, rather than writing a migration to update objects to a new schema, it may be faster to delete and then re-generate the objects with the app itself.



