# Instalando o llama.cpp

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/798ea31e-9d9e-46eb-9bdd-3ce9480333c8" />

O llama.cpp é um projeto de código aberto escrito em C e C++ para execução eficiente de modelos de linguagem (LLMs) em CPU e, opcionalmente, em GPU. Ele foi criado originalmente para rodar o modelo LLaMA da Meta em hardware modesto, sem depender de bibliotecas pesadas como PyTorch, e hoje suporta uma grande variedade de arquiteturas de modelos abertos.

Para a nossa disciplina, o llama.cpp é relevante por três motivos:

1. Ele permite rodar LLMs localmente, sem custo de API, o que se encaixa na exigência de usarmos apenas ferramentas gratuitas.
2. Ele funciona bem em instâncias EC2 modestas, já que os modelos são carregados no formato GGUF, um formato quantizado que reduz drasticamente o consumo de memória e processamento.
3. Ele permite baixar modelos diretamente do Hugging Face Hub por linha de comando, o que facilita a reproducibilidade das aulas.

Fonte oficial do projeto: https://github.com/ggml-org/llama.cpp

## Observação importante antes de começar

O comando de instalação de dependências abaixo faz referência ao pacote `gcc-16`. No momento em que este material foi escrito, a versão estável mais recente do GCC disponibilizada nos repositórios padrão do Ubuntu é anterior a 16. Antes da aula, verifique com `apt-cache search gcc-` quais versões estão disponíveis na sua instância EC2 e ajuste o comando se necessário. Na maioria dos casos, o pacote genérico `gcc` (sem número de versão) é suficiente para compilar o projeto.

## Passo 1: Atualizar a lista de pacotes

```bash
sudo apt-get update
```

Este comando não instala nem atualiza nenhum programa. Ele apenas sincroniza a lista local de pacotes disponíveis com os repositórios configurados no sistema (os "servidores" de onde o Ubuntu baixa programas). É uma boa prática rodar este comando antes de qualquer instalação, para garantir que estamos instalando as versões mais recentes disponíveis e evitar erros de pacote não encontrado.

## Passo 2: Instalar as dependências de compilação

```bash
sudo apt-get install -y g++ gcc-16 libssl-dev cmake
```

O llama.cpp é distribuído como código-fonte em C e C++, então precisamos compilá-lo antes de usá-lo. Este comando instala três dependências:

- `gcc`: o compilador de C, responsável por traduzir código C em instruções executáveis.
- `g++`: o compilador de C++, necessário porque parte do llama.cpp é escrita em C++.
- `libssl-dev`: os arquivos de desenvolvimento da biblioteca OpenSSL, que permitem que o llama.cpp se conecte a servidores HTTPS. Isso é o que possibilita baixar modelos diretamente do Hugging Face Hub usando a flag `-hf`, sem precisar baixar o arquivo manualmente.
- `cmake`: ferramenta de código aberto e multiplataforma usada para gerenciar e automatizar o processo de construção (build) de software de forma independente do compilador

A flag `-y` responde "sim" automaticamente a qualquer confirmação pedida pelo instalador, o que é útil em scripts e em aulas com tempo limitado.

## Passo 3: Clonar o repositório do projeto

```bash
git clone https://github.com/ggml-org/llama.cpp
```

Este comando baixa (clona) todo o código-fonte do llama.cpp do GitHub para a instância EC2, criando uma pasta local chamada `llama.cpp` com o histórico completo do repositório. É o equivalente a baixar o código-fonte de um programa que ainda precisa ser compilado.

## Passo 4: Entrar na pasta do projeto

```bash
cd llama.cpp/
```

Muda o diretório de trabalho do terminal para dentro da pasta que acabou de ser criada pelo clone. Todos os comandos seguintes assumem que estamos dentro desta pasta, pois é ali que estão os arquivos de configuração de build (`CMakeLists.txt`).

## Passo 5: Configurar o build com o CMake

```bash
cmake -B build -DLLAMA_OPENSSL=ON
```

O CMake é uma ferramenta que gera os arquivos necessários para compilar um projeto C/C++ em diferentes sistemas operacionais, sem que o desenvolvedor precise escrever esses arquivos manualmente para cada plataforma.

