---
layout: post
title: BlocChat
feature-img: "img/sample_feature_img.png"
thumbnail-path: "https://d13yacurqjgara.cloudfront.net/users/3217/screenshots/2030974/bloctalk_1x.png"
short-description: BlocChat is a chat service in which multiple users can join text conversations.

---
##  Summary
This project involved producing a chatroom service using the Angular JavaScript framework for the frontend, and Firebase for the database.

##  Explanation
Producing BlocChat is a task I was given via my apprenticeship at Bloc.  The particular tasks I was responsible for were:
- Fabricating the html and css for the chat service page and rooms.  
- Locating, factoring in, and modifying the different chat service modals to function properly for the user.  Implementing the logic for appearance of the modals at the proper time for the user
- Designing a data structure for the database that would organize user names, messages, chat rooms, and defining characteristics of each.  
- Automated Storage and retrieval of data based on user requests and activity.
- Producing code to support all the moving parts of the chat service.  This entails the complex job of factoring the various parts of the service to function within the unique AngularJS structure.


##  Problem
The project presented many obstacles.
The modals were challenging.  Regarding the SetUserName modal, the first modal every user would encounter when visiting the site:
- (1)How would a user set a username?  How would the various usernames be stored and organized?
- (2)How to deny the user access to the site until the user enters a name into the input box?

And a chat service has to have chat rooms.  And those rooms have to have messages.  How would I:

- (3)Create messages?
- (4)Send and store the messages in their different rooms?
- (5)Display a Room's Messages
- (6)Create rooms?
- (7)Navigate to the desired room?
- (8)How does one make this all work within the Angular framework?  One controller for everything?  Several controllers?

##  Solutions
(1)How would a user set, store, and organize a username?

The way this was solved was to place a function in the controller for the SetUserName Modal that set a $cookie which holds each username.  Cookies have been around for over 20 years.  A $cookie is a small piece of data, in this case the username, stored in each user pc's web browser.  This is one of the nice things about cookies - the developer does have to write some code that allows cookies to function within the particular app, but the framework is already established.  In this case, Cookies have two basic parts - the value and a corresponding key.  The value is the username, set by the chatroom user.  But, how would the corresponding key be set?  The answer to this question wasn't immediately available in a search of the $cookies documentation and other sources.  My solution was to use the value, given by the user, to set the key.  I would add an "a" to the value.  If the value was "Mike", the key would be "Mike + a".  So, inside the controller, the basic function to store a $cookie is:
```js
$cookies.put(key, value);
```
My version of this, using the ng-model directive for the user input, would be:
```js
$cookies.put("username + a", this.username);
```
My mentor smartly suggested to streamline this a bit further, and make the key the same as the value.
```js
$cookies.put("username", this.username);
```
(2) How to require the user to supply a username as a condition for granting access?

My way of forcing the user to provide a username was an "if" statement in the controller.
```js
  if((this.username !"") && (this.username!null)){
    $cookies.put("username", this.username);
    }
```
I ran this by my mentor, and he suggested something that was more dry and works nicely:
```js
  if(this.username){
          $cookies.put("username", this.username);
          $uibModalInstance.close();
        }
```

(3)Creating messages.  

This is where things get a little more involved.  Messages are created inside rooms, which are entered by clicking on a room in home.html.  Inside the room, there is a "Create and Send A Message" button that the user can click on to create a message.  An ng-click directive in the html routes to a function in the home controller which opens the modal in which a message is created.

(4)Send and store the messages in their different rooms.

The key function of this message service happens when the user clicks the "OK" button in the message modal. An ng-click directive on the OK button triggers a function in the message modal controller.  This function is linked with the data - the message the user created - here in the controller.  

Here is how the message data gets to the controller: Within the input tag in the modal html (the box where the user types the message text), is an ng-model directive which carries the data, "newMessage", to the modal controller.  In the controller it is linked with the function Message.send,  which carries the data, newMessage, as its parameter.
```  js
Message.send(this.newMessage);
```
This Message.send function is expressed in the Messages.js service, which is "injected into" the message modal controller.  Here it adds the message to the Message array.  This involves adding not just the content of the message, but also the other three characteristics (a.k.a."children", in dataspeak) of the message (roomId, time, and username) in language the computer can understand.  Getting this language right took some trial and error.
```js
Message.send = function(newMessage){
  messages.$add({content: newMessage, roomId: Room.activeroom.$id, sentAt: Date(), username: un });
};
```
The roomId child of the message is a piece of data that is used to arrange the messages in their proper rooms, which brings us to solving another problem:

