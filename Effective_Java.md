---
title: "Effective Java"
date: 2022-11-07T20:51:14Z
draft: false
tags: ["java"]
categories: ["note"]
---

## Chapter2 Create and Destroy Object

### 1 use static factory method to replace constructor

Advantage:

1. has name to describe the return object.
2. don't need create a new object every times.
3. can return any subclass.
4. change the return object according to the parameter.
5. class returned by method needn't exist when write class including the static factory method.

Disadvantage:

1. class cannot be extended, if it doesn't have a public or protected constructor.
2. programmer cannot find a static factory method.

static method idiomatic name:

1. from - type conversion
2. of - polymerization
3. valueOf - conversion
4. instance or getInstance - return according to the parameter
5. create or newInstance - return new object according to the parameter
6. get*type* - return *type* object like *getInstance*
7. new*type* - return new *type* object like *newInstance*
8. *type* - short version of getType or newType

### 2 consider using a builder when the constructor has multiple parameters

1. lombok `@Builder`
2. static class Builder\<T extends Builder\<T\>\>

### 3 use private constructor or enum class to strengthen singleton attribute

```java

// 1 attacked by AccessibleObject.setAccessible
public class Singleton{
    public static final instance = new Singleton();
}

// 2 consider use transient key word and readResolve method when need serialize
public class Singleton{
    private static final instance = new Singleton();
    public static Singleton getInstance() {return instance;}
}

// 3 cannot extend superclass
public enum Singleton{
    INSTANCE;
}

// 4 double check lock
public class Singleton{
    private Singleton(){}
    private static final instance;

    public static Singleton getInstance() {
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }

        return instance;
    }
}
```

### 4 use private constructor to strengthen the ability to be non-instantiable

let utility class which only has static methods and fields have a private constructor to avoid instantiating

```java
public class UtilityClass{
    private UtilityClass(){
        throw new AssetionError();
    }
}
```

### 5 prioritize dependency injection to bring in resources

static utility class and Singleton class are not suitable for a class that needs to reference the underlying resource.

simply model: bring in resources to the constructor when create a new instance.

### 6 avoid creating unnecessary objects

e.g.

```java
String s = new String("bikini"); // DON'T DD THiS!
String s = "bikini";             // RIGHT
```

creating a class `Pattern` costs much, so should create a static `Pattern` when use regular expression repeatedly

consider autoboxing when use basic types.

but don't need to create a object pool for every class, just for high cost class such as database connection pool.

### 7 delete expired object's reference

when use array to implement a stack, queue and so on, attention to delete reference in reference after poping

use `WeakHashMap` to **cache** or record **callback**


### 8 avoid using finalizer and cleaner

finalizer: `Object`'s finalize method **never use**

cleaner: implement `AutoCloseable` interface **use as a safe net**

### 9 `try-with-resources` has priority over `try-finally`

```java
try (InputStream in = new FileInputStream(src)){

}

InputStream in = new FileInputStream(src);
try {
    
} finally{
    in.close();
}
```

if error occurred in physical devices, `close` method will throw exception too.

## Chapter 3 General Methods For All Objects

### 10 compliance with common conventions when override `equals` method

1. class's every instance is essentially the only.
2. class needn't provide 'logical equality' function.
3. superclass has overrode equals, subclass can reuse, such as `AbstractSet` and `AbstractList`.
4. class is private or default, its equals method will never be used.

`equals` method implement equivalence relation

1. reflexive x.equals(x) return true
2. symmetric x.equals(y) == y.equals(x)
3. transitive if x.equals(y) return true and y.equals(z) return true, x.equals(z) return true
4. consistent without changing, x.equals(y) should be always same
5. non-nullity x.equals(null) return false

write high-quality equals method:

1. use `==` to check if the object is this object's reference
2. use `instanceof` to check if parameter is right type
3. cast parameter to right type
4. check every significant fields, basic data types except `double` and `float` use `==` to compare; `double` and `float` use static method `Float::compare` and `Double::compare`; array objects use `Arrays::equals`; other objects use `Objects::equals`

warnings:

1. always override `hashcode` when override `equals`
2. don't attempt to let `equals` too smart
3. don't change `Object` parameter in `equals`' declaration

can use Google's open source framework *AutoValue* to generate `equals` method

### 11 always override `hashcode` when override `equals`

Object standard：

1. `hashcode` should return the same value, as long as fields used in `equals` method don't change.
2. `hashcode` must be same, if `equals` return true.
3. `hashcode` needn't be different, if `equals` return false.

### 12 always override `toString`

default `toString` method return *class name*@*hash code*

in practical applications `toString` method should return all information worthing attention

### 13 override `clone` cautiously

`Cloneable` interface is a mixin interface meaning the object allow cloning.
