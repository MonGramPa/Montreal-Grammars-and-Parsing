---
layout: chapter
title: Notes on Daniel's parser
description: Data structures in Parser
---

# Data structures in Daniel's parser
Some notes taken last May. Eva P. 06/2017

### Recursive type structures

 $$ constituent \rightarrow  completion \rightarrow edge \rightarrow traversal \rightarrow constituent $$


 * constituent \*
    - start/end
    - list of completions

 * completion (2 types : Terminal and non-terminal)
    - edge
    - rule (from Grammar)

 * edge \*
    - start/end
    - list of traversals

 * traversal
    - previous edges
    - constituent

\* Items: (constituents and edges) The places with choices for different possible trees

#### Probabilities

* constituent
  $$ P \rightarrow \Sigma P(completions) $$

* completion
  $$ P \rightarrow \Sigma P(edge) \times P(rule) $$

* edge
  $$ P \rightarrow \Sigma P(traversals) $$

* traversal
  $$ P \rightarrow \Sigma P(constituent) \times P($$ edge it came from $$ ) $$


### Other Structures  

 * Logbook
    - everything discovered
 * Chart
    - $$ Chart \cap Agenda = \emptyset $$
    - completed items
 * Agenda
    - Before going to chart
    - incomplete items (we don't know everything about them yet)
    - priority Q
