Mostache == Mo Mustache
========


### What and Why
Mostache has a modest goal: unobtrusively add a few key features to standard mustache templates while changing the original mustache library code as little as possible (~8 meaningful splices).

I love mustache, but I felt it could be slightly easier to feed it JSON without re-inventing the wheel or adding dozens of kilobytes of parsing code and while making sure that Mustache performance (currently great with version 0.7+) doesn't suffer from the modification. As a result, virtually any valid mustache template can be use with mostache, so upgrading is easy. Once you switch from mustache to mostache, you pickup the following features:

### New Features

#### {INDEX} mini-tag
  Note the single-brace. Returns the current index when iterating an Array.  <br />
    `{{#persons}}<li> #{INDEX}. {{firstName}} {{lastName}} </li> {{/persons}}`

#### {SEP} mini-section
   Note the single-brace. Returns the enclosed block for every value except for the last. <br />
    `{SEP} <br /> {/SEP}`

#### native method detection
Normally mustache passes any functions in the data to a special helper function, which doesn't work with native methods like `"".toUpperCase()`. Mostache detects natives methods in data and runs them against the data itself. it was always awkward to try to mash in methods to JSON data, but now at least the handy primitive native methods work without fuss. 

It's harder to explain than use. ex: `{{name.toUpperCase}}` now produces the data's name property's value in UPPERCASE, instead of the useless "[object Object]". Regular functions in your data still act as expected, it's only the existing hidden primitive methods in your data that run smarter. Mostly, these are for strings, but `{{number.toFixed}}` will produce an integer from a number, and `{{array.sort}}` will sort an array as alphabetical case-sensitive text.

#### {|filters}
The biggest improvement: calling helpers from the template instead of the data. Placed after a normal mustache path, filters pass the value to a filter for processing, formatting, or selection. You can use many filters on the same value by separating each one with a pipe ("|"), and the result will be passed left to right through each of the filters. The function is reached by it's gloabl JS path, so any object or function can be used to process, without registering to mustache ahead of time. 

Also note that if no value if found by the path/text to the left of the first "|", the path itself as a string (or nothing) will be used instead. That allows a particularly neat way of using JS mid-template: `{{location|eval}}`, or just injecting function returns: `{{|Date}}`. 

Finally, a special method indicator prefix (".") in front of the method name forces a method of the value to be executed, for example `"".toUpperCase()` as `{{name|.toUpperCase}}`.



