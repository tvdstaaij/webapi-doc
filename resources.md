Resources
=========

This document gives a detailed description of the available resources. You can find an overview description in the [Brain web API documentation](README.md).

Contents
--------

* [Items](#items)
* [Spots](#spots)
* [Locations](#locations)
* [Presences](#presences)
* [Events](#events)  

Items
-----

The items resource will contain all [item](README.md#item) objects that where detected in one of the [zones](README.md#zone). They are automatically added as soon as they are are detected for the first time. The items resource couples a unique id to every item and gives a place to add more information to an item.

Every item contains at least a unique id (`id`), the detected `code` and the `technology` (this combiniation is unique). The `item_id` is the reference to the item that is used in all other places in the system.

You may add a `label` and a `custom` value to the item. The label is used to shwo a human readable name on several places in our user interface. 

The item is also a placeholder for the output of the localisation service. If the item is beeing detected on multiple places then we offer a most likely `location`, based on all avaialble information. The `location` only changes when the item moves to another place. If it moves out of reach then it sticks to the last known `location`. You can use `is_present` to see if the item is actively detected. We believe that this information is enough for most use cases, it just tells you where your items are.

Every item has the following fields defined:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `code_hex` | string | String representation of the unique code that this item transmits. By default this is a hexadecimal representation. This number could be so long (> 40 bytes!) that a decimal representation would be useless to generate.
| `technology`  | string | Type of technology that was used to detect this item. Can be 'EPC Gen2' or 'Bluetooth LE'. |
| `label`| string | A name or a label for this item. Is shown in our user interface, may also be empty. |
| `custom` | value | The `custom` value is only for your own reference, you may use it to save additional attributes. The `custom` value is not used on any other place. This field may contain any datatype that you like: null (default), text, number, boolean or object. |
| `is_present`| boolean | Is this item actively detected in one of the zones at this moment? True when it is, false if it's not. |
| `location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to the [location](#locations) resource where the item is located. Or, if the item is out of reach, the last known location. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource created? |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource updated for the last time? Can be a location update or a label update. |

You may be worried about the amount of items that could flow into your system. In the future you can configure the spots to only allow certain code ranges with the flexible item sets approach. With this approach you can filter the amount of tags that come into your system.

At this moment it's not possible to delete items.

Spots
-----

The Intellifi Spot devices form the eyes and the ears of the server logic. They generate events for every item that is detected. By doing so they 'implement' the earlier described [zone](README.md#zone) abstraction. 

Every spot has it's own representation inside the spots resource. This allows you to see and monitor the current status of a spot. You can also configure the location that the spot reports it's detections to.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `report_location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to location resource that this overall spot reports it's detection to. You may set this to null if you don't want the spot to report overall presences. |
| `antenna_report_locations` | object | You may configure this field to an object which couples individual antenna ports to locations. An example is given below. |
| `serial_number` | number | This is the fixed and unqiue spot number. It's assigned during the production process and used to identify an individual device during it's lifetime. |
| `is_online` | boolean | True when the spot is active and capable of sending events. |
| `state` | string | A text string indicating the current state of the spot. At this moment 'active' is the only value that yuo should see. |
| `request_counter` | number | The total number of HTTP requests that the spot has done |
| `status` | object | An object with specific information about the spot, directly send by the spot itself when the connection is created. |
| `config` | object | An object with the current spot configuraton, also directly send by the spot itself when the connection is created. |
| `config_request` | object | You can request a config change by doing a PUT on this field. You should put the same name is in the `config` object with the value that you would like. The request is forwarded to the spot, and then the setting is applied (when valid). The field is cleared as soon as the request is transferred to the spot. |
| `senses` | object | Senses are values that in most cases are generated inside the spot (number of presences, spot booted etc.). We also have a few senses that can be controlled by the brain. See [update remote sense](#update-remote-sense) for more info. |
| `senses_request` | object | You can request a change for a brain sense by doing a PUT on this field. As soon as the request is handled the field is cleared. See [update remote sense](#update-remote-sense) for more info.
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. The timestamp of the first HTTP request to this server. |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change in this resource? Is not updated by the request_counter, you can use time_last_request to see that. |

You can't add a label or a note to the spot. This is by design. We gave the seperate [location resource](#locations) responsibility for labels and notes.

The spots are automatically added to this resource when they are connected to this server. Spots are never deleted automatically, you may delete a spot that is offline with the HTTP delete action.

We will include a way to authenticate a spot in the future. This is a critical feature to refrain people from abusing the openness of our system. At this moment we advise you to use production spots in a 'closed' network only. Please let us know if you are very eager to have this feature.

### Defining report locations

Every spot can report it's presences to one location. You can also configure individual antennas to report to different locations (antenna presences). You may do an HTTP PUT with the following body to configure 4 individual antennas to report to some location. You have to supply the id's of the locations that you want to report to.
```
{
	"report_location": "54f97c3cc573f4a82099749f",
	"antenna_report_locations":[
        {"antenna_number":1, "report_location": "54f97c3cc573f4a82099749c"},
        {"antenna_number":2, "report_location": "54f97c3cc573f4a82099749c"},
        {"antenna_number":3, "report_location": "54f97c42c573f4a82099749d"},
        {"antenna_number":4, "report_location": "54f97c42c573f4a82099749d"}
	]
}
```

If you want to clear all report locations then you should send this JSON message:
```
{
	"report_location": null,
	"antenna_report_locations": []
}
```

Normally you would only have a filled `report_location` ór a filled `antenna_report_locations` field. If you supply both then items will become visible on multiple locations at the same time.

### Update remote senses
You can change a remote sense by sending a PUT request to the spot directly (api/spots/`id`). 5 remote sense are supported at this moment. bs1 to bs5 (where bs is short for brain sense). The remote sense are described in an object. Please note that any other key added to this object will be ignored at spot level.

You only need to add the senses that you want to change. I.e. updating the first three senses is done by sending the following object: 
```
{
	"senses_request": {
		"bs1":24,
		"bs2":1,
		"bs3":0
	}
}
```

The object is cleared as soon as the request is forwarded to the spot. The normal `senses` field will contain the current status of the senses.

Locations
---------

The locations resource allows you to create, read and update the definitions for your locations. A location couples a [zone](README.md#zone)(i.e an Intellifi Spot) to a `label` and `custom` field that you can fill in with any information that you like.

A zone or reader device reports to a location. In most situations your Intellifi Spot will behave itself as a single zone and will report to a single location. An antenna that is connected to the Intellifi spot may also be used as a zone (virtual spot). And thus can also trigger a location. Locations are coupled by a bottum-up approach. Every zone may report to a single locaton. You may let different zones report to a single location.

This concept allows you to use the Intellifi Spots in a lot of situations. You can let multiple spots report to a single big location. Effectively a bigger zone. You can also let one Spot report to several smaller locations (virtual spot). Every antenna can be used as a seperate zone.

A default location for an Intellifi Spot is automatically created when you connect it to the server for the fist time. You may edit or remove this location.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `label` | string | How do you name this resource? Or how do you refer to it in your own applicaton?
| `custom` | value | The `custom` value is only for your own reference, you may use it to save additional attributes. The `custom` value is not used on any other place. This field may contain any datatype that you like: null (default), text, number, boolean or object. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change in this resource? |

Presences
---------

An item can be present on a location. A presence resource is automatically created when one of the defined location triggers says that an item is detected. A presence is deleted when it has not been detected for n seconds. Where n is the hold time in seconds (you may configure this parameters on your Smart Spot). So the presence resource exactly tells you where your items are beeing detected at this very moment.

An item can be present on multiple locatons at the same time. This is logical if you know that the used technolgies all have a big range. Your items may be picked up by multiple devices at the same moment. In this resource we present all this information to you. If you just only want to know where something is excatly located then we have good news: we already did the hard work for you. The localisation service does a best fit and determines where your item is. The calculated location is directly saved within the [items](#items) resource. You don't need to query this resource in that situation. This resource reveals more of what is happening inside the system. For some use cases this is really usefull.

Presence are deleted when their hold time expires, or in other words: when they have not been seen for a some time. This is an important difference to the first API version that we had. You can use the [events resource](#events) to query all events that took place in the system. Including create, update and delete events for presences. So you can always reconstruct the presences that where avaialble at some time. Please let us know if it would make you happy that we did this for you.

The presence contains these fields:

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `item` | [reference](README.md#reference) | Reference to the item that was detected |
| `location` | [reference](README.md#reference) | Reference to the locaton that this item was seen on. |
| `proximity` | string | Strongest proximity of all 'child' presences, see next paragraph. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | Created time, when was the first hit received for this presence? |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last edit in this presence resource? |

We add an estimated proximity to every presence. This is a rough estimate on the distance from the item to the receiver. 3 possible values are returned:

1. 'far': the item is detected, but the received signal is weak. In most cases this means that the item is far away, but it also might indicate that you have interference or a seriously low battery.
2. 'near': the item is detected with an average signal strength.
3. 'immediate': the item is detected with a very strong signal. It must be very close to your antenna.

The returned value depends on the configured signal levels. It's possible to adjust these levels to your situation, please refer to the detailed Intellifi spot documentation if you would like to do this.

Events
------

The events resource keeps a copy of all relevant events that occured. This is an exact copy of the events that are avaialble on the message bus. Please note that lots of events are flowing through the system. The history of events is kept for a limited time. If you would like to retreive all events then you should consider connecting to our message bus through one of the avaialble [push technologies](https://github.com/intellifi-nl/doc-push).

Every event is envelopped in an JSON object with the following fields:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `topic` | object | A message is always accompanied by a topic string. In the brain we use a fixed format for this string. The topic string is parsed and the individual fields are shown in the object that is placed into this field. 
| `payload` | object | An object that contains the used encoding and the actual payload (if any). We will try to get this in line with the websockets output. Possible encodings: 'json', 'utf8' or 'base64'. We might add 'null', for now it's just an empty utf8 string if nothing was send.  |
| `time_event` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When did this event actually took place on the device? This is the device it's own timestamp. Could be different due to buffering and clock differences. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this event resource created at the server? |
| `time_expire` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When is this event going to removed from the database? |

The topic object is always filled with these properties:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `resource_type` | string | One of the defined [resources](#resource). Is also written in it's plural form. I.e. 'spots', 'items'. |
| `resource` | [reference](README.md#reference) | Reference to one of the resources. Please note that it's an event from the past, the resource may not exist anymore. |
| `action` | string | Indicates the kind of event that was executed. In most cases it's a verb. I.e. 'connected', 'created' etc.
| `arguments` | object | Extra arguments may be added to a topic string, it depends on the `resource_type` and the `action` what extra arguments are added.

You may also query on these value by using a dot(.) in the url. More background information on the [topic format](https://github.com/intellifi-nl/doc-push/blob/master/mqtt_topics.md#format) can be found in the push documentation.

Be carefull with the given `time_create` and `time_device` fields. Intellifi Spots can buffer events in case of a network loss (a very small number for the moment). If you are traversing over the resource then you should look at `time_create`, eventual old events that pop up will just be in your result. If you are actually doing something with the event then you should look at `time_device`!

An event is never changed (can't be by definition!), so we don't offer a `time_update` field on this resource.

Future resources
----------------

We are working hard on new features. Some of the new forseen features are already mentioned here as a sneak preview.

* [Paths](#paths)
* [Passings](#passings)
* [Sets](#sets)
* [Senses](#senses)