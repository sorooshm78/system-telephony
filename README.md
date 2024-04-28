# System Telephony

# SIP
## What is a SIP Header?
A SIP Header is a component of a SIP message that is used to convey information about the SIP message. Including the correct SIP Header and correctly formatting these SIP Headers is critical to ensure that requests are successfully routed to the right recipients. We'll try and run through some of the most common SIP Headers in this blogpost.

## Headers for the Standard SIP Call
Usually, an  INVITE message initiates a session—essentially a phone call—on the SIP protocol. While there are many other SIP headers, the nine outlined below supply the minimum required information to initiate  a call over a SIP trunking network. A  BYE request is used to terminate calls.

## To
The To header specifies the recipient of the call.
To: Indicates the intended recipient of the SIP request or response

Format
```
To: “(name)” <sip: (user)@(domain)>
```

## Via
The  Via header identifies a call’s path with the protocol name, protocol version, transport type,  user agent client (UAC), the protocol port for the request and a branch parameter which serves as a unique identifier for each  SIP transaction . The Via header routes SIP responses to the correct device, similar to a return address on a package. If a SIP request is routed through multiple devices, each UAC adds its own VIA header to the request before sending it on.
Via: Provides information about the routing path and transport protocol used in the SIP message

Format
```
Via: SIP/(protocol version)/(transport type) (UAC):(protocol port);branch=(branch number) 
```

## Call-ID
The Call-ID SIP Header creates a globally unique identifier for the call. To ensure that each Call-ID identifier is globally unique, a random number is generated (which often looks like this: f_169eac17a017b0a4e0adfa8_I), and the sender’s IP address is appended to this number. This guarantees that the Call-ID number will be globally unique, since no two devices will have the same IP address.
Call-ID: Generated by the sender, it’s a unique identifier that allows the related SIP messages to be grouped as part of the same call

Format
```
Call-ID: (generated number)@(ip\_address) 
```

## Contact
The Contact header identifies the most direct route for sending future requests to the requesting device. The Contact header specifies a caller domain name or IP address and a transport type.

Format
```
Contact: sip:(user)@(domain);transport=(transport type) 
```

## From
The From header specifies who the call is coming from.
From: Specifies the sender of the SIP request or response

Format
```
From: “(name)” <sip: (user)@(domain)> 
```

## Content-Length
The Content-Length header specifies the size of the message content in bytes. A Content-Length of 0 indicates that there is no message body.

Format
```
Content-Length: (number of bytes in message body) 
```

## CSeq
The CSeq header specifies the number of requests of each type that have been sent. For example, CSeq: 15 INVITE means that is the 15th invite request. The number increases by one for each additional request of the same type.
CSeq: Contains a sequence number and method (such as INVITE, ACK, and BYE) to indicate the order of SIP messages in a transaction

Format
```
CSeq: (number) (request type) 
```

## Max-Forwards
The Max-Forwards header limits the number of times a request can be forwarded on its way to the recipient. The number is reduced by one each time the request is forwarded. The Max-Forwards header prevents a request from endlessly circling the SIP network if the recipient cannot be found. The default value is 70.

Format
```
Max-Forwards: (maximum number of forwards) 
```

## Content-Type
If the message has a body, the Content-Type header identifies how the body is formatted. A text message might be identified as text/HTML or an application making a call might identify the content as application/SDP.

Format
```
Content-Type: (type of content)
```

# Kamailio Introduction
Kamailio (formerly OpenSER) is an open source SIP server, but Kamailio is a bit difficult to grasp what “it is“, but once you understand it’s all very logical.

Over this series I’ll attempt to explain what Kamailio is (and isn’t), and through a series of examples, show you how to use Kamailio to build cool stuff.

I’ll try and make it accessible for people with a background / understanding of VoIP, specifically with an understanding of SIP.

There’s a lot of meticulous documentation out there on specific Kamailio modules, but not much I could find that gives an overview of how the platform works, so over this series of Tutorials, I’ll attempt to cover the basics of using Kamailio to solve problems, as together we build a basic PBX with Kamailio, touching upon some of the common modules and core concepts of Kamailio.


## So what is Kamailio ?
Kamailio is a SIP Server.

