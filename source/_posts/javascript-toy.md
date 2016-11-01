title: JavaScript Toies
date: 2016-09-09 09:50:14
tags: [javascript]
---

``` js
    function DNAStrand(dna) 
    {
      let pairs = [
        'A': 'T',
        'T': 'A',
        'G': 'C',
        'C': 'G'
      ]
      return dna.replace(/./g, char => pairs[char])
    }

    var guess = 10
    Math.floor = number => 10


    function order(words)
    {
      return words.split(' ').sort((a, b) => a.match(/\d/) - b.match(/\d/)).join(' ')
    }

```