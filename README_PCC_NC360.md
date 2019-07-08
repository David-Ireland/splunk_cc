

# Splunk Custom Python Command Extract NC360 Data

## Installation
### Requirements

* Install Splunk
* Install Python

`$ pip install splunk-sdk`

## Add Files

### Commands.conf File

At location:

`$SPLUNKHOME/etc/apps/<APP_NAME>/local`

Create or add to the commands.conf file:

```
[nc360_extract]
filename = nc360_extract.py
overrides_timeorder = true
```

### Python File

`At Location:`

$SPLUNKHOME/etc/apps/<APP_NAME>/bin

Insert the python file nc360_extract.py:

```python

import splunk.Intersplunk
import re

try:
    # get the previous search results
    results,unused1,unused2 = splunk.Intersplunk.getOrganizedResults()

    # Regex to pull all possibly relevant data
    regex_string="(?P<_time>[0-9\.]*\ [0-9:\.]*)[\ \']*(?P<method>(?:GET|POST|PUT|DELETE)?)[\ \']*(?P<tran_type>(?:Calculate|TicketClose){1})[\' ]*(?P<status>[a-zA-Z\ ]*)"
    regex=re.compile(regex_string)

    nc360_list=[]

    # for each results, add a 'shape' attribute, calculated from the raw event text
    for result in results:
        nc360_dict = [m.groupdict() for m in regex.finditer(result["_raw"])]
        for event in nc360_dict:
            del event["method"]
            nc360_list.append(event)
    
    # with open("/home/<user>/Desktop/nc360_events.txt", 'w') as f:
    #   f.write(str(nc360_list))

    splunk.Intersplunk.outputResults(nc360_list)

except:
    import traceback
    stack =  traceback.format_exc()
    # with open("/home/<user>/Desktop/nc360_errors_log.txt", 'w') as f:
    #     f.write(str(stack))
```

### Splunk Search
Search inside the edited Splunk app:

`index="nc360" | nc360_extract`



