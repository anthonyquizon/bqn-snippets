# bqn-snippets
**Useful BQN snippets and examples**



Argument parsing for optional flags
```bqn
flgs←{
  a⇐∾𝕩(⊣/˜·»⍷˜)⋈"-a"
  b⇐∾𝕩(⊣/˜·»⍷˜)⋈"-b"
  c⇐∾𝕩(⊣/˜·»⍷˜)⋈"-c"
}•args 
# usage eg. `bqn -a foo -b bar -c baz → {a⇐"foo"⋄b⇐"bar"⋄c⇐"baz"}`
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

String formatting
```bqn
n←100000
Num ⇐ {(0⊸≤)◶⟨'-'⊸∾∘•Fmt|,•Fmt⟩(⌊0.5⊸+)⌾((10⋆4)⊸×)(-n)⌈n⌊𝕩}        # formats number eg. ¯9.999999 to -9.99
Str ⇐ (2⌊•Type)◶⟨{1∧´2=•Type¨𝕩 ? 𝕩; •Fmt 𝕩},Num,≍⟩
Fmt ⇐ { ch 𝕊 𝕩: 
    s‿e←ch⊸=¨"{}" ⋄ f←(+`s+»e)⊸⊔ch                                 # strigify fragments
    m←∾('}'=¯1↑¨f)∧'{'=1↑¨f ⋄ t←m/f ⋄ v←Str¨(≠t)↑𝕩∾⟨⟩¨↕0⌈(≠t)-(≠𝕩) # replacement values
    ∾v⌾(m⊸/)f
}

"Hello {}! {}" Fmt ⟨"world", 123⟩ # → "Hello world! 123"
```

Tree rendering
```bqn
Draw⇐{
    d←(⊢∾1+⊏⟜⥊)´⌽p←𝕩                                   # p: parent vector, d: depth vector
    w←{∾⍟(1-˜≡)(𝕨(⊣⊔˜⊏)p)⊸(⊢∾¨⊏˜)⌾((𝕩∊𝕨⊏p)⊸/)𝕩}´⌽d⊔↕≠p # order depths depending on parent groups
    n←(1+w⊏p)⊸∧˘(1+⌈´d)↑w=⌜w⊏d                         # matrix of ordered depth path cut off
    n↩(0≠∨˝n)⊸/˘n↩>(↕≠)⊸({«⍟𝕨 𝕩}˘)n                    # shift columns under parent node and remove empty space
    s←0<∊⊸∧˘n⋄e←0<(⌽∘∊∘⌽)⊸∧˘n⋄h←s+`∘-˘e⋄m←0<h∧n⋄v←s∧e  # s: start, e: ends, h: horizontals, m: middles, v: verticals 
    b←" ─┬├┐│"⊏˜(0⥊˜≠⍉n)∾1↓⌈´h‿m‿s‿e‿v×¨1+↕5           # branches - nullify root layer. 1: horizontal, 2: middle, 3: start, 4: end, 5: vertical, 
    o←∾˘(0⊸≢)◶" "‿"•"¨n                                # convert nodes to characters
    @⊣•Out˘b{𝕨∾(@+10)∾𝕩}˘o                               # combine branches and nodes and print
}

Draw 0‿0‿1‿1‿1‿2‿0‿2
# Output:
# •
# ├──┐
# •  •
# ├┬┐
# •••

```

Property based testing (TODO)
```bqn
prop⇐{
  r←•MakeRand 10
  gr←2 ⋄ gl←1.5⋄ sr←1⋄ sl←0.9 # sl: length shrink factor, gr: range factor, rs: shrink range factor, gl: length factor

  _P_←{ F _𝕣_ G m:
    j←10⋄v←@⋄rs←1⋄i←0 ⋄ t←⍉>⟨0∾gr⋆↕10, 0∾⌊gl⋆↕10⟩ # rs: result

    {𝕊: # grow
      x‿y←i⊏t ⋄ i↩i+1 ⋄ l‿h←x×¯1‿1 ⋄ p↩v
      (i<≠t)∧rs↩G F v↩l+y r.Range 1+h-l
    } •_while_ (0⊸≢) @

    {𝕊: # shrink
      v {𝕩↑˜⌊sl×≠𝕩}⍟(0.5<(r.Range 0)⋆1.2) ↩ # shrink length
      v {(⊣-sr⊸×∘×)⌾((⊑⍒|𝕩)⊸⊏) 𝕩}⍟(0⊸≠≠) ↩ # shrink lagrest delta from 0
      (j-⟜1↩)∧(1≡(G⟜F)⍟(≢⟜p) v)
    } •_while_ {(0≢𝕩)∧¬rs} @

    ⊢◶⟨
      {𝕊: •Out¨m‿"Counter example:"‿(•Fmt F p) }
      {𝕊: •Out m∾": Ok" }
    ⟩ rs
  }
}

```

Inline unit tests
```bqn
{𝕩∨´∘⍷˜⋈"test"?
  "test one" ! 1≡1
  "test two" ! 2≡2
  •Out "☺"
;·:@} •args

# run via bqn file.bqn test
# runs tests only with `test` argument
# prints ☺ when all tests pass
```


