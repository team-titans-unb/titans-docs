# 4. Receiving Data

### 1. Download the Required Files
Some files are necessary for the system to function properly.

Download the following files and place them in the `pb` folder of your project:

1. [wrapper.proto](https://github.com/team-titans-unb/titans-vision/blob/main/client/python/pb/wrapper.proto)
2. [messages_robocup_ssl_detection.proto](https://github.com/team-titans-unb/titans-vision/blob/main/client/python/pb/messages_robocup_ssl_detection.proto)
3. (optional) [messages_robocup_ssl_geometry.proto](https://github.com/team-titans-unb/titans-vision/blob/main/client/python/pb/messages_robocup_ssl_geometry.proto)
4. [compile.sh](https://github.com/team-titans-unb/titans-vision/blob/main/client/python/pb/compile.sh)

> 🔍 **Note:** About **GeometryData**: 
> It is responsible for storing field geometry data, such as point positions and field dimensions. Since we are not using this information, you can remove the following line from the `wrapper.proto` file:
> ```proto
> optional SSL_GeometryData geometry = 2;
> ```

### 2. Compiling the `.proto` Files
The `.proto` files are Protobuf message definition files. To generate the `.py` files used by the system, you need to compile these files.

To do this, run the `compile.sh` script inside the `pb` folder of your project:

```bash
cd pb
sh compile.sh
```

### 3. Initializing the Client
Now that you have the generated `.py` files, you can initialize the client. The code below shows how to set up and initialize the client that communicates with the vision system.

```python
import socket
import struct

class VisionClient:
    def __init__(self, vision_ip="224.5.23.2", vision_port=10015):
        # Vision IP and Port settings
        self.vision_ip = vision_ip
        self.vision_port = vision_port
        
        # Create the UDP socket for communication with vision
        self.vision_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        
        # Set socket options
        self.vision_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.vision_sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 128)
        self.vision_sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_LOOP, 1)
        
        # Configure to join multicast group
        self.vision_sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, struct.pack("=4sl", socket.inet_aton(self.vision_ip), socket.INADDR_ANY))
        
        # Bind to listen to the vision port
        self.vision_sock.bind((self.vision_ip, self.vision_port))

        print(f"Vision client started. Waiting for data at IP: {self.vision_ip}, Port: {self.vision_port}")

# Initialize the vision client
client = VisionClient()
```

### 4. Receiving Data

Now that the client is initialized, you can start receiving data from the vision system. The code below shows how to receive and process the data.

``` python
from pb import wrapper_pb2 as wr

class VisionDataReceiver:
    def __init__(self, vision_client):
        self.vision_sock = vision_client.vision_sock

    def receive_frame(self):
        """Receive and decode the packet."""
        data = None
        while True:
            try:
                data, _ = self.vision_sock.recvfrom(1024)
            except Exception as e:
                print(f"Error receiving data: {e}")
            if data is not None:
                break

        if data is not None:
            try:
                # Unpack the SSL_WrapperPacket
                frame = wr.SSL_WrapperPacket().FromString(data)
                
                # Process robots and balls
                robots_blue = [(robot.robot_id, robot.x, robot.y, robot.orientation) for robot in frame.detection.robots_blue]
                robots_yellow = [(robot.robot_id, robot.x, robot.y, robot.orientation) for robot in frame.detection.robots_yellow]
                
                balls = [(b.x, b.y, b.confidence) for b in frame.detection.balls]

                # Return full structure with robot and ball data
                return {
                    "robots_blue": robots_blue,
                    "robots_yellow": robots_yellow,
                    "balls": balls
                }

            except Exception as e:
                print(f"Error processing frame: {e}")
        return None
```

### 5. Example Usage
To use the `VisionDataReceiver` class, you can do the following:

```python
vision_client = VisionClient()

while True:
    data_receiver = VisionDataReceiver(vision_client)
    frame_data = data_receiver.receive_frame()

    if frame_data:
        print("Blue Robots: ", frame["robots_blue"])
        print("Yellow Robots: ", frame["robots_yellow"])
        print("Balls: ", frame["balls"])
    else:
        print("Error receiving data.")
