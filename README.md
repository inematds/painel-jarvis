# painel-jarvis

[![painel-jarvis: o que falhou hoje? Seu Jarvis responde](guia/assets/banner.jpg)](https://inematds.github.io/painel-jarvis/guia/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## O que é

Quem deixa coisas rodando sozinhas no computador (um assistente, um backup, um robô que publica vídeo, um site) costuma saber que algo quebrou só quando alguém reclama. O painel-jarvis é um pequeno programa em Python que olha tudo isso por você: memória e disco da máquina, serviços, rotinas, sites e o que mudou nos seus projetos. Você vê num painel que abre no celular como um app, ou simplesmente pergunta ao seu assistente de IA "o que falhou hoje?". Precisa de um computador com Linux e Python 3.11 ou mais novo.

## 📖 Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/painel-jarvis/guia/**

Python puro (só a biblioteca padrão, 3.11+). Um núcleo, três jeitos de usar:

| Jeito | Para quê | Comando |
|---|---|---|
| Plugin do **INTELECTO v2** | o seu Jarvis em Python ganha as ferramentas `painel_resumo`, `painel_falhas` e `painel_quadro` | `INTELECTO_PLUGINS=painel intelecto` |
| Servidor **MCP** | qualquer agente que fale MCP (Claude Code, Codex, Cursor, outro Jarvis) usa as mesmas ferramentas | `claude mcp add painel -- painel-jarvis mcp` |
| **Painel web** instalável | cards no navegador e no celular (PWA), com senha | `painel-jarvis web` |

## Os quadros

Cada seção do `painel.toml` liga um quadro. Tudo é só leitura.

| Quadro | O que mostra | Fica amarelo/vermelho quando |
|---|---|---|
| `maquina` | memória, disco, carga, GPU (nvidia-smi) | memória < 10 % livre, disco ≥ 90 % |
| `servicos` | serviços e timers do systemd | serviço falhou, timer parado, unidade não existe |
| `rotinas` | rotinas do cron pelo log | erro nas últimas linhas, ou parada além do prazo |
| `sites` | um GET leve por site | site fora do ar |
| `git` | o que fizemos: commits de 24 h e 7 dias | — |
| `uso` | qualquer JSON que outro script grava (visitas, vendas, fila) | arquivo parado além do prazo |

O quadro `uso` é a ponte para os seus números: o seu script coleta (e decide se pode chamar alguma API) e grava um JSON; o painel só lê.

## Instalar

```bash
pipx install git+https://github.com/inematds/painel-jarvis    # ou pip install num venv (ative o venv antes)
mkdir -p ~/.config/painel-jarvis
painel-jarvis exemplo > ~/.config/painel-jarvis/painel.toml   # liga a máquina; tire o "#" das seções que quiser
painel-jarvis cartoes                                          # confere no terminal
```

## O processo, do início ao fim

| # | Passo | Comando | Quem |
|---|---|---|---|
| 1 | Instalar o pacote | `pipx install git+https://github.com/inematds/painel-jarvis` | 🧠 você |
| 2 | Escrever o `painel.toml` (o que vigiar) | `painel-jarvis exemplo > ~/.config/painel-jarvis/painel.toml` | 🧠 você |
| 3 | Conferir os quadros no terminal | `painel-jarvis cartoes` · `painel-jarvis falhas` | 🧠 você |
| 4 | Ligar no Jarvis: INTELECTO (mesmo Python: mesmo venv ou `pipx inject intelecto …`) | `"plugins": ["painel"]` no `config.json` ou `INTELECTO_PLUGINS=painel` | 🧠 você |
| 5 | Ligar em qualquer agente MCP | `claude mcp add painel -- painel-jarvis mcp` | 🧠 você |
| 6 | Definir a senha do painel web | `painel-jarvis senha` | 🧠 você |
| 7 | Deixar o painel web no ar | `painel-jarvis web --bind 0.0.0.0` (serviço systemd, ver abaixo) | ⚙️ systemd |
| 8 | Abrir fora de casa | túnel (Cloudflare Tunnel, Tailscale) na frente da porta 8790 | ⚙️ túnel |
| 9 | Instalar no celular | Chrome → ⋮ → Instalar app (iPhone: Compartilhar → Tela de Início) | 🧠 você |
| 10 | Perguntar ao Jarvis | "o que falhou hoje?" → `painel_falhas` | 🤖 seu Jarvis |

### Serviço systemd (passo 7)

Defina a senha antes (`painel-jarvis senha`); sem ela o serviço fica só em 127.0.0.1.

```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/painel-jarvis.service <<'FIM'
[Unit]
Description=painel-jarvis web
[Service]
ExecStart=%h/.local/bin/painel-jarvis web --bind 0.0.0.0
Restart=on-failure
[Install]
WantedBy=default.target
FIM
systemctl --user daemon-reload
```

`systemctl --user enable --now painel-jarvis` e `loginctl enable-linger $USER` (sobe no boot e não morre no logout). Instalou num venv? Troque o `ExecStart` pelo caminho `~/.venv-painel/bin/painel-jarvis`.

## Segurança

- Sem senha, o painel web só escuta em `127.0.0.1`. Fora da máquina, só com senha (scrypt, 12+ caracteres), cookie `HttpOnly`, 5 tentativas erradas = 15 min de espera.
- O painel não fala HTTPS sozinho: para abrir pela internet, use um túnel com HTTPS na frente.
- As ferramentas só leem (`readOnlyHint` no MCP). No INTELECTO, um plugin nunca substitui uma ferramenta de fábrica (a trava de comandos continua a mesma).
- O manifesto, o `sw.js` e os ícones são públicos (não têm dado). O service worker não guarda dado em cache.

## Testes

```bash
python3 -m pytest -q
```

Licença MIT · feito pelo [INEMA](https://inema.club).
