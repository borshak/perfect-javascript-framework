Nowadays, in front-end development, we have many frameworks and libraries. Some
of them are good, some of them are not. Often we only like a particular concept,
module or maybe a certain syntax. The truth is that there is no universal 
instrument. This article is about the future framework — the framework that does
not exist yet. I’ve summarized the pros and cons of some of the available 
JavaScript frameworks and I dare to dream about the perfect solution.

## Abstraction is dangerous {#abstraction-is-dangerous}

We all like simple tools. Complexity kills. It makes our work difficult and
gives us much steeper learning curve. Programmers need to know how things work. 
Otherwise, they feel insecure. If we work with a complex system, then we have a 
big gap between “I am using it” and “I know how it works”. For example, code 
like this hides complexity:

    var page = Framework.createPage({
    	'type': 'home',
    	'visible': true
    });
    

Let’s say that this is a real framework. Behind the scenes, `createPage`
generates a new*view* class that loads the `home.html` template. Based on the
`visible` parameter we append (or not) the newly created DOM element to the
tree. Now, let’s put ourselves in the developer’s shoes. We read in the 
documentation that this creates a new page with a certain template. We do not 
know any particular details because this is an abstraction.

Some of today’s frameworks have not one, but many levels of abstractions.
Sometimes, in order to use the framework properly, we have to know the details. 
Abstracting, in fact, is a powerful instrument because it wraps functionalities.
It encapsulates design decisions. However, it should be used wisely because it 
leads to untraceable processes.

What if we transform the above example to the following:

    var page = Framework.createPage();
    page
    	.loadTemplate('home.html')
    	.appendToDOM();
    

Now the developer knows what is going on. The template and the appending are
exposed to different API methods. So, he/she can do something in between and 
control the processes.

Let’s take [Ember.js][1]. It is a great framework. It gives us the power of
building single page applications with just a few lines of code. However, this 
comes at its own price. It defines several classes behind the scenes. For 
example:

    App.Router.map(function() {
    	this.resource('posts', function() {
    		this.route('new');
    	});
    });
    

The framework creates three routes, and each one of them has a controller
attached. You may or may not use these classes, but they exist. The framework 
needs them to power up the application.

Very often, our project requires custom functionality. There is no framework
that covers all use cases. So, we meet problems that do not have simple 
solutions. We have to understand how everything works in order to find the right
way of doing things. Every other direction that we take looks more like hacking 
the framework and not playing with it.

[Backbone.js][2], for example, introduces just a few predefined objects. They
contain the core functionality, but the real implementation is up to the 
programmer. The`DocumentView` class below extends `Backbone.View`. That’s it
. We have only one level between the code that we use and the framework’s core 
features.

    var DocumentView = Backbone.View.extend({
    	'tagName': 'li',
    	'events': {
    		'mouseover .title .date': 'showTooltip',
    		'click .open': 'render'
    	},
    	'render': function() { ... },
    	'showTooltip': function() { ... }
    });
    

Personally, I prefer to use a framework that doesn’t have many levels of
abstractions — a framework that provides transparency.

## The missing constructor {#the-missing-constructor}

Some of the frameworks accept our class definitions, but they do not produce
constructors. The framework decides where and when to create an instance. I 
would like to see more frameworks that let us do exactly this. For example, in
[Knockout][3]:

    function ViewModel(first, last) {
    	this.firstName = ko.observable(first);
    	this.lastName = ko.observable(last);
    }
    ko.applyBindings(new ViewModel("Planet", "Earth"))
    

We define our model, and we initialize it. In AngularJS, for example, it is a
bit different:

    function TodoCtrl($scope) {
    	$scope.todos = [
    		{ 'text': 'learn angular', 'done': true },
    		{ 'text': 'build an angular app', 'done': false }
    	];
    }
    

We, again, define our class, but we are not running it. We just say that this
is our controller, and the framework decides how to process it. We may find this
confusing because we lose the key points — the key points that we use to 
visualize the application’s flow.

## DOM manipulations {#dom-manipulations}

