# specdiff

An experiment with seeing how a pull request changes an application by diffing
its spec output with master, using RSpec's documentation formatter.

## Usage

To compare a branch named *my-feature-branch* with master:

```sh
specdiff my-feature-branch
```

To compare *my-feature-branch* with *my-other-feature-branch*:

```sh
specdiff my-other-feature-branch my-feature-branch
```

Any tree reference can be used for the arguments:

```sh
specdiff origin/master 7b67bf9
```

## DIFF environment variable

The `DIFF` environment variable will be used if present.

## Example

Using [thoughtbot/hound#567][hound-pr]:

```sh
specdiff gl-refactor-hound-addition
```

```diff
--- before.txt	2015-01-15 13:27:29.000000000 -0800
+++ after.txt	2015-01-15 13:27:48.000000000 -0800
@@ -1,4 +1,4 @@
-I, [2015-01-15T13:27:12.568131 #12613]  INFO -- : ** [Raven] Raven 0.9.4 ready to catch errors
+I, [2015-01-15T13:27:34.973605 #12653]  INFO -- : ** [Raven] Raven 0.9.4 ready to catch errors
 
 ActivationsController#create
   when activation succeeds
@@ -136,26 +136,6 @@
   is buildable
 
 GithubApi
-  #add_user_to_repo
-    when repo is part of an organization
-      when repo is part of a team
-        when request succeeds
-          adds Hound user to first repo team with admin access and return true
-        when request fails
-          tries to add Hound user to first repo team with admin access and returns false
-      when repo is not part of a team
-        when Services team does not exist
-          creates a Services team and adds user to the new team
-        when 'Services' team exists
-          adds user to 'Services' team
-          when Services team is not on the first page of results
-            adds user to Services team
-          when Services team has pull access
-            updates permissions to push access
-        when 'services' team exists
-          adds user to 'services' team
-    when repo is not part of an organization
-      adds user as collaborator
   #repos
     fetches all repos from Github
   #create_hook
@@ -183,6 +163,16 @@
       does not crash
   #create_success_status
     makes request to GitHub for creating a success status
+  #add_user_to_team
+    makes a request to GitHub
+  #add_repo_to_team
+    makes a request to GitHub
+  #create_team
+    makes a request to GitHub
+  #add_collaborator
+    makes a request to GitHub
+  #update_team
+    makes a request
 
 JobQueue
   .push
@@ -667,6 +657,26 @@
   customer.subscription.deleted
     passes the stripe customer id to RepoDeactivator
 
+AddHoundToRepo
+  #run
+    with org repo
+      when repo is part of a team
+        when request succeeds
+          adds Hound user to first repo team with admin access
+        when request fails
+          returns false
+      when repo is not part of a team
+        when Services team does not exist
+          adds hound to new Services team
+        when Services team exists
+          adds user to existing Services team
+          when team name is lowercase
+            adds user to the team
+      when Services team has pull access
+        updates permissions to push access
+    when repo is not part of an organization
+      adds user as collaborator
+
 BuildRunner#run
   with active repo and opened pull request
     creates a build record with violations
@@ -693,28 +703,25 @@
 RepoActivator
   #activate
     when repo activation succeeds
-      activates repo
-      makes Hound a collaborator
-      returns true if the repo activates successfully
+      marks repo as active
+      adds Hound to the repo
       when https is enabled
         creates GitHub hook using secure build URL
       when https is disabled
         creates GitHub hook using insecure build URL
-    when repo activation fails
-      returns false if API request raises
-      reports raised exceptions to Sentry
+    when adding hound to rope results in an error
+      returns false
+      reports raised exception to Sentry
       only swallows Octokit errors
-      when Hound cannot be added to repo
-        returns false
     hook already exists
       does not raise
   #deactivate
-    when repo activation succeeds
-      deactivates repo
+    when repo deactivation succeeds
+      marks repo as deactivated
       removes GitHub hook
-      returns true if the repo activates successfully
-    when repo activation succeeds
-      returns false if the repo does not activate successfully
+      returns true
+    when repo deactivation fails
+      returns false
       only swallows Octokit errors
 
 DeactivatePaidRepos
@@ -752,5 +759,5 @@
   loads the Segment.io Javascript library
   records a pageview
 
-Finished in 16.16 seconds (files took 2.84 seconds to load)
-328 examples, 0 failures
+Finished in 13.22 seconds (files took 2.82 seconds to load)
+330 examples, 0 failures
```

[hound-pr]: https://github.com/thoughtbot/hound/pull/567
