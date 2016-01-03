Socket.io
Perhaps the most tantalizing thing about Node.js is the ability to use web sockets.  Up to this point, we have been communicating with our web servers with only HTTP requests and responses: the client makes a request and the server processes the request and prepares and sends a response.  Same old story, no matter what language.  This method clearly works; 99.9% of web traffic is done using these standard cycles, but what if we wanted to do something like build a chat room or a multiplayer game?  Think about the concept of a chatroom:

You log on to the chatroom and have the ability to chat with other users.
Every time you post a message, everyone in the chatroom gets your message.
Every time a different user posts, their message appears in your screen.
If a user disconnects, you get a message saying they've left.
ALL OF THESE STEPS MUST ALSO HAPPEN WITHOUT REFRESHING!
Try mapping out this process using HTTP requests, or even AJAX.  You'll find the standard request/response lifecycle just can't quite get it done.  Why?  Because in steps 3 and 4 above, you'll realize there is a need for the server to be able to instantiate an interaction with the client, which is backwards compared to our standard model.  Also, it seems like isolated events just aren't going to cut it, refreshing the page each time something happens just can't do.  We need a persistent connection. That is, we need our client and server to always be connected instead of just sending requests back and forth.  We don't want to send morse code; we want something like a phone call.  By 'like a phone call' we mean the ability for the client and the server to engage each other at the same time.  Even if it's annoying, when you're on a phone call, data can pass both ways at all times: I can talk and transmit data to you while you talk and transmit data to me.  This is what's called non-blocking communication.

Enter sockets
With Node.js, making this connection is super easy.  We can set up a real-time connection between client and server that is always listening for events from the other party.  This is called a web-socket connection.  A very important facet of this connection is that is it not performed using HTTP.  The other important facet is that socket connections are event-driven.

Events
The best way, hands down, to learn about sockets is to learn about the logic you'll need to build applications with sockets before you ever touch the code.  The nice thing about Node.js is that the syntax is easy; it's the thought process that is most difficult to develop.  Just like jQuery, sockets are event-driven.  That means that the code we write for sockets will happen only as the events we tie to the code are triggered.  How do we trigger these events? Great question!

With sockets, as you would imagine, we will be writing both client-side and server-side code.  Both sides will have the ability to emit events and listen for them. Let's dive into those terms:

Listening
A socket event is very much like a jQuery event: a click, a hover, a form submission, etc.  The difference is that in jQuery, we were given a particular set of events and wrote code for them. In socket-land, we write our own events, ie, all events are user-defined!  This is how we are able to have our Node servers react just the way we want them, because we tell them exactly what events to fire on.  Both the server and the client can listen for events. This is super important!

Emitting
Emitting an event is the act of signaling to either a client or server: "HEY!  I'm doing something!".  If my server is waiting for a chatroom client to emit a "new_text" event, when the necessary actions are completed by the client, the event "new_text" will be emitted by the client to the server.  If the client is listening for a "new_user_entered_room" event, when the server gets a new user added to the chatroom, it will emit a "new_user_entered_room" event. Notice that the name of an event a client or server listens for must match exactly the name of the event that the opposite party emits! No exceptions!!  Another very important rule:

Clients emit to the server, not other clients!

Let's go back to the chatroom example: if I type some text into my chat box and I enter it, my text should appear in the screens of the other users (as well as mine).  What is the process that's happening here? Let's walk through this:

I enter text and submit it. This should trigger a client-side emit: "new_text"
The server is listening for an event called "new_text", and it is triggered by the client. This gets triggered.
When the server gets the "new_text" event, it in turn is programmed to emit to all the clients an event called "updated_chat" and pass the new chat text to the clients.
The clients are all listening for an event called "updated_chat" and when they get that event, the new message appends to the chat box on their screen.
Notice that we only really are talking abou two things happening: the user enters text, and all the chatrooms get updated. But we have four (4) steps above! This is what we like to refer to as duplicity:

For any one specific socket action, there will be two associated steps
This should make crystal-clear sense. If we want to do ONE action, we have to have TWO steps:

1) A listener for the event is present
2) An emit of the listened-for event to trigger the action
The duplicity stems from the fact that if we have n actions we want to account for with our sockets, we must have 2n events between our listeners and emits combined!

Different types of emits (server only)
When it comes to emits, the client only has one option: emit to the server.  The server has a little bit more choice.  There are three different types of emits from the server side.  At first you might think "what? This is insane!" Woah there, nelly.  We promise that all three events are useful.

Emit
The first type of emit is just called an emit. Boring, yes, but necessary.  The standard emit is used after an event is trigged on the server.  That is, after the client emits a particular event that the server is listening for.  Within the code we write to facilitate the server's response, we will have the option to emit back to the particular client that triggered the event.  That means: we are able to talk back directly to the client that triggered our event. In our chat room, what this means is that when a user first connects via the socket connection, we could capture that event on the server and emit back to that particular user and only that particular user!  Read this again if this is not absolutely clear!

Broadcast
In our chat room, after we emit from the server to the particular client that just joined, we might want to emit to all the other users that a new user joined the chat room.  Cue our second type of emit.  The event of sending out an event to all of the sockets except for the socket connection that triggered the event is called a broadcast.  Imagine, when you join a chat room, you don't really need to be informed of the fact you joined the chatroom.  But everyone else does.  That is why the Node server has the ability to broadcast an event to everyone except the person who triggered it!

Full broadcast
The full broadcast goes to every connected socket.  Period.  Any client who has a connection to the server via websockets will get the event emitted by a full broadcast.  Nuff said.

