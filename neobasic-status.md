# NeoBasic — onde paramos (09/11/2026)

**Última versão entregue:** NeoBasic 0.6.13 (ZIP único: Windows `NeoBasic.exe` + Linux `NeoBasic.sh` + `NeoBasic.html`). A 0.6.12 foi feita pelo ChatGPT (MySQL obrigatório no Linux, controles novos, WebForms); a 0.6.13 corrige os erros dela e traz o instalador MySQL do Windows. Detalhes em `CORRECOES-0.6.13.txt` e `MYSQL-README.txt` dentro do ZIP.

## Regras do usuário
- **Interface sempre em inglês, nunca em português** — IDE, janelas de instalação, avisos, mensagens dos lançadores (os textos de ajuda .txt para ela podem ser em português).
- MySQL é **obrigatório** nos dois sistemas, não opcional, e não deve ser trocado por SQLite.
- Ordem ao abrir: 1) abre a **tela principal do NeoBasic** (com o ícone do NeoBasic na barra de tarefas); 2) se faltar MySQL, aparece por cima uma janela dizendo que é necessário e perguntando se aceita instalar (Install MySQL / Exit); Exit fecha o NeoBasic; 3) já instalado = abre direto.
- Só o NeoBasic instala. Abrir um .html (IDE ou programa exportado) nunca instala nem pede permissão; mostra aviso (em inglês) para abrir o .sh/.exe.
- Ícone próprio do NeoBasic na barra de tarefas com o Chrome (reclamou de ícone de despertador no Linux e de ícone do Chrome).

## Fluxo de abertura (0.6.13)
- Pronto: lançador liga o bridge e abre `http://127.0.0.1:43821/NeoBasic.html?app=1`.
- Faltando: lançador abre `file://…/NeoBasic.html?app=1&setup=1` → IDE mostra a tela principal com um aviso "Preparing MySQL Server…" e consulta `/api/health` (fetch no-cors) a cada 1 s; o lançador mostra a pergunta/progresso por cima; quando o bridge sobe, a IDE recarrega pelo http.
- Ícone: Windows = NeoBasic.exe aplica na janela do Chrome AppUserModelID `Drimah.NeoBasic` + Relaunch* + WM_SETICON (o atalho do Menu Iniciar tem o mesmo AppUserModelID); Linux = `neobasic-wmicon.py` (ctypes/libX11) põe `_NET_WM_ICON` 16–256 px de `icone/neobasic-icon.argb` e reaplica.

## Linux (Mint/Ubuntu)
- `NeoBasic.sh` → `mysql-bootstrap.sh` (estados missing/stopped/unconfigured/ready/mariadb, zenity com progresso real do apt) → `mysql-install-root.sh` via pkexec (Oracle 8.4 LTS; se falhar, remove o repo e usa o MySQL do Ubuntu 8.0). Conexão por socket.

## Windows
- `NeoBasic.exe` (C, mingw; `codigo-fonte/launcher/neobasic.c`) roda `neobasic-windows.ps1 -Phase start` (0 = URL pronta, 3 = precisa instalar, 2 = não abrir); se 3, abre a tela principal e roda `-Phase setup` (janela própria Install MySQL / Exit, progresso, UAC).
- ps1 (PowerShell 5.1, UTF-8 com BOM): downloads com progresso (VC++ se faltar, Python 3.12.10 embed de python.org ou NuGet, MySQL 8.4 ZIP com versão lida da página da Oracle + versões de reserva), worker elevado com status em arquivo: `C:\Program Files\NeoBasic\{MySQL\server,runtime\python}`, dados `C:\ProgramData\NeoBasic\MySQL`, serviço `NeoBasicMySQL` (127.0.0.1, 3306 ou próxima livre), root aleatório em `ProgramData\NeoBasic\mysql-root.txt`, PyMySQL de `runtime-windows\*.whl`. Log: `C:\ProgramData\NeoBasic\setup.log`.
- Bridge no Windows: TCP + SSL (sem `cryptography`), config `%USERPROFILE%\.config\neobasic\mysql.json` com `"ssl": true`.
- Não testado num Windows real (só PowerShell no Linux, analisador PS 5.1, exe no Wine).

## Comum
- `mysql-bridge.py` em 127.0.0.1 (IDE porta fixa 43821; apps exportados porta por hash da pasta); injeta token no HTML; versão no /api/health; substitui bridge antigo.
- Dados: cada banco do NeoBasic = banco MySQL `nb_<projeto>_<banco>` (ex.: `nb_books_alunos`), tabelas com nome real, tipos via `_nb_cols` + valores, troca atômica com RENAME TABLE. Catálogo `neobasic_runtime.__nb_catalog_v2`. Formato 0.6.12 migrado sozinho.
- IDE continua com SQL.js em memória (DAO síncrono) e grava no MySQL a cada alteração.
- Make App: .sh + bootstrap + bridge sempre; com .exe também o `neobasic-windows.ps1`; README.txt em inglês.
- Textos do NavigationBar em tempo de execução seguem o Locale do app (pt-BR por padrão) — não foram mudados.

## O que já existia até 0.6.11
- IDE estilo VB5/6 (HTML/JS em janela própria de navegador Chromium); interface em inglês, datas/números em pt-BR.
- Visual: tema claro azul, toolbox 2 colunas, barra do topo no padrão Drimah, botões em cápsula; `Corners`, `Alignment`.
- Seleção múltipla, Undo/Redo, Menu Editor, Pincel de formatação, alinhar estilo Inkscape.
- DAO do VB5 (sql.js), abre .mdb/.accdb; controle Data; importar CSV/TSV; painel Data gera telas.
- NavigationBar (`NB.NavigationBar`, Align Top/Bottom/None, ButtonOrder, LockFields etc.); TextBox `Mask`.
- Make App: .html / .exe / .sh em `Executables\`. Navegador em `navegador.txt`.

## 0.6.12 (ChatGPT)
- Controles novos (Panel, GroupBox, TabControl, DataGrid, DatePicker, Slider, ProgressBar, SearchBox, ImageList, WebBrowser, RichTextBox, TreeView, FormReport) — por enquanto só visuais; `LockPosition`; WebForms com editor visual; NavigationBar com Search/Print/Refresh.

## Ideias para depois
- Dar função real aos controles novos (DataGrid ligado ao RecordSource, FormReport com dados/PDF).
- Importar .xlsx; Google Forms + respostas; mudar propriedades de vários controles de uma vez; DBGrid/DBList; Crystal Reports (fase 2).
