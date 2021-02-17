# Excel Testing

Project tests interlok-excel features

## What it does
 
 The config contains one workflow, which reads in a spreadsheet from file system, turns it into xml, then converts into formatted json and then outputs to file system.

 - At the start of the workflow there is a fs-consumer polling the excel-in folder every 20 secs
 - The message is processed by excel-to-xml-service, and this service reads in the xls file and outputs XML
 - Then this xml payload is converted to json using the json-xml-transform-service
 - This json is then formatted using the json-transform-service (using jolt mapping spec)
 - Then the json array is split up (one message per array element) and written to file using the basic-message-splitter-service
 - At the end of the workflow there is a fs-producer which outputs the json array to file
 
## Getting started

* `./gradlew clean build`
* `(cd ./build/distribution && java -jar lib/interlok-boot.jar)`

At this point you have a running Interlok instance.
 
* now you can copy the `\excel-testing\src\test\resources\jira-ticket-example.xls` into the `./build/distribution/messages/excel-in` folder
 
 Once processed the output is available:
 
 * `./build/distribution/messages/json-output`
 * `./build/distribution/messages/json-split-output`
 

## Example Outputs

Excel source:

![Excel source example](/excel-example-screenshot.png "Excel source example")

excel-to-xml-service transforms the excel input to xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<spreadsheet>
   <sheet name="JIRA">
      <row>
         <Issue_key>INTERLOK-3710</Issue_key>
         <Summary>UI - Support Jolokia http request in the UI</Summary>
         <Story_Points>8.0</Story_Points>
         <Issue_Type>Story</Issue_Type>
         <Updated>2021-02-16T04:29:00+0000</Updated>
         <Status>Open</Status>
      </row>
      <row>
         <Issue_key>INTERLOK-3708</Issue_key>
         <Summary>Rename interlok-fx-installer to interlok-installer</Summary>
         <Story_Points>1.0</Story_Points>
         <Issue_Type>Story</Issue_Type>
         <Updated>2021-02-16T04:25:00+0000</Updated>
         <Status>Open</Status>
      </row>
      <row>
         <Issue_key>INTERLOK-3707</Issue_key>
         <Summary>UI - create missing optional component icons</Summary>
         <Story_Points>1.0</Story_Points>
         <Issue_Type>Story</Issue_Type>
         <Updated>2021-02-15T09:42:00+0000</Updated>
         <Status>Closed</Status>
      </row>
      <row>
         <Issue_key>INTERLOK-3705</Issue_key>
         <Summary>Add Jetty Configurable Key Security Handler</Summary>
         <Story_Points/>
         <Issue_Type>Story</Issue_Type>
         <Updated>2021-02-15T05:48:00+0000</Updated>
         <Status>Open</Status>
      </row>
      <row>
         <Issue_key>INTERLOK-3704</Issue_key>
         <Summary>UI - Dependency vulnerabilities in shiro, httpclient and guava</Summary>
         <Story_Points>1.0</Story_Points>
         <Issue_Type>Story</Issue_Type>
         <Updated>2021-02-14T06:05:00+0000</Updated>
         <Status>Closed</Status>
      </row>
   </sheet>
</spreadsheet>
```

json-xml-transform-service transforms this xml to json:

```json
{
   "spreadsheet":[
      {
         "@name":"JIRA",
         "row":[
            {
               "Issue_key":"INTERLOK-3710",
               "Summary":"UI - Support Jolokia http request in the UI",
               "Story_Points":"8.0",
               "Issue_Type":"Story",
               "Updated":"2021-02-16T04:29:00+0000",
               "Status":"Open"
            },
            {
               "Issue_key":"INTERLOK-3708",
               "Summary":"Rename interlok-fx-installer to interlok-installer",
               "Story_Points":"1.0",
               "Issue_Type":"Story",
               "Updated":"2021-02-16T04:25:00+0000",
               "Status":"Open"
            },
            {
               "Issue_key":"INTERLOK-3707",
               "Summary":"UI - create missing optional component icons",
               "Story_Points":"1.0",
               "Issue_Type":"Story",
               "Updated":"2021-02-15T09:42:00+0000",
               "Status":"Closed"
            },
            {
               "Issue_key":"INTERLOK-3705",
               "Summary":"Add Jetty Configurable Key Security Handler",
               "Story_Points":[
                  
               ],
               "Issue_Type":"Story",
               "Updated":"2021-02-15T05:48:00+0000",
               "Status":"Open"
            },
            {
               "Issue_key":"INTERLOK-3704",
               "Summary":"UI - Dependency vulnerabilities in shiro, httpclient and guava",
               "Story_Points":"1.0",
               "Issue_Type":"Story",
               "Updated":"2021-02-14T06:05:00+0000",
               "Status":"Closed"
            }
         ]
      }
   ]
}
```

the json-transform-service then reformats the json:

using jolt mapping spec:
```json
  {
    "operation": "shift",
    "spec": {
      "spreadsheet": {
        "*": {
          "row": {
            "*": {
              "Issue_key": "[&1].issue-key",
              "Summary": "[&1].summary",
              "Story_Points": "[&1].story-points",
              "Issue_Type": "[&1].issue-type",
              "Updated": "[&1].updated",
              "Status": "[&1].status"
            }
          }
        }
      }
    }
  }
```

json output:
```json
[
   {
      "issue-key":"INTERLOK-3710",
      "summary":"UI - Support Jolokia http request in the UI",
      "story-points":"8.0",
      "issue-type":"Story",
      "updated":"2021-02-16T04:29:00+0000",
      "status":"Open"
   },
   {
      "issue-key":"INTERLOK-3708",
      "summary":"Rename interlok-fx-installer to interlok-installer",
      "story-points":"1.0",
      "issue-type":"Story",
      "updated":"2021-02-16T04:25:00+0000",
      "status":"Open"
   },
   {
      "issue-key":"INTERLOK-3707",
      "summary":"UI - create missing optional component icons",
      "story-points":"1.0",
      "issue-type":"Story",
      "updated":"2021-02-15T09:42:00+0000",
      "status":"Closed"
   },
   {
      "issue-key":"INTERLOK-3705",
      "summary":"Add Jetty Configurable Key Security Handler",
      "story-points":[],
      "issue-type":"Story",
      "updated":"2021-02-15T05:48:00+0000",
      "status":"Open"
   },
   {
      "issue-key":"INTERLOK-3704",
      "summary":"UI - Dependency vulnerabilities in shiro, httpclient and guava",
      "story-points":"1.0",
      "issue-type":"Story",
      "updated":"2021-02-14T06:05:00+0000",
      "status":"Closed"
   }
]
```