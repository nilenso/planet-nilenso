---
title: "FlyMake does not like JRuby"
kind: article
created_at: 2011-02-11 18:54:00 UTC
author: Steven Deobald
layout: post
---
<div>JRuby often works out-of-the-box wherever MRI does. FlyMake is not one of those cases.</div><div><br /></div><div>Flymake expects a particular format for error messages returned from the interpreter: MRI error messages. <a href="http://www.ruby-forum.com/topic/211566">Charlie and Friends have decided the MRI error messages can be a bit opaque</a>, and upgraded them to something a little more readable. Sadly, this breaks anything attempting to parse those messages. </div><div><br /></div><div><a href="http://blog.gaz-jones.com/">Gaz</a> and I decided to try switching things back to MRI so we could get our syntax errors highlighted in emacs. Lo and behold! Everything's happy again. As an added bonus, you don't need to wait for JRuby (and thus, a JVM) to start up every time FlyMake runs.</div><div><br /></div><div>You can grab our changes with our latest emacs-starter-kit: https://github.com/drwti/emacs-starter-kit</div><div><br /></div><div>If you're using Phil Hagelberg's emacs-starter-kit, make the appropriate change to starter-kit-ruby.el:</div><div><br /></div><code><div><div><span class="Apple-style-span">-    (list "ruby" (list "-c" local-file))))</span></div><div><span class="Apple-style-span">+    (list "/usr/bin/ruby" (list "-c" local-file))))</span></div></div></code><div><br /></div><div>Enjoy!</div><div class="author">
  <img src="http://nilenso.com/people/steven-200.png" style="width: 96px; height: 96;">
  <span style="position: absolute; padding: 32px 15px;">
    <i>Original post by <a href="http://twitter.com/">Steven Deobald</a> - check out <a href="http://blog.deobald.ca/">Hungry, horny, sleepy, curious.</a></i>
  </span>
</div>
