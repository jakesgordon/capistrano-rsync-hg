# Capistrano 3 deploy with Rsync & Hg

Deploy with Rsync to your server from a local Mercurial repository when using Capistrano. Saves
you from having to install Mercurial on your production machine and allows you to customize which
files you want to deploy.

>> **NOTE**: This library is a derivation of a Git flavored original [moll/capistrano-rsync](https://github.com/moll/capistrano-rsync)

## Using

Install with:

    gem install capistrano-rsync-hg           # or add to your Gemfile

Require it at the top of your Capfile

    require "capistrano/rsync"

Tell capistrano to use our Rsync SCM strategy within `deploy.rb`

    set :scm, :rsync

And add other related options to your liking:

    set :user,           "deploy"
    set :repo_url,       "ssh://hg@bitbucket.org/myusername/myrepository"
    set :branch,         "default"
    set :rsync_exclude,  %w[ .hg* ]
    set :rsync_options,  %w[ --archive --recursive --delete --delete-excluded ]
    set :local_cache,    ".cache_#{fetch(:stage)}"
    set :remote_cache,   "shared/cache"

 * `:user` - the user used in the rsync connection (optional).
 * `:repo_url` - the repository to clone.
 * `:branch` - the branch to checkout.
 * `:rsync_exclude` - array of files/paths to exclude from rsync (optional).
 * `:rsync_options` - additional rsync options (optional).
 * `:local_cache` - the local cache folder (where the repo will be checked out) - either absolute or relative to the local project root.
 * `:remote_cache` - the remote cache folder (where the files will be rsynced to) - either absolute or relative to capistrano `deploy_to` variable.

And after setting regular Capistrano options, deploy as usual:

    cap production deploy

## Implementation

 1. Clones and updates your repository to `local_cache` on your local machine.
 2. Checks out the branch set in the branch variable.
 3. Rsyncs to the `remote_cache` directory on the server.
 4. Copies the content of the `remote_cache` directory to a new release directory.

After that, Capistrano takes over and runs its usual tasks and symlinking.

## Exclude files from being deployed

If you don't want to deploy everything you've committed to your repository, specify `:rsync_exclude`

    set :rsync_exclude, %w[
      .hg*
      /config/database.yml
      /test/***
    ]

## Other tasks

    cap rsync:stage      # checkout the repo/branch into the :local_cache
    cap rsync:sync       # ... and sync to the server(s)
    cap rsync:release    # ... and copy to a release folder (but WITHOUT symlinking the current directory)

