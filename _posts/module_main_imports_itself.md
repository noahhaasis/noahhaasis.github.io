---
layout: post
title:  "Haskell: module 'Main' imports itself"
---

# Problem:
When importing "Main" in Spec.hs, ghc complains:
```
Module imports form a cycle:
  module ‘Main’ (Spec.hs) imports itself
```

# Solution:
If you don't name your module ghc defaults to "Main", so you have to declare a module at the top of the file.
```
module Test where
import Main hiding (main)
```
