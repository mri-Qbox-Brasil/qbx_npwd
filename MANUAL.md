# qbx_npwd — Manual

Recurso-ponte que conecta o telefone [npwd](https://github.com/project-error/npwd) ao `qbx_core`: registra jogadores no npwd, controla o telefone como item do inventário e faz o unload no logout.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Convars](#convars)
5. [Arquivo `config.json`](#arquivo-configjson)
6. [Comportamento](#comportamento)
7. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
8. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `npwd` | Sim | O telefone em si. Este recurso só faz a ponte (`newPlayer`, `unloadPlayer`, `setPhoneDisabled`, `setPhoneVisible`) |
| `qbx_core` | Sim | Eventos de load/unload do jogador, `GetQBPlayers`, `CreateUseableItem` |
| `ox_lib` | Sim | `lib.versionCheck` |
| `oxmysql` | Sim | Grava o número do telefone na coluna `phone_number` da tabela `players` |
| `ox_inventory` | Sim | Conta os itens de telefone no inventário (`Search`, evento `ox_inventory:itemCount`) |

---

## Instalação

1. Copie a pasta `qbx_npwd` para `resources/`.
2. Adicione ao `server.cfg`, **depois** de `qbx_core` e `ox_inventory` e **antes** de `npwd`:
   ```
   ensure qbx_npwd
   ```
3. A tabela `players` precisa ter a coluna `phone_number` — o servidor executa `UPDATE players SET phone_number = ?` a cada load de personagem. Essa coluna faz parte do patch de banco do próprio `npwd`.
4. Cadastre no `ox_inventory` os itens listados em `PhoneList` (por padrão, apenas `phone`). Para arrastar o item sobre o botão "Use" do inventário, o item precisa estar como `usable` e `shouldClose` no `qbx_core/shared/items.lua`; dar duplo clique já funciona sem isso.

---

## Configuração

Arquivo: `config.lua`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `PhoneList` | array de strings | Sim | Nomes dos itens tratados como telefone. Cada item vira um item usável (abre o npwd) e é contado para habilitar/desabilitar o telefone. Padrão: `{ 'phone' }` |

---

## Convars

| Convar | Padrão | Descrição |
|---|---|---|
| `qbx_npwd-debug` | `0` | Definido como `1`, imprime no console do servidor cada jogador registrado no npwd (source, citizenid, número) |

```
set qbx_npwd-debug 1
```

---

## Arquivo `config.json`

O `config.json` na raiz **não é carregado por este recurso** — o `fxmanifest.lua` não o referencia. Ele é um exemplo de configuração do próprio `npwd`, para ser copiado para lá. O bloco relevante para esta ponte é:

```json
"PhoneAsItem": {
  "enabled": true,
  "exportResource": "qbx_npwd",
  "exportFunction": "HasPhone"
}
```

Ou seja: com o telefone como item, o `npwd` consulta o export `HasPhone` deste recurso para saber se o jogador possui um telefone.

---

## Comportamento

- **Load do personagem** (`QBCore:Server:PlayerLoaded`): grava `charinfo.phone` na coluna `phone_number` e registra o jogador no npwd com citizenid, número, nome e sobrenome.
- **Restart do recurso**: todos os jogadores online são re-registrados no npwd automaticamente.
- **Contagem de telefones**: o cliente conta os itens de `PhoneList` no login e a cada evento `ox_inventory:itemCount`. Sem nenhum telefone, o npwd é desabilitado (`setPhoneDisabled(true)`).
- **Logout** (`QBCore:Client:OnPlayerUnload`): o telefone é desabilitado e o jogador é removido do npwd.
- **Usar o item**: abre o telefone (`setPhoneVisible(true)`).

---

## Entrypoints para outros recursos

### Export `HasPhone` (cliente)

Retorna se o jogador tem algum item de `PhoneList` no inventário. É o export consumido pelo `npwd` via `PhoneAsItem`.

```lua
local hasPhone = exports.qbx_npwd:HasPhone()
```

### Eventos internos

| Evento | Lado | Descrição |
|---|---|---|
| `qbx_npwd:client:setPhoneVisible` | Cliente | Recebe um bool e repassa para `npwd:setPhoneVisible` |
| `qbx_npwd:server:UnloadPlayer` | Servidor | Remove o jogador do npwd (disparado pelo cliente no unload) |

O recurso também declara `provide 'qb-npwd'`, ocupando o lugar do `qb-npwd` para dependências de outros scripts.

---

## Estrutura de arquivos

```
qbx_npwd/
├── client.lua        — contagem de telefones no inventário, export HasPhone, abrir/fechar o npwd
├── server.lua        — registro no npwd, gravação do phone_number, item usável, debug
├── config.lua        — PhoneList (itens tratados como telefone)
├── config.json       — exemplo de configuração do npwd (não carregado por este recurso)
└── fxmanifest.lua
```
