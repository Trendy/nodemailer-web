+++
date = "2017-01-21T00:40:56+02:00"
toc = true
prev = "/usage/using-gmail/"
next = "/message/"
weight = 9
title = "Delivering bulk mail"

+++

Here are some tips how to handle bulk mail, for example if you need to send 10 million messages at once.

1. **Use a dedicated delivery provider**. Do not use services that offer SMTP as a sideline or for free (that's Gmail or the SMTP of your homepage hosting company) to send bulk mail – you'll hit all the hard limits immediately or get labelled as spammer. Basically you get what you pay for and if you pay zero then your deliverability is near zero as well. Email might seem free but it is only free to a certain amount and that amount certainly does not include 10 million emails in a short period of time.
2. **Use a dedicated queue manager,** for example [RabbitMQ](http://www.rabbitmq.com/) for queueing the emails. Nodemailer creates a callback function with related scopes etc. for every message so it might be hard on memory if you pile up the data for 10 million messages at once. Better to take the data from a queue when there's a free spot in the connection pool (previously sent message returns its callback).
3. **Use pooled SMTP** by setting _pool_ option to _true_ (assuming you always send using the same credentials). You do not want to have the overhead of creating a new connection and doing the SMTP handshake dance for every single email. Pooled connections make it possible to bring this overhead to a minimum.
4. **Set _maxMessages_ option to _Infinity_** for the pooled SMTP transport. Dedicated SMTP providers happily accept all your emails as long you are paying for these, so no need to disconnect in the middle if everything is going smoothly. The default value is 100 which means that once a connection is used to send 100 messages it is removed from the pool and a new connection is created.
5. **Set _maxConnections_ to whatever your system can handle.** There might be limits to this on the receiving side, so do not set it to _Infinity_, even 20 is probably much better than the default 5\. A larger number means a larger amount of messages are sent in parallel.
6. **Use file paths not URLs for attachments.** If you are reading the same file from the disk several million times, the contents for the file probably get cached somewhere between your app and the physical hard disk, so you get your files back quicker (assuming you send the same attachment to all recipients). There is nothing like this for URLs – every new message makes a fresh HTTP fetch to receive the file from the server.
7. If the SMTP service accepts HTTP API as well you still might prefer SMTP and not the HTTP API as HTTP introduces additional overhead. You probably want to use HTTP over SMTP if the HTTP API is bulk aware – you send a message template and the list of 10 million recipients and the service compiles this information into emails itself, you can't beat this with SMTP.
