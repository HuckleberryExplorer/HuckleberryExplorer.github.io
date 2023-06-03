---
layout: default
title:  rust圣经笔记
categories: 
tag:    
---

[返回](./)

## Rust圣经

https://course.rs/about-book.html


## General naming conventions

In general, Rust tends to use `UpperCamelCase` for "type-level" constructs
(types and traits) and `snake_case` for "value-level" constructs. More
precisely, the proposed (and mostly followed) conventions are:

| Item | Convention |
| ---- | ---------- |
| Crates | `snake_case` (but prefer single word) |
| Modules | `snake_case` |
| Types | `UpperCamelCase` |
| Traits | `UpperCamelCase` |
| Enum variants | `UpperCamelCase` |
| Functions | `snake_case` |
| Methods | `snake_case` |
| General constructors | `new` or `with_more_details` |
| Conversion constructors | `from_some_other_type` |
| Local variables | `snake_case` |
| Static variables | `SCREAMING_SNAKE_CASE` |
| Constant variables | `SCREAMING_SNAKE_CASE` |
| Type parameters | concise `UpperCamelCase`, usually single uppercase letter: `T` |
| Lifetimes | short, lowercase: `'a` |