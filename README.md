# Crptochat Android

[![Licença MIT](https://img.shields.io/badge/licen%C3%A7a-MIT-blue.svg)](LICENSE)

Cliente Android **open-source** do Crptochat: uma camada mobile de comunicação
sobre o [crptochat-oss](https://github.com/hacking-table/crptochat-oss/).

Este app é um WebView enxuto que abre o cliente web publicado pelo projeto
original e fala com o mesmo servidor por `wss://`. Ele **não reimplementa** o
protocolo, a criptografia nem o relay: toda a conversa cifrada de ponta a ponta
continua sendo feita pelo cliente do `crptochat-oss`. Aqui só entra a integração
com o Android — origem fixa, endpoint WebSocket, ícone e permissões mínimas.

- Protocolo, cliente web, cifra e servidor:
  [hacking-table/crptochat-oss](https://github.com/hacking-table/crptochat-oss/)
- Endpoint público deste build: `wss://crptochat.acpt.app.br/ws`
- Sem ponte JavaScript e sem permissões de câmera, microfone, arquivos ou
  localização.
- Licença MIT (o mesmo modelo do projeto de origem).

## Configuração

O build já aponta para a instalação pública: a página em
`https://crptochat.acpt.app.br/web/` e o WebSocket em
`wss://crptochat.acpt.app.br/ws` (mesmo host/porta, conforme a CSP do cliente).

```powershell
./gradlew assembleDebug
```

O endpoint WebSocket é fixado pelo app no `localStorage` da página, então uma
URL salva em outro teste não troca o servidor silenciosamente. Para outro
domínio, sobrescreva os dois valores juntos:

```powershell
./gradlew assembleDebug -PchatOrigin=https://outro.exemplo.org/web/ -PchatWsUrl=wss://outro.exemplo.org/ws
```

O WebView permite somente páginas e subrecursos no mesmo host e porta.
Navegação externa, conteúdo misto, certificados inválidos, permissões web e
acesso `file://` são bloqueados.

### Testar no emulador (localhost)

Para usar um servidor local, sobrescreva as duas propriedades e use
`adb reverse` para que `localhost` dentro do emulador chegue à máquina que roda
o servidor — **não use `10.0.2.2`**: a CSP do cliente web e o `--ws-origin` do
servidor aceitam apenas `localhost` quando a página é HTTP.

```powershell
./gradlew assembleDebug -PchatOrigin=http://localhost:8123/ -PchatWsUrl=ws://localhost:7778
```

```powershell
$adb = "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe"
& $adb reverse tcp:8123 tcp:8123
& $adb reverse tcp:7778 tcp:7778
```

Na máquina, a partir de `crptochat-oss/`:

```powershell
python -m http.server 8123 --bind 127.0.0.1 --directory web/dist
crptochat-server --no-tls --ws-port 7778 --ws-origin http://localhost:8123
```

Com `-PchatWsUrl=ws://localhost:7778`, o próprio app preenche esse valor na
página. Se ainda aparecer "falha ao conectar servidor (WebSocket)", confirme se
o relay foi iniciado com `--ws-port`: sem isso a entrada WebSocket não existe.

## Abrir no Android Studio

1. Abra esta pasta no Android Studio.
2. Use o padrão para a instalação pública; para testar localmente defina
   `chatOrigin` e `chatWsUrl` em `~/.gradle/gradle.properties` ou pelos
   parâmetros de build acima.
3. Rode em um dispositivo Android 8.0 ou superior (o app compila e tem como
   alvo a API 36).

O projeto depende do cliente web publicado pelo projeto principal. Antes de
distribuir, conclua os requisitos de integridade e auditoria do cliente web
descritos em `crptochat-oss/web/README.md`.

## Ícone

O ícone de chat cyberpunk (balão neon com prompt `>_`, glitch magenta e grade
de terminal) está em `app/src/main/res/mipmap-*`, com versões adaptive,
redonda, legada e monocromática (tema do Android 13). Para regenerar os PNGs:

```powershell
python tools/generate_icons.py
```
