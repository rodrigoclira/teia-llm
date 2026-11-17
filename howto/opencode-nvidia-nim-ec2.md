---
# Instalação do OpenCode com NVIDIA NIM (API gratuita) em Ubuntu Linux na AWS EC2


> **Visão geral**: Este guia instala o **OpenCode** (agente de codificação open-source, alternativa ao Claude Code/Codex) configurado com a **API gratuita da NVIDIA NIM** — 100% sem custo para estudantes. O custo total da infraestrutura é R$ 0 dentro do free tier da AWS.

Se você estiver usando o EC2, siga o passo a passo abaixo. Caso você esteja utilizando outra infraestrutura, pule para o Passo 5, onde é exibida a instalação do OpenCode.

---

## Passo 1 — Criar a API Key gratuita da NVIDIA

1. Acesse **https://build.nvidia.com** e clique em **"Sign in" / "Join"**.
2. Crie a conta com seu e-mail — isso te inscreve automaticamente no **NVIDIA Developer Program gratuito** (sem cartão de crédito, sem empresa).
3. Clique em **"Get API Key"** (canto superior direito da página) e depois em **"Generate Key"**.
4. Copie a chave gerada (formato `nvapi-xxxx...`) e **guarde-a** — ela só é exibida uma única vez.

> Você recebe aproximadamente **1.000 créditos de inferência** (1 crédito ≈ 1 chamada de API) e um limite de **40 requisições por minuto**. Suficiente para uso acadêmico/estudantil.

---

## Passo 2 — Criar a instância EC2

1. Acesse o console AWS → **EC2** → **Launch instance**.

2. Indique um nome, por exemplo: `opencode-agent`, e escolha a opção de SO: **Ubuntu Server 24.04 LTS**.

3. Escolha o tipo de instância `t3.micro` (gratuito no free tier) e o seu key pair (`.pem`).

> 💡 **Importante**: a `t3.micro` tem apenas 1 GB de RAM. O OpenCode (Node.js + TUI) pode estourar esse limite — veja o Passo 6 para configurar o swap e evitar o processo ser morto pelo sistema ("Killed").

4. Em **Configuração da Rede** (configuração do firewall/security group), libere a porta **SSH (22)**.

5. Na configuração de **armazenamento**, aumente o HD para **20 GB** (gp3).

6. Clique em **Launch instance** e aguarde o estado `running`.

7. Conecte-se via SSH:

```bash
chmod 400 sua-chave.pem
ssh -i sua-chave.pem ubuntu@<IP-PUBLICO-DA-INSTANCIA>
```

---

## Passo 3 — Instalar o Node.js 22+

O OpenCode é distribuído via npm e requer Node.js recente. O repositório padrão do Ubuntu 24.04 traz o Node 18 (desatualizado), então use o repositório NodeSource 22.x:

```bash
echo "Installing Node.js 22..."

sudo apt update && sudo apt upgrade -y

curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

echo "Node version:"
node -v   # deve exibir v22.x ou superior
```

---

## Passo 4 — Instalar o OpenCode

```bash
echo "Installing OpenCode..."

sudo npm install -g opencode-ai@latest

echo "OpenCode version:"
opencode --version
```

---

## Passo 5 — Configurar a API key da NVIDIA

Defina a chave como variável de ambiente (persistente) e crie o arquivo de configuração do OpenCode:

```bash
# 1) Define a chave como variável de ambiente (permanente)
echo 'export NVIDIA_API_KEY="nvapi-SUA-CHAVE-AQUI"' >> ~/.bashrc
source ~/.bashrc

# 2) Cria o diretório de configuração
mkdir -p ~/.config/opencode

# 3) Cria o config com o provider NVIDIA NIM
cat > ~/.config/opencode/opencode.jsonc << 'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "nvidia": {
      "name": "NVIDIA NIM",
      "env": ["NVIDIA_API_KEY"],
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "https://integrate.api.nvidia.com/v1"
      },
      "models": {
        "deepseek-ai/deepseek-v4.1-flash": {
          "name": "DeepSeek V4.1 Flash (1M ctx)"
        },
        "moonshotai/kimi-k2.6": {
          "name": "Kimi K2.6"
        },
        "z-ai/glm-5.3": {
          "name": "GLM-5.3"
        },
        "minimaxai/minimax-m3.1-flash": {
          "name": "MiniMax M3.1 Flash"
        }
      }
    }
  },
  "model": "nvidia/deepseek-ai/deepseek-v4.1-flash"
}
EOF
```

