# bqn-snippets
Useful BQN snippets and examples

Argument flag parsing
```bqn
# parse optional argument flags
# eg. bqn -a foo -b bar -c baz → {a⇐"foo"⋄b⇐"bar"⋄c⇐"baz"}
flgs←{
  a⇐∾𝕩(⊣/˜·»⍷˜)⋈"-a"
  b⇐∾𝕩(⊣/˜·»⍷˜)⋈"-b"
  c⇐∾𝕩(⊣/˜·»⍷˜)⋈"-c"
}•args 
```

Using namespaces to organise objects
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

# usage. Deconstruct is easier than deconstructing a list since order is not needed and deconstruction can be partial
# eg. a‿b‿c‿....x‿y‿z←lst
a‿c⇐y

# However updating is slighly tedious and cannot use ⌾ to update a single item if it were a list instead
# but shouldn't be an issue if the namespace is small
d←⟨4,5,7⟩
y←{a‿b‿...x‿y‿z⇐x⋄c⇐d} # Copy namespace but update c
```
