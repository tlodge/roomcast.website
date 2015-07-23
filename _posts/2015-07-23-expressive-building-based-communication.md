---
title: Expressive building-based communication
layout: post
---
<div class="row">
  <div class="large-4 columns">
    <img src="/assets/img/buildinggraph.png"/>
  </div>
  <div class="large-6 columns">
    <p>One thing that makes roomcast powerful is its ability to route messages according to an expressive set of criteria.  To support this, our underlying graph database has a simple notion of AccessGroups.  An AccessGroup can be, for example,<i> all tenants </i> or <i> all owners </i> or all people on the <i> first floor </i>.  So when the system needs to send to all people on the first floor, it simply looks up the first floor Access Group and sends the message to all users that belong to it.  With the live communication (i.e. sending out messages to each user's button panel) - we create channels that correspond to AccessGroups.  So a user can be a part of the tenants channel, the first floor channel the 'male' channel and so on.   This makes it easy for us to broadcast messages - rather than loop through each user in an access group to send out a message, we can simply send a message to the channel that represents the access group.  So far so good.    
  </div>
<div>

<div class="row">
  <div class="large-10 columns">
    <p> It gets a little more complex when we want to build more expressive recipient lists.  Let's say we want to send a message to all female tenants and owners living on the first or second floor.  Hmmm.  Clearly we cannot simply broadcast a message to all of the access groups that are implicated (i.e. female, tenants, owners, first floor, second floor) - since this would unintended recipients (such as people on the first floor who are not female), instead we will need to send a message to people that are tenants <strong>OR</strong> owners <strong> AND </strong>female<strong>AND</strong>on the first <strong>OR</strong>second floor</p>.  The rule is simple - you <strong>OR</strong> the access groups that are of the same type and <strong>AND</strong> the access groups of different types. 
    
    <p>Once we have figured out which users satisfy our 'all female tenants and owners living on the first or second floor' criteria, we could, in theory, create a new access group and a new channel, to avoid having to loop through each user and send a message.  There are several things that make this complex - and for now - we think would make the system quite brittle.  First, what happens when a new user joins?  Currently, we'll simply add him or her to all relevant access groups.  But how would we know whether to add the user to our 'all female tenants and owners living on the first or second floor' access group, unless we knew the rules that were used to compose it in the first place? We have the same problem when a user leaves - how would we know which access groups to remove him or her from?  Second, if under a new setting, the system wishes to send a message to 'all female tenants and owners living on the first or second floor' - how do we know whether an access group already exists for this combination?  The only way to accomplish this (that we can think of) is to model these composite AccessGroups as a tree of rules from which they are constructed.  Then when a new user joins we can re-evaluate all of the rules of all of the composite access groups and determine whether this user should be a member.  But...it complicates our model quite considerably, and adds a whole bunch of new management code to handle these composite groups.  So for these reasons, we're not going to do this.  Instead, when messages are to be directed anything other than a single access group, we'll work out on the fly the list of recipients and send the message to each.  This will mean that communication will 'just work' regardless of when new users join and leave... 
    </p>
  </div>
</div>
