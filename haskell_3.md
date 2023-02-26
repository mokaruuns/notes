# Lecture 3: Datas, Classes, Instances

Содержание:
- type: type aliases
- ADT's (algebraic data types):
    - product types
    - sum types
- data and examples
- Record syntax
    - -XDuplicateRecordFields
    - -XRecordWildCards
- newtype
- Type classes: class
- instance
    - compare to Java interface
    - -XInstanceSigs
    - examples of standard classes: Eq, Ord, Num, Show, Read
- deriving
- -XGeneralizedNewtypeDeriving
- Modules cheatsheet
- Church-encoding ADT

Presentation: http://slides.com/fp-ctd/lecture-3#/

## Type

type aliases - псевдонимы типов

```haskell
type String = [Char]

type User = (String, Int)
```

## ADT's (algebraic data types)

ADT - тип данных, который может быть представлен как сумма (sum) или произведение (product) других типов данных.

### Product types

Произведение типов данных. Примеры:

```haskell
data Point = Point Double Double
data Color = RGB Int Int Int | CMYK Double Double Double Double
data Point2D a = Point2D a a
```

### Sum types

Сумма типов данных. Примеры:

```haskell
data Bool = False | True
data Maybe a = Nothing | Just a
data Either a b = Left a | Right b
data TrafficLight = Red | Yellow | Green
```

Пример работы с data type User:

```haskell
data User = MkUser String Int

user1 = MkUser "John" 20
user2 = MkUser "Jane" 30

getName :: User -> String
getName (MkUser name _) = name

getAge :: User -> Int
getAge (MkUser _ age) = age
```


## data and examples

### Enums

```haskell
data TrafficLight = Red | Yellow | Green
```

### Structures 

```haskell
data User = MkUser String Int
```

### Parametric


```haskell
data Point2D a = Point2D a a
```

### Param Sum

```haskell
data Shape a = Circle a | Rectangle a a
```

### Maybe

```haskell
data Maybe a = Nothing | Just a
```


### Either

```haskell
data Either a b = Left a | Right b
```

### Recursive

```haskell
data List a = Nil | Cons a (List a)
```
 
## Record syntax

Record syntax - синтаксис для создания структур данных с именованными полями.

```haskell
-- name и age - селекторы
data User = MkUser
  { name :: String
  , age  :: Int
  }

Ivan :: User
Ivan = MkUser "Ivan" 20


```

эквивалентно:

```haskell
data User = MkUser String Int

name :: User -> String
name (MkUser n _) = n

age :: User -> Int
age (MkUser _ a) = a
```

### Record update syntax

```haskell
CloneIvan :: User
CloneIvan = Ivan { age = 21 }
```

### Operator record syntax

```haskell
data R = R { (-->) :: Int -> Int }
let r = R { (-->) = (+1) }
r --> 1 -- 2
```

## -XDuplicateRecordFields

это расширение позволяет использовать одинаковые имена полей в разных структурах данных.


```haskell
{-# LANGUAGE DuplicateRecordFields #-}
```

## -XRecordWildCards
это расширение позволяет использовать синтаксис обновления записи, но без указания имени структуры данных.

```haskell
{-# LANGUAGE RecordWildCards #-}
```

## newtype

newtype вводит новый тип, чьё представление идентично существующему. newtype используется для оптимизации, чтобы избежать лишних копирований. Он не может иметь более одного конструктора.

```haskell
data Message = Message String

newtype Message = Message String
```

Давайте посмотрим на пример, где newtype используется для дополнительной проверки типов:

```haskell
-- проблема в том, что мы можем создать невалидную пару ключей
derivePublicKey :: String -> String

checkKeyPair :: String -> String -> Bool
checkKeyPair publicKey privateKey = derivePublicKey privateKey == publicKey

-- решение: newtype
newtype PublicKey = PublicKey String
newtype PrivateKey = PrivateKey String

derivePublicKey :: PrivateKey -> PublicKey

checkKeyPair :: PublicKey -> PrivateKey -> Bool
checkKeyPair publicKey privateKey = derivePublicKey privateKey == publicKey
```


## Type classes: class and instance

Type classes - это интерфейсы, которые определяют поведение типов. Они позволяют создавать полиморфные функции, которые работают с разными типами данных. Изпользуются для реализации ad-hoc полиморфизма.

```haskell
class Printable a where
  toString :: a -> String

data Foo = Foo | Bar

instance Printable Foo where
  toString Foo = "Foo"
  toString Bar = "Bar"
```

использование:

```haskell
greeting :: Printable a => a -> String
greeting x = "Hello, " ++ toString x ++ "!!"

greeting Foo -- "Hello, Foo!!"
greeting Bar -- "Hello, Bar!!"
```

## type classes

### Eq
```haskell
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)

    -- minimal complete definition: (==) or (/=)
```

### Ord
```haskell
data Ordering = LT | EQ | GT

class Eq a => Ord a where
    compare :: a -> a -> Ordering
    (<) :: a -> a -> Bool
    (<=) :: a -> a -> Bool
    (>) :: a -> a -> Bool
    (>=) :: a -> a -> Bool

    compare x y
        | x == y = EQ
        | x <= y = LT
        | otherwise = GT
    
    x <= y = compare x y /= GT
    x < y = compare x y == LT
    x >= y = compare x y /= LT
    x > y = compare x y == GT

    -- minimal complete definition: compare or (<=)
```

### Num
```haskell
class Num a where
    (+), (-), (*) :: a -> a -> a
    negate :: a -> a
    abs :: a -> a
    signum :: a -> a
    fromInteger :: Integer -> a

    x - y = x + negate y
    negate x = 0 - x
    abs x
        | x >= 0 = x
        | otherwise = negate x
    signum x
        | x == 0 = 0
        | x > 0 = 1
        | otherwise = -1

    -- minimal complete definition: (+), (*), abs, signum, fromInteger
```

### Show 
```haskell
class Show a where
    show :: a -> String
    -- minimal complete definition: show
```

### Read
```haskell
class Read a where
    read :: String -> a
    -- minimal complete definition: read
```

`Read` при ошибках парсинга выбрасывает `runtime exception`. Чтобы избежать этого, можно использовать `readMaybe/readEither` из `Text.Read`:

```haskell
readMaybe :: Read a => String -> Maybe a
readEither :: Read a => String -> Either String a
```

## undefined

`undefined` - это специальное значение, которое может быть использовано в качестве значения любого типа

## deriving

`deriving` - это специальный синтаксис для создания инстансов классов типов. Он позволяет автоматически создавать инстансы для структур данных, которые не имеют параметров.

```haskell
data Foo = Foo | Bar deriving (Eq, Show)
```


## XGeneralizedNewtypeDeriving

`XGeneralizedNewtypeDeriving` - это расширение, которое позволяет автоматически создавать инстансы классов типов для newtype.

```haskell
{-# LANGUAGE GeneralizedNewtypeDeriving #-}

newtype Size = Size Int deriving (Eq, Show, Num, Ord)
```

## XOverloadedStrings

`-XOverloadedStrings` - это расширение, которое позволяет использовать строки в качестве значений типов `String` и `Text`.


## Modules cheatsheet

```haskell
module MyModule
  ( foo
  , bar
  ) where

foo :: Int
foo = 42

bar :: String
bar = "Hello, world!"
```





