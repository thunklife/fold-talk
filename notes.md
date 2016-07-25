Know When To Fold 'Em
=====================

An exploration into `Array.prototype.reduce`

What Is `reduce`
---------------

> The reduce() method applies a function against an accumulator and each value of
the array (from left-to-right) to reduce it to a single value

> ```arr.reduce(callback[, initial])```

So `reduce` is built into all arrays and is a way to transform the array, item
by item, into a new value.

In order to use `reduce` you need to supply a function to accumulate the values
and an initial value. The initial value is optional. If you omit it, the first
item in the array is used.

Here is a simple example to sum all the numbers in a list.
```
const xs = [1, 2, 3];
const sum = xs.reduce((prev, current) => prev + current, 0);

// or

const sum2 = xs.reduce((prev, current) => prev + current)

/*
---------------------------------------------------
 */
xs.reduce((prev, curr) => prev + curr, 0)
            0     1            1
            1     2            3
            3     3            6

xs.reduce((prev, curr) => prev + curr)
            1     2            3
            3     3            6
```

Here is another example to check if all values in an array or truthy
```
const allTrue = bools.reduce((prev, curr) => return prev && curr);
```

Side note: The name `prev`, while cribbed from MDN, is a pretty bad name. The
argument is really only the previous value in the second example, and even then
it's only the previous value once. After that, it ceases to represent the previous
value and really represents the accumulating value. I much prefer the name `acc`,
short for 'accumulator'.

So what is `reduce`? Well it's an abstraction. It's a generalization of recursing
over an array and accumulating the values somehow. How you accumulate them is up
to you.

Don't let MDN description fool you, the 'single value' that `reduce` returns
can be anything you like. Numbers, booleans or strings are obvious single values,
but arrays and objects are also single values.

Let's say you have an array of names, and you want to see how often each name
appears in the list. You could:

```
const names = ['Al', 'Jean', 'Aaron', 'Al', 'John', 'John'];

const nameCounts = {};
for(const i = 0, i < names.length, i++) {
  const name = names[i];
  nameCounts[name] = nameCounts[name] + 1 || 1;
}
```

And that totally works, but we've got all this extra stuff to keep track of. There's a
counter, and bounds check, you've got access each item in the array by index. Of
course, loops are very common place. Most of us don't have to think twice about
keeping that stuff straight.

You could also:
```
const nameCounts = {};

names.forEach((name) => {
  nameCounts[name] = nameCounts[name] + 1 || 1;
});
```

This is better than the loop. All of those additional variables are gone and that
is a big plus. Let's compare this to the `reduce` version before going any further.

```
const nameCounts = names.reduce((acc, name) => {
  acc[name] = acc[name] + 1 || 1;
  return acc;
}, {});
```

Pretty similar actually, but I would argue there are a couple of things about
`forEach` here that lead me to argue that it is the wrong choice.

First, the most common use of `forEach` is mutation, which is not what we want.
If we wanted to append " rocks!" to the end of each name, that would be
different. Looping through each name and changing it is a more appropriate
use of `forEach` because we are mutating the values being iterated over.

Second, the it isn't pure. It relies on a variable not passed to it to do its
work. It may not be a big deal in this example, but anything could happen to
the target object before the loop runs, so we have no guarantee that the result
will always be the same. The fact that `forEach` has no return value is also an
indicator of it's impurity.

Lastly you could use a `for...of` loop, but it suffers from the same drawbacks
as `forEach` so we won't go into it.

What can you do with `reduce`
----------------------------

The idea that `reduce` is just an abstract way to transform an array is extremely
powerful. And once you begin to see `reduce` in that light, you realize that lots
of functions can be defined using `reduce`.

For example, `map` is just reducing an array into another array.

```
const myMap = (fn, arr) => arr.reduce((prev, curr) => {
    prev.push(fn(curr));
    return prev;
  }, []);
```

Similarly, `filter` can be defined using `reduce`.

```
const myFilter = (pred, arra) => arr.reduce((acc, curr) => {
  if (pred(curr)) { acc.push(curr); }
  return acc;
}, [])
```

Of course, the built-in array methods have the benefit of being chainable. If we
wanted to take an array of all the numbers from 0 to 100, take only those divisible
by 3 and then square them, we could:

```
onToOneHundred
  .filter((n) => n % 3 === 0)
  .map((n) => n * n);
```

But that means we pass through the array twice. Each array function enumerates
the values, and returns a new array. If more functions get chained, that's more
passes through the array. Not really a concern here, but what if the array was much
larger?

If you know that `filter` and `map` are just special versions of `reduce` you can
use `reduce` to combine the two.

```
onToOneHundred.reduce((acc, curr) => {
  if (curr % 3 === 0) {
    acc.push(curr * curr);
  }

  return acc;
}, []);
```
