---
title: "Recursive Postgres Queries"
kind: article
created_at: 2013-06-24 03:00:00 UTC
author: Timothy Andrew
layout: post
---
<h3>Introduction</h3>

<p>At <a href="http://nilenso.com/">Nilenso</a>, I&#8217;m working on an (<a href="http://github.com/nilenso/ashoka-survey-web">open-source!</a>) app to design and conduct surveys.</p>

<p>Here&#8217;s a simple survey being designed:</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/survey-overview.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/survey-overview.png" alt="Survey Builder Overview" /></a></p>

<p>Internally, this is represented as:</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/internal-survey-overview.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/internal-survey-overview.png" alt="Survey Model Overview" /></a></p>

<p>A survey is made up of many <em>questions</em>. A number of questions can (optionally) be grouped together in a <em>category</em>. Our actual data model is a bit more complicated than this (sub-questions, especially), but let&#8217;s assume that we&#8217;re just working with questions and categories.</p>

<p>Here&#8217;s how we preserve the ordering of questions and categories in this survey.</p>

<p>Each question and category has an <code>order_number</code> field. It&#8217;s an integer that specifies the relative ordering of itself and its siblings.</p>

<p>For example, in this case,</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/order-number-overview.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/order-number-overview.png" alt="Order Number Overview" /></a></p>

<p>the question <code>Bar</code> will have an <code>order_number</code> that is <em>less than</em> the order number of <code>Baz</code>.</p>

