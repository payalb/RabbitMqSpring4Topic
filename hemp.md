The routing algorithm behind a direct exchange is simple - a message goes to the queues whose binding key exactly matches the routing key of the message.

Although using the direct exchange improved our system, it still has limitations - it can't do routing based on multiple criteria.

Messages sent to a topic exchange can't have an arbitrary routing_key - it must be a list of words, delimited by dots. The words can be anything, but usually they specify some features connected to the message. A few valid routing key examples: "stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit". There can be as many words in the routing key as you like, up to the limit of 255 bytes.
The binding key must also be in the same form. The logic behind the topic exchange is similar to a direct one - a message sent with a particular routing key will be delivered to all the queues that are bound with a matching binding key. However there are two important special cases for binding keys:
* (star) can substitute for exactly one word.
# (hash) can substitute for zero or more words.

 @Bean
        public Binding binding1a(TopicExchange topic, 
            Queue autoDeleteQueue1) {
            return BindingBuilder.bind(autoDeleteQueue1)
                .to(topic)
                .with("*.orange.*");
        }

        @Bean
        public Binding binding1b(TopicExchange topic, 
            Queue autoDeleteQueue1) {
            return BindingBuilder.bind(autoDeleteQueue1)
                .to(topic)
                .with("*.*.rabbit");
        }
 @Bean
        public Binding binding2a(TopicExchange topic, 
            Queue autoDeleteQueue2) {
            return BindingBuilder.bind(autoDeleteQueue2)
                .to(topic)
                .with("lazy.#");
        }
These bindings can be summarised as:
Q1 is interested in all the orange animals.
Q2 wants to hear everything about rabbits, and everything about lazy animals.
A message with a routing key set to "quick.orange.rabbit" will be delivered to both queues. Message "lazy.orange.elephant" also will go to both of them. On the other hand "quick.orange.fox" will only go to the first queue, and "lazy.brown.fox" only to the second. "lazy.pink.rabbit" will be delivered to the second queue only once, even though it matches two bindings. "quick.brown.fox" doesn't match any binding so it will be discarded.
What happens if we break our contract and send a message with one or four words, like "orange" or "quick.orange.male.rabbit"? Well, these messages won't match any bindings and will be lost.
On the other hand "lazy.orange.male.rabbit", even though it has four words, will match the last binding and will be delivered to the second queue.
