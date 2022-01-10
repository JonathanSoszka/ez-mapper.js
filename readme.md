# ez-mapper.js

A simple, zero dependency object mapper for javascript and typescript.
### Typescript
ez-mapper is written in typescript and has full typing support.

# Getting Started
Install ez-map using npm

```npm install ez-mapper```

or `yarn`

```yarn add ez-mapper```

## Mapping between objects

Lets say we have the following classes, `User` and `UserDto` 

```js
class User {
    firstName: string = ""
    lastName: string = ""
}

class UserDTO {
    firstName: string = ""
    lastName: string = ""
}
```

Lets now say we have an instance of `User` that we would like to map to an instance of `UserDto`. Here is how we would do it

```js
import { map } from "../../src"

const user = new User()
user.firstName = "Jonathan"
user.lastName = "Soszka"

const userDto = new UserDto()
map(user,UserDto);

console.log(userDto) // {firstName: 'Jonathan', lastName: 'Soszka}
```

This is great we can perform simple mappings easily. Here we built our own `User` instance but in reality it would probably already be built and you would just need to make the DTO and map it.

### Nested properties
ez-mapper will also copy all nested properties by value. Lets assume our `User` and `UserDto` classes now also have a property `address` which is of type `Address` and `AddressDto` respectively
```js
class Address {
    city: string = ""
    state: string = ""
}

class AddressDto {
    city: string = ""
    state: string = ""
}
```
```js
import { map } from "ez-mapper"

const user = new User()
user.address.city = "Richmond"
user.address.state = "VA"

const userDto = new UserDto()
map(user,UserDto);

console.log(userDto.address) // {city:"Richmond", state:"VA"}

```
The nested values are copied by value not by reference, this means altering the destination object will not have side effects on the source object.

### Custom Mappings
What if our DTO has some extra fields like `fullName`. we can take care of this using a `customMapper` passed to the map method


```js
map(src, dst, (src, dst) => {
     dst.fullName = src.firstName + " " + src.lastName
 })

```

### Reusing Custom Mappings
Lets say we dont want to write the custom mappings every time we map between `User` and `UserDto` ez-mapper does not offer anyway to do this out of the box. The reason for this is that ez-mapper aims to be a lightweight no configuration required library. The way other tools achieve this like [AutoMapperTypescript](https://automapperts.netlify.app/) is to create a large configuration object to hold all of your custom mappings.

So what can we do instead? We would advise creating a mapping method in the Dto and using ez-mapper there like such. In our opinion this is better than maintaining a configuration file because it keeps the logic close to the Dto itself.

```js
//UserDto.ts
import { map } from "ez-mapper"
class UserDTO {
    firstName: string = ""
    lastName: string = ""
    fullName: string = ""

    static mapFrom(user:User){
        const dto = new UserDto()
        map(user,dto, (src,dst) => {
            dst.fullName = src.firstName + src.lastName
        })
        return dto
    }
}

//SomeOtherFile.ts
const userDto = UserDto.mapFrom(user)
```



### Transforms
ez-mapper currently has one configuration option which can be applied called `transforms` a transform is a function applied to all property names on both the source and destination target before matching occurs.

### Default Transformer
the default transformer removes all underscores from property names and also converts all property names to lowercase before attempting to find matches. This would mean that if we had two objects one with a property `FirstName` and another with a property `first_name` ez-mapper would automatically map these two properties. if you do not like this it can be turned off by adding a custom transformer to do nothing at all, see the Disable Transforms section

### Custom Transformer
to add a custom transformer to ez-mapper you need to use the `init` method somewhere in your project before any mappings are performed.

lets say we want to remove the string xyz from property names before looking for a match so that `firstNamexyz` would match `firstName`. Here is how we would do that


```js
import { init, defaultTransformer } from "ez-mapper"
init({
    transform: (x) => {
        let y = x.replace("xyz", "");
        return defaultTransformer(y);
    }
})
```

you also see we are explicitly calling the defaultTransformer at the end so we can keep the default rules as well.


### Disable Transforms
to disable all transforms use the init method as such

```js
import { init, defaultTransformer } from "ez-mapper"
init({
    transform: (x) => x
})
```



