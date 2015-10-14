---
layout: post
title: "Connecting to a Database: JDBC and SQL"
excerpt: "An introduction to database connection in Java"
tags: [database, code_snippet]
comments: true
<!-- image:
  feature: queen2.jpg
  credit: AlmostQueen
  creditlink: http://www.almostqueen.com/photos/ -->
---

I'm writing this post because it seems that a lot of my colleagues and classmates have some confusion about getting their Java programs to communicate with their SQL databases. Have no worries, y'all: I've got your back.

So the first thing y'all have to realize is that you need the proper driver for your database format. If you're using a SQLite database, you can use [Xerial](https://bitbucket.org/xerial/sqlite-jdbc). If you're using Oracle, you can use the [Oracle drivers](www.oracle.com/technetwork/database/features/jdbc/index-091264.html). For the purposes of this demo, I'll just be using SQLite because it's a single file and we won't have to worry too much about scale.

 So let's start a new java file and import the relevant SQL classes. A few good ones are `java.sql.DriverManager` and `java.sql.Connection`. For a more complete list, check out the [docs](http://docs.oracle.com/javase/7/docs/api/java/sql/package-summary.html).

## Registering JDBC Driver

Once you've imported your basic Java SQL classes, it's time to register your driver. The most common way to do this is by using the `Class.forName()` method. This method just returns the class object associated with the string-name; it explicitly loads the driver. For example, in SQLite, it would be: `Class.forName("org.sqlite.JDBC");`. Assuming everything goes well, this explicit loading of the JDBC driver *should* create an instance and call `DriverManager.registerDriver()` with that instance as the input parameter. Once that happens, your driver is now in the `DriverManager`'s list of drivers, and is now available for creating a connection.

## Creating A Database Connection

Assuming your database doesn't have a username and password associated with it, establishing a connection is fairly simple: `Connection c = DriverManager.getConnection(url);`, where `url` is the database url (such as `jdbc:sqlite:mydb.db`. But if we were to have a username and password, we use a variant of the method: `Connection c = DriverManager.getConnection(url, user, pass);`. Finally, you can also use a `Properties` object (which holds a key:value pair) instead of passing in a username and password.

The final thing is a reminder to close your JDBC connections at the end of your JDBC program. Yes, it still works if you *don't* close your connection, but it's bad practice. Since you likely have the previous calls in a try-catch block anyhow, just add a `finally` block to cap off the program. Closing the connection is pretty trivial: `myConnection.close();`.

So to cap this off, let's do a quick database demo:

{% highlight java %}
import ...

class MyClass {
  static final URL = "jdbc:sqlite:myawesomedb";

  public static void main(String[] args) {
    Connection c;
    Statement stmt;

    try {
      //Register your JDBC driver.
      Class.forName("org.sqlite.JDBC");
      //Open a connection
      c = DriverManager.getConnection(URL);

      //Execute a query
      stmt = c.createStatement();
      String sql = "SELECT id FROM Employees WHERE dept = 2";
      ResultSet rs = stmt.executeQuery(sql);

      //Get your data from your result-set
      while(rs.next()) {
        id = rs.getInt(id);
        //Do something with id
      }
      //Clean up
      rs.close();
      stmt.close();
      c.close();
    }
    catch (SQLException | ClassNotFoundException e) {
      e.printStackTrace();
    }
    finally {
      try {
        if (stmt != null) stmt.close();
      } catch (SQLException s) {
        //do nothing
      }
      try {
        if (c != null) c.close();
      } catch (SQLException s) {
        //do nothing
      }
    }
  }
}
{% endhighlight %}

The code up above should help you get started on your own projects. Give it a shot on your own database or table. If you run into any issues, feel free to shoot me an email and I'd be happy to help diagnose the problem.

Good luck, and happy coding.
