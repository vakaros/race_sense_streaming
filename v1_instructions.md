# RaceSense Streaming v1 Instructions


## How to Connect
1. **Get required information**  
    We'll need to gather a few pieces of information before we start.  
    First is the API key, which will be provided to you via email.  
    Second is the "streaming ID", this can be found within the Race Control app, on the "Race Committee Dashboard". Click the share button in the upper right: ![](/assets/htc_1-1.jpg)  
    Then in the dialog that opens copy the "streaming ID": ![](/assets/htc_1-2.jpg)  
    Streaming IDs are unique for a division, so this will need to be done again for any new divisions that're created.  
    
2. **Acquire Connection Bundle from API**
    Now you'll get the Connection Bundle, which will contain:  
    - MQTT Hostname  
    - JSON Web Token (JWT)
    - MQTT Topic

    The request should look something like this:  
    ```sh
    curl -H "X-Custom-PSK: $API_KEY_GOES_HERE" -H "STREAMING_ID: W3HAUxuP0plcNcZFZNek/Test" https://connection-bundle.vakaros.workers.dev/
    ```
    Optionally you can also include a previously issued JWT be including it in the header like so:
    ```sh
    -H "EXISTING_JWT: $EXISTING_JWT_HERE"
    ```

    This will return some JSON with the following fields:
    - `mqttHostname`
    - `jwt`
    - `mqttTopic`

3. **Connect to MQTT**  
    Now you can connect to the MQTT broker and start receiving data. This can be done using an MQTT client like MQTTX, or programmatically. Regardless you'll want to set things up like so:
    - Hostname/Broker URL: `mqtts://$MQTT_HOSTNAME_FROM_BUNDLE`
    - Port: 8883
    - Username: $YOUR_EMAIL
    - PASSWORD: $JWT_FROM_BUNDLE
    - MQTT Version: 5

    Once a connection is established you can subscribe to the topic that was provided in the Connection Bundle. If the tablet running Race Control is connected to the internet and running an event you should start to receive frames within 10 seconds of Connecting, and then every second thereafter.


## Data Format
Everyone second or so you'll receive a new frame over MQTT with the following format:
Note: Any fields not documented are either unused or are for internal use only
### Data Frame
- `timestamp`: An ISO-8601 string representing when this frame happened
- `participants`: A map of device serial number to participant update ([see below](#participant-update) for information on that format.)
- `flags`: The flags which are currently shown. A list of all flags can be found [here](#flags).

### Participant Update
- `sn`: Serial number of the device
- `pos`: The coordinates of this participant, as an array, where latitude is the first element, and longitude is the second. This can be null
- `hdgDeg`: The heading of the participant, in degrees.
- `statusBitField`: Bitfield indicating the status of this participant. Currently only the first bit is used, to indicate if the participant is OCS or not.
- `timestamp`: An ISO-8601 string representing when this update was reported by the participant's device
- `role`: The role of this participant within the network, valid values are: `competitor`, `markStartL`, `markStartR`, `repeater`, `observer`.
- `speed`: The speed at which this participant is moving, in knots
- `finish`: Stats related to the boat's finish in this most recent race, [see below](#finishing-stats). Can be null

### Flags
The current flags in use:
- l
- ap
- n
- firstSubstitute
- p
- classFlag
- m
- i
- z
- u
- black
- countdown1
- countdown2
- countdown3
- countdown4
- countdown5

### Finishing Stats
- `sailNumber`: Sail number for which these finishing stats are for
- `maxSpeed`: The max speed recorded during the race, in knots
- `distanceTraveled`: The distance traveled during the race, in nautical miles
- `finishingTime`: An ISO-8601 string representing the time that this participant finished the race


