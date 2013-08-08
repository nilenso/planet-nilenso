---
title: "Using bundler with executables"
kind: article
created_at: 2012-06-30 07:31:00 UTC
author: Tejas Dinkar
layout: post
---
<h2>What Is Bundler?</h2><p>Bundler is a ruby gem loading system. It loads up all the relevant gems for your project, and ensures the versions are correct, before your app takes over.</p><h2>How do I ensure that my executables are bundler aware?</h2><p>When doing a bundle install (or just bundle), always add the option --binstubs. This will create binary wrappers in the bin/ folder of your project. <code>bundle --binstubs</code></p><p>Instead of executing commands directly like this:   <code>rake</code></p><p>Execute the command command that is in the bin folder.   <code>./bin/rake</code></p><h2>Do I check these /bin/* files in?</h2><p>I usually git ignore them, but, according to the documentation, it&#39;s safe to check in.</p><h2>I hate typing ./bin/rake every time!</h2><p>No problem. You can add a relative path like ./bin to your $PATH</p><ul><li>If you are using rbenv, just add ./bin to your path in your ~/.bashrc</li><li>If you are using rvm, add ./bin into every .rvmrc</li></ul><div class="author">
  <img src="http://nilenso.com/people/tejas-200.png" style="width: 96px; height: 96;">
  <span style="position: absolute; padding: 32px 15px;">
    <i>Original post by <a href="http://twitter.com/">Tejas Dinkar</a> - check out <a href="http://blog.gja.in/">Advice about nothing</a></i>
  </span>
</div>
