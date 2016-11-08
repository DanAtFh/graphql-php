# Type System
To start using GraphQL you are expected to implement a Type system. 

In **graphql-php** `type` is an instance of internal class from 
`GraphQL\Type\Definition` namespace: `ScalarType`, `ObjectType`, `InterfaceType`, 
`UnionType`, `InputObjectType` (or one of it's subclasses).

But most of the types in your schema will be [object types](/type-system/object-types/).

# Type Definition Styles
Several styles of type definitions are supported depending on your preferences.

Inline definitions:
```php
<?php
namespace MyApp;

use GraphQL\Type\Definition\ObjectType;
use GraphQL\Type\Definition\Type;

$myType = new ObjectType([
    'name' => 'MyType',
    'fields' => [
        'id' => Type::id()
    ]
]);
```

Class per type, using inheritance:
```php
<?php
namespace MyApp;

use GraphQL\Type\Definition\ObjectType;
use GraphQL\Type\Definition\Type;

class MyType extends ObjectType
{
    public function __construct()
    {
        $config = [
            // Note: 'name' is not needed in this form:
            // it will be inferred from class name by omitting namespace and dropping "Type" suffix
            'fields' => [
                'id' => Type::id()
            ]
        ];
        parent::__construct($config);
    }
}
```

Class per type, using composition:
```php
<?php
namespace MyApp;

use GraphQL\Type\Definition\ObjectType;
use GraphQL\Type\Definition\Type;
use GraphQL\Type\DefinitionContainer;

class MyType implements DefinitionContainer
{
    private $definition;

    public function getDefinition()
    {
        return $this->definition ?: ($this->definition = new ObjectType([
            'name' => 'MyType',
            'fields' => [
                'id' => Type::id()
            ]
        ]));
    }
}
```

You can also mix-and-match styles for convenience. For example:
```php
<?php
namespace MyApp;

use GraphQL\Type\Definition\ObjectType;
use GraphQL\Type\Definition\Type;

class BlogPostType extends ObjectType
{
    public function __construct()
    {
        $config = [
            'fields' => [
                'body' => new ObjectType([
                    'name' => 'BlogPostBody',
                    'fields' => [
                        'html' => Type::string(),
                        'text' => Type::string(),
                    ]
                ])
            ]
        ];
        parent::__construct($config);
    }
}
```

# Type Registry
Every type must be presented in Schema with single instance (**graphql-php** 
throws when it discovers several instances with the same `name` in schema).

Therefore if you define your type as separate PHP class you must ensure that only one 
instance of that class is added to schema.

Typical way to do this is to create registry of your types:

```php
<?php
namespace MyApp;

class TypeRegistry
{
    private $myAType;
    private $myBType;
    
    public function myAType()
    {
        return $this->myAType ?: ($this->myAType = new MyAType($this));
    }
    
    public function myBType()
    {
        return $this->myBType ?: ($this->myBType = new MyBType($this));
    }
}
```
And use this registry in type definition:

```php
<?php
namespace MyApp;
use GraphQL\Type\Definition\ObjectType;

class MyAType extends ObjectType
{
    public function __construct(TypeRegistry $types) 
    {
        parent::__construct([
            'fields' => [
                'b' => $types->myBType()                
            ]
        ]);
    }
}
```
Obviously you can automate this registry as you wish to reduce boilerplate or even 
introduce Dependency Injection Container if your types have other dependencies.

Alternatively all methods of registry could be static if you prefer - then there is no need
to pass it in constructor - instead just use use `TypeRegistry::myAType()` in your type definitions.

# Custom Metadata
All types in **graphql-php** accept configuration array. In some cases you may be interested in 
passing your own metadata for type or field definition.

**graphql-php** preserves original configuration array in every type or field instance in 
public property `$config`. Use it to implement app-level mappings and definitions.