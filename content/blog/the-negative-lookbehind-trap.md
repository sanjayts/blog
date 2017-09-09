+++
date = "2017-09-02T12:00:00+06:00"
lastmod = "2017-09-02T22:00:00+06:00"
title = "The Negative look-behind trap -- regex"
categories = ["programming"]
tags = ["regex", "python"]
+++
----------------------------

A request was made in this [Daniweb thread](http://www.daniweb.com/software-development/java/threads/331201/java-split-text-by-spaces-and-punctuation#post1798456) to split a string based on punctuations and whitespaces but with a twist. The splitting should make exceptions/allowances for special cases.

For e.g. let's say we are splitting on `.` (the dot character). Our splitting should keep titles like `Mrs.` and `Mr.` intact. What does this mean? Let's have a look at it with an example (for those wondering, the code snippets are in Python programming language):

```python
# Regular splitting based on punctuation (we need the backslash before the dot
# because we need to make the regex engine aware that we are looking for a literal
# dot and not the regex metacharacter which matches anything.
>>> print re.split(r'\.', 'a.b.c.Mr.d.Mrs.i.p')
['a', 'b', 'c', 'Mr', 'd', 'Mrs', 'i', 'p']

# Special splitting
>>> print re.split(r'WE_NEED_SOMETHING_HERE', 'a.b.c.Mr.d.Mrs.i.p')
['a', 'b', 'c', 'Mr.', 'd', 'Mrs.', 'i', 'p']
```

As you can see from the example above, we do split the string based on the dot character but skip the splitting in case of titles like `Mr.` and `Mrs.`. How would we go about doing it with regular expressions? Well, [lookaround assertions](http://www.regular-expressions.info/lookaround.html) to the rescue, or something like that. ;) The most obvious fix here would be to just add negative lookbehinds and be done with it, right?

```python
# Splitting based on lookbehind
>>> print re.split(r'(?<!Mr)\.', 'a.b.c.Mr.d.Mrs.i.p')
['a', 'b', 'c', 'Mr.d', 'Mrs', 'i', 'p']
```

Whoa, we are looking good here. For those who haven't noticed, we skipped the `.` following `Mr` and hence `Mr.` was included in the split result. If it makes any easier, compare this output with the output of example one, it should make things clear. So, one more lookbehind and we should be golden. Let's try that:
```python
# Splitting based on lookbehind
>>> print re.split(r'(?<!Mr)\.|(?<!Mrs)\.', 'a.b.c.Mr.d.Mrs.i.p')
['a', 'b', 'c', 'Mr', 'd', 'Mrs', 'i', 'p']
```

Huh, what happened? Here's what happened:

* You ask the regex engine to split based on the alternations. So our regex literally reads as "try to split based on dot unless it is followed by the literal `Mr` **OR** try to split based on dot unless it is followed by `Mrs`"
* The splitting of first three dots is as expected i.e. `a.b.c.REM` results in the parts as `a`, `b`, `c` and the `REMAINING STRING`. This is because the negative lookbehind fails for this section of the string and hence normal splitting based on dot occurs.
* Now comes the tricky part; the regex engine is now positioned on the dot after `Mr`. It does a negative lookbehind assertion and confirms that we should ignore the dot if it is preceded by `Mr`. *But* it doesn't end here. We have an alternation in our regex.
* Now that the first alternation fails, the regex engine moves on to test the next altenation, which is `(?<!Mrs)\.`. This negative lookbehind assertion is not satisfied because we are right now at the dot character which is preceded by `Mr` but we are checking for `Mrs`. So basically, the second alternation test passes and the dot character is used for splitting the string.
* This is totally unexpected behaviour but a side-effect of chaining negative lookbehinds in an alternation. Think of it as an "OR" test used in programming languages. If *any one* of the condition is satisfied, the check succeeds which is counter-productive to our case here.

So, what's the solution? The solution is to ensure that for baking an exception in your regex (i.e. split based on dot char only if it isn't preceded by `Mr` or `Mrs`) you club the similar negative lookbehinds instead of keeping them separate. A logical solution here would be:
```python
>>> print re.split(r'(?<!Mr|Mrs)\.', 'a.b.c.Mr.d.Mrs.i.p')
['a', 'b', 'c', 'Mr', 'd', 'Mrs', 'i', 'p']
```

Alas, it doesn't work with Python. The error which greets you is: "sre_constants.error: look-behind requires fixed-width pattern". What is this trying to say to us? It means that the pattern used in lookbehind needs to be fixed width. So basically no `+` or `*`. In our case, we have an alternation with one pattern having length as 2 and the other pattern having length as 3 which causes the failure. But fear not, because we have a simple workaround which can help us get the correct value. Note that we don't care what precedes `Mr` as long as it is followed by the dot char. We can tweak/modify the regex a bit to make it fixed length (i.e. length 3) by adding a dot metacharacter in the mix. Here is something which should work:
```python
>>> print re.split(r'(?<!.Mr|Mrs)\.', 'a.b.c.Mr.d.Mrs.i.p')
['a', 'b', 'c', 'Mr.d', 'Mrs.i', 'p']
```

For those who didn't notice, I added a `.` just before the `Mr` in the alternation. Here is hoping this piece of write-up might help you in case you are stuck with a similar problem! :)
