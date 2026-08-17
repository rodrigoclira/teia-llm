# Gerando o token do Hugging Face

Necessário a partir da **E01** para as chamadas a LLM do laboratório. É gratuito e leva ~3 minutos.

O token é usado com os **Inference Providers** do HF: uma API única que roteia a chamada para
provedores parceiros (Groq, Together, Cerebras, Novita...), no formato compatível com OpenAI.

---

## 1. Criar a conta

<https://huggingface.co/join> — e-mail, usuário e senha. Confirme o e-mail antes de seguir; sem
confirmar, a criação do token falha.

Se já tem conta, pule para o passo 2.

## 2. Gerar o token

Use o link abaixo — ele já abre o formulário com o tipo e a permissão certos marcados:

**<https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained>**

Na tela:

1. **Token name** — dê um nome que identifique o uso, ex.: `colab-teia-llm`.
   (A recomendação do HF é um token por aplicação: se vazar, você revoga só aquele.)
2. Confirme que o tipo está em **Fine-grained**.
3. Confirme que a permissão **`Make calls to Inference Providers`** está marcada. É ela que
   autoriza a chamada ao LLM — sem ela, a resposta é `401`.
4. Clique em **Create token**.

> Chegando pelo menu em vez do link: avatar (canto superior direito) → **Access Tokens** →
> botão de criar novo token. Aí você precisa marcar a permissão do item 3 na mão.

## 3. Copiar o token — só aparece uma vez

O token começa com `hf_...`. Copie **agora**: ao fechar o modal, o valor não é mais exibido.

Perdeu? Não tem recuperação — apague e gere outro. Não custa nada.

## 4. Guardar o token no Colab

Nunca cole o token numa célula de código. Notebook vai para o GitHub, e token no GitHub é token
cancelado (o HF varre repositórios públicos e revoga automaticamente).

No notebook do Colab:

1. Clique no ícone de **chave 🔑** na barra lateral esquerda (*Secrets*).
2. **`+ Adicionar novo secret`**.
3. **Nome:** `HF_TOKEN` — exatamente assim, maiúsculas, sem espaço.
4. **Valor:** cole o `hf_...`.
5. Ligue a chavinha **Acesso ao notebook**. É o passo mais esquecido da lista.

O notebook lê o secret assim:

```python
from google.colab import userdata
userdata.get("HF_TOKEN")
```

## 5. Testar

Rode a célula da Geração 3 no notebook do laboratório. Se ela imprimir uma frase respondendo
"o que é NLP?", está tudo certo.

---

## Quando dá errado

| Erro | O que é | O que fazer |
|---|---|---|
| `401 Unauthorized` | Token inválido, ou sem a permissão de Inference Providers | Confira o passo 2, item 3. Na dúvida, gere um token novo pelo link |
| `SecretNotFoundError` / valor vazio | O secret não existe ou está com "Acesso ao notebook" desligado | Passo 4, itens 3 e 5. Confira o nome: `HF_TOKEN` |
| `402` ou mensagem de crédito | Crédito mensal gratuito esgotado | Espere o ciclo virar, ou troque `PROVEDOR = "gemini"` no notebook |
| `404` no nome do modelo | Aquele modelo não está sendo servido por nenhum provedor no momento | Veja abaixo |

### Conferir se o modelo está sendo servido

O catálogo muda toda hora — um modelo servido no semestre passado pode não estar hoje. Para ver
os que estão ativos agora:

- Na web: <https://huggingface.co/models?inference_provider=all&other=conversational>
- No terminal: `hf models ls --warm`

O notebook usa `Qwen/Qwen2.5-7B-Instruct`. Se ele não estiver disponível, `openai/gpt-oss-120b` é
a alternativa que a própria documentação do HF usa nos exemplos. Basta trocar o valor de `MODELO`.

### Se o token vazar

Apague em <https://huggingface.co/settings/tokens> e gere outro. Se você publicou num repositório,
apagar o commit **não** basta — o token já foi indexado. Revogue.

---

## Observações para a disciplina

- O crédito gratuito de Inference Providers é limitado (o HF não publica o valor exato e ele muda).
  Nosso laboratório faz **2 chamadas**, então cabe com folga — mas não deixe um `for` chamando o
  modelo 500 vezes sem olhar.
- Um token de tipo **Read** também funciona para inferência, segundo a documentação do HF. O
  fine-grained é preferível porque limita o estrago em caso de vazamento.
- Alternativa ao HF: chave do Google AI Studio (<https://aistudio.google.com/apikey>), guardada
  como secret `GOOGLE_API_KEY`. No notebook, troque `PROVEDOR = "gemini"`.

## Referências

| Recurso | Link |
|---|---|
| HF — User Access Tokens | https://huggingface.co/docs/hub/security-tokens |
| HF — Inference Providers | https://huggingface.co/docs/inference-providers |
| Modelos servidos agora | https://huggingface.co/models?inference_provider=all |
| Google AI Studio (alternativa) | https://aistudio.google.com/apikey |
