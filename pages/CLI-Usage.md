Running `spacialdb help` displays the following usage summary:
```console
$ spacialdb help
Usage: spacialdb COMMAND [command-specific-options]

Primary help topics, type "spacialdb help TOPIC" for more details:

  auth  # authentication (signup, login, logout)
  db    # manage databases (create, destroy)

Additional topics:

  help     # show this help
  version  # version
```

## Authentication Commands

The `signup` command simply signs you up to a free plan:
```console
$ spacialdb signup
Sign up to Spacialdb.
Email: kashif@spacialdb.com
Username: kashif
Password: 
Password confirmation: 
Signed up successfully.
```

The `login' command will log you in with you credentials and place your private API key in `~/.spacialdb/credentials`.
```console
$ spacialdb login
Enter your Spacialdb credentials.
Email or Username: krasul
Password: 
Logged in successfully.
$ cat ~/.spacialdb/credentials
kashif@spacialdb.com
92b2d0974304202d68827055cc04344d
```

The `logout` command will remove your credential file.