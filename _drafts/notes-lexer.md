---
layout: post
title:  "The Unison Lexer"
---
# The Unison Lexer
# Motivation
This is not supposed to be a design document or a official documentation. This is just overview of the Unison Lexer I wrote for myself to better understand it.
# Write-up
The source code is a list of lexemes.
```haskell
-- Design principle:
--   `[Lexeme]` should be sufficient information for parsing without
--   further knowledge of spacing or indentation levels
--   any knowledge of comments
data Lexeme
  = Open String      -- start of a block
  | Semi             -- separator between elements of a block
  | Close            -- end of a block
  | Reserved String  -- reserved tokens such as `{`, `(`, `type`, `of`, etc
  | Textual String   -- text literals, `"foo bar"`
  | Backticks String -- an identifier in backticks
  | WordyId String   -- a (non-infix) identifier
  | SymbolyId String -- an infix identifier
  | Blank String     -- a typed hole or placeholder
  | Numeric String   -- numeric literals, left unparsed
  | Hash Hash        -- hash literals
  | Err Err
  deriving (Eq,Show,Ord)
```
A token consists of a payload and it's location.
```haskell
data Token a = Token {
  payload :: a,
  start :: Pos,
  end :: Pos
} deriving (Eq, Ord, Show, Functor)
```
In the lexer the payload mostly is a lexeme.
Layout is used inside the parser to keep track of blocks.
```haskell
type Layout = [(BlockName,Column)]
```
NOTE: Since the top level of a file is itself a block this can be rewritten to
```haskell
type Layout = Nes.NonEmpty (BlockName, Column)
```
some time later. There are a number of helper functions for accessing and transforming the Layout.
```haskell
top :: Layout -> Column
topBlockName :: Layout -> Maybe BlockName

pop :: [a] -> [a]
```
TODO: Explain the tree T

`lexer` is the main entry point:
```haskell
lexer :: String -> String -> [Token Lexeme]
lexer scope rem =
  let t = tree $ lexer0 scope rem
  in toList $ reorderTree reorder t
``` 
whereas `lexer0` does the heavy lifting.
There are a number of local helper functions defined in `lexer0`:
```haskell
-- skip whitespace and comments
goWhitespace :: Layout -> Pos -> [Char] -> [Token Lexeme]

popLayout :: Layout -> Pos -> [Char] -> [Token Lexeme]

-- Examine current column and pop the layout stack
-- and emit `Semi` / `Close` tokens as needed
popLayout0 :: Layout -> Pos -> [Char] -> [Token Lexeme]

-- todo: is there a reason we want this to be more than just:
-- go1 (top l + 1 : l) pos rem
-- looks for the next non whitespace, non-comment character, and
-- pushes its column onto the layout stack
pushLayout :: BlockName -> Layout -> Pos -> [Char] -> [Token Lexeme]

-- assuming we've dealt with whitespace and layout, read a token
go :: Layout -> Pos -> [Char] -> [Token Lexeme]
```
`go` does most of the matching typically associated with lexing.
Following are just helper functions and that either process Strings or implement deal with keywords
and other policy.

