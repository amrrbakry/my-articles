# TIL: Using Different Database Connection with ActiveRecord
_Updated on Feb 14, 2019_

I came upon an interesting problem a few days ago at work. I had some code inside an [active record transaction](https://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html) that updated some record in the db, called an external service, and then updated another record based on the response from that external service. It was something like this:

```ruby
def call
  ActiveRecord::Base.transaction do
    record1.update!(some_data)
    external_service_response = Communicator.call!(record1, save_error: true) # raises exception if response is not a success
    record2.update!(external_service_response)
  end
end
```

The communicator class uses `record1` data to build a request and call an external service. It also handles authentication and response parsing; if the response is a success, it parses and returns it. otherwise, _it updates a record in the db with the error response_ if the `save_error` flag is set to `true`, and finally raises an exception, causing the active record transaction to roll back everything.


## The Problem

the issue was that the error response was not saved to the db. Do you see why?! Because it's inside the transaction; so when the communicator class raises an exception, the transaction rolls back everything, including the operation to save the error message to the database. well, duh.


## The Solution

the most obvious solution and the one you should go for in most cases is to move the communicator call outside the transaction, and the save error operation won't be rolled back. However, I couldn't do that so I had to do more thinking and googling...

I came across another neat solution and definitely learned something new. It uses the fact that an active record transaction "acts on a single database connection"; which basically means that if we use a different connection -from the one the transaction is using- to update a record, the transaction won't cover the update operation, and it won't be rolled back in case of an exception.

So, to solve my problem, I need to make a new thread (since each thread will use a different database connection), obtain a connection, and update the record with the error response.

## ActiveRecord::Base.connection_pool

and to achieve all that, and to manage the database connections properly and make sure it's thread safe, we have [`ActiveRecord::Base.connection_pool`](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html)


From the docs:

> A connection pool synchronizes thread access to a limited number of database connections. The basic idea is that each thread checks out a database connection from the pool, uses that connection, and checks the connection back in. ConnectionPool is completely thread-safe, and will ensure that a connection cannot be used by two threads at the same time, as long as ConnectionPool's contract is correctly followed.


and so the final solution becomes:


```ruby
# communicator class
#
# in case of error response
def save_error_response(record)
  Thread.new do
    ActiveRecord::Base.connection_pool.with_connection do
      record.update(external_service_error_response)
    end
  end.join
end
```

the [`with_connection`](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html#method-i-with_connection) method will check out a connection, yield to the block (update the record), and check in the connection again to the pool after finishing. We also need to `join` the thread to make sure the main thread will wait for it to finish before exiting.


That's it! Now, the error response will be saved to the database and won't be rolled back in case of an exception as it's in a different thread and different connection from the transaction.

I hope my explanation of the problem and the solution was clear.

_this post was also published on [Dev.to](https://dev.to/amrrbakry/til-using-different-database-connection-with-activerecord-transactions-345o)._

_this post was also published on [medium](https://medium.com/@amrrbakry/til-using-different-database-connection-with-activerecord-transactions-adb3c25cc9f1)._
