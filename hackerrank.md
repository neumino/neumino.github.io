---
layout: page
title: Contact
---

### Climbing the Leaderboard

```js
function main() {
  var n = parseInt(readLine());
  scores = readLine().split(' ');
  scores = scores.map(Number);
  var m = parseInt(readLine());
  alice = readLine().split(' ');
  alice = alice.map(Number);
  // your code goes here
  
  // Create a list of scores without duplicate. This implicitly give the rank.
  var rank_score = [];
  rank_score.push(scores[0])
  for(var i=1; i<scores.length; i++) {
    if (scores[i] != rank_score[rank_score.length-1]) {
      rank_score.push(scores[i]);
    }
  }
  
  var aliceRank = rank_score.length; // Alice is last.
  for(var i=0; i<alice.length; i++) { // Loop through all Alice's score.
    var next = aliceRank-1;
    // As long as Alice's score is greater that the next in rank, move Alice's
		// rank (decrease it).
    while(next >= 0 && alice[i] >= rank_score[next]) { 
      next--;
    }
    aliceRank = next+1;
    console.log(aliceRank + 1); // +1 because rank are 1-indexed.
  }
}
```

### The Hurdle Race

```js
function main() {
  var n_temp = readLine().split(' ');
  var n = parseInt(n_temp[0]);
  var k = parseInt(n_temp[1]);
  height = readLine().split(' ');
  height = height.map(Number);
  // your code goes here
  var max_height = Math.max.apply(Math, height)
  console.log(Math.max(0, max_height-k));
}
```

### Designer PDF Viewer

```js
function main() {
  h = readLine().split(' ');
  h = h.map(Number);
  var alphabet = 'abcdefghijklmnopqrstuvwxyz';
  var chars = alphabet.split('');
  var letter_height = {}; // map char -> height
  for(var i=0; i<alphabet.length; i++) {
    letter_height[alphabet[i]] = h[i];
  }
  var word = readLine();
  var letters = word.split('');
  var max_height = 0;
  for(var i=0; i<letters.length; i++) {
    if (max_height < letter_height[letters[i]]) {
      max_height = letter_height[letters[i]];
    }
  }
  console.log(max_height*letters.length)
}
```
