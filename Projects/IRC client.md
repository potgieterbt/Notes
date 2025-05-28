Servers connect to each other in a distributed computation model.

Clients are anything connected to a server other than another server.
* Each client is differentiated by a unique nickname with a max length of 9 characters(See grammar rules to see what is allowed as a nickname).
* In addition to nickname, servers must have following information about all clients:
	1. Real name of host that client is running on.
	2. Username of the client on that host.
	3. The server to which the client is connected.

* Operators:
	* To allow for order, special class of clients (Operators) is allowed to preform maintenance functions.
	* Needs to be able to perform basic networking tasks such as disconnecting and reconnecting servers (SQUIT and CONNECT).
	* More controversial power is being able to remove a user from connected network by ‘force’ (KILL).

Channels
* Named group of one or more clients which all receive messages addressed to channel.
* Channel is implicitly when first client joins and ceases to exist when last client leaves it'.
* While the channel exists the members can reference it by using the name of the channel.
* Channel names are strings (beginning with an & or # character) of length up to 200 characters. Only other requirement is that channel name cannot contain any spaces, Control+G (^G) or a comma.
* 2 types of channels allowed by protocol.
	* One is a distributed channel which is known to all the servers that are connected to the network.
	* The other, only clients on the server where it exists may join.
	* These are marked by the first character being a only clients on server where it exists may join it. Distinguished by a leading &.
* There are various channel modes to alter the characteristics of individual channels.
* To create new channel or become part of an existing channel, a user is required to JOIN the channel. If the channel does not exist prior to joining, the channel is created and the creating user becomes a channel operator. If the channel already exists, whether or not request to JOIN channel is honoured depends on the current modes.
* The protocol allows users to be part of several channels at once, but limit to 10 channels is recommended as being ample for users.
* **How to deal with a disjoint between servers:** If the IRC network becomes disjoint because of split between 2 servers, channel on each side is only composed of clients which are connected to servers on the respective sides of the split, possibly ceasing to exist on one side of the split. When split is healed, connecting servers announce to each other who they think is in the channel and the channel mode. If channel exists on both sides, JOINs and MODEs are interpreted in inclusive manner so both sides of new connection will agree which clients are in the channel and what modes the channel has.

* Channel Operators(chop or chanop):
	* Considered to ‘own’ the channel and are endowed with certain powers to enable then to control channel.
	* As owners of the channel, operators do not need reasons for their actions, although if their actions are generally antisocial or otherwise abusive, it might be reasonable to ask an IRC operator to intervene or for the user to leave and go elsewhere and form own channel.
	* #### Commands:
		* KICK - Eject a client from the channel
		* MODE - Change channel’s mode
		* INVITE - Invite client to and invite-only channel (mode +i)
		* TOPIC - Change channel topic in mode +t channel.
	* A channel operator is identified by the @ symbol next to their nickname whenever it is associated with a channel (i.e. replies to the NAMES, WHO and WHOIS commands).

### Server Protocol:
* 3 basic services required for real-time conferencing (IRC-ARCH): Client locator (IRC-CLIENT), Message relaying (This document) and channel hosting and management (IRC-CHAN).
* IRC Protocol defines fairly distributed model, each server maintains a global state database about whole IRC network. Database is the, in theory, identical on all servers.
	* Servers: Each server is uniquely identified by their name which has a maximum length of 63 characters. Each server is typically known by all other servers
		* However it is possible to define a hostmask to group servers together according to their name.
		* Inside hostmasked area, all servers have a name which matches hostmask, and any other server with a name matching the hostmask SHALL NOT be connected to the IRC network outside the hostmasked area.
		* Servers which are outside the hostmasked area do not have any knowledge of the individual servers inside the hostmasked area, instead presented with virtual server which has hastmask for name.
	* Clients: For each client, all servers MUST have the following information: a network unique identifier (format depends on type of client) and server which the client is connected.
		* Users: Each user is identified by a unique nickname having a maximum of 9 characters. All servers MUST have the name of the host the user is running on, username of the user on that host and the server which the client is connected.
		* Services: Each service is identified from other services by service name composed of nickname and server name, nickname has a maximum length of 9 characters and the server name used to compose the service name is the name of the server which the service is connected. In addition this service name all servers MUST know the service type.
			* Services differ from users by format of the identifier, but more importantly services and users don’t have the same type of access to the server: services can request part or all of the global state information that the server maintains, but more restricted set of commands available to them and are not allowed to join channels. 
			* Services are not usually subject to the Flood control mechanism.
	* Channels: Like services, channels have a scope (IRC-CHAN) and are not necessarily know to all servers. When channel existence is know to server, server MUST keep track of the channel members and channel modes.
	* ## Server to Server - Specifications:
		* 

### IRC Server Specification:
* Client to server connections are more restricted than server to server connections because clients are considered untrustworthy by default.
* Character Codes:
	* No specific character set is specified. Protocol is based on a set of codes which are composed of 8 bits. Each message may be composed of any number of these octets, however, some octet values are used for control codes which act as message delimiters.
	* Even being an 8-bit protocol, delimiters and keywords are such that protocol is mostly usable from US-ASCII terminal and telnet connection.
	* Characters {}|^ are considered lowercase equivalents of \[]\\~. This is a critical issue when determining equivalence of two nicknames or channel names.
* Messages:
	* 

# Client Protocol:
* Labels:
	* Servers are uniquely identified by their name, maximum of 63 characters.
	* Clients - For each client, all servers must know: a netwide identifier (format depends on client) and server which introduced the client.
	* Users - each user is identified by unique nickname, maximum of 9 characters.
		* Operators - Special class of users allowed to perform general maintenance functions on network. Should be able to perform basic network tasks such as disconnecting and reconnecting servers as needed. Also are allowed to remove a user from connected network by force.
	* Services - Each service is distinguished from other services by service name composed of a nickname and a server name. Nickname has maximum of 9 characters.
	* Channels - Names are strings (beginning with ‘&’, ‘#’, ‘+’, or ‘!’ character) of length up to 50 characters. Only other restriction on channel name is that it cannot include any spaces (‘ ’), control G(‘^G’ or ASCII 7), comma (‘,’). Channel names are case insensitive.
		* Space is used as parameter separator and command is used as a list item separator by protocol. Colon (‘:’) can also be used as a delimiter for channel mask.
	* Each prefix characterizes a different channel type. Definition of channel types is not relevant to client-server protocol.
* ## Client to Server Specifications:
	* Character Codes:
		* No specific character set specified. Protocol based on set of codes composed of 8-bits(octet). Each message can be composed of any of these octets, however some octets are used for control codes, act as message delimiters. Mostly usable from US-ASCII terminal and telnet connection.
		* Characters \{\}\|\^ are considered to be lower case equivalents of the characters \[\]\\\~ respectively. Critical issue when determining the equivalence of two nicknames or channel names.
	* Messages:
		* Servers and clients send each other messages, may or may not generate a reply. If message contains a valid command client should expect a reply as specified but it is not advised to wait forever for the reply; client to server and server to server communication is essentially asynchronous by nature.
			* Each IRC message may consist of up to three main parts: prefix(optional), command and the command parameters(maximum of 15). Prefix, command and all parameters are separated by one ASCII space character each.
			* Presence of prefix is indicated with single leading colon(‘:‘, ASCII 0x3b), must be first character of the message. must be no gap between the colon and the prefix. Prefix is used by servers to indicate the true origin of the message. If prefix if missing from the message, it is assumed to have originated from the connection from which it was received from. Clients should not use prefix when sending a message; if they use one, only valid prefix is the registered nickname associated with client.
			* Command must either be valid IRC or a 3 digit number represented in ASCII text.
			* IRC messages are always lines of characters terminated with CR-LF(\r\n) pair and cannot exceed 512 characters in length, counting all characters including trailing CR-LF, therefore, 510 maximum characters allowed for command and parameters. No provision for continuation of message lines.
				* Message format in Augmented BNF - Protocol messages must be extracted from the contiguous stream of octets. Current solution is to use CR-LF as message separators. Empty messages are ignored, permits use of sequence CR-LF between messages without extra problems. 
				* Extracted message is parse into components: prefix, command and list of parameters.
				* Augmented BNF for this is:
					* message = \[“:“ prefix SPACE\] command \[params\] crlf
					* prefix = servername / (nickname \[ \[ “!” user \] “@” host \])
					* mmand = 1 \* letter / 3 * \* digit
					* params = *14( SPACE middle ) \[ SPACE \[ “:” trailing \] \]
							 =/ 14( SPACE middle ) \[ SPACE \[ “:” trailing \] \]
					* nospcrlfcl = %x01-09 / %x0B-0C / %x0E-1F / %x21-39 / %x3B-FF; any octet except NUL, CR, LF, “ ” and “:”
					* middle = nospcrlfcl \*(“:” / nospcrlfcl)
					* trailing = \*(“:” / “ “ / nospcrlfcl)
					* SPACE = %x20
					* crlf = %x0D %x0A
				* ### Check [Client Protocol](https://www.rfc-editor.org/rfc/pdfrfc/rfc2812.txt.pdf) page 7 for info on target parameters.
				* Numeric replies
					* Most messages sent to the server generate a reply of some kind. Most common reply is the numeric reply, used for both errors and normal replies. Must be sent as one message consisting of the sender prefix, three-digit numeric and target of reply. Numeric reply is not allowed to originate from a client. In all other respects, numeric reply is like normal messages, except that keyword is made up of 3 numeric digits rather than string of letters. (List of replies in section 5)
				* Wildcard expressions
					* When wildcards are allowed in a string, it is referred to as a mask. For string matching purposed the protocol allows use of two special characters: ‘?’ to match on and only one character, and ‘\*’ to match any number of any characters. These 2 characters can be escaped using ‘\’.
						* Syntax:
							* [PDF](https://www.rfc-editor.org/rfc/pdfrfc/rfc2812.txt.pdf) page 9
		* ## Message Details:
			* The following are descriptions of each message recognised by IRC server and client. All commands must be implemented by server for protocol.
			* Where the reply ERR_NOSUCHSERVER is returned, means that the target of the message could not be found. Server must not send any other replies after this message for that command.
			* Server to which a client is connected is required to parse the complete message and return any appropriate errors.
			* If multiple parameters are presented, each must be checked for validity and appropriate responses must be sent back to client. In the case of incorrect messages which use parameter lists with comma as an item separator, reply must be sent for each item.
			* Connection Registration:
				* Commands are used to register a connection with an IRC server as a user as well as correctly disconnect.
				* “PASS” command is not required for a client connection to be registered, but must precede latter of teh NICK/USER combination (for user connection) or the SERVICE command (service connection). Recommended order for a client to register is as follows:
					1. Pass message
					2. Nick message
					3. User message
						OR
					2. Service message
				* Upon success, client will receive an RPL_WELCOME (users) or RPL_YOURESERVICE (services) message indicating the connection is now registered and know to teh entire IRC network. Reply message must contain the full client identifier upon which was registered.
					* Password message:
						Command    : PASS
						Parameters : \<password\>
						
						The PASS command is used to set a ‘connection password’. Optional password can and must be set before any attempt to register the connection is made. Currently requires that user send a PASS command before sending the NICK/USER combination.
						
						Numeric Replies:
							ERR_NEEDMOREPARAMS
							ERR_ALREADYREGISTERED

					* Nick message
						Command    : NICK
						Parameters : \<nickname\>
						
						NICK command used to give user a nickname or change the existing one.
						
						Numeric Replies:
							ERR_NONICKNAMEGIVEN
							ERR_ERRONEUSNICKNAME
							ERR_NICKNAMEINUSE
							ERR_NICKCOLLISION
							ERR_UNAVAULRESOURCE
							ERR_RESTRICTED
							
					* user message
						Command    : USER
						Parameters : \<user\> \<mode\> \<unused\> \<realname\>
						
						
						
						Numeric Replies:
							ERR_NEEDMOREPARAMS
							ERR_ALREADYREGISTERED
							
					* Oper message
						Command    : PASS
						Parameters : \<password\>
						
						The PASS command is used to set a ‘connection password’. Optional password can and must be set before any attempt to register the connection is made. Currently requires that user send a PASS command before sending the NICK/USER combination.
						
						Numeric Replies:
							ERR_NEEDMOREPARAMS
							ERR_ALREADYREGISTERED
					* User mode message
						Command    : PASS
						Parameters : \<password\>
						
						The PASS command is used to set a ‘connection password’. Optional password can and must be set before any attempt to register the connection is made. Currently requires that user send a PASS command before sending the NICK/USER combination.
						
						Numeric Replies:
							ERR_NEEDMOREPARAMS
							ERR_ALREADYREGISTERED
					* Service message
						Command    : PASS
						Parameters : \<password\>
						
						The PASS command is used to set a ‘connection password’. Optional password can and must be set before any attempt to register the connection is made. Currently requires that user send a PASS command before sending the NICK/USER combination.
						
						Numeric Replies:
							ERR_NEEDMOREPARAMS
							ERR_ALREADYREGISTERED
					* Quit
						Command    : PASS
						Parameters : \<password\>
						
						The PASS command is used to set a ‘connection password’. Optional password can and must be set before any attempt to register the connection is made. Currently requires that user send a PASS command before sending the NICK/USER combination.
						
						Numeric Replies:
							ERR_NEEDMOREPARAMS
							ERR_ALREADYREGISTERED
					* Squit
						Command    : PASS
						Parameters : \<password\>
						
						The PASS command is used to set a ‘connection password’. Optional password can and must be set before any attempt to register the connection is made. Currently requires that user send a PASS command before sending the NICK/USER combination.
						
						Numeric Replies:
							ERR_NEEDMOREPARAMS
							ERR_ALREADYREGISTERED
			* Channel operations
				* Join message
				* Part message
				* Channel mode message
				* Topic message
				* Names message
				* List message
				* Invite message
				* Kick message
			* Sending messages
				* Private messages
				* Notice
			* Server Queries and commands
				* Motd message
				* Lusers message
				* Version message
				* Stats message
				* Links message
				* Time message
				* Connect message
				* Trace message
				* Admin command
				* Info command
			* Service Query and commands
				* Servlist message
				* Squery
			* User based queries
				* Who query
				* Whois query
				* Whowas
				* Miscellaneous messages
					* Kill message
					* Ping message
					* Pong message
					* Error
	* #### Optional Features
		* Away
		* Rehash message
		* Die message
		* Restart message
		* Summon message
		* Users
		* Operwall message
		* Userhost messages
		* Ison message
	* Replies
		* Command responses
		* Error Replies
		* Reserved numerics
		* 



## A much better solution is to simply parse the message. The IRC specs in RFC1459 and RFC2812 give some pretty useful hints here. My advice from experience is to split on " :" (space then colon) - this is the last parameter of the message, then split the first half by spaces. If the first entry in your list starts with a space, split it again by ! and @ to get the parts of the nickname/username/hostname tuple. Follow this method, and you'll have the base to a much more robust and extensible parser than one you could ever build using regular expressions.