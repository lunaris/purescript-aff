## Module Control.Monad.Aff

#### `Aff`

``` purescript
data Aff :: # ! -> * -> *
```

An asynchronous computation with effects `e`. The computation either
errors or produces a value of type `a`.

This is moral equivalent of `ErrorT (ContT Unit (Eff e)) a`.

##### Instances
``` purescript
(Semigroup a) => Semigroup (Aff e a)
(Monoid a) => Monoid (Aff e a)
Functor (Aff e)
Apply (Aff e)
Applicative (Aff e)
Bind (Aff e)
Monad (Aff e)
MonadEff e (Aff e)
MonadError Error (Aff e)
Alt (Aff e)
Plus (Aff e)
Alternative (Aff e)
MonadPlus (Aff e)
MonadRec (Aff e)
MonadCont (Aff e)
```

#### `PureAff`

``` purescript
type PureAff a = forall e. Aff e a
```

A pure asynchronous computation, having no effects other than
asynchronous computation.

#### `Canceler`

``` purescript
newtype Canceler e
  = Canceler (Error -> Aff e Boolean)
```

A canceler is asynchronous function that can be used to attempt the
cancelation of a computation. Returns a boolean flag indicating whether
or not the cancellation was successful. Many computations may be composite,
in such cases the flag indicates whether any part of the computation was
successfully canceled. The flag should not be used for communication.

##### Instances
``` purescript
Semigroup (Canceler e)
Monoid (Canceler e)
```

#### `cancel`

``` purescript
cancel :: forall e. Canceler e -> Error -> Aff e Boolean
```

Unwraps the canceler function from the newtype that wraps it.

#### `cancelWith`

``` purescript
cancelWith :: forall e a. Aff e a -> Canceler e -> Aff e a
```

This function allows you to attach a custom canceler to an asynchronous
computation. If the computation is canceled, then the custom canceler
will be run along side the computation's own canceler.

#### `launchAff`

``` purescript
launchAff :: forall e a. Aff e a -> Eff (err :: EXCEPTION | e) Unit
```

Converts the asynchronous computation into a synchronous one. All values
are ignored, and if the computation produces an error, it is thrown.

Catching exceptions by using `catchException` with the resulting Eff
computation is not recommended, as exceptions may end up being thrown
asynchronously, in which case they cannot be caught.

If you do need to handle exceptions, you can use `runAff` instead, or
you can handle the exception within the Aff computation, using
`catchError` (or any of the other mechanisms).

#### `runAff`

``` purescript
runAff :: forall e a. (Error -> Eff e Unit) -> (a -> Eff e Unit) -> Aff e a -> Eff e Unit
```

Runs the asynchronous computation. You must supply an error callback and a
success callback.

#### `makeAff`

``` purescript
makeAff :: forall e a. ((Error -> Eff e Unit) -> (a -> Eff e Unit) -> Eff e Unit) -> Aff e a
```

Creates an asynchronous effect from a function that accepts error and
success callbacks. This function can be used for asynchronous computations
that cannot be canceled.

#### `makeAff'`

``` purescript
makeAff' :: forall e a. ((Error -> Eff e Unit) -> (a -> Eff e Unit) -> Eff e (Canceler e)) -> Aff e a
```

Creates an asynchronous effect from a function that accepts error and
success callbacks, and returns a canceler for the computation. This
function can be used for asynchronous computations that can be canceled.

#### `later`

``` purescript
later :: forall e a. Aff e a -> Aff e a
```

Runs the asynchronous computation off the current execution context.

#### `later'`

``` purescript
later' :: forall e a. Int -> Aff e a -> Aff e a
```

Runs the specified asynchronous computation later, by the specified
number of milliseconds.

#### `finally`

``` purescript
finally :: forall e a b. Aff e a -> Aff e b -> Aff e a
```

Compute `aff1`, followed by `aff2` regardless of whether `aff1` terminated successfully.

#### `forkAff`

``` purescript
forkAff :: forall e a. Aff e a -> Aff e (Canceler e)
```

Forks the specified asynchronous computation so subsequent computations
will not block on the result of the computation.

Returns a canceler that can be used to attempt cancellation of the
forked computation.

#### `forkAll`

``` purescript
forkAll :: forall f e a. (Foldable f) => f (Aff e a) -> Aff e (Canceler e)
```

Forks many asynchronous computation in a synchronous manner while being
stack-safe up to the selected Foldable instance.

Returns a canceler that can be used to attempt cancellation of all
forked computations.

#### `attempt`

``` purescript
attempt :: forall e a. Aff e a -> Aff e (Either Error a)
```

Promotes any error to the value level of the asynchronous monad.

#### `apathize`

``` purescript
apathize :: forall e a. Aff e a -> Aff e Unit
```

Ignores any errors.

#### `liftEff'`

``` purescript
liftEff' :: forall e a. Eff (err :: EXCEPTION | e) a -> Aff e (Either Error a)
```

Lifts a synchronous computation and makes explicit any failure from exceptions.

#### `nonCanceler`

``` purescript
nonCanceler :: forall e. Canceler e
```

A constant canceller that always returns false.


