XPath in my experience
======================

> "XPath is used to navigate through elements and attributes in an XML document." W3Schools Description of XPath

#### The following is a set of tips that could come in handy, when writing XPath.
***

## Primer
### Filesystem style traversal through xml documents
`/` From the root of the document

`/example` Select all root nodes named **example**

`/path/to/nodes` Select all elements **nodes**, children of all elements **to**, children of all elements **path**

`//` Search down through everything from the root of the document

`//example` Search down through the entire xml, from the root, for any element named **example**

`/path//to/nodes` Select all elements **nodes**, children of all elements **to**, *descendants*(!) of elements **path**

### Types and Axes
#### Common Types
`element():` `<xml/>`

`attribute():` `<a `**`href="example.com"`**`>example</a>`

`text():` `<a href="example.com">`**`example`**`</a>`

`node():` `<`**`a`****`href="example.com"`**`>`**`example`**`</a>`

#### Axes
##### xml example 
    <a>
        <b/>
        <b/>
    </a>
    <c>
        <d/>
        <d/>
    </c>


###### child::

`/a/child::b` equivalent to `/a/b`

    (<b/>,<b/>)


###### parent::

`//b/parent::a`

    (<a> … </a>)


###### following::

`/a/b[1]/following::*`

    (<b/>,<c> … </c>,<d/>,<d/>)


###### preceding::

`/c/d[1]/preceding::*`

    (<a> … </a>,<b/>,<b/>)

###### following-sibling::

`/c/d[1]/following-sibling::d`

    (<d/>)

###### preceding-sibling::

`/c/preceding-sibling::a`

    (<a> … </a>)


### Filters

#### Simple filters
A way of reducing the resulting sequence based on a set of conditions defined in square brackets.

Example:

`/a[child::b]` (equiv `/a[b]`)

Which will select any element **a**, child of the root of the document. However, it will only select those with a child element named **b**.

Note: Empty sequence in conditional statements are regarded as the boolean false(). Hence why you can just write `/a[b]`.
The filter is applied from the context of those selected previously in the XPath statement (e.g. in the example above **a**).
When an element **a** does not have a child **b** the selector **child::b** (**b**) results in the empty sequence, i.e. `/a[ () ]`
which is treated as `/a[ false() ]` and that **a** is not selected. 

#### Nesting Filters
`//a[b[@c]]` which reads as: Select all elements **a**, which have a child element **b**, which have an attribute named **c**.

### Namespaces

***

## Indexes

### The first element
Sorry guys, things start at **[1]** not **[0]**! So when referencing items from a sequence don't slip up.

### Indexes don't always result in a single resulting item
This is a common mistake, which leads to incorrect arguments being passed to a function. See the following example:

##### <a id="xml">XML</a>
    <a>
      <b>
        <c/>
        <c/>
      </b>
      <b>
        <c/>
      </b>
      <b>
        <c/>
      </b>
    </a>

##### XPath selector
`fn:name(//b/c[1])`

##### Error
`fn:name()` expects a single argument `item()`, however, `item()*` found.


Actual Result of `//b/c[1]`:

`(<c/>,<c/>,<c/>)`

#### Why?
It is often the case that we expect an xpath expression to return a sequence of results. It is also often that we just want the first node in that sequence. However,
what we end up getting isn't always the sequence we expected. If we break down the XPath expression `//b/c[1]`, we can see whats actually going on.

Firstly `//` is referencing from the root of the document all the way down

Secondly `//b`, means we are looking for all elements with the name `b` in the document

What we have so far: `(<b>...</b>,<b>...</b>,<b>...</b>)`

Thirdly `/c[1]` we are looking for the **FIRST** child of the **EACH** node in the current context named `c`.

So we get the first child `c` of **EACH** node `b`.

#### How do we get around this?
1. ##### Specificity

    Be more specific e.g.

    `/a/b[1]/c[1]` will give the first child `<c/>` of the first child `<b/>` of `<a/>`.
    
    `//b[1]/c[1]` in the example will also give the first child `<c/>` of the first occurance**s** of the element `<b/>` in the document.

2. ##### Make sure you're indexing a single sequence
    
    Heres a good rule in XPath, a sequence of sequences is a sequence... Helpful right?
    
    `((<c/>),(<c/>),(<c/>)) Eq (<c/>,<c/>,<c/>)`
    
    If we build a new sequence from our expression results, we can index the first child.
    
    Change `//b/c[1]` to `(//b/c)[1]`.
    
    By explicitly wrapping our lookup for all children `<c/>` of `<b/>` in the sequence constructor `()`, we produce a new sequence of all elements `<c/>`, which we can then reference the first item from.

3. ##### Use a filter
    `//c[parent::b][1]`
    
    Filters are applied across the resulting axes to produce a new axes. This resulting sequence can then be indexed.
    
***

## <a id="func">Functions</a>

A little understanding of determinism in XPath can open up a lot neat solutions.
XPath has a notion of Deterministic and Non-Deterministic functions. As well as functional context and focus dependencies. This is best explained through the following examples:

### <a id="deterministic">Deterministic function (context and focus independent)</a>

`fn:concat($string as xs:string...) as xs:string`

Takes a variable number of string arguments and concatenates them. Why is it deterministic? Because it will always produce the same result for the same set of inputs!
Why is it context and focus independant? Because it doesn't imply any information from the static or dynamic context!

`name($item as item()) as xs:string` (Note the 1 argument version of name($1) not the zero argument function name())

Takes a single item and returns the items respective name.

### <a id="context">Context Dependent Determinstic Functions</a>

`fn:resolve-uri($relative as xs:string?) as xs:anyURI?`

This function is dependent upon the state of the static base uri of the execution context which can be acquired using the `fn:static-base-uri()` function.
If the two argument declaration of this function is called:

`fn:resolve-uri($relative as xs:string?, $base as xs:string) as xs:anyURI?`

Then it is a context independent function, as the second argument is used in place of the `fn:static-base-uri()` function. Making the deterministic regardless of the contextual or focus dependencies.

### <a id="focus">Focus Dependant Deterministic Functions</a>
Note: Focus dependency is a subset of context dependency. Therefore, it isn't possible to have a focus dependent function that is context independent.

The zero argument function name:

`fn:name() as xs:string`

This function is dependent upon the implicit context item, e.g. the XPath statement `//b/c/name()`.
This function is mapped accross the sequences of all children 'c' of all nodes 'b' with the implicit argument of the context item `<c/>`.

In the [XML](#xml) example under 'Indexes' the XPath above would give the result sequence `(<c/>,<c/>,<c/>,<c/>)`

Calling `//b/name()` over the same example will give `(<b/>,<b/>,<b/>)`

The single argument function `fn:name($arg as node()?) as xs:string` function requires an explicit passing of the context item, as described in the section on [Context Dependent Deterministic Functions](#context).

Calling this function
in different contexts, will lead to the same output.

`/a//b/name(/a/b[1])` will result in 'b' for each call on the context node 'b'

`/a//b/c/name(/a/b[1])` will also result in 'b' for each call on the context node 'c' child of each 'b'

However, calling the zero argument function, within different contexts, will lead to different
outputs. This isn't to say it is non-deterministic, but that it is deterministic with regards to the focus.

