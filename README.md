# Sahla Farm n8n Workflows :

## Rapport :

Here are the modules and their functionalities in details:

### *1) Data Fetcher :*
each X minutes, it fetches all the states from home assistant and filter them to make the payload
for all the data, everything is initiated either to null, or to its value in the HA entity
for the actuators, each object contains its : type, status (won't be changed in the workflow), control_mode, classification (will be changed in the worklflow, and describes in each step what would be happen to the actuator), the run time, and the re evaluation 
in which we have those cases: 
first, if the control mode is SEMI-AUTO, the classifications are either ON (if the status is on), OFF (if the status if off), FUTURE-RE-EVALUATION (if the now time is in the interval of re evaluation, and in this case the actuator 'll be skipped in the workflow and won't be considered)
second,if the control mode is AUTO, there are 4 classifciations: WORKING (if it is on and running), PENDING (if it is off but 'll be running in a future time), FUTURE-RE-EVALUATION (the same logic as the semi-auto mode) (all of those classificaions force the actuator to be skipped and not considered) and the fourth case is FREE (the status is off and no action 'll be taking in the future)

REMARK: 
in the weather object, you'll find something called "feedback" (ie: [{"pump": ..., "fan": ...}], this is used in the final decisions agent to see how far to trust the weather forecast, in which the possible values are (usefull : if the weather info was usefull and matches the image processing info, useless: if contradicts the image processing info, neutral: if the actuator is being skipped (like the case of FUTURE-RE-EVALUATION, WORKING... (where the actuators are being skipped)))


### *2) Preliminary Decisions :*
this agent makes decisions on non-skipped actuators, before this agent, i made a filter to keep only the needed info to make pre decisions (on or off) which are : time, farm info, sensors 
the agent will modify the classification of the each non-skipped actuator, in which, new classification are made: TURN-ON, KEEP-OFF (for auto actuators) and TURN-ON, TURN-OFF, KEEP-ON, KEEP-OFF (for semi-auto actuators)

### *3) Weather Fetcher :*
this module fetches the weather and make a summary, give the state, the hourly data, insights (max & min) and the precipitation_probability

### *4) History Loader :*
brings the 7 last days history (for both runs and images) and make averages

### *5) Image Processing :*
whether give an image summary + flags, or refuse to take a picture (no image needed case)

### *6) Final Decisions :*
i made a code before that agent to filter only the data it needs, which is : time, farm, sensors, weather, history (only averages and 3 last runs (giving too much runs to the agent is not sufficient at all, it just needs to know the last runs as a pattern, and the averages for the week summaries)), image processing, and actuators (to see and modify on them)
the agent 'll receive only actuators that are going to be running (TURN-ON case), for the others (like KEEP-OFF..), he can't see them, and won't recommend anything for them
then, for the available actuators, it decides according to the given data, whether to run now, run later (here it updates the run object), or flag a NEED-RE-EVALUATION (modify the classification of the actuator) and set the interval for the re evaluation

the agent also make a general recommendation + updating the feedback (useful/useless/neutral(in case of re evaluation))

### *7) Sensor Updater :*
this make descriptions on the sensors (needed in the web app)

### *8) Alerts Updater :*
make description for the warnings, and set the severity