Whatever we do, we need to interact with the DOM. It is important how we do it
, because typically, every manipulation of the nodes on the page fires a reflow 
or a repaint, which can be expensive operations. Let’s, for example, consider 
the usage of the following JavaScript class:

    var Framework = {
    	'el': null,
    	'setElement': function(el) {
    		this.el = el;
    		return this;
    	},
    	'update': function(list) {
    		var str = '<ul>';
    		for (var i = 0; i < list.length; i++) {
    			var li = document.createElement('li');
    			li.textContent = list[i];
    			str += li.outerHTML;
    		}
    		str += '</ul>';
    		this.el.innerHTML = str;
    		return this;
    	}
    }
    

This little framework generates an unordered list with given data. We send the
DOM element, which will host a list, and call the`update` function that shows
the data to the screen.

    Framework
    	.setElement(document.querySelector('.content'))
    	.update(['JavaScript', 'is', 'awesome']);
    

The result after running the code is as follows:<figure class="figure">

![][4]</figure>
In order to illustrate why this is bad design, we will add a link to the page
and will attach a`click` event listener. The function will call the `update`
method again but with different items.

    document.querySelector('a').addEventListener('click', function() {
    	Framework.update(['Web', 'is', 'awesome']);
    });
    

We’re sending almost the same data; we only change the first element of the
array. However, because we are using`innerHTML`, a repaint is triggered after
each click. The browser does not know that we need to modify only the first row.
It repaints the whole list. Let’s use Opera’s DevTools and run the profile. 
Check out the following animated GIF demonstrating the result:<figure class="
figure
">

![][5]</figure>
Notice that after each click the whole content is repainted. This is a problem
, especially if we use the same technique heavily on the page.

It is much better if we remember the created `<li>` nodes and only update
their content. That way, we are not modifying the whole list but only its 
children. The first change that we have to make is in`setElement`:

    setElement: function(el) {
    	this.list = document.createElement('ul');
    	el.appendChild(this.list);
    	return this;
    }
    

Like this, we do not need a reference to the host element anymore. We just need
to create a new`<ul>` element and append it once.

The logic that improves the performance is in the body of the `update` method
:

    'update': function(list) {
    	for (var i = 0; i < list.length; i++) {
    		if (!this.rows[i]) {
    			var row = document.createElement('LI');
    			row.textContent = list[i];
    			this.rows[i] = row;
    			this.list.appendChild(row);
    		} else if (this.rows[i].textContent !== list[i]) {
    			this.rows[i].textContent = list[i];
    		}
    	}
    	if (list.length < this.rows.length) {
    		for (var i = list.length; i < this.rows.length; i++) {
    			if (this.rows[i] !== false) {
    				this.list.removeChild(this.rows[i]);
    				this.rows[i] = false;
    			}
    		}
    	}
    	return this;
    }
    

The first `for` loop goes through the data that is passed in and creates 
`<li>` elements if necessary. `this.rows` keeps the created tags. If
there is a node at a certain index, the framework updates its`textContent`
property when applicable. The loop at the end removes nodes if the passed array 
has fewer elements than the current one.

Here is the result:<figure class="figure">

![][6]</figure>
The browser repaints only the part that is changed.

The good news is that frameworks like [React][7] are already handling the DOM
manipulations correctly. The browsers become smarter and they also apply tricks 
to decrease the repaints. However, it is always good to have this in mind and 
check what our framework of choice provides.

I hope that in the near future we will be able to stop thinking about such
problems: frameworks should automatically cover them for us.

## DOM events handling {#dom-events-handling}

JavaScript based applications usually communicate with the users through DOM
events. The elements on the page dispatch messages and our code processes them. 
Here is a piece of Backbone.js code that performs an action when the user 
interacts with the page:

    var Navigation = Backbone.View.extend({
    	'events': {
    		'click .header.menu': 'toggleMenu'
    	},
    	'toggleMenu': function() {
    		// ...
    	}
    });
    

So, there should be an element matching the `.header.menu` selector and once
the user clicks on it we have to toggle the menu. The problem with this design 
is that we bind the JavaScript object to one particular DOM item. If we tweak 
the HTML and change`.menu`. to `.main-menu` we have to modify the JavaScript
too. I believe that our controllers should be independent, and we should 
decouple them from the DOM.

