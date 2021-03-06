
### What is RESTCONF

As per RFC 8040 (RESTCONF Protocol), the IETF describes RESTCONF as:

> an HTTP-based protocol that provides a programmatic interface for accessing data defined in YANG, using the datastore concepts defined in the Network Configuration Protocol (NETCONF).

### Difference RESTCONF and NETCONF

We learned that NETCONF is using SSH, is RPC based and only supports XML. RESTCONF, as the name indicates, supports REST. Hence, it works with HTTP and both XML and JSON are supported. This makes is essentially lighter and simpler to work with.

### RESTCONF URIs

A word about RESTCONF URI's as it will make it easier to understand the upcoming examples. A RESTCONF URI has the following format.

> https://\<ADDRESS>/\<ROOT>/\<DATASTORE>/\<[YANGMODULE:]CONTAINER>/\<LEAF>[?\<OPTIONS>]

Here's some explanation:

- ADDRESS: the address of the RESTCONF agent
- ROOT: the main entry point for RESTCONF requests
- DATASTORE: the datastore that is being queried
- [YANGMODULE:]CONTAINER: the base model container being used
- LEAF: individual element from within that container
- OPTIONS: optional parameters

Let's go through an example:

> https://ios-xe-mgmt-latest.cisco.com:9443/restconf/data/ietf-interfaces:interfaces/interface=GigabitEthernet3

- ADDRESS: ios-xe-mgmt-latest.cisco.com:9443
- ROOT: restconf
- DATASTORE: data
- [YANGMODULE:]CONTAINER: ietf-interfaces:interfaces
- LEAF: interface

Let's go through some examples.

### Capabilities

In this example, we will query the IOS XE device for the capabilities list. We will get back a complete list of YANG modules supported by our device.

### Use Case 1: retrieve device inventory

As part of the capabilities list, we see a YANG model called `"http://cisco.com/ns/yang/Cisco-IOS-XE-device-hardware-oper?module=Cisco-IOS-XE-device-hardware-oper&revision=2018-10-29"`. Let's say we want to check this out in more detail. To achieve that, we have to use the module name in the POSTMAN URL, which in this example would be `https://{{host}}:{{port}}/restconf/data/Cisco-IOS-XE-device-hardware-oper`.

If we do that, you will see we get an error message.

We need to specify what we are interested in. As we are using the `device-hardware` YANG model, let's say...we are interested to find a list of the various hardware components in our device (with serial numbers....)

There are multiple ways to go about. I usually go to a website called [YANG Search](https://yangcatalog.org/yang-search/). There is type in the name of the module I'm interested in. In our case that would be our module name, being `Cisco-IOS-XE-device-hardware-oper`.

You will receive back a list of items that are supported by the YANG model. As we are interested in the hardware components, we search for the keyword `hardware` in the search box on the right (note: put the filter on name).

We will get back 3 modules. We will dive in deeper into the `device-hardware-data` model. In the path column, you'll notice that we can retrieve the device hardware data by adding the `device-hardware-data` to our path. The full POSTMAN URL now becomes `https://{{host}}:{{port}}/restconf/data/Cisco-IOS-XE-device-hardware-oper:device-hardware-data`.

You see that we receive now a complete list of hardware related data. In order to drill down further in the results, there are two ways:

A) Look at the POSTMAN response

In the POSTMAN response, you will see containers called `entity-information` and `device-hardware`. This means you can put them after the current URL. Our complete URL would then become `https://{{host}}:{{port}}/restconf/data/Cisco-IOS-XE-device-hardware-oper:device-hardware-data/device-hardware`.

B) Look at the YANG model website

In the search results (see screenshot above), there is 'module' column. In that column there is a tree view. When you click on that, you get back an overview of the containers. See screenshot below:

Both methods work well. In any case, we now know the container we can add to our URL. The full URL to be used in POSTMAN would now be `https://{{host}}:{{port}}/restconf/data/Cisco-IOS-XE-device-hardware-oper:device-hardware-data/device-hardware/device-inventory`. Let's try it out.

And here you have a list of your devic. inventory including the serial numbers.

### Use Case 2: retrieve interface information with IETF YANG model

In the capabilities list (see first screenshot in this blog post), we also got a module called `"urn:ietf:params:xml:ns:yang:ietf-interfaces?module=ietf-interfaces&revision=2014-05-08&features=pre-provisioning,if-mib,arbitrary-names"`. The module name is called `ietf-interfaces`.

Let's go again to the [YANG Search website](https://yangcatalog.org/yang-search) and search for this module name.

Let's search for 'interfaces' (as we are interested to retrieve interface information). We get back a list of containers as indicated in the screenshot below:

The POSTMAN URL would now be `https://{{host}}:{{port}}/restconf/data/ietf-interfaces:interfaces`.

Let's look at the tree for the 'ietf-interfaces' module.

The POSTMAN URL would now be `https://{{host}}:{{port}}/restconf/data/ietf-interfaces:interfaces/interface`.

You see now an overview of all the interfaces of our device. Let's say we are interested in one particular interface. We can do this as follows: `https://{{host}}:{{port}}/restconf/data/ietf-interfaces:interfaces/interface=GigabitEthernet3`.

### Use Case 3: retrieve interface information with Cisco YANG model

Sometimes there are multiple YANG models to achieve the same output. In use case 2, we retrieved a list of interfaces. We can also use a Cisco YANG model for this, e.g `Cisco-IOS-XE-interfaces-oper` (also retrieved from the capabilities list). This model focuses more on operational data but when it comes to listing the interfaces, it can be used as well. 

We won't go through all the details using the YANG Search website as we did that already twice in use case 1 and 2. If you follow the same procedure, you will see that the URL can be `https://{{host}}:{{port}}/restconf/data/Cisco-IOS-XE-interfaces-oper:interfaces`.

To list the details of a specific interface, use `https://{{host}}:{{port}}/restconf/data/Cisco-IOS-XE-interfaces-oper:interfaces/interface=GigabitEthernet3`.

The Cisco YANG model is a bit different than the IETF YANG model, hence the data returned will also be a bit different.
