# Go

## Print struct

At this point, it's magic to my, no idea how it works.

```go
fmt.Printf("%+v\n", struct)
```

## Comma OK ideom

```go
if entry, ok := thing[index]; ok {
    return entry
}
```

`entry` will have the value of the lookup, ok will be `true` or `false`.

`entry` will also be available in future branches, i.e. an `else` following.
