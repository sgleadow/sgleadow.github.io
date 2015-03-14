stewgleadow
===========

Installaton
-----------

```
gem install bundler
bundle install --path vendor
```

Running
-------

Use bundler to run `jekyll`. Since the *baseurl* is configured to run under a sub-directory, we need to override the *baseurl* parameter:

```
bundle exec jekyll serve --baseurl ''
```

And since I can never remember that, I've added a short helper script:

```
./serve
```

Once running, you should be able to hit the site at `http://localhost:4000/`.

Authors
-------

Stewart Gleadow
Gmail: sgleadow
Github: sgleadow
Blog: stewgleadow.com
