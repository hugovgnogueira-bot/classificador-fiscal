# Classificador Fiscal Inteligente

> Automação de cadastro e classificação contábil com IA — para qualquer empresa, sem instalação.

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/hugovgnogueira-bot/classificador-fiscal/blob/main/Classificador_Fiscal.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python](https://img.shields.io/badge/Python-3.x-blue.svg)](https://www.python.org/)
[![IA: Groq](https://img.shields.io/badge/IA-Groq%20(gratuito)-orange.svg)](https://console.groq.com)

Classifica automaticamente itens e serviços dentro da taxonomia contábil da **sua empresa**,
preenchendo todos os campos de cadastro a partir de uma descrição textual:

**Natureza · Tipo · Grupo · Subgrupo · Classe · Conta contábil · Código fiscal (NCM ou Serviço) · Decisão de imobilização**

Segue as normas brasileiras (CPC 27 / NBC TG 27) e roda 100% no Google Colab,
com modelos de linguagem abertos via API gratuita.

---

## Índice

- [Comece agora](#comece-agora)
- [O que ele faz](#o-que-ele-faz)
- [Como funciona](#como-funciona)
- [O que você informa](#o-que-você-informa)
- [Estrutura do repositório](#estrutura-do-repositório)
- [A taxonomia](#a-taxonomia)
- [O plano de contas](#o-plano-de-contas)
- [Estoque: onde a automação para](#estoque-onde-a-automação-para)
- [Campos fiscais](#campos-fiscais)
- [Requisitos](#requisitos)
- [Sobre o projeto](#sobre-o-projeto)
- [Limitações](#limitações)
- [Roadmap](#roadmap)
- [Licença](#licença)

---

## Comece agora

1. Clique no botão **Abrir no Colab** acima
2. Crie uma conta gratuita no [Groq](https://console.groq.com) e pegue sua API key
3. Execute os blocos em ordem, de cima para baixo
4. Comece a classificar seus itens

Leva cerca de 5 minutos para ter tudo funcionando.

---

## O que ele faz

- **Classificação completa** — natureza, tipo, grupo, subgrupo, classe e conta contábil de uma vez
- **Separação entre produto e serviço** — você informa a natureza e o modelo só enxerga os tipos daquela natureza, o que evita confundir a máquina com a manutenção da máquina
- **Decisão de imobilização** — decide entre Ativo (imobilizar) ou despesa, com base no valor e nas normas CPC 27 / NBC TG 27
- **Classe Estoque** — compra que entra no almoxarifado aponta só a conta de estoque; a conta de resultado fica com o ERP
- **Código fiscal com validação** — NCM para produtos e Código de Serviço (LC 116/2003) para serviços, escolhidos entre candidatos e conferidos antes de entrar no cadastro
- **Controle de versões** — itens com a mesma classificação viram versões, sem poluir a base com duplicatas
- **Anexo de orçamento** — lê PDF ou imagem do orçamento para enriquecer a análise
- **Classificação em lote** — suba uma planilha, classifique tudo de uma vez
- **Busca semântica** — encontra itens similares mesmo com palavras diferentes
- **Validação humana** — nada entra na base sem sua aprovação
- **Taxonomia customizável** — adapte a estrutura à realidade da sua empresa
- **Validação de integridade** — verifica se toda conta referenciada existe no plano
- **Aprendizado com o histórico** — item semelhante já aprovado por você entra como sinal na validação do código fiscal
- **Medição de precisão** — compare com um gabarito e meça a acurácia

---

## Como funciona

O sistema combina duas técnicas de IA:

1. **Busca semântica (RAG)** — converte descrições em vetores e encontra itens parecidos já validados
2. **Modelo de linguagem** — raciocina sobre a taxonomia e escolhe a melhor classificação

```
Natureza (produto/serviço) + Descrição + Aplicação + Setor + Estoque + Valor + (Orçamento)
                  ↓
      Filtra os tipos da natureza informada
                  ↓
      Busca itens similares na base
                  ↓
   Modelo classifica usando a taxonomia
                  ↓
   Decide: imobilizar ou despesa · aplica a classe Estoque · busca código fiscal
                  ↓
   Você valida → entra na base de conhecimento
```

A base começa **vazia** e cresce só com os itens que você aprova, mantendo a qualidade alta,
sem poluição de catálogos genéricos.

---

## O que você informa

O bloco de classificação pergunta, nesta ordem:

| # | Pergunta | Para que serve |
|---|----------|----------------|
| 1 | É produto ou serviço? (P / S) | Limita os tipos da taxonomia e define se o código fiscal será NCM ou Serviço |
| 2 | Descrição do item | Entrada principal da classificação e da busca semântica |
| 3 | Aplicação | Contexto de uso, decisivo em itens ambíguos |
| 4 | Setor (Produção / Administrativo / Vendas) | **É a classe.** Define a conta de resultado |
| 5 | NCM (opcional, só produto) | Se você tem o NCM da nota, o sistema confere em vez de calcular |
| 6 | A compra entra em estoque? (s/N) | Só para produto. Ver [Estoque](#estoque-onde-a-automação-para) |
| 7 | Valor unitário | Entra na regra de imobilização |

### O que é regra e o que é modelo

O modelo decide só o que exige interpretação: **tipo, grupo e subgrupo**. O resto é consequência
do que você informou, e o sistema aplica mesmo que o modelo sugira outra coisa:

| Campo | Como é decidido |
|-------|-----------------|
| Natureza | Você responde |
| Classe | Entrou em estoque, é Estoque; senão é o setor que você informou; subgrupo de classe única (Ativo, Receita, Provisões) não admite escolha |
| Imobilizar | Decorre da classe ser Ativo |
| Conta contábil | Sai da combinação tipo, grupo, subgrupo e classe na taxonomia |
| NCM | O que você informar tem prioridade sobre qualquer sugestão |

Isso tira do modelo três campos que ele errava por conta própria e torna o resultado auditável:
dado o subgrupo e o setor, a conta é sempre a mesma, com ou sem IA.

Duas conferências de valor acompanham o resultado, e são justamente as que pegam o erro caro:
item classificado como Ativo com valor unitário abaixo do limite, e produto acima do limite que
não foi imobilizado. As duas saem na coluna `alertas`.

---

## Estrutura do repositório

```
classificador-fiscal/
├── Classificador_Fiscal.ipynb    # notebook principal (abra no Colab)
├── README.md
├── LICENSE
├── .gitignore
├── docs/
│   ├── taxonomia_classificacao.xlsx   # taxonomia e regras em planilha, para consulta
│   └── cadastro_de_itens.pptx         # apresentação: por que o cadastro decide o resultado
└── exemplos/
    ├── taxonomia.json            # modelo de taxonomia (4 níveis + imobilização)
    ├── plano_contas.csv          # modelo de plano de contas (Lei 6.404/76)
    ├── ncm.json                  # tabela NCM com contexto hierárquico (produtos)
    ├── codigo_servico.json       # lista de serviços LC 116/2003
    └── exemplo_lote.xlsx         # 41 itens de teste, com NCM do fornecedor e gabarito
```

A planilha em `docs/` traz as regras de decisão, as 162 combinações da taxonomia de exemplo e os
totais por tipo e por classe. É o caminho mais rápido para entender a estrutura sem abrir o JSON.
A apresentação percorre a jornada de um item dentro do ERP e mostra, com números, o que um erro
de classificação custa. A planilha `exemplo_lote.xlsx` alimenta o bloco de classificação em massa
e traz uma aba de gabarito, com o resultado esperado de cada item, para medir a precisão.

---

## A taxonomia

A estrutura tem **4 níveis hierárquicos**:

```
TIPO → GRUPO → SUBGRUPO → CLASSE → conta contábil
```

Cada tipo declara a **natureza** (`Produto` ou `Servico`). O modelo de exemplo traz três:

| Tipo | Natureza | Para que serve |
|------|----------|----------------|
| 1. Imobilizado | Produto | Bens de uso próprio acima do limite, mais o grupo de Intangível (CPC 04) |
| 2. Uso e Consumo | Produto | Materiais que não se incorporam ao produto final e bens duráveis abaixo do limite |
| 9. Servicos | Serviço | Serviços tomados de terceiros, mais os serviços que formam ativo |

Exemplo de um serviço:

```
Tipo: 9. Servicos                          (natureza: Servico)
  Grupo: 7. Locacoes
    Subgrupo: 1. Locacao de Equipamentos
      Classe: Producao (C.I.)  → conta 299 · 4.1.2.06.005
      Classe: Administrativo   → conta 410 · 4.2.2.04.005
      Classe: Vendas           → conta 352 · 4.2.1.04.005
```

Exemplo de um material, com a classe de estoque:

```
Tipo: 2. Uso e Consumo                     (natureza: Produto)
  Grupo: 2. Limpeza, Higiene e Seguranca
    Subgrupo: 2. EPI
      Classe: Estoque          → conta 35  · 1.1.3.04.001
      Classe: Producao (C.I.)  → conta 287 · 4.1.2.05.005
      Classe: Administrativo   → conta 398 · 4.2.2.03.005
      Classe: Vendas           → conta 340 · 4.2.1.03.005
```

As **regras de imobilização** (limite de valor, vida útil, critérios e destino) também ficam na
taxonomia, seguindo o CPC 27 / NBC TG 27 e o limite fiscal de dedução do art. 15 do
Decreto-Lei 1.598/1977, com redação dada pela Lei 12.973/2014.

Subgrupos e grupos podem ter um campo `regra`, um texto curto que o notebook envia ao modelo
junto do menu. É aí que você resolve as dúvidas recorrentes da sua operação, do tipo
"empilhadeira é máquina, não veículo".

Tipos particulares de cada empresa, como matéria-prima, insumos, embalagens, produto acabado e
mercadoria para revenda, **não** fazem parte do modelo, porque variam demais de negócio para
negócio. Quem adotar a automação inclui esses tipos seguindo a mesma estrutura: os números de 3 a 8
ficam livres para isso, e o arquivo de exemplo traz um modelo pronto no bloco
`_orientacao_novos_tipos`.

Veja o modelo completo em [`exemplos/taxonomia.json`](exemplos/taxonomia.json).

---

## O plano de contas

O plano de exemplo segue a estrutura da Lei 6.404/76 (Ativo, Passivo, Receitas, Custos e Despesas)
e tem 446 contas, das quais 337 analíticas. As colunas são:

| Coluna | Conteúdo |
|--------|----------|
| `mascara` | Classificação hierárquica, por exemplo `4.2.2.02.001` |
| `codigo` | Código reduzido, que é a chave usada pela taxonomia |
| `descricao` | Nome da conta |
| `tipo_conta` | `S` para sintética, `A` para analítica |

A arquitetura é de fonte única da verdade: a taxonomia guarda apenas a **referência** (o código
reduzido) e o plano de contas guarda os **detalhes** (máscara e descrição). O bloco de validação
percorre a taxonomia inteira e avisa se alguma conta referenciada não existir no plano, ou se
apontar para uma conta sintética.

Para usar o seu plano, troque o CSV e ajuste os `conta_ref` da taxonomia. Nada mais muda.

---

## Estoque: onde a automação para

Em toda compra de uso e consumo existe uma bifurcação:

- **Consumo imediato** — a nota vai direto para resultado, na conta do setor que consome
- **Entrada em estoque** — a nota vai para o estoque e a despesa só nasce na requisição

No segundo caso o classificador indica apenas a conta de estoque, através da classe **Estoque**,
que existe em todos os subgrupos de Uso e Consumo e aponta para `1.1.3.04.001` (código 35),
Materiais de Uso e Consumo.

A partir daí, a conta de resultado que recebe a baixa é definida pela parametrização do ERP na
requisição (centro de custo, natureza de movimento, destino do material). **Isso é regra do
sistema da empresa e não faz parte desta automação.** Tentar adivinhar essa conta no cadastro
seria duplicar uma regra que já existe no ERP, com o risco de as duas divergirem.

Quem decide entre Estoque e setor é você, na pergunta 5. Se você marcar estoque e o item cair em
um subgrupo sem essa classe, um notebook de R$ 5.000 que vai para o imobilizado, por exemplo, o
sistema mantém a classe do setor e avisa na tela, em vez de forçar a conta de estoque.

---

## Campos fiscais

O sistema preenche o código fiscal correto conforme a natureza informada:

| Natureza | Código fiscal | Fonte |
|----------|---------------|-------|
| Produto | NCM (Nomenclatura Comum do Mercosul) | Tabela oficial da Receita Federal |
| Serviço | Código de Serviço | Lista de Serviços da LC 116/2003 |

### A busca devolve candidatos, não uma decisão

**O NCM que você informar vale mais que qualquer sugestão.** O fornecedor manda o NCM na nota, e
esse é o dado bom. Na classificação em lote, preencha a coluna `ncm`; no item avulso, responda a
pergunta do NCM. O sistema então não calcula: confere se o código existe na tabela e se o
capítulo conversa com o tipo de item, avisa quando algo não fecha, e ainda economiza uma chamada
à API. O cálculo abaixo só roda quando o campo vem vazio.

Sem o NCM informado, a escolha tem quatro etapas:

1. **Capítulo e tradução** — o modelo diz em que capítulo da NCM o item cai e reescreve a
   descrição nos termos da tabela. Ninguém escreve "máquina automática para processamento de
   dados" na requisição de compra, escreve "notebook". Isso sai na mesma resposta da
   classificação contábil, sem custar uma chamada extra
2. **Busca dentro do capítulo** — os 12 códigos mais próximos, procurados só naquele ramo
3. **Escolha** — o modelo enquadra entre esses 12 e justifica
4. **Conferência** — a escolha passa por testes antes de ser aceita

Se nada dentro do capítulo servir, uma última busca percorre a tabela inteira e o resultado vem
marcado como `revisar`, com o motivo "fora do capítulo previsto". É melhor entregar uma sugestão
para conferir do que um campo vazio.

Restringir ao capítulo faz diferença de recall. Em um teste com 20 itens, o código correto
aparecia entre os candidatos em 60% das vezes buscando na tabela inteira com 10 candidatos, e
em 90% buscando com 10 candidatos dentro do capítulo certo. Para chegar aos mesmos 90% sem o
filtro seriam necessários 100 candidatos, o que triplica o consumo de tokens e ainda dispersa
o modelo entre linhas parecidas.

O que é testado:

| Teste | O que pega |
|-------|------------|
| O código está entre os candidatos | Código inventado pelo modelo é descartado |
| O capítulo escolhido é o previsto | Dipirona caindo em adubos, GLP em medidores de gás |
| Posição do escolhido na busca | Enquadramento resgatado do fim da lista |
| Linha "Outros" havendo linha específica | O erro clássico de cair no genérico |
| Divergência com item semelhante já aprovado | Dois cadastros parecidos com NCM diferente |
| Coerência com o subgrupo contábil (serviço) | Manutenção na contabilidade e consultoria na nota |

O status **não** vem da confiança que o modelo declara. Em um teste real, 8 de 13 itens
marcados como corretos tinham NCM errado, com confiança média de 0,99. Confiança declarada não
é medida de acerto, e por isso ela entra apenas como sinal fraco: os testes acima é que
decidem.

Se nenhum candidato servir, o modelo reescreve a descrição do item nos termos da nomenclatura
ou da lista de serviços, e a busca roda de novo. No máximo duas voltas, porque só faz sentido
repetir quando há informação nova: perguntar duas vezes a mesma coisa devolve a mesma resposta.

Cada item sai com um status, que vira coluna na classificação em lote:

- `informado` — veio de você, conferido contra a tabela
- `ok` — passou em todos os testes
- `revisar` — foi enquadrado, mas um dos testes acendeu a luz amarela
- `nao_aplicavel` — o item não tem código de serviço a informar (ver abaixo)
- `nao_encontrado` — nada compatível; o campo fica vazio para enquadramento manual

Na revisão do Excel, filtrar por `revisar` e `nao_encontrado` separa o que precisa de olho
humano. Para desligar a validação e voltar ao primeiro resultado da busca, mude
`VALIDAR_NCM` ou `VALIDAR_SERVICO` para `False` no bloco de preparação.

### Nem toda despesa tem código de serviço

Obrigar o modelo a escolher sempre um subitem da LC 116 produz enquadramento errado com cara de
certo. Locação de empilhadeira não é serviço: a Súmula Vinculante 31 do STF afastou o ISS sobre
locação de bem móvel. Frete interestadual é ICMS. Seguro, juro, multa e tributo não são serviços
de terceiros.

Nesses casos o campo sai vazio, com status `nao_aplicavel` e a justificativa, em vez do subitem
mais parecido. O sistema sugerindo `15.03 Locação e manutenção de cofres` para uma empilhadeira
alugada não é um erro de busca, é uma pergunta mal formulada.

### Por que a tabela NCM precisa de contexto

Boa parte das descrições da NCM só faz sentido junto do nível superior: 2.056 entradas são
literalmente "Outros" e outras milhares são fragmentos como "De carga radial". Comparar a
descrição de um item com esses textos não leva a lugar nenhum.

Por isso o `ncm.json` de exemplo traz o campo `descricao_busca`, que junta a descrição do item
à posição e à subposição a que ele pertence:

```
8482.10.10
  descricao        "De carga radial"
  descricao_busca  "De carga radial. outras partes de rolamentos. rolamentos de esferas"
```

A busca usa esse campo; o que aparece para você continua sendo a descrição oficial. A lista da
LC 116 não precisa disso, porque cada subitem já é uma frase completa.

---

## Requisitos

- Uma conta gratuita no [Groq](https://console.groq.com) (sem cartão de crédito)
- Navegador — todo o resto roda no Google Colab

### Configurando a chave de forma segura

O notebook lê a chave do **cofre de Secrets do Colab**. Configure uma vez:

1. No Colab, clique no ícone de chave (Secrets) no menu lateral
2. Adicione um secret com nome `GROQ_API_KEY` e cole sua chave no valor
3. Ative o acesso ao notebook

Assim a chave nunca fica exposta no código.

---

## Sobre o projeto

O objetivo é tornar acessível a classificação fiscal automatizada para empresas brasileiras de
qualquer porte, com uma estrutura aberta que cada uma adapta à própria realidade. Nasceu como
trabalho de conclusão de pós-graduação e segue em evolução.

Contribuições originais:

- **Taxonomia contábil brasileira estruturada** em 4 níveis, aberta e customizável
- **Separação por natureza** — produto e serviço percorrem ramos distintos da taxonomia, o que reduz o erro mais comum desse tipo de cadastro
- **Pipeline RAG em português** aplicado ao contexto fiscal brasileiro
- **Lógica de imobilização** baseada nas normas CPC 27 / NBC TG 27 e no limite fiscal de dedução
- **Fronteira explícita com o ERP** — a classe Estoque delimita onde a automação termina e a parametrização do sistema da empresa começa
- **Integração de códigos fiscais** (NCM e Lista de Serviços LC 116) via busca semântica
- **Arquitetura de fonte única da verdade** — taxonomia e plano de contas separados, vinculados por referência

---

## Limitações

Este é um sistema de **classificação assistida**: para casos de baixa confiança, a validação
humana continua recomendada. Ele acelera e padroniza o trabalho, mas não substitui o
julgamento contábil final. As sugestões de código fiscal (NCM e serviço) devem ser conferidas
por um profissional, pois a responsabilidade tributária é da empresa.

A validação dos códigos fiscais confere o que é conferível: se o código existe na tabela, se o
modelo não caiu no genérico e se o enquadramento conversa com a classificação contábil. Ela não
aplica as regras gerais de interpretação da NCM nem conhece a lista de serviços e a alíquota de
ISS do seu município. O status `ok` significa que passou nos testes, não que a classificação
fiscal está correta.

A taxonomia de exemplo cobre imobilizado, uso e consumo e serviços. Os tipos ligados ao produto da
empresa ficam a cargo de quem adota a automação, pelo motivo explicado acima.

O exemplo que acompanha o repositório é de **indústria no lucro real**. Uma empresa do Simples
Nacional, uma prestadora de serviços ou um comércio têm plano de contas mais enxuto e outra
lógica de crédito, então a taxonomia delas será menor e diferente. A estrutura aguenta, mas os
arquivos de `exemplos/` são ponto de partida, não modelo a copiar.

Os arquivos são de preenchimento obrigatório por quem adota a ferramenta: taxonomia, plano de
contas, regras de imobilização, tabela NCM e lista de códigos de serviço. Nada disso é opcional,
e é essa definição prévia que faz a classificação ser auditável em vez de opinião.

No plano gratuito da Groq os dois limites que aparecem são o de tokens por minuto e o de tokens
por dia. Entre classificação, código fiscal e os tokens de raciocínio do modelo, cada item consome perto
de 3.500 tokens, o que dá algo em torno de 55 itens por dia. O consumo cai mais se você reduzir
o esforço de raciocínio do modelo: a constante `ESFORCO_RACIOCINIO`, no bloco de preparação,
aceita `"low"` e corta quase pela metade o que é gasto pensando. Vale medir o efeito na precisão
pelo Bloco 10 antes de adotar. Para o cadastro
do dia a dia sobra folga; para reclassificar um catálogo inteiro, não.

A classificação em lote lida com os dois limites de formas diferentes. O limite por minuto ela
absorve: espaça as chamadas e espera os poucos segundos que a API pedir. O limite diário ela
reconhece pela espera longa, encerra o lote e guarda o que já foi feito. O resultado é gravado
a cada item, então uma queda do Colab ou o fim da cota não custam nada: o que rodou sai pelo
Bloco 9 e os itens que faltaram vêm prontos em `itens_pendentes.xlsx`, para você subir no dia
seguinte.

A base de conhecimento vive na memória da sessão do Colab. A memória de verdade é o ERP da
empresa: baixe a listagem de itens ativos já classificados e carregue na carga inicial no começo
de cada sessão, ou ao menos uma vez por semana. Incluindo as colunas `ncm` e `codigo_servico`
nessa listagem, o histórico fiscal também passa a valer como sinal, e um item novo que divergir
de um parecido já cadastrado sai marcado para conferência.

---

## Roadmap

- [ ] Persistência da base vetorial entre sessões
- [ ] Votação por consistência no código fiscal (três execuções, aceita com duas iguais)
- [ ] Taxonomia de materiais completa com NCM integrado
- [ ] Fine-tuning com dados setoriais
- [ ] Versão multi-empresa (SaaS)
- [ ] Integração direta com ERPs

---

## Licença

Distribuído sob a licença MIT. Veja [LICENSE](LICENSE) para mais detalhes.

## Contribuindo

Contribuições são bem-vindas. Abra uma [issue](https://github.com/hugovgnogueira-bot/classificador-fiscal/issues)
ou envie um pull request.

---

Desenvolvido por **Hugo Nogueira** · [LinkedIn](https://www.linkedin.com/in/hugo-nogueira-6a152b117)

*Classificador Fiscal Inteligente · 2026*
