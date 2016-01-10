---
title: Managing Object Life Cycle in RxJava
tags: [java, rxjava]
---
Manually managing the life span of an object is often an side effect in programming. How does it fit into the functional reactive programming with [RxJava](http://reactivex.io)?

<!-- more -->

To make an SQL query in [Vertx](http://vertx.io),  you first need an [`SQLConnection`](http://vertx.io/docs/apidocs/io/vertx/rxjava/ext/sql/class-use/SQLConnection.html) from the [`JDBCClient`](http://vertx.io/docs/apidocs/io/vertx/rxjava/ext/jdbc/JDBCClient.html). Then you will issue the query from the `SQLConnection`.  With RxJava API, the code looks like this

``` java
jdbcClient.getConnectionObservable()
	.flatMap(conn -> conn.queryObservable("SELECT * FROM mutable;"))
	.subscribe(res -> System.out.println(res));
```

Pretty standard [observable pattern](http://reactivex.io/documentation/observable.html), isn't it? 

However, it is worth pointing out that we need to [close](http://vertx.io/docs/vertx-jdbc-client/java/#_closing_the_client) the `SQLConnection` and return it to the connection pool after we are done with the query. To achieve that, we might want to revise the code as

``` java
jdbcClient.getConnectionObservable()
	.subscribe(conn ->conn.queryObservable("SELECT * FROM mutable;")
        .subscribe(
	        res -> {
	            System.out.println(res);
	            conn.close();
	        }, 
            throwable -> conn.close()
		)
	);
```

Oh, this piece of code is probably the worse example to showcase the RxJava API: There are nested subscriptions, and the `close` method has to be repeated twice in the `onNext` callback and the `onError` callback. All such tedious boilerplate code is only for the mundane purpose of returning the connection back to the pool. Can we do better than this?

Actually there is a very clean solution by using the `using` operator. (Yes, the operator is called `using` so there is no typo here.) Essentially, it will create a disposable resource that has the same lifespan as the Observable. The following is the simplified signature of `using`:

``` java
Observable<T> using(
	Func0<Resource> resourceFactory,
	Func1<Resource,Observable<T>> observableFactory,
	Action1<Resource> disposeAction)
```

The parameters are 

* resourceFactory - the factory function to create a resource object that depends on the Observable
* observableFactory - the factory function to create an Observable
* disposeAction - the function that will dispose of the resource

Basically, you can use the factory to maintain the life cycle of the object emitted from the `Observable`. For example, we can define the following `SQLConnectionFactory`:

``` java
public class SQLConnectionFactory {
    private SQLConnection connection = null;

    public Observable<SQLConnection> create(JDBCClient jdbc){
        return jdbc.getConnectionObservable().flatMap(conn -> {
            connection = conn;
            return Observable.just(connection);
        });
    }

    public void dispose(){
        if (connection != null){
            connection.close();
			connection = null;
        }
    }
}
```

With `SQLConnectionFactory`, we only to revise the first line of code snippet to ensure the connection is probably closed, regardless whether the query succeeds or fails. 

``` java
Observable.using(SQLConnectionFactory::new, f -> f.create(jdbcClient), f -> f.dispose())
	.flatMap(conn -> conn.queryObservable("SELECT * FROM mutable;"))
	.subscribe(res -> System.out.println(res));
```


