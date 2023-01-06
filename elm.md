# Elm language

Pure functions
Strong Static typing
Type inference
Immutable data

## Comments

```elm
a + b -- This is a coment
```

## Functions

Pure

```elm
functionName : arg1Type -> arg2Type -> retType       -- Optional type annotation
functionName arg1 arg2 =
  <single expression>
```

### Currying

```elm
add : Int -> Int -> Int
add a b = a + b

-- Can be written as
add : Int -> (Int -> Int)   -- Its a fn that takes an Int and returns another fn
add a = ((+) a)             -- ((add a) b)

-- So we can call
add1 = add 1        -- Returns a fn of type Int -> Int
add1 2 = 3
```

### Operators as functions

Just wrap operator in parentheses.

```elm
doubleNumbers = List.map ((*) 2)
```

### Pipes

Result above is passed as last arg.

```elm
list
  |> List.map ((*) 2)
  |> List.sum
```

## Types

### List

Linked list: iterable, non-indexable

```elm
[1, 2, 3]
```

### Tuples

Only 2 or 2 (TODO how many max?) elements.

```elm
dog : ( String, Int )
dog = ( "Tucker", 11 )

name = Tuple.first dog  -- "Tucker"
age = Tuple.second dog  -- 11
```

### Records

TODO constructors

```elm
dog : { name : String, age : Int, breed : String }
dog =
  { name = "Tucker"
  , age = 11
  , breed = "Sheltie"
  }

dog.age  -- 11

.name  -- this is a function taking a record and pulling "name" from it

-- "Updating", actually cloning with changes (immutability)
olderDog = { dog | age = 12 }
dog.age         -- 11 (unchanged)
olderDog.age    -- 12
```

### Type aliasing

```elm
type alias Dog =
  { name : String
  , age : Int
  , breed : String
  }


dog : Dog
dog =
  { name = "Tucker"
  , age = 11
  , breed = "Sheltie"
  }

-- With the alias we also get a constructor, so we can do
dog = Dog "Tucker" 11 "Sheltie"  -- args in order of declaration
```

### Union types

```elm
type Breed
  = Sheltie
  | Cogri
  | GoldenRetriever
  | Mix Breed Breed     -- parametric

-- Generates constructors for all Breed values
-- The parametric ones (TODO parametric is the correct name?) have params,
-- so we'd call
(Mix Sheltie Cogri)
```

### `Maybe` type instead of nullity

```elm
-- Defined by Elm
type Maybe a         -- a is a type param
  = Just a
  | Nothing

divide : number -> number -> Maybe Float
divide x y =
  if y == 0 then
    Nothing
  else
    Just (x / y)

divide 4 2  -- Just 2
divide 4 0  -- Nothing

-- Then we can't crash...
case divide 4 2 of
  Just n ->
    "Result is " ++ (toString n)

  Nothing ->        -- ...since we're forced by the compiler to handle this case
    "No result"
```

## Pattern Matching

TODO

## Files and modules

```elm
module ModuleName exposing (a, b)  -- or exposing all (..)

import Dependency as Alias exposing (..)  -- same syntax for imports
```

# Framework

## Elm Architecture

Model-View-Update (MVU)
Unidirectional bindings
  The View doesn't have any state. All state is in the Model.
The runtime wrapps and isolates your app
  All comms are done through it
    Kinds:
      Commands: HTTP, random nums, etc.
      Events: clicks, etc.
      Subscriptions: websockets, mouse position, etc.
    Generate Tasks that return custom Messages

TODO full picture here!
...Msg -> Update Model -> View(Model) -> Browser DOM -> Msg...

## Views

```elm
view : Model -> Html Msg
view model =
  div [] [ text "aja!" ]        -- <elmFn> [ <attrFn> ] [ <children> ]
```

# Tooling

## Package manager

TODO

## Compiler

TODO

## REPL

TODO

## Dev Server

TODO

# Resources

Toward A Better Front End Architecture: ELM
  by Jeremy Fairbank (@elpapapollo)
https://www.youtube.com/watch?v=aNour03V98E

