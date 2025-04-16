
# 1.Guia de Instalação

Este guia explica como clonar, configurar e executar o sistema **Titans Vision**, tanto usando **Docker** quanto compilando manualmente no **Ubuntu 22.04 LTS**.

---

## 🐳 Opção 1: Usando Docker (recomendado)

1. **Clone o repositório:**

   Abra o terminal e execute:
   ```bash
   git clone https://github.com/team-titans-unb/titans-vision
   ```

2. **Acesse o diretório do projeto:**
   ```bash
   cd titans-vision
   ```

3. **Construa a imagem Docker:**

   Este script cria a imagem Docker com todas as dependências configuradas.
   ```bash
   ./docker_build
   ```

4. **Execute o sistema:**

   Use o comando abaixo para rodar o sistema. Substitua `[ID da câmera]` pelo número correspondente à câmera (por exemplo, `0` para `/dev/video0`).
   ```bash
   ./docker_run [ID da câmera]
   ```

   🔎 **Nota:**  
   O ID da câmera pode ser encontrado em `/dev/video{id}`.  
   Se você não fornecer um ID, o sistema será executado **sem utilizar uma câmera** conectada.


![Tela inicial](assets/img/TelaTitansVision.png)

5. **Inicie a captura do campo:**

   Após a interface gráfica abrir, clique no botão **“Start Capture”** para iniciar a captura da visão do campo.

![Botão de Captura](assets/img/parte1.png)
---

## 🛠️ Opção 2: Compilação Manual (Ubuntu 22.04 LTS)

Caso prefira ou precise compilar o projeto manualmente, siga as instruções abaixo:

### 1. Instale as dependências necessárias

No diretório raiz do projeto, execute:
```bash
./InstallDependencies
```

Esse script instalará todas as bibliotecas e ferramentas exigidas pelo sistema.

### 2. Compile o projeto com CMake

No terminal:

```bash
# A partir da raiz do repositório
mkdir build
cd build
cmake ..
make
```

Esse processo criará os arquivos binários dentro do diretório `build`.

### 3. Execute o sistema

A partir da raiz do projeto, execute:

```bash
cd src
./VSS-VISION
```
