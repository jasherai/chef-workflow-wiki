Contributing to the Wiki
------------------------

We don't allow edits by those without commit access to the wiki. This just
makes it a lot more reliable as a documentation source and keeps the spam at
bay.

Github provides its wikis as git repositories themselves, which is awesome --
unfortunately it does not make it especially easy to submit pull requests to
them, which is what we want; that way changes can be reviewed and discussed.

What we've done to resolve this is provide a mirror of the wiki repository as a
separate github repository: [here](https://github.com/chef-workflow/chef-workflow-wiki).

* Make a fork of the above repository
* Clone the repository to your local machine
* `gem install gollum`
* `cd` into the clone and type `gollum`
* Point your web browser at `localhost:4567`
* Make your edits
* Push your repository
* File a pull request.

Yes, this is more cumbersome than a couple of clicks to make your edit, but I
think it will result in higher-quality content for everyone, contributors and
casual readers alike. Thank you for your understanding.
