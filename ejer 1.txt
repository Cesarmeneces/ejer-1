class SampleTask implements Callable<Object>
{
   Set<MessageListener> listeners =      
      Collections.newSetFromMap(new ConcurrentHashMap<MessageListener, Boolean>();
   Topic targetTopic;
   TopicConnectionFactory tcf;

   public SampleWork(TopicConnectionFactory tcf, Topic targetTopic)
   {
      this.targetTopic = targetTopic;
      this.tcf = tcf;
   }

   public void addMessageListener(MessageListener listener) 
   {
      listeners.add(listener);
   }

   public Object call() throws JMSException
   {
      // setup our JMS stuff.TopicConnection
      tc = tcf.createConnection();
      try
      {
         TopicSession sess = tc.createSession(false, Session.AUTOACK);
         tc.start();
         while( !Thread.currentThread().isInterrupted() )
         {
            // block for up to 5 seconds.
            Message msg = sess.receiveMessage(5000);
            if( msg != null )
               for (MessageListener listener : listeners)
                  listener.onMessage(msg);
         }
         tc.close();
      }
      finally
      {
         if (tc != null) tc.close();
      }
      return null;
   }
}