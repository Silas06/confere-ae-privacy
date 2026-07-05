# Confere Aê — Funcionalidades

App para montar listas de compras, registrar preços e quantidades durante a compra no mercado, comparar promoções e guardar um histórico de compras finalizadas. Disponível em iOS, Android e Web.

## Sumário

- [Fluxo principal](#fluxo-principal)
- [Nova compra](#nova-compra)
- [Lista de compras (tela principal)](#lista-de-compras-tela-principal)
- [Compartilhamento de lista em tempo real](#compartilhamento-de-lista-em-tempo-real)
- [Histórico de compras](#histórico-de-compras)
- [Detalhe de uma compra / Reativar lista](#detalhe-de-uma-compra--reativar-lista)
- [Máscara de preço](#máscara-de-preço)
- [Reconhecimento de voz](#reconhecimento-de-voz)
- [Foto de produto/promoção](#foto-de-produtopromoção)
- [Categorias, busca e filtros](#categorias-busca-e-filtros)
- [Modo escuro](#modo-escuro)
- [Anúncios](#anúncios)
- [Persistência offline](#persistência-offline)

---

## Fluxo principal

1. Usuário abre o app (splash screen) e cai na **Home**.
2. Inicia uma **Nova Compra**, escolhendo o nome da loja.
3. Monta a **Lista de Compras**: adiciona produtos, preços, quantidades, categorias, marca promoções, tira fotos.
4. Opcionalmente **compartilha a lista em tempo real** com outra pessoa (ex.: cônjuge indo no mesmo mercado).
5. **Finaliza a compra**, que vai para o **Histórico**.
6. A partir do histórico, pode **reativar** uma compra antiga como nova lista em andamento.

---

## Nova compra

- Tela para dar nome à loja (chips de sugestão: Supermercado, Mercado, Atacadão, Carrefour, Extra, Pão de Açúcar, Assaí, Feira, Hortifruti, Farmácia).
- **Reaproveitar lista anterior**: se já existe histórico, é possível escolher uma compra antiga e copiar seus itens (preços, categorias) para a nova lista, com novos IDs.

## Lista de compras (tela principal)

- Adicionar produto: nome (com ditado por voz), preço (com máscara de moeda), quantidade, categoria, marcação de "Promoção" e foto.
- Editar e remover produtos (com confirmação).
- Marcar item como "comprado" (aparência riscada) e incrementar/decrementar quantidade direto no card.
- Agrupamento dos produtos em seções por categoria, com accordion expansível/colapsável.
- Busca por nome (com normalização de acentos) e filtros por categoria e "somente promoções".
- Rodapé fixo com total de itens, contagem de promoções e valor total.
- Finalização da compra com confirmação, overlay de loading e exibição de anúncio intersticial antes de voltar para a Home.
- Preview de fotos em tela cheia com zoom.

## Compartilhamento de lista em tempo real

Permite que duas ou mais pessoas acompanhem e editem a mesma lista de compras ao mesmo tempo (ex.: um vai ao mercado enquanto o outro acompanha de casa), usando Firebase (Auth anônimo + Firestore) como backend.

### Criar e compartilhar
- Botão de compartilhar no header da lista. Exige ao menos 1 item na lista.
- Gera um **código de compartilhamento** único de 7 caracteres (sem letras/números ambíguos).
- Grava a lista (nome da loja, itens, dono) no Firestore, com TTL de **90 dias** de expiração a partir da última atividade.
- Abre o compartilhamento nativo do sistema (WhatsApp, SMS, etc.) com um link (`https://confereae.com/join/{código}`) e mensagem pronta explicando que é em tempo real.

### Entrar em uma lista compartilhada
- Ao abrir o link recebido, o app entra direto na lista compartilhada (deep link) — sem necessidade de login/cadastro (autenticação anônima é transparente).
- Se o link expirou ou a lista foi excluída, mostra uma tela de "lista indisponível".

### Sincronização em tempo real
- Enquanto a lista compartilhada está aberta, qualquer alteração feita por qualquer participante (adicionar, editar, remover item, marcar como comprado) aparece quase instantaneamente para todos os outros participantes.
- Indicador de status no header: "Compartilhada • ao vivo" (sincronizado), "dados salvos" (offline, usando cache local) ou erro de sincronização.
- Alterações locais também são gravadas no Firestore, com proteção contra falha de rede (reverte a alteração local se a escrita falhar).
- Toda atividade renova o prazo de expiração da lista (90 dias).

### Sair da lista compartilhada
- Opção no header para sair, com confirmação explicando que a pessoa deixa de receber atualizações (os demais continuam com acesso normal).
- Mesma tela é usada quando a lista deixou de existir.

> Observação de segurança: o acesso à lista compartilhada é controlado pela posse do link/código (não há lista explícita de "membros"); qualquer pessoa com o código pode ler e editar a lista.

## Histórico de compras

- Lista de todas as compras já finalizadas, com nome da loja, data/hora, total de itens, quantidade de promoções e valor total.
- Toque em uma compra abre o detalhe.

## Detalhe de uma compra / Reativar lista

- Visualização somente leitura dos itens de uma compra finalizada, com busca, filtro por categoria e "somente promoções".
- **Excluir** a compra do histórico.
- **Reativar lista**: recria uma compra em andamento a partir dos itens dessa compra finalizada (útil para repetir uma compra recorrente). Se já existe uma lista em andamento, pede confirmação antes de substituí-la.

## Máscara de preço

- Campo de preço formata o valor digitado automaticamente no padrão brasileiro (ex.: `1.234,56`) conforme o usuário digita, como um campo de valor monetário.
- Preço é opcional ao adicionar um item; se informado, deve ser maior que zero.

## Reconhecimento de voz

- Ditado por voz no campo "nome do produto" ao adicionar item, com transcrição em tempo real (funciona na Web e nos apps nativos).

## Foto de produto/promoção

- Tirar foto pela câmera ou escolher da galeria para registrar o produto ou a promoção vista na gôndola.
- Visualização das fotos em tela cheia com zoom.

## Categorias, busca e filtros

- 12 categorias fixas (Hortifruti, Açougue/Peixaria, Padaria, Laticínios e Frios, Mercearia, Bebidas, Limpeza, Higiene Pessoal, Congelados, Pet Shop, Farmácia, Outros), cada uma com ícone e cor.
- Usadas para categorizar produtos, agrupar a lista em seções e filtrar tanto na lista em andamento quanto no histórico.

## Modo escuro

- Alternância de tema claro/escuro, com preferência salva no dispositivo.

## Anúncios

- Banner fixo na tela (fora da versão web).
- Anúncio intersticial exibido ao finalizar uma compra, com timeouts de segurança para não travar a navegação caso o anúncio falhe.

## Persistência offline

- Lista em andamento, histórico de compras, preferência de tema e sessão são salvos localmente no dispositivo — o app funciona sem internet no fluxo normal (não compartilhado).
- A lista compartilhada depende do Firestore, mas se beneficia de cache local para leitura offline.
