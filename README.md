QueryMutator
============

### Queryable and Enumerable extensions for automapping objects and mapping multiple expressions into one.

What's new in 1.2.0-beta?
-------------------------
* Easily make a shallow copy of an object.
```c#
var dog = new Dog { Name = "Bloki" };
var cloneDog = dog.Clone();
```
This is only an easy shorthand for the appropriate MapTo method.

What's new in 1.1.0-beta?
-------------------------
* Now you can map a single object to another!
```c#
var dog = new Dog { Name = "Bloki" };
var cloneDog = dog.MapTo<Dog, Dog>();
var cat = dog.MapTo<Dog, Cat>();
```
Keep in mind that this uses a safe extension, so calling MapTo on 
an object won't cause a NullReference exception if the object is null.

What's it for?
--------------

It's an object mapper for .NET (4.5.1, 4.6.1) and .NET Core (Framework 5.4). If you ever tried to map objects 
from an IQueryable to a custom 
type of your own, you probably saw and wrote lots of code like this:
```c#
List<UserDto> users;
using (var ctx = new MyContext())
{
    users = ctx.Users.Select(u => new UserDto
    {
        Id = u.Id,
        FirstName = u.FirstName,
        // and so on
    }).ToList();
}
```

Using QueryMutator makes the above query a lot easier. Basically, you can register type-pairs which you 
would like to convert between, or you can let it lazy load and generate the mapping at runtime.

```c#
List<UserDto> users;
using (var ctx = new MyContext())
{
    users = ctx.Users.MapToList<User, UserDto>();
}
```

QueryMutator matches properties by name, and tries to match the object types against each other. It works 
by recursively transversing the type graphs.

Why not just use AutoMapper instead?
------------------------------------

QueryMutator has a neat little side effect: as the query is built at runtime (even if you don't register the 
expressions you want to use for mapping), you can *merge* the automatically generated or custom registered 
mapping with _another expression_ to effectively extend your object or remap a property at runtime.

```c#
List<UserWithFriendsCountDto> users;
using (var ctx = new MyContext())
{
    users = ctx.Users.MapToList<User, UserWithFriendsCountDto>(u => new UserWithFriendsCountDto 
    {
        FriendsCount = u.Friends.Count()
		// Nothing else needs to be mapped!
    });
}
```

The above code will automap the properties of User to the DTO type, *and add another property to the 
mapped object*. This will all happend as the query is executed, so no unneeded round trips to the database!

Great! What else?
-----------------

QueryMutator lives in the ```System.Linq``` namespace. It might seem a bit overkill, but actually it can
make your life easier, as wherever you might find a Select extension method, you'll find QueryMutator's methods:
* IQueryable MapTo extension: automatically map the source to the target or provide a merge expression to merge
the output with another object. At runtime, only one instantiation will occur.
* IQueryable MapToList extension: same as above, but you don't have to call ToList() on it to pull it to memory.
* IEnumerable MapTo and MapToList extensions: same as with IQueryable, but compiles the available expression at
runtime so that you can use QueryMutator in every situation.
* QueryMutator.GetMapping: retrieve or lazy load a mapping between two types.
* QueryMutator.RegisterMapping: register an automatically mapped expression or provide a custom one to use 
later. You can even provide that types as parameters instead of generic type arguments, as to make it easier to 
automate generating mappings in your own code using Reflection.
* QueryMutator.CurrentConfiguration: the configuration object allows you to set options on how QueryMutator
generates expressions, when should it throw an exception, etc. The extension methods use the
CurrentConfiguration static property, but you can switch it out anytime at runtime.

I'm sold. Where can I get it?
-----------------------------

You can get the *source* here at [GitHub](https://github.com/yugabe/QueryMutator), feel free to send a pull 
request if you'd like to contribute. The project is open source and free to use.  
You can download the NuGet package from [NuGet](https://www.nuget.org/packages/QueryMutator/).