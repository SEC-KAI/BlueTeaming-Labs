This lab is about elastick stack and kibana. Here are key concepts and terminologies to know as a soc analyst level 1.

1. Elastic stack contains these 4 concepts that work together to collect and display logs.
   Beats - They are agents in end devices used to collect logs which are often called like filebeat or packetbeat
   Logstash - its then forwarded to logstash which is responsible for filtering out fields and parsing them and formatting them
   Elastic Search - Its where the logstash forwards the formatted logs to and stores it
   Kibana - This is what the analyst uses to display these logs in a dashboard.

2. I found that this is very similar to splunk since they are both SIEM solutions. The only difference is the interface, method
   of collecting logs and language used.

3. Opening kibana, you can go to discover tab to view these parsed logs. You will select an index from the dropbox and add
   filters by hitting the + icon beside the field you want. You can also create tables for easier view and save it for future
   use.

4. FILTERING SEARCH: kibana uses KQL, different from splunk but the idea is the same.
     entering john on the search bar will display logs with john. Wildcards can also be used.
     Using AND OR operators. United States AND Virginia, United States OR Japan

5. Creating visualizations can be used to make charts, tables, graphs etc. This can also be saved and be displayed in a dashboard

