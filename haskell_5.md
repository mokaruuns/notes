# Lecture 5: Monads

Содержание:
- Typed holes
- What is Monad?
- Monad type class
- Monad laws
- State monad
- Reader monad
- Maybe as example, philosophy about null-safety
- Either monad instance
- Monad laws

Presentation: http://slides.com/fp-ctd/lecture-5-2019#/

## Typed holes


## What is Monad?

`Монада` - это абстракция, которая позволяет выразить программу в виде последовательности вычислений, которые зависят от результатов предыдущих вычислений.

## Monad type class

```haskell
class Applicative m => Monad m where
    (>>=) :: m a -> (a -> m b) -> m b -- bind/стрелка Клейсли
    return :: a -> m a -- pure/return
```

### Meybe monad instance

```haskell
instance Monad Maybe where
    return :: a -> Maybe a
    return = Just

    (>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
    Nothing >>= _ = Nothing
    Just x  >>= f = f x
```

example:

```haskell
Just 5 >>= \x -> Just (x + 1) -- Just 6
Nothing >>= \x -> Just (x + 1) -- Nothing
Just 3 >>= \x -> Just (x + 1) >>= \y -> Just (y + 1) -- Just 5
```


## Identity monad

```haskell
newtype Identity a = Identity { runIdentity :: a }

instance Monad Identity where
    return :: a -> Identity a
    return = Identity

    (>>=) :: Identity a -> (a -> Identity b) -> Identity b
    Identity x >>= f = f x
```

## Either monad

```haskell
data Either a b = Left a | Right b

instance Monad (Either e) where
    return :: a -> Either e a
    return = Right

    (>>=) :: Either e a -> (a -> Either e b) -> Either e b
    Left e  >>= _ = Left e
    Right x >>= f = f x
```

## Monad composition

```haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
(<=<) :: Monad m => (b -> m c) -> (a -> m b) -> (a -> m c)
(>=>) :: Monad m => (a -> m b) -> (b -> m c) -> (a -> m c)
```

## instance Monad list

```haskell
instance Monad [] where
    return :: a -> [a]
    return x = [x]

    (>>=) :: [a] -> (a -> [b]) -> [b]
    xs >>= f = concatMap f xs
```

## Monad Zen

Аналог const в монадах:

```haskell
(>>) :: Monad m => m a -> m b -> m b
m >> k = m >>= \_ -> k

ghci> Just 3 >> Just 4
Just 4
ghci> Just 3 >> Nothing
Nothing
ghci> Nothing >> Just 4 -- special case
Nothing
```

## Joining monads

```haskell
join :: Monad m => m (m a) -> m a
join x = x >>= id
```

## Functions on monads

```haskell
-- fmap для монад
liftM :: Monad m => (a -> b) -> m a -> m b
liftM f m = m >>= \x -> return (f x)
```

```haskell
liftM2 :: Monad m => (a -> b -> c) -> m a -> m b -> m c
liftM2 f m1 m2 = m1 >>= \x1 -> m2 >>= \x2 -> return (f x1 x2)```

```haskell
(||^) :: Monad m => m Bool -> m Bool -> m Bool
(||^) m1 m2 = m1 >>= \x1 -> if x1 then return True else m2

(&&^) :: Monad m => m Bool -> m Bool -> m Bool
(&&^) m1 m2 = m1 >>= \x1 -> if x1 then m2 else return False
```

## Monad laws

```haskell
1. return x >>= f = f x -- left identity
2. m >>= return = m -- right identity
3. (m >>= f) >>= g = m >>= (\x -> f x >>= g) -- associativity
```

## Writer monad

`Writer` - это монада, которая позволяет вести лог в процессе вычислений.

```haskell
newtype Writer w a = Writer { runWriter :: (a, w) } -- a - результат, w - лог

instance (Monoid w) => Monad (Writer w) where
    return :: a -> Writer w a
    return x = Writer (x, mempty)

    (>>=) :: Writer w a -> (a -> Writer w b) -> Writer w b
    Writer (x, v) >>= f = let Writer (y, v') = f x in Writer (y, v <> v')


tell :: (Monoid w) => w -> Writer w ()
tell w = Writer ((), w)

execWriter :: Writer w a -> w
execWriter m = snd (runWriter m)

writer :: (a, w) -> Writer w a -- использовать для создания Writer
writer (x, v) = Writer (x, v)
```

## Reader monad

`Reader` - это монада, которая позволяет передавать контекст в процессе вычислений.

```haskell