By defining functions, we delegate tasks to our JavaScript classes. If these
tasks are handlers of DOM events, then it makes sense to construct them in the 
HTML.

I like how AngularJS handles events:

    <a href="#" ng-click="go()">click me</a>
    

`go` is a function registered in our controller. Following this approach, we do
not have to think about DOM selectors. We just apply behavior directly to the 
HTML nodes. It is a powerful approach because we skip the boring interaction 
with the DOM.

In general, I would like to see such kind of logic inside HTML. Interestingly,
we spent ages convincing developers to split the content (HTML) and the behavior
(JavaScript); we taught them to avoid inline styling and scripting. However, now
I see that doing this actually can save us much time and makes our components 
flexible. Of course, I don’t mean code like this:

    <div onclick="javascript:App.doSomething(this);">banner text</div>
    

Instead, I’m talking about descriptive attributes that control the behavior
of the element. For example:

    <div data-component="slideshow" data-items="5" data-select="dispatch:selected">
    	...
    </div>
    

It should not be like JavaScript coding in HTML, but more like setting
configurations.

## Dependency management {#dependency-management}

Managing the dependencies is an important job in our development process. We
usually depend on external functions, modules or libraries. In fact, we are 
producing dependencies all the time. We don’t write everything into one method. 
We split the application’s tasks into functions and wire them. In the ideal case,
we want to encapsulate logic into modules that behave like black boxes. They 
know details only about their job and nothing else.

[RequireJS][8] is one of the popular instruments for resolving dependencies.
The idea is to wrap your code in a closure that accepts the needed modules:

    require(['ajax', 'router'], function(ajax, router) {
    	// ...
    });
    

In the example above, our function needs two modules — `ajax` and `router`. The
magical`require` method reads the passed array and calls our function with the
proper arguments. The definition of the`router` looks like this:

    // router.js
    define(['jquery'], function($) {
    	return {
    		'apiMethod': function() {
    			// ...
    		}
    	}
    });
    

Notice that we have another dependency here — jQuery. It’s also important
to mention that we have to return our module’s public API. Otherwise, the code 
which requires our module can’t access the defined functionalities.

AngularJS goes a little bit further by giving us something called *factory*. We
register our dependencies there, and they are[magically][9] available in our
controllers. For example:

    myModule.factory('greeter', function($window) {
    	return {
    		'greet': function(text) {
    			alert(text);
    		}
    	};
    });
    function MyController($scope, greeter) {
    	$scope.sayHello = function() {
    		greeter.greet('Hello World');
    	};
    }
    

In general, this approach simplifies our job. We don’t have to use a function
like`require` to fetch the dependency. All we have to do is to type the right
words in the arguments’ list.

Ok, these two ways of dependency injection work, but they are bound to a
specific style of code writing. In the future, I would like to see frameworks 
that eliminate this constraint. It will be much elegant if we are able to apply 
metadata during the variables’ definition. The language right now doesn’t offer 
such capabilities. It will be nice if the following is possible:

    var router:<inject:Router>;
    

Placing the dependency along with the variable’s definition means that we
will perform the injection only if needed. RequireJS and AngularJS for example 
work on a functional level. So, you may use a module only in specific cases, but
the initialization and its injection happen every time. There is also a specific
place where we have to define our dependencies. We are bound to that.

## Templates {#templates}

We use template engines a lot. And we do so because we need to distinguish the
data from the HTML markup. How do today’s frameworks handle templates? Here are 
the most popular approaches:

### The template is defined in a `<script>`  {#the-template-is-defined-in
-a-script
}

    <script type="text/x-handlebars">
    	Hello, <strong> </strong>!
    </script>
    

This is used often because the template is placed in the HTML. It looks natural
, and it makes sense because HTML naturally contains tags. The browser doesn’t 
render the contents of`<script>` elements so it doesn’t mess up the
page’s layout.

### The template is loaded using Ajax {#the-template-is-loaded-using-ajax}

    Backbone.View.extend({
    	'template': 'my-view-template',
    	'render': function() {
    		$.get('/templates/' + this.template + '.html', function(template) {
    			var html = $(template).tmpl();
    		});
    	}
    });
    

