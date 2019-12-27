---
layout: springboot
permalink: /springboot/Messaging/
---
## Messaging
Messaging solutions are designed to solve two types of architecture requirements: messaging from one point in an application to another known point, and messaging from one point in an application to many other unknown points. These patterns are the middleware equivalents of telling somebody something face to face and saying something over a loud speaker to a room of people, respectively.

- **Topic**: If you want messages sent on a message queue to be broadcast to an unknown set of clients who are “listening” for the message (as in the loud speaker analogy), you send the message on a topic. A topic is for the publish-subscribe communication model
- **Queue**: If you want the message sent to a single, known client, then you send it over a queue. A queue is for the point-to-point communication model


## WebSockets
The web was built around the idea that a client’s job is to request data from a server, and a server’s job is to fulfill those requests. This paradigm went unchallenged for a number of years but with the introduction of AJAX, many people started to explore the possibilities of making connections between a client and server bidirectional.

Web applications had grown up a lot and were now consuming more data than ever before. The biggest thing holding them back was the traditional HTTP model of client initiated transactions. To overcome this a number of different strategies were devised to allow servers to push data to the client.

The problem with all of these solutions is that they carry the overhead of HTTP. Every time you make an HTTP request a bunch of headers and cookie data are transferred to the server. This can add up to a reasonably large amount of data that needs to be transferred, which in turn increases latency. If you’re building something like a chat application, reducing latency is crucial to keeping things running smoothly. The worst part of this is that a lot of these headers and cookies aren’t actually needed to fulfil the client’s request.

What we really need is a way of creating a persistent, low latency connection that can support transactions initiated by either the client or server. This is exactly what WebSockets provide.

WebSockets provide a persistent connection between a client and server that both parties can use to start sending data at any time. 

The client establishes a WebSocket connection through a process known as the WebSocket handshake. This process starts with the client sending a regular HTTP request to the server. An Upgrade header is included in this request that informs the server that the client wishes to establish a WebSocket connection. If the server supports the WebSocket protocol, it agrees to the upgrade and communicates this through an Upgrade header in the response. Now that the handshake is complete the initial HTTP connection is replaced by a WebSocket connection that uses the same underlying TCP/IP connection. At this point either party can starting sending data.

With WebSockets you can transfer as much data as you like without incurring the overhead associated with traditional HTTP requests. Data is transferred through a WebSocket as messages, each of which consists of one or more frames containing the data you are sending (the payload).

A WebSocket connection will only end when either the server or the client ends it. This makes WebSocket more preferred than HTTP, because when using HTTP, the connection will be closed once the server responds to the client, and the connection can be opened again by a request from the client only after waiting for some time interval.

