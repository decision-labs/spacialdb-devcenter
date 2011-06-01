# Querying SpacialDB via iOS

It is possible to perform complex geospatial queries from any iOS device and receive results, as say JSON, without too much hassle. This article will hopefully get you started with setting up the development environment so you can quickly start thinking about your app's business logic. Once again you need to sign up and make sure you have a SpacialDB database that you can use. Note down its connection settings as we will need it below.

## Getting Started

The [[iOS Dev Center|http://developer.apple.com/devcenter/ios/index.action]] has all the information you would need to get started with getting XCode etc. So once that is done, the next thing you would need to make things easier is to have [[Git|http://git-scm.com/]] installed:

```console
$ brew install git
```

## Download libpq for iOS

You will need to pull the PostgreSQL's libpq for iOS library from [[github|https://github.com/spacialdb/libpq-ios]].

```console
$ git clone git://github.com/spacialdb/libpq-ios.git
```

will create a `libpq-ios` folder in the directory where you ran the above command. Inside it you should double-click the `libpq-ios.xcodeproj/` folder to open it in XCode 4 and it should build `libpq.a` for any iOS device or simulator without any problems. Once you get this working, it simply a matter of linking your own iOS app with `libpq.a` and sending SpacialDB a query string via its API which you can find in [[Chapter 31|http://www.postgresql.org/docs/current/static/libpq.html]] of the PostgreSQL manual.

## libpq crash course

The very basic API simply opens a database connection using the `PQconnectdbParams()` function, given some keywords like `host`, `hostaddr`, `port`, etc. and this either returns a valid pointer to a `PGconn` structure or `null` in which case one can check for the error via the `PQstatus()` function. 

Once a `PGconn` is established we can send queries to the database server via the `PQexecParams()` method which returns a pointer to `PGresult` structure which hopefully contains the result of our query. 

Finally we can close the connection by calling the `PQfinish()` method. 

Obviously it is also possible to do this in a *non-blocking* manner as most of these functions have non-blocking analogs which are the preferred API to use anyways.

## Creating a new iOS app

In order to use the `libpq.a` in your own iOS app, simply drag and drop the `libpq-ios.xcodeproj/` inside your XCode's project navigator under your current project:

![XCode-1](/img/xcode-1.png)

After that you want to add the `libpq-ios` as a **Target Dependency** in the **Build Phases** of our iOS app.

![XCode-2](/img/xcode-2.png)

Then we add the `libpq.a` static library to Link it with our binary and we move the `libpq.a` into our Frameworks group.

![XCode-3](/img/xcode-3.png)


Finally we add the `libpq` headers so that we can include it inside our code. Simply right click on the **Support Files** group and select the **Add Files to...** item and select the `include` folder inside the root of the  `libpq-ios` repository. Make sure you have selected the **Create groups for any added folders** option.

![XCode-4](/img/xcode-4.png)


Finally you can now call the `libpq` include file: `libpq-fe.h`  inside your app and the project should compile without any issues.

![XCode-5](/img/xcode-5.png)

## Writing business logic

Now its up to you to write the logic for your app and use the `libpq` connection to query SpacialDB and receive results. In particular to connect you will need your connection string which is constructed from the database information in the SpacialDB Dashboard or CLI.

For example given something like this in our `appViewController.h`:

```objective-c
#import <UIKit/UIKit.h>
#include "libpq-fe.h"

@interface appViewController : UIViewController {
    const char *conninfo;
    PGconn     *conn;
    PGresult   *res;
}

@end
```

we could do something like the following in `appViewController.m`:

```objective-c
- (void)viewDidLoad
{
    [super viewDidLoad];

    conninfo = "dbname=spacialdb0_krasul host=beta.spacialdb.com port=9999 user=krasul password=mypasswd";
    conn = PQconnectdb(conninfo);

    if (PQstatus(conn) != CONNECTION_OK)
    {
        NSString *title = @"Connection to database failed:";
        NSString *message = [[NSString alloc] initWithUTF8String:PQerrorMessage(conn)];

        UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:title message:message delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil];
        [alertView show];
        [alertView release];
        [message release];
    }
}

- (void)viewDidUnload
{
    [super viewDidUnload];
    PQfinish(conn);
}
```