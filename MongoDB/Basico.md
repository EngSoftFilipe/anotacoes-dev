# MongoDB

## Diferença entre SQL e NoSQL

| | **SQL** | **NoSQL (MongoDB)** |
|---|---|---|
| **Nomenclatura** | Tabela, linha, coluna | Coleção, documento, campo |
| **Formato** | Tabelas com colunas fixas, definidas antes | Documentos flexíveis, parecidos com JSON |
| **Mudar a estrutura** | Precisa alterar a tabela (`ALTER TABLE`) | É só salvar o documento com o campo novo |
| **Dados relacionados** | Tabelas separadas, ligadas por chave (`JOIN`) | Normalmente ficam juntos, um dentro do outro |
| **Linguagem** | SQL, igual em todo banco (MySQL, PostgreSQL…) | Cada banco tem a sua (`find`, `updateOne`…) |
| **Crescer** | Normalmente com uma máquina **mais forte** | Com **mais máquinas** dividindo os dados |
| **Garantias** | Foco total em consistência | Foco em flexibilidade e velocidade |
| **Uso típico** | Banco, pagamentos, estoque, ERP | Catálogo, chat, logs, redes sociais, cache |
| **Registros do mesmo tipo** | Campos que só alguns registros usam, viram colunas cheias de NULL | Cada registro tem só os campos que precisa |
| **Estilo** | Padronizado | Ter cada documento de um jeito, mas podendo deixar tudo padronizado se quiser |

## Como usar o MongoDB no terminal?

- Vai no terminal e digita "mongosh" 
- Irá aparecer um texto mostrando o ID, o servidor, a versão do Mongo e a versão do Mongosh
- Embaixo de tudo isso vai ter um 'test>'
- Tudo pronto para usar
- 'exit' para sair do Mongosh

## Mapa

> Mapa gerado com ajuda de IA

```text
Banco (nomeDoBanco)                          → entra com "use nomeDoBanco"
 └── Coleção (estados)                  → usa no comando: db.estados
      ├── Documento (Acre)          → o bloco { } inteiro
      │    ├── Campo: nome: 'Acre'
      │    ├── Campo: sigla: 'AC'
      │    └── Campo: regiao: 'Norte'
      │
      └── Documento (São Paulo)
           ├── Campo: nome: 'São Paulo'
           ├── Campo: sigla: 'SP'
           └── Campo: cidades: [ ]  → array de subdocumentos
                ├── Subdocumento (Guarulhos)
                └── Subdocumento (São Paulo)
```

## Se eu quiser subir um documento que está no 'test' para o banco?

- É usada uma função em JS (Mongosh usa JS por padrão)

```js
use test // Seleciona o banco de teste
db.colecao.find().forEach(doc => db.getSiblingDB("colecao").colecao.insertOne(doc))
// uma função onde vai achar, percorrer e adicionar os documentos um por um no banco selecionado

use colecao // Para sair de 'test' e entrar no seu banco
db.colecao.find() // Para ver se os dados foram movidos do 'test' para cá
```

## Quero filtrar uma coleção 'estados' apenas pelo nome e sigla, como faço?

```js
db.estados.find({}, {nome: 1, sigla: 1, _id: 0})

// Para quebrar linha no terminal do Mongo, basta digitar uma parte e dar enter e aparecerá um '|' a cada quebra
// Ambos são válidos

db.estados.find(
    {}, {nome: 1, sigla: 1, _id: 0}
)


// { } --> Filtro vazio, portanto seleciona todos os documentos
// Nome, sigla e _id, podem ser 0 e 1, (0 --> false | 1 --> true)
// _id é o único campo que aparece por padrão, por isso deve-se deixa-lo recebendo 0, caso queira esconder

Resultado:

[
  { nome: 'Acre', sigla: 'AC' },
  { nome: 'Alagoas', sigla: 'AL' },
  { nome: 'Rio de Janeiro', sigla: 'RJ' },
  { nome: 'São Paulo', sigla: 'SP' }
]
```

## E para apagar?

```js
db.estados.deleteOne({
    _id: ObjectId('6add6de103cacf1d7acb5436')
    })

// Deletar pelo ObjectId é mais simples, cada '_id' é único
```

