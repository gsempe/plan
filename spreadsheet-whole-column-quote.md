---
date: 2019-06-17
tags: spreadsheet csv
---

With Google spreadsheet `ArrayFormula` function it's possible to apply a formula to a whole column. But to quote all cells of a column into another column the special syntax with 4 quotes has to be used

For instance, the formula below copy and quote the column H

```
=ArrayFormula("""" & H1:H & """")
```




Reference:

[How to Copy a Formula Down an Entire Column in Google Sheets](https://www.labnol.org/internet/arrayformula-copy-formulas-in-entire-column/29711/)