- `-B build`: instrui o CMake a criar uma pasta chamada `build` e colocar ali todos os arquivos gerados pelo processo de configuração, mantendo o código-fonte original limpo.
- `-DLLAMA_OPENSSL=ON`: ativa a opção de suporte a OpenSSL no projeto, que é a que permite o download de modelos via HTTPS diretamente do Hugging Face Hub (dependência que instalamos no Passo 2 com o `libssl-dev`).

Este passo apenas prepara a compilação, ainda não gera nenhum programa executável.

## Passo 6: Compilar o projeto

```bash
cmake --build build --config Release -j 3
```

Este é o comando que efetivamente compila o código-fonte, transformando-o em binários executáveis.

- `--build build`: informa ao CMake que a compilação deve usar os arquivos gerados na pasta `build` no passo anterior.
- `--config Release`: compila na configuração "Release", que aplica otimizações de desempenho e remove informações de depuração, resultando em um programa mais rápido (em oposição à configuração "Debug", usada por desenvolvedores durante a correção de erros).
- `-j 3`: define o número de processos de compilação em paralelo. O número 3 deve ser ajustado de acordo com a quantidade de vCPUs disponíveis na instância EC2 usada em aula (por exemplo, em uma instância com 4 vCPUs, `-j 4` tende a ser mais eficiente). Um valor muito alto em instâncias com pouca memória RAM pode causar falhas de compilação por falta de memória.

Este passo costuma ser o mais demorado do processo, podendo levar de alguns minutos a mais de meia hora, dependendo do tamanho da instância.

## Passo 7: Baixar um modelo e testar a inferência

```bash
./build/bin/llama-cli -hf unsloth/Qwen3.5-2B-GGUF:UD-Q4_K_XL -p "What is a LLM?" -n 1000 --reasoning off
```

Este comando executa o binário `llama-cli`, gerado no Passo 6, para carregar um modelo e gerar uma resposta a partir de um prompt.

- `-hf unsloth/Qwen3.5-2B-GGUF:UD-Q4_K_XL`: instrui o programa a baixar o modelo diretamente do Hugging Face Hub, no formato `usuario/repositorio:tag_de_quantizacao`. Caso o modelo ainda não esteja em cache local, ele será baixado automaticamente antes da execução. A parte após os dois-pontos (`UD-Q4_K_XL`) indica a variante de quantização a ser usada, isto é, o nível de compressão aplicado aos pesos do modelo.
- `-p "What is a LLM?"`: define o prompt inicial enviado ao modelo.
- `-n 1000`: define o número máximo de tokens a serem gerados na resposta.
- `--reasoning off`: desativa o modo de raciocínio estendido (chain-of-thought) em modelos que suportam essa funcionalidade, fazendo com que o modelo vá direto para a resposta final.

Antes da aula, verifique se o repositório e a tag de quantização informados no comando ainda existem no Hugging Face Hub, acessando a página do modelo (por exemplo, https://huggingface.co/unsloth), pois nomes e tags de repositórios de modelos são alterados com frequência pelos mantenedores. Como alternativa testada e disponível publicamente, o modelo `ggml-org/gemma-3-1b-it-GGUF` pode ser usado como opção de contingência em sala de aula.

---

## Atividade proposta

**Objetivo:** familiarizar os estudantes com o processo de compilação e execução local de um LLM quantizado, e com o impacto prático da quantização no desempenho.

**Enunciado:**

1. Reproduza os Passos 1 a 6 deste guia na sua instância EC2, registrando o tempo total de compilação.
2. Escolha um modelo pequeno (até 3 bilhões de parâmetros) disponível no Hugging Face Hub no formato GGUF, e execute o Passo 7 utilizando pelo menos duas variantes de quantização diferentes do mesmo modelo (por exemplo, `Q4_K_M` e `Q8_0`).
3. Para cada variante, registre em uma tabela:
   - o tamanho do arquivo do modelo baixado;
   - o tempo aproximado para gerar a resposta;
   - a qualidade percebida da resposta, em uma escala de 1 a 5, com uma breve justificativa.
4. Escreva um parágrafo curto relacionando os resultados observados com o conceito de quantização apresentado em aula, explicando o compromisso entre tamanho do modelo, velocidade de inferência e qualidade da saída.
5. Envie a tabela e o parágrafo em um arquivo Markdown ou PDF.

Esta atividade não exige GPU nem qualquer chave de API paga, sendo compatível com o requisito de uso exclusivo de ferramentas gratuitas do curso.

---

**Fonte:** documentação oficial do projeto llama.cpp, disponível em https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md
