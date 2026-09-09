This room is about the basics of splunk.

1. Splunk uses these 3 things which are:
   Forwarder - Exists on every end device and responsible for collecting these logs and forwarding it to the indexer
   Indexer - stores all these collected logs and indexes them to match appropriate fields making them easier to search and
   normalizing them.
   Search head - it is where analysts search for specific events and logs.

2. when adding data here are some things to consider:
   Manual - It refers to manually uploading log files into splunk. These fields are worth mentioning as well
     Host - This is the machine host where the log came from. So it could be PC-01, VPN_SERVER_01
     Index - This is where splunk stores these logs from the host. Ex, VPN_LOGS, WINDOWS_LOGS
   Port forwarding - Splunk can have a listening port where devices can connect to to send their logs.
   Agent - Splunk agents can be installed in each device to do the forwarding

3. When searching in splunk, it is very case sensitive so make sure capitalization and spelling is correct.
   Here are some useful basic searches:
   index = (index_name), this will result in showing the entire log for that index. Ex, entire VPN_LOGS log.
   | stats count , This will show the amount of logs captured.
   | search UserName="Phill", This will filter out the UserName in displaying Phill in the search bar.
   
   you can use also wildcards like:
   email = *"hello", this means any value in the email field that ends with hello
   email = "hello"*, this means any value in the email field that starts with hello
   email = *"hello"*, Any value in email field that contains hello.
