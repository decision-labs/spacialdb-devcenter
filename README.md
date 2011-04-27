# The SpacialDB Wiki

This repo holds the content to the SpacialDB [devcenter.spacialdb.com][dev-center].

We are using [Gollum][gollum] and [Gollum-Site][gol-site] to generate this site and to make it easier for people to contribute.

## How to Contribute

Treat this documentation like any code repository. If you have a minor addition you can submit an issue with the change or for something entirely new you can simply:

1. [Fork this repo][spacialdb-wiki]

2. Build a local copy by doing the following in the console:
```console
  git clone git://github.com/spacialdb/spacialdb-wiki.git
  cd spacialdb-wiki
  gem install bundler
  bundle install
  gollum-site generate        # Will generate the files
  gollum-site serve           # Will start the gollum-site server
  gollum-site serve --watch   # Will start the gollum-site server and will 
                              # regenerate the site when changes are made
```

3. Create a new branch:
```console
  git checkout -b <branch-name>
```

4. Make your changes

5. Run the specs and fix any formatting issues
```console
  rake spec
```

6. Commit changes to your branch

7. Send a pull request

[dev-center]: http://devcenter.spacialdb.com
[spacialdb-wiki]: https://github.com/spacialdb/spacialdb-wiki
[gollum]:     https://github.com/github/gollum "Gollum Repo"
[gol-site]:   https://github.com/dreverri/gollum-site "Gollum-Site Repo"