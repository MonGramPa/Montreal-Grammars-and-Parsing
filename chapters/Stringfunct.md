---
layout: chapter
title: Notes on string functions for minimalist parser
description: String functions for parser side of minimalist parser
---

## String functions
by Eva P. Aug 23, 2017

#### Merge 1-2
(xy,a,b) <- (x,a) (y,b)
(xy,a,b) <- (y,b) (x,a)
#### Move 1
(xy,a,b) <- (y,x,a,b)

where y,a,b must precede x linearly and do not overlap
Because of the string function for Move 1, we can guarantee this.
#### Merge 3
(x,y,a,b) <- (x,a) (y,b)
(x,y,a,b) <- (y,b) (x,a)
#### Move 2
(x,y,a,b) <- (x,y,a,b)
