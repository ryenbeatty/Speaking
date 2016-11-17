![](cover.jpg)

^
- intro: what is fp?
  - fp is a paradigm rooted in describing what you want to achieve, not how you want to achieve it (imperative)
  - a declarative style , as opposed to imperative, means that we will write expressions, as opposed to step by step instructions.
  - avoids changing state and mutable data

---
# f(Rᴳ)
- mental models
- list operations > ```map/filter/reduce```
- currying
- function composition

^  
- this is what I will be covering
- this talk a beginner's perspective on some basic fp concepts

---
# fp in js
- first class functions
- functions can be
  - inputs
  - stored in arrays
  - assigned to variables

^
- why do fp in js?
- like haskell/lisps/erlang/scala, javascript has first class functions (or equal class functions, equal to other primitives)
  - refer to them from constants and variables
  - pass them as parameters to other functions
  - return them as results from other functions
- we will be taking advantage of this

---
# a little mathsing
![left filtered](fig1.png)

```javascript
f(x) = 2x^2 + 3

// example
f(2) // 11
```

^
- given the equation at the top
- when we graph that equation, we get the parabola drawn in blue
- we can see that for any value of x, say 2, the result is 11, corresponding to point (2, 11) on the graph
- what is 11 though? it is the return value of the f(x) function
- for every value of x we plug in, we get another y value that pairs with it as a coordinate for a point

^
- why maths and graphs?
- in math, a function always takes inputs, and always gives an output
- because fp is about embracing using functions in this mathematical sense
- in out programs, functions don't need to have a relationship to the curve on a graph

^
- in fp there is a concept of functional 'purity'
- pure functions are functions that have no side-effects
  - they don't assign to outside variables
  - they don't produce output (e.g. logging)
  - they don't interact with a database or a network
  - they don't mutate the parameters they are passed
- not using them as collections of functionality, that may/not have inputs, and may/not have return values.
- additionally, pure functions always return the same result when given the same arguments, making them consistent, reliable, and easy to test

---
# lodash-fp
- data last[^1]

