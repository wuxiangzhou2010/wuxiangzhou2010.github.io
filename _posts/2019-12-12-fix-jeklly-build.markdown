---
layout: post
title: "fix jekyll build failure"
date: 2019-12-12 12:12 +0800
categories: tools
published: false
---

## jekyll build fails

```sh
Traceback (most recent call last):
        12: from /Users/takesachishuu/.gem/ruby/2.6.0/bin/jekyll:23:in `<main>'
        11: from /Users/takesachishuu/.gem/ruby/2.6.0/bin/jekyll:23:in `load'
        10: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/jekyll-4.0.0/exe/jekyll:11:in `<top (required)>'
         9: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/jekyll-4.0.0/lib/jekyll/plugin_manager.rb:52:in `require_from_bundler'
         8: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler.rb:107:in `setup'
         7: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler/runtime.rb:20:in `setup'
         6: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler/runtime.rb:108:in `block in definition_method'
         5: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler/definition.rb:226:in `requested_specs'
         4: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler/definition.rb:237:in `specs_for'
         3: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler/definition.rb:170:in `specs'
         2: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler/spec_set.rb:81:in `materialize'
         1: from /Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler/spec_set.rb:81:in `map!'
/Users/takesachishuu/.gem/ruby/2.6.0/gems/bundler-2.0.2/lib/bundler/spec_set.rb:87:in `block in materialize': Could not find public_suffix-3.0.3 in any of the sources (Bundler::GemNotFound)
```

## Google the error I get

https://talk.jekyllrb.com/t/could-not-find-public-suffix-3-0-1-in-any-of-the-sources-bundler-gemnotfound/1603

```sh
gem install public_suffix --version 3.0.1
# not work
# ERROR:  While executing gem ... (Gem::FilePermissionError)
# You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory.

bundle update

# get
# Your user account isn't allowed to install to the system RubyGems.
#   You can cancel this installation and run:

bundle install --path vendor/bundle
```

## Fixed after reinstallation

- [Do a clean remove](https://stackoverflow.com/questions/53403684/how-to-start-over-with-a-clean-gems-install-for-jekyll)

```sh
gem uninstall --all

sudo gem uninstall --all
```

- [Reinstall Jekyll](https://jekyllrb.com/docs/installation/macos/)

The install method has changed for new version of MacOs

```sh
gem install --user-install bundler jekyll

# install in vendor foder
bundle install --path vendor/bundle
# start
bundle exec jekyll serve
```
