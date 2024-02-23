# Blank Notebook
A personal web site where Ramesh shares his views, thoughts and experiences.

## Tech stack
* Github Pages [[↗]](https://pages.github.com/)
* Jekyll [[↗]](https://jekyllrb.com/)
* [Github Discussions](https://docs.github.com/en/discussions) with [Giscus](https://giscus.app/)

## Setup Local machine
- Install Ruby version >2.6 
  - Prefer using rbenv, rvm or chruby & ruby-install
  - For chruby & ruby install, follow below steps
    - `brew install chruby ruby-install`
    - Follow the post install steps from brew to add chruby source
    - `ruby-install ruby`
    - `chruby <version of ruby installed in previous step>` 
    - or with in project folder run `echo <ruby version> > .ruby-version`
- Enure you have proper versions of `gem`, `gcc` & `make`
- Update `.zshrc` or `.bashrc` to install gems to user folders
  ```
  sh
    # Ruby exports
    export GEM_HOME=$HOME/gems
    export PATH=$HOME/gems/bin:$PATH
  ```
- Install `Jekyll` & `bundler` by running `gem install jekyll bundler`
- Install dependencies `bundle install`

## How to run & test?
- To run the website locally `bundle exec jekyll serve`. with drafts `bundle exec jekyll serve --draft`