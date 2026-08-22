# Xitike Célia

Bot de WhatsApp para gestão de xitiques (Cabelos e Iphones), com
confirmação automática de pagamentos M-Pesa/e-Mola e um painel web.

## Instalação e run

npm install
node init_db.js
node index.js

Na primeira vez, aparecer um QR code no terminal — escaneia com o WhatsApp
(Aparelhos ligados > Ligar aparelho). Depois disso a sessão fica
guardada em `auth_info/` e não precisa escanear de novo.
O painel web fica em `https://xitikecelia-nuvem.onrender.com'

## Comandos do bot (dentro do grupo de WhatsApp)

- `!novo Nome Valor DiasAteReceber` — cria um xitique novo (só a Admins)
- `!cadastrar 840000000 Nome Completo` — regista um membro que ainda não escreveu no grupo (só a Admins)
- `!pagos` (seguido de uma linha `numero Nome valor` por membro) — importa em massa quem já pagou, pra grupos que já estavam a decorrer antes do bot (só a Célia)
- `!atribuir IDTransacao 840000000 ` — liga manualmente um pagamento pendente a um membro (só a Admins)
- `!banir 840000000 ` — remove um membro do grupo (só a Admins)
- `!pendentes` — lista pagamentos que ainda precisam de atribuição manual
- `!resumo` — mostra a situação de cada membro (em dia / dívida / crédito)
- `!ajuda` — mostra esta lista dentro do WhatsApp

Cliente: cola no grupo a SMS de confirmação de pagamento (M-Pesa ou e-Mola) —
**ou manda um print da confirmação**, o bot lê o texto de dentro da imagem
automaticamente (OCR). O bot regista automaticamente se o destino bater com
um número oficial configurado; caso contrário, avisa em vez de creditar sozinho.

## SMS direto do telemóvel da Célia (mais seguro que colar no grupo)

Em vez de alguém colar o comprovativo no grupo, dá pra configurar o iPhone
da Célia pra mandar a SMS de "Recebeste" direto pro servidor assim que
chega — ninguém consegue fabricar isto, porque só quem tem o telemóvel
físico dela é que gera essa chamada.


### No Celular do xitike:
1. App **Atalhos** > separador **Automação** > **+** > **Criar Automação Pessoal**
2. Escolhe **Mensagem**
3. Remetente: escreve `M-Pesa` ou `eMola` (ou o número/nome exato como aparece)
4. "A mensagem contém": escreve `Recebeste` (só confirmações de recebimento)
5. Ativa **Executar Imediatamente** e desativa **Perguntar Antes de Executar**
6. Adiciona a ação **Obter Conteúdo de URL**
7. Configura:
   - URL: `https://https://xitikecelia-nuvem.onrender.com/api/gateway/sms`
   - Método: **POST**
   - Cabeçalhos: `Authorization` → `xitike da9085b747b0313b56f9be82e474c2d920df68801a92957f` (o token está no `.env`, muda os dois lados juntos se alterares)
   - Corpo: **JSON**, com os campos:
     - `texto_sms` → variável "Conteúdo da Mensagem"
     - `remetente_sms` → variável "Remetente" (ou escreve manualmente "M-Pesa"/"eMola")
8. Concluído.

O servidor **não** escolhe o grupo/membro sozinho a partir da SMS — quem diz
o grupo certo é sempre o cliente, ao postar a própria confirmação lá dentro
(uma pessoa pode estar em vários xitiques ao mesmo tempo, então adivinhar
pelo nome não seria seguro). A SMS real só serve como prova: o bot cruza pelo
**ID da transação**, que é o mesmo dos dois lados (quem manda e quem recebe
usam a mesma referência, confirmado com exemplos reais).

Como funciona na prática:
- Cliente posta a confirmação no grupo → se a SMS real já tiver chegado com
  o mesmo ID, credita na hora. Se ainda não chegou, avisa "a aguardar SMS
  real" e fica guardado.
- A SMS real chega via Atalho → se já havia alguém à espera com esse ID,
  completa o pagamento nesse instante. Se não, fica guardada à espera de
  alguém postar.
- `!pendentes` (admin) mostra os dois tipos de pendência.

## Coisas por validar, em caso de falhas

- **Leitura de imagens (OCR) não é 100% fiável.** Prints recomprimidos pelo
  WhatsApp podem sair com números lidos errado (0 vs O, 1 vs l). 
- A primeira vez que uma imagem for processada, o Tesseract vai descarregar o
  pacote de idioma (português) da internet — precisa de rede nessa altura.
  Depois disso fica guardado localmente e não precisa mais.
- O bot precisa de ser **admin do grupo de WhatsApp** pra conseguir remover
  membros (`!banir`, duplicado de comprovativo). Sem isso, ele avisa mas não
  consegue agir.
- A regra de dívida/crédito (abate dívida antiga, depois cobre o dia, resto
  vira crédito) é uma interpretação — confirma que bate com o que precisas.
