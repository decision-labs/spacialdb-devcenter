# The SpacialDB Devcenter

This repo holds the content to the SpacialDB [devcenter.spacialdb.com][dev-center].

We are using [Gollum][gollum], [Gollum-Site][gol-site] and [Pygments][pygments-site] to generate this site and make it easier for people to contribute. Before contributing do make sure you have Pygments installed:

    easy_install Pygments

## How to Contribute

Treat this documentation like any `ruby` code repository. If you have a minor addition or fix, then select the particular file and press the 'Fork and edit file' button in Github. For something entirely new you can:

1. [Fork this repo][spacialdb-devcenter]

2. Build a local copy:

        git clone git://github.com/spacialdb/spacialdb-devcenter.git
        cd spacialdb-devcenter
        gem install bundler
        bundle install


3. Create a new branch:

        git checkout -b <branch-name>


4. Make your changes in [Markdown][markdown] using your favourite editor. To view your edits run the following command and go to [localhost:8000][localhost]:

        bundle exec gollum-site --watch --working serve


5. Run the specs and fix any formatting issues:

        rake spec

6. Commit, Push and send us a Pull request.

[dev-center]: http://devcenter.spacialdb.com
[spacialdb-devcenter]: https://github.com/spacialdb/spacialdb-devcenter
[gollum]:     https://github.com/github/gollum "Gollum Repo"
[gol-site]:   https://github.com/dreverri/gollum-site "Gollum-Site Repo"
[pygments-site]: http://pygments.org/ "Pygments a generic syntax highlighter"
[markdown]: http://daringfireball.net/projects/markdown/ "Markdown"
[localhost]: http://localhost:8000 "Gollum-site frontend"