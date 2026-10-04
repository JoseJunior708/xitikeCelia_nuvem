# Patch do Baileys

`@whiskeysockets+baileys+7.0.0-rc14.patch` é aplicado automaticamente no
`postinstall`. Sem ele, **o bot não consegue emparelhar com o WhatsApp**.

## Porque é que existe

Em ~28-07-2026 a Meta adicionou um passo novo ao registo de aparelhos: depois
de leres o QR, o WhatsApp envia `<notification type="companion_reg_refresh">`.
O Baileys limitava-se a fazer *ack* e a descartar.

O `advSecretKey` que o QR anuncia é uma das quatro peças do código, e esse
passo manda o servidor **retirá-lo**. Resultado: o QR que fica no ecrã advertise
um segredo que já não vale, o telefone lê, diz que não conseguiu ligar, e
`pair-success` nunca chega. O mesmo se vê no `whatsmeow`, porque o problema é
do protocolo e não da biblioteca.

Isto não é do código deste projecto. Referências:

- WhiskeySockets/Baileys#2737 — QR nunca completa, `companion_reg_refresh` não tratado
- WhiskeySockets/Baileys#2765 — o fix upstream, **aberto e marcado stale, não mergeado, não publicado**
- WhiskeySockets/Baileys#2602 — notificação `link_code_companion_reg` sem campos crypto rebenta com *Invalid buffer*
- WhiskeySockets/Baileys#2749 — ACK antes do login rebenta com TypeError quando `creds.me` ainda não existe
- WhiskeySockets/Baileys#2559 — `requestPairingCode` devolvia um código que o servidor nunca registou
- evolution-api#2727 — mesma correcção, verificada em produção

A versão `6.7.24` (dist-tag `legacy`) **não tem** o fix. Não há rc15. O `master`
do GitHub também não.

## O que o patch faz

| Ficheiro | Alteração |
| --- | --- |
| `lib/Utils/companion-reg-client-utils.js` | `makePairingQRRenderer` (re-desenha o QR actual sem gastar um ref) e `handleCompanionRegRefresh` (roda o `advSecretKey`) |
| `lib/Socket/socket.js` | handler de `CB:notification,type:companion_reg_refresh`; `requestPairingCode` espera a resposta do servidor com `query()` em vez de devolver um código fantasma |
| `lib/Socket/messages-recv.js` | guard no `link_code_companion_reg`; `creds.me?.id` no ACK |

## Quando o Baileys publicar isto

O `postinstall` foi propositadamente tolerante: se o patch não aplicar, imprime
um aviso e o deploy continua em vez de rebentar. Ao actualizares o Baileys:

1. `npm install`
2. Se aparecer o aviso, o patch não aplicou — o `companion_reg_refresh` já vem
   corrigido upstream, ou o código mudou.
3. `npx patch-package @whiskeysockets/baileys` para regenerar o patch.
4. `git add patches package.json package-lock.json && git commit -m "..."`

O aviso no log do Render é o sinal. Se aparecer e o bot deixar de parear, é isto.