Like a web URI relies on HTTP or HTTPS, WebSocket URI uses `ws` or `wss` schemes (for example, ws://www.sample.org/ or wss://www.sample.org/) to communicate. WebSocket’s ws works in a similar way to HTTP by transmitting non-encrypted data over TCP/IP.

WebSocket transmits data in the frame format, and apart from a single bit to distinguish between text and binary data, it is neutral to the message’s content. In order to handle the message’s format, the message needs some extra metadata, and the client and server should agree on an application-layer protocol, known as a subprotocol. The parties choose the subprotocol during the initial handshake. Spring supports Simple Text Orientated Messaging Protocol (STOMP) as a subprotocol—known as STOMP over WebSocket—in a WebSocket communication.

## STOMP
WebSocket simply transforms bytes into messages and transports them between client and server. Those messages still need to be understood by the application itself, which is where subprotocols such as STOMP come into play. When working with WebSocket, typically a subprotocol such as STOMP will be used as a common format between the client and server so both ends know what to expect and react accordingly. STOMP is supported out of the box by the Spring Framework. Out of the box, the Spring Framework supports a simple broker that handles subscription requests and message broadcasting to connected clients in memory. Additionally, WebScoket protocol knows nothing about how to route a frame from the source to the destination, that’s why a WebSocket message does not contain any instructions about the frame routing process. To achieve this two-way communication and route the frames properly, we use STOMP, which is supported by Sprint Boot.

STOMP is an application layer protocol that specifies many commands that help you handle text messages without worrying about which format is being exchanged between the client and the server (the “format” would be the STOMP specification itself). The client and server should specify the subprotocol to be used during the WebSocket connection in the handshake phase. Just keep in mind that you’re not required to use subprotocols, but using them makes your life much easier.

Spring provides an annotation programming model for routing and  processing messages from WebSocket clients. This means that controller  methods can be used to process HTTP requests (when methods are annotated  with @RequestMapping) and can also be used to process WebSocket messages when methods are annotated with @MessageMapping. The complementary operation, sending the result of the method back to the client, is implemented using the @SendTo annotation, which is used to mark a subscription endpoint to which all the potential clients are registered; this way, they are identified as receivers of messages from the server.

Clients can use the SEND or SUBSCRIBE commands to send or subscribe for messages along with a “destination” header that describes what the message is about and who should receive it. This enables a simple publish-subscribe mechanism that can be used to send messages through the broker to other connected clients or to send messages to the server to request that some work be performed.

A STOMP client can do the following:

- Produce messages with a destination header.
- Consume messages by subscribing to a destination for messages that come from the server. In this case, the client sends a SUBSCRIBE frame where the destination header indicates what the client wants to subscribe to.

The Spring Framework implements WebSocket communication with STOMP as the messaging protocol. The `spring-boot-starter-websocket` dependency includes the required libraries for server side implementation of WebSockets.

#### STOMP Broker
To work with STOMP, you need a STOMP broker . Basically, this is the component that keeps track of subscriptions and that broadcasts messages to subscribed users. An in-memory STOMP broker is enabled by declaring two destinations, /queue/ and /topic/ - these are broker destinations, which means that any frame sent to a destination starting with these prefixes will be handled directly by the STOMP broker.

Everything starting with /topic/ is a broker destination and any message sent to this destination is handled by the broker. The broker will receive this message and forward it to every subscribed user at this destination (including the user who sent the message because that user is also subscribed to this destination.

The application destination prefix is /app. Basically, when a frame is sent to a destination that starts with /app, a class annotated with @Controller will handle the frame before forwarding it to the broker. More specifically, a method annotated with @MessageMapping inside the @Controller annotated class will handle it.

Clients are going to connect to the STOMP endpoint using JavaScript (SockJS).

## Workflow
When configuring and using STOMP with Spring WebSocket support, the WebSocket application acts as a broker for all connected clients. The broker can be an in-memory broker or an actual full-blown enterprise solution that supports the STOMP protocol (like RabbitMQ or ActiveMQ). 

To add messaging over the WebSocket protocol, Spring uses Spring Messaging. To be able to receive messages, you need to mark a method in an @Controller with @MessageMapping and tell it from which destination it will receive messages.

The MessageBroker exposes an HTTP endpoint to which clients can use for establishing the WebSocket channel using SockJS. Once the channel is established, the server and client exchange messages over the channel. While the server listens to and responds to client messages, the client can also register a callback for “push” messages from the server. This allows the server to send notifications to the client as and when required without an explicit client request.

Messages are routed to @Controller message-handling methods or to a simple, in-memory broker that keeps track of subscriptions and broadcasts messages to subscribed users.

- Messages whose destination starts with “/app” are routed to message-handling methods (i.e. application work)
- Messages whose destinations start with “/topic” or “/queue” will be routed to the message broker (i.e. broadcasting to other connected clients).

<img src="/assets/img/websocket.png" alt="WebSocket Workflow" class="img-fluid img-thumbnail" style="height: 400px;">

In our messaging architecture, channels decouple receivers and senders. The messaging architecture contains three channels:

- Client Inbound (or Request) channel for request messages sent from the client side
- Client Outbound (or Response) channel for messages sent to the client side
- Broker channel for internal server messages to the broker

Below is the message flow that utilizes a simple broker:

1. When you send a frame through WebSocket over STOMP, the message will first reach the client inbound channel.
2. The message will then be routed to a specific MessageHandler depending on the destination name:
	- **If the name starts with /app**, a class annotated with @Controller will handle the frame before forwarding it to the broker. More specifically, a method annotated with @MessageMapping inside the @Controller annotated class will handle it. Then, from the @MessageMapping annotated method, the message will be forwarded to broker channel, which will send it to the simple broker message handler. Message is then forwarded to the client outbound channel, which will finally send the message to the client.
	- **If the name starts with /topic**, then it will route it directly to the broker.

The “/app” prefix is arbitrary. You can pick any prefix. It’s simply meant to differentiate messages to be routed to message-handling methods to do application work vs messages to be routed to the broker to broadcast to subscribed clients. The “/topic” and “/queue” prefixes depend on the broker in use. In the case of the simple, in-memory broker the prefixes do not have any special meaning; it’s merely a convention that indicates how the destination is used. In the case of using a dedicated broker, most brokers use “/topic” as a prefix for destinations with pub-sub semantics and “/queue” for destinations with point-to-point semantics.

## Chat Application using WebSocket, STOMP and SockJS
Our application will comprise of the following:
- Login Page: The root URL will present a login page to the user.
- Chat Page: Once the login is successful, user will be directed to `/chat` route where he will have the following functionalities available:
	- Displays the number of users online
	- Provides a common chatroom to all connected users
	- Displays messages when a user has joined or left the chatroom

#### Dependencies
Our application will use Thymeleaf, Spring Web, WebSocket, jQuery, BootStrap, SockJS-Client dependencies:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>jquery</artifactId>
	<version>3.4.1</version>
</dependency>
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>bootstrap</artifactId>
	<version>4.3.1</version>
</dependency>
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>sockjs-client</artifactId>
	<version>1.1.2</version>
</dependency>
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>stomp-websocket</artifactId>
	<version>2.3.3-1</version>
</dependency>
```

#### Login Page
 
Since we are using Thymeleaf as our temlate engine, the Login and Chat HTML pages will reside in `src/main/resources/templates` folder.

Our login page consists of a form with the following attributes and fields:
- Form submission will be handled by the `/login` route in our controller.
- `username` field will be sent the controller when the form is submitted.
- We've also linked the SockJS, STOMP, jQuery and BootStrap libraries along with our custom CSS.

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<script src="webjars/jquery/3.4.1/jquery.min.js"></script>
<link href="/webjars/bootstrap/4.3.1/css/bootstrap.min.css" rel="stylesheet">
<link href="https://fonts.googleapis.com/css?family=Solway&display=swap" rel="stylesheet"> 
<link href="main.css" rel="stylesheet">
<script src="/webjars/sockjs-client/1.1.2/sockjs.min.js"></script>
<script src="/webjars/stomp-websocket/2.3.3-1/stomp.min.js"></script>
<title>Chat Room | Login</title>
</head>
<body class="bg-light">
	<div class="container h-100 bg-light">
		<div class="row h-100 w-80 mx-auto">
			<div class="col-md-6 mx-auto text-center my-auto bg-dark p-4 border-0 rounded">
				<h2 class="text-light">Chat Room</h2>
				<p class="p-2 text-warning w-60 rounded" id="error" data-th-text="${error}"></p>
				<form action="/login" method="post">
					<div class="form-group w-100">
						<input type="text" name="username" id="username" placeholder="Username" class="text-center border-bottom-1 border-left-0 border-top-0 border-right-0 my-2 text-light"><br>
						<input type="submit" value="Submit" class="text-center bg-primary text-light border-0 rounded my-2 p-2">
					</div>
				</form>
			</div>
		</div>
	</div>
</body>
</html>
```
<br>

#### Chat Page
Our chat page consists of the following:
- Form to handle logout, form submission is handled by the `/logout` route of our Controller.
- Form to allow user's to enter and submit chat messages.
- A `<div...</div>` tag that displays the number of connected users, and chat messages broadcast to the chatroom.


```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<script src="webjars/jquery/3.4.1/jquery.min.js"></script>
<link href="/webjars/bootstrap/4.3.1/css/bootstrap.min.css" rel="stylesheet">
<link href="https://fonts.googleapis.com/css?family=Solway&display=swap" rel="stylesheet">
<link href="main.css" rel="stylesheet">
<script src="/webjars/sockjs-client/1.1.2/sockjs.min.js"></script>
<script src="/webjars/stomp-websocket/2.3.3-1/stomp.min.js"></script>
<script src="chat.js"></script>
<title>Chat Room</title>
</head>
<body class="bg-light h-100 border">

	<div class="container h-100">
		<div class="row">
			<div class="col-md-12 bg-dark rounded mt-1 d-flex justify-content-between p-1">
				<p class="lead d-inline text-light my-auto ml-2">Chat Room</p>
				<form class="d-inline mt-1" id="logout" action="/logout" method="post">
					<div class="form-group">
						<input type="submit" value="Logout" class="bg-danger border-0 rounded text-light p-2 mr-2">
					</div>
				</form>
			</div>	
		</div>
		<div class="row mt-2  d-flex justify-content-between ">
			<div class="col-md-5 ml-2 bg-secondary rounded">
				<p class="text-warning mt-2 lead">Welcome <span data-th-text="${username}" id="username"></span></p>
				<form id="chatform">
					<div class="form-group p-2">
						<input type="text" id="chatmessage" placeholder="Send Text" class=" w-100 border-left-0 border-top-0 border-right-0 text-light"><br>
						<div class="mt-3">
							<input type="submit" value="Send (Enter)" class="border-0 rounded p-2 bg-success text-light">
							<input type="button" value="Clear" id="clear" class="border-0 rounded p-2 bg-danger text-light ml-5">
						</div>
					</div>
				</form>
			</div>				
			<div class="col-md-6 mr-2 bg-secondary rounded text-light ml-2 h-90" id="chatwall">
				<button class="border-0 rounded bg-primary text-light p-2 mt-2">Online Users: <span id="usercount"></span></button>
				<div id="onlineusers" class="bg-info p-2 mt-2 rounded"></div>
			</div>
		</div>
	</div>
</body>
</html>
```
<br>

#### JavaScript
Our Javascript file resides in `src/main/resources/static` folder and we've linked it to Chat HTML page. All of our JavaScript code is within the document.ready function:

```javascript
var stompClient = null;
$(document).ready(function(){

	//all JavaScript code goes here

})
```
<br>

When the login is successful and the chat.html is loaded, below code will:
- Create a WebSocket connection, our WebSocket endpoint to which clients can connect is `ws-chat`
- Once WebSocket connenction is created:
	- Subscribe to `/topic/public` - this is where all chat, login and logout messages will be broadcast.
	- We are also sending the username captured from the login page to `/app/chat.register` 

```javascript
var socket = new SockJS("/ws-chat");
stompClient= Stomp.over(socket);
stompClient.connect({},onConnected);
$("#chatform").on("submit",onSend);

function onConnected(){
	stompClient.subscribe("/topic/public",onMessageReceived);
	stompClient.send('/app/chat.register',{},JSON.stringify({sender:$("#username").text(),type:'JOIN'}));
}
```
<br>

Below function is triggered when a user sends a chat message:

```javascript
function onSend(){
	var chatMessage = {
		sender:$("#username").text(),
		content: $("#chatmessage").val(),
		type:'CHAT'
	};
	$("#chatmessage").val('');
	stompClient.send('/app/chat.send',{},JSON.stringify(chatMessage));	
}
```
<br>

Below function handles the messages received from the server:

```javascript
function onMessageReceived(message){
		 var message = JSON.parse(message.body);
		 var userList="";
		 for(i=0;i<message.onlineUsers.length;i++){
			 userList += "<p>" + message.onlineUsers[i] + "</p>";
		 }
		 console.log(message);
		 if(message.type == 'JOIN'){
			 message.content = message.sender + ' joined';
			 $("#usercount").text(message.onlineUsers.length);
			 $("#onlineusers").html(userList);
			console.log(userList);
			 $("#chatwall").append("<p class='text-warning rounded mt-1'>" + message.content + "</p>");
		 } else if(message.type == 'CHAT'){
			 $("#chatwall").append("<p class='bg-info rounded mt-1 p-2'>[" + message.sender +"]: " + message.content + "</p>");
		 } else if (message.type == 'LEAVE'){
			 $("#usercount").text(message.onlineUsers.length);
			 $("#onlineusers").html(userList);
			 $("#chatwall").append("<p class='text-light rounded mt-1'>" + message.content + "</p>");
		 }
		
	}
```
<br>

Below function handles the logout feature:

```javascript
function onLogout(){
		stompClient.send('/app/chat.leave',{},JSON.stringify({sender:$("#username").text(),type:'LEAVE'}));
		stompClient.disconnect()
		
		$.ajax({
			type:'POST',
			url:$("#logout").attr('action'),
			success:function(){
				window.location.href="/"
			}
		})
		
	}
```
<br>

Additionally, below functions handled the following:
- Prevent default behavior of chat form submission
- Toggle the list of online users
- Clear the input field when the "Clear" button is clicked
- Handle user logout when Logout button is clicked

```javascript
$("form").on("submit",function(e){
		e.preventDefault();
	})
	
	//Toggle the list of online users
	$("#onlineusers").hide();
	
	$("button").on("click",function(){
		$("#onlineusers").toggle();
	})
		
	// Function clears the input typed in the chat window
	$("#clear").on("click",function(){
		$("#chatmessage").val("");
	})
	
	//Handle logout
	$("#logout").on("submit", onLogout)
```
<br>

#### Chat Message Model
We are modeling the chat message as a Java class (ChatMessage.java) which has the following fields:
- sender represents the username.
- content represents the actual chat message.
- MessageType is an Enum that represents if a user has joined, left or chatting.
- We also have an ArrayList where we are storing the list of all connected users. Note, this is a Class field.
- Additionally, we have the standard getter and setter methods for these fields.

```java
import java.util.ArrayList;

public class ChatMessage {
	private String sender;
	private String content;
	private MessageType type;
	private static ArrayList<String> onlineUsers = new ArrayList<String>();

	public ArrayList<String> getOnlineUsers() {
		return onlineUsers;
	}

	public static void addUser(String user) {
		onlineUsers.add(user);
	}

	public static void removeUser(String user) {
		onlineUsers.remove(user);
	}

	public enum MessageType{CHAT,LEAVE,JOIN}

	//Constructor
	public ChatMessage() {}

	//Getters and Setters
	public String getSender() {
		return sender;
	}

	public void setSender(String sender) {
		this.sender = sender;
	}

	public String getContent() {
		return content;
	}

	public void setContent(String content) {
		this.content = content;
	}

	public MessageType getType() {
		return type;
	}

	public void setType(MessageType type) {
		this.type = type;
	}
}
```
<br>

#### Configuration
Below is our WebSocketConfig.java:

- **@EnableWebSocketMessageBroker** enables WebSocket message handling, backed by a message broker  using a higher-level messaging sub-protocol (like STOMP).
- **WebSocketMessageBrokerConfigurer** interface defines methods for configuring message handling with simple messaging protocols (e.g. STOMP) from WebSocket clients. Here, we are implementing 2 methods from this interface:
	- **registerStompEndpoints**: This method registers the /ws-chat endpoint, enabling SockJS fallback options so that alternate transports may be used if WebSocket is not available. The SockJS client will attempt to connect to this endpoint and use the best transport available (websocket, xhr-streaming, xhr-polling, etc).
	- **configureMessageBroker**: It configures the message broker. It starts by calling enableSimpleBroker() to enable a simple memory-based message broker to carry the greeting messages back to the client on destinations prefixed with "/topic". It also designates the "/app" prefix for messages that are bound for @MessageMapping-annotated methods.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
	@Override
	public void registerStompEndpoints(StompEndpointRegistry registry) {
		registry.addEndpoint("/ws-chat").withSockJS();
	}
	@Override
	public void configureMessageBroker(MessageBrokerRegistry config) {
		config.enableSimpleBroker("/topic");
		config.setApplicationDestinationPrefixes("/app");
	} 
}
```
<br>

#### Controller

- **@MessageMapping()**: This annotation ensures that if a message is sent to the specified destination (for example chat.register) then the underlying handler method (register()) is called.
- **@SendTo()**: The response returned by the handler method is broadcast to all subscribers to the destination in the @SendTo annotation.


```java
import javax.servlet.http.HttpSession;

import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

@Controller
public class WebSocketChatServer {
	
	//Root route displays the login page
	@RequestMapping("/")
	public String home() {
		return "login";
	}
	
	// Route handles server side validation for empty username, redirects to /chat
	@RequestMapping(value="/login",method=RequestMethod.POST)
	public String login(@RequestParam(value="username") String username, HttpSession session,RedirectAttributes redirectAttributes) {
		if(username != null && !username.isEmpty()) {
			session.setAttribute("username", username);
			return "redirect:/chat";
		}
		redirectAttributes.addFlashAttribute("error","Please enter a username.");
		return "redirect:/";
	}
	
	//Route displays the chat form upon successful login
	@RequestMapping("/chat")
	public String chat(HttpSession session, Model model,RedirectAttributes redirectAttributes) {
		
		if(session.getAttribute("username")!=null) {
			String username = (String) session.getAttribute("username");
			model.addAttribute("username",username);
			return "chat";
		}
		
		redirectAttributes.addFlashAttribute("error","Please login first.");
		return "redirect:/";
				
	}
	
	// User join
	@MessageMapping("/chat.register")
	@SendTo("/topic/public")
	public ChatMessage register(@Payload ChatMessage chatMessage,SimpMessageHeaderAccessor headerAccessor) {
		headerAccessor.getSessionAttributes().put("username", chatMessage.getSender());
		ChatMessage.addUser(chatMessage.getSender());
		return chatMessage;
	}
	
	// User sends chat message
	@MessageMapping("/chat.send")
	@SendTo("/topic/public")
	public ChatMessage sendMessage(@Payload ChatMessage chatMessage) {
		return chatMessage;
	}
	
	// User leaves the chatroom
	@MessageMapping("/chat.leave")
	@SendTo("/topic/public")
	public ChatMessage leave(@Payload ChatMessage chatMessage,SimpMessageHeaderAccessor headerAccessor) {
		headerAccessor.removeHeader("username");
		chatMessage.setContent(chatMessage.getSender() + " has left the chat.");
		ChatMessage.removeUser(chatMessage.getSender());
		return chatMessage;
	}
	
	@RequestMapping("/logout")
	public String logout(HttpSession session) {
		session.invalidate();
		return "redirect:/";
	}
}
```








