# znr

`znr` (pronounced "zee-nur", like the zener diode) is an issue tracker backend. It is meant to be highly automatable, written API-first instead of UI-first.

## `znrd`
`znrd` is the backend itself, which interacts with storage (typically PostgreSQL, though other storage should be possible) and presents a REST interface operating on JSON objects and literals. `znrd` offers a versioned API with all endpoints prefixed by their API version.

## `znr`
`znr` is a command-line tool to interact with `znrd`. In essence it has three functions:

1. Create and maintain `.znr.conf` files to hold state and create the notion of "workspaces"
2. Issue requests and receive responses from `znrd`
3. Format responses to be useful 

A typical session could look like this:
```
$ mkdir -p ~/office
$ cd ~/office
$ znr init -track -user ironiridis https://znr.host/ironiridis/theoffice/
Created new tracker ironiridis/theoffice
New local workspace tracking https://znr.host/ironiridis/theoffice/
$ znr new "Grant Jack access to kitchen printer"
Issue 1: Grant Jack acces to kitchen printer (created by ironiridis - June 14 16:25:40 CDT)
$ znr get -all -fmt json 1
{  
   "id":1,
   "state":"new",
   "stateuuid":"a7e715f8-fe83-4b0c-b319-a2ad5c3440b6",
   "attributes":{  
      "title":"Grant Jack access to kitchen printer"
   },
   "history":[  
      {  
         "ts":"2016-06-14T21:25:40Z",
         "source":"user:ironiridis",
         "action":"created"
      },
      {  
         "ts":"2016-06-14T21:25:40Z",
         "source":"user:ironiridis",
         "action":"setattribute",
         "attribute":"title",
         "value":"Grant Jack access to kitchen printer"
      }
   ]
}
$ znr flow reopen 1
Issue 1: Error 403 - Cannot find matching flow "reopen" for current state "new". Available flows are: "closeinvalid", "complete", "pendinfo", "pendvendor", "assign", "start"
$ znr modify -attr vendorticket:81033 1
Issue 1: Created attribute "vendorticket" with value "81033"
$ znr flow -comment "Filed a ticket with InitechIT." pendvendor 1
Issue 1: Created attribute "pendingon" with value "vendor"
Issue 1: State is now "pending" (was "new")
$ znr new -attr priority:1 "Install Julie's new desk phone"
Issue 2: Install Julie's new desk phone (created by ironiridis - June 14 17:02:19 CDT)
Issue 2: Created attribute "priority" with value "1"
$ znr flow -comment "Arrived today. Jen, please take care of this in the morning." assign 2
Issue 2: Error 403 - Flow "assign" requires attribute "assigned" to be set.
$ znr flow -comment "Arrived today. Jen, please take care of this in the morning." -attr assigned:jen@example.com assign 2
Issue 2: Added comment
Issue 2: Created attribute "assigned" with value "jen@example.com"
Issue 2: State is now "assigned" (was "new")
$ znr flow -comment "Just kidding, this is all just for the README." closeinvalid 1 2
Issue 1: Added comment
Issue 1: Deleted attribute "pendingon"
Issue 1: Created attribute "closereason" with value "invalid"
Issue 1: State is now "closed" (was "pending")
Issue 2: Added comment
Issue 2: Deleted attribute "assigned"
Issue 2: Created attribute "closereason" with value "invalid"
Issue 2: State is now "closed" (was "assigned")
```
