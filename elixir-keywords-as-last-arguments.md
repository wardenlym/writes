# [syntactic-sugar](https://hexdocs.pm/elixir/1.12/syntax-reference.html#syntactic-sugar)

https://hexdocs.pm/elixir/1.12/syntax-reference.html#keywords-as-last-arguments

Elixir also supports a syntax where if the last argument of a call is a keyword list then the square brackets can be skipped. This means that the following:

```elixir
config :friends, Friends.Repo,
  database: "friends_repo",
  username: "root",
  password: "root",
  hostname: "localhost",
  port: 3306
```

```elixir
config(:friends, Friends.Repo, [database: "friends_repo",
                                username: "root",
                                password: "root",
                                hostname: "localhost",
                                port: 3306])
```