(5) Display a room's messages.  

This is done by associating the roomId data property (child) of a message with the room.$id property of a room.
```js
Message.getByRoomId = function(room) {
      var newref = ref.orderByChild("roomId").equalTo(room.$id);
      // Filter the messages by their room ID.
      Message.current = $firebaseArray(newref);
      // Reference the database.
      return Message.current;

};
```
And this is some of the html that is used to display the messages.
```js
<ul  ng-repeat="message in home.message.current">
        <b>{{ message.username }}</b> <li>{{ message.content }}</li>  {{ message.sentAt }}</li>
      <!-- <p>ng-repeat{{home.activeRoom.message}}</p> -->
      </ul>
```

(6) Create rooms.

This is done in a way similar to creating a message.  In the home html, there is a button, "Open a New Room", with an ng-click directive, that opens a modal in which the user can type in the name of the new room.  Clicking the OK button links the user-entered data (ng-model in the view => roomName) and the action to be performed on the data (ng-click on OK in the view => controller function{Room.addRoom(this.roomName);}).  

Thus, the new room, roomName, is added to the array of rooms.

(7)Navigating to different rooms.
This is relatively simple.  The different rooms are listed in the view. Clicking on a particular room links, through the home controller, to the setActive function, which then sets the clicked-on room as the Active Room in the view.
```js
<div id="sidebar" >
    <div ng-repeat="room in home.rooms">
      <a ng-click="home.setActive(room)">{{ room.$value }}</a>
    </div>
</div>
```
Clicking on a particular room links, through the home controller, to the setActive function, which then sets the clicked-on room as the Active Room in the view.

(8)How does one make this all work within the Angular framework?  One controller for everything?  Several controllers?

Except for the first modal a user encounters, the modal to set a username, everything in the app is routed through the home controller, HomeCtrl.js, mainly because the user experience starts with the home page view.  The selections available in the home view are the modal to create a room, and the choice of an existing room.  From a particular room, the user can open the message-creating modal, which has its own controller.  The room-creating modal has its own controller also.

The username-setting modal is separate from the home controller, and, by design, restricts access to the rest of the program until the user supplies the username.

So, there is one controller - the home controller - for everything, but there are actually four controllers to make the chat service function properly.  Three controllers have to do with modals - setUserName modal, createRoom modal, and createMessage modal.  Room selection is done through the home controller.


##  Results

For the most part, testing was done by impersonating a user.  I would log in, entering a username (or not enter a username to test the functionality of $cookies and the setUserName modal).  I selected a room and created messages within that room, debugging as I installed new services in the app.  

##  Conclusion
One of the interesting things about creating this chat service is it forced me to make software that conforms to and functions somewhat like human behavior.  By human behavior, I mean something most humans do every day - talk with their friends, family, neighbors, or co-workers.  Different conversations between different numbers of people in different places, all within the framework of the software. I had to think about the user behavior to be accommodated, then mold the view and controllers to it.

I learned quite a bit about Angular in the process.  My mentor taught me the value of, INSTEAD OF throwing in a bunch of code, starting with a console.log message to test for linkage between the view, controller, and, if applicable, a service.  Starting with things that are simple and likely to work is probably a good practice for building anything.  I can say this by experience: writing unorganized code that doesn't address the desired goal, or isn't linked properly to the other parts of the program just doesn't work.

I also learned a lot of the "do's and don't's" in writing code the right way(s) to link data from the view to the actions performed on that data in the controllers and services. One that comes to mind is the necessity to carry the data notation from the html to the controller, usually in the form of an html directive:
```js
ng-repeat="room in home.rooms";
```
Then the data element would be expressed in the controller as a parameter in a function.
```js
Room.setActive = function(room){...};
```

What may be the most important thing I have taken away from this is having to think through how the different parts of the app function, and using this information to make decisions regarding where to place different functions of the app within the overall file structure.
