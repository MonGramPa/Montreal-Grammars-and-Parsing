---
layout: chapter
title: Notes on Daniel's parser
description: Data structures in Parser
---

# Data structures in Daniel's parser
Some notes taken last May. Eva P. 06/2017

### Recursive type structures

 $$ constituent → completion → edge → traversal → constituent $$


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
  $$ P → Σ P(completions) $$

* completion
  $$ P → Σ P(edge) × P(rule) $$

* edge
  $$ P → Σ P(traversals) $$

* traversal
  $$ P → Σ P(constituent)× P(edge it came from) $$


### Other Structures  

 * Logbook
    - everything discovered
 * Chart
    - $$ Chart ∩ Agenda = ∅ $$
    - completed items
 * Agenda
    - Before going to chart
    - incomplete items (we don't know everything about them yet)
    - priority Q
