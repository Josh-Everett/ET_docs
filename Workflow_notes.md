## Workflow Notes ##

```bash
gerrit take-bug
```

Taking a bug will:

    -> Update its state in Jira to IN_PROGRESS
    -> Update its state in Bugzilla to ASSIGNED
    -> Create and use a git 'branch' called bug_<bug id>

Verify that this branch exists using this git command:

```bash
    git-branch
    git-status
```

This is the point where you start investigations, asking questions, making 
changes, debugging, writing unit tests, and release notes.

Checking in for a code review. This is done in a number of steps:

- First you can check your branch to see what changes you have added to it.
```bash
    git-status
```

You can see the actual changes that you could add with using this command:
```bash
    git-diff
```

You can add new files to your change list using this command:
```bash
    git-add <path_to_new_file_with_file_name_and_extension>
```

Add modifications to your change list with doing this:
```bash
    git-add -p
```

Review your handy work with git-status. That should show the list of files that you want code reviewed. You should also do a:
```bash
    git-diff --staged
```

Next we commit those changes to your local git repository. 
The first way is ONLY done on the FIRST commit for a fix (unless you like pain):
```bash
    git-commit
```

After the first commit, use the following variant for commits against the same
bug:
```bash
    git-commit --amend
```
You will be prompted for a commit message. 
Commit messages take the following format.
```
<a short description for a title>

<a longer description that can 
be multiple lines if

that is what 
you need>

Bug: <Bugzilla bug number>
```
_Avoid reusing commit comments without some change_

### Preparing code for code-review ###
verify your commit to see what has been commited
```bash

    git-show
```
and check if anything important has been left out
```bash
    git-diff
```

Now we use Jon's gerrit script to submit the changes
```bash
gerrit submit
```

which will not only request a code review but will also change:

    * the Bugzilla status to POST and
    * the Jira status to UNDER REVIEW.


Any negative score means you need to start over from the first git-status step 
and address the issues that were raised.

When you get +2 points, your code review is successful and you make continue. 
You can do this with the gerrit script:

    gerrit merge

But most of us just click on the Code-Review+2 button on the top of the screen
then click on the blue Submit button that will pop up.

Either way,

    -> The Bugzilla status will be updated to Modified and 
    -> The Jira status will be updated to MERGED


# Example workflow using Bug 1644051
https://bugzilla.redhat.com/show_bug.cgi?id=1644051
_ET API may assign incorrect Doc Reviewer_

```bash
gerrit --help

gerrit take-bug 1644051
```

Browse to local ET instance with an example advisory we can test
http://0.0.0.0:3000/advisory/21130

Now we modify the controller to allow us to debug a portion of the process by placing
the line 
```ruby
binding.pry
```
in /errata-rails/app/controllers/errata_controller.rb on line 1624

If we refresh the page the browser halts on the line 
We can use the rails console to set a breakpoint on line 1630 and use

```bash
continue
```
to allow lines to run until we reach the breakpoint


_change_docs_reviewer_modal.html.erb
```ruby
<div class="flash_container fixedTop">
    <% unless User.display_user.in_role?('docs') && User.display_user.enabled?  %>
    <% {'warning' => "The donkeys are loose!"}.each do |key, value| %>
      <div class="alert alert-<%= key %>">
        <a href="#" class="close" data-dismiss="alert">&times;</a>
        <%= value %>
      </div>
    <% end %>
    <% end %>
</div>
```

Now we can check our modifications on Git
```bash
git-diff
git add -p
```

### Unit tests ###
Unit tests need to be made/run after changes are made
Inside of our docker image we run
```bash
rake test TEST=test/integration/browser_tests/change_state_modal_browser_test.rb
```

Now we make our initial commit
```
git diff --staged
git commit
```

Changes made after are ammended to original commit

git commit --amend
git show

Documentation for bug fix should be detailed in
```
publican_docs/release_notes/markdown/next_release
```