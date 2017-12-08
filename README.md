# JS Gotchas

## Closures and SetTimeout

```javascript
// Q: What will the following code output?
for (var i = 0; i < arr.length; i++) {
	setTimeout(function() {
		console.log(`Index ${i} element: ${arr[i]}`)
	}, 3000);
}

// A: Index 4 element: undefined (4 times)
```

### Discussion
The problem deals with closures, setTimeout and scope. The setTimeout method takes in a function which creates a __closure__. The closure has access to variable `i` in the outer scope. After the timeout of 3 seconds has elapsed the callback is executed and prints the value of `i` by which time the has ended and the value of `i` is now 4 and `arr[4]` does not exist and is therefore `undefined`.

### Solutions
There are two solution to this problem. The first involves creating a parametrised inner function and passing the current value of `i` within each loop iteration.
```javascript
for (var i = 0; i < arr.length; i++) {
  // pass in variable i so the innermost function is
  // 
  setTimeout(function(i_local) {
    return function() {
      console.log(`Index ${i_local} element: ${arr[i_local]}`)
        }
  }(i), 3000);
}
```
The second possible solution uses ES6 and is more concise
```javascript
for (let i = 0; i < arr.length; i++) {
  setTimeout(function() {
    console.log(`Index ${i} element: ${arr[i]}`)
  }, 3000);
}
```
