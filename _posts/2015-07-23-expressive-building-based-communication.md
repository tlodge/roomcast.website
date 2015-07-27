---
title: Expressive building-based communication
layout: post
---

<div class="row">

  <div class="large-4 columns">
    <img src="/assets/img/buildinggraph.png"/>
  </div>
  
  <div class="large-6 columns">
    <p> The ability to appropriately define and create discrete groups of users is a fundamental part of what makes <strong>roomcast</strong> so powerful. This blog visits some of our design decisions and finishes up with a look at how they are accomplished in code.  With <strong>roomcast</strong>, the communication system must make it easy for us to define groups of users that makes sense in a residential community.  There are lots of quite complex ways that we might want to define an audience of users.  Groups might be defined spatially : 'all the users living on the first floor' or 'all the users living on the second floors of block A and block B'.</p> 
  </div>
</div>
 
<div class="row">

  <div class="large-10 columns">
  <p>
 But we might also might also wish to target users based on a particular attribute: 'the person who's registration number is XYZ ABC'.  Or it could be a combination of attributes: 'All female users living on the second floors of block A and block B'.  We could conceivably wish to reach out to users based upon spatial relationships, so for example all landlords of apartments directly beneath apartment 56.   If we support this richness of expression, then it will mean that our platform, and any others that make use of it, can target users extremely accurately.  
 </div>
</div>

<div class="row">
  <div class="large-6 columns">
	<p>
		As a starting point, there are a bunch of core groups that users will be assigned to, dependent on who they are and where they live.  We know that we will sometimes want to differentiate between owners, tenants and residents (residents are all people that live it a building regardless of whether they own or rent it).  We know that we may also wish to differentiate between users based upon where their apartment is (i.e by block or by floor).   There are also  some basic attributes such as gender, age and length of tenancy that could be useful.  Finally, when we think about 'spaces', rather than 'users', we might want to communicate with devices located in the lobby, next to the lifts, in the underground parking area and so on. 
	</p>
  </div>

  <div class="large-4 columns">
	 <img src="/assets/img/accessgroups.png"/>
  </div>
</div>	

<div class="row">
	<div class="large-10 columns">
		 <p>  Once we have these basic groups (which we call AccessGroups), it is easy to join up each user to each group that they should be in.  But we also want to use these basic groups to compose more targeted recipient lists.  Let's say we want to create a button that can only be used by female tenants and owners on the first or second floor of block A.  Clearly we cannot simply send the button to all of the groups that are implicated (i.e. female, tenants, owners, first floor, second floor, block A) - since this would contain many unintended recipients (such as male users on the first floor). Instead, we need to break the intention down, into a set of rules joined together by ANDs and ORs: tenants <strong> OR </strong> owners <strong> AND </strong> females <strong> AND </strong> in block A <strong> AND </strong> on the first <strong> OR </strong> second floor. So we <strong>OR</strong> the access groups that are of the same type and <strong>AND</strong> the access groups of different types. </p>
	</div>
</div>

<div class="row">
	<div class="large-4 columns">
		 <img src="/assets/img/rules.png"/>
	</div>
	<div class="large-6 columns">
		 <p> So far, so good.  Once we have established who belongs to these more complex user groups we can create a new access group to store the users and use it again (and, crucially, know exactly who got a particular message or button). The final concern is what happens when a new user joins?  We can easily add them to the relevant core access groups, but we can't know whether they should also belong to the more complex access groups unless </p> 
	</div>
</div>   

<div class="row">
	<div class="large-10 columns">
 		<p>  we record the logic that was used to generate them in the first place.  To represent to rules that have been used to create one of these access groups, we can build a tree, with a root node which is our new CompositeAccessGroup.  All children of the CompositeAccessGroup node are ANDed together, and all children of those children are ORed.  When a new user joins, we simply evaluate whether they exist in each of the AND branches (by ORing together all of the leaf nodes), and if they do, then they can be added.  To accomplish this with a cypher query, we do the following: for each of the AND branches, collect together all users that satisfy membership. Then return all users that are found at least once in each branch.  These are the users that satisfy the conditions of the rule.   In the following code we first create a bunch of rules which are just memberships of access groups.  We then create some CompositeAccessGroups - which are like AccessGroups except that they have a tree of the rules that were used to create them.  The final query shows how we then add the relevant users to the CompositeAccessGroups.
		 </p>
		 <p>
		 <pre>
//create sets of rule (combinations of access groups)
CREATE  (owners)<-[:IN]-(t1:Rule {name:'ownersandtenants_rule'})-[:IN]->(tenants)
CREATE  (firstfloor)<-[:IN]-(t2:Rule {name:'firstfloor_rule'})
CREATE  (male)<-[:IN]-(t3:Rule {name:'male_rule'})
CREATE  (female)<-[:IN]-(t4:Rule {name:'female_rule'})
CREATE  (fourthfloor)<-[:IN]-(t5:Rule {name:'fourthfloor_rule'})

//create an access group: male tenants and owners on the first floor
CREATE  (cag1:CompositeAccessGroup {name:"male tenants and owners on first floor"})
CREATE (cag1)-[:HAS_RULE]->(t1)
CREATE (cag1)-[:HAS_RULE]->(t2)
CREATE (cag1)-[:HAS_RULE]->(t3)

//create an access group: female tenants and owners on the fourth floor
CREATE  (cag2:CompositeAccessGroup {name:"female tenants and owners on the fourth floor"})
CREATE (cag2)-[:HAS_RULE]->(t1)
CREATE (cag2)-[:HAS_RULE]->(t4)
CREATE (cag2)-[:HAS_RULE]->(t5);

//now attach all relevant users to the compound access group.
MATCH (cag:CompositeAccessGroup)-[:HAS_RULE]->(r1:Rule)
WITH cag, collect(r1) as rules
WITH cag, rules, length(rules) as rulestosatisfy
UNWIND rules as rule
MATCH rule-[:IN]->ag<-[:BELONGS_TO]-(u:User)
WITH cag, rule, rulestosatisfy, collect(DISTINCT(u.userId)) as users
WITH cag, rulestosatisfy, collect({rule:rule, users:users}) as rules
UNWIND rules as users
WITH cag, rulestosatisfy, EXTRACT (user in users.users | user) AS extracted
UNWIND extracted as user
WITH cag, rulestosatisfy, user, length(FILTER(auser in collect(user)  WHERE auser = user)) as counted
WHERE counted = rulestosatisfy
MATCH (u:User {userId:user})
CREATE UNIQUE (u)-[:BELONGS_TO]->(cag);
	</pre> 
	</p>
  </div>
</div>