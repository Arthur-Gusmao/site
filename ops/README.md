# Servidor do site

```text
PR → GitHub Actions → build rootless + testes HTTP
master → mesmos testes → GHCR (imagem com SHA do commit)
notebook → cron a cada minuto → stage → testes → Caddy reload
visitante → Cloudflare Tunnel → Caddy → container rootless
```

O usuário `werc` executa Podman sem root e sem acesso à sessão gráfica.
O root só instala os serviços e troca a configuração do Caddy, usando
scripts locais de propriedade do root. O servidor não executa scripts
baixados do GitHub. Não precisa de runner local, webhook, chave SSH
de deploy ou secret do GitHub. A imagem GHCR deve ser pública.

## Instalar no Alpine/OpenRC

```sh
git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
cd site
doas sh ops/werc-oci-install
```

O instalador cria ou reutiliza `werc`, aloca subUIDs/subGIDs livres,
instala as dependências e os scripts, habilita OpenRC e adiciona um único
job ao cron. Pode ser executado novamente. Preserva o Caddyfile existente
e guarda os scripts anteriores em `/etc/werc-oci/install-backup.*`.

Configure o Cloudflare Tunnel uma vez: salve o token em
`/etc/cloudflared/token` com proprietário root e permissão `0600`.
A rota de `arthurgusmao.com.br` deve apontar para `http://127.0.0.1:8080`.
O token fica somente no servidor; nunca coloque no Git.

Em um servidor novo, use `ops/Caddyfile` com a porta inicial `8081`
(o instalador faz isso quando o arquivo não existe). É apenas o estado
inicial; não precisa executar lighttpd no host. Depois:

```sh
doas rc-service caddy start
doas werc-oci-deploy
doas rc-service werc-oci start
doas rc-service cloudflared start
```

O primeiro deploy precisa que o túnel esteja conectado para verificar
o domínio público. Se ainda não estiver, inicie cloudflared antes do deploy.
Após o primeiro deploy, o OpenRC restaura os containers no boot sem rede,
antes do Caddy. O cron procura novas imagens quando a conexão volta.

## Uso diário

Abra uma PR e espere o check `CI / image`. Ao fazer merge em `master`,
a imagem é publicada e o notebook a encontra em até um minuto após o CI.
Se o build falhar ou a imagem não estiver disponível, mantém a versão atual.
Uma atualização usa o slot inativo (`blue:18085`, `green:18086`), testa
páginas, CSS, fonte, SHA e proteção dos arquivos privados, e só então troca
o Caddy. Falhas na promoção restauram a configuração anterior.

```sh
doas rc-service werc-oci verify
doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a
curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt
doas tail -50 /var/log/messages
doas tail -50 /var/log/werc-oci.log
```

Use `werc-podman` para os containers de produção. Os caminhos são fixos:
armazenamento `/var/lib/werc/.local/share/werc-oci/storage` e runtime
`/run/werc-oci`. O Podman pessoal de `aw` continua independente.

## Rollback e manutenção

```sh
doas touch /etc/werc-oci/paused
doas cat /etc/werc-oci/active-slot
# Se green está ativo, blue é a versão anterior:
doas werc-oci-promote blue
```

O arquivo `paused` suspende o deploy automático para que o próximo minuto
não desfaça o rollback. Para voltar: `doas rm /etc/werc-oci/paused`.
Para recuperar containers parados: `doas rc-service werc-oci restore`.

Os dois slots são preservados para rollback. Imagens antigas podem ser
removidas explicitamente com `doas su-exec werc env HOME=/var/lib/werc
werc-podman image prune -a`; imagens usadas pelos slots não são removidas.
Não use `system reset` para manutenção.

O notebook precisa estar ligado, acordado e conectado para atender o site.
Reinstalar os scripts exige executar o instalador; o CD atualiza apenas
a imagem do site. A rota antiga `deploy.arthurgusmao.com.br` não é utilizada.

Referências: [Podman rootless](https://docs.podman.io/en/stable/markdown/podman.1.html),
[Caddy reverse proxy](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy)
e [Cloudflare Tunnel](https://developers.cloudflare.com/tunnel/get-started/).
