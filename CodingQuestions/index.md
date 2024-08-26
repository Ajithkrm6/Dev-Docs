# Coading Questions:

1. Palindrome:

```
const checkPlaindrome = (str) => {
    let temp = str.toString();
    if(str === str.split('').reverse('').join('')){
        console.log(`${str} is a palinrome`);
    }
    else{
        console.log(`${str} is not a palindrome`)
    }
}

```

2. Write a JavaScript function that takes an array of numbers and returns a new array with only the even numbers.

```
const checkEvenNumInArr = (nums) => {
    return nums.filter((num)=>num%2 === 0)
}

let arr = [1,23,5,3,4,7,6,7,8];
const result = checkEvenNumInArr(arr);
console.log(result);
```

3. write factorial of a given number

```
const factorial = (num) => {
    if(num === 0 || num ===1){
        return 1
    }
    else{
        return num8factorial(num-1)
    }
}

let num = 5;
const result = factorial(num);
console.log(result)
```

4. Prime Number

```
const ifPrime = (num) => {
    let temp= 0;
    if(num <= 1){
        return false;
    }
    for(let i=2; i<= num/2;i++){
        if(num%i === 0){
            temp++
        }
    }
    if (temp > 0){
        return false;
    }
    else{
        return true
    }
}

const num = 2;
const result = ifPrime(num);
if(result){
    console.log(`${num} is prime number`)
}
else{
    console.log(`${num} is not a prime number`)
}
```

5. Write a JavaScript program to find the largest element in a nested array.

```
const largestElem = () => {

}

```
