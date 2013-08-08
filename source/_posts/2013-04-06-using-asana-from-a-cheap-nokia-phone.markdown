---
title: "Using Asana From a Cheap Nokia Phone"
kind: article
created_at: 2013-04-06 17:59:00 UTC
author: Timothy Andrew
layout: post
---
<p>I love <a href="http://asana.com/">Asana</a>. Its made me a lot more organized than I used to be.</p>

<p>I&#8217;d love to be able to check my tasks when I&#8217;m not at a computer, but my phone looks like this.</p>

<p><img src="http://blog.timothyandrew.net/images/c2-01.jpg" alt="Nokia C2-01" /></p>

<p>There&#8217;s no way the Asana app will load on its Opera Mini browser.</p>

<p>It does have email, though. And Asana lets me <a href="http://asana.com/guide/tags-email/email-basics">send tasks</a> in via email.</p>

<p>Unfortunately, there seems to be no way to actually <em>view</em> all your tasks from a phone like this.</p>

<p><strong>Until now.</strong></p>

<p>I wrote a small <a href="http://sinatrarb.com/">Sinatra</a> app that pulls my tasks from Asana and displays them on a simple HTML page. It&#8217;s very rough around the edges, but works well enough for my needs. You can grab the code and set it up <a href="http://github.com/timothyandrew/asana-light">here</a>.</p>

<p>There is <strong><em>no</em> authentication</strong> yet, so anyone with a link can view your tasks.</p>

<p>Read on for an explanation of how it works.</p>

<!-- more -->


<p>Luckily for us, there&#8217;s a ruby interface to Asana&#8217;s REST API in the form of the <a href="http://github.com/rbright/asana">asana gem</a>, which we can use to pull all our tasks from Asana.</p>

<p>First, we need to get the API key for our Asana account for authentication.</p>

<ul>
<li>Login to Asana</li>
<li>Visit this link: <a href="http://app.asana.com/-/account_api">http://app.asana.com/-/account_api</a></li>
<li>Copy the API key.</li>
</ul>


<p>We use this to set up the gem.</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
</pre></td><td class='code'><pre><code class='ruby'><span class='line'><span class="n">configure</span> <span class="k">do</span>
</span><span class='line'>  <span class="no">Asana</span><span class="o">.</span><span class="n">configure</span> <span class="k">do</span> <span class="o">|</span><span class="n">client</span><span class="o">|</span>
</span><span class='line'>    <span class="n">client</span><span class="o">.</span><span class="n">api_key</span> <span class="o">=</span> <span class="s2">&quot;YOUR_API_KEY&quot;</span>
</span><span class='line'>  <span class="k">end</span>
</span><span class='line'><span class="k">end</span>
</span></code></pre></td></tr></table></div></figure>


<p>Next, we need to get all your tasks. We get all the tasks assigned to you from each of your workspaces and put them all in one array.</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
</pre></td><td class='code'><pre><code class='ruby'><span class='line'><span class="n">user</span> <span class="o">=</span> <span class="ss">Asana</span><span class="p">:</span><span class="ss">:User</span><span class="o">.</span><span class="n">me</span>
</span><span class='line'><span class="n">workspaces</span> <span class="o">=</span> <span class="ss">Asana</span><span class="p">:</span><span class="ss">:Workspace</span><span class="o">.</span><span class="n">all</span>
</span><span class='line'><span class="n">tasks</span> <span class="o">=</span> <span class="n">workspaces</span><span class="o">.</span><span class="n">reduce</span><span class="p">(</span><span class="o">[]</span><span class="p">)</span> <span class="k">do</span> <span class="o">|</span><span class="n">memo</span><span class="p">,</span> <span class="n">workspace</span><span class="o">|</span>
</span><span class='line'>    <span class="n">memo</span> <span class="o">&lt;&lt;</span> <span class="n">workspace</span><span class="o">.</span><span class="n">tasks</span><span class="p">(</span><span class="n">user</span><span class="o">.</span><span class="n">id</span><span class="p">)</span><span class="o">.</span><span class="n">map</span><span class="p">(</span><span class="o">&amp;</span><span class="ss">:name</span><span class="p">)</span>
</span><span class='line'>    <span class="n">memo</span>
</span><span class='line'><span class="k">end</span><span class="o">.</span><span class="n">flatten</span>
</span></code></pre></td></tr></table></div></figure>


<p>We send all this to this ERB template:</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
<span class='line-number'>7</span>
</pre></td><td class='code'><pre><code class='erb'><span class='line'><span class="x">&lt;html&gt;&lt;body&gt;</span>
</span><span class='line'><span class="x">  &lt;ul&gt;</span>
</span><span class='line'><span class="x">  </span><span class="cp">&lt;%</span> <span class="n">tasks</span><span class="o">.</span><span class="n">each</span> <span class="k">do</span> <span class="o">|</span><span class="n">task</span><span class="o">|</span> <span class="cp">%&gt;</span><span class="x"></span>
</span><span class='line'><span class="x">      &lt;li&gt;</span><span class="cp">&lt;%=</span> <span class="n">task</span> <span class="cp">%&gt;</span><span class="x">&lt;/li&gt;</span>
</span><span class='line'><span class="x">  </span><span class="cp">&lt;%</span> <span class="k">end</span> <span class="cp">%&gt;</span><span class="x"></span>
</span><span class='line'><span class="x">  &lt;/ul&gt;</span>
</span><span class='line'><span class="x">&lt;/body&gt;&lt;/html&gt;</span>
</span></code></pre></td></tr></table></div></figure>


<p>Which renders the tasks in a format any browser can comfortably read.<br/>
And that&#8217;s pretty much all the code that&#8217;s necessary.</p>

<hr />

<p>There are a lot of ways this can be improved, I&#8217;m sure. Off the top of my head:</p>

<ul>
<li>Group tasks by workspace</li>
<li>Sort tasks by date</li>
<li>Add authentication. Anyone who visits the URL shouldn&#8217;t be able to see all your tasks.</li>
</ul><div class="author">
  <img src="http://nilenso.com/people/timothy-200.jpg" style="width: 96px; height: 96;">
  <span style="position: absolute; padding: 32px 15px;">
    <i>Original post by <a href="http://twitter.com/">Timothy Andrew</a> - check out <a href="http://blog.timothyandrew.net/">Timothy's Blog</a></i>
  </span>
</div>