We place our code into external HTML files and avoid the usage of additional 
`<script>` tags. However, this means that we need more HTTP requests
which is not always appropriate (at least not until HTTP2 becomes more 
widespread
).

*   The template is part of the page’s markup — the framework reads the
    template from the DOM tree. It relies on already generated HTML. We don’t have 
    to perform additional HTTP requests, create a new file or add additional
   `<script>` elements.

### The template is part of the JavaScript {#the-template-is-part-of-the-
javascript
}

    var HelloMessage = React.createClass({
    	render: function() {
    		// Note: the following line is invalid JavaScript.
    		return <div>Hello {this.props.name}</div>;
    	}
    });
    

This approach introduced by React uses its own parser that converts the invalid
part of the JavaScript to valid code.

### The template is not HTML {#the-template-is-not-html}

Some frameworks don’t use HTML directly at all. They use templates in the
form of JSON or YAML.

### Final thoughts about templates {#final-thoughts-about-templates}

Ok, where do we go from here? The framework of the future should make us think
only about the data and only about the markup. Nothing in between. We don’t want
to deal with loading HTML strings or passing data to special functions. We want 
to apply values to variables and get the DOM updated. The popular*two-way data
binding* should not be a feature, but a *must-have* core functionality.

In fact, AngularJS is close to the desired behavior. It reads the template from
the provided page’s content and has the magical data binding implemented. 
However, it is still not ideal. Sometimes there is a flickering effect. It 
happens when the browser renders the HTML but AngularJS’s boot mechanisms are 
still not fired. Also, AngularJS uses*dirty checking* to find out what is
changed. This approach could cost a lot in some cases. Hopefully
[`Object.observe`][10] will soon be supported in all browsers, so that we have
better data binding.

The question about dynamic templates comes up to every developer sooner or
later. For sure, we have parts of our application that appear after the 
bootstrapping. The framework ought to handle that easily. We shouldn’t think 
about Ajax requests, and we should work with an API that makes the process look 
synchronous.

## Modularity {#modularity}

I like the idea of turning features off and on. If we don’t use something,
then why is it in our code base? It would be nice if the framework has a builder
that generates a version containing only modules that we need. Like, for example
[YUI][11], which has a configurator. We choose the modules that we want and get
a minified JavaScript file ready to use.

Even now, there are frameworks that have something usually called *core*.
Additionally we are able to use bunch of plugins (or modules). However, we could
improve that. The process of choosing the needed features shouldn’t involve 
downloading files. We should not include them manually in the page. It ought to 
be somehow part of the framework’s code.

After having appropriate setup capabilities, the perfect environment must
provide extensibility. We should be able to write our own modules and share them
with other developers. In other words, there should be a friendly environment 
for creating modules. We can’t develop a strong community without the existence 
of a proper developer environment.

## Public APIs {#public-apis}

So far, most of the frameworks provide APIs for their core functionalities.
However, these APIs give access to parts that vendors think we need. And that’s 
the place where hacking becomes an option. We want to achieve something, but we 
don’t have the right instruments. We trick the framework with some ugly 
workarounds. Let’s have a look at the following example:

    var Framework = function() {
    	var router = new Router();
    	var factory = new ControllerFactory();
    	return {
    		'addRoute': function(path) {
    			var rData = router.resolve(path);
    			var controller = factory.get(rData.controllerType);
    			router.register(path, controller.handler);
    			return controller;
    		}
    	}
    };
    var AboutCtrl = Framework.addRoute('/about');
    

We have a framework that has a built-in router. We define a path, and our
controller is automatically initialized. Once the user goes to the right URL, 
our router fires the`handler` method of the controller. That’s great, but
what happens if we need to execute a simple JavaScript function as a response to
the URL matching? For some reason, we don’t want to create a new controller. 
This is not possible with the current API.

We could use another design like this one, for example:

    var Framework = function() {
    	var router = new Router();
    	var factory = new ControllerFactory();
    	return {
    		'createController': function(path) {
    			var rData = router.resolve(path);
    			return factory.get(rData.controllerType);
    		}
    		'addRoute': function(path, handler) {
    			router.register(path, handler);
    		}
    	}
    }
    var AboutCtrl = Framework.createController({ 'type': 'about' });
    Framework.addRoute('/about', AboutCtrl.handler);
    

