---
layout: post
title:  "Fun with jsbeautifier"
subtitle: a better way to hide your code *
permalink: 2015-08-28-fun-with-js-beautifier
date:   2015-08-28
categories: jsbeautifier deobfuscation packing javascript 
---

We all know [jsbeautifier](http://jsbeautifier.org) and its friendly feature of making this:

{% highlight js linenos %}
var names=["John", "Jane", "George", "Gene" ];var greet=function(name) {return "hello "+name;};names.forEach(function(name) {greet(name);});
{% endhighlight %}

into this:
{% highlight js linenos %}
var names = ["John", "Jane", "George", "Gene"];
var greet = function(name) {
    return "hello " + name;
};
names.forEach(function(name) {
    greet(name);
});
{% endhighlight %}

However, if you'll read its description closely, you'll see that it's more than just a beautifier:

> Beautify, unpack or deobfuscate JavaScript and HTML, make JSON/JSONP readable, etc.

## Unpacking? deobfuscating?

before we go any further, here are the two definition, taking straight from wikipedia:

### Packing
> * Reducing the redundancy in the script (by removing comments, white space and shorten variable and functions names). This does not alter the behavior of the script.

> * Compressing the original script and create a new script that contains decompression code and compressed data. This is similar to binary executable compression.

###Obfuscation

> In software development, obfuscation is the deliberate act of creating obfuscated code, i.e. source or machine code that is difficult for humans to understand. Like obfuscation in natural language, it may use needlessly roundabout expressions to compose statements.


So in a short, jsbeautifier claims it can reverse the two.

## So jsbeautifier can reverse the process?
Well yes and no: it uses some of the more known packing/obfuscating methods and tries its best.

One of the more famous packers, supported by jsb, can be found on [dan's Tools](http://www.danstools.com/javascript-obfuscate/).
It can take a JS input, let's say the above, and turn it into something like this:

{% highlight js linenos %}
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('2 1=["6","5","7","b"];2 4=3(0){a"8 "+0};1.9(3(0){4(0)});',12,12,'name|names|var|function|greet|jane|john|george|hello|forEach|return|jene'.split('|'),0,{}))
{% endhighlight %}

The magic happens when you'll take the code above, and place it inside jsb:

{% highlight js linenos %}
var names = ["john", "jane", "george", "jene"];
var greet = function(name) {
    return "hello " + name
};
names.forEach(function(name) {
    greet(name)
});
{% endhighlight %}

Violla! completely reversed. 

## But how does this work?
Like i've mentioned earlier, jsb will use all sorts of detections in order to identify one of the packers/obfuscators it supports.
It will then use its own unpacking methods to reverse the process, some are as simple as evaluating the script.

The code for this specific packer, can be found right [here](http://jsbeautifier.org/js/lib/unpackers/p_a_c_k_e_r_unpacker.js)
and the detection regex looks like this:

{% highlight js linenos %}
var P_A_C_K_E_R = {
    detect: function (str) {
        return (P_A_C_K_E_R.get_chunks(str).length > 0);
    },

    get_chunks: function(str) {
        var chunks = str.match(/eval\(\(?function\(.*?(,0,\{\}\)\)|split\('\|'\)\)\))($|\n)/g);
        return chunks ? chunks : [];
    },
    
    ...
});
{% endhighlight %}

what it actually does is a simple [string match](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/match),

## The fun part
Looking carefully (or not), we can reduce the matching string into the following:

{% highlight js linenos %}
eval(function() {}(''.split('|')))
{% endhighlight %}

if we'll put it inside jsb, we'll get a simple, yet a very interesting response:

{% highlight js linenos %}
undefined
{% endhighlight %}

it basically gives us  the output of the only function there, let's try another:

{% highlight js linenos %}
eval(function() { return "dude"}(''.split('|')))
> dude
{% endhighlight %}

Congratulations. you can now control the output of jsb. Let's take it a step further.

Assuming the following code:

{% highlight js linenos %}
eval(function(arr){ return document.domain == "example.com" ? arr[0] : arr[1] }('"private stuff"|"public stuff"'.split('|')))
{% endhighlight %}

Looking only at the function, we can see that it will return "private stuff" when executed under example.com,
or "public stuff" when executed elsewhere, give it a go by going to [example.com](example.com), opening the console and executing the code above.

By harnesing jsb's nature, we can "deobfuscate" it into just the "private stuff", removing everything else.

We can even take it a step further and packing it once more, resulting with:

{% highlight js linenos %}
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('c(d(2){7 6.5=="4.8"?2[0]:2[1]}(\'"b 3"|"a 3"\'.9(\'|\')))',14,14,'||arr|stuff|example|domain|document|return|com|split|public|private|eval|function'.split('|'),0,{}))
{% endhighlight %}

Which will work perfectly while executed or when being deobfuscated by jsb.

I'm sure you can think of all sorts of uses for a thing like that;
I personally might hide a message for someone who not only uses tools, but also knows how they work.


\* This is fun, but far from secure.