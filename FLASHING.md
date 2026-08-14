# Como compilar, gravar e testar o firmware do MUMA

Isso precisa rodar na SUA máquina, com o aparelho conectado por USB —
não dá pra compilar nem flashear num ambiente sem o hardware físico.

## 1. Instalar o ESP-IDF

Precisa da versão 5.x. Duas formas:

**Opção A -- instalador oficial (recomendado se é a primeira vez):**
Siga https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/get-started/
pro seu sistema (Windows/Linux/macOS). No fim, você vai ter um comando
`get_idf` (ou `export.sh`/`export.bat`) pra ativar o ambiente em cada
terminal novo.

**Opção B -- se já usa Claude Code localmente:** este repo já vem com
as skills `build-idf`/`flash-idf` em `.claude/skills/` -- abrindo
Claude Code aqui, é só pedir "compila o firmware" e ele detecta/ativa
o ambiente sozinho (mas o ESP-IDF ainda precisa estar instalado na
máquina, a skill não instala).

## 2. Clonar este repositório

```bash
git clone https://github.com/bernardobbs/Numa-firmware.git
cd Numa-firmware
```

## 3. Trocar a senha de admin

Abra `main/boards/spotpear/sp-esp32-s3-1.54-muma/sp-esp32-s3-1.54-muma.cc`,
ache a linha:

```cpp
#define FAMILY_ADMIN_PASSWORD "SENHA-QUE-VOCES-ESCOLHEREM"
```

e troque pela senha real que vocês vão usar em `/admin`. **Sem isso**,
o firmware compila e funciona normalmente, só que `/admin` fica sempre
bloqueado (403) até vocês colocarem uma senha real.

## 4. Ativar o ambiente ESP-IDF

```bash
# Linux/macOS (ajuste o caminho pro onde instalou)
. $HOME/esp/esp-idf/export.sh

# Windows (PowerShell)
C:\Espressif\Initialize-Idf.ps1
```

Repita isso em todo terminal novo (ou use o atalho que o instalador
criou, ex.: "ESP-IDF 5.x CMD" no menu iniciar do Windows).

## 5. Escolher a placa (só na primeira vez, ou se trocar de placa)

```bash
idf.py set-target esp32s3
idf.py menuconfig
```

Dentro do menuconfig: `Xiaozhi Assistant` -> `Board Type` -> escolha
**`Spotpear ESP32-S3 1.54 MUMA`**. Salve e saia (`S`, depois `Q`).

## 6. Compilar

```bash
idf.py build
```

Primeira vez demora bastante (baixa dependências, compila tudo do
zero) -- pode levar 10-20 minutos dependendo da máquina. As próximas
são incrementais e bem mais rápidas.

Se der erro de compilação, me manda a mensagem completa -- pode ser
algo que só aparece compilando de verdade (nunca testei isso, só
revisei o código).

## 7. Conectar o aparelho e achar a porta

Conecte o MUMA por USB. Depois:

```bash
# Linux
ls /dev/ttyUSB* /dev/ttyACM*

# macOS
ls /dev/cu.*

# Windows
# Veja no Gerenciador de Dispositivos, algo como COM3, COM4...
```

## 8. Gravar (flash)

```bash
idf.py -p <PORTA> flash monitor
```

Troque `<PORTA>` pelo que achou no passo 7 (ex.: `/dev/ttyUSB0`,
`COM3`). O `monitor` no final deixa o log ao vivo aberto -- útil pra
ver se o boot completa sem crash. Pra sair do monitor: `Ctrl+]`.

## 9. Primeira configuração (depois do boot)

1. Se for a primeira vez, o aparelho entra em modo de configuração de
   Wi-Fi -- conecte no ponto de acesso que ele cria e informe a rede
   de casa (fluxo padrão do xiaozhi-esp32, não é algo que a gente
   escreveu).
2. Depois de conectado, ache o IP do aparelho (aparece no log do
   `idf.py monitor`, ou no painel do seu roteador).
3. Abra `http://<IP-do-aparelho>/admin` no navegador, use a senha do
   passo 3. Preencha nome e data de nascimento da criança, confira as
   regras sugeridas pela idade, e defina o tópico do ntfy se quiser
   alertas de sobrecarga.
4. Abra `http://<IP-do-aparelho>/` -- essa é a página da criança.

## 10. Roteiro de teste manual

Sem isso ser um teste automatizado, pelo menos confira cada peça uma
vez:

- [ ] Boot completa sem crash (log do `idf.py monitor` limpo)
- [ ] `/` carrega, mostra o nome (depois de configurado em `/admin`)
- [ ] Escolher um bichinho diferente em `/` funciona e persiste depois
      de reiniciar o aparelho
- [ ] Rotina do dia aparece em `/`, marcar tarefa como feita funciona
- [ ] Pomodoro: iniciar em `/` mostra o cronômetro **na tela do
      aparelho** (canto superior direito) contando pra baixo
- [ ] Depois de pontos suficientes, o selo de estágio (canto inferior
      direito da tela) muda de "Ovo" pra "Filhote"
- [ ] `/admin` pede senha (Basic Auth) e bloqueia sem ela
- [ ] Salvar em `/admin` funciona (testar rotina e regras)
- [ ] Se ligar "semáforo de sobrecarga" em `/admin`, o assistente de
      voz aceita comandos de nível (isso exige o backend MCP
      configurado -- ver `muma-mcp-bridge` se quiser estender)

Qualquer um desses passos que falhar, me manda o log do `idf.py
monitor` na hora do problema -- isso me dá muito mais pra trabalhar
do que "não funcionou".

## Atualizações futuras

O remote `upstream` deste clone aponta pro `78/xiaozhi-esp32`
original. Pra trazer atualizações de lá:

```bash
git fetch upstream
git merge upstream/main   # ou rebase, como preferir
```

(esse remote só existe na cópia que eu preparei -- se você clonar o
`Numa-firmware` do zero, adicione de novo com
`git remote add upstream https://github.com/78/xiaozhi-esp32`)
