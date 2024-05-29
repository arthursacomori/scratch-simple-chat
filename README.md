# Simple Scratch Cloud Based Chat

Very straightforward, a chat made into scratch.
There are still a few bugs or uncatched unexpected situations, but the project is made to be simple, patchs will be done in the future.

It is recommended to use Forkphorus or Turbowarp, as you can set a custom cloud-host.

## Scratch Project:
https://scratch.mit.edu/projects/1025418669/

## Forkphorus:
https://forkphorus.github.io/?chost=wss://clouddata.turbowarp.org&username=user#1025418669
- <sub> Change 'username' tag to whatever you want, but it needs to be supported by Scratch </sub>
- <sub> The cloud being used is wss://clouddata.turbowarp.org (Default for Turbowarp) </sub>

## Source Code:
This repository contains the SB3 file for the project, there are no comments into it, but i will give a detailed description of its contents:

### How CLOUD variables work
Scratch cloud variables are a type of variable which is shared among ALL users running the project, has a limit of 256 characters and ONLY supports numbers (based on cloud-host, but scratch website has a default one) (used to be limited to 128)
It means all users can READ and WRITE to the same variable, a change on one client, changes the variable on the other client as well.
<br />All changes made to this variable is registered to a public history (Cloud Monitor), which can be acessed by going to the project page, and clicking on "Cloud Variables" under the project canvas.
<br />It's initial purpose was for projects to have a Leaderboard system, by changing the values of clouds you could register user scores.
However, people found ways to use it for full multiplayer projects on scratch, by building a protocol to send and receive data between connected users instead of just scores.

Elements in the project:

Variables:
| Type  | Variable | Description |
| ------------- | ------------- | ------------- |
| Cloud | join_queue  | Will be set to the encoded nickname of whoever user joins the project  |
| Cloud  | message_source  | Will be set to the encoded nickname of whoever sent the message  |
| Cloud  | message_queue  | Will be set to the encoded message content of whatever message is sent  |
| Global  | count  | Dummy variable, used for improvised FOR loops |
| Global  | count2  | Dummy variable, used for improvised FOR loops |
| Global  | count3  | Dummy variable, used for improvised FOR loops |
| Global  | debug  | Dummy variable, used to show or hide the list `encoded_chat_log` |
| Global  | encoded_message  | When you send a message, it is encoded and stored here first |
| Global  | encoded_username  | When you click the green flag, it encodes your username and stores it here for easier usage |
| Global  | old_joinqueue  | Stores `join_queue` each cycle to test for disparity|
| Global  | old_messagequeue  | Stores `message_queue` each cycle to test for disparity|
| Global  | server_messages  | Stores the decoded result of a message sent to chat, the input is from `message_queue` |
| Global  | server_usernames  | Stores the decoded result of the nickname of a user who just joined/connected to the project, the input is from `join_queue`|
| Global  | server_usernames2  | Stores the decoded result of the nickname of the user who sent a message to chat, the input is from `message_source`|

Lists:
| Type  | Variable | Description |
| ------------- | ------------- | ------------- |
| Global | chat  | Used for the chat itself |
| Global | encoded_chat_log  | The undecoded version of the chat, receives the messages normally, but does not filter anything, used for debug purposes |
| Global | encoder | Core of the project, it is used as reference for the message protocol, each valid character has a index in this list |

Events:
| Event | Description |
| ------------- | ------------- |
| open_message_input | Triggers the inputbox for typing a new message |
| server_thread | Starts the loops that will wait for a new message to arrive, the receiver in the Ping-Pong System |

When the project starts, it clears both `chat` and `encoded_chat_log`
<br />Then it encodes your username by converting each character to its index position on the `encoder`
<br />The protocol was prepared to read up to 999 different characters, this is the set limit, it can be made higher at the cost of less characters per message.
<br />It proceeds by setting up `join_queue` to your encoded username value. This change will trigger events on other connected users project
<br />Now it finally calls up the `server_thread` event, starting 2 different cycles acting like threads

<br /> The first cycle waits until `join_queue` has changed, meaning an new user has connected, and feeds the given encoded username to the decoder
<br /> The second cycle waits until `message_queue` has changed, meaning a new message was sent, it also means that `message_source` probally has changed as well, as it is the register of the user who sent the new message, it feeds both to the decoder

<br />And for the message input system, it is called by pressing **Spacebar** or the visible button on the screen
<br />It opens an scratch default inputbox, feeds the value to a encoder loop like it does to the username, and feeds the result to `message_queue`, also sets `message_source` to your stored encoded username

<br />During those processes, there are "[Client]" prefixes on some messages, they are just a way to say that this specific message was only added to your chat, and was not sent to cloud, meaning only you can see it

<br />This is a simple chat log with its respective encoded log

Normal Chat:
```
[CLIENT] Joining . . .
[CLIENT] Connected
[>] user joined the chat
user: hello world
```
Encoded:
```
-- Joining . . .
-- Connected
>> 023021007020 joined the chat
023021007020: 010007014014017002025017020014006
```
Side to side
| Chat  | Encoded |
| ------------- | ------------- | 
| [CLIENT] Joining . . . | -- Joining . . .|
| [CLIENT] Connected | -- Connected |
| [>] user joined the chat | >> 023021007020 joined the chat |
| user: hello world | 023021007020: 010007014014017002025017020014006 |