It’s a bit confusing at the start, because Kamailio isn’t like FreeSWITCH, Asterisk, YATE, an SBC, a PBX or any of other telephony platforms you may have encountered before, because out of the box, Kamailio doesn’t really do anything.

You’ve got to tell Kamailio how to do everything.

Let’s take a SIP INVITE message, used to start a call (aka session) that we might send to a PBX with the domain name biloxi.example.com and a SIP endpoint registered as ‘101’:

INVITE sip:101@biloxi.example.com SIP/2.0
If we sent this message to a generic PBX, the PBX would have the logic to know that it has an extension 101 and the PBX would ring extension 101.

Our generic PBX looks at the Request URI in the INVITE message it received and has the logic predefined to know that 101 is a device it has registered and that we want to connect to that device, so sends the call to the matching device.

If we sent the same INVITE to an Asterisk box, Asterisk would take a look at our SIP INVITE message, and see if there’s an entry in the dialplan under the current context for 101. Asterisk doesn’t assume if you have a user registered on SIP/101 and receive a SIP INVITE to 101 that you want to get to SIP/101, it’d need to be told this through the dialplan.

Kamailio takes this example even further.

If we want to dial 101 on Kamailio and have it ring the device registered on 101, you have to tell Kamailio what to do when it receives an INVITE message in the first place, lookup if that destination is in our AOR (we’ll get to that) table, and then forward the INVITE to the destination if it exists, and forward the provisional responses (1xx) and finally the final response (200 OK) from the remote end back to the originator. Plus we’ve got to think about how we handle a scenario where the destination doesn’t exist, or isn’t registered, of if the destination returns a 4xx response to the INVITE, how we handle provisional responses and CANCEL messages and finally the BYE message (if we’re record routing).

Phew. This seems like a lot to handle.

It all seems pretty daunting at first, calling from one SIP Endpoint to another seems like a pretty rudimentary thing in a telephony product, but by putting how the system thinks, routes and manipulates messages up to you, we open the doors to all the possibilities.

What if you want something to sit in front of your servers and only allow certain SIP User Agents? Or load balance between several soft-switches? Or route least-cost between connected carriers and seamlessly failover if one is lost? Rate limit dodgy traffic before it hits your environment? Manage hundreds of thousands of registrations?

Kamailio can do all of that.

Kamailio can do anything you can think of (to do with signaling).

And that’s the awesome part of Kamailio. It is, what you define it to be.


## Configuration
We’ll open kamailio.cfg – the text file that contains all our routing info, and get to work.

```
vi /etc/kamailio/kamailio.cfg
```

You’ll see the config starts by defining what modules to load, and the config for each of these modules. For us, the defaults will work for now, let’s get to the juicy bits. Keep moving down the config file until you hit this section:

```
/* Main SIP request routing logic
 * - processing of any incoming SIP request starts with this route
 * - note: this is the same as route { ... } */
request_route {
```
The request_route section is what handles the initial SIP message that comes through.