<p>This guarantees that the a category can fetch its sub-questions in the right order:</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
</pre></td><td class='code'><pre><code class='ruby'><span class='line'><span class="c1"># In category.rb</span>
</span><span class='line'>
</span><span class='line'><span class="k">def</span> <span class="nf">sub_questions_in_order</span>
</span><span class='line'>  <span class="n">questions</span><span class="o">.</span><span class="n">order</span><span class="p">(</span><span class="s1">&#39;order_number&#39;</span><span class="p">)</span>
</span><span class='line'><span class="k">end</span>
</span></code></pre></td></tr></table></div></figure>


<p>In fact, this is how we fetch the entire survey at this point. Each category calls <code>sub_questions_in_order</code> on each of its children, and so on down the entire tree.</p>

<p>This gives us a depth-first ordering of this tree:</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/naive-order.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/naive-order.png" alt="Naive Ordering" /></a></p>

<p>For large surveys with 5+ levels of nesting, and more than a hundred questions, this is pretty slow.</p>

<!-- more -->


<h3>Recursive Queries</h3>

<p>I&#8217;ve used gems like <a href="https://github.com/collectiveidea/awesome_nested_set"><code>awesome_nested_set</code></a> before, but as far as I could find, none of them supported fetching results across multiple models.</p>

<p>Then I stumbled on <a href="http://www.postgresql.org/docs/9.1/static/queries-with.html">a page</a> describing PostgreSQL&#8217;s support for recursive queries! That seemed perfect.</p>

<p>Let&#8217;s solve this particular problem using recursive queries. (My understanding of this is still very basic, so please don&#8217;t take my word for any of this)</p>

<p>To define a recursive Postgres query, we need to define an initial query, which is called the non-recursive term.</p>

<p>In our case, that would be the top level questions and categories.
Top level elements don&#8217;t have a parent category, so their <code>category_id</code> is <code>NULL</code>.</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
<span class='line-number'>7</span>
<span class='line-number'>8</span>
<span class='line-number'>9</span>
</pre></td><td class='code'><pre><code class='sql'><span class='line'><span class="p">(</span>
</span><span class='line'>  <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">order_number</span><span class="p">,</span> <span class="k">type</span><span class="p">,</span> <span class="n">category_id</span> <span class="k">FROM</span> <span class="n">questions</span>
</span><span class='line'>  <span class="k">WHERE</span> <span class="n">questions</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">questions</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'><span class="p">)</span>
</span><span class='line'><span class="k">UNION</span>
</span><span class='line'><span class="p">(</span>
</span><span class='line'>  <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">order_number</span><span class="p">,</span> <span class="k">type</span><span class="p">,</span> <span class="n">category_id</span> <span class="k">FROM</span> <span class="n">categories</span>
</span><span class='line'>  <span class="k">WHERE</span> <span class="n">categories</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">categories</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'><span class="p">)</span>
</span></code></pre></td></tr></table></div></figure>


<p>(That query, and all subsequent ones, assume that the survey we&#8217;re querying has id = 2)</p>

<p>This gives us the top-level elements.</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/top-level-elements-query.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/top-level-elements-query.png" alt="Top-Level Queries" /></a></p>

<p>Now on to the recursive term. According to the Postgres docs:</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/steps.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/steps.png" alt="Postgres Steps" /></a></p>

<p>Our recursive term simply has to find all the direct children of the elements fetched by the non-recursive term.</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
<span class='line-number'>7</span>
<span class='line-number'>8</span>
<span class='line-number'>9</span>
<span class='line-number'>10</span>
<span class='line-number'>11</span>
<span class='line-number'>12</span>
<span class='line-number'>13</span>
<span class='line-number'>14</span>
<span class='line-number'>15</span>
<span class='line-number'>16</span>
<span class='line-number'>17</span>
<span class='line-number'>18</span>
</pre></td><td class='code'><pre><code class='sql'><span class='line'><span class="k">WITH</span> <span class="k">RECURSIVE</span> <span class="n">first_level_elements</span> <span class="k">AS</span> <span class="p">(</span>
</span><span class='line'>  <span class="c1">-- Non-recursive term</span>
</span><span class='line'>  <span class="p">(</span>
</span><span class='line'>    <span class="p">(</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">order_number</span><span class="p">,</span> <span class="n">category_id</span> <span class="k">FROM</span> <span class="n">questions</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">questions</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">questions</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'>    <span class="k">UNION</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">order_number</span><span class="p">,</span> <span class="n">category_id</span> <span class="k">FROM</span> <span class="n">categories</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">categories</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">categories</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'>    <span class="p">)</span>
</span><span class='line'>  <span class="p">)</span>
</span><span class='line'>  <span class="k">UNION</span>
</span><span class='line'>  <span class="c1">-- Recursive Term</span>
</span><span class='line'>  <span class="k">SELECT</span> <span class="n">q</span><span class="p">.</span><span class="n">id</span><span class="p">,</span> <span class="n">q</span><span class="p">.</span><span class="n">content</span><span class="p">,</span> <span class="n">q</span><span class="p">.</span><span class="n">order_number</span><span class="p">,</span> <span class="n">q</span><span class="p">.</span><span class="n">category_id</span>
</span><span class='line'>  <span class="k">FROM</span> <span class="n">first_level_elements</span> <span class="n">fle</span><span class="p">,</span> <span class="n">questions</span> <span class="n">q</span>
</span><span class='line'>  <span class="k">WHERE</span> <span class="n">q</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">q</span><span class="p">.</span><span class="n">category_id</span> <span class="o">=</span> <span class="n">fle</span><span class="p">.</span><span class="n">id</span>
</span><span class='line'><span class="p">)</span>
</span><span class='line'><span class="k">SELECT</span> <span class="o">*</span> <span class="k">from</span> <span class="n">first_level_elements</span><span class="p">;</span>
</span></code></pre></td></tr></table></div></figure>


<p>But wait. The recursive term is only fetching questions. What if the child of a first-level category is a category?
Postgres doesn&#8217;t let us reference the non-recursive term more than once. So doing a <code>UNION</code> of categories and questions is out.
We have to be a little creative here:</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
<span class='line-number'>7</span>
<span class='line-number'>8</span>
<span class='line-number'>9</span>
<span class='line-number'>10</span>
<span class='line-number'>11</span>
<span class='line-number'>12</span>
<span class='line-number'>13</span>
<span class='line-number'>14</span>
<span class='line-number'>15</span>
<span class='line-number'>16</span>
<span class='line-number'>17</span>
<span class='line-number'>18</span>
<span class='line-number'>19</span>
<span class='line-number'>20</span>
<span class='line-number'>21</span>
<span class='line-number'>22</span>
<span class='line-number'>23</span>
<span class='line-number'>24</span>
</pre></td><td class='code'><pre><code class='sql'><span class='line'><span class="k">WITH</span> <span class="k">RECURSIVE</span> <span class="n">first_level_elements</span> <span class="k">AS</span> <span class="p">(</span>
</span><span class='line'>  <span class="p">(</span>
</span><span class='line'>    <span class="p">(</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">order_number</span><span class="p">,</span> <span class="n">category_id</span> <span class="k">FROM</span> <span class="n">questions</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">questions</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">questions</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'>    <span class="k">UNION</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">order_number</span><span class="p">,</span> <span class="n">category_id</span> <span class="k">FROM</span> <span class="n">categories</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">categories</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">categories</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'>    <span class="p">)</span>
</span><span class='line'>  <span class="p">)</span>
</span><span class='line'>  <span class="k">UNION</span>
</span><span class='line'>  <span class="p">(</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">content</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">order_number</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">category_id</span>
</span><span class='line'>      <span class="k">FROM</span>
</span><span class='line'>      <span class="p">(</span>
</span><span class='line'>        <span class="c1">-- Fetch questions AND categories</span>
</span><span class='line'>        <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">order_number</span><span class="p">,</span> <span class="n">category_id</span> <span class="k">FROM</span> <span class="n">questions</span> <span class="k">WHERE</span> <span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span>
</span><span class='line'>        <span class="k">UNION</span>
</span><span class='line'>        <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">order_number</span><span class="p">,</span> <span class="n">category_id</span> <span class="k">FROM</span> <span class="n">categories</span> <span class="k">WHERE</span> <span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span>
</span><span class='line'>      <span class="p">)</span> <span class="n">e</span><span class="p">,</span> <span class="n">first_level_elements</span> <span class="n">fle</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">category_id</span> <span class="o">=</span> <span class="n">fle</span><span class="p">.</span><span class="n">id</span>
</span><span class='line'>  <span class="p">)</span>
</span><span class='line'><span class="p">)</span>
</span><span class='line'><span class="k">SELECT</span> <span class="o">*</span> <span class="k">from</span> <span class="n">first_level_elements</span><span class="p">;</span>
</span></code></pre></td></tr></table></div></figure>


<p>We perform a <code>UNION</code> of categories and questions <em>before</em> joining it with the non-recursive term.</p>

<p>This yields all the survey elements:</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/all-elements-without-order-query.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/all-elements-without-order-query.png" alt="Query without ordering" /></a></p>

<p>Unfortunately, it looks like the ordering is way off.</p>

<h3>Ordering a Recursive Query</h3>

<p>The problem is that since we&#8217;re effectively <em>appending</em> second-level elements to first-level elements, we&#8217;re effectively performing a <a href="https://en.wikipedia.org/wiki/Breadth-first_search"><em>breadth-first search</em></a> instead of a <a href="http://en.wikipedia.org/wiki/Depth-first_search"><em>depth-first search</em></a>.</p>

<p>How can we correct that?</p>

<p>Postgres has <a href="http://www.postgresql.org/docs/9.1/static/arrays.html">arrays</a> that can be built during a query.</p>

<p>Let&#8217;s build an array of order numbers for each element that we fetch. Let&#8217;s call this the <code>path</code>. The <code>path</code> for an element is:</p>

<blockquote><p>the path of it&#8217;s parent category (if one exists) + its own order_number</p></blockquote>

<p>If we sort the final result by <code>path</code>, we&#8217;ve converted it into a depth-first search!</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
<span class='line-number'>7</span>
<span class='line-number'>8</span>
<span class='line-number'>9</span>
<span class='line-number'>10</span>
<span class='line-number'>11</span>
<span class='line-number'>12</span>
<span class='line-number'>13</span>
<span class='line-number'>14</span>
<span class='line-number'>15</span>
<span class='line-number'>16</span>
<span class='line-number'>17</span>
<span class='line-number'>18</span>
<span class='line-number'>19</span>
<span class='line-number'>20</span>
<span class='line-number'>21</span>
<span class='line-number'>22</span>
<span class='line-number'>23</span>
</pre></td><td class='code'><pre><code class='sql'><span class='line'><span class="k">WITH</span> <span class="k">RECURSIVE</span> <span class="n">first_level_elements</span> <span class="k">AS</span> <span class="p">(</span>
</span><span class='line'>  <span class="p">(</span>
</span><span class='line'>    <span class="p">(</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">category_id</span><span class="p">,</span> <span class="nb">array</span><span class="p">[</span><span class="n">id</span><span class="p">]</span> <span class="k">AS</span> <span class="n">path</span> <span class="k">FROM</span> <span class="n">questions</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">questions</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">questions</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'>    <span class="k">UNION</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">category_id</span><span class="p">,</span> <span class="nb">array</span><span class="p">[</span><span class="n">id</span><span class="p">]</span> <span class="k">AS</span> <span class="n">path</span> <span class="k">FROM</span> <span class="n">categories</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">categories</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">categories</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'>    <span class="p">)</span>
</span><span class='line'>  <span class="p">)</span>
</span><span class='line'>  <span class="k">UNION</span>
</span><span class='line'>  <span class="p">(</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">content</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">category_id</span><span class="p">,</span> <span class="p">(</span><span class="n">fle</span><span class="p">.</span><span class="n">path</span> <span class="o">||</span> <span class="n">e</span><span class="p">.</span><span class="n">id</span><span class="p">)</span>
</span><span class='line'>      <span class="k">FROM</span>
</span><span class='line'>      <span class="p">(</span>
</span><span class='line'>        <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">category_id</span><span class="p">,</span> <span class="n">order_number</span> <span class="k">FROM</span> <span class="n">questions</span> <span class="k">WHERE</span> <span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span>
</span><span class='line'>        <span class="k">UNION</span>
</span><span class='line'>        <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">category_id</span><span class="p">,</span> <span class="n">order_number</span> <span class="k">FROM</span> <span class="n">categories</span> <span class="k">WHERE</span> <span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span>
</span><span class='line'>      <span class="p">)</span> <span class="n">e</span><span class="p">,</span> <span class="n">first_level_elements</span> <span class="n">fle</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">category_id</span> <span class="o">=</span> <span class="n">fle</span><span class="p">.</span><span class="n">id</span>
</span><span class='line'>  <span class="p">)</span>
</span><span class='line'><span class="p">)</span>
</span><span class='line'><span class="k">SELECT</span> <span class="o">*</span> <span class="k">from</span> <span class="n">first_level_elements</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">path</span><span class="p">;</span>
</span></code></pre></td></tr></table></div></figure>


<p><a href="http://blog.timothyandrew.net/images/recursive-pg/almost.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/almost.png" alt="Query with duplicate" /></a></p>

<p>That&#8217;s <em>almost</em> right. There are two entries for <em>What&#8217;s your favourite song?</em></p>

<p>This is happening because when we do an ID comparison to look for children:</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
</pre></td><td class='code'><pre><code class='sql'><span class='line'><span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">category_id</span> <span class="o">=</span> <span class="n">fle</span><span class="p">.</span><span class="n">id</span>
</span></code></pre></td></tr></table></div></figure>


<p><code>fle</code> contains both questions and categories. But we want this to match only categories (because questions can&#8217;t have children).</p>

<p>Let&#8217;s hard-code a type into each of these queries, so that we don&#8217;t try and check for children of a question:</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
<span class='line-number'>7</span>
<span class='line-number'>8</span>
<span class='line-number'>9</span>
<span class='line-number'>10</span>
<span class='line-number'>11</span>
<span class='line-number'>12</span>
<span class='line-number'>13</span>
<span class='line-number'>14</span>
<span class='line-number'>15</span>
<span class='line-number'>16</span>
<span class='line-number'>17</span>
<span class='line-number'>18</span>
<span class='line-number'>19</span>
<span class='line-number'>20</span>
<span class='line-number'>21</span>
<span class='line-number'>22</span>
<span class='line-number'>23</span>
<span class='line-number'>24</span>
</pre></td><td class='code'><pre><code class='sql'><span class='line'><span class="k">WITH</span> <span class="k">RECURSIVE</span> <span class="n">first_level_elements</span> <span class="k">AS</span> <span class="p">(</span>
</span><span class='line'>  <span class="p">(</span>
</span><span class='line'>    <span class="p">(</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">category_id</span><span class="p">,</span> <span class="s1">&#39;questions&#39;</span> <span class="k">as</span> <span class="k">type</span><span class="p">,</span> <span class="nb">array</span><span class="p">[</span><span class="n">id</span><span class="p">]</span> <span class="k">AS</span> <span class="n">path</span> <span class="k">FROM</span> <span class="n">questions</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">questions</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">questions</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'>    <span class="k">UNION</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">category_id</span><span class="p">,</span> <span class="s1">&#39;categories&#39;</span> <span class="k">as</span> <span class="k">type</span><span class="p">,</span> <span class="nb">array</span><span class="p">[</span><span class="n">id</span><span class="p">]</span> <span class="k">AS</span> <span class="n">path</span> <span class="k">FROM</span> <span class="n">categories</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">categories</span><span class="p">.</span><span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span> <span class="k">AND</span> <span class="n">categories</span><span class="p">.</span><span class="n">category_id</span> <span class="k">IS</span> <span class="k">NULL</span>
</span><span class='line'>    <span class="p">)</span>
</span><span class='line'>  <span class="p">)</span>
</span><span class='line'>  <span class="k">UNION</span>
</span><span class='line'>  <span class="p">(</span>
</span><span class='line'>      <span class="k">SELECT</span> <span class="n">e</span><span class="p">.</span><span class="n">id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">content</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="n">category_id</span><span class="p">,</span> <span class="n">e</span><span class="p">.</span><span class="k">type</span><span class="p">,</span> <span class="p">(</span><span class="n">fle</span><span class="p">.</span><span class="n">path</span> <span class="o">||</span> <span class="n">e</span><span class="p">.</span><span class="n">id</span><span class="p">)</span>
</span><span class='line'>      <span class="k">FROM</span>
</span><span class='line'>      <span class="p">(</span>
</span><span class='line'>        <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">category_id</span><span class="p">,</span> <span class="s1">&#39;questions&#39;</span> <span class="k">as</span> <span class="k">type</span><span class="p">,</span> <span class="n">order_number</span> <span class="k">FROM</span> <span class="n">questions</span> <span class="k">WHERE</span> <span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span>
</span><span class='line'>        <span class="k">UNION</span>
</span><span class='line'>        <span class="k">SELECT</span> <span class="n">id</span><span class="p">,</span> <span class="n">content</span><span class="p">,</span> <span class="n">category_id</span><span class="p">,</span> <span class="s1">&#39;categories&#39;</span> <span class="k">as</span> <span class="k">type</span><span class="p">,</span> <span class="n">order_number</span> <span class="k">FROM</span> <span class="n">categories</span> <span class="k">WHERE</span> <span class="n">survey_id</span> <span class="o">=</span> <span class="mi">2</span>
</span><span class='line'>      <span class="p">)</span> <span class="n">e</span><span class="p">,</span> <span class="n">first_level_elements</span> <span class="n">fle</span>
</span><span class='line'>      <span class="c1">-- Look for children only if the type is &#39;categories&#39;</span>
</span><span class='line'>      <span class="k">WHERE</span> <span class="n">e</span><span class="p">.</span><span class="n">category_id</span> <span class="o">=</span> <span class="n">fle</span><span class="p">.</span><span class="n">id</span> <span class="k">AND</span> <span class="n">fle</span><span class="p">.</span><span class="k">type</span> <span class="o">=</span> <span class="s1">&#39;categories&#39;</span>
</span><span class='line'>  <span class="p">)</span>
</span><span class='line'><span class="p">)</span>
</span><span class='line'><span class="k">SELECT</span> <span class="o">*</span> <span class="k">from</span> <span class="n">first_level_elements</span> <span class="k">ORDER</span> <span class="k">BY</span> <span class="n">path</span><span class="p">;</span>
</span></code></pre></td></tr></table></div></figure>


<p><a href="http://blog.timothyandrew.net/images/recursive-pg/final.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/final.png" alt="Final Query" /></a></p>

<p>That seems about right. We&#8217;re done here.</p>

<p>Let&#8217;s see how much of a performance boost this gives us.</p>

<h3>Performance</h3>

<p>Using this script (after creating a survey from the UI), I generated 10 sub-question chains; each 6 levels deep.</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
<span class='line-number'>6</span>
<span class='line-number'>7</span>
<span class='line-number'>8</span>
</pre></td><td class='code'><pre><code class='ruby'><span class='line'><span class="n">survey</span> <span class="o">=</span> <span class="no">Survey</span><span class="o">.</span><span class="n">find</span><span class="p">(</span><span class="mi">9</span><span class="p">)</span>
</span><span class='line'><span class="mi">10</span><span class="o">.</span><span class="n">times</span> <span class="k">do</span>
</span><span class='line'>  <span class="n">category</span> <span class="o">=</span> <span class="no">FactoryGirl</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="ss">:category</span><span class="p">,</span> <span class="ss">:survey</span> <span class="o">=&gt;</span> <span class="n">survey</span><span class="p">)</span>
</span><span class='line'>  <span class="mi">6</span><span class="o">.</span><span class="n">times</span> <span class="k">do</span>
</span><span class='line'>    <span class="n">category</span> <span class="o">=</span> <span class="no">FactoryGirl</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="ss">:category</span><span class="p">,</span> <span class="ss">:category</span> <span class="o">=&gt;</span> <span class="n">category</span><span class="p">,</span> <span class="ss">:survey</span> <span class="o">=&gt;</span> <span class="n">survey</span><span class="p">)</span>
</span><span class='line'>  <span class="k">end</span>
</span><span class='line'>  <span class="no">FactoryGirl</span><span class="o">.</span><span class="n">create</span><span class="p">(</span><span class="ss">:single_line_question</span><span class="p">,</span> <span class="ss">:category_id</span> <span class="o">=&gt;</span> <span class="n">category</span><span class="o">.</span><span class="n">id</span><span class="p">,</span> <span class="ss">:survey_id</span> <span class="o">=&gt;</span> <span class="n">survey</span><span class="o">.</span><span class="n">id</span><span class="p">)</span>
</span><span class='line'><span class="k">end</span>
</span></code></pre></td></tr></table></div></figure>


<p>Each chain looks like this:</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/chain.png"><img src="http://blog.timothyandrew.net/images/recursive-pg/chain.png" alt="Sub-question Chain" /></a></p>

<p>Let&#8217;s see if recursive queries are faster.</p>

<figure class='code'><figcaption><span></span></figcaption><div class="highlight"><table><tr><td class="gutter"><pre class="line-numbers"><span class='line-number'>1</span>
<span class='line-number'>2</span>
<span class='line-number'>3</span>
<span class='line-number'>4</span>
<span class='line-number'>5</span>
</pre></td><td class='code'><pre><code class='ruby'><span class='line'><span class="n">pry</span><span class="p">(</span><span class="n">main</span><span class="p">)</span><span class="o">&gt;</span> <span class="no">Benchmark</span><span class="o">.</span><span class="n">ms</span> <span class="p">{</span> <span class="mi">5</span><span class="o">.</span><span class="n">times</span> <span class="p">{</span> <span class="no">Survey</span><span class="o">.</span><span class="n">find</span><span class="p">(</span><span class="mi">9</span><span class="p">)</span><span class="o">.</span><span class="n">sub_questions_using_recursive_queries</span> <span class="p">}}</span>
</span><span class='line'><span class="o">=&gt;</span> <span class="mi">36</span><span class="o">.</span><span class="mi">839999999999996</span>
</span><span class='line'>
</span><span class='line'><span class="n">pry</span><span class="p">(</span><span class="n">main</span><span class="p">)</span><span class="o">&gt;</span> <span class="no">Benchmark</span><span class="o">.</span><span class="n">ms</span> <span class="p">{</span> <span class="mi">5</span><span class="o">.</span><span class="n">times</span> <span class="p">{</span> <span class="no">Survey</span><span class="o">.</span><span class="n">find</span><span class="p">(</span><span class="mi">9</span><span class="p">)</span><span class="o">.</span><span class="n">sub_questions_in_order</span> <span class="p">}</span> <span class="p">}</span>
</span><span class='line'><span class="o">=&gt;</span> <span class="mi">1145</span><span class="o">.</span><span class="mi">1309999999999</span>
</span></code></pre></td></tr></table></div></figure>


<p>31x faster? Not bad.</p>

<p><a href="http://blog.timothyandrew.net/images/recursive-pg/not-bad.jpg"><img src="http://blog.timothyandrew.net/images/recursive-pg/not-bad.jpg" alt="Not bad" /></a></p><div class="author">
  <img src="http://nilenso.com/people/timothy-200.jpg" style="width: 96px; height: 96;">
  <span style="position: absolute; padding: 32px 15px;">
    <i>Original post by <a href="http://twitter.com/">Timothy Andrew</a> - check out <a href="http://blog.timothyandrew.net/">Timothy's Blog</a></i>
  </span>
</div>
