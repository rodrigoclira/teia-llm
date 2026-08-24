# Instalação do VSCODE Dev


Seguindo os passos descritos no site [https://github.com/coder/code-server](https://github.com/coder/code-server)

```
curl -fsSL https://code-server.dev/install.sh | sh
```


## Configurando redirecionamento de porta no EC2

Você pode configurar o kernel do Linux para encaminhar todo o tráfego que chega na porta 80 diretamente para a porta padrão interna do code-server (8080).

Agora vamos redirecionar tanto a porta 80 (HTTP) quanto a 443 (HTTPS) para a porta interna 8080.
Redirecione a porta 443 (HTTPS):
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080
```

Redirecione a porta 80 (HTTP) se ainda não fez:
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

Torne a regra persistente

```bash
sudo apt install iptables-persistent -y
```

Para evitar que essas regras sumam quando a instância EC2 for reiniciada, salve-as utilizando o iptables-persistent:

```bash
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

Garanta que o code-server aceita conexões externas editando o arquivo `~/.config/code-server/config.yaml`:

```bash
bind-addr: 0.0.0.0:8080
```

Reinicie o code-server:
```bash
sudo systemctl restart code-server@ubuntu
```

Lembrar de liberar as portas security group do EC2