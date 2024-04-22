# System Telephony

# Kamailio
This tutorial collects the functions and parameters exported by Kamailio core to configuration file.

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

