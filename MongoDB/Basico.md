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