# projeto-extensionista-2026-1
Um projeto extensionista para a UEMG no primeiro semestre de 2026


# 🔌 Comunicação entre Frontend (JavaScript) e Backend (FastAPI)

Este documento define como a nossa interface envia dados e como o servidor em Python processa cada requisição. O Python não lê o HTML, CSS ou JavaScript diretamente; a comunicação acontece exclusivamente por meio de rotas (endpoints) e verbos HTTP, utilizando o formato JSON como linguagem universal.

---

## 🔁 Métodos HTTP e Integração de Código

Cada ação que o frontend deseja realizar possui um método HTTP correspondente. O FastAPI intercepta essas requisições usando decoradores específicos (`@app.get`, `@app.post`, etc.).

# 1. GET (Leitura de Dados)
Utilizado quando o frontend precisa buscar informações do servidor. Não envia dados no corpo (`body`) da requisição.

#### Frontend (JavaScript)
```javascript
// O fetch realiza uma requisição GET por padrão
fetch('[https://api.com/produtos](https://api.com/produtos)') ---- esse link é onde de acordo com a nossa api vai estar o JSON
    .then(response => response.json())
    .then(dados => console.log("Produtos recebidos:", dados))
    .catch(erro => console.error("Erro ao buscar:", erro));
```

#### Backend (Python)
```Python
from fastapi import FastAPI

app = FastAPI()

@app.get("/produtos") ---Aqui fala o caminho onde esta ou ira ficar o JSON
def listar_produtos():
    # O Python processa a lógica e retorna um dicionário/lista
    lista_de_produtos = ["Notebook", "Mouse", "Teclado"]
    return {"produtos": lista_de_produtos}  # Convertido automaticamente para JSON
```

# 2. Post (Criação de Dados)

Utilizado para enviar novos dados do frontend para serem salvos ou processados no servidor. Os dados são enviados estruturados no corpo da requisição.

### Frontend (JavaScript)

```javascript
const novoProduto = { nome: "Monitor", preco: 1200.00 };

fetch('[https://api.com/produtos](https://api.com/produtos)', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json' // Informa ao servidor que o corpo é um JSON
    },
    body: JSON.stringify(novoProduto) // Transforma o objeto JS em string JSON
})
.then(response => response.json())
.then(dados => console.log("Confirmação de cadastro:", dados));
```

### Backend (Python)

```Python
from pydantic import BaseModel

# Define a estrutura de dados esperada pelo backend
class Produto(BaseModel):
    nome: str
    preco: float

@app.post("/produtos")
def criar_produto(produto: Produto):
    # O FastAPI valida automaticamente se os tipos enviados do JS estão corretos
    print(f"Inserindo no banco: {produto.nome} - R$ {produto.preco}")
    return {"status": "sucesso", "item_criado": product}
````

# 3. PUT (Atualização completa)

Utilizado para modificar um registro já existente. O identificador do item (ID) é enviado diretamente na URL, e os novos dados vão no corpo da requisição.

### Frontend (JavaScript)

```javascript
const dadosAtualizados = { nome: "Notebook Pro", preco: 4500.00 };
const idDoProduto = 42;

fetch(`https://api.com/produtos/${idDoProduto}`, {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(dadosAtualizados)
})
.then(response => response.json())
.then(dados => console.log("Item atualizado:", dados));
```

### Backend (Python)

```Python
@app.put("/produtos/{produto_id}")
def atualizar_produto(produto_id: int, produto: Produto):
    # produto_id é capturado da URL; produto é capturado do corpo do JSON
    return {
        "mensagem": f"Produto ID {produto_id} foi modificado.",
        "dados_novos": produto
    }
```
# 4.DELETE (Remoção de dados)

Utilizado para excluir um registro do sistema. Assim como o GET, normalmente não envia dados no corpo, apenas o ID do alvo na URL.

### Frontend (JavaScript)

```javascript
const idParaDeletar = 42;

fetch(`https://api.com/produtos/${idParaDeletar}`, {
    method: 'DELETE'
})
.then(response => response.json())
.then(dados => console.log("Status da exclusão:", dados));
```

### Backend (Python)

```Python
@app.delete("/produtos/{produto_id}")
def deletar_produto(produto_id: int):
    # Executa a lógica de remoção no banco de dados usando o ID recebido
    return {"status": "sucesso", "mensagem": f"Produto ID {produto_id} removido."}
```
### Erros de validação comum

Erro 404 (Not Found): Ocorre quando a URL chamada no JavaScript não existe no backend ou o método HTTP está incorreto para aquela rota.

Erro 422 (Unprocessable Entity): Erro gerado automaticamente pelo FastAPI quando o JSON enviado pelo JavaScript não possui as chaves ou os tipos de dados esperados pelo modelo do Python (ex: enviar preco como string em vez de número).

Alinhamento de Chaves: Os nomes das propriedades enviadas no objeto JavaScript precisam corresponder exatamente aos atributos definidos na classe do FastAPI.