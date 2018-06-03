Host map with tags:
https://www.dropbox.com/s/cwi22do0qrddrs8/Screenshot%202018-05-16%2010.30.37.png?dl=0

I installed mongodb (for Mac) and had issues attempting to configure the DataDog Agent:
https://www.dropbox.com/s/0k2w1hp4usoikjs/Screen%20Shot%202018-05-16%20at%2011.10.06%20AM.png?dl=0
https://www.dropbox.com/s/jcl0h6vs4mftmik/Screen%20Shot%202018-05-16%20at%2011.29.10%20AM.png?dl=0

Some of the documentation seems out of date, specifically, the instruction, "Execute the 'info' command."
quigbook:~ quiglw$ /usr/local/bin/datadog-agent info
Error: unknown command "info" for "agent"
Run 'agent --help' for usage.

Regardless, 'agent status' did not recognize the mongodb agent, because it failed to parse my mongodb configuration file, which looks like this:

init_config:

instances:
  # Specify the MongoDB URI, with database to use for reporting (defaults to "admin")
# mongodb://datadog:LnCbkX4uhpuLHSUrcayEoAZA@localhost:27016/my-db
 - server: mongodb://datadog:password@localhost:27017/test

Not sure what's wrong with that, but the agent reports this:

Config Errors
  ==============
    mongo
    -----
      yaml: unmarshal errors:
  line 6: cannot unmarshal !!map into []check.ConfigRawMap



Here is my custom agent check:

from checks import AgentCheck
import random
class QuigCheck(AgentCheck):
  def check(self, instance):
    self.gauge('my.metric', random.randint(0,1000))
    
You can change the collection interval without modifying the check file by modifying the global collection interval.

Here is my timeboard:
https://www.dropbox.com/s/hwzmdqcrglc7uf9/Screenshot%202018-05-17%2008.30.30.png?dl=0

Here are some screenshots of the process:
https://www.dropbox.com/sh/9onshpf8ab9b4gp/AACB7YtkCVKn8SydOMtZtVbHa?dl=0

5-minute snapshot:
https://www.dropbox.com/s/riw26pmwt0x6vx7/Screenshot%202018-05-29%2009.21.52.png?dl=0

Metric Monitor:
https://www.dropbox.com/s/n1l6e0ce514p1mf/Screenshot%202018-05-17%2008.45.51.png?dl=0
https://www.dropbox.com/s/08i51bei0uhwor1/Screenshot%202018-05-17%2008.55.36.png?dl=0
https://www.dropbox.com/s/fwj7i0zjcp3hhls/Screenshot%202018-05-17%2008.58.57.png?dl=0

Email for alert:
https://www.dropbox.com/s/orycj9fplflu0kf/Screenshot%202018-05-29%2009.26.42.png?dl=0

Alert downtime:
https://www.dropbox.com/s/52xa99tevfxkfjt/Screenshot%202018-05-17%2009.04.47.png?dl=0

Downtime email:
https://www.dropbox.com/s/jg12vos953fnei4/Screenshot%202018-05-17%2009.05.06.png?dl=0

Flask app instrumentation:
https://www.dropbox.com/s/g8zkt87nqpzroaz/Screenshot%202018-05-17%2009.53.16.png?dl=0
https://www.dropbox.com/s/pdmqelgy6fvx6wm/Screenshot%202018-05-17%2009.56.15.png?dl=0

My app:
from flask import Flask
import logging
import sys

from ddtrace import tracer

with tracer.trace("web.request", service="my_service") as span:
  span.set_tag("my_tag", "my_value")

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run()

