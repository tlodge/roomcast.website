---
title: How to store a building
layout: post
---
<div class="row">
  <div class="large-4 columns">
    <img src="/assets/img/buildinggraph.png"/>
  </div>
  <div class="large-6 columns">
    <p>Ok, slightly techie post this one.  Buildings are complex structures.  They have floors and apartments.  Each apartment may or may not have a neighbouring apartment, and may or may not have apartments above and below it.  When we were designing <strong>rooomcast</strong> we knew that it would be important to be able to talk about buildings in this way.  We could imagine many a scenario where it would be useful to get information about an apartment's location in relation to other things in the building.  So for example, it would be great to be able to get answers to questions like: which apartments are directly beneath <strong>B.3</strong>? or who are <strong>B.4's</strong> neighbours?</p>
  </div>
<div>

<div class="row">
  <div class="large-10 columns">

    <p>It turns out that modeling these kind of relationships in a traditional database, whilst possible, is fairly challenging.  Traditional databases store information as tables of rows and columns. The spatial relationships that we we're talking about don't translate all that well to this format. </p>

    <p>Fortunately, the world of social media has come to our rescue. Take Facebook, which is continually mining its users social graphs in a bunch of complex ways: which of Bob's friends of friends also liked this post?  Given that Alice knows Jane, who else might she know?  It's a similar problem.  Fortunately, in recent years people have been thinking about databases differently, and have been figuring out new ways to store information that is more appropriate to the kinds of queries that will be performed on it. </p>

    </p>And the one that solves our problems is the graph database.  It's brilliant for modeling not just buildings, but the residents who live in them, the owners, staff and the interactions between them.  Graph databases allow us to much more naturally structure the complex relationships between entities.  So now we can easily build a model that says Bob lives in apartment B.3 which is owned by John and which is adjacent to B.2 and above B.1 and on the second floor and in Block B and had a previous tenant called Katie and two problems with its intercom system and ...</p>

    <p>Anyway. <strong>Roomcast</strong> is built on a graph database.  And because of this we'll be talking about some pretty exciting new functionality in the future.</p>
  </div>
</div>
