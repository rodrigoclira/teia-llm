# Instalando Ollama e Open WebUI

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/73d5e560-3a9e-4e15-849a-3ce372b506ee" />


Guia prático para instalar o Ollama e o Open WebUI em uma instância (por exemplo, uma EC2 Ubuntu), criando um playground local para experimentação com LLMs open source.

## O que é o Ollama

O Ollama é uma ferramenta que permite baixar, rodar e gerenciar LLMs open source (como Llama, Gemma, Mistral, entre outros) diretamente na própria máquina, sem depender de uma API paga na nuvem. Ele expõe os modelos tanto via linha de comando quanto via uma API HTTP local (`http://localhost:11434`), o que facilita integrar os modelos a outras aplicações, como o Open WebUI.

## Pré-requisitos

- Instância EC2: use a maior instância disponível no laboratório, `t3.large`. Modelos de LLM consomem bastante memória RAM e CPU, e instâncias menores (`t3.micro`, `t3.small`) tendem a travar ou rodar de forma muito lenta.
- Sistema operacional Linux (Ubuntu) com acesso sudo.
- No grupo de segurança da instância, libere a porta 80 (entrada, origem `0.0.0.0/0` ou o IP da sua rede) para acessar o Open WebUI pelo navegador.

## 1. Instalar o Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

O script instala o binário do Ollama e configura um serviço systemd (`ollama.service`) que roda em segundo plano, escutando por padrão em `127.0.0.1:11434`.

## 2. Baixar e testar modelos

O comando `ollama run` baixa o modelo (se ainda não existir localmente) e abre um chat interativo no terminal.

```bash
ollama run llama3.2:latest
```

```bash
ollama run llama3.2:1b
```

```bash
ollama run gemma2:2b
```
> Analise outros modelos disponíveis no site do ollama [https://ollama.com/search](https://ollama.com/search)


Modelos menores (como `llama3.2:1b` e `gemma2:2b`) exigem menos memória e CPU, úteis em instâncias sem GPU. Para sair do chat interativo, use `/bye`.

## 3. Testar a API do Ollama

Com o serviço rodando, é possível fazer requisições HTTP diretamente para a API local, sem passar pelo terminal interativo:

```bash
curl -X POST http://localhost:11434/api/generate -d '{
  "model": "gemma2:2b",
  "prompt": "Olá, tudo bem?"
}'
```

## 4. Expor o Ollama para a rede

Por padrão, o Ollama aceita apenas conexões locais (`127.0.0.1`). Para que o container do Open WebUI (ou outras máquinas) consiga acessá-lo, é preciso configurar a variável de ambiente `OLLAMA_HOST`.

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d && echo -e "[Service]\nEnvironment=\"OLLAMA_HOST=0.0.0.0\"" | sudo tee /etc/systemd/system/ollama.service.d/override.conf
```

Aplique a mudança:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Confirme que o serviço está escutando em todas as interfaces:

```bash
sudo ss -tulpn | grep 11434
```

Se aparecer `0.0.0.0:11434` ou `*:11434`, o Ollama está pronto para receber conexões externas.

## 5. Instalar o Docker

```bash
sudo apt update
sudo apt install docker.io -y
```

## 6. Instalar e rodar o Open WebUI

```bash
sudo docker run -d -p 80:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

O que cada opção faz:

- `-p 80:8080`: expõe a interface web na porta 80 do host.
- `--add-host=host.docker.internal:host-gateway`: permite que o container resolva `host.docker.internal` para o IP da máquina hospedeira, alcançando o Ollama que roda fora do container.
- `-v open-webui:/app/backend/data`: cria um volume persistente para dados do Open WebUI (conversas, configurações, usuários).
- `--restart always`: reinicia o container automaticamente após reboot ou falha.

## 7. Acessar o Open WebUI

Abra no navegador o IP público da instância na porta 80:

```
http://SEU_IP_PUBLICO
```

se não funcionar:

```
http://SEU_IP_PUBLICO:80
```

Crie a conta de administrador no primeiro acesso (esse cadastro fica salvo apenas localmente, no volume `open-webui`). Os modelos baixados via `ollama run` (llama3.2, gemma2, etc.) devem aparecer automaticamente na lista de modelos disponíveis para chat.

## 8. (Opcional) Usar a NVIDIA NIM no lugar da API da OpenAI

O Open WebUI também aceita conexões com qualquer API compatível com o padrão OpenAI. Uma alternativa gratuita (com créditos de uso) à API da OpenAI é a NVIDIA NIM, que disponibiliza diversos modelos hospedados pela NVIDIA.

1. Crie uma conta em [build.nvidia.com](https://build.nvidia.com/).
2. Abra a página de um modelo (por exemplo, Llama 3.3 70B Instruct) e clique em "Get API Key" para gerar uma chave no formato `nvapi-xxxxxxxxxxxxxxxx`. Contas novas recebem um saldo inicial de créditos de inferência.
3. No Open WebUI, vá em Configurações (Settings) > Conexões (Connections) > OpenAI API e adicione uma nova conexão com:
   - URL base: `https://integrate.api.nvidia.com/v1`
   - Chave de API: a chave `nvapi-` gerada no passo anterior
4. Salve e selecione um dos modelos da NVIDIA NIM na lista de modelos do chat.

Essa opção é útil para comparar modelos maiores (que não rodam bem na instância local) com os modelos rodando via Ollama, sem precisar de uma chave da OpenAI.

## Fontes

- Rhian Lopes, "Desvendando o Ollama: construindo um playground com Ollama e Open WebUI para experimentação de LLMs", Medium (roteiro de comandos original desta instalação).
- [Documentação oficial do Ollama, FAQ](https://docs.ollama.com/faq)
- [Open WebUI, Quick Start](https://docs.openwebui.com/getting-started/quick-start/)
- [Open WebUI, repositório oficial no GitHub](https://github.com/open-webui/open-webui)
- [NVIDIA NIM, catálogo de modelos e API](https://build.nvidia.com/)
