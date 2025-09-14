# bqn-snippets
Useful BQN snippets and examples

Argument parsing for optional flags
eg. `bqn -a foo -b bar -c baz → {a⇐"foo"⋄b⇐"bar"⋄c⇐"baz"}`
```bqn
flgs←{
  a⇐∾𝕩(⊣/˜·»⍷˜)⋈"-a"
  b⇐∾𝕩(⊣/˜·»⍷˜)⋈"-b"
  c⇐∾𝕩(⊣/˜·»⍷˜)⋈"-c"
}•args 
```

Using namespaces to organise collections
```bqn
x←{
    a⇐⟨1,2,3⟩
    b⇐⟨1,2,3⟩
    c⇐⟨1,2,3⟩
    ...
    x⇐...
    y⇐...
    z⇐...
}

# usage. Deconstruct is easier than when using a list since order is not needed and can be partial
# eg. a‿b‿c‿....x‿y‿z←lst
a‿c⇐y

# However updating is slighly tedious and cannot use ⌾ to update a single item if it were a list
# but shouldn't be an issue if the namespace is small
d←⟨4,5,7⟩
y←{a‿b‿...x‿y‿z⇐x⋄c⇐d} # Copy namespace but update c
```