Notice that we are not exposing our router. It is not visible, but now we have
control of the two processes — creating the controller and registering a route. 
Of course, the proposed design matches our personal use case. We may find this 
approach much complex because we have to create controllers manually. While we 
design APIs, we have to think about the[single responsibility principle][12]
and the idea of*doing one thing and doing it right*. I’m seeing that more and
more frameworks decentralize their functionalities. They split the complex 
methods in smaller and smaller pieces. That’s a good sign, and I hope that we 
will see more of this in the future.

## Testability {#testability}

There is no need to convince you to write tests for your code. The point is not
only in writing tests, but writing testable code. Sometimes this is extremely 
difficult and takes time. I’m sure that if we miss a test for something, even 
small, that’s the exact place where our application will start getting buggy. 
This is especially the case if we talk about client-side JavaScript. Several 
browsers, several operating systems, new specs, new features and their polyfills:
there are so many reasons to start using test-driven development.

There is something else that we get from having tests. We are not only proving
that our framework (application) works today. We make sure that it will work 
tomorrow and even after that. If there is a new feature that has to land in the 
code base, we write a test for it. And it is important that we make this test 
pass. However, it is also important that our previous tests pass as well. This 
is how we guarantee that we didn’t break anything.

I’d like to see more standardized tools and methods for testing. I wish I
could use only one tool and test every framework with it. It’s also good if the 
testing is somehow integrated into the development process. Services like
[Travis][13] need more attention. They act as an indicator not only for the
programmer that makes the changes but also for the other contributors.

I’m still working with PHP. I had to deal with frameworks like WordPress for
example. And a lot of people are asking me how I test my applications: what 
testing framework do I use? How do I run my tests? Do I even have unit tests? 
The truth is that I don’t. And that’s because I don’t have units. The same goes 
for some JavaScript frameworks. It is difficult to test some parts of them 
because they do not have units. The developers should also think in this 
direction. Yes, they have to give us smart, elegant and working code. But that 
code should be testable, too.

## Documentation {#documentation}

I believe that without good documentation any project will die sooner or later
. We have so many frameworks and libraries coming out every week. Documentation 
is the first thing that the developers see. No one wants to spend hours 
searching about what the certain tool is doing or what its features are. Only 
listing of the main functionalities is not enough. Especially for a big 
framework.

I could split the successful documentation into three parts:

*   *What you can do* section — the documentation has to teach the users, and
    it should do it right. No matter how awesome and powerful our framework is, 
    there really has to be a proper explanation. Some people prefer watching video 
    clips, others — reading articles. In both cases, the vendor should lead the 
    developers from the very basic stuff to the advanced components of the framework.
   
*   *API documentation* — this is usually included. It’s a list of all the
    public methods of the framework, their parameters, what they return and maybe an
    example usage.
   
*   *How it works* section — usually this section is missing. It’s nice if
    someone explains the structure of the framework — even a simple schema of the 
    core functionalities and their relation will help. This will make the code 
    transparent. It will help the developers who want to make custom modifications.
   

It is, of course, difficult to predict the future. What we can do though is to
dream about it! It is important that we talk about what we expect and what we 
need from JavaScript frameworks! If you have any feedback or want to share your 
thoughts, tweet them using the[#jsframeworks][14] hashtag.

 [1]: http://emberjs.com/
 [2]: http://backbonejs.org/
 [3]: http://knockoutjs.com/
 [4]: img/repaint-1.gif
 [5]: img/repaint-2.gif
 [6]: img/repaint-3.gif
 [7]: https://facebook.github.io/react/
 [8]: http://requirejs.org/

 [9]: http://krasimirtsonev.com/blog/article/Dependency-injection-in-JavaScript#the-reflection-approach
 [10]: http://www.html5rocks.com/en/tutorials/es7/observe/
 [11]: http://yuilibrary.com/yui/configurator/
 [12]: http://en.wikipedia.org/wiki/Single_responsibility_principle
 [13]: https://travis-ci.org/
 [14]: https://twitter.com/hashtag/jsframeworks