> O ID do modelo usa o formato `provider/modelID` (ex.: `nvidia/deepseek-ai/deepseek-v4.1-flash`). A configuração acima já define o modelo padrão.

---

## Passo 6 — Configurar swap (evitar o processo ser morto por falta de memória)

A instância `t3.micro` possui apenas 1 GB de RAM. Sem swap, o sistema operacional mata o processo OpenCode com a mensagem `Killed opencode:` (OOM Killer). Adicione 2 GB de swap:

```bash
echo "Configuring 2GB swap..."

sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Torna permanente (sobrevive a reboot)
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

echo "Memory status:"
free -m   # deve exibir "Swap: 2047"
```

> Para confirmar se o problema era falta de memória: `sudo dmesg | grep -i -E "oom|killed"`.

---

## Passo 7 — Testar e usar o OpenCode

### Teste rápido (one-shot, sem TUI)

```bash
opencode run "Responda exatamente: OPENCODE_SMOKE_OK"
# deve imprimir OPENCODE_SMOKE_OK
```

### Sessão interativa (TUI)

```bash
mkdir ~/meu-projeto && cd ~/meu-projeto
opencode
```

### Comandos úteis dentro do TUI

| Comando | Ação |
|---------|------|
| `/models` (ou Ctrl+X M) | Trocar de modelo |
| `/agents` (ou Tab) | Alternar entre agentes build/plan |
| `/sessions` (ou Ctrl+X L) | Gerenciar sessões |
| `Ctrl+C` | Sair (**`/exit` não é comando válido**) |

### One-shot fora do TUI

```bash
opencode run "Crie um CRUD em Flask com SQLite" --model nvidia/deepseek-ai/deepseek-v4.1-flash
opencode run "Revise este arquivo" -f config.yaml
```

---

## Passo 8 — Manter a sessão ativa (recomendado para servidor headless)

Use `tmux` para o OpenCode continuar rodando quando o SSH for fechado:

```bash
sudo apt install -y tmux
tmux new -s opencode
opencode   # roda dentro do tmux
# desconectar: Ctrl+B depois D
# voltar: tmux attach -t opencode
```

---

## Modelos recomendados (gratuitos no NVIDIA NIM)

| Modelo | Contexto | Observação |
|--------|----------|------------|
| `deepseek-ai/deepseek-v4.1-flash` | 1M tokens | **Padrão recomendado**: rápido, tool calling ✓, ótimo custo-benefício |
| `moonshotai/kimi-k2.6` | 262K tokens | Bom para tarefas complexas |
| `z-ai/glm-5.3` | 1M tokens | Alto contexto, mais lento |
| `minimaxai/minimax-m3.1-flash` | 262K tokens | Flash, rápido |

## Limitações do plano gratuito

- **Créditos**: ~1.000 chamadas iniciais (1 crédito ≈ 1 chamada). Ao esgotar, crie outra conta ou adquira créditos.
- **Rate limit**: 40 requisições/minuto — suficiente para uso acadêmico.
- **Latência**: modelos grandes são mais lentos; prefira o `deepseek-v4.1-flash` no dia a dia.
- **Segurança**: rode como usuário `ubuntu` (nunca `root`); o OpenCode executa comandos no sistema com as permissões do usuário logado.

# Referências

- https://opencode.ai/docs
- https://opencode.ai/docs/providers/
- https://build.nvidia.com
- https://github.com/anomalco/opencode
