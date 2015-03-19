# Trello CSS Guide

â€œI perfectly understand our CSS. I never have any issues with cascading rules. I never have to use `!important` or inline styles. Even though somebody else wrote this bit of CSS, I know exactly how it works and how to extend it. Fixes are easy! I have a hard time breaking our CSS. I know exactly where to put new CSS. We use all of our CSS and itâ€™s pretty small overall. When I delete a template, I know the exact corresponding CSS file and I can delete it all at once. Nothing gets left behind.â€

You often hear updog saying stuff like this. Whoâ€™s updog? Not much, who is up with you?

This is where any fun you might have been having ends. Now itâ€™s time to get serious and talk about rules.

Writing CSS is hard. Even if you know all the intricacies of position and float and overflow and z-index, itâ€™s easy to end up with spaghetti code where you need inline styles, !important rules, unused cruft, and general confusion. This guide provides some architecture for writing CSS so it stays clean and maintainable for generations to come.

There are eight _fascinating_ parts.

1. [Tools](#1-tools)
2. [Components](#2-components)
    - [Modifiers](#modifiers)
    - [State](#state)
    - [Media Queries](#media-queries)
    - [Keeping It Encapsulated](#keeping-it-encapsulated)
3. [JavaScript](#3-javascript)
4. [Mixins](#4-mixins)
5. [Utilities](#5-utilities)
6. [File Structure](#6-file-structure)
7. [Style](#7-style)
8. [Miscellany](#8-miscellany)
    - [Performance](#performance)


## 1. Tools

> Use only imports, variables, and mixins (and only for vender-prefixed features) from CSS preprocessors.

To keep our CSS readable, we try and keep our CSS very vanilla. We use LESS, but only use imports, data-uri, variables, and some mixins (only for vender-prefixed stuff). We use imports so that variables and mixins are available everywhere and it all outputs to a single file. We occasionally use nesting, but only for very shallow things like `&:hover`. We donâ€™t use more complex functions like guards and loops.

If you follow the rest of the guide, you shouldnâ€™t need the complex functions in preprocessors. Have I piqued your interest? Read onâ€¦


## 2. Components

> Use the `.component-descendant-descendant` pattern for components.

Components help encapsulate your CSS and prevent run-away cascading styles and keep things readable and maintainable. Central to componentizing CSS is namespacing. Instead of using descendant selectors, like `.header img { â€¦ }`, youâ€™ll create a new hyphen-separated class for the descendant element, like `.header-image { â€¦ }`.

Hereâ€™s an example with descendant selectors:

``` LESS
.global-header {
  background: hsl(202, 70%, 90%);
  color: hsl(202, 0%, 100%);
  height: 40px;
  padding: 10px;
}

  .global-header .logo {
    float: left;
  }

    .global-header .logo img {
      height: 40px;
      width: 200px;
    }

  .global-header .nav {
    float: right;
  }

    .global-header .nav .item {
      background: hsl(0, 0%, 90%);
      border-radius: 3px;
      display: block;
      float: left; 
      -webkit-transition: background 100ms;
      transition: background 100ms;
    }

    .global-header .nav .item:hover {
      background: hsl(0, 0%, 80%);
    }
```

And hereâ€™s the same example with namespacing:

``` LESS
.global-header {
  background: hsl(202, 70%, 90%);
  color: hsl(202, 0%, 100%);
  height: 40px;
  padding: 10px;
}

  .global-header-logo {
    float: left;
  }

    .global-header-logo-image {
      background: url("logo.png");
      height: 40px;
      width: 200px;
    }

  .global-header-nav {
    float: right;
  }

    .global-header-nav-item {
      background: hsl(0, 0%, 90%);
      border-radius: 3px;
      display: block;
      float: left; 
      -webkit-transition: background 100ms;
      transition: background 100ms;
    }

    .global-header-nav-item:hover {
      background: hsl(0, 0%, 80%);
    }
```

Namespacing keeps specificity low, which leads to fewer inline styles, !important declarations, and makes things more maintainable over time.

Make sure **every selector is a class**. There should be no reason to use id or element selectors. No underscores or camelCase. Everything should be lowercase.

Components make it easy to see relationships between classes. You just need to look at the name. You should still **indent descendant classes** so their relationship is even more obvious and itâ€™s easier to scan the file. Stateful things like `:hover` should be on the same level.


### Modifiers

> Use the `.component-descendant.mod-modifier` pattern for modifier classes.

Letâ€™s say you want to use a component, but style it in a special way. We run into a problem with namespacing because the class needs to be a sibling, not a child. Naming the selector `.component-descendant-modifier` means the modifier could be easily confused for a descendant. To denote that a class is a modifier, use a `.mod-modifier` class.

For example, we want to specially style our sign up button among the header buttons. Weâ€™ll add `.global-header-nav-item.mod-sign-up`, which looks like this:

``` HTML
<!-- HTML -->

<a class="global-header-nav-item mod-sign-up">
  Sign Up
</a>
```

``` LESS
// global-header.less

.global-header-nav-item {
  background: hsl(0, 0%, 90%);
  border-radius: 3px;
  display: block;
  float: left; 
  -webkit-transition: background 100ms;
  transition: background 100ms;
}

.global-header-nav-item.mod-sign-up {
  background: hsl(120, 70%, 40%);
  color: #fff;
}
```

We inherit all the `global-header-nav-item` styles and modify it with `.mod-sign-up`. This breaks our namespace convention and increases the specificity, but thatâ€™s exactly what we want. This means we donâ€™t have to worry about the order in the file. For the sake of clarity, put it after the part of the component it modifies. Put modifiers on the same indention level as the selector itâ€™s modifying.

**You should never write a bare `.mod-` class**. It should always be tied to a part of a component. `.header-button.mod-sign-up { background: green; }` is good, but `.mod-sign-up { background: green; }` is bad. We could be using `.mod-sign-up` in another component and we wouldnâ€™t want to override it.

Youâ€™ll often want to overwrite a descendant of the modified selector. Do that like so:

``` LESS
.global-header-nav-item.mod-sign-up {
  background: hsl(120, 70%, 40%);
  color: #fff;

  .global-header-nav-item-text {
    font-weight: bold;
  }

}
```

Generally, we try and avoid nesting because it results in runaway rules that are impossible to read. This is an exception.

Put modifiers at the bottom of the component file, after the original components.


### State

> Use the `.component-descendant.is-state` pattern for state. Manipulate `.is-` classes in JavaScript (but not presentation classes).

State classes show that something is enabled, expanded, hidden, or what have you. For these classes, weâ€™ll use a new `.component-descendant.is-state` pattern.

Example: Letâ€™s say that when you click the logo, it goes back to your home page. But because itâ€™s a single page app, it needs to load things. You want your logo to do a loading animation. This should sound familiar to Trello users.

Youâ€™ll use a `.global-header-logo-image.is-loading` rule. That looks like this:

``` LESS
.global-header-logo-image {
  background: url("logo.png");
  height: 40px;
  width: 200px;
}

.global-header-logo-image.is-loading {
  background: url("logo-loading.gif");
}
```

JavaScript defines the state of the application, so weâ€™ll use JavaScript to toggle the state classes. The `.component.is-state` pattern decouples state and presentation concerns so we can add state classes without needing to know about the presentation class. A developer can just say to the designer, â€œThis element has an .is-loading class. You can style it however you want.â€. If the state class were something like `global-header-logo-image--is-loading`, the developer would have to know a lot about the presentation and it would be harder to update in the future.

Like modifiers, itâ€™s possible that the same state class will be used on different components. You donâ€™t want to override or inherit styles, so itâ€™s important that **every component define its own styles for the state**. They should never be defined on their own. Meaning you should see `.global-header.is-hidden { display: none; }`, but never `.is-hidden { display: none; }` (as tempting as that may be). `.is-hidden` could conceivably mean different things in different components.

We also donâ€™t indent state classes. Again, thatâ€™s only for descendants. State classes should appear at the bottom of the file, after the original components and modifiers.


### Media Queries

> Use media query variables in your component.

It might be tempting to add something like a `mobile.less` file that contains all your mobile-specific rules. We want to avoid global media queries and instead include them inside our components. This way when we update or delete a component, weâ€™ll be less likely to forget about the media rules.

Rather than writing out the media queries every time, weâ€™ll use a media-queries.less file with media query variables. It should look something like this:

``` LESS
@highdensity:  ~"only screen and (-webkit-min-device-pixel-ratio: 1.5)",
               ~"only screen and (min--moz-device-pixel-ratio: 1.5)",
               ~"only screen and (-o-min-device-pixel-ratio: 3/2)",
               ~"only screen and (min-device-pixel-ratio: 1.5)";

@small:        ~"only screen and (max-width: 750px)";
@medium:       ~"only screen and (min-width: 751px) and (max-width: 900px)";
@large:        ~"only screen and (min-width: 901px) and (max-width: 1280px)";
@extra-large:  ~"only screen and (min-width: 1281px)";

@print:        ~"print";
```

To use a media query:

``` LESS
// Input
@media @large { 
  .component-nav { â€¦ }
}

/* Output */
@media only screen and (min-width: 901px) and (max-width: 1280px) {
  .component-nav { â€¦ }
}
```

You can use commas to include multiple variables, like `@media @small, @medium { â€¦ }`.

This means weâ€™re using the same breakpoints throughout and you donâ€™t have to write the same media query over and over. Repeated phrases like media queries are easily compressed so you donâ€™t need to worry about CSS size getting too big. This practice was taken from [this CodePen from Eric Rasch](http://codepen.io/ericrasch/pen/HzoEx).

Note that print is a media attribute, too. Keep your print rules inside components. We donâ€™t want to forget about them either.

Put media rules at the bottom of the component file.


## Keeping It Encapsulated

Components can control a large part of the layout or just a button. In your templates, youâ€™ll likely end up with parts of one component inside another component, like a `.button` inside a `.member-list`. We need to change the buttonâ€™s size and positioning to fit the list. 

This is tricky. Components shouldnâ€™t know anything about each other. If the smaller button can be reused in multiple places, add a modifier in the button component (like, `.button.mod-small`) and use it in member-list. Do the positioning with a member list component with a descendant, since thatâ€™s specific to the member list and not the button.

Hereâ€™s an example:

``` HTML
<!-- HTML --> 

<div class="member-list">
  <div class="member-list-item">
    <p class="member-list-item-name">Gumby</p>
    <div class="member-list-item-action">
      <a href="#" class="button mod-small">Add</a>
    </div>
  </div>
</div>
```

``` LESS
// button.less

.button {
  background: #fff;
  border: 1ps solid #999;
  padding: 8px 12px;
}

.button.mod-small {
  padding: 6px 10px;
}


// member-list.less

.member-list {
  padding: 20px;
}

  .member-list-item {
    margin: 10px 0;
  }

    .member-list-item-name {
      font-weight: bold;
      margin: 0;
    }

    .member-list-item-action {
      float: right;
    }
```

A _bad_ thing to do would be this:

``` HTML
<!-- HTML -->

<div class="member-list">
  <div class="member-list-item">
    <p class="member-list-item-name">Pat</p>
    <a href="#" class="member-list-item-button button">Add</a>
  </div>
</div>
```

``` LESS
// member-list.less

.member-list-item-button {
  float: right;
  padding: 6px 10px;
}
```

In the _bad_ example, `.member-list-item-button` overrides styles specific to the button component. It assumes things about button that it shouldnâ€™t have to know anything about. It also prevents us from reusing the small button style and makes it hard to clean or change up later if needed.

You should end up with a lot of components. Always be asking yourself if everything inside a component is absolutely related and canâ€™t be broken down into more components. If you start to have a lot of modifiers and descendants, it might be time to break it up.


## 3. JavaScript

> Separate style and behavior concerns by using `.js-` prefixed classes for behavior. 

For example:

``` HTML
<!-- HTML -->

<div class="content-nav">
  <a href="#" class="content-nav-button js-open-content-menu">
    Menu
  </a>
</div>
```

``` JavaScript
// JavaScript (with jQuery)

$(".js-open-content-menu").on("click", function(e){
  openMenu();
});
```

Why do we want to do this? The `.js-` class makes it clear to the next person changing this template that it is being used for some JavaScript event and should be approached with caution.

Be sure to **use a descriptive class name**. The intent of `.js-open-content-menu` is more clear than `.js-menu`. A more descriptive class is less likely to conflict with other classes and itâ€™s lots easier to search for. The class should almost always include a verb since itâ€™s tied to an action. 

**`.js-` classes should never appear in your stylesheets**. They are for JavaScript only. Inversely, there is never a reason to see presentation classes like `.header-nav-button` in JavaScript. You will see state classes like `.is-state` in your JavaScript and your stylesheets as `.component.is-state`.


## 4. Mixins

> Prefix mixins with `.m-` and only use them sparingly for shared styles.

Mixins are shared styles that are used in more than one component. Mixins should not be standalone classes or used in markup. They should be single level and contain no nesting. Mixins make things complicated fast, so **use sparingly**.

Previously, we used mixins for browser prefixed features, but we use [autoprefixer](https://www.npmjs.com/package/autoprefixer) for that now.

When using a mixin, it should include the parenthesis to make it more obvious that itâ€™s a mixin. Example usage:

``` LESS
// mixins.less
.m-list-divider () {
  border-bottom: 1px solid @light-gray-300;
}

// component.less
.component-descendent {
  .m-list-divider();
}
```


## 5. Utilities

> Prefix utility classes with `.u-`.

Sometimes we need a universal class that can be used in any component. Things like clear fixes, vertical alignment, and text truncation. Denote these classes by prefixing them with `.u-`. For example:

``` LESS
.u-truncate-text {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

All the utils should be in a single file. There shouldnâ€™t be any need to overwrite them in components or mixins.

You should really only need a few utilities. We donâ€™t need something like `.u-float-left { float: left; }` where including `float: left;` in the component is just as easy and more visible.


## 6. File Structure

The file will look something like this:

``` LESS
@charset "UTF-8"

@import "normalize.css"

// Variables
@import "media-queries.less"
@import "colors.less"
@import "other-variables-like-fonts.less"

// Mixins
@import "mixins.less"

// Utils
@import "utils.less"

// Components
@import "component-1.less"
@import "component-2.less"
@import "component-3.less"
@import "component-4.less" // and so forth
```

Include [normalize.css](http://necolas.github.io/normalize.css/) at the top of the file. It standardizes CSS defaults across browsers. You should use it in all projects. Then include variables, mixins, and utils (respectively).

Then include the components. Each component should have its own file and include all the necessary modifiers, states, and media queries. If we write components correctly, the order should not matter. 

This should output a single `app.css` file (or something similarly named).


## 7. Style

Even following the above guidelines, itâ€™s still possible to write CSS in a ton of different ways. Writing our CSS in a consistent way makes it more readable for everyone. Take this bit of CSS:

``` LESS
.global-header-nav-item {
  background: hsl(0, 0%, 90%);
  border-radius: 3px;
  display: block;
  float: left; 
  padding: 8px 12px;
  -webkit-transition: background 100ms;
  transition: background 100ms;
}
```

It sticks to these style rules:

-	Use a new line for every selector and every declaration.
- Use two new lines between rules.
-	Add a single space between the property and value, for example `prop: value;` and not `prop:value;`.
-	Alphabetize declarations.
-	Use 2 spaces to indent, not 4 spaces and not tabs.
-	No underscores or camelCase for selectors.
- Use shorthand when appropriate, like `padding: 15px 0;` and not `padding: 15px 0px 15px 0px;`.
-	When using vendor prefixed features, put the standard declaration last. For example: `-webkit-transition: all 100ms; transition: all 100ms;`. (Note: Browsers will optimize the standard declaration, but continue to keep the old one around for compatibility. Putting the standard declaration after the vendor one means it will get used and you get the most optimized version.)
-	Prefer hsl(a) over hex and rgb(a). Working with colors in code is easier with hsl, especially when making things lighter or darker, since you only have one variable to adjust.


## 8. Miscellany

You might get the impression from this guide that our CSS is in great shape. That is not the case. While weâ€™ve always stuck to .js classes and often use namespaced-component-looking classes, there is a mishmash of styles and patterns throughout. Thatâ€™s okay. Going forward, you should rewrite sections according to these rules. Leave the place nicer than you found it.

Some additional things to keep in mind:

- Comments rarely hurt. If you find an answer on Stack Overflow or in a blog post, add the link to a comment so future people know whatâ€™s up. Itâ€™s good to explain the purpose of the file in a comment at the top.
- In your markup, order classes like so `<div class="component mod util state js"></div>`.
- You can embed common images and files under 10kb using datauris. In the Trello web client, you can use `embed(/path/to/file)` to do this. This saves a request, but adds to the CSS size, so only use it on extremely common things like the logo.
- Avoid body classes. There is rarely a need for them. Stick to modifiers within your component.
- Explicitly write out class names in selectors. Donâ€™t concatenate strings or use preprocessor trickery to build a class name. We want to be able to search for class names and that makes it impossible. This goes for `.js-` classes in JavaScript, too.
- If you are worried about long selector names making our CSS huge, donâ€™t be. Compression makes this a moot point.

Some additional reading on CSS architecture around the web:

- [Mediumâ€™s CSS guidelines.](https://gist.github.com/fat/a47b882eb5f84293c4ed) I ~~stole~~ learned a lot from this.
- [â€œCSS Atâ€¦â€ from CSS Tricks](http://css-tricks.com/css/). This is a big list of CSS practices at various companies.
- The BEM, or â€œblock, element, modifierâ€, methodology is similar to our components. It is well explained in [this CSS Wizardry article](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/).


### Performance

Performance probably deserves itâ€™s own guide, but Iâ€™ll talk about two big concepts: selector performance and layouts/paints.

Selector performance seems to matters less and less these days, but can be a problem in a complex, single-page app with thousands of DOM elements (like Trello). [The CSS Tricks article about selector performance](http://css-tricks.com/efficiently-rendering-css/) should help explain the important concept of the key selector. Seemingly specific rules like `.component-descendant-descendant div` are actual quite expensive in complex apps because rules are read _from right to left_. It needs to look up all the divs first (which could be thousands) then go up the DOM from there. 

[Juriy Zaytsevâ€™s post on CSS profiling](http://perfectionkills.com/profiling-css-for-fun-and-profit-optimization-notes/) profiles browsers on selector matching, layouts, paints, and parsing times of a complex app. It confirms the theory that highly specific selectors are bad for big apps. Harry Roberts of CSS Wizardry also wrote about [CSS selector performance](http://csswizardry.com/2011/09/writing-efficient-css-selectors/).

You shouldnâ€™t have to worry about selector performance if you use components correctly. It will keep specificity about as low as it gets.

Layouts and paints can cause lots of performance damage. Be cautious with CSS3 features like text-shadow, box-shadow, border-radius, and animations, [especially when used together](http://www.html5rocks.com/en/tutorials/speed/css-paint-times/). We wrote a big blog post about performance [back in January 2014](http://blog.fogcreek.com/we-spent-a-week-making-trello-boards-load-extremely-fast-heres-how-we-did-it/). Much of this was due to layout thrashing caused by JavaScript, but we cut out some heavy styles like borders, gradients, and shadows, which helped a lot.

