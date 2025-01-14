# network-wait

A lightweight Haskell library for waiting on networked services to become available. This is useful if you are e.g. building a web application which relies on a database server to be available, but which may not be immediately available on application startup.

## Example

All functions provided by this library work with retry policies from [`Control.Retry`](https://hackage.haskell.org/package/retry) which provide good control over the retry behaviour. To wait for a PostgreSQL server to become available on the same machine:

```haskell
import Control.Retry
import Network.Wait

main :: IO ()
main = do
    waitTcp retryPolicyDefault "localhost" "5432"
    putStrLn "Yay, the server is available!"
```

The haddock documentation for this library contains more examples.

## Example: PostgreSQL

The `network-wait` package can be compiled with the `network-wait:postgres` flag (e.g. `stack build --flag network-wait:postgres`) which enables support for checking the readiness of PostgreSQL servers specifically. Unlike the functions in the `Network.Wait` module, which only check that connections can be established, the functions in `Network.Wait.PostgreSQL` also check that a PostgreSQL server is ready to accept commands. To wait for a PostgreSQL server to become available and accept commands:

```haskell
import Control.Retry (retryPolicyDefault)
import Database.PostgreSQL.Simple (defaultConnectInfo)
import Network.Wait.PostgreSQL (waitPostgreSQL)

main :: IO ()
main = do
    waitPostgreSQL retryPolicyDefault defaultConnectInfo
    putStrLn "Yay, the PostgreSQL server is ready to accept commands!"
```

Internally, this uses `postgresql-simple` to connect to the specified server (`defaultConnectInfo` in the example above) and send a `SELECT 1;` query. If the query is answered correctly, we consider the server to be in a state ready to accept commands.

The `Network.Wait.PostgreSQL` module is gated behind the `network-wait:postgres` flag so that the PostgreSQL-specific dependencies are only required when PostgresSQL support is required by users of this library.

## See also

- [wait-for](https://github.com/eficode/wait-for) is a popular shell script with the same objectives as this library.
- The [port-utils](https://hackage.haskell.org/package/port-utils) package has some functions for waiting on TCP servers to become available, with a fixed timeout.