## E se eu quiser editar?

```js
db.estados.updateOne(
    { sigla: "SP" }, // quem
    { $set: { regiao: "Sudeste" }} // o que muda
)

// Costuma-se usar o '$' para casos especiais
```
```js
// Se tiver um documento com array

{
  _id: ObjectId('6adfe7f2e1023a2ce8c5c6ad'),
  nome: 'São Paulo',
  sigla: 'SP',
  regiao: 'Sudeste',
  cidades: [
    { nome: 'Guarulhos', area: 133.9, populacao: 3265781 },
    { nome: 'São Paulo', area: 145.3, populacao: 8765781 }
  ]
}

db.estados.updateOne(
    { sigla: "SP", "cidades.nome": "Guarulhos" },
    { $set: { "cidades.$.populacao": 1291771 } }
)
```

## Aggregate
- É um método que possui mais de 40 estágios
- Todos os stages só podem ser usados no pipeline

### Project
- É um stage do aggregate
- Faz o mesmo papel do find(), porém `consegue editar dados (mostrar/esconder campos/criar campos novos na saída) sem alterar no banco`
- Só existe dentro do pipeline do aggregate (Precisa chamar o aggregate para poder usar o project. EX: Só pode chamar um filter, map, reduce em Java se usar um Stream() )
- Muito útil para `mostrar centavos em reais` evitando problemas com ponto flutuante. (Lembrando que quem evita o problema de ponto flutuante é guardar em centavos, e não o $project em si)

```js
// Dados no banco
{nome: "Filipe", saldoCentavos: 15000}

db.contas.aggregate([
  {$project: {
    _id: 0,
    nome: 1,
    saldoReais: {$divide: ["$saldoCentavos", 100]}
  }}
])

// O output
{nome: 'Filipe', saldoReais: 150}
```
### Match
- É um stage do aggregate
- É um `filtro`, funciona igual o .filter() do Java, JS e etc
- Funciona como uma `peneira` e só passa o que deve

```js
// Documentos na coleção 'estados'
{ nome: "Sergipe", sigla: "SE", regiao: "Nordeste" }
{ nome: "Bahia", sigla: "BA", regiao: "Nordeste" }
{ nome: "Minas Gerais", sigla: "MG", regiao: "Sudeste" }
{ nome: "Paraná", sigla: "PR", regiao: "Sul" }
{ nome: "Amazonas", sigla: "AM", regiao: "Norte" }

db.estados.aggregate([
  {$match: {regiao: "Nordeste"}}
])

// O output
{ nome: "Sergipe", sigla: "SE", regiao: "Nordeste" }
{ nome: "Bahia", sigla: "BA", regiao: "Nordeste" } 
```
### Sort
- Ordena os documentos: `1` = crescente (A→Z, 0→9) e `-1` = decrescente (Z --> A, 9 --> 0). No Java Stream, é o `sorted`

```js
// Documentos
{ nome: "Bahia", populacao: 14141626 }
{ nome: "Sergipe", populacao: 2210004 }
{ nome: "Pernambuco", populacao: 9058931 }

// Consulta (do mais populoso pro menos)
db.estados.aggregate([
  { $sort: { populacao: -1 } }
])

// Resultado
{ nome: "Bahia", populacao: 14141626 }
{ nome: "Pernambuco", populacao: 9058931 }
{ nome: "Sergipe", populacao: 2210004 }
```

### Group
- É um stage do aggregate
- Separa os documentos em grupos pelo `_id` e calcula algo em cada grupo.
- `$sum: 1` conta quantos documentos tem em cada grupo. No Java Stream, é o `groupingBy`; no SQL, o `GROUP BY`

```js
// Documentos
{ nome: "Sergipe", regiao: "Nordeste" }
{ nome: "Bahia", regiao: "Nordeste" }
{ nome: "Paraná", regiao: "Sul" }

// Consulta (quantos estados por região)
db.estados.aggregate([
  { $group: { _id: "$regiao", quantidade: { $sum: 1 }}}
])

// Resultado
{ _id: "Nordeste", quantidade: 2 }
{ _id: "Sul", quantidade: 1 }
```

