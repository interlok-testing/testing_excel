# Excel Testing

Project tests interlok-excel features

# What it does
 
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
 
