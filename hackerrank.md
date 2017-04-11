---
layout: page
title: Contact
---

Some solutions for some [HackerRank](https://www.hackerrank.com) problems. Use
it to learn something, not to score points.

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


### Beautiful Days at the Movies

```js
function processData(input) {
  var values = input.split(' ');
  var i = parseInt(values[0]);
  var j = parseInt(values[1]);
  var k = parseInt(values[2]);
  var result = 0;
  for(var day=i; day<=j; day++) {
    if (Math.abs(day-parseInt((''+day).split('').reverse().join(''))) % k == 0) {
      result++;
    }
  }
  console.log(result);
}
```


### Viral Advertising

```js
function processData(input) {
  var n = parseInt(input);
  var reached = 5;
  var liked = 0;
  for(var i=0; i<n; i++) {
    var new_liked = Math.floor(reached/2);
    liked += new_liked;
    reached = new_liked*3;
  }
  console.log(liked);
} 
```

<<<<<<< HEAD
### Absolute Permutation

```js
function main() {
  var t = parseInt(readLine());
  for(var a0 = 0; a0 < t; a0++){
    var n_temp = readLine().split(' ');
    var n = parseInt(n_temp[0]);
    var k = parseInt(n_temp[1]);
    var result = [];
    var used = {};
    for(var i=1; i<=n; i++) {
      // abs(posi-i) = k
      // posi = k + i or posti = i-k
      var valid = [];
      if (i-k>0) {
        valid.push(i-k);
      }
      if (i+k<=n) {
        valid.push(i+k);
      }
      valid.sort(function(a, b) { return a-b});
      var found = false;
      for(var j=0; j<valid.length; j++) {
        if (used[valid[j]] == null) {
          result.push(valid[j]);
          used[valid[j]] = true;
          found = true;
          break;
        }
      }
      if (found === false) {
        result = null;
        break;
      }
    }
    if (result == null) {
      console.log(-1);
    } else {
      console.log(result.join(' '));
=======
### Save the Prisoner!

```js
function processData(input) {
  var lines = input.split('\n');
  var T = parseInt(lines[0]);
  for(var t=0; t<T; t++) {
    var data = lines[t+1].split(' ');
    var N = parseInt(data[0]); // number of prisoners
    var M = parseInt(data[1]); // number of sweets
    var S = parseInt(data[2]); // first prisoner
    
    var rest = M % N;
    var result = (S+rest-1)%N;
    if (result == 0) { result = N; }
    console.log(result);
  }
} 
```

### Circular Array Rotation

```js
function main() {
  var n_temp = readLine().split(' ');
  var n = parseInt(n_temp[0]); // size array
  var k = parseInt(n_temp[1]); // number of rotation
  var q = parseInt(n_temp[2]); // number of queries
  a = readLine().split(' ');
  a = a.map(Number);
  for(var a0 = 0; a0 < q; a0++){
    var m = parseInt(readLine());
    var key = (m-k)%n;
    if (key < 0) { key += n }
    console.log(a[key])
  }
}

```

### Sequence Equation

```js
process.stdin.on("end", function () {
  var lines = input.split('\n');
  var map = {}; // p(x) -> x
  var data = lines[1].split(' ');
  for(var i=0; i<data.length; i++) {
    map[data[i]] = i+1;
  }
  for(var i=0; i<data.length; i++) {
    console.log(map[map[i+1]])
  }
});
```

### Jumping on the clouds: Revisited

```js
function main() {
  var n_temp = readLine().split(' ');
  var n = parseInt(n_temp[0]);
  var k = parseInt(n_temp[1]);
  c = readLine().split(' ');
  c = c.map(Number);

  var position = 0;
  var energy = 100;
  while(energy > 0) {
    position += k;
    position = position % n;
    if (c[position % n] == 1) {
      energy -= 2;
    }
    energy--;
    
    if (position == 0) {
      break;
    }
  }
  console.log(energy);
}
```

### Append and Delete

```js
function main() {
  var s = readLine().split('');
  var t = readLine().split('');
  var k = parseInt(readLine());
  // deletions or insertions required to make the strings the same length.
  var required_operation = Math.abs(t.length-s.length);
  if (k < required_operation) {
    console.log('No');
    return;
  }
  var operation_left = k-required_operation;
  var result = true;
  if (operation_left % 2 == 1) {
    // We have an odd number of operation left.
    // This can match only if we can delete everything and do one no-op deletion.
    if (Math.min(s.length, t.length) > Math.floor((operation_left)/2)) {
      console.log('No');
      return;
    }
  }
  for(var i=0; i<Math.min(s.length, t.length)-Math.floor((operation_left)/2); i++) {
    if (s[i] != t[i]) {
      result = false;
      break;
    }
  }
  console.log(result ? 'Yes' : 'No')
}
```

### Non-Divisible Subset

```js
function processData(input) {
  var lines = input.split('\n');
  var k = parseInt(lines[0].split(' ')[1])
  var a = lines[1].split(' ').map(Number)

  var mods = {};
  for(var i=0; i<a.length; i++) {
    var rest = a[i] % k;
    if (mods[rest] == null) {
      mods[rest] = 1;
    } else {
      mods[rest]++;
    }
  }
  var subset_count = {};
  for(var key in mods) {
    if (k == 2*key || key == 0) {
      subset_count[key] = 1;
      continue;
    }

    var sub_key;
    if (key > k/2) {
      sub_key = Math.abs(key-k)
    } else {
      sub_key = key;
    }

    if (subset_count[sub_key] == null) {
      subset_count[sub_key] = mods[key];
    } else if (subset_count[sub_key] < mods[key]) {
      subset_count[sub_key] = mods[key];
    }
  }

  var result = 0;
  for(var key in subset_count) {
    result += subset_count[key]
  }
  console.log(result);
}
```

### Repeated String

```js
function main() {
  var s = readLine();
  var n = parseInt(readLine());
  var letters = s.split('');
  var a_per_word = 0;
  for(var i=0; i<letters.length; i++) {
    if (letters[i] == 'a') {
      a_per_word++;
    }
  }
  var rest = n % s.length;
  var num_full_word = (n-rest)/s.length
  var result = num_full_word * a_per_word;
  for(var i=0; i<rest; i++) {
    if (letters[i] == 'a') {
      result++;
    }
  }
  console.log(result);
}
```

### Jumping on the clouds

```js
function main() {
  var n = parseInt(readLine());
  c = readLine().split(' ');
  c = c.map(Number);
  var current_position = 0;
  var num_jump = 0;
  while (current_position < n-1) {
    if (c[current_position+2] == 0) {
      current_position += 2;
    } else {
      current_position += 1;
    }
    num_jump++;
  }
  console.log(num_jump)
}
```

### Equalize the Array 

```js
function processData(input) {
  var values = input.split('\n')[1].split(' ');
  var map_values = {};
  for(var i=0; i<values.length; i++) {
    if (map_values[values[i]] == null) {
      map_values[values[i]] = 1;
    } else {
      map_values[values[i]]++;
    }
  }
  var max = null;
  for(var key in map_values) {
    if (max == null || map_values[key] > max) {
      max = map_values[key];
    }
  }
  console.log(values.length-max)
}
```

### Organizing Containers of Balls

```js
function main() {
  var q = parseInt(readLine());
  for(var a0 = 0; a0 < q; a0++){
    var n = parseInt(readLine());
    var M = []; // [container][ball]
    for(M_i = 0; M_i < n; M_i++){
       M[M_i] = readLine().split(' ');
       M[M_i] = M[M_i].map(Number);
    }
    var num_balls_per_container = []; // container -> num balls
    var num_balls_per_type = []; // type -> num balls
    for(var i=0; i<n; i++) {
      var balls_in_container = 0;
      var balls_of_type = 0;
      for(var j=0; j<n; j++) {
        balls_in_container += M[i][j];
        balls_of_type += M[j][i];
      }
      num_balls_per_container.push(balls_in_container);
      num_balls_per_type.push(balls_of_type);
    }
    num_balls_per_container.sort(function(a,b) { return a - b; })
    num_balls_per_type.sort(function(a,b) { return a  b; })
    var result = "Possible"
    for(var i=0; i<n; i++) {
      if (num_balls_per_container[i] != num_balls_per_type[i]) {
        result = 'Impossible'
        break;
      }
    }
    console.log(result);
  }
}
```

### Beautiful Triplets

```js
function processData(input) {
  var lines = input.split('\n');
  var d = parseInt(lines[0].split(' ')[1])
  var numbers = lines[1].split(' ').map(Number)
  var map = {} // ai -> i
  for(var i=0; i<numbers.length; i++) {
    map[numbers[i]] = parseInt(numbers[i]);
  }
  var result = 0;
  //for(var i=0; i<numbers.length; i++) {
  for(var key in map) {
    var ai = map[key]
    if (map[d+ai] != null) {
      aj = map[d+ai]
      if (map[d+aj] != null) {
        result++;
      }
    }
  }
  console.log(result);
}
```

### Minimum Distances 

```js
function main() {
  var n = parseInt(readLine());
  A = readLine().split(' ');
  A = A.map(Number);
  var map = {};
  var min = null;
  for(var i=0; i<A.length; i++) {
    if (map[A[i]] != null) {
      min = min == null ? i-map[A[i]] : Math.min(min, i-map[A[i]]);
    }
    map[A[i]] = i;
  }
  if (min == null) {
    console.log(-1);
  } else {
    console.log(min);
  }
}
```

### Flatland Space Stations

```js
function main() {
  var n_temp = readLine().split(' ');
  var n = parseInt(n_temp[0]);
  var m = parseInt(n_temp[1]);
  c = readLine().split(' ');
  c = c.map(Number);
  c.sort(function(a, b) { return a - b});
  var max = c[0];
  for(var i=1; i<c.length; i++) {
    max = Math.max(max, Math.floor((c[i]-c[i-1])/2));
  }
  max = Math.max(max, n-1-c[c.length-1])
  console.log(max);
}
```

### Fair Rations

```js
function main() {
  var N = parseInt(readLine());
  B = readLine().split(' ');
  B = B.map(Number);
  var result = 0;
  for(var i=0; i<B.length-1; i++) {
    if (i == B.length-2) {
      if (B[i] % 2 == 0 && B[i+1] % 2 == 0) {
        console.log(result);
      } else if (B[i] % 2 == 1 && B[i+1] % 2 == 1) {
        console.log(result+1);
      } else {
        console.log('NO');
      }
    } else {
      if (B[i] % 2 == 1) {
        //B[i]++;
        B[i+1]++;
        result += 2;
      }
    }
  }
}
```

### Happy Ladybugs

```js
function main() {
  var Q = parseInt(readLine());
  for(var a0 = 0; a0 < Q; a0++){
    var n = parseInt(readLine());
    var b = readLine().split('');
    var map = {};
    var has_empty_cell = false;
    for(var i=0; i<b.length; i++) {
      if (b[i] == '_') {
        has_empty_cell = true;
      } else {
        if (map[b[i]] == null) {
          map[b[i]] = 1;
        } else {
          map[b[i]]++;
        }
      }
    }
    if (has_empty_cell) {
      var result = 'YES'
      for(var key in map) {
        if (map[key] == 1) {
          result = 'NO'
        }
      }
      console.log(result);
    } else {
      var result = 'YES'
      var last = b[0];
      var count = 1;
      for(var i=1; i<b.length; i++) {
        if (b[i] == last) {
          count++;
        } else {
          if (count == 1) {
            result = 'NO'
          } else {
            last = b[i];
            count = 1;
          }
        }
      }
      if (count == 1) {
        result = 'NO';
      }
      console.log(result)
    }
  }
}
```

### Strange Counter

```js
function main() {
  var t = parseInt(readLine());
  var start = 3;
  var position = 1;
  while(true) {
    // position             -> start
    // position + 1         -> start - 1
    // position + start - 1 -> 1
    if (position+start-1 < t) {
      position = position+start;
      start *= 2
    } else {
      console.log(start-t+position);
      return;
>>>>>>> 42d90dfca2411af7b7e2f1df603416323ce56197
    }
  }
}
```
