---
author: Jeffrey W. Hollister & Lindsay Carr
date: 9999-01-02
slug: Reproduce
title: I. Repeat and Reproduce
image: img/main/intro-icons-300px/repeat.png
menu: 
  main:
    parent: Introduction to R Course
    weight: 1
---
You now have a basic understanding of how to conduct a typical data analysis workflow in R. All that is left is to be able to write it up in such a way that others can not only understand what we did, but repeat it exactly on their own machines. To do this effectively we need to understand how to create reusable R code and create reproducible reports. This will be a very high level introduction to both concepts, but should hopefully give you a jumping off place for more learning.

Remember that we are using the NWIS dataset for all of these lessons. If you successfully completed the [Clean](/intro-curriculum/clean) lesson, then you should have the cleaned up version of the data frame. If you did not complete the Clean lesson (or are starting in a new R session), just load in the cleaned csv by downloading it from [here](/intro-curriculum/data), saving it in a folder called "data", and using `read.csv` (see below).

``` r
intro_df <- read.csv("data/course_NWISdata_cleaned.csv", stringsAsFactors = FALSE, 
                     colClasses = c("character", rep(NA, 6)))
```

Quick Links to Exercises and R code
-----------------------------------

-   [Exercise 1](#exercise-1): Using for loops and conditionals
-   [Exercise 2](#exercise-2): Writing functions
-   [Exercise 3](#exercise-3): Creating an RMarkdown document

Lesson Goals
------------

-   Be able to use basic programming control structures (loops, conditional statements)
-   Understand how to create your own functions
-   Gain familiarity with Markdown and `knitr`
-   Create a simple, reproducible document and presentation

Using conditional (if-else) statements
--------------------------------------

If you have done any programing in any language, then `if-else` statements are not new to you. All they do is tell your code how to make decisions. For instance, you might want a subroutine only applied to certain sites or when certain data is (or is not) present. That's easy with if-else control structures. Here's the basic syntax of an if-else statement:

    if(some condition){
      do something
    } else {
      do something different
    }

Let's first go through basic if-else structures.

``` r
x <- 2

# logical statement inside of () needs to return ONE logical value - TRUE or FALSE. TRUE means it will enter the following {}, FALSE means it won't.
if(x < 0){
  print("negative")
} 

# you can also specify something to do when the logical statement is FALSE by adding `else`
if(x < 0){
  print("negative")
} else {
  print("positive")
}
```

    ## [1] "positive"

The key to if-else is using a logical statement that only returns a single TRUE or FALSE, not a vector of trues or falses. The if statement needs to know if it should enter the `{}` or not, and therefore needs a single answer - yes or no. Here are some useful functions that allow you to use vectors in if-else, but still return a single yes or no:

``` r
y <- 1:7

# use "any" if you want to see if at least one of the values meets a condition
any(y > 5)
```

    ## [1] TRUE

``` r
# use "!any" if you don't want any of the values to meet some condition (e.g. vector can't have negatives)
!any(y < 0) 
```

    ## [1] TRUE

``` r
# use "all" when every value in a vector must meet a condition
all(y > 5)
```

    ## [1] FALSE

``` r
# using these in the if-else statement
if(any(y < 0)){
  print("some values are negative")
} 
```

And you can you use multiple `if` statements by stringing them together with `else`

``` r
num <- 198

if(num > 0) {           # first condition
  print("positive")
} else if (num < 0) {   # second condition
  print("negative")
} else {                # everything else
  print("zero")
}
```

    ## [1] "positive"

Now that you have a basic understanding of if-else structures, let's use them with `dplyr` to manipulate a data frame. We want to add a new column if some condition is met, or filter a column if the condition is not met. We can use the `if-else` structure along with `mutate` and `select` from `dplyr` to accomplish this (see the [Clean](/intro-curriculum/clean) lesson to learn about dplyr).

``` r
# first, make sure dplyr is loaded
library(dplyr)

# if the column "DO_mgmL" (dissolved oxygen in mg/mL) does not exist, we want to add it
if(!'DO_mgmL' %in% names(intro_df)){
  head(mutate(intro_df, DO_mgmL = DO_Inst/1000))
} 
```

    ##    site_no            dateTime Flow_Inst Flow_Inst_cd Wtemp_Inst pH_Inst
    ## 1 02336360 2011-05-03 21:45:00      14.0            X       21.4     7.2
    ## 2 02336300 2011-05-01 08:00:00      32.0            X       19.1     7.2
    ## 3 02337170 2011-05-29 22:45:00    1470.0            A       24.0     6.9
    ## 4 02203655 2011-05-25 01:30:00       7.5          A e       23.1     7.0
    ## 5 02336120 2011-05-02 07:30:00      16.0            A       19.7     7.1
    ## 6 02336120 2011-05-12 16:15:00      14.0          A e       22.3     7.2
    ##   DO_Inst DO_mgmL
    ## 1     8.1  0.0081
    ## 2     7.1  0.0071
    ## 3     7.6  0.0076
    ## 4     6.2  0.0062
    ## 5     7.6  0.0076
    ## 6     8.1  0.0081

``` r
# if there are more than 1000 observations, we want to filter out high temperature observations
if(nrow(intro_df) > 1000){
  head(filter(intro_df, Wtemp_Inst >= 15))
}
```

    ##    site_no            dateTime Flow_Inst Flow_Inst_cd Wtemp_Inst pH_Inst
    ## 1 02336360 2011-05-03 21:45:00      14.0            X       21.4     7.2
    ## 2 02336300 2011-05-01 08:00:00      32.0            X       19.1     7.2
    ## 3 02337170 2011-05-29 22:45:00    1470.0            A       24.0     6.9
    ## 4 02203655 2011-05-25 01:30:00       7.5          A e       23.1     7.0
    ## 5 02336120 2011-05-02 07:30:00      16.0            A       19.7     7.1
    ## 6 02336120 2011-05-12 16:15:00      14.0          A e       22.3     7.2
    ##   DO_Inst
    ## 1     8.1
    ## 2     7.1
    ## 3     7.6
    ## 4     6.2
    ## 5     7.6
    ## 6     8.1

### Optional: the `ifelse` function

Let's learn a new way to apply if-else logic - the function `ifelse`. This function is similar to the previous if-else structure syntax, but the function can only return one value (rather than doing a series of commands within `{}`). See `?ifelse` for more information. Basic syntax for `ifelse()`:

`ifelse(condition, yesValue, noValue)`

Add a new column to `intro_df` that removes the flow value if it is erroneous (code is "X"), otherewise, retain the flow value.

``` r
#use mutate along with ifelse to add a new column
intro_df_revised <- mutate(intro_df, Flow_revised = ifelse(Flow_Inst_cd == "X", NA, Flow_Inst))
```

Looping
-------

A `for` loop allows you to repeat code. You specify a variable and a range of values and the `for` loop runs the code for each value in your range. There are other ways to repeat code (e.g. apply suite of functions), but we are only going to discuss for loops (some in the R world think loops are be bad since R is optimized for working on vectors, but the concept is useful). The basic structure looks like:

    for(a_name in a_range) {
      code you want to run
      may or may not use a_name
    }

``` r
# sequentially increase the value of some number
vec <- 1:10
j <- 0 # variables used in loops must exist already - so we initialize them
for(i in vec) {
  j <- i + j
  print(j)
}
```

    ## [1] 1
    ## [1] 3
    ## [1] 6
    ## [1] 10
    ## [1] 15
    ## [1] 21
    ## [1] 28
    ## [1] 36
    ## [1] 45
    ## [1] 55

Again a bit of a silly example since all it is doing is looping through a list of values and summing it. In reality you would just use `sum()` or `cumsum()`. This also highlights the fact that loops in R can be slow compared to vector operations and/or primitive operations (see Hadley Wickham's section on [Primitive functions](http://adv-r.had.co.nz/Functions.html#function-components)).

Let's dig a bit more into this issue with another example. This time, let's look at adding two vectors together. We haven't touched on this yet, but R is really good at dealing with this kind of operation. It is what people mean when they talk about "vectorized" operations. For instance:

``` r
# A simple vectorize operation
x <- 1:100
y <- 100:1
z <- x + y
z
```

    ##   [1] 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101
    ##  [18] 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101
    ##  [35] 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101
    ##  [52] 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101
    ##  [69] 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101
    ##  [86] 101 101 101 101 101 101 101 101 101 101 101 101 101 101 101

Pretty cool. This kind of thing doesn't come easily with many languages. You would need to program it yourself using a loop. For the sake of argument (and practice), let's try the loop version with R. You'll notice that we have set `out <- NULL` at the beginning. We are adding values to `out` using the loop, but the object must exist first. Thus, we make it NULL or you can use `out <- c()` if you know it will be a vector.

``` r
#We will assume vectors of the same length...
out <- NULL 
for(i in 1:length(x)) {
  out[i] <- x[i] + y[i]
}
```

So, these do the same thing, big deal. It is big though when you look at the timing of the two. Let's create two large vectors and see what happens.

``` r
large_vec1 <- as.numeric(1:100000)
large_vec2 <- as.numeric(100000:1)

# vectorized happens almost instaneously
vec_time <- large_vec1 + large_vec2

# we have to wait for this loop (~17 seconds)
out <- NULL
for(i in 1:length(large_vec1)) {
  out[i] <- large_vec1[i] + large_vec2[i]
}
```

Wow, quite a difference in time! It is examples like this that lead to all the talk around why R is slow at looping. In general I agree that if there is an obvious vectorized/base solution (in this case simply adding the two vectors) use that. That being said, it isn't always obvious what the vectorized solution would be. In that case there are some easy things to do to speed this up. With loops that write to an object and that object is getting re-sized, we may also know the final size of that object so we can do one simple thing to dramatically improve perfomance: pre-allocate your memory, like this:

``` r
#We will assume vectors of the same length...
out <- vector("numeric",length(large_vec1))
for(i in 1:length(large_vec1)) {
  out[i] <- large_vec2[i] + large_vec2[i]
}
```

Now that's better. In short, if an obvious vector or primitive solution exists, use that. If those aren't clear and you need to use a loop, don't be afraid to use one. There are plenty of examples where a vectorized solution exists for a loop, but it may be difficult to code and understand. Personally, I think it is possible to go too far down the vectorized path. Do it when it makes sense, otherwise take advantage of the `for` loop! You can always try and speed things up after you have got your code working the first time.

Exercise 1
----------

For this exercise we are going to practice using control structures. We are going to make a plot of DO vs water temperature for each site in our dataset.

1.  Create a for loop that loops through each site and creates a plot. Don't save the plot, just have it render in the plot window. Hint: use `unique(intro_df$site_no)` to get a vector of sites.
2.  Let's imagine that we had one site that we knew had bad data, and we don't want to render a plot for it. Add a conditional statement to your loop that skips the plotting code for this particular site.

Functions in R
--------------

At this point we should be pretty well versed at using functions. They have a name and some arguments, and they do something. Some return a value, some don't. In short they form the basic structure of R. One of the cool things about R (and programming in general), is that we are not stuck with the functions provided to us. We can and should develop our own, as we often want to do things repeatedly, and in slightly different contexts. Creating a function to deal with this fact helps us a great deal because we do not have to repeat ourselves, we can just use what we have already written. Creating a function is really easy. We use the `function()` function. It has the basic structure of:

    function_name <- function(arguments) {
      code goes here
      use arguments as needed
    }

So a real example with no arguments might look like:

``` r
say_hi <- function() {
  print("Hello, World!")
}
say_hi()
```

    ## [1] "Hello, World!"

Well that's nice... Not really useful, but shows the main components, `function()`, and the `{}` which are really the only new things.

It would be a bit better if it were more flexible. We can do that because we can specify our own arguments to use within the body of the function. For example,

``` r
print_twice <- function(my_text) {
  print(my_text)
  print(my_text)
}

print_twice("Hello, World!")
```

    ## [1] "Hello, World!"
    ## [1] "Hello, World!"

``` r
print_twice("Hola, mundo")
```

    ## [1] "Hola, mundo"
    ## [1] "Hola, mundo"

``` r
print_twice("Howdy, Texas")
```

    ## [1] "Howdy, Texas"
    ## [1] "Howdy, Texas"

Functions are most useful when we want to repeat a general procedure with different specifics each time. Since we have been working most recently with creating plots, I could imagine us wanting to create a plot with a similar layout, but with different source data and then save that plot to a file, all with a single function call. That might look like:

``` r
myplot <- function(x, y, grp, file) {
  my_dat <- data.frame(X=x, Y=y, Grp=grp)
  my_p <- ggplot(my_dat, aes(x=X, y=Y)) +
    geom_point(aes(color=Grp, shape=Grp), size=5) +
    geom_smooth(method="lm", aes(color=Grp)) +
    labs(x=substitute(x), y=substitute(y))
  ggsave(my_p, file=file)
  return(my_p)
}

#Call the function using intro_df
library(ggplot2)
myplot(intro_df$Flow_Inst, intro_df$Wtemp_Inst, 
       intro_df$Flow_Inst_cd, "q_Wtemp.jpg")
```

<img src='../static/Reproduce/plot_function_examp-1.png'/ title='ggplot2 scatter plot of pH versus flow'/>

``` r
myplot(intro_df$Flow_Inst, intro_df$DO_Inst, 
       intro_df$Flow_Inst_cd, "q_do.jpg")
```

<img src='../static/Reproduce/plot_function_examp-2.png'/ title='ggplot2 scatter plot of dissolved oxygen versus flow'/>

Cool, a function, that does something useful.

### return

The last control structure we are going to talk about is `return()`. All `return()` does is provide a result from a function and terminates the function. You may be asking yourself, didn't we terminate and get a value from the functions we just created? We did, and `return()` is not mandatory for R functions as they will return the last calculation. Some people argue that using `return()` is good practice because it allows us to be more explicit. Others argue that concise is beautiful and that it's not hard to see what is being implicitly returned. Think about it for yourself as you learn more R. To see how `return()` can be used, let's take a look at the `odd_even()` and `sum_vec()` functions from before and make simple changes to take advantage of `return()`.

First, `odd_even()`.

``` r
odd_even <- function(num) {
  if(num %% 2 == 0){
    return("EVEN")
  } 
  return("ODD")
}
```

Now, `sum_vec()`.

``` r
sum_vec <- function(vec) {
  j <- 0
  for(i in vec) {
    j <- i + j
  }
  return(j)
}
```

Exercise 2
----------

For this exercise we are going to practice with functions.

1.  Our first task is to create a simple function. Create ONE function that allows you to calculate the mean or standard deviation (hint: sd()) of an input vector.
2.  Our second task is going to be to make a function that returns a simple `ggplot2` or base scatter plot. The function should take an x and y as input, and have an optional argument for a file name to save it as an image. Depending on how complex your plot was you may need to add in some additional arguments. To help things along, I have provided some starter code:

``` r
create_my_plot <- function(x, y, filename=NULL){
  #plot code
  #Note: ggplot requires a data frame as input.  How would you deal with that?
  
  #saving image code here
  #look into the is.null() function
  if(put condition here) {
    ggsave() # or png(), jpeg(), etc
    #Note when saving using png(), jpeg(), etc plot needs to be printed after
  }
  
  #Need to return something ...
  return()
}
```

Markdown
--------

Markdown isn't R, but it has become an important tool in the R ecosystem as it can be used to create package vignettes, can be used on [GitHub](http://github.com), and forms the basis for several reproducible research tools in RStudio. Markdown is a tool that allows you to write simply formatted text that is converted to HTML/XHTML. The primary goal of markdown is readibility of the raw file. Over the last couple of years, Markdown has emerged as a key way to write up reproducible documents, create websites, and make presentations. For the basics of markdown and general information look at [Daring Fireball](http://daringfireball.net/projects/markdown/basics) and [RStudio's Cheatsheet](https://www.rstudio.com/wp-content/uploads/2015/02/rmarkdown-cheatsheet.pdf).

*note: this text borrowed liberally from another class [SciComp2014](http://scicomp2014.edc.uri.edu)*

To get you started, here is some of that same information on the most common markdown you will use in your posts: Text, Headers, Lists, Links, and Images.

### Text

So, for basic text... Just type it!

### Headers

In pure markdown, there are two ways to do headers but for most of what you need, you can use the following for headers:

    # Header 1
    ## Header 2
    ...
    ###### Header 6

### List

Lists can be done many ways in markdown. An unordered list is simply done with a `-`, `+`, or `*`. For example

-   this list
-   is produced with
-   the following
-   markdown.
    -   nested

<pre>    
- this list
- is produced with
- the following 
- markdown
    - nested
</pre>
Notice the space after the `-`.

To create an ordered list, simply use numbers. So to produce:

1.  this list
2.  is produced with
3.  the following
4.  markdown.
    -   nested

<pre>
1. this list
2. is produced with
3. the following
4. markdown.
    - nested
</pre>
### Links and Images

Last type of formatting that you will likely want to accomplish with R markdown is including links and images. While these two might seem dissimilar, I am including them together as their syntax is nearly identical.

So, to create a link you would use the following:

    [Course Website](http://scicomp2014.edc.uri.edu)

The text you want linked goes in the `[]` and the link itself goes in the `()`. That's it! Now to show an image, you simply do this:

    ![CI Logo](http://www.edc.uri.edu/nrs/classes/nrs592/CI.jpg?s=150)

The only difference is the use of the `!` at the beginning. When parsed, the image itself will be included, and not just linked text. As these will be on the web, the images need to also be available via the web. You can link to local files, but will need to use a path relative to the root of the document you are working on. Let's not worry about that. It's easy, but beyond the scope of this tutorial.

Reproducible Documents and Presentations
----------------------------------------

By itself Markdown is pretty cool, but doesn't really provide any value added to the way most of us already work. However, when you add in a few other things, it, in my opinion, changes things dramatically. Two tools in particular that, along with Markdown, have moved reproducible research forward (especially as it relates to R) are, the `knitr` package and a tool called pandoc. We are not going to cover the details of these, but we will use them via RStudio.

In short, these three tools allow us to write up documents, embed code via "code chunks", run that code and render the final document with nicely formatted text, results, figures, etc. into a final format of our choosing. We can create `.html`, `.docx`, `.pdf`, ... The benefit of doing this is that all of our data and code are a part of the document. I share my source document, then anyone can reproduce all of our calculations. For instance, I can make a manuscript that looks like this:

[![Rendered Manuscript](../static/img/rendered.jpg "Rendered Manuscript")](figure/manuscript.pdf)

from a source markdown document that looks like:

[![Raw RMarkdown](../static/img/source.jpg "Raw RMarkdown")](figure/manuscript.Rmd)

While we can't get to this level of detail with just the stock RStudio tools, we can still do some pretty cool stuff. We are not going to do an exercise on this one, but we will walk through an example to create a simple reproducible research document and a presentation using the RStudio interface.

First, let's talk a bit about "code chunks."

### Code Chunks

Since we are talking about markdown and R, our documents will all be R Markdown documents (i.e. .Rmd). To include R Code in your .Rmd you would do something like:

    ```{r}
    x <- rnorm(100)
    x
    ```

This identifies what is known as a code chunk. When written like it is above, it will echo the code to your final document, evalute the code with R and echo the results to the final document. There are some cases where you might not want all of this to happen. You may want just the code returned and not have it evalutated by R. This is accomplished by adding a chunk option:

    ```{r eval=FALSE}
    x <- rnorm(100)
    ```

Alternatively, you might just want the output returned, as would be the case when using R Markdown to produce a figure in a presentation or paper:

    ```{r echo=FALSE}
    x <- rnorm(100)
    y <- jitter(x, 1000)
    plot(x, y)
    ```

Lastly, each of your code chunks can have a label. That would be accomplished with something like:

    ```{r myFigure, echo=FALSE}
    x <- rnorm(100)
    y <- jitter(x, 1000)
    plot(x, y)
    ```

Now, let's get started and actually create a reproducible document

### Create a Document

To create your document, go to File: New File : R Markdown. You should get a window that looks something like:

![New RMarkdown](../static/img/newrmarkdown.jpg)

Add title and author, select "HTML" as the output and click "OK". RStudio will open a new tab in the editor and in it will be your new document, with some very useful examples.

In this document we can see a couple of things. First at the top we see:

    ---
    title: "My First Reproducible Document"
    author: "Author Name"
    date: "1/1/2016"
    output: html_document
    ---

This is the YAML(YAML Ain't Markup Language) header or front-matter. It is metadata about the document that can be very useful. For our purposes we don't need to know anything more about this. Below that you see text, code chunks, and if it were included some markdown. At its core this is all we need for a reproducible document. We can now take this document, pass it through `knitr::knit()` (remember this syntax from the first lesson?) and pandoc and get our output. We can do this from the console and/or shell, or we can use RStudio.

If you look near the top of the editor window you will see:

![knit it](../static/img/knit.jpg)

Click this and behold the magic! It should be easy to see how this could be used to write the text describing an analysis, embed the analysis and figure creation directly in the document, and render a final document. You share the source and rendered document and anyone has access to your full record of that research!

### Create a Presentation

Creating a presentation is not much different. We just need a way to specify different slides.

Repeat the steps from above, but this time instead of selecting "Document", select "Presentation". Only thing we need to know is that a second level header (i.e. `##`) is what specifies the title of the next slide. Any thing you put after that goes on that slide. Similar to before, play around with this, add a slide with some new text, new code and knit it. There you have it, a reproducible presentation.

I know you will probably wonder whether you can change the look and feel of this presentation, and the answer is yes. I have done that, but using a different method for creating slides by using the `slidify` package. An example of that presentation is in a talk I gave on [Social Media and Blogging](http://jwhollister.com/epablogpresent). It does take a bit more work to set this up, but you can make stylish and reproducible slides this way.

Exercise 3
----------

1.  Follow the 'Create a Document' example above to create a new RMarkdown document of your own.
2.  Spend some time playing around with this document. Add in some markdown (e.g., section titles), text, and a code chunk.
3.  Push the `Knit` button to see the outcome.
4.  Challenge: use the `Qall` dataset from `smwrData` to add a plot. Try changing the plot size (hint: chunk options).