### Unwind
- Transforma array em documentos. Um pra cada item

```js
// Documento
{ nome: "Sergipe", sigla: "SE", cidades: ["Aracaju", "Lagarto", "Itabaiana"] }

// Consulta 
db.estados.aggregate([
  { $unwind: "$cidades" }
])

// Resultado
{ nome: "Sergipe", sigla: "SE", cidades: "Aracaju" }
{ nome: "Sergipe", sigla: "SE", cidades: "Lagarto" }
{ nome: "Sergipe", sigla: "SE", cidades: "Itabaiana" }
```
> 1 documento com 3 cidades no array virou 3 documentos, um para cada cidade. Documentos sem o array somem do resultado.

## Update
- Funciona com a mesma lógica do SQL, porém usando a sintaxe do Mongo (JS se for Mongosh)

```js
db.colecao.updateOne({ quem }, { o que muda })
```

### Métodos
- `updateOne`	altera o primeiro documento que bater com o filtro
- `updateMany` altera todos os que baterem com o filtro
- `replaceOne` substitui o documento inteiro por outro (os campos não citados somem)

```js
// Um documento: Sergipe ganha a cidade Aracaju
db.estados.updateOne(
  { sigla: "SE" },
  { $push: { cidades: "Aracaju" } }
)

// Vários documentos: todo o Nordeste ganha um campo novo
db.estados.updateMany(
  { regiao: "Nordeste" },
  { $set: { pais: "Brasil" } }
)

// Item dentro do array: o $ aponta pra cidade que o filtro achou
db.estados.updateOne(
  { sigla: "SP", "cidades.nome": "Guarulhos" },
  { $set: { "cidades.$.populacao": 1291771 }}
)
```

### Operadores
- `$set` Cria ou altera um campo	
```js
{ $set: { regiao: "Nordeste" }}
```
- `$unset` Remove um campo	
```js
{ $unset: { apelido: "" }}
```
- `$inc` Soma (ou subtrai, com número negativo) 
```js
{ $inc: { populacao: 1000 }}
```
- `$mul` Multiplica 
```js
{ $mul: { populacao: 2 }}
```
- `$rename`	Renomeia um campo	
```js
{ $rename: { regiao: "macroRegiao" }}
```
- `$push`	Adiciona um item no array	
```js
{ $push: { cidades: "Lagarto" }}
```
- `$addToSet`	Adiciona no array só se ainda não existir	
```js
{ $addToSet: { cidades: "Lagarto" }}
```
- `$pull`	Remove um item do array	
```js
{ $pull: { cidades: "Lagarto" }}
```

### Upsert
- Atualiza ou cria (cria se não existir)
- Parecido com o método `PUT` do `http` 
```js
db.estados.updateOne(
  { sigla: "TO" },
  { $set: { nome: "Tocantins", regiao: "Norte" } },
  { upsert: true }
)
```
### Lookup

- É um stage do aggregate
- Busca documentos de **outra coleção** e junta no resultado. É o `JOIN` do Mongo
- O resultado sempre vem dentro de um **array**, mesmo quando encontra um só

```js
// Coleção 'estados'
{ _id: 1, nome: "Sergipe", sigla: "SE" }
{ _id: 2, nome: "Bahia", sigla: "BA" }

// Coleção 'empresas' (guarda só o _id do estado)
{ nome: "Padaria do Zé", estadoId: 1 }
{ nome: "Mercado Bom Preço", estadoId: 2 }

// Consulta
db.empresas.aggregate([
  { $lookup: {
      from: "estados",         // coleção onde vai buscar
      localField: "estadoId",  // campo da coleção atual (empresas)
      foreignField: "_id",     // campo da outra coleção que tem ser igual
      as: "estado"             // nome do campo onde o resultado vai entrar
  }}
])

// Resultado
{ nome: "Padaria do Zé", estadoId: 1, estado: [{ _id: 1, nome: "Sergipe", sigla: "SE" }]}
{ nome: "Mercado Bom Preço", estadoId: 2, estado: [{ _id: 2, nome: "Bahia", sigla: "BA" }]}
```

> Para tirar o resultado de dentro do array, é só usar um `$unwind: "$estado"` logo depois do `$lookup`.