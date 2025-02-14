Conditional Programming in R
================
Agoston Reguly

Data used:
[wms-management](https://gabors-data-analysis.com/datasets/#wms-management-survey)

## What are conditional statements

Conditionals are expressions that perform different computations or
actions depending on whether a predefined logical expression (aka
boolean) condition is `TRUE` or `FALSE`.

### The `if` statement

The `if` statement takes a condition and if the condition evaluates to
`TRUE` the associated R code is executed.

Pseudo-code:

``` r
if (condition) {
  command
}
```

The condition to check appears inside parentheses, while the R code that
has to be executed if the condition is TRUE, follows in curly brackets.

Lets take a simple example:

``` r
x <- 5
if ( x == 5){
  print( 'x is equal to 5')
}
```

    ## [1] "x is equal to 5"

### Using `else` to complement conditional

The `else` statement complements the `if` condition: if the condition
evaluates to `FALSE` with the `else` statement you can specify what
alternative command should R execute.

Pseudo-code:

``` r
if (condition) {
  command
} else{
  do_something_else
}
```

To continue our example Lets take a simple example:

``` r
x2 <- 4
if ( x2 == 5){
  print( 'x2 is equal to 5')
} else{
  print( 'x2 is not equal to 5')
}
```

    ## [1] "x2 is not equal to 5"

### Multiple `if` and `else` statements:

It is possible to make more elaborate conditional execution with
multiple `if` and `else` statements:

Pseudo-code:

``` r
if ( condition_1 ){
  command_1
} else if ( condition_2 ){
  command_2
} else if ( condition_3 ){
  command_3
} else if( ... ){
  ...
} else{
  if_none_of_conditions_are_true_do_this
}
```

and to show a real example:

``` r
x <- -5
if ( x > 0 ){
  print( 'positive number')
} else if( x == 0 ){
  print( 'zero value')
} else{
  print('negative number')
}
```

    ## [1] "negative number"

**Tip:** it is always a good practice to use an `else` part in the
conditionals. In many cases we do mistakes and R will not evaluate any
of our commands, even if we believe we mapped all the possible
scenarios. In this case you can put an `else` statement with an error or
print out some message to notify that you need to refine your
statements.

## Multiple conditions

From the logic of `if` and `else` statements it follows that multiple
conditions are possible when creating a conditional execution. The only
challenge is that the end result should be one logical value.

### Multiple logical statements

I call multiple logical statements the case when there are multiple
single-valued variables evaluated in the condition. In this case always
use double logical operators such as `&&` or `||`, which ensures that
the end result is one logical value. However, be careful, if you supply
a vector instead of a single-valued variable, you may end up with
unintended results.

``` r
y <- 10
if ( x > 0 && y > 0 ){
  print('x and y are positive numbers')
} else{
  print('one of y or x is non-positive')
}
```

    ## [1] "one of y or x is non-positive"

### Conditional with one vector

Another possibility is when one works with vectors (or matrices), which
does not result in one logical, but rather in a vector (or matrix) of
logical. Let us consider the following simple example. Before we use
them in an `if-else` statement, let us do some logical operations:

``` r
v <- c( 0 , 1 , 10 )
```

First, let check if elements of v larger than 0

``` r
v > 0
```

    ## [1] FALSE  TRUE  TRUE

which results in a vector. To suppress this into one logical outcome, we
need to decide what we want:

-   if any element is larger than zero -\> use `any()`
-   if all elements are larger than zero -\> use `all()`

Let us say we want any elements to be larger than zero, thus:

``` r
if ( any( v > 0 ) ){
  print('We have at least one element in v, which is larger than zero!')
} else {
  print('All elements in v, are smaller than zero!')
}
```

    ## [1] "We have at least one element in v, which is larger than zero!"

### Conditional with two or more vector

If we have more than one vector, conditionals can be tricky and the
programmer must be careful at which level are the logical operations.
Let us create another vector `q`.

``` r
q <- c( 2 , 0 , 8 )
```

Let us check again if elements are larger than zero or not. We have
again two cases under the assumption `v` and `q` have the same elements.

-   one of the elements in `v` and `q` – **evaluated pairwise** - is
    larger than zero:

``` r
v | q > 0
```

    ## [1] TRUE TRUE TRUE

-   both elements in `v` and `q` - **evaluated pairwise** is larger than
    zero:

``` r
v & q > 0
```

    ## [1] FALSE FALSE  TRUE

At this point we can check the differences between single-operators and
double-operators

``` r
v | q > 0
```

    ## [1] TRUE TRUE TRUE

``` r
v || q > 0
```

    ## [1] TRUE

Using double-operators will imply `any()` for `||` and `all()` for `&&`:

``` r
( v || q > 0 ) == any( v | q > 0 )
```

    ## [1] TRUE

``` r
( v && q > 0 ) == all( v & q > 0 )
```

    ## [1] TRUE

therefore one needs to be careful, when using these operators with
vectors, as the results can be different if mixing these up, e.g.

``` r
v && q > 0
```

    ## [1] FALSE

``` r
any( v & q > 0 )
```

    ## [1] TRUE

Finally, with two or more vectors there is an other way to evaluate a
conditional: first decide for each element of each vector if the
statement is true and then compare that result with each other. It may
result in a outcome. E.g. check if all elements are greater than 0 for
each vector and then compare if both of them are true:

``` r
all( v > 0 ) && all( q > 0)
```

    ## [1] FALSE

or check if all elements in `v` but any elements in `q` and then use the
or comparison:

``` r
all( v > 0 ) || any( q > 0 )
```

    ## [1] TRUE

## Conditionals for creating a new variable

In the data analysis business conditionals are not used that often or
that explicitly. Most commonly it used, when a new variable is created
and its values are conditional on other variable(s).

To show a simple example, let us use the
[wms-management](https://gabors-data-analysis.com/datasets/#wms-management-survey)
data,

``` r
library(tidyverse)
# Import wms-management data
wms <- read_csv('https://osf.io/uzpce/download')
```

Look at `emp_firm`, which is the number of employee in the firm. Let us
create a new variable based on number of employees. We consider three
types of firms:

-   large: no. employees is more or equal than 1000
-   medium: no. employees is more or equal than 200, but less than 1000
-   small: no. employees is less than 200

We will cover two cases:

-   using simple base-R methods to demonstrate conditionals
-   introduce `ifelse()` function, which is more compatible with
    `tidyverse` approach.

### Base R for creating new variables with conditionals

Using previous conditional statements it is easy to use logical
operations to index certain values and then redefine them.

``` r
# Create a new variable with NA values by default -> it is going to be a character vector
wms$firm_size <- NA_character_
wms$firm_size[ wms$emp_firm >= 1000 ] = 'large'
wms$firm_size[ wms$emp_firm < 1000 & wms$emp_firm >= 200 ] = 'medium'
wms$firm_size[ wms$emp_firm < 200 ] = 'small'
```

### `ifelse()` function

There is a built-in `ifelse(condition,if_true,if_false)` function, which
is a compact form of `if-else` statements. This is a bit harder to read,
but useful with `tidyverse`. We need to use multiple `ifelse()` function
as if the first is false, we need to still consider the other cases.
Again as usual, try to cover all possible cases!

``` r
wms <- wms %>% mutate( firm_size2 = ifelse( emp_firm >= 1000 , 'large',
                                    ifelse( wms$emp_firm < 1000 & wms$emp_firm >= 200 , 'medium',
                                    ifelse( wms$emp_firm < 200, 'small', NA_character_ ) ) ) ) 
```

**Task:** Check if `firm_size` and `firm_size2` have exactly the same
values!

## Extra material

In the following, we cover some non-essential, but good-to-know
materials related to conditionals.

### Conditional installation of packages

Let us start with a somewhat un-related issue. It is really useful to
install a package conditional on if it has been already installed or
not. It helps a lot for working with others.

A simple example for `tidyverse`:

``` r
if ( !require(tidyverse) ){
  install.packages('tidyverse')
  library(tidyverse)
}
```

This can help you to avoide to install a package multiple times!

Pseudo-code:

``` r
if(!require(somepackage)){
    install.packages('somepackage')
    library(somepackage)
}
```

### Spacing and formatting

It is possible to write short `if-else` statements in one line:

``` r
if ( x > 5 ){ print(' x > 5') } else { print('x <= 5' ) }
```

    ## [1] "x <= 5"

However, it is not recommended as it makes reading the code much harder.

### The `xor` operator

There is another logical operator `xor`, which takes two logical
value/vectors as inputs. `xor` will result in `TRUE` if, during the
element-by-element comparison, the logical expressions are different in
the vectors and `FALSE` if they are the same. E.g.

``` r
xor(c(T,F,F,T),c(T,T,F,F))
```

    ## [1] FALSE  TRUE FALSE  TRUE

### `switch` statement

The `switch` statement is a handy tool if you have multiple possible
scenarios and you do not want to use logicals, but strings. Let us check
the following toy-example:

``` r
type <- 'apple'
switch(type,
       apple = 'I love apple!',
       banana = 'I love banana!',
       orange = 'I love orange!',
       error('type must be either \'apple\',\'banana\', or \'orange\'')
       )
```

    ## [1] "I love apple!"

The last part of this expression `error()` is called error handling,
which we will discuss later.
