Mostache == Mo Mustache
========


### What and Why
Mostache has a modest goal: unobtrusively add a few key features to standard mustache templates while changing the original mustache library code as little as possible.

I love mustache, but I felt it could be slightly easier to feed it JSON without re-inventing the wheel or adding dozens of kilobytes of parsing code and while making sure that Mustache performance doesn't suffer from the modification. As a result, virtually any valid mustache template can be used with mostache, so upgrading is easy. 

Try online examples at <http://danml.com/mostache/>


### New Features

#### {INDEX} mini-tag
  Note the single-brace. Returns the current index when iterating an Array.  <br />
    `{{#persons}}<li> #{INDEX}. {{firstName}} {{lastName}} </li> {{/persons}}`

#### {SEP} mini-section
   Note the single-brace. Returns the enclosed block for every value except for the last. <br />
    `{SEP} <br /> {/SEP}`

#### {{!path}} else synax
  `{{!path}}` turns into `{{/path}}{{^path}}`, for simpler _else_ handling. <br />
    `<p>{{#ok}}Y{{!ok}}N{{/ok}}</p>` == `<p>{{#ok}}Y{{/ok}}{{^ok}}N{{/ok}}</p>`
    
#### {{#k=v}} conditionals
  Compares are made against a _primitive_ value to the right of the `=`, no code/path/expressions evaluated. <br />
  `{{#sec=main}}Home{{/sec=main}}` turns into `Home` with `{sec:"main"}` and nothing with `{sec:'about'}` . <br />
  `{{#a.length=3}}{{.}}{{/a.length=3}}` turns into `abc` with `{a:"abc"}` and nothing with `{a:'a'}` . <br />
  For performance reasons, long paths are not parsed, only one dot to the left of the equal, but wrapper blocks can drill.
  
#### {{#k!=v}} NOT conditionals
  Compares are made against a _primitive_ value to the right of the `=`, no code/path/expressions evaluated. <br />
  `{{#sec=!main}}Home{{/sec=!main}}` turns into `Home` with `{sec:"about"}` and nothing with `{sec:"main"}` . <br />
  `{{#a.length!=3}}{{.}}{{/a.length!=3}}` turns into `abc` with `{a:"a"}` and nothing with `{a:'abc'}` . <br />
  For performance reasons, long paths are not parsed, only one dot to the left of the equal, but wrapper blocks can drill.


#### <@macro@> synax
  Using `<@` and `@>` in a template invokes macro mode, where the template is run twice.<br  />
  First, delimeters are switched to `<@` and `@>` and the template is applied  with normal data. <br />
  Then, the result is re-fed as to re-render normally. This can be used for sub-queries, marcos, even UI generation. <br />
`Mustache.to_html("Selected: {{data.<@selected@>}}", {data:["A","B","C"], selected: 1})` yields `Selected: B` <br />
The internal intermediate template for the above looks like `Selected {{data.1}}` after the first pass.

#### {{#obj:key}} object iteration
 Iterates over objects using a placeholder name on the section tag, prefixed by ":". <br />
 Inside the section, the key as a tag will equal the name of the object property's key. <br />
`{{#a:k}}{{k}}={{.}} {{/a:k}}` turns into `b=1 c=5 ` with `{a:{b:1,c:5}}`

#### {{__.key}} root synax
  `{{__.key}}` reaches _key_ on the data object given to Mustache, bypassing local conflicts. <br />
  `{{#b}}{{a}}|{{__.a}}{{/b}}` turns into `1|123` with `{a:123, b:[{a:1}]}`


#### parameters for partials
  `{{> mailto email}}` passes an argument of `email` to a function partial. <br />
  This can be used to make more powerful/flexible custom helpers, and help to avoid hard-coding helper to data:  <br />
  `{{name}} ({{> mailto email}})` with `{name:"Mary", email: "mary@example.com"}` where <br />
  
    function(method, args, context){
      switch(method){ // values can be          interpolated    -or-  realized now
        case "mailto": return '<a href="mailto:{{'+args[0]+'}}">'+context[args[0]].toUpperCase()+'</a>' ;break;
      } 
    }
is the partial yields `Mary (<a href="mailto:mary@example.com">MARY@EXAMPLE.COM</a>)` <br />
The original mustache would need `email` hard-coded into the partial above.


#### razor syntax
  Activated by including `{{@@}}` inside a template, allows a leaner razor-style syntax <br />
  It replaces `{{` with `@` and `}}` with ` `, allowing leaner {{[#^/]section}} and {{name}} tags <br />
  It needs a non-letter to the left of the `@` so as to avoid emails, but regular mustache works as well. <br />
  This syntax is enabled via one RegExp replace() on the template before compilation
```
Mustache.to_html(
  "{{@@}} <ul> @#actors <li>@name  @/actors", 
  { actors: [ 
          { name: "Jeff Bridges" },
          { name: "John Goodman" },
          { name: "Steve Buscemi" }
      ]
}); 
```
Which is pre-converted internally to the template: <br />
`<ul> {{#actors}} <li>{{name}}  {{/actors}}` <br />
Which renders html as:
* Jeff Bridges
* John Goodman
* Steve Buscemi


#### native method detection
Normally mustache passes any functions in the data to a special helper function, which doesn't work with native methods like `"".toUpperCase()`. Mostache detects natives methods in data and runs them against the data itself. it was always awkward to try to mash in methods to JSON data, but now at least the handy primitive native methods work without fuss. 

It's harder to explain than use. ex: `{{name.toUpperCase}}` now produces the data's name property's value in UPPERCASE, instead of the useless "[object Object]". Regular functions in your data still act as expected, it's only the existing hidden primitive methods in your data that run smarter. Mostly, these are for strings, but `{{number.toFixed}}` will produce an integer from a number, and `{{array.sort}}` will sort an array as alphabetical case-sensitive text.

#### {|helpers}
The biggest improvement: calling helpers from the template instead of the data. Placed after a normal mustache path, filters pass the value to a filter for processing, formatting, or selection. You can use many filters on the same value by separating each one with a pipe ("|"), and the result will be passed left to right through each of the filters. The function is reached by it's gloabl JS path, so any object or function can be used to process, without registering to mustache ahead of time. 

Also note that if no value if found by the path/text to the left of the first "|", the path itself as a string (or nothing) will be used instead. That allows a particularly neat way of using JS mid-template: `{{location|eval}}`, or just injecting function returns: `{{|Date}}`. 

[Live Demo of helpers, adapted from handlbars demo](http://pagedemos.com/4cy5k6jwxyrf/) <br />

Finally, a special method indicator prefix (".") in front of the method name forces a method of the value to be executed, for example `"".toUpperCase()` as `{{name|.toUpperCase}}`.



