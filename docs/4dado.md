# 4. Recebendo os Dados

### 1. Baixe os Arquivos Necessários
Alguns arquivos são necessários para o funcionamento do sistema.

Faça o download dos seguintes arquivos e coloque-os na pasta `pb` do seu projeto:

1. [wrapper.proto](https://github.com/team-titans-unb/titans-vision/blob/main/client/python/pb/wrapper.proto)
2. [messages_robocup_ssl_detection.proto](https://github.com/team-titans-unb/titans-vision/blob/main/client/python/pb/messages_robocup_ssl_detection.proto)
3. (opcional) [messages_robocup_ssl_geometry.proto](https://github.com/team-titans-unb/titans-vision/blob/main/client/python/pb/messages_robocup_ssl_geometry.proto)
4. [compile.sh](https://github.com/team-titans-unb/titans-vision/blob/main/client/python/pb/compile.sh)

> 🔍 **Nota:** Sobre o **GeometryData**:
> Ele é responsável por armazenar os dados de geometria do campo, como a posição dos pontos e as dimensões do campo. Como não estamos utilizando essa informação, você pode remover a seguinte linha do arquivo `wrapper.proto`:
> ```proto
> optional SSL_GeometryData geometry = 2;
> ```

### 2. Compilando os Arquivos `.proto`
Os arquivos `.proto` são arquivos de definição de mensagens do Protobuf. Para gerar os arquivos `.py` que serão usados no sistema, você precisa compilar esses arquivos.

Para isso, execute o script `compile.sh` na pasta `pb` do seu projeto:

```bash
cd pb
sh compile.sh
```

### 3. Inicializando o Cliente
Agora que você já tem os arquivos `.py` gerados, pode inicializar o cliente. O código a seguir mostra como configurar e inicializar o cliente que irá se comunicar com o sistema de visão.

```python
import socket
import struct

class VisionClient:
    def __init__(self, vision_ip="224.5.23.2", vision_port=10015):
        # Configurações do IP e Porta de visão
        self.vision_ip = vision_ip
        self.vision_port = vision_port
        
        # Criação do socket UDP para comunicação com a visão
        self.vision_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        
        # Definindo opções do socket
        self.vision_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.vision_sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 128)
        self.vision_sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_LOOP, 1)
        
        # Configuração para ingressar no grupo multicast
        self.vision_sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, struct.pack("=4sl", socket.inet_aton(self.vision_ip), socket.INADDR_ANY))
        
        # Bind para escutar a porta de visão
        self.vision_sock.bind((self.vision_ip, self.vision_port))

        print(f"Cliente de visão iniciado. Aguardando dados no IP: {self.vision_ip}, Porta: {self.vision_port}")

# Inicializando o cliente de visão
client = VisionClient()
```

### 4. Recebendo os Dados

Agora que o cliente está inicializado, você pode começar a receber os dados do sistema de visão. O código a seguir mostra como receber e processar os dados.

``` python
from pb import wrapper_pb2 as wr

class VisionDataReceiver:
    def __init__(self, vision_client):
        self.vision_sock = vision_client.vision_sock

    def receive_frame(self):
        """Receber e decodificar o pacote."""
        data = None
        while True:
            try:
                data, _ = self.vision_sock.recvfrom(1024)
            except Exception as e:
                print(f"Erro ao receber dados: {e}")
            if data is not None:
                break

        if data is not None:
            try:
                # Descompacta o pacote SSL_WrapperPacket
                frame = wr.SSL_WrapperPacket().FromString(data)
                
                # Processando robots e bolas
                robots_blue = [(robot.robot_id, robot.x, robot.y, robot.orientation) for robot in frame.detection.robots_blue]
                robots_yellow = [(robot.robot_id, robot.x, robot.y, robot.orientation) for robot in frame.detection.robots_yellow]
                
                balls = [(b.x, b.y, b.confidence) for b in frame.detection.balls]

                # Retorna a estrutura completa com os dados dos robots e bolas
                return {
                    "robots_blue": robots_blue,
                    "robots_yellow": robots_yellow,
                    "balls": balls
                }

            except Exception as e:
                print(f"Erro ao processar o frame: {e}")
        return None
```

### 5. Exemplo de Uso
Para usar a classe `VisionDataReceiver`, você pode fazer o seguinte:

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
        print("Erro ao receber os dados.")
```