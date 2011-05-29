# Querying SpacialDB from Android

By using the PostgreSQL's [[JDBC driver|http://jdbc.postgresql.org]] it is possible to connect to SpacialDB from a device running Android. This article will walk you through setting up the development environment to start writing your awesome android app.

## PostgreSQL JDBC Drivers for Android

For JDK 1.6, you should use the JDBC4 driver versions, even though support for JDBC4 methods is limited at the moment, and in fact several methods are stubbed out. For JDK 1.4 and 1.5, you should use the JDBC3 driver which contains support for SSL and `javax.sql`.

For Android the driver needs to be *newer* that build 801, and in this example we are using the build 900 dev version.

## Adding an External Library (.jar) using Eclipse

You can add the PostgreSQL JDBC jar in your application by adding it to your Eclipse project as follows:

1. In the **Package Explorer** panel, right-click on your project and select **Properties**.
2. Select **Java Build Path**, then the **Libraries** tab.
3. Press the **Add External JARs...** button and select the JAR file.

Alternatively, if you want to include the JDBC JAR with your package, create a new directory for it within your project and in the project **Properties**:

![Android-1](/img/android-1.png)

select **Add Library...** and add it there instead.

![Android-2](/img/android-2.png)

## JDBC Crash course

To use the the [[PostgreSQL JDBC Interface JDBC driver|http://jdbc.postgresql.org/documentation/head/index.html]] you will need to first import:

```java
import java.sql.*;
import java.util.Properties;
```

Before you can connect to a database, you need to load the driver:

```java
try {
  Class.forName("org.postgresql.Driver");
} catch (ClassNotFoundException e) {
  // TODO Auto-generated catch block
  e.printStackTrace();
}
```

And to connect, you need to get a `Connection` instance from JDBC. To do this, you use the `DriverManager.getConnection()` method which needs a `url` string and some properties:

```java
String url = "jdbc:postgresql://beta.spacialdb.com:9999/spacialdb0_krasul";
Properties props = new Properties();
props.setProperty("user","krasul");
props.setProperty("password","passwd");

try {
  Connection conn = DriverManager.getConnection(url, props);
} catch (SQLException e) {
  // TODO Auto-generated catch block
  e.printStackTrace();
}
```

Any time you want to issue SQL statements to the database, you require a `Statement` or `PreparedStatement` instance. Once you have a `Statement` or `PreparedStatement`, you can use issue a query. This will return a `ResultSet` instance, which contains the entire result.

In the end you will need to close the connection:
```java
finally {
  if (conn != null) {
    try {
      conn.close();
    } catch (SQLException e) {
      // TODO Auto-generated catch block
    }
  }
}
```