---
# Instalação do OpenCode com NVIDIA NIM (API gratuita) em Ubuntu Linux na AWS EC2


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

3. Escolha o tipo de instância `t3.small` e o seu key pair (`.pem`).

> 💡 **Importante**: a `t3.small` tem apenas 1 GB de RAM. O OpenCode (Node.js + TUI) pode estourar esse limite — veja o Passo 6 para configurar o swap e evitar o processo ser morto pelo sistema ("Killed").

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

## Passo 5 — Autenticar na API da NVIDIA via `/connect`

O OpenCode usa o catálogo de providers do **models.dev**, que já inclui a **NVIDIA NIM** com seus modelos gratuitos. Não é necessário criar arquivo de configuração — basta autenticar pelo próprio TUI:

1. Inicie o OpenCode:

```bash
opencode
```

2. Dentro do TUI, digite o comando:

```
/connect
```

3. Na lista de provedores, escolha **NVIDIA**.

4. Quando solicitado, cole a sua API key (`nvapi-SUA-CHAVE-AQUI`).

5. Agora selecione o modelo padrão:

```
/models
```

6. Escolha `deepseek-ai/deepseek-v4.1-flash` (1M de contexto, tool calling, gratuito).

> A credencial fica salva no arquivo `auth.json` do OpenCode (não é commitada). Para desconectar, use `/logout` e selecione NVIDIA.

---

## Passo 6 — Testar e usar o OpenCode

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
