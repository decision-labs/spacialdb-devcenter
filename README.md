# The SpacialDB Wiki

This repo holds the content to the SpacialDB [devcenter.spacialdb.com][dev-center].

We are using [Gollum][gollum] and [Gollum-Site][gol-site] to generate this site and to make it easier for people to contribute.

## How to Contribute

Treat this documentation like any code repository. If you have a minor addition or fix, then select the particular file and press the 'Fork and edit file' button. For something entirely new you can:

1. [Fork this repo][spacialdb-wiki]

2. Build a local copy:
```console
  git clone git://github.com/spacialdb/spacialdb-wiki.git
  cd spacialdb-wiki
  gem install bundler
  bundle install
```

3. Create a new branch:
```console
  git checkout -b <branch-name>
```

4. Make your changes with Gollum by running the following and going to [localhost:4567][localhost]:
```console
  gollum --page-file-dir pages
```

5. Run the specs and fix any formatting issues
```console
  rake spec
```

6. Push and send us a pull request

[dev-center]: http://devcenter.spacialdb.com
[spacialdb-wiki]: https://github.com/spacialdb/spacialdb-wiki
[gollum]:     https://github.com/github/gollum "Gollum Repo"
[gol-site]:   https://github.com/dreverri/gollum-site "Gollum-Site Repo"
[localhost]: http://localhost:4567 "Gollum frontend"