###Channel Coffescript Usage Examples

1. Ensure you have a Redis server installed and running. For example, once installed, you can run it locally by

        redis-server &

1. Require `node-redis-queue` Channel

        Channel = require('node-redis-queue').Channel

1. Create a Channel instance and connect to Redis

        channel = new Channel()  
        channel.connect ->
          console.log 'ready'
          myMainLogic()

  Alternatively, you can provide an existing Redis connection (i.e., a redis client instance)

        channel.attach redisConn

1. Optionally, handle error events

        channel.on 'error', (error) ->  
            console.log 'Stopping due to: ' + error  
            process.exit()

1. Optionally, handle lost connection events

        channel.on 'end', ->
          console.log 'Connection lost'

1. Optionally, handle drain events

        channel.on 'drain', ->
          console.log 'Command queue too full -- slowing input'
          throttleInput()

1. Optionally, handle timeout events

        channel.on 'timeout', (queueNames...) ->
          console.log 'Timed out waiting for ', queueNames

1. Optionally, clear previous data from the queue, providing a callback
   to handle the data.

        channel.clear queueName, ->
          console.log 'cleared'
          doImportantStuff()

1. Optionally, push data to your queue

        channel.push queueName, myData

1. Optionally, pop data off your queue

        channel.pop queueName, (myData) ->  
            console.log 'data = ' + myData

   or, alternatively, pop off any of multiple queues

        channel.popAny queueName1, queueName2, (myData) ->
            console.log 'data = ' + myData 

   Once popping data from a queue, avoid pushing data to the same queue from the same connection, since
   a hang could result. This appears to be a Redis limitation when using blocking reads.

1. To avoid blocking indefinitely there are variations of the above that accept a timeout parameter.

        channel.popTimeout queueName, timeout, (myData) ->  
            console.log 'data = ' + myData

   or, alternatively, pop off any of multiple queues

        channel.popAnyTimeout queueName1, queueName2, timeout, (myData) ->
            console.log 'data = ' + myData

   where the timeout is in seconds.

1. When done, quit the Channel instance

        channel.disconnect()

  or, alternatively, if consuming data from the queue, end the connection

        channel.end()

  or, if there may be a number of redis commands queued,

        channel.shutdownSoon()

###WorkQueueMgr Coffeescript Usage Example

1. Ensure you have a Redis server installed and running. For example, once installed, you can run it locally by

        redis-server &

1. Require `node-redis-queue` WorkQueueMgr

        WorkQueueMgr = require('node-redis-queue').WorkQueueMgr

1. Create a WorkQueueMgr instance and connect to Redis

        mgr = new WorkQueueMgr()  
        mgr.connect ->
          console.log 'ready'
          myMainLogic()

  Alternatively, you can provide an existing Redis connection (i.e., a redis client instance)

        mgr.attach redisConn

1. Optionally, handle error events

        mgr.on 'error', (error) ->  
            console.log 'Stopping due to: ' + error  
            process.exit()

1. Optionally, handle lost connection events

        mgr.on 'end', ->
          console.log 'Connection lost'

1. Optionally, handle drain events

        mgr.on 'drain', ->
          console.log 'Command queue too full -- slowing input'
          throttleInput()

1. Optionally, handle timeout events

        mgr.on 'timeout', (queueNames...) ->
          console.log 'Timed out waiting for ', queueNames

1. Create a work queue instance

        queue = mgr.createQueue queueName

1. Optionally, clear previous data from the queue, providing a callback
   to handle the data.

        queue.clear ->
          console.log 'cleared'   
          doImportantStuff()

   Alternatively, you can clear multiple queues at once.

        mgr.clear queueName1, queueName2, ->
          console.log 'cleared both queues'
          doImportantStuff()

1. Optionally, send data to your queue

        queue.send myData

1. Optionally, consume data from your queue and call ack when ready to consume another data item

        queue.consume (myData, ack) ->  
            console.log 'data = ' + myData   
            ...
            ack()

   or, alternatively,

        queue.consume (myData, ack) ->  
            console.log 'data = ' + myData   
            ...
            ack()
        , arity

   where arity is an integer indicating the number of async callbacks to schedule in parallel. See demo 04 for example usage.

   If multiple queues are being consumed, they are consumed with highest priority given to the queues consumed first
   (i.e., in the order in which the consume statements are executed).

   Note that ack(true) may be used to indicate that no further data is expected from the given work queue.
   This is useful, for example, in testing, when a clean exit from a test case is desired.

   Once consuming from a queue, avoid sending data to the same queue from the same connection
   (i.e., the same mgr instance), since a hang could result. This appears to be a Redis limitation when using
   blocking reads. You can test `mgr.channel.outstanding` for zero to determine if it is OK to send on the same connection.

   Following the arity parameter, one may provide a timeout parameter so that the operations do not block indefinitely.

1. Optionally, destroy a work queue if it no longer is needed. Assign null to the queue variable to free up memory.

        queue.destroy()
        queue = null

1. When done, quit the WorkQueueMgr instance

        mgr.disconnect()

  or, alternatively, if consuming data from the queue, end the connection

        mgr.end()

  or, if there may be a number of redis commands queued,

        mgr.shutdownSoon()

