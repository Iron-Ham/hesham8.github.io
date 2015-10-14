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

So the first thing y'all have to realize is that you need the proper driver for your database format. If you're using a SQLite database, you can use Xerial or Zentus. If you're using Oracle, Google around for Oracle's JDBC driver (it's there -- I promise), and so on and so forth. For the purposes of this demo, I'll just be using SQLite because it's a single file and we won't have to worry too much about scale (if any of you dare take a SQLite DB into production I will be very unhappy with y'all. If you already have a SQLite database and you want to take it into production, take a look at [converting your SQLite database to PostgreSQL](https://wiki.postgresql.org/wiki/Converting_from_other_Databases_to_PostgreSQL)).

Since I'm using SQLite, I'll be using the Xerial driver. The first thing you want to do in your Java program is import the SQL packages. A lot of people will tell you to do a `import java.sql.*;` call, but I think that's tacky and bad practice (if you don't agree with me, just know that you're also disagreeing with [Google's Java Style Guide](http://google-styleguide.googlecode.com/svn/trunk/javaguide.html)). Instead, just import the few packages you need. A few basic ones are `java.sql.DriverManager` and `java.sql.Connection`.

## Registering JDBC Driver

Once you've imported your basic Java SQL classes, it's time to register your driver. The most common way to do this is by using the `Class.forName()` method, but before I show you the code to do this, I should explain what this method *does*. `Class.forName()` explicitly loads the driver class and is actually the recommended way of doing it if you're using the `DriverManager` framework (which you probably are). So for the purpose of this example, let's load our Xerial driver: `Class.forName("org.sqlite.JDBC");`. Assuming that the driver was written correctly (a safe assumption), it should create an instance and call `DriverManager.registerDriver` with that instance as the parameter. After that occurs, your driver is in the `DriverManager` list of drivers and available for creating a connection.

As a quick aside, you can also do things the Microsoft way by calling `DriverManager.registerDriver()` instead of using `Class.forName()`. This *totally* works but is not recommended; it's meant for non-JDK compliant JVMs.

{% highlight java %}
Driver d = new org.sqlite.JDBC();
DriverManager.registerDriver(d);
{% endhighlight %}

## Creating A Database Connection

Assuming your database doesn't have a username and password associated with it, establishing a connection is fairly simple: `Connection c = DriverManager.getConnection(url);`, where `url` is the database url (such as `jdbc:sqlite:mydb.db`). But since the real world is a funny place with odd things like "security measures", we often have usernames and passwords associated with our database. In this case, we use a variant of the method: `Connection c = DriverManager.getConnection(url, user, pass);`. Finally, you can also use a `Properties` object instead of passing in a username and password. I'm not going to get into this method, but it's relatively straightforward: A `Properties` object holds a key:value pair, such that this is the replacement for the username and password strings in the call.

The final thing I'm going to touch on here is that you *must* close your JDBC connections at the end of your JDBC program (okay, if you forget, Java's GC will take care of it but that's really bad practice). Since you likely have the previous calls in a try-catch block anyhow, just add a `finally` block to cap off the program. Closing the connection is pretty trivial: `myConnection.close();`.

So to cap this off, let's do a quick database demo:

{% highlight java %}
import java.sql.*; //JUST for the purpose of THIS demo. Don't actually do this in YOUR code!

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
      e.printSTackTrace();
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

The code up above should help you get started on your own projects. You'll notice that if you try to compile that code, you'll probably have some issues because you probably don't have an employee table or a database named "myawesomedb". Give it a shot on your own database or table. If you run into any issues, feel free to shoot me an email and I'd be happy to help diagnose the problem.

Good luck, and happy coding.
