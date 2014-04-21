Oh right.

A [stray interaction with Garrett Miller](https://twitter.com/heyitsgarrett/status/407634567621660672) about how he put a [colorful easter egg](http://consolemessage.com/post/68895207995/mapbox-via-drinks) in the developer console for [Mapbox](https://www.mapbox.com/) sent me on a journey of console self-discovery. This ultimately resulted in me putting way too much time into [isitchristmas.com](https://isitchristmas.com)'s developer console this year.

<img src="https://konklone.com/assets/images/blog/console/console-all-together.png" class="block border" />

This makes use of four tricks: applying **basic text styling** to `console.log` output, **hacking in sprites** through background-images, executing code **without using parentheses**, and **suppressing return values**. Yes.

### CSS in the console

This isn't really a trick, but I don't know that enough people are aware that `console.log` output can be styled with CSS using a **%c** flag, [in Chrome](https://developers.google.com/chrome-developer-tools/docs/console#styling_console_output_with_css) and [in Firebug](http://getfirebug.com/wiki/index.php/Console.log). It looks like this:

<img src="https://konklone.com/assets/images/blog/console/console-styling-example.png" class="block border" />

Sadly, Firefox's more recent built-in console doesn't support this, and nor does Internet Explorer. I'm told Safari does. Fortunately, Chrome is by far [the most used browser](https://github.com/isitchristmas/data/blob/gh-pages/2012/browsers.csv) for my site, so I'm fine with this. I have [a simple console.log wrapper](https://github.com/isitchristmas/web/blob/e332f791f83853e161019ac6098f73becdd8235e/views/index.html#L1056-L1074) that "downgrades" the console to plain-text in Internet Explorer.

### Creating sprites

While you can't use `%c` for much past basic text styling, you **can** apply a background image to a spaces, and achieve some crude sprites. You can't set a width — just pick the best number of spaces — but with a `background-position` of `cover`, it should roughly do what you want.

<img src="https://konklone.com/assets/images/blog/console/console-images-1.png" class="block border" />

I use this on isitchristmas.com to display flags representing where people are from:

<img src="https://konklone.com/assets/images/blog/console/console-flags.png" class="block border" />

And to add full support for [emoji](http://www.emoji-cheat-sheet.com/) in chat messages:

<img src="https://konklone.com/assets/images/blog/console/console-emoji.png" class="block border" />

The number of spaces you want will depend on how high your text is. At the default text size, I found 2 spaces to work perfectly for emoji, and 3 spaces to work well enough for flags on isitchristmas (though some wider flags get cut off).

In practice, using inline CSS in JavaScript is annoying. I try to make it less so be keeping a table of styles and style generators handy, like this:

```javascript
var styles = {
  spam: "color: #336699",
  please: "color: #336699; font-weight: bold",
  emoji: function(emoji) {
    return "background-image: url(\"https://isitchristmas.com/emojis/" + emoji + ".png\"); background-size: cover";
  }
};
```

Which can be used like this:

<pre><code class="block">var msg = "%cPlease%c don't spam and ruin the chat! %c  %c %c  %c %c  ";
log.system(
  msg, styles.please, styles.spam, styles.emoji("smiley"),
  styles.spam, styles.emoji("heart"),
  styles.spam, styles.emoji("christmas_tree")
);</code></pre>

Which produces:

<img src="https://konklone.com/assets/images/blog/console/console-spam-only.png" class="block" />

### Running code without parentheses

If you define a function and run it without its parentheses, it will just print the function out:

<img src="https://konklone.com/assets/images/blog/console/console-undefined-6.png" class="block border" />

And of course, if you execute a bare word that's not been defined, the console will throw a ReferenceError. (Sadly, you can't catch and repurpose errors thrown in the console itself with [window.onerror](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers.onerror).)

However, the console always tries to print out the return value of the line you execute. You can take advantage of this by making your variable a no-op function, and then overriding that no-op's `toString` with a function that does whatever you want.

<img src="https://konklone.com/assets/images/blog/console/console-undefined-5.png" class="block border" />

I use a function as the base to call `toString` on, because as it turns out, the console will ignore attempts to override `toString` on objects and strings. And of course, this isn't documented behavior at all — it could change any time!

### Suppressing return values

In the example above, the console spat out "undefined" after running "help". That's because `console.log` doesn't return anything, and the console always tries to print the return value.

You can cause the console to print blankness instead, by returning a single space at the end of your `toString` override.

<img src="https://konklone.com/assets/images/blog/console/console-undefined-4.png" class="block border" />

Note that this only works inside of a toString override! So, if you're running a normal function, with parentheses, and you want to suppress output, you need to do something like this:

<img src="https://konklone.com/assets/images/blog/console/console-undefined-3.png" class="block border" />

More useful is putting these tricks together. Since a chat room means writing a message and passing it as an argument, I've been asking users to use proper functions — e.g. `say("Hi!")` — but that means people have to type parentheses and quotes all the time, which can be annoying and confusing.

I can use the `toString` trick to provide two methods of chatting - one with parentheses, and one without.

This is what the code looks like:

<pre><code class="block">var say = function(message) {
  // ... code to send message ...

  return blank();
};

say.toString = function() {
  say(prompt("What do you want to say?"));

  return " ";
};</code></pre>

And I document it like this:

<img src="https://konklone.com/assets/images/blog/console/console-documenting.png" class="block border" />

### There is no good reason to do this

The developer console is absolutely not meant for this stuff, and it is all a completely silly endeavor. But it was fun to figure out how to do, and I think the Internet is better with more easter eggs in it (and the folks at [Console Message](http://consolemessage.com/) obviously agree).

Maybe there's a useful library to extract from all this, maybe not. Regardless, I hope this crucial and painstakingly crafted engineering advice is helpful in your professional endeavors!

<a href="https://konklone.com/assets/images/blog/console/console-closing.png" target="_blank">
  <img src="https://konklone.com/assets/images/blog/console/console-closing-small.png" class="block border" />
</a>