We’ll remove all the text after request_route { leaving us with a blank canvas in terms of how we handle messages. We’ll then put in a single line to log to syslog when we get a SIP message:

```
/* Main SIP request routing logic
 * - processing of any incoming SIP request starts with this route
 * - note: this is the same as route { ... } */
request_route {
        xlog("I got a message");
}
```

If we were to stop here and restart Kamailio, when a SIP message comes into our SIP server, we’d just write an entry to syslog saying “I got a message” each time we get a SIP message, but Kamailio won’t respond to the received SIP message, it’ll just enter a log entry each time it gets a SIP message. So let’s respond to any SIP message we get with a 501 “Not Implemented” message.

```
/* Main SIP request routing logic
 * - processing of any incoming SIP request starts with this route
 * - note: this is the same as route { ... } */
request_route {
        xlog("I got a message");
        sl_reply("501", "Not Implemented");
}
```

So let’s break down the two functions we just used.

## xlog("");
xlog is your friend.
Put simply xlog prints whatever you put inside the parenthesis to syslog.
You can use it to check to see if your messages are getting to the part of the config you want, check what pseudovariables are doing and many other things, but I’m getting ahead of myself. For now you just need to know xlog is like NoOp() in Asterisk, print() in Python, etc.

## sl_reply();
sl_reply is stateless reply.
It takes two parameters, the SIP Response Code and the text to go with it, when called it sends the SIP response code and text back to the requester.
In this case we’re using SIP Response Code 501, which translates to Not Implemented, meaning the server does not support the functionality required to fulfill the request aka:

## What is PBX (Private Brand Exchange)?
PBX stands for Private Branch Exchange. Think of it as an internal telephone network of a business or other entity. PBX phone system users can communicate with one another over the phone via internal lines, and make and receive external calls as well. A PBX phone system usually delivers business telephony features such as call forwarding, call transfer, call queue, auto-attendant, voicemail, etc.

PBX systems operate by using either VoIP (Voice Over Internet Protocol) or via analog or digital phone lines. With a PBX phone system, the physical phone line coming into your business can be split into multiple lines, allowing you to support more telephones. Better yet, calls between users are free

![](https://getvoip.com/uploads/VoIP-vs-PBX-e1662681663590.png)


# Kamailio
This tutorial collects the functions and parameters exported by Kamailio core to configuration file.

## Table Sql Script
[sql script](https://github.com/kamailio/kamailio/blob/master/utils/kamctl/mysql)

## Structure
The structure of the kamailio.cfg can be seen as three parts:

* global parameters
* modules settings
* routing blocks

### Global Parameters Section
This is the first part of the configuration file, containing the parameters for the core of kamailio and custom global parameters.

Typically this is formed by directives of the form:

```
name=value
```

The name corresponds to a core parameter as listed in one of the next sections of this document. If a name is not matching a core parameter, then Kamailio will not start, rising an error during startup.

The value is typically an integer, boolean or a string

### Modules Settings Section
This is the second section of the configuration file, containing the directives to load modules and set their parameters.

It contains the directives loadmodule and modparam. In the default configuration file starts with the line setting the path to modules (the assignment to mpath core parameter).

Example of content:
```
loadmodule "debugger.so"
...
modparam("debugger", "cfgtrace", 1)

```

### Routing Blocks Section
This is the last section of the configuration file, typically the biggest one, containing the routing blocks with the routing logic for SIP traffic handled by Kamailio.

The only mandatory routing block is request_route, which contains the actions for deciding the routing for SIP requests.

See the chapter Routing Blocks in this document for more details about what types of routing blocks can be used in the configuration file and their role in routing SIP traffic and Kamailio behaviour.

Example of content:
```
request_route {

    # per request initial checks
    route(REQINIT);

    ...
}

branch_route[MANAGE_BRANCH] {
    xdbg("new branch [$T_branch_idx] to $ru\n");
    route(NATMANAGE);
}
```

## Mysql Module
This is a module which provides MySQL connectivity for Kamailio. It implements the DB API defined in Kamailio.

### Install 
1. install mysql 
2. setup kamailio
in kamailio.config

    Pre-Processor
    ```
    #!define WITH_MYSQL
    ```

## SecFilter Module
### Overview
This module has been designed to offer an additional layer of security over our communications. To achieve this, the following features are available:

- Blacklist to block user agents, IP addresses, countries, domains and users.
- Whitelist to allow user agents, IP addresses, countries, domains and users.
- Blacklist of destinations where the called number is not allowed.
- SQL injection attacks prevention.

When a function is called, it will be searched in the whitelist. If the value is not found, then the blacklist will be searched.

All data will be loaded into memory when the module is started. There is an RPC reload command to update all the data from database. It is also possible to add new data to the blacklist or whitelist using other RPC commands.

### Dependencies
The following modules must be loaded before this module:
* database -- Any db_* database module


### Database setup
Before running Kamailio with the secfilter module, it is necessary to setup the database table where the module will read the blacklist data from. In order to do that, if the table was not created by the installation script or you choose to install everything by yourself you can use the secfilter-create.sql SQL script in the database directories in the kamailio/scripts folder as a template. Database and table name can be set with module parameters so they can be changed, but the name of the columns must match the ones in the SQL script. You can also find the complete database documentation on the project webpage, https://www.kamailio.org/docs/db-tables/kamailio-db-devel.html.

Example 1.25. Example database content - secfilter table
```
		...
		+----+-----------+-----------+------------------+
		| id | action    | type      | data             |
		+----+-----------+-----------+------------------+
		|  1 | 0         | 2         | 1.1.1.1          |
		|  2 | 0         | 0         | friendly-scanner |
		|  3 | 0         | 0         | pplsip           |
		|  4 | 0         | 0         | sipcli           |
		|  5 | 0         | 4         | sipvicious       |
		|  6 | 0         | 1         | ps               |
		|  7 | 0         | 3         | 5.56.57.58       |
		|  8 | 1         | 0         | asterisk pbx     |
		|  9 | 1         | 2         | sip.mydomain.com |
		| 10 | 2         | 0         | 555123123        |
		| 11 | 2         | 0         | 555998776        |
		+----+-----------+-----------+------------------+
		...
```

Action values are:
* 0 (blacklist)
* 1 (whitelist)
* 2 (destination)

Type values are:
* 0 (user-agent)
* 1 (country)
* 2 (domain)
* 3 (IP address)
* 4 (user)

### Parameters
#### db_url (string)
Database URL.
Default value is ""
```
		...
		modparam("secfilter", "db_url", "mysql://user:pass@localhost/kamailio")
```

### Functions
#### secf_check_ip ()
It checks if the source IP address is blacklisted. The search is approximate and data stored in the database will be compared as a prefix. For example, if we have blacklisted IP address 192.168.1. all messages from IPs like 192.168.1.% will be rejected.

Return values are:

* 2 = the value is whitelisted
* 1 = the value is not found
* -2 = the value is blacklisted

Example 1.9. secf_check_ip usage
```
		...
        secf_check_ip();
        if ($? == -2) {
                xlog("L_ALERT", "$rm from $si blocked because IP address is blacklisted");
                exit;
        }
		...
```

#### secf_check_ua ()
It checks if the user-agent is blacklisted. The search is approximate and the comparison will be made using the values of the database as a prefix. If we add to the user-agent blacklist the word sipcli, every message whose user-agent is named, for example, sipcli/1.6 or sipcli/1.8 will be blocked. It is very useful to block different versions of the same program.

Return values are:

2 = the value is whitelisted
1 = the value is not found
-1 = error
-2 = the value is blacklisted
Example 1.10. secf_check_ua usage
```
		...
        secf_check_ua();
        if ($? == -2) {
                xlog("L_ALERT", "$rm from $si blocked because UserAgent '$ua' is blacklisted");
                exit;
        }
		...
```

#### secf_check_country (string)
Similar to secf_check_ua. It checks if the country (IP address) is blacklisted. Geoip module must be loaded to get the country code.

Return values are:

2 = the value is whitelisted
1 = the value is not found
-1 = error
-2 = the value is blacklisted
Example 1.11. secf_check_country usage
```
		...
        if (geoip2_match("$si", "src")) {
                secf_check_country($gip2(src=>cc));
                if ($avp(secfilter) == -2) {
                        xlog("L_ALERT", "$rm from $si blocked because Country '$gip2(src=>cc)' is blacklisted");
                        exit;
                }
        }
		...
```

#### secf_check_from_hdr ()
It checks if any value of from header is blacklisted. It checks if from name or from user are in the users blacklist or whitelist. It also checks if the from domain is in the domains blacklist or whitelist. The blacklisted value will be used as a prefix and if we block, for example, the user sipvicious, all users whose name starts with this word will be considered as blacklisted.

Return values are:

4 = from name is whitelisted
3 = from domain is whitelisted
2 = from user is whitelisted
1 = from header not found
-1 = error
-2 = from user is blacklisted
-3 = from domain is blacklisted
-4 = from name is blacklisted
Example 1.12. secf_check_from_hdr usage
```
		...
        secf_check_from_hdr();
        switch ($?) {
                case -2:
                        xlog("L_ALERT", "$rm to $si blocked because From user '$fU' is blacklisted");
                        exit;
                case -3:
                        xlog("L_ALERT", "$rm to $si blocked because From domain '$fd' is blacklisted");
                        exit;
                case -4:
                        xlog("L_ALERT", "$rm to $si blocked because From name '$fn' is blacklisted");
                        exit;
        };
		...
```

#### secf_check_to_hdr ()
Do the same as secf_check_from_hdr function but with the to header.

Return values are:
4 = to name is whitelisted
3 = to domain is whitelisted
2 = to user is whitelisted
1 = to header not found
-1 = error
-2 = to user is blacklisted
-3 = to domain is blacklisted
-4 = to name is blacklisted

Example 1.13. secf_check_to_hdr usage
```
		...
        secf_check_to_hdr();
        switch ($?) {
                case -2:
                        xlog("L_ALERT", "$rm to $si blocked because To user '$tU' is blacklisted");
                        exit;
                case -3:
                        xlog("L_ALERT", "$rm to $si blocked because To domain '$td' is blacklisted");
                        exit;
                case -4:
                        xlog("L_ALERT", "$rm to $si blocked because To name '$tn' is blacklisted");
                        exit;
        };
		...
```

#### secf_check_contact_hdr ()
Do the same as secf_check_from_hdr function but with the contact header.

Return values are:
3 = contact domain is whitelisted
2 = contact user is whitelisted
1 = contact header not found
-1 = error
-2 = contact user is blacklisted
-3 = contact domain is blacklisted

Example 1.14. secf_check_contact_hdr usage
```
		...
        secf_check_contact_hdr();
        switch ($?) {
                case -2:
                        xlog("L_ALERT", "$rm to $si blocked because Contact user '$ct' is blacklisted");
                        exit;
                case -3:
                        xlog("L_ALERT", "$rm to $si blocked because Contact domain '$ct' is blacklisted");
                        exit;
        };
		...
```

#### secf_check_dst (string)
It checks if the destination number is blacklisted. It must be user for INVITE messages. If the value of dst_exact_match is 1, the call will appear as blacklisted if the destination is exactly the same. If the value is 0, every destination whose number begins with a number appearing on the destination blacklist will be rejected.

Return values are:
2 (if the value is whitelisted)
1 (if the value not found)
-2 (if the value is blacklisted)

Example 1.15. secf_check_dst usage
```
		...
		if (is_method("INVITE")) {
			secf_check_dst($rU);
			if ($? == -2) {
				xlog("L_ALERT", "$rm from $si blocked because destination $rU is blacklisted");
				send_reply("403", "Forbidden");
				exit;
			}
		}
		...
```

## Dispatcher Module
### Params
#### ds_ping_interval (int)
With this parameter you can define the interval for sending a request to a gateway marked as inactive upon a failed request routing to it. This parameter is only used, when the TM-Module is loaded. If set to “0”, the pinging of inactive gateway is disabled.

Default value is “0”.

Example 1.22. Set the “ds_ping_interval” parameter
```
...
modparam("dispatcher", "ds_ping_interval", 30)
...
```

#### ds_ping_method (string)
With this method you can define, with which method you want to probe the gateways. Pinging gateways feature depends on ds_ping_interval parameter.

Default value is “OPTIONS”.

Example 1.20. Set the “ds_ping_method” parameter

...
modparam("dispatcher", "ds_ping_method", "INFO")
...

#### SIP requests
There are fourteen SIP Request methods of which the first six are the most basic request / method types:

* INVITE = Establishes a session
* ACK = Confirms an INVITE request
* BYE = Ends a session
* CANCEL = Cancels establishing of a session
* REGISTER = Communicates user location (host name, IP)
* OPTIONS = Communicates information about the capabilities of the calling and receiving SIP phones
* PRACK = Provisional Acknowledgement
* SUBSCRIBE = Subscribes for Notification from the notifier
* NOTIFY = Notifies the subscriber of a new event
* PUBLISH = Publishes an event to the Server
* INFO = Sends mid session information
* REFER = Asks the recipient to issue call transfer
* MESSAGE = Transports Instant Messages
* UPDATE = Modifies the state of a session

#### SIP responses
SIP Requests are answered with SIP responses, of which there are six classes:

* 1xx = Informational responses, such as 180 (ringing)
* 2xx = Success responses
* 3xx = Redirection responses
* 4XX = Request failures
* 5xx = Server errors
* 6xx = Global failures