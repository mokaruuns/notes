# Lecture 4: Basic typeclasses: Monoid. Functor. Applicative

Содержание:
- Math in programming
    - Semigroup and Monoid
    - A lot of examples
- foldr and foldl
- Foldable type class
- Functor
- Applicative
- liftAN & Applicative style programming
- Alternative
- List comprehension syntax sugar
- Traversable type class (and instances for Maybe, List)
- Automatic deriving
    - -XDeriveFunctor
    - -XDeriveFoldable
    - -XDeriveTraversable
- Phantom types
- Type extensions:
    - -XScopedTypeVariables
    - -XTypeApplications
    - -XAllowAmbiguousTypes

Presentation: http://slides.com/fp-ctd/lecture-4#/

## Math in programming

### Semigroup 
`Semigroup` -  это бинарная ассоциативная операция над элементами определенного типа. В результате получается элемент того же типа.

```haskell
class Semigroup a where
    (<>) :: a -> a -> a
    sconcat :: NonEmpty a -> a
    stimes :: Integral b => b -> a -> a
    -- minimal complete definition: (<>)

-- Пример для типа [a]
instance Semigroup [a] where
    (<>) = (++)
    sconcat = foldr1 (++)
    stimes = replicate
```

### example with two instances

```haskell
-- this code is't compilable
instance Semigroup Int where
    (<>) = (+)
    (<>) = (*)

-- this code is compilable

newtype Sum a = Sum { getSum :: a }
newtype Product a = Product { getProduct :: a }

instance Num a => Semigroup (Sum a) where
    Sum x <> Sum y = Sum (x + y)

instance Num a => Semigroup (Product a) where
    Product x <> Product y = Product (x * y)


3 <> 4 :: Sum Int -- Sum {getSum = 7}
3 <> 4 :: Product Int -- Product {getProduct = 12}
```

### Monoid

`Monoid` - это `Semigroup` с нейтральным элементом.

```haskell
class Semigroup a => Monoid a where
    mempty :: a
    mappend :: a -> a -> a
    mconcat :: [a] -> a
    -- minimal complete definition: mempty
-- Пример для типа [a]

instance Monoid [a] where
    mempty = []
    mappend = (++)
    mconcat = foldr (++) []
```

## foldr and foldl

`foldr` - это функция, которая принимает на вход бинарную функцию, начальное значение и список. Функция применяется к каждому элементу списка слева направо, начиная с начального значения.

```haskell
foldr :: (a -> b -> b) -> b -> [a] -> b
foldr f z []     = z
foldr f z (x:xs) = f x (foldr f z xs)
```

`foldl` - это функция, которая принимает на вход бинарную функцию, начальное значение и список. Функция применяется к каждому элементу списка справа налево, начиная с начального значения.

```haskell
foldl :: (b -> a -> b) -> b -> [a] -> b
foldl f z []     = z
foldl f z (x:xs) = foldl f (f z x) xs
```

- `foldr` не требует хранить промежуточные значения, а `foldl` требует
- на бесконечных списках `foldr` работает, а `foldl` нет
- `foldr` работает быстрее, чем `foldl`

## Foldable type class

Общий вид foldr и foldl для типов, которые являются экземплярами `Foldable`:

```haskell
foldr :: Foldable t => (a -> b -> b) -> b -> t a -> b
foldl :: Foldable t => (b -> a -> b) -> b -> t a -> b
```

`Foldable` - это тип, который можно свернуть в одно значение.

```haskell
class Foldable t where -- :k t = * -> *
    fold :: Monoid m => t m -> m
    foldMap :: Monoid m => (a -> m) -> t a -> m
    foldr :: (a -> b -> b) -> b -> t a -> b
    -- minimal complete definition: foldr or foldMap
```


```haskell
instance Foldable [] where
    foldr :: (a -> b -> b) -> b -> [a] -> b
    foldr f z []     = z
    foldr f z (x:xs) = f x (foldr f z xs) 
```


## Functor

`Functor` - класс типов, который принимает типовой параметр и определяет единственную функцию `fmap`. Используется для преобразования контейнеров.

```haskell
class Functor f where -- :k f = * -> *
    fmap :: (a -> b) -> f a -> f b

-- functor laws:
-- fmap id = id
-- fmap (f . g) = fmap f . fmap g
```

instance для списка:
```haskell
instance Functor [] where
    fmap :: (a -> b) -> [a] -> [b]
    fmap f []     = []
    fmap f (x:xs) = f x : fmap f xs
``` 

instance для Maybe:
```haskell
instance Functor Maybe where
    fmap :: (a -> b) -> Maybe a -> Maybe b
    fmap f Nothing  = Nothing
    fmap f (Just x) = Just (f x)
```

`<$>` - это infix-оператор для `fmap`:
```haskell
infix 4 <$>
(<$>) :: Functor f => (a -> b) -> f a -> f b
(<$>) = fmap

-- примеры:
fmap (+1) [1,2,3] -- [2,3,4]
(+1) <$> [1,2,3] -- [2,3,4]
```

```haskell
:k (-->)
(->) :: * -> * -> *
(->) a b = a -> b
```

instance для функции:
```haskell
instance Functor ((->) a) where
    fmap :: (b -> c) -> (a -> b) -> (a -> c)
    fmap = (.)
```

## Applicative

`Applicative` - это класс типов, который принимает типовой параметр и определяет две функции: `pure` и `<*>`. 

`Applicative laws`:
- `pure id <*> v = v` - identity - 
- `pure (.) <*> u <*> v <*> w = u <*> (v <*> w)` - composition
- `pure f <*> pure x = pure (f x)` - homomorphism
- `u <*> pure y = pure ($ y) <*> u` - interchange

```haskell
class Functor f => Applicative f where -- :k f = * -> *
    pure :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
    liftA2 :: (a -> b -> c) -> f a -> f b -> f c

    -- minimal complete definition: pure and <*>
```

## Meybe applicative

`Maybe` - это тип, который может содержать значение или не содержать.
```haskell
instance Applicative Maybe where
    pure :: a -> Maybe a
    pure = Just

    (<*>) :: Maybe (a -> b) -> Maybe a -> Maybe b
    Nothing <*> _ = Nothing
    (Just f) <*> something = fmap f something

    -- minimal complete definition: pure and <*>
```

## List applicative

`[]` - это тип, который может содержать несколько значений.
```haskell
instance Applicative [] where
    pure :: a -> [a]
    pure x = [x]

    (<*>) :: [a -> b] -> [a] -> [b]
    fs <*> xs = [f x | f <- fs, x <- xs]

    -- minimal complete definition: pure and <*>
```

## Arrow applicative
```haskell
instance Applicative ((->) a) where
    pure :: b -> (a -> b)
    pure = const

    (<*>) :: (a -> b -> c) -> (a -> b) -> (a -> c)
    f <*> g = \x -> f x (g x)

    -- minimal complete definition: pure and <*>
```

## liftAN & Applicative style programming

`liftA2` - это функция, которая принимает на вход бинарную функцию, два контейнера и возвращает контейнер, который содержит результат применения бинарной функции к элементам контейнеров.

```haskell
liftA2 :: Applicative f => (a -> b -> c) -> f a -> f b -> f c
liftA2 f a b = f <$> a <*> b
```

`liftAN` - аналог `liftA2`, но для функций с большим количеством аргументов.

```haskell
liftA3 :: Applicative f => (a -> b -> c -> d) -> f a -> f b -> f c -> f d
liftA3 f a b c = f <$> a <*> b <*> c
```

`Applicative style programming` - это стиль программирования, в котором мы используем `Applicative` для преобразования контейнеров.

## Alternative

`Alternative` - это класс типов, который принимает типовой параметр и определяет две функции: `empty` и `<|>`. Моноид для апликативных функторов.

```haskell
class Applicative f => Alternative f where -- :k f = * -> *
    empty :: f a
    (<|>) :: f a -> f a -> f a

    -- minimal complete definition: empty and <|>
```

### Maybe alternative

```haskell
instance Alternative Maybe where
    empty :: Maybe a
    empty = Nothing

    (<|>) :: Maybe a -> Maybe a -> Maybe a
    Nothing <|> r = r
    l <|> _ = l

    -- minimal complete definition: empty and <|>
```

### List alternative

```haskell
instance Alternative [] where
    empty :: [a]
    empty = []

    (<|>) :: [a] -> [a] -> [a]
    (<|>) = (++)

    -- minimal complete definition: empty and <|>
```

## Traversable

`Traversable` - это класс типов, который принимает типовой параметр и определяет две функции: `traverse` и `sequenceA`. 

```haskell
class (Functor t, Foldable t) => Traversable t where -- :k t = * -> *
    traverse :: Applicative f => (a -> f b) -> t a -> f (t b)
    sequenceA :: Applicative f => t (f a) -> f (t a)

    -- minimal complete definition: traverse
```

`traverse` - это функция, которая принимает на вход функцию, которая преобразует элементы контейнера в контейнер, и контейнер, а возвращает контейнер, который содержит результат применения функции к элементам контейнера.

instance для Maybe:
```haskell
instance Traversable Maybe where
    traverse :: Applicative f => (a -> f b) -> Maybe a -> f (Maybe b)
    traverse f Nothing = pure Nothing
    traverse f (Just x) = Just <$> f x

    sequenceA :: Applicative f => Maybe (f a) -> f (Maybe a)
    sequenceA Nothing = pure Nothing
    sequenceA (Just x) = Just <$> x
```

instance для List:
```haskell
instance Traversable [] where
    traverse :: Applicative f => (a -> f b) -> [a] -> f [b]
    traverse f [] = pure []
    traverse f (x:xs) = (:) <$> f x <*> traverse f xs

    sequenceA :: Applicative f => [f a] -> f [a]
    sequenceA [] = pure []
    sequenceA (x:xs) = (:) <$> x <*> sequenceA xs
```

## Phantom types

`Phantom types` - это типы, которые не используются в реализации типа.



## Type extensions

undefined