[^1]: Brian Lonsdorf: [Hey Underscore, You're Doing It Wrong! (Youtube)](https://www.youtube.com/watch?v=m3svKOdZijA)

^
- throughout the talk i am going to be using lodash-fp for the example code
- as opposed to regular lodash, lodash-fp is an even more 'functional' utility library
- the key difference is that the argument order is flipped
- in lodash your data comes first, but in lodash-fp (and other utility libraries like ramda), the data comes last
- the benefits of this will become apparent a little later
- this lets us pre-load functions with arguments in a process known as currying, but we will talk more about that later

---
# list operations
### ```map/filter/reduce```

^
- these are the core collection iteration functions
- and they let us replace loops
- especially useful in the creation of processing pipelines
- with imperative code, each intermediate result in a set is stored in variables through assignment, the more of these you use, the harder to verify correctness.

---
# non-FP list operations
- `forEach(...)`
- `some(...)`
- `every(...)`

^
- designed for each function call to operate with side effects (push to array, modify object etc.)

---
# map
### `map :: (a -> b) -> [a] -> [b]`
![inline 100%](fig2.png)

^
- map takes a function and an array, and applies that function to every element in that array, returning a new array
- as I have mentioned, functional programming emphasizes immutable data, so we will return new data from this function

^
- map takes a function from any type a to the same or different type b, then takes an array of a's and returns an array of b's.

---
# single value transformation

```javascript
const multiplyBy3 = v => v * 3
let x = 2;
let y; // y = undefined

y = multiplyBy3(x)
// 6

// y has been mapped from undefined to 3
```

^
- mapping can be described as a transformation from one value to another value

---
# multiple value transformation
```javascript
// imperative
let makes = []
for (var i = 0; i < cars.length; i++) {
  makes.push(cars[i].make)
}

// declarative
const makes = map(
  (car) => car.make,
  cars
)
```

^
- transforms all the values of a list as it projects them to a new list:
- use imported map from lodash-fp/ramda to avoid 3rd this argument that allows passing this around
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Map

---
# filter
### `filter :: (a -> Bool) -> [a] -> [a]`
![inline 100%](fig3.png)

^
- filter takes a predicate function from any type a to a Boolean, then takes an array of a's and returns a new array of a's
- as with map, we return a new array
- filter out unwanted stuff
- takes a function to decide if each value in the original array should be in the new array

---
# diy filter
```javascript
const pool = [
  { name: 'Ryen', rgScore: 141.99 },
  { name: 'Sergey', rgScore: 161.2, },
  { name: 'Steven', rgScore: 150.5, },
]

function getResearchers(researchers, minRgScore) {
  let results = []

  for (var i = 0; i < researchers.length; i++) {
    if (researchers[i].rgScore >= minRgScore) {
      results.push(researcher)
    }
  }

  return results
}

const powerUsers = getResearchers(pool, 150)
// [{'name':'Sergey','rgScore':161.2},{'name':'Steven','rgScore':150.5}]
```

---
# filter
```javascript
const pool = [
  { name: 'Ryen', rgScore: 141.99 },
  { name: 'Sergey', rgScore: 161.2, },
  { name: 'Steven', rgScore: 150.5, },
]

const isPowerUser = (researcher) => {
  return researcher.rgScore >= 150
}

const powerUsers = filter(isPowerUser, pool)
// [{'name':'Sergey','rgScore':161.2},{'name':'Steven','rgScore':150.5}]

const weakUsers = filter(complement(isPowerUser), pool)
// [{'name': 'Ryen','rgScore':141.99}]
```

^
- here is the functional version
- power users becomes much more succinct
- this adds some flexibility as well
- we can use complement/negate from lodash-fp to give us the inverse filter

---
# reduce
### `reduce :: (b -> a -> b) -> b -> [a] -> b`
![inline 100%](fig4.png)

^
- the type signature is a little tricky, but it takes a function that expects a b and an a, and produces a b (accumulated value, current value -> accumulated value)
- it also takes an initialValue(b), an initial Array ([a]), and returns a type b
- reduce is a function used for reducing a collection of values down to a single value.
- slightly different to map and filter, reduce can return a value of any type

---
# diy reduce
```javascript
const pool = [
  { name: 'Ryen', rgScore: 141.99, paperCount: 2 },
  { name: 'Sergey', rgScore: 161.2, paperCount: 4 },
  { name: 'Steven', rgScore: 150.5, paperCount: 7 },
]

function tallyPapers(arr) {
  let totalPapers = 0
  for (var i = 0; i < arr.length; i++) {
    totalPapers += arr[i].paperCount
  }

  return totalPapers
}

tallyPapers(pool)
// 13
```

---
# reduce
```javascript
const pool = [
  { name: 'Ryen', paperCount: 2 },
  { name: 'Sergey', paperCount: 4 },
  { name: 'Steven', paperCount: 7 },
]

const accumulatePapers = (acc, current) => acc + current.paperCount;

reduce(
  accumulatePapers,
  0,
  pool
)
// 13
```

^
- as we can see, reduce takes an accumulator function
- which gets the accumulated value and the current item as arguments
- we could equally pass in a concatenateNames function, or some other function that changed how that pool was reduced

---
# currying
### `curry :: a -> b -> c`

^
- also known as partial application
- currying is the process of transforming a function that takes multiple arguments into a function that takes just a single argument and returns another function if any arguments are still needed.
- it is implemented as a helper function provided by lodash
- but to illustrate what it does internally, here is a simple example
- you may have heard of partial application, see here for how they are different http://www.datchley.name/currying-vs-partial-application/

---
# partial application
```javascript
function ajax(url, data, callback) {
    // ..
}

function getOrder(data, cb) {
    ajax('http://some.api/order', data, cb)
}
```

^
- getOrder is a partial application of ajax
- this terminology comes from the idea that arguments are applied to parameters at the call signature
- as you can see, we are only applying some of the arguments up front (url parameter)

---
# a simple example
```javascript
const match = function(what) {
  return function(str) {
    str.match(what)
  }
}

const hasSpaces = match(/\s+/g)
// function(str) { return str.match(/\s+/g) }

filter(hasSpaces, ['stephen_hawking', 'stephen gould'])
// ['stephen gould']
```

^
- we have a function called match, which takes a regular expression (What)
- and returns a function that takes a string
- so we can create a new utility called hasSpaces, by passing a single argument to match
- and match will return us a function that is just waiting for a string to test

---
# with helper function
```javascript
import { curry } from 'lodash/fp'

const match = curry(function(what, str) {
  return str.match(what)
})
```

^
- let's see what our match function looks like using the curry utility
- the curry utility will let us write code as if all arguments are received at the same time
- this is particularly useful when only know some of the arguments your function will need

---
# further currying

```javascript
import { curry } from 'lodash/fp'

const replace = curry((what, replacement, str) => {
  return str.replace(what, replacement)
})

const noVowels = replace(/[aeiouy]/ig)
// function(replacement, x) { return x.replace(/[aeiouy]/ig, replacement) }

const censored = noVowels('*')
// function(x) { return x.replace(/[aeiouy]/ig, '*') }

censored('Chocolate Rain')
// 'Ch*c*l*t* R**n'
```

^
- another example, here replace takes 3 arguments, a regex, a replacement text and a string
- we can create noVowels by just passing the first argument - What, and get back a function that is waiting for a replacement and a string
- we can then pass noVowels the replacement and get back a function that just wants a string
- and finally we can call censored with our string
- curry does not need a helper function when you know the arity of the function

---
# composition
### `compose :: (f, g) -> (x) -> f(g(x))`

^
- functional programming emphasises building programs from small units (functions)
- so you may find yourself in the situation of wanting to combine multiple funtions together
- similar to the pipe utility, except it applies the functions in right-to-left order instead of left to right
- (f, g)(x) === f(g(x))
- compose is easier to translate into the nested function style: f(g(x))
- In mathematics f ∘ g (f composed with g) is the function that given x, returns f(g(x)).

---
# composition
```javascript
const toUpperCase = x => x.toUpperCase()
const exlcaim = x => `${x}!`

const shout = compose(exclaim, toUpperCase)
shout('send in the scientists')
// 'SEND IN THE SCIENTISTS!''
```

^
- the composition of two functions returns a new function
- functions run right to left, the g will run before the f

---
# composition

```javascript
const head = x => x[0]
const reverse = reduce((acc, x) => [x].concat(acc), [])
const last = compose(head, reverse)

last(['tripod', 'crucible', 'microscope'])
// 'microscope'
```

^
- another example where order matters
- head returns the first item in a list
- reverse will reverse the array it is given
- last will combine this functionality
- the order matters because if we pass reverse as the first argument to compose, then we will get the reverse of tripod as an array of letters
- this also illustrates why functions need return values, so that we can compose their functionality together

---
# what we gain from fp style

- elegance and simplicity
- better decomposition of problems
- straightforward unit testing
- easier debugging

---
## thankyou!
# 👽✌️
#### ..questions..

---
# resources
- https://www.researchgate.net/project/Functional-Javascript-l
- http://codepen.io/ryenjbeatty/pen/gLaMvV?editors